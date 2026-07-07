---
name: french-lesson
description: Generate a per-lesson TCF Canada French study page (HTML) from the user's teacher's lesson folder — embedded variable-speed audio, vocab tables, transcripts, sentence-pattern summary, and 造句 homework. Use when the user drops a new lesson folder (e.g. "做016的学习页", "新的一课", or a folder like NNN主题听力+阅读).
---

# French lesson study page

Build ONE self-contained `Leçon NNN 学习页.html` inside the lesson folder.

## Inputs (in the lesson folder, e.g. ~/Downloads/NNN主题…/)
- 课件 PDF (~12 slide pages) — read ALL pages with the `pages` param. This is the content source.
- Devoirs PDF — read it; its listening exercise defines the second audio's task.
- 1–2 mp3s (课文听力 + Devoirs 听力). Ignore any `X0.8…` mp3 (speed buttons replace it). NEVER touch the 直播录屏 mp4.

## Extraction rules (follow the 课件's own section headings)
1. **Agenda slide** tells you the lesson structure — mirror it.
2. **Every Vocabulaire slide** → its own table: 词/短语 | 中文 | 搭配·例句. Include ALL words AND the yellow "Notes:" box content (fold notes into the 搭配 column or a 动词辨析 block).
3. **Transcript slides** (课文 + answer slides) → full dialogue with `<span class="hl">` on the expressions the teacher underlined; add one-line 中文 glosses under each dialogue.
4. **句型/表达框** on slides (e.g. "Pour se déplacer en métro ou en bus") → reproduce verbatim in the 句型专题 section.
5. **Reading (Compréhension écrite)** slides → key stats/quotes as tables, flag reusable TCF speaking templates in a `.tip` box.
6. **Devoirs exercises** → give 做题顺序 guidance; transcript + answers go in collapsed `<details>` ("做完题再看"). Work out exercise answers (连线/排序) and include them there.

## Page structure (fixed order)
1. `⭐ 本课必会句型` — numbered pattern list distilled from the whole lesson, one example each; self-test note.
2. Vocab table ① → 3. 课文听力 (audio) → 4. Vocab table ② → 5. 阅读 → 6. `📖 句型专题` (grammar expanded, tables) → 7. Devoirs 听力 (audio) → 8. `✍️ 造句作业`.
3. Top nav (`.toc`) linking to all sections.

## Audio
- Base64-embed each mp3: `data:audio/mpeg;base64,…` via a python script (never through model output).
- Each player gets speed buttons 0.7/0.8/0.9/1.0 (0.8× labeled 跟读 — it's the teacher's standing homework).
- Write the HTML template with `{{AUDIO_X}}` placeholders, then inject with python and write the final file into the lesson folder.

## 精听 (sentence-level intensive listening — the teacher's standing homework "听力断句跟读 0.8倍速")
- Get per-sentence timestamps LOCALLY (zero tokens) with whisper.cpp:
  1. `ffmpeg -i in.mp3 -ar 16000 -ac 1 out.wav`
  2. `whisper-cli -m ~/.cache/whisper-models/ggml-base.bin -l fr -oj -of name out.wav` → segment timestamps; if segments merge two dialogue turns, rerun with `-ml 1 -sow` for word-level times and split at turn boundaries yourself (cross-check with `ffmpeg silencedetect` gaps).
  3. Align whisper text to the known transcript (whisper's French spelling may be off — trust the slides' text, whisper's TIMES).
- Textbook audio starts with a 报幕 announcement ("Page NN, document X…") — give it its own muted `.intro.zh` row, don't fold it into line 1.
- Markup: each dialogue `<p>` gets `data-audio-line="aN" data-s="… " data-e="…"`. Long turns are split into one `<p>` per sentence (speaker label only on the first).
- JS (in template): auto-prepend ▶ (play segment, auto-pause at end) and 🔁 (loop segment, click again to stop) to every `p[data-s]`; highlight the currently-playing line via `timeupdate`; manual seeking outside the segment exits segment mode. Segment playback respects the speed buttons (0.8× = 跟读).
- whisper-cli + ggml-base.bin are already installed (brew whisper-cpp; model at ~/.cache/whisper-models/). If missing on a new machine, reinstall the same way.

## Interactive features (all via browser APIs, zero tokens, zero external services)
- **单词点读**: JS walks every `.fr` element at load, wraps latin-letter tokens (≥2 chars, regex `[A-Za-zÀ-ÿœŒ]{2,}`) in `<span class="w">`, skipping `.zh`/buttons/audio; click speaks the word via `speechSynthesis` with a fr-FR voice (prefer Amélie/Thomas/Audrey), rate 0.8.
- **整句朗读**: auto-prepend a 🔊 `button.say` to each `.dialogue p` and each 造句 参考答案 `.fr`; on click, clone node, strip `.speaker/.zh/.say`, speak at rate 0.85.
- **遮中文自测**: auto-insert a `🙈 遮住中文自测` toggle before each vocab table (#vocab1/#vocab2); toggling adds `.hidezh` which masks the 2nd (中文) column; clicking a masked cell `.peek`s it.
- Add a one-line `.ttsnote` under the subtitle telling the user words are clickable and 整句语调以老师录音为准 (system TTS is for words; real audio is the pronunciation reference).
- After assembling, extract the inline `<script>` and run `node --check` on it before delivering.

## 造句作业
- 10 prompts, each pinned to a specific pattern/word from THIS lesson (use `<code>` for the target structure).
- 参考答案 hidden in `<details>` per item.
- End tip: 先口头说再写；写完发回批改.

## Style
- Reuse the CSS/JS from the 015 template (warm paper palette, .hl highlight, .key pattern card, .tip boxes, serif for French).
- Chinese UI, French in `.fr` class. Mobile-friendly (max-width 880px).
- After generating, SendUserFile with display render.

Template reference: the scratchpad template from lesson 015; if unavailable, recreate from this spec.
