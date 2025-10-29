# Changelog

## Unreleased

- Added a chord/riff mode toggle to `index.html`; riff mode now logs each picked note and clears the fretboard for the next hit while chord mode retains the classic multi-string voicings.
- Reworked the layout so the 6-input bulk editor sits in a right-hand column with wrapping controls, keeping the fretboard front and center.
- Integrated the standalone tuner into the main page with microphone start/stop controls and a capture button that sends the detected pitch into the riff log.
- Documented the new workflow and tuner requirements in `README.md` and `AGENTS.md`.
- Added a tuner input-source selector so players can choose microphones explicitly, including Safari-friendly guidance.
- Default session now opens in riff mode to streamline tuner captures; added oscilloscope and signal meter visuals to confirm audio input.
