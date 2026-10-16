# 01 — Hardware: HHKB Professional HYBRID Type-S / HHKB Studio (JIS)

Two JIS HHKB models are supported. Both use the same bottom-row codes, the same Karabiner
modifications (scoped per device), and the same Hammerspoon config. Only the USB product ID and
the DIP switches differ.

## Identify the keyboard

| Property | HYBRID Type-S | Studio |
| --- | --- | --- |
| Model | HHKB Professional HYBRID Type-S, JIS layout (日本語配列) | HHKB Studio, JIS layout (日本語配列) |
| Manufacturer | PFU Limited | PFU Limited |
| USB vendor ID | `1278` (0x04FE) | `1278` (0x04FE) |
| USB product ID | `34` (0x0022) | `21` (0x0015) |
| Karabiner device name | `HHKB-Hybrid` | USB: `HHKB-Studio` (shows up 3 times: keyboard, pointing device, consumer). Bluetooth: `HHKB-Studio<n>`, n = pairing slot (one combined keyboard + pointing device; see `07-hhkb-studio.md`) |
| macOS keyboard type | JIS (`42` in `/Library/Preferences/com.apple.keyboardtype`) | JIS (`42`) |

Confirm on the target machine:

```sh
'/Library/Application Support/org.pqrs/Karabiner-Elements/bin/karabiner_cli' --list-connected-devices \
  | python3 -c "import json,sys;[print(d.get('product'),d['device_identifiers']) for d in json.load(sys.stdin)]"
# expect one or both of:
#   HHKB-Hybrid {'is_keyboard': True, 'product_id': 34, 'vendor_id': 1278}
#   HHKB-Studio {'is_keyboard': True, 'product_id': 21, 'vendor_id': 1278}
#   (over USB the Studio also lists is_pointing_device and is_consumer entries; only the is_keyboard one is configured)
#   HHKB-Studio2 {'is_keyboard': True, 'is_pointing_device': True, 'product_id': 21, 'vendor_id': 1278}
#   (Studio over Bluetooth: ONE combined entry, which needs its own device entry in 03-karabiner.md)
```

If the IDs are different (another HHKB revision, for example), use the real IDs wherever
`1278` / `34` / `21` appear in `03-karabiner.md`. Configure only the models the user actually
has. An entry for a keyboard that isn't connected is harmless, though.

## Step 1 — DIP switches (do this FIRST)

The DIP switches are on the underside, under the small cover. The keyboard should be **off or
unplugged** while you flip them.

**HYBRID Type-S JIS** (Mac mode):

| SW1 | SW2 | SW3 | SW4 | SW5 | SW6 |
| --- | --- | --- | --- | --- | --- |
| **ON** | OFF | OFF | OFF | **ON** | OFF |

**Studio JIS** (the owner's setting): **all switches OFF**. SW4 ON reverses the scroll direction
if you want that (`05-customization.md`). With SW4 either ON or OFF, the owner's Studio emits the
bottom-row codes in Step 2 (checked on a Mac).

> **Set the DIP switches BEFORE the keymap.** On the HYBRID Type-S, changing DIP switches
> overwrites the keymap written with the HHKB Keymap Tool, and you have to apply it again.
> **On the Studio this was tested for SW4 only:** flipping SW4 in **either direction** (ON → OFF
> and OFF → ON) did **not** reset the custom keymap written with the HHKB Studio Keymap Tool
> (`07-hhkb-studio.md`). The other switches weren't tested, so keep the DIP-first order on the
> Studio too.

## Step 2 — Bottom-row keymap

Use PFU's keymap tool for the model (**HHKB Keymap Tool** for the HYBRID Type-S, the Studio's
own keymap tool for the Studio; free downloads from the PFU / HHKB official site) so the bottom
row emits these codes, left to right. Check first. If the board already emits them (the owner's
Studio did), there's nothing to write.

| # | Legend / role | Must emit (Karabiner key code) | macOS meaning |
| --- | --- | --- | --- |
| 1 | Fn | (HHKB internal Fn layer) | — |
| 2 | **HHKB** (logo key; **半/全** on the Studio) | `grave_accent_and_tilde` | Karabiner turns it into `f18` → Korean 2-Set |
| 3 | Option | `left_option` | ⌥ |
| 4 | Command | `left_command` | ⌘ |
| 5 | **英数 (Romaji)** | `japanese_eisuu` | → English ABC |
| 6 | Space | `spacebar` | |
| 7 | **かな (Kana)** | `japanese_kana` | → Japanese (Kana typing) |
| 8 | Command | `right_command` | ⌘ |
| 9 | Option | `right_option` | ⌥ |
| 10 | Fn | (HHKB internal Fn layer) | — |
| 11–13 | ← ↓ → | `left_arrow`, `down_arrow`, `right_arrow` | |

On the Studio, Fn is labeled **Fn1**, and the pointing-stick buttons and gesture pads aren't
part of the trilingual setup. The owner's full Studio keymap (all layers, with screenshots of the
keymap tool), pointing-stick and gesture-pad settings are in
[`07-hhkb-studio.md`](07-hhkb-studio.md).

Other fixed points of the layout:

- **The key left of A is Control** (`left_control`). The HHKB has no Caps Lock key; Caps Lock is
  **Fn+Tab**.
- The ↑ arrow is the key above ↓ (right of the right Shift row), as usual on the JIS HHKB.
- Fn-layer keys this setup depends on (stock Fn layer on both models, nothing to change):
  - **Fn+P → `pause`** (macOS sees it as F15). Karabiner turns this into media Play/Pause.
  - **Fn+I → `print_screen`** (macOS sees it as F13). Karabiner turns this into Cmd+Shift+4.

Use the keycaps that match these roles (Option/Command swapped as needed) so the legends match.

## Step 3 — Check with Karabiner-EventViewer

After Karabiner is installed (`03-karabiner.md`), open **Karabiner-EventViewer → Main** and
press each bottom-row key **on each HHKB**. The "key_code" column must match the table above.
EventViewer shows the codes *after* Karabiner's changes, so the HHKB key shows `f18` once the
rules are in place. Use the **Unknown Events** tab or turn off the device's modifications for a
moment if you need the raw code.
