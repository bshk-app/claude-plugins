---
name: setup
description: Set up voice dictation with Rech — install or update rech, choose the dictation language and microphone, download the speech model, and check microphone access. Use when the user runs /rech:setup, asks to set up, fix or configure dictation or Rech, or /rech:dictate reported a problem.
allowed-tools: Bash(rech *), Bash(command -v rech), Bash(brew install bshk-app/tap/rech), Bash(brew upgrade rech), Bash(mkdir -p *), Write
---
# Setting up Rech dictation

`/rech:dictate` records one phrase (Tink → speak → pause → Pop) and transcribes it on this Mac
with `rech`. Setup makes that work and fast: with a known language the model loads while the
user speaks and the text is ready under a second after they stop; with `auto` it takes
several seconds longer and phrases under 3 s go to Parakeet instead of the right model.

## Where things stand

- rech: !`command -v rech >/dev/null 2>&1 && { rech --version; rech --help 2>&1 | grep -q '^  listen ' && echo "has listen" || echo "too old: no listen"; } || echo "not installed"`
- Language from /config → Rech: `${user_config.language}` (still reading `${…}` means never saved)
- Language saved by setup: !`cat "${CLAUDE_PLUGIN_DATA}/language" 2>/dev/null || echo "none"`
- Microphone saved by setup: !`cat "${CLAUDE_PLUGIN_DATA}/microphone" 2>/dev/null || echo "none (the system input)"`
- Inputs (name, UID, default): !`rech listen --list-devices 2>/dev/null || echo "unavailable"`

Data directory for the saved choices: `${CLAUDE_PLUGIN_DATA}`

## Steps

1. **Install.** If rech is not installed, or too old to have `listen`, ask the user, then run
   `brew install bshk-app/tap/rech` or `brew upgrade rech`. Stop if that fails.

2. **Language.** A language saved in /config wins; otherwise the one saved by setup is used.
   If neither is set, or the user wants to change it, ask which language they dictate in
   (Russian → `ru`, English → `en`, …; offer `auto` only for someone who switches languages).
   Save the code with the Write tool to `${CLAUDE_PLUGIN_DATA}/language`, one line, e.g. `ru`.

3. **Microphone.** The system input is used unless one is saved. Show the inputs above and ask
   whether to use a specific one (an external USB microphone, say), or keep following the system
   input. To pin one, write its exact name, one line, to `${CLAUDE_PLUGIN_DATA}/microphone`; to
   follow the system input, write an empty file. Skip virtual devices (BlackHole, Teams Audio,
   Background Music, Tish Audio) unless the user picks one.

4. **Model and access.** Run, with a 10-minute timeout because the first run downloads the model:

   ```sh
   rech listen --prepare --language <code> [--device "<microphone>"]
   ```

   It prints the microphone, whether this app may use it, and whether the model loaded.
   - First run: macOS may ask for microphone access for the app running Claude Code
     (Terminal, iTerm, Ghostty, VS Code, …). Tell the user to allow it.
   - "Microphone access is off": the user turns it on in System Settings → Privacy & Security →
     Microphone for that app, then restarts the app.

5. **Report** what is set (language, microphone, model) in a few lines, mention that `/config`
   → Rech holds the pause length and start/stop sounds, and suggest trying `/rech:dictate`.
