# 01 — Hardware: HHKB Professional HYBRID Type-S (JIS)

## Identify the keyboard

| Property | Value |
| --- | --- |
| Model | HHKB Professional HYBRID Type-S, JIS layout (日本語配列) |
| Manufacturer | PFU Limited |
| USB vendor ID | `1278` (0x04FE) |
| USB product ID | `34` (0x0022) |
| Karabiner device name | `HHKB-Hybrid` |
| macOS keyboard type | JIS (`42` in `/Library/Preferences/com.apple.keyboardtype`) |

Confirm on the target machine:

```sh
'/Library/Application Support/org.pqrs/Karabiner-Elements/bin/karabiner_cli' --list-connected-devices \
  | python3 -c "import json,sys;[print(d.get('product'),d['device_identifiers']) for d in json.load(sys.stdin)]"
# expect: HHKB-Hybrid {'is_keyboard': True, 'product_id': 34, 'vendor_id': 1278}
```

If the IDs are different (another HHKB revision, for example), use the real IDs everywhere
`1278` / `34` appear in `03-karabiner.md`.

## Step 1 — DIP switches (do this FIRST)

The DIP switches are on the underside, under the small cover. The keyboard should be **off or
unplugged** while you flip them.

| SW1 | SW2 | SW3 | SW4 | SW5 | SW6 |
| --- | --- | --- | --- | --- | --- |
| **ON** | OFF | OFF | OFF | **ON** | OFF |

That is **Mac mode** on the HYBRID Type-S JIS.

> **DIP switches must be set BEFORE the HHKB keymaps.** Changing DIP switches overwrites the
> keymap written with the HHKB Keymap Tool. If you change them later, apply the keymap again.

## Step 2 — Bottom-row keymap (HHKB Keymap Tool)

Use PFU's **HHKB Keymap Tool** (free download from PFU / HHKB official site; connect over USB)
to make the bottom row match this layout, left to right:

| # | Legend / role | Must emit (Karabiner key code) | macOS meaning |
| --- | --- | --- | --- |
| 1 | Fn | (HHKB internal Fn layer) | — |
| 2 | **HHKB** (logo key) | `grave_accent_and_tilde` | Karabiner turns it into `f18` → Korean 2-Set |
| 3 | Option | `left_option` | ⌥ |
| 4 | Command | `left_command` | ⌘ |
| 5 | **英数 (Romaji)** | `japanese_eisuu` | → English ABC |
| 6 | Space | `spacebar` | |
| 7 | **かな (Kana)** | `japanese_kana` | → Japanese (Kana typing) |
| 8 | Command | `right_command` | ⌘ |
| 9 | Option | `right_option` | ⌥ |
| 10 | Fn | (HHKB internal Fn layer) | — |
| 11–13 | ← ↓ → | `left_arrow`, `down_arrow`, `right_arrow` | |

Other fixed points of the layout:

- **The key left of A is Control** (`left_control`). The HHKB has no Caps Lock key; Caps Lock is
  **Fn+Tab**.
- The ↑ arrow is the key above ↓ (right of the right Shift row), as usual on the JIS HHKB.
- Fn-layer keys this setup depends on (HHKB stock Fn layer, nothing to change):
  - **Fn+P → `pause`**. Karabiner turns this into media Play/Pause.
  - **Fn+I → `print_screen`**. Karabiner turns this into Cmd+Shift+4.

Use the keycaps that match these roles (Option/Command swapped as needed) so the legends match.

## Step 3 — Check with Karabiner-EventViewer

After Karabiner is installed (`03-karabiner.md`), open **Karabiner-EventViewer → Main** and
press each bottom-row key. The "key_code" column must match the table above. EventViewer shows
the codes *after* Karabiner's changes, so the HHKB key shows `f18` once the rules are in place.
Use the **Unknown Events** tab or turn off the device's modifications for a moment if you need
the raw code.
