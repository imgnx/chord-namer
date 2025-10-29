# Chord Naming Rules & Conventions

## Purpose

Define a machine-friendly and human-readable system for naming guitar chords. The rules establish canonical forms, parsing grammar, normalization steps, and examples so that parsers, display UIs, and musicians agree on chord identity.

## Design Goals

- **Unambiguous**: every valid name parses deterministically.
- **Compact**: prefer terse musical symbols over prose.
- **Parseable**: grammar is regex/BNF friendly and token order is fixed.
- **Canonical**: normalize synonymous spellings to a single stored form.
- **Intent-preserving**: respect user-specified accidentals when rendering.
- **Extensible**: support jazz/pop alterations, polychords, and slash basses.

## High-Level Conventions

- **Root**: uppercase `A`-`G` with optional `#` or `b` (e.g., `C`, `F#`, `Bb`).
- **Major triad**: bare root (`C` -> C major).
- **Minor**: `m` suffix; canonical form collapses `min` -> `m`.
- **Major 7**: use `maj7` (never `M7`).
- **Dominant 7**: bare `7` (e.g., `C7`).
- **Minor 7**: `m7`.
- **Extensions**: append `9`, `11`, `13` after quality (`C7#9`, `Cm11`).
- **Add tones**: `add9`, `add11`, etc. for non-tertian additions.
- **Suspensions**: `sus2` or `sus4`; `sus` alone implies `sus4`.
- **Alterations**: prefix alteration with accidental (`C7b9#11`).
- **Slash chords**: `Root/Bass` (single bass note).
- **Polychords**: `ChordA|ChordB`.

## Informal Grammar

```
Chord      := ChordBlock ('|' ChordBlock)* ('/' BassNote)?
ChordBlock := Root Quality? ExtensionList? AddList? AlterList? Sus? Voicing?
Root       := Note
Note       := [A-G] ('#' | 'b')?
Quality    := 'm' | 'min' | 'maj' | 'dim' | 'aug' | 'sus' | '°' | '+'
ExtensionList := ('6' | '7' | '9' | '11' | '13')(ExtensionListTail)*
ExtensionListTail := ('9' | '11' | '13' | ('#'|'b')?[0-9]+)
AddList    := ('add'[0-9]+)*
AlterList  := (('#'|'b')[0-9]+)*
Sus        := 'sus2' | 'sus4' | 'sus'
Voicing    := '(' voicing-string ')'
BassNote   := Note
```

The grammar is intentionally permissive. Semantic validation should enforce musical logic (for example, allowing `add9` without a third is musically odd but not prohibited).

Token order is canonical: `Root → Quality → Extensions → Additions → Alterations → Suspension → Voicing`.

## Canonical Formatting Rules

1. **Root**: uppercase with sharps for storage (`Db` → `C#` when `preferSharps` is on).
2. **Quality**: blank for major triads; normalize synonyms (`min`, `-` → `m`, `°` → `dim`, `+` → `aug`).
3. **Extensions**: emit ascending numbers (`7`, `9`, `11`, `13`). Use `maj7` explicitly for major sevenths.
4. **Alterations**: order ascending by scale degree (`b9` before `#11`).
5. **Additions**: append `add9`, `add11`, etc., after alterations.
6. **Suspensions**: `sus2` or `sus4` replace the third; when combined with adds use `Dsus4add9`.
7. **Slash bass**: append `/Bass`.
8. **Polychords**: join chord blocks with `|` without spaces.
9. **Whitespace**: canonical names contain no spaces (`C maj 7` → `Cmaj7`).

## Regex Tokenizer (JavaScript-Oriented)

```
^\s*([A-G])([#b]?)(?:(maj|m|min|dim|aug|sus|°|\+))?((?:6|7|9|11|13|maj7)?(?:[#b]?\d+)*)?(?:((?:add\d+)*))?(?:((?:[#b]\d+)*))?(?:/(?:([A-G])([#b]?)))?\s*$
```

Capture groups:

1. Root letter.
2. Root accidental (optional).
3. Quality token.
4. Extensions/number blob.
5. Additions blob.
6. Alterations blob.
7. Bass letter (if slash present).
8. Bass accidental.

Post-processing steps:

- Split number blobs into individual tokens (interpret `maj7`, `#9`, etc.).
- Validate recognized degrees (`6`, `7`, `9`, `11`, `13`) and accidentals.
- Normalize synonyms (`min` → `m`, `M7` → `maj7`, etc.).

## Canonicalization Workflow

1. Trim and normalize whitespace/Unicode.
2. Tokenize with the regex.
3. Map synonyms to canonical tokens.
4. Optionally convert flats to sharps for storage (`preferSharps` default `true`).
5. Sort extensions and alterations numerically.
6. Remove redundant tokens (e.g., `Cmaj` → `C`).
7. Recompose canonical string in the required order.

```js
function canonicalize(chordStr, opts = { preferSharps: true }) {
  // 1. clean input
  // 2. regex match and extract
  // 3. normalize synonyms
  // 4. convert accidentals if preferSharps
  // 5. sort extensions and alterations
  // 6. join tokens → canonical string
}
```

## Interval Reference (Semitone Distance From Root)

| Token        | Semitones |
| ------------ | --------- |
| Root         | 0         |
| b2 / #1      | 1         |
| 2 / 9        | 2         |
| b3           | 3         |
| 3            | 4         |
| 4 / 11       | 5         |
| #4 / b5      | 6         |
| 5            | 7         |
| #5 / b6      | 8         |
| 6 / 13       | 9         |
| b7           | 10        |
| maj7         | 11        |
| 9            | 14        |
| 11           | 17        |
| 13           | 21        |

Common sets:

- Major triad: `0, 4, 7`
- Minor triad: `0, 3, 7`
- Diminished: `0, 3, 6`
- Augmented: `0, 4, 8`
- Dominant 7: `0, 4, 7, 10`
- Major 7: `0, 4, 7, 11`
- Minor 7: `0, 3, 7, 10`
- Half-diminished (`m7b5`): `0, 3, 6, 10`
- Dominant 9: `0, 4, 7, 10, 14`

## Examples

- `C` → major triad (`0,4,7`)
- `Cm` / `Cmin` → minor triad (`0,3,7`)
- `C7` → dominant seventh (`0,4,7,10`)
- `Cmaj7` → major seventh (`0,4,7,11`)
- `Cm7` → minor seventh (`0,3,7,10`)
- `C7#9` → dominant with sharp nine
- `F#maj7#11` → major seventh with raised eleventh
- `Bbmin7b5` → half-diminished
- `G/B` → slash chord (first inversion)
- `Cmaj7|D7` → polychord

## Display vs. Storage

- **Storage**: canonicalize to sharps by default for equality checks and dictionaries.
- **Display**: offer sharps or flats (`♯`/`♭` glyphs if supported). Flats are common in jazz contexts.
- **Input**: be permissive; output canonical spelling plus optional localized display.

## Implementation Notes

- Keep parsing, validation, canonicalization, and rendering as separate modules.
- Provide a `chordCodexVersion` to support future schema updates.
- Avoid double accidentals; resolve enharmonically instead (`E##` → `F#`).
- Support parentheses in user input (`C(add9)`, `C(add9)/E`) but drop them in canonical output (`Cadd9/E`).
- Interpret `C6` as `0,4,7,9` (no 7th); `C13` implies a dominant seventh unless qualified (`Cmaj13`, `Cm13`).
- For `sus` chords with extensions, append additions (`Csus4add9`).
- Distinguish `add9` (no 7th) from `9` (implies dominant 7 unless otherwise specified).
- Disallow multi-bass slash chains (`C/E/G`); prefer polychords (`C/E|G/B`) if multiple bass notes are required.

## Accidentals Preference Toggle

- Default to sharps.
- Provide a display option that re-spells notes as flats when requested.
- The storage layer remains in sharps for canonical equality checks.

## Versioning & Extensibility

- Tag serialized chord data with `chordCodexVersion`.
- Build the parser to accept plug-ins for new alterations, voicings, or polychord semantics.

