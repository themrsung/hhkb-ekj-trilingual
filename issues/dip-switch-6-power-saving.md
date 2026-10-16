# DIP switch 6: power saving

**Applies to:** HHKB Studio, mostly when connected over Bluetooth.
**Status:** workaround; the owner recommends it if you hit the symptom.

## Symptom

Over Bluetooth, the keyboard keeps going to sleep. You have to wake it again, and the first
keypress may be lost.

## Cause

The Studio's power saving is **very aggressive**, and **SW6 is inverted**:

| SW6 | Power saving |
| --- | --- |
| **OFF** (the repo default, "all OFF") | **On** |
| **ON** | **Off** |

## What to do

If the keyboard keeps sleeping over Bluetooth, **flip SW6 ON** to turn power saving off. That's the
repo author's recommendation. Expect shorter battery life.

Tested on the owner's board (2026-10-17), on Profile 4 over Bluetooth: flipping SW6 **OFF → ON**
kept the custom keymap, kept the active profile (still Profile 4, with the F keys working), and kept
the Bluetooth pairing in the same slot. No re-pairing and no keymap rewrite are needed. The repo
default stays **all OFF**. Change SW6 only if you have the symptom.

See also: `../docs/07-hhkb-studio.md` (DIP switches).
