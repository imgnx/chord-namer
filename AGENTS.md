# Agent Notes

## Context

- Single-page React app living in `index.html`; now supports chord analysis and riff capture without navigation.
- Bulk editor lives in the right-hand sidebar (`flex-wrap`) so the fretboard retains primary focus.
- Integrated tuner (`./tuner` logic) mounts inline with microphone controls and can push captured notes into the riff log.
- Tuner UI now includes an input-source selector backed by `enumerateDevices` so players can pick a specific mic.
- Oscilloscope and signal meter sit inside the tuner to visualize analyser data for sanity checks.

## Mode Expectations

- `Riff` mode is the default on load—verify the fretboard starts muted and the riff log is ready to capture immediately.
- `Chord` mode must continue to enforce one note per string and drive chord-name suggestions.
- `Riff` mode should clear the fretboard after every pluck while appending the detected pitch to the log; logs persist until the user clears them.
- Mode toggles should be instantaneous—avoid full reloads or focus trapping in the sidebar.

## Tuner Workflow

- Relies on the Web Audio API; browsers need `navigator.mediaDevices.getUserMedia` permission for microphone input.
- Capture button should insert a single detected pitch into the riff log; ensure duplicate presses do not spam when a note is still sounding.
- Provide error messaging when the device denies access or lacks audio support.
- Input-source selector must refresh after permissions change and handle Safari's delayed device-label availability.
- Oscilloscope and signal meter should react in near-real-time to confirm audio input; keep them in sync with the analyser buffer.

## Testing Guidance

- Exercise both modes manually after UI changes—verify chord suggestions remain accurate and riff logging resets the board.
- Test tuner start/stop/capture flows on at least one browser with microphone access; there is no automated coverage yet.
- When adding tooling, keep riff capture and tuner integration hooks intact so future automation can observe the same dispatcher events.
