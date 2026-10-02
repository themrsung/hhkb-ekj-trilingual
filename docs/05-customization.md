# 05 — Customization

This repo has the owner's settings and they're the default. Users can change them: edit the
values below **directly in their own config**. Don't add a settings file, wrapper, or generator
to this repo.

## HHKB key → any input source

Korean 2-Set is plug-and-play, but the HHKB key can switch to **any** input source. Change the
`KOREAN` constant in `~/.hammerspoon/init.lua` (renaming it is fine) to the target source's ID,
and enable that source in System Settings → Keyboard → Input Sources.

| Target | Input source ID |
| --- | --- |
| Korean 2-Set (default) | `com.apple.inputmethod.Korean.2SetKorean` |
| Korean 3-Set (390 / Final) | `com.apple.inputmethod.Korean.390Sebulshik` / `com.apple.inputmethod.Korean.3SetKorean` |
| German QWERTZ | `com.apple.keylayout.German` |
| Swiss German QWERTZ | `com.apple.keylayout.SwissGerman` |
| French AZERTY | `com.apple.keylayout.French` |
| Belgian AZERTY | `com.apple.keylayout.Belgian` |
| US (instead of ABC) | `com.apple.keylayout.US` |
| Dvorak | `com.apple.keylayout.Dvorak` |
| Chinese Pinyin | `com.apple.inputmethod.SCIM.ITABC` |

Get the exact IDs on the machine (they change between macOS releases):

```sh
hs -c 'hs.inspect(hs.keycodes.layouts(true))'   # keyboard layouts
hs -c 'hs.inspect(hs.keycodes.methods(true))'   # input methods / modes
hs -c 'hs.keycodes.currentSourceID()'           # whatever is active right now
```

Keep the always-select + check-then-retry structure for any target (see `04-hammerspoon.md`).
For a plain keyboard layout (QWERTZ, AZERTY) the retry rarely fires, and it costs nothing.

## 英数 / かな keys → other targets

By default these use native macOS behavior (英数 → ABC, かな → Japanese Kana). To send either key
somewhere else, use the same pattern as the HHKB key:

1. Karabiner: add to the HHKB device's `simple_modifications`, for example
   `japanese_eisuu → f19` or `japanese_kana → f20`.
2. Hammerspoon: `hs.hotkey.bind({}, "f19", function() … end)` with the same select/check/retry
   body and a different ID.

To get Japanese **Romaji** typing on かな instead of Kana typing, change the Japanese input
source's Input setting to Romaji (`02-macos.md`) instead of remapping anything. Note that the
ろ rule's `^ja$` condition then also skips Romaji-typing Japanese.

## Window cycling shortcut

Option+Tab is the repo default for "Move focus to next window". Any shortcut works. Change it in
System Settings → Keyboard → Keyboard Shortcuts → Keyboard, or write different `parameters` to
symbolic hotkey 27 (`02-macos.md`). Parameters are `(character code or 65535, virtual keycode,
modifier mask)`. Modifier masks: Shift 131072, Control 262144, Option 524288, Command 1048576
(add them together for combinations).

## ろ key

To keep `\` from ろ in Japanese too (making ろ impossible to type in Kana), remove the
`conditions` array from the ろ rule. Not recommended.

## Things to keep regardless

- DIP switches **before** the Keymap Tool (otherwise the keymap gets overwritten).
- Karabiner virtual keyboard = JIS and macOS keyboard type for the HHKB = JIS.
