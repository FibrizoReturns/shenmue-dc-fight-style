# 01 — The `EN_*.BIN` fight style file

## Where the file lives

A fighting style is a single file named `EN_<STYLE>.BIN`.

- **Shenmue I** — loose files on the disc, e.g. `FIGHT/EN_RYOU.BIN`,
  `SCENE/02/GMCT/EN_CHAI.BIN`, `SCENE/03/MEND/EN_CHAI2.BIN`.
- **Shenmue II (Dreamcast)** — packed inside the `PAKS` archive of the stage
  that uses them, e.g. `SCENE/04/AB00/QKBT.PKS`.

The specimen used throughout this page is Shenmue I's `EN_CHAI.BIN`
(2,292 bytes). The same layout was confirmed on Shenmue II styles.

## Header

The file opens with five little-endian `u32` values.

| Offset | Meaning |
| --- | --- |
| `0x00` | style / library id |
| `0x04` | absolute offset → **section 0** |
| `0x08` | absolute offset → **section 1** |
| `0x0C` | absolute offset → **section 2** |
| `0x10` | absolute offset → **section 3** |

`EN_CHAI.BIN` decodes to ids and offsets `6 / 20 / 1212 / 1796 / 2258`.
Section 0 always starts immediately after the header, at `0x14`.

Section lengths follow directly from the successive offsets:

| Section | Range | Size |
| --- | --- | --- |
| 0 | 20 – 1212 | 1,192 |
| 1 | 1212 – 1796 | 584 |
| 2 | 1796 – 2258 | 462 |
| 3 | 2258 – 2292 | 34 |

The last section ends exactly at the end of the file.

## Section 0 — reaction program

A bytecode stream interpreted by the fight reaction handler. **This is where
the moves of a style actually live.**

An instruction with opcode `5` commits an action. The 32-bit value carried by
that instruction encodes the action; the **low 12 bits** are the motion ID:

```text
action = (program[4] << 24) | (program[5] << 16) | (program[6] << 8) | program[7]
motion = action & 0xFFF
```

`EN_CHAI.BIN` contains three combat entries in this stream that resolve to real
battle motions (`CHA_CHAIBATTLE_RIMON`, `_RIMON_2`, `_RIMON_SPIN`), plus a
number of idle / locomotion / getup entries.

Editing section 0 changes which motion a style plays. Editing section 3 or 4
does not.

## Section 1 — camera script

Fixed 20-byte records. The engine indexes them as `section1 + 20 * choice`
where `choice` is derived from the current action. When a style fails to load
and the engine falls back to a default, the camera records come from the
default style too — which is why a broken archive shows the *default* style
while the camera snaps to an unrelated position.

## Section 2 — parameters, ranks and AI

A parameter block, not motion data. Observed structure:

| Offset | Meaning |
| --- | --- |
| `+25` | bit flags used by the stage logic |
| `+26` | 9 bytes per rank, indexed by the fighter's rank |
| `+111` | start of the AI behaviour byte stream |

At `+111` the stage code walks a byte sequence and uses each byte as an index
into the AI behaviour function table. Only part of the grouping (marker bytes,
terminator) has been mapped so far; the byte stream itself is confirmed.

## Section 3 — contact geometry

A fixed 34-byte block. `EN_CHAI.BIN`:

```text
00 00 07 8c  00 01 cc cd  00 01 cc cd  00 01 cc cd
00 01 66 66  00 03 00 00  00 2d 00 00  00 01 00 00
00 00
```

The leading floats are the authored fixed values: 1.8 forward, 1.8 backward,
1.4 sideways, 45.0 knockback.

## Section 4 — the 163-slot state table

`motionParameters[4]` is a fixed table of **163 entries × 2 bytes (326 bytes)**
mapping a fighter *state* to a motion ID. It is read only through byte offsets
that correspond to hit reactions, knockdowns, getups and guard animations —
never through a move index.

> **This is not a move list.** Writing move IDs into it produces no visible
> change, and overwriting a wide range of entries breaks the state machine
> while leaving the fighter visually unchanged.

## How the engine resolves a style

Load order, from the stage code:

1. Build the file name as `EN_%s` from the style-name table.
2. Ask the file loader for that name.
3. On failure, fall back to the default style (`styles[0]`).

A lookup only succeeds if the archive's table of contents is intact and the
entry name matches **exactly, as 8 bytes**. A near-miss is not an error: the
engine silently serves the default style. See
[03 — PKS repacking](03-pks-repack.md).

## See also

- [02 — Motion channels](02-motion-channels-69.md)
- [03 — PKS repacking](03-pks-repack.md)
