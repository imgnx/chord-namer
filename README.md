# Chord Namer (Codex Prototype)

## Overview

This single-page React prototype converts six-string guitar voicings into chord names or captures single-note riffs. It analyses the notes produced by a selected fret pattern, enumerates possible roots, and surfaces theory-driven labels to help with transcription, teaching, and composition. In riff mode each plucked string is logged so you can sketch melodic ideas without leaving the fretboard.

## Quick Start

1. Open `index.html` in a modern desktop browser.
2. The toolbar defaults to `Riff` mode so tuner captures work immediately; switch between `Riff` and `Chord` as needed.
   - `Riff` clears the fretboard after each pluck and appends the detected pitch to the riff log.
   - `Chord` keeps the one-note-per-string voicing workflow.
3. Select frets on the virtual neck, paste patterns into the right-hand bulk editor, or use the tuner capture button to insert the live pitch.
4. Review the suggested chord names or riff log and explore note breakdowns, inversions, and exports.
5. Use the transpose control to audition alternate tunings in semitone increments.

No build step is required—the page loads React and Babel from public CDNs.

## Key Features

- Interactive virtual fretboard with mute/open controls for every string.
- Mode toggle that switches between chord voicings and single-note riff capture without reloading.
- Bulk entry grid for pasting or typing multiple voicings, with per-row validation.
- Responsive right-hand column for the bulk editor that uses `flex-wrap` to keep dropdowns, buttons, and inputs readable on narrow widths.
- Rule-based chord name suggestions driven by interval analysis.
- Export helpers for copying or downloading the chosen chord names.
- Integrated guitar tuner (Web Audio API) with microphone start/stop, input-source selector, oscilloscope + signal meter, and a capture button that pushes the detected pitch into the riff log.
- Note breakdown panel that shows each pitch and string mapping.

## Working Modes

- **Riff mode** (default): Every plucked note is appended to the riff log, the fretboard resets immediately, and the log persists until cleared manually.
- **Chord mode**: Each string tracks a single note, ideal for voicings. Changing frets updates the chord analysis instantly.

Switching modes does not reload the page—use it freely while experimenting.

## Tuner Notes

- Grant microphone access, then use the inline input-source selector to switch between built-in and external mics without leaving the page.
- Some browsers (notably Safari) only reveal device names after the microphone has been started; refresh the page if you change permissions mid-session.
- The oscilloscope and level meter provide instant visual feedback that audio is flowing—use them to confirm levels before capturing notes.

## Architecture Notes

- Front-end only, implemented as a React app embedded in `index.html`.
- Uses inline Babel transpilation for JSX; suitable for quick prototyping.
- Riff logging and tuner capture share the app's central dispatcher so exporters and bulk edits see consistent events.
- Musical logic lives in pure helper functions (`analyzePositions`, `computeChordSuggestions`, etc.).
- Default tuning is standard EADGBE; transpose applies a uniform semitone shift across strings.
- Pitch detection relies on the Web Audio API; if the browser denies microphone access the tuner card surfaces an actionable error.

## Naming Rules Reference

The long-form chord naming policy—covering grammar, canonicalization, intervals, and display preferences—is documented separately in `docs/chord-naming-rules.md`. Highlights:

- Canonical order: `Root → Quality → Extensions → Alterations → Adds → Suspension → /Bass → |Polychord`.
- Storage prefers sharp spellings; display should allow a sharps/flats toggle with sharps as the default.
- Extensions (`7`, `9`, `11`, `13`) and alterations (`#11`, `b13`, etc.) must be numerically sorted.

## Repository Layout

```
index.html              # Application entry point (React + UI)
docs/chord-naming-rules.md  # Detailed chord naming specification
README.md               # This guide
.gitignore
```

Legacy scaffolding (`blueprint.html`, `old/`, etc.) has been removed to keep the footprint lean.

## Development Tips

- Run the page via a static file server (`npx serve .`) for clipboard/download helpers to work reliably.
- Keep business logic modular; separate parsing, validation, and rendering to ease future refactors.
- When expanding the parser, update the rules document and add usage examples to the bulk entry grid.
- Test riff capture and tuner flows manually in a browser with microphone access; automated verification is not yet wired up.
- Spot-check the tuner input selector with alternate devices where possible—Safari can require a refresh after changing microphone permissions—and ensure the oscilloscope/level meter respond to incoming audio.

## Known Gaps / Next Steps

1. Implement the sharps ↔ flats display toggle described in the rules document (current UI always shows sharps).
2. Upgrade the chord canonicalizer so that output names follow the documented canonical order (e.g., dominant chords should surface as `C7` instead of `C add b7`).
3. Add automated tests (unit or property-based) for chord parsing and naming before shipping beyond prototype status.
