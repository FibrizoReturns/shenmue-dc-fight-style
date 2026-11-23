# 03 — Repacking a `PAKS` archive

## Layout

A `PAKS` archive is a 16-byte header, the concatenated payload blocks, a
16-byte trailing block, and then the table of contents, which runs to the end
of the file.

| Offset | Content |
| --- | --- |
| `0x00` | `PAKS` magic |
| `0x04` | `0x10` |
| `0x08` | 8 zero bytes |
| `0x10` | first payload block |
| … | further payload blocks, back to back |
| `data_end` | 16-byte trailing block |
| `data_end + 16` | table of contents |

Relevant header fields:

| Offset | Meaning |
| --- | --- |
| `0x14` | `data_end` — end of the payload area |
| `0x18` | entry count |
| `0x1C` | `data_end` again (duplicate field) |

The first payload block starts at exactly `0x10`, so its TOC offset is `16`.

## Table of contents

Each entry is 20 bytes:

| Offset | Size | Meaning |
| --- | --- | --- |
| `0` | 8 | name, **NUL-padded** |
| `8` | 4 | type |
| `12` | 4 | offset, **absolute** from the start of the file |
| `16` | 4 | length |

## Lookup

Two checks run before a file is served:

1. The reader validates the magic — the first `u32` must equal `PAKS`.
2. The name is resolved by a **binary search** over the table of contents,
   and the final comparison is a `memcmp` of exactly 8 bytes.

Consequences, all of them load-bearing:

- An entry name must be exactly 8 bytes with NUL padding. Pad with spaces and
  the comparison fails.
- Entries must be sorted strictly ascending by name, or the binary search
  converges on the wrong slot.
- Appending a new entry at the end without re-sorting breaks every later lookup.

## Failure modes, each observed in practice

| Symptom | Cause |
| --- | --- |
| Fighter uses the *default* style and the camera jumps to a distant angle | The intended `EN_*` entry was never found: bad name padding, broken magic, or a TOC sitting at the wrong offset |
| The clip plays once, then both fighters freeze | A newly appended motion's relative offset was not rebased against the block's `data` base |
| Swapping the style byte has no effect at all | Only one of the style bytes was changed; every reference to the style must be rewritten |
| Nothing changes at all | The move list was edited in the wrong section |

## Safe edit checklist

1. Preserve the `PAKS` magic and the `0x10` at `0x04`.
2. Keep the TOC sorted strictly ascending by name.
3. Pad every entry name to 8 bytes with NUL, never with spaces.
4. Recompute `data_end` and write it to both `0x14` and `0x1C`.
5. Keep the 16-byte trailing block before the TOC.
6. If any block changes length, recompute **every** absolute offset that
   follows it — the TOC stores absolute offsets, not relative ones.
7. Update the entry count at `0x18`.

Skipping step 6 is the single most common way a repack produces a working file
that the engine refuses to use.

## See also

- [01 — EN file format](01-en-file-format.md)
- [02 — Motion channels](02-motion-channels-69.md)
