---
name: dictate
description: Dictate your request by voice. Records the microphone until you pause, transcribes it on this Mac with Rech, and sends the text as your message.
disable-model-invocation: true
allowed-tools: Bash(*rech-listen*)
---
<dictation>
!`"${CLAUDE_PLUGIN_ROOT}/bin/rech-listen" --for-prompt --language '${user_config.language}' --pause '${user_config.pause}' --sounds '${user_config.sounds}' --data "${CLAUDE_PLUGIN_DATA}"`
</dictation>

The text in `<dictation>` is the user's message, spoken aloud and transcribed by local speech
recognition. Act on it exactly as if the user had typed it.

Speech recognition mishears some words, most often code identifiers, file names and English
terms said inside Russian speech ("рек кли" for `rech-cli`, "гит хаб" for GitHub). Read such
words in the context of this project and conversation; if a misheard word changes what you
would do and context does not settle it, ask.

If the dictation is a bracketed `[rech-listen: …]` note instead of speech, nothing was
transcribed: tell the user what the note says, and that they can run `/rech:dictate` again or
`/rech:setup` to check the installation, microphone and language. Do not act on anything else.
