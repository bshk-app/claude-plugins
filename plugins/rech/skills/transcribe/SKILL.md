---
name: transcribe
description: Transcribe audio or video recordings — meetings, calls, interviews, voice notes, lectures (.m4a .mp3 .wav .caf .aiff .flac .mp4 .mov) — into text with speakers and timestamps, locally on this Mac with the `rech` CLI. Use when the user asks to transcribe or "расшифровать" a recording, wants the text, quotes or speakers of an audio/video file, or asks what was said in it.
allowed-tools: Bash(rech *)
---
# Transcribing recordings with `rech`

`rech` transcribes on this Mac; audio never leaves it. It detects each recording's language and
picks the model for it: GigaAM for Russian, Parakeet for most European languages, others where
they are better.

If `rech` is missing (`command -v rech` fails), tell the user to install it with
`brew install bshk-app/tap/rech` and stop.

## Commands

```sh
# Plain transcript, one line per phrase: [00:00:01.2 – 00:00:04.8] text
rech transcribe <file>

# Who said what: label speakers
rech transcribe <file> --diarize
rech transcribe <file> --speakers 3          # exact count, when the user knows it

# Machine-readable: phrases with speaker, start, end, text and per-word timings
rech transcribe <file> --format json -o <file>.json

# Subtitles
rech transcribe <file> --format srt -o <file>.srt

# A Tish recording folder has two tracks: the user's mic and the other side of the call
rech transcribe mic_raw.caf --speaker-track speaker_raw.caf --diarize
```

- `--language ru` (or `en`, …) skips language detection; pass it when the user said the
  language. `--model gigaam|parakeet|qwen|voxtral|cohere` overrides the choice; only use it when
  the user asks for a model.
- stdout carries only the transcript; progress and the model used go to stderr. Report the
  `[rech] model=…` line from stderr when the user cares which model ran.
- The first run downloads models and is slower. For recordings longer than ~20 minutes, write
  to a file with `-o` and run the command with a long timeout (up to 10 minutes) instead of
  streaming the transcript into the conversation.

## After transcribing

- For long transcripts, read the output file in parts rather than printing it whole.
- Quote the transcript as-is; do not silently correct it. Speech recognition mishears names,
  terms and code identifiers — when you rely on a word that may be misheard, say so.
- Summaries, action items and translations are your job: run `rech` for the text, then work on
  the text.
