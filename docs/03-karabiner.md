# 03 — Karabiner-Elements

Checked with Karabiner-Elements **16.3.0** (config schema with `keyboard_type_v2`).

## Install and permissions

```sh
brew install --cask karabiner-elements
```

Or download the `.dmg` from <https://karabiner-elements.pqrs.org/>. Then the **user** must:

1. Open Karabiner-Elements and follow its setup prompts.
2. System Settings → General → **Login Items & Extensions** → Driver Extensions → allow
   **Karabiner-DriverKit-VirtualHIDDevice**.
3. System Settings → Privacy & Security → **Input Monitoring** → allow `karabiner_grabber`
   and `karabiner_observer` (Karabiner-Core-Service on newer versions).
4. Karabiner-Elements → **Devices**: make sure **HHKB-Hybrid** has "Modify events" **on**.
5. Karabiner-Elements → **Virtual Keyboard**: keyboard type **JIS** (ISO/ANSI are wrong for
   this setup). This is `"virtual_hid_keyboard": {"keyboard_type_v2": "jis"}` below.

## What Karabiner does in this setup

| # | Scope | From | To | Purpose |
| --- | --- | --- | --- | --- |
| A | HHKB only (simple modification) | `grave_accent_and_tilde` (HHKB key) | `f18` | Hammerspoon catches F18 and selects Korean 2-Set |
| B | HHKB only (simple modification) | `pause` (Fn+P) | consumer `play_or_pause` | Media play/pause |
| C | Global (complex rule) | `international1` (ろ), Caps Lock optional | `international3` (¥) + `left_option` | Types `\` unshifted; Shift+ろ isn't matched and types `_` as usual. **Only when the input source language isn't `ja`** |
| D | Global (complex rule) | `print_screen` (Fn+I on HHKB), any modifiers | `4` + `left_command` + `left_shift` | Screenshot selection |
| E | Profile | — | `virtual_hid_keyboard.keyboard_type_v2 = "jis"` | Karabiner's virtual keyboard must report JIS, or ¥/ろ/英数/かな come out wrong |

Notes:

- **A** uses a simple modification, so it applies whatever modifiers you hold. Hammerspoon only
  binds bare F18, so Shift/Cmd+HHKB does nothing. That's fine: on JIS, `` ` `` is Shift+@ and
  `~` is Shift+^, so you lose nothing.
- **C**: on the macOS JIS ABC layout, both ろ and Shift+ろ type `_`, and the backslash is only
  reachable as Option+¥. The rule makes ろ type `\` (by sending Option+¥) and leaves Shift+ろ
  untouched so `_` still works. `from.modifiers.optional` lists only `caps_lock`, so a held
  Shift means *no match* and the key passes through. The `input_source_unless language ^ja$`
  condition keeps ろ typeable in Japanese Kana. Without it, ろ would be impossible to type.
- **D** is a global rule that came before the HHKB setup. The HHKB is the keyboard that sends
  `print_screen` (via Fn+I), so it's part of this behavior set. Leave it out only if the user
  asks.
- The HHKB key (`grave_accent_and_tilde`) is remapped with a **device-scoped simple
  modification**. Other keyboards keep their `` ` `` key.

## Fresh install: full `~/.config/karabiner/karabiner.json`

Use this only if the file doesn't exist yet or holds only the default empty profile:

```json
{
  "profiles": [
    {
      "name": "Default profile",
      "selected": true,
      "virtual_hid_keyboard": { "keyboard_type_v2": "jis" },
      "complex_modifications": {
        "rules": [
          {
            "description": "Print Screen to Cmd+Shift+4",
            "manipulators": [
              {
                "type": "basic",
                "from": { "key_code": "print_screen", "modifiers": { "optional": ["any"] } },
                "to": [ { "key_code": "4", "modifiers": ["left_command", "left_shift"] } ]
              }
            ]
          },
          {
            "description": "JIS ろ (_) → \\ unshifted, _ when shifted (non-Japanese input sources only)",
            "manipulators": [
              {
                "type": "basic",
                "from": { "key_code": "international1", "modifiers": { "optional": ["caps_lock"] } },
                "to": [ { "key_code": "international3", "modifiers": ["left_option"] } ],
                "conditions": [
                  { "type": "input_source_unless", "input_sources": [ { "language": "^ja$" } ] }
                ]
              }
            ]
          }
        ]
      },
      "devices": [
        {
          "identifiers": { "is_keyboard": true, "vendor_id": 1278, "product_id": 34 },
          "ignore": false,
          "simple_modifications": [
            { "from": { "key_code": "grave_accent_and_tilde" }, "to": [ { "key_code": "f18" } ] },
            { "from": { "key_code": "pause" }, "to": [ { "consumer_key_code": "play_or_pause" } ] }
          ]
        }
      ]
    }
  ]
}
```

Karabiner watches this file and reloads it as soon as it changes. Karabiner also rewrites the
file in its own format the next time a setting changes in the GUI. That's expected.

## Existing install: merge without clobbering

If the user already has rules (other devices, games, …), **merge**. Back up first:

```sh
KJ=~/.config/karabiner/karabiner.json
cp "$KJ" "$KJ.bak-$(date +%Y%m%d%H%M%S)"
```

Then apply this to the **selected** profile with `jq`. It's idempotent: it replaces rules and
the HHKB device entry by their description and identifiers, and never adds duplicates:

```sh
KJ=~/.config/karabiner/karabiner.json
jq '
  def hhkb_rules: [
    { "description": "Print Screen to Cmd+Shift+4",
      "manipulators": [ { "type": "basic",
        "from": { "key_code": "print_screen", "modifiers": { "optional": ["any"] } },
        "to": [ { "key_code": "4", "modifiers": ["left_command", "left_shift"] } ] } ] },
    { "description": "JIS ろ (_) → \\ unshifted, _ when shifted (non-Japanese input sources only)",
      "manipulators": [ { "type": "basic",
        "from": { "key_code": "international1", "modifiers": { "optional": ["caps_lock"] } },
        "to": [ { "key_code": "international3", "modifiers": ["left_option"] } ],
        "conditions": [ { "type": "input_source_unless", "input_sources": [ { "language": "^ja$" } ] } ] } ] }
  ];
  def hhkb_device: {
    "identifiers": { "is_keyboard": true, "vendor_id": 1278, "product_id": 34 },
    "ignore": false,
    "simple_modifications": [
      { "from": { "key_code": "grave_accent_and_tilde" }, "to": [ { "key_code": "f18" } ] },
      { "from": { "key_code": "pause" }, "to": [ { "consumer_key_code": "play_or_pause" } ] } ] };
  .profiles |= map(
    if .selected then
      .virtual_hid_keyboard = ((.virtual_hid_keyboard // {}) + { "keyboard_type_v2": "jis" })
      | .complex_modifications.rules = (
          [ (.complex_modifications.rules // [])[]
            | select(.description as $d | [hhkb_rules[].description] | index($d) | not) ]
          + hhkb_rules)
      | .devices = (
          [ (.devices // [])[]
            | select((.identifiers.vendor_id == 1278 and .identifiers.product_id == 34
                      and (.identifiers.is_keyboard // false)) | not) ]
          + [hhkb_device])
    else . end)
' "$KJ" > "$KJ.tmp" && mv "$KJ.tmp" "$KJ"
```

If an existing HHKB device entry has *other* simple modifications or settings the user added,
put them back into `hhkb_device` before running this. The script replaces that entry.

## Check

```sh
jq '.profiles[] | select(.selected) | {vk: .virtual_hid_keyboard,
     rules: [.complex_modifications.rules[].description],
     hhkb: [.devices[] | select(.identifiers.vendor_id==1278)]}' ~/.config/karabiner/karabiner.json
```

Then in Karabiner-EventViewer: the HHKB key → `f18`, Fn+P → `play_or_pause`, Fn+I →
`left_command`+`left_shift`+`4`, ろ (in ABC or Korean) → `left_option`+`international3`.

## Troubleshooting: 英数 doesn't reach ABC from Korean

The default setup relies on macOS' own 英数 handling (see `02-macos.md`). If on some macOS
release 英数 stops reaching ABC from Korean (for example it leaves you in Korean), don't change
the default behavior. Make it explicit with a Karabiner rule that also keeps the key's native
event:

```json
{
  "description": "英数 → always ABC",
  "manipulators": [ {
    "type": "basic",
    "from": { "key_code": "japanese_eisuu", "modifiers": { "optional": ["any"] } },
    "to": [ { "key_code": "japanese_eisuu" },
            { "select_input_source": { "input_source_id": "^com\\.apple\\.keylayout\\.ABC$" } } ]
  } ]
}
```

The current setup doesn't need this rule and doesn't include it.
