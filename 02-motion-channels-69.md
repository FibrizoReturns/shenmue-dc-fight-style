# 02 — The 69-channel motion format

## Container

A motion is a fixed-size header followed by a table of channels and their
keyframes.

Header, four little-endian `u16`:

| Offset | Meaning |
| --- | --- |
| `+0` | version |
| `+2` | flags |
| `+4` | frame count |
| `+6` | absolute offset of the keyframe data |

In every file inspected the keyframe offset equals `8 + channel count + 1`,
i.e. an 8-byte header, a one-byte channel count, then one byte per channel,
and the keyframes begin right after the table.

Frame count is the length of the clip. Observed clips range from about 30
frames (short jabs) to about 195 frames (long throws).

## Channel layout

The full pose pipeline consumes **69 channels**. Channels are sparse: a motion
may declare far fewer than 69 and inherit the rest from the stance, which
explains why trimming a channel list can *look* fine while silently freezing a
limb.

| Channels | Drives |
| --- | --- |
| 0–2 | root translation |
| 3–5 | pelvis rotation |
| 9–11 | left hip rotation |
| 12–14 | left foot IK target |
| 15–17 | left ankle |
| 18–20 | right hip rotation |
| 21–23 | right foot IK target |
| 24–26 | right ankle |
| 27–29 | chest |
| 30–32 | chest align |
| 33–35 | upper chest |
| 36–38 | head base |
| 39–41 | head middle |
| 42–44 | head align |
| 45–56 | **right arm** |
| 57–68 | **left arm** |

## Arm channel breakdown

Both arms use the same twelve-channel block:

| Block offset | Meaning |
| --- | --- |
| `+0 … +2` | shoulder rotation (Z, Y, X order) |
| `+3 … +5` | elbow rotation |
| `+6 … +8` | **IK target for the hand** |
| `+9` | wrist rotation X |
| `+10` | hand rotation Z |
| `+11` | hand rotation Y |

The IK channels are not world-space positions. They are solved locally: the
engine aligns the target into the bone's local frame and derives the elbow
angle from it, so an IK channel drives both the upper arm direction and the
elbow bend at once.

Filling those three channels with the wrong bone's data — or with the left
limb's data on the right side — produces the classic "arms raised and twisted
outward, waist bent backward" failure.

## Values

Channel values are **offsets from the stance**, not absolute angles. An entry
of `0.000` means "leave this bone alone". Most working channels sit in the
±0.3 range; larger values come from exaggerated or spinning motions.

Most motions end with a key at `0.000` on every channel, returning the pose to
the stance. That final key is what makes a clip blend back into idle cleanly.

## Sanity checks

Before shipping a hand-built clip, verify:

1. Version and flags are consistent across the whole set.
2. Frame count matches the intended clip length, not the source clip's.
3. The keyframe offset equals `8 + channel count + 1`.
4. Every channel index you write is one you actually mapped to a real bone.

Skipping step 4 is the most common cause of a clip that plays to completion
while looking broken.

## See also

- [01 — EN file format](01-en-file-format.md)
- [03 — PKS repacking](03-pks-repack.md)
