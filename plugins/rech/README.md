# Rech plugin for Claude Code

Speech recognition on this Mac, for agents: you dictate prompts, and Claude transcribes
recordings. Both go through `rech`, which picks the model for the language (GigaAM for Russian).
Audio never leaves the machine — unlike Claude Code's built-in `/voice`, which sends it to
Anthropic.

| Skill | Who runs it | What it does |
|---|---|---|
| `/rech:setup` | you, once (or Claude, when dictation fails) | Installs or updates `rech`, saves your language and microphone, downloads the model, checks microphone access. |
| `/rech:dictate` | you | Tink → speak → pause → Pop. The transcript becomes your message. |
| `rech:transcribe` | Claude, when you ask to transcribe a file | Runs `rech transcribe` on audio or video, with speakers and timings on request. |

## Install

```sh
brew install bshk-app/tap/rech            # 0.4 or later: `rech listen` records the microphone
claude plugin marketplace add bshk-app/claude-plugins
claude plugin install rech@bshk
```

Then run `/rech:setup` in Claude Code. Updates: `claude plugin marketplace update bshk`.

Needs macOS 26 on Apple silicon (for `rech`) and Claude Code 2.1.271 or later (the language
picker in the plugin's settings). To try a local copy for one session without installing:
`claude --plugin-dir path/to/rech`.

## How dictation works

`/rech:dictate` runs `bin/rech-listen`, a wrapper over `rech listen`:

1. The Tink plays, then the microphone opens.
2. A voice-activity model (Silero, the one Tish uses) decides what is speech; recording starts
   at the first word (0.3 s earlier, so it is not clipped) and stops after the pause. Noise and
   clicks do not hold it open the way a volume threshold would.
3. With a known language, the model for it (GigaAM for Russian, Parakeet for most European
   languages) loads while you speak, and the text is ready about 0.7 s after the pause.
   With `auto`, the language is detected first (it needs 3 s of speech) and the model loads
   after: 4–5 s, and short phrases go to Parakeet.

## Settings

| Setting | Where | Default |
|---|---|---|
| Language | `/rech:setup`, or `/config` → Rech (wins) | `auto` |
| Pause that ends a phrase | `/config` → Rech | 1.5 s |
| Start and stop sounds | `/config` → Rech | on |
| Microphone | `/rech:setup` | the system input |

The microphone is not a `/config` setting because a device name is free text: Claude Code puts
plugin settings into the skill's command line verbatim, and a name like `Akira's AirPods` would
break its quoting. Setup keeps it in the plugin's data directory instead.

Environment variables in the shell Claude Code starts from also work, below the plugin
settings: `RECH_LANGUAGE`, `RECH_LISTEN_PAUSE`, `RECH_LISTEN_DEVICE`, `RECH_LISTEN_SOUNDS=0`,
`RECH_BIN` (another `rech` binary).

## Notes

- On the first prompt of a session, recording starts only after other plugins' SessionStart
  hooks finish. Wait for the Tink.
- Claude Code inlines a skill command's stderr into the prompt, so `rech-listen --for-prompt`
  logs to `~/Library/Logs/rech-listen.log` instead. Look there when dictation says
  "nothing heard": `the input delivered only digital silence` means the wrong device, or no
  microphone access for the app running Claude Code.
- `bin/rech-listen` and `rech listen` also work on their own and in other agents: they print
  one phrase to stdout (exit 0), or exit 3 when nothing was heard.
