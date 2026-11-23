# Shenmue DC — Fight Style & Motion Format Notes

Reverse-engineering notes on how the Dreamcast Shenmue engine stores and plays
fighting styles, and on the per-bone motion channel system those styles reference.

> **Notes only.** No game data, no ROMs, no patched images and no extracted
> assets are hosted here. Everything described assumes you own a legitimate
> copy of the game.

## Overview

In the Dreamcast build, each fighting style is a single file `EN_<NAME>.BIN`
stored inside a `PAKS` archive. The file is a short header plus five
independently addressed sections. Knowing what each section actually does is
the prerequisite for editing moves, swapping styles between characters, or
porting animations between games.

## Documents

| # | Document | Contents |
|---|---|---|
| 01 | [EN file format](docs/01-en-file-format.md) | Header layout, the five sections, what each one really is |
| 02 | [Motion channels](docs/02-motion-channels-69.md) | The 69-channel per-frame pose system and its bone mapping |
| 03 | [PKS repacking](docs/03-pks-repack.md) | `PAKS` container structure and the mistakes that break it |

## Summary of findings

- The style header is a five-entry offset table. Section 0 is the
  **reaction program** — a bytecode stream that carries the actual move IDs.
- Section 4 is **not** a move table. It holds hit / knockdown / guard / getup
  animation IDs indexed by state. Editing it as a move list does nothing.
- A move is encoded as opcode `5` followed by a 32-bit value; the low 12 bits
  of that value are the motion ID.
- The 69-channel motion format drives the skeleton through a mix of direct
  rotations, IK targets and inherited channels. Channels 45–56 are the right
  arm, 57–68 the left arm, 9–26 the legs, 27–44 the torso and head.
- `PAKS` archives keep a 20-byte-per-entry table of contents at the end of the
  file, and lookups are a binary search with an exact 8-byte name match.
  Changing entry count or ordering without rebuilding the TOC makes the engine
  silently fall back to the default style.

## Related work

Container, model and texture formats are already well covered elsewhere:

- `new-yokosuka/SHENMUE_DOCUMENTATION` — `PAKS`/`PAKF`, `MT5`/`HRCM`, `PVR` textures
- `brynnb/ghidra-dreamcast-shenmue` — Ghidra loader, cutscene `SEQ` research

Both focus on the container and model layer. The fight data layer is what this
repository adds.

## License

MIT — see [LICENSE](LICENSE).
