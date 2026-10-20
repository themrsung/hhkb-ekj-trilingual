# 07 — HHKB Studio (JIS): keymap and settings

The owner's HHKB Studio JIS, as set in PFU's keymap tool for the Studio (**Profile1**). Use this
as the reference when setting up a Studio. `01-hardware.md` covers the parts both models share
(the bottom-row codes and the DIP-before-keymap order). This file covers what's specific to the
Studio.

## Fast path: load the default settings file

[`hhkb-studio-setting-all.hks`](hhkb-studio-setting-all.hks) is the owner's full export from the
HHKB Studio Keymap Tool. It's the quickest way to put all of this page's settings on a Studio, for
macOS and Windows alike. It holds the keymap for **every profile**:

| Profile | For |
| --- | --- |
| **Profile1** | macOS (this page) |
| Profile2 | PFU's default Mac mode, unchanged |
| **Profile3** | Windows without AutoHotkey (`windows/01-hardware.md`) |
| **Profile 4** | Windows with AutoHotkey (`windows/01-hardware.md`) |

It also holds the pointing-stick and gesture-pad settings in the table below. In the keymap tool,
load the file and write it to the keyboard. **Importing replaces all profiles**, so first export
your own settings if you keep a custom keymap on any profile. Then pick the profile for your OS and
check the layers against the screenshots. The DIP switches are physical, so set them by hand
first (all OFF).

Checked on the owner's board: with these settings, the Standard bottom row emits
**`f18`**, `left_option`, `left_command`, `japanese_eisuu`, `spacebar`,
`japanese_kana`, `right_command`, `right_option`, then the arrows. Fn1+P emits `pause` (macOS sees
F15), and Fn1+I emits `print_screen` (macOS sees F13).

## Settings outside the keymap

| Setting | Value |
| --- | --- |
| DIP switches | **All OFF** (SW4 ON would reverse the scroll direction; SW6 ON turns the aggressive power saving **off**, see `../issues/dip-switch-6-power-saving.md`) |
| Pointing stick → Button | **On** (LB / Fn2 / RB keys below the space bar) |
| Gesture Pad – Left Side | **Off** |
| Gesture Pad – Front Left | **Off** |
| Gesture Pad – Front Right | On: **MW←** / **MW→** (horizontal mouse wheel) |
| Gesture Pad – Right Side | On: MW↓ / MW↑ (vertical mouse wheel) |

As with the HYBRID Type-S, **set the DIP switches before writing the keymap** (`01-hardware.md`).

**Finding: flipping SW4 doesn't reset the Studio's keymap.** On the owner's board, with a custom
keymap written by the HHKB Studio Keymap Tool in place, SW4 was flipped **ON → OFF**, then
**OFF → ON → OFF**. After every flip the custom keybinds were all still there, and SW4 still
switched the scroll direction. So changing the scroll direction with SW4 is safe on the Studio.

**Finding: flipping SW6 (power saving) doesn't reset it either.** On 2026-10-17, SW6 was flipped
**OFF → ON** on the owner's board, which was on Profile 4 over Bluetooth. Afterwards the keyboard
was still on Profile 4, the F keys still worked, and the Bluetooth pairing stayed in the same slot.
SW6 is inverted: **OFF = power saving on**. See `../issues/dip-switch-6-power-saving.md`.

Only SW4 and SW6 were tested. For the other switches, set DIP switches first and recheck the
keymap afterwards.
This differs from the HYBRID Type-S, where changing DIP switches overwrites the keymap.

## The F18 key (the Studio's "HHKB key")

The key in the **半/全** position (left of Opt, bottom row) is set to **F18** in the HHKB Studio
Keymap Tool, on **every layer** (Standard, Fn1, Fn2, Fn3). The Studio sends F18 straight from its
firmware, and Hammerspoon turns it into JIS Korean. Karabiner doesn't remap this key on the
Studio.

- Checked on the owner's board (USB): F18 pressed 3× → `f18` 3× reached Hammerspoon with no
  Karabiner remap, and the input source switched to Korean 2-Set.
- Set it on all four layers, so the key still switches to Korean while Fn1 or Fn2 is held, and
  so no layer falls back to `` ` ``.
- The **HYBRID Type-S can't do this**: its HHKB Keymap Tool can't assign F18, so it still needs
  the Karabiner `` ` `` → F18 remap (`03-karabiner.md`, rule A).
- Upgrading a Studio that was set up with the old `` ` `` remap: `08-migrate-studio-f18.md`.

## Karabiner and the Studio

Karabiner's only Studio-specific config is the device entries for vendor `1278` / product `21`
(`03-karabiner.md`): one `is_keyboard` entry for USB, and one combined entry for Bluetooth (below).
They hold only Fn1+P → Play/Pause. The F18 key needs no Karabiner entry.
Hammerspoon and the macOS settings don't care how the Studio is connected.
Over USB, leave the Studio's pointing-device and consumer entries alone. The gesture pads and
pointing stick keep working through Karabiner without any rules.

## Bluetooth pairing

From PFU's [HHKB Studio manual](https://happyhackingkb.com/manual/studio/ug-us/en/ug/topic/connect_bluetooth.html).
**The key order matters:**

1. Power switch on (slides right, turns green). Unplug the USB cable.
2. **Fn+Q** → pairing standby: the LEDs sweep side to side.
3. **Fn+Ctrl+n** (n = slot 1–4) → the n-th LED blinks quickly in blue. This starts pairing on
   slot n and overwrites whatever was paired there before.
4. On the Mac: System Settings → Bluetooth → Nearby Devices → **HHKB-Studio\<n\>** → Connect.
   Type the number macOS shows on the Studio, then press Enter.
5. **Fn+X** leaves pairing standby or pairing mode. **Fn+Ctrl+n** on its own (no Fn+Q before
   it) just switches to the device already paired on slot n.

The device name carries the slot number: slot 2 shows up as `HHKB-Studio2` (checked).
If you press **Fn+Ctrl+n first and then Fn+Q**, the Studio ends up pairing on a different slot
than the one you meant. On the owner's board it advertised `HHKB-Studio1` instead of
`HHKB-Studio2`, and connecting from the Mac failed: macOS sent the connection request, the Studio
never answered, and Settings gave up after about 30 s. Doing the steps in the order above fixed it.

> **Stay on keymap Profile1.** The manual says to press **Fn+C, then 2** (keymap **Profile2**)
> after pairing with a Mac. That's a keymap profile, not the Bluetooth slot. The owner's keymap
> (this file) is written to **Profile1**, so switching to Profile2 would replace it. Skip that step.

**Keyboard Setup Assistant (finding):** after pairing the Studio over Bluetooth this way, macOS
ran the Keyboard Setup Assistant. It registered the keys next to the Shift keys correctly and
identified the board as JIS. No separate Bluetooth entry appeared in
`/Library/Preferences/com.apple.keyboardtype`: the Studio keeps using `"21-1278-0" = 42`, the
same as over USB. If the assistant misreads those keys, or doesn't appear, set JIS by hand
(`02-macos.md`).

## Bluetooth: one combined device

Over USB, Karabiner lists the Studio three times (keyboard, pointing device, consumer). Over
**Bluetooth** it lists it **once**, as `HHKB-Studio<n>` (n = pairing slot), with both flags set:

```
HHKB-Studio2 {'is_keyboard': True, 'is_pointing_device': True, 'product_id': 21, 'vendor_id': 1278}
transport: Bluetooth Low Energy
```

- Karabiner matches device identifiers exactly. The USB-style `is_keyboard`-only entry doesn't
  match this device, and Karabiner treats it as a pointing device it only *observes*. Nothing is
  remapped: Fn1+P isn't Play/Pause (with the old `` ` `` remap, 半/全 also typed `` ` `` instead of
  switching to Korean). The global rules (ろ, Print Screen) don't apply either, because they only run on grabbed devices.
  `/var/log/karabiner/core_service.log` shows `HHKB-Studio<n> … monitor is started (grabbed)` once
  the combined entry is in place.
- Fix: the second Studio entry in `03-karabiner.md`
  (`is_keyboard` + `is_pointing_device`, vendor `1278`, product `21`).
- Side effect: Karabiner can't grab only the keyboard half of a combined device, so it also grabs
  the pointing stick and gesture pads. Checked on the owner's board: the stick, vertical and
  horizontal scroll, and the arrow pads all keep working through Karabiner.
- The number in the name is the Bluetooth pairing slot (`HHKB-Studio1` … `HHKB-Studio4`). The
  entries match on vendor/product ID, not the name, so one combined entry covers every slot.
  Checked on slot 2: the HHKB-key role → Korean, 英数 → ABC, かな → Kana, Fn1+P → Play/Pause, and the stick and
  scroll pads all work.
- Also checked on a second machine: **MacBook Air (M4), macOS 27.0.1**, Studio paired over
  Bluetooth on **slot 3** (`HHKB-Studio3`), Karabiner-Elements 16.3.0, Hammerspoon 1.1.1. Rows 3–15 of the
  checklist in `06-verification.md` passed, except that in Korean 2-Set ろ typed `¥` instead of
  `\` (accepted then; checked before JIS Korean became the default, which types `\` itself).

## Using the gesture pads

The pads are touch strips on the **edges of the case**, not on the top: one on the left side, one
on the right side, and two on the front face (below the space bar), front-left and front-right.
Slide a bare fingertip **along** the strip. Taps and brushes over the top edge don't register.

| Pad | Slide | Owner's setting |
| --- | --- | --- |
| Left side | front ↔ back | **Off** (was ↑ / ↓) |
| Right side | front ↔ back | vertical scroll |
| Front left | left ↔ right, along the front face | **Off** (was ← / →) |
| Front right | left ↔ right, along the front face | horizontal scroll |

Turn a pad back on with Fn2+A / S / D / F (Fn2 layer below), or in the keymap tool.

If the front pads seem dead while the side pads work, check the technique first: slide along the
front face itself, and keep a wrist rest or resting thumbs off that edge. On the owner's board the
front pads "didn't work" until they were used this way. Nothing in the keymap or on the Mac was
wrong.

## Standard layer

![HHKB Studio keymap, Standard layer](images/hhkb-studio/keymap-standard.png)

Stock JIS layout except for the bottom row:

```
Fn1 | F18 | Opt | ⌘ | 英数 | Space | かな | ⌘ | Opt | Fn1 | ← | ↓ | →
              LB | Fn2 | RB          (pointing-stick buttons, below Space)
```

- **F18** (in the 半/全 position) is the Studio's "HHKB key" (the Studio has no HHKB-logo key on
  the bottom row). It emits `f18` itself → JIS Korean via Hammerspoon.
- **Opt / ⌘ / 英数** on the left and **かな / ⌘ / Opt** on the right match the HYBRID Type-S
  layout.
- The key left of A is **Control**. ↑ sits in the Shift row, left of the right Shift.

## Fn1 layer

![HHKB Studio keymap, Fn1 layer](images/hhkb-studio/keymap-fn1.png)

Hold either **Fn1** key (bottom row, outer ends). Keys not listed pass through as Standard.

| Key | Fn1 + key |
| --- | --- |
| Esc | Power |
| 1 … 0, -, ^ | F1 … F12 |
| ¥ / BS | Insert / Delete |
| Tab | Caps Lock |
| I / O / P | **Print Screen** (→ Cmd+Shift+4 via Karabiner) / Scroll Lock / **Pause** (→ Play/Pause via Karabiner) |
| @ | ↑ |
| Enter | Numpad Enter |
| A / S / D / F | Volume Down / Volume Up / Mute / Eject |
| H / J | Numpad `*` / Numpad `/` |
| K / L | Home / Page Up |
| ; / : | ← / → |
| N / M | Numpad `+` / Numpad `-` |
| , / . | End / Page Down |
| / | ↓ |
| ↑ (Shift row) | Right Shift |
| ← / ↓ / → | Delete / Right ⌘ / Right Control |

## Fn2 layer

![HHKB Studio keymap, Fn2 layer](images/hhkb-studio/keymap-fn2.png)

Hold **Fn2** (between LB and RB). Pointing-stick and gesture-pad controls:

| Key | Fn2 + key |
| --- | --- |
| 1 / 2 / 3 / 4 | Pointing-stick speed 1 / 2 / 3 / 4 |
| G / B | Pointing-stick speed + / − |
| 6 / 7 / 8 / 9 | Gesture-pad speed Low / Mid / Hi / Max |
| A / S / D / F | Gesture pad on: Left Side / Front Left / Front Right / Right Side |
| Z / X / C / V | Gesture pad off: Left Side / Front Left / Front Right / Right Side |
| Space | Middle mouse button |

The bottom row stays the same as Standard, so F18, 英数 and かな work while Fn2 is held.

## Fn3 layer

![HHKB Studio keymap, Fn3 layer](images/hhkb-studio/keymap-fn3.png)

Unused: identical to Standard.

## Check

With Karabiner and Hammerspoon set up, run the checklist in `06-verification.md` on the Studio.
Rows 3–14 cover the bottom row (F18 → `f18`), Fn1+P and Fn1+I.
