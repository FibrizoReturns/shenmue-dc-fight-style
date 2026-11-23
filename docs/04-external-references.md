# External reference index

Public research worth reading alongside these notes. Everything here is a
**link only** — no text is mirrored and no game data is hosted.

## Fighting style and motion

| Source | What it covers | Why it matters here | Caveat |
|---|---|---|---|
| [Shenmue-Mods/Knowledgebase](https://github.com/Shenmue-Mods/Knowledgebase) | Community reverse-engineering repo, ~58 documents across Shenmue I, II and Dreamcast formats | The single best index of the surrounding layers | A collection, not one unified document |
| [Changing Fighting Style (Shenmue I)](https://github.com/Shenmue-Mods/Knowledgebase/blob/master/Shenmue_I/Changing_Fighting_Style.md) | The engine resolves a style through the `EN_%s.BIN` name template; lists all 58 style IDs | Independent confirmation that the style is picked by a 4-character ID and that a missing file silently falls back | Author notes it only works in Free Battle — Practice freezes |
| [MOTION.BIN file structure](https://github.com/Shenmue-Mods/Knowledgebase/blob/master/Whats_Shenmue/MOTION_BIN_File_Structure.md) | File-level tables: 20-byte header, data look-up table, name look-up table, 40-byte entry prefix | The other half of [02](02-motion-channels-69.md): that note explains what the channels mean, this one explains how the file indexes the animations | Names such as `AB_031` are per-file, not global IDs |

## Character and slot mapping

| Source | What it covers | Why it matters here |
|---|---|---|
| [Shenmue II character database](https://github.com/Shenmue-Mods/Knowledgebase/blob/master/Shenmue_II/Character_Database.md) | Character IDs for Shenmue II | Needed when binding a style to a specific playable slot |
| [Character database per filenames](https://github.com/Shenmue-Mods/Knowledgebase/blob/master/Shenmue_II/Character_Database_per_Filenames.md) | ID ↔ filename mapping | Verifies which file a slot actually loads |
| [Character swapping tutorial (Shenmue I)](https://github.com/Shenmue-Mods/Knowledgebase/blob/master/Shenmue_I/Character_Swapping_Tutorial.md) | Whole-character swap: unpack `HUMANS.AFS` + `HUMANS.IDX`, rewrite the 4-letter ID inside `PKF`/`PKS`, repack | Useful only as a lesson in ID consistency — see the warning below |

### Do not confuse the two layers

That swapping tutorial works at the **container and identity layer**: it
exchanges two whole characters, dragging their models, textures, AI, dialogue
and quest triggers along with them. The notes in this repository work at the
**behaviour layer**: the style file and the motion it points at.

Mixing the two produces exactly the failure this repository documents —
a character that stands idle because the style block it expects is no
longer reachable. If a whole-character swap is what you actually want, follow
that tutorial on a Shenmue I build; do not transplant it into an
`EN_*.BIN` style edit.

## Textures, text and area data

| Source | What it covers |
|---|---|
| [PVR graphic format](https://github.com/Shenmue-Mods/Knowledgebase/blob/master/Dreamcast_Information/PVR_Graphic_Format.md) | Dreamcast texture container |
| [SPSD files](https://github.com/Shenmue-Mods/Knowledgebase/blob/master/Dreamcast_Information/SPSD_files.md) | `SPSD` sprite sheets |
| [Notebook editing (Shenmue II)](https://github.com/Shenmue-Mods/Knowledgebase/blob/master/Shenmue_II/Notebook_Editing.md) | Notebook page text |
| [Notebook hacking (Shenmue I)](https://github.com/Shenmue-Mods/Knowledgebase/blob/master/Shenmue_I/Notebook_Hacking.md) | Notebook page text, earlier build |
| [1ST_READ text quirks](https://github.com/Shenmue-Mods/Knowledgebase/blob/master/Whats_Shenmue/1ST_READ_Text_quirks.md) | Text encoding and pointer oddities |
| [1ST_READ text pointer mods](https://github.com/Shenmue-Mods/Knowledgebase/blob/master/Shenmue_I/1ST_READ_Text_Pointer_mods.md) | Relocating text without breaking pointers |
| [Analysis of a simple MAPINFO](https://github.com/Shenmue-Mods/Knowledgebase/blob/master/Shenmue_I/Analysis_of_a_simple_MAPINFO.md) | `MAPINFO.BIN` layout |

## Layer map

1. **Container** — `PAKS` / `AFS` archives and their tables of contents.
   See [03](03-pks-repack.md).
2. **Model and texture** — `PKF`, `HRCM`, `MT5`, `PVR`, `SPSD`.
3. **Behaviour** — `EN_*.BIN` sections and the motion channels they reference.
   See [01](01-en-file-format.md) and [02](02-motion-channels-69.md).

Most community documents stop at layer 1 or 2. These notes exist to cover
layer 3.
