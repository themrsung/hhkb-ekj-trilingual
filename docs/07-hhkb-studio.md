# 07 — HHKB Studio (JIS): keymap and settings

The owner's HHKB Studio JIS, as set in PFU's keymap tool for the Studio (**Profile1**). Use this
as the reference when setting up a Studio. `01-hardware.md` covers the parts both models share
(the bottom-row codes and the DIP-before-keymap order). This file covers what's specific to the
Studio.

Checked on the owner's board: with these settings, the Standard bottom row emits
`grave_accent_and_tilde`, `left_option`, `left_command`, `japanese_eisuu`, `spacebar`,
`japanese_kana`, `right_command`, `right_option`, then the arrows. Fn1+P emits `pause` (macOS sees
F15), and Fn1+I emits `print_screen` (macOS sees F13).

## Settings outside the keymap

| Setting | Value |
| --- | --- |
| DIP switches | **SW4 ON** (reverse scroll direction), SW1–3, 5, 6 OFF |
| Pointing stick → Button | **On** (LB / Fn2 / RB keys below the space bar) |
| Gesture Pad – Left Side | On: ↑ / ↓ (arrow keys) |
| Gesture Pad – Front Left | On: ← / → (arrow keys) |
| Gesture Pad – Front Right | On: **MW←** / **MW→** (horizontal mouse wheel) |
| Gesture Pad – Right Side | On: MW↓ / MW↑ (vertical mouse wheel) |

As with the HYBRID Type-S, **set the DIP switches before writing the keymap** (`01-hardware.md`).

**Finding: flipping SW4 doesn't reset the Studio's keymap.** On the owner's board, SW4 was
flipped from **ON to OFF** with a custom keymap written by the HHKB Studio Keymap Tool in place.
Afterwards everything was the same: the custom keybinds were all still there. So changing the
scroll direction with SW4 is safe on the Studio. Only SW4 ON → OFF was tested. Other switches
and OFF → ON weren't, so for those, set DIP switches first and recheck the keymap afterwards.
This differs from the HYBRID Type-S, where changing DIP switches overwrites the keymap.

Karabiner's only Studio-specific config is the device entries for vendor `1278` / product `21`
(`03-karabiner.md`): one `is_keyboard` entry for USB, and one combined entry for Bluetooth (below).
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
  remapped: 半/全 types `` ` `` instead of switching to Korean, and Fn1+P isn't Play/Pause. The
  global rules (ろ, Print Screen) don't apply either, because they only run on grabbed devices.
  `/var/log/karabiner/core_service.log` shows `HHKB-Studio<n> … monitor is started (grabbed)` once
  the combined entry is in place.
- Fix: the second Studio entry in `03-karabiner.md`
  (`is_keyboard` + `is_pointing_device`, vendor `1278`, product `21`).
- Side effect: Karabiner can't grab only the keyboard half of a combined device, so it also grabs
  the pointing stick and gesture pads. Checked on the owner's board: the stick, vertical and
  horizontal scroll, and the arrow pads all keep working through Karabiner.
- The number in the name is the Bluetooth pairing slot (`HHKB-Studio1` … `HHKB-Studio4`). The
  entries match on vendor/product ID, not the name, so one combined entry covers every slot.
  Checked on slot 2: 半/全 → Korean, 英数 → ABC, かな → Kana, Fn1+P → Play/Pause, and the stick and
  scroll pads all work.
- Also checked on a second machine: **MacBook Air (M4), macOS 27.0.1**, Studio paired over
  Bluetooth on **slot 3** (`HHKB-Studio3`), Karabiner-Elements 16.3.0, Hammerspoon 1.1.1. Rows 3–15 of the
  checklist in `06-verification.md` passed, except that in Korean 2-Set ろ types `¥` instead of
  `\` (accepted; see the note under the checklist).

## Using the gesture pads

The pads are touch strips on the **edges of the case**, not on the top: one on the left side, one
on the right side, and two on the front face (below the space bar), front-left and front-right.
Slide a bare fingertip **along** the strip. Taps and brushes over the top edge don't register.

| Pad | Slide | Owner's setting |
| --- | --- | --- |
| Left side | front ↔ back | ↑ / ↓ |
| Right side | front ↔ back | vertical scroll |
| Front left | left ↔ right, along the front face | ← / → |
| Front right | left ↔ right, along the front face | horizontal scroll |

If the front pads seem dead while the side pads work, check the technique first: slide along the
front face itself, and keep a wrist rest or resting thumbs off that edge. On the owner's board the
front pads "didn't work" until they were used this way. Nothing in the keymap or on the Mac was
wrong.

## Standard layer

![HHKB Studio keymap, Standard layer](images/hhkb-studio/keymap-standard.png)

Stock JIS layout except for the bottom row:

```
Fn1 | 半/全 | Opt | ⌘ | 英数 | Space | かな | ⌘ | Opt | Fn1 | ← | ↓ | →
              LB | Fn2 | RB          (pointing-stick buttons, below Space)
```

- **半/全** is the Studio's "HHKB key" (the Studio has no HHKB-logo key on the bottom row). It
  emits `grave_accent_and_tilde`, which Karabiner turns into F18 → Korean 2-Set.
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

The bottom row stays the same as Standard, so 半/全, 英数 and かな work while Fn2 is held.

## Fn3 layer

![HHKB Studio keymap, Fn3 layer](images/hhkb-studio/keymap-fn3.png)

Unused: identical to Standard.

## Check

With Karabiner and Hammerspoon set up, run the checklist in `06-verification.md` on the Studio.
Rows 3–14 cover the bottom row, Fn1+P and Fn1+I.
