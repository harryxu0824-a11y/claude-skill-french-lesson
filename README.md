# claude-skill-french-lesson

A [Claude Code](https://claude.com/claude-code) Skill that turns a weekly French-class lesson folder (slide deck PDF + homework PDF + audio) into **one self-contained, interactive HTML study page** — with embedded variable-speed audio, sentence-level "intensive listening" (精听) controls, click-to-pronounce vocabulary, and auto-generated fill-in-the-blank homework.

Built for TCF Canada exam prep, but the pattern generalizes to any recurring lesson format (podcasts, language classes, recorded lectures).

## Why this exists

The naive way to use an LLM for studying is to open a chat every time you want to review: "read this to me," "explain this grammar point," "quiz me." That's a **pay-per-use** pattern — every review session burns tokens repeating the same setup work.

This skill instead treats one lesson as a **one-time build**: the model reads the lesson materials once (~45k tokens) and produces a static HTML artifact. After that, review is free — forever, offline, on any device.

| | Per-review chat | This skill |
|---|---|---|
| Cost after first pass | tokens every time | **$0 / 0 tokens** |
| Works offline | no | yes |
| Sentence-level replay/loop | no | yes |
| Click-to-pronounce | no | yes |

### The three-layer split that makes it cheap

1. **The model** reads unstructured source material (slide images, PDFs) and produces *structure* — this is the only step that costs tokens.
2. **Local tools** handle anything binary or perceptual:
   - `ffmpeg` converts audio for processing
   - [`whisper.cpp`](https://github.com/ggml-org/whisper.cpp) (running fully offline) generates per-sentence timestamps for intensive-listening controls — the model never "listens" to audio
   - a small Python script base64-embeds the audio into the HTML (embedding it via model output would be catastrophically token-expensive)
3. **The browser** carries every interactive feature at zero further cost: `<audio>` + `playbackRate` for variable speed, the Web Speech API (`speechSynthesis`) for click-to-pronounce words, plain CSS/JS for the "cover the translation, quiz yourself" toggle.

## What the output looks like

See [`example-output.html`](example-output.html) — a real generated page (French transportation vocabulary lesson). Open it in a browser to try:

- **⭐ Pattern overview** — the lesson's core sentence patterns distilled to the top, before the detail
- **Vocabulary tables** — every word + the teacher's margin notes, with a "🙈 cover translations" self-test toggle
- **Sentence-level intensive listening (精听)** — every dialogue line gets a ▶ (play just this sentence, auto-stop) and 🔁 (loop this sentence) button, synced to the real audio via local Whisper timestamps
- **Click-to-pronounce** — click any French word to hear it spoken (system TTS)
- **Grammar deep-dive** — organized by the lesson's own slide headings, not a generic template
- **Generated sentence-construction homework** — 10 prompts tied to this lesson's patterns, answers hidden behind `<details>`

(Audio is stripped from the example file — it's normally a self-contained ~3MB file with the lesson's real audio embedded.)

## Install

```bash
mkdir -p ~/.claude/skills/french-lesson
cp SKILL.md ~/.claude/skills/french-lesson/SKILL.md
```

Prerequisites for the intensive-listening feature (one-time setup):

```bash
brew install ffmpeg whisper-cpp
mkdir -p ~/.cache/whisper-models
curl -L -o ~/.cache/whisper-models/ggml-base.bin \
  https://huggingface.co/ggerganov/whisper.cpp/resolve/main/ggml-base.bin
```

## Usage

Drop a lesson folder (slide PDF + homework PDF + audio files) somewhere Claude Code can read it, then:

```
/french-lesson do this lesson
```

Claude reads `SKILL.md` and follows its extraction rules, page structure, and audio-embedding steps to produce `Leçon NNN 学习页.html` in the lesson folder.

## License

MIT.
