# 06 — Verification checklist and troubleshooting

Don't call the setup done until every row passes. Rows marked 👤 need the user to press keys and
tell you the result. Ask them, and don't assume. With both models, run rows 3–14 on **each** HHKB.

## Acceptance checklist

| # | Action | Expected |
| --- | --- | --- |
| 1 | `karabiner_cli --list-connected-devices` | `HHKB-Hybrid` with vendor 1278 / product 34 and/or `HHKB-Studio` (keyboard) with vendor 1278 / product 21 |
| 2 | `defaults read /Library/Preferences/com.apple.keyboardtype` | each HHKB (`34-1278-…`, `21-1278-0`) = 42; Karabiner virtual keyboard (`593-1452-0`) = 42 |
| 3 | 👤 Press the key left of A + C in Terminal | Interrupts (Control) |
| 4 | 👤 EventViewer: bottom row left → right | `fn`, `f18` (HHKB key on the HYBRID Type-S, F18 key on the Studio), `left_option`, `left_command`, `japanese_eisuu`, `spacebar`, `japanese_kana`, `right_command`, `right_option`, `fn`, arrows |
| 5 | 👤 From ABC, press HHKB key | Korean 2-Set, overlay shown **once** |
| 6 | 👤 From Japanese, press HHKB key | Korean 2-Set, overlay shown **once** |
| 7 | 👤 While in Korean, press HHKB key | Still Korean 2-Set; key is never "dead" |
| 8 | 👤 From Korean or Japanese, press 英数 | ABC (English QWERTY) |
| 9 | 👤 From ABC or Korean, press かな | Japanese, **Kana** typing (`a` key types ち) |
| 10 | 👤 In ABC: ろ / Shift+ろ | `\` / `_` |
| 11 | 👤 In Korean 2-Set: ろ / Shift+ろ | `\` / `_` (`¥` / `_` is acceptable, see the note below) |
| 12 | 👤 In Japanese Kana: ろ | ろ |
| 13 | 👤 Fn+P with music playing | Play/Pause toggles |
| 14 | 👤 Fn+I | Screenshot selection crosshair (Cmd+Shift+4) |
| 15 | 👤 Option+Tab with two windows of the same app | Focus moves to the app's other window |
| 16 | Reboot or log out/in, repeat 5 and 8–9 | Still works (Karabiner and Hammerspoon start at login) |

> **Note on row 11 (Korean 2-Set: ろ types `¥`).** On a MacBook Air (M4) running macOS 27.0.1, with
> an HHKB Studio JIS connected over **Bluetooth** (`HHKB-Studio3`, Karabiner-Elements 16.3.0,
> Hammerspoon 1.1.1), ろ typed `¥` instead of `\` in Korean 2-Set. Shift+ろ still typed `_`, and in
> ABC ろ typed `\` as expected. Rows 3–15 otherwise passed (row 16 was not run). The rule sends Option+¥, and Korean 2-Set
> evidently keeps that as `¥`. This is acceptable and not treated as a failure: use ABC (英数)
> when you need `\`. No extra rule was added for it.

## Troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| Bottom row keys are wrong after flipping a DIP switch | DIP change overwrote the keymap (HYBRID Type-S; on the Studio, flipping SW4 either way was tested and doesn't do this) | Set DIP switches first (HYBRID Type-S: 1 & 5 ON; Studio: all OFF), then apply the keymap again with the keymap tool |
| HHKB key types `` ` `` / `^` / nothing in EventViewer | Karabiner not modifying the HHKB, or device IDs differ | Karabiner → Devices → enable HHKB-Hybrid / HHKB-Studio; check that `karabiner.json` has a device entry for that model's IDs (34 / 21) |
| EventViewer shows `f18` but the source doesn't change | Hammerspoon not running or config not loaded | Start Hammerspoon, Reload Config, check the console for errors; Launch at login on |
| Menu bar says Korean but the field still types English/Kana (rare, ~1%) | Known macOS bug: a background `TISSelectInputSource` didn't reach the focused field. The retry guard can't detect it | Press the HHKB key again, or click out of the field and back. See "Known issue" in `04-hammerspoon.md`. Don't add a focus-bounce or unconditional retry |
| Korean overlay appears twice | Retry is missing its check | Restore the `if hs.keycodes.currentSourceID() ~= KOREAN` guard (`04-hammerspoon.md`) |
| HHKB key does nothing while "already" in Korean | Someone added an early return | Remove `if … == KOREAN then return end` (`04-hammerspoon.md`) |
| 英数 lands in Japanese 英字 mode instead of ABC | Romaji/英字 input mode enabled in the Japanese IME | Turn it off (`02-macos.md`); if needed add the optional rule in `03-karabiner.md` |
| かな gives Romaji input | Japanese input set to Romaji | Japanese input source → Input: Kana |
| ろ types `_` in ABC | Rule missing, or virtual keyboard not JIS | Check rule C and `keyboard_type_v2: "jis"` |
| ろ types `\` in Japanese | Rule's `input_source_unless ^ja$` condition missing | Restore the condition |
| `¥` / `\` / `_` come out swapped or wrong everywhere | macOS keyboard type for the HHKB isn't JIS | System Settings → Keyboard → Change Keyboard Type… → JIS |
| Studio over Bluetooth: Fn1+P isn't Play/Pause, ろ / Print Screen rules don't apply | Karabiner sees the Bluetooth Studio as one combined keyboard + pointing device, and the keyboard-only entry doesn't match it | Add the combined Studio entry (`is_keyboard` + `is_pointing_device`, product 21) from `03-karabiner.md`; see `07-hhkb-studio.md` |
| Studio front gesture pads do nothing (side pads work) | Swiping the wrong way, or a wrist rest / thumb resting on the front edge | Slide along the front face of the case (`07-hhkb-studio.md`, "Using the gesture pads") |
| Studio: Space pastes the clipboard, other keys act strangely | Probably the Fn2 layer is stuck: Fn2+Space is the middle mouse button, which pastes in many apps (not confirmed) | Tap Fn2. If that doesn't help, turn the Studio off and on |
| Studio's HHKB-key position types `` ` `` | The key isn't set to F18 in the Studio's keymap tool (older setup), and the Karabiner `` ` `` remap for the Studio is gone | Set the key to F18 on every layer in the HHKB Studio Keymap Tool (`08-migrate-studio-f18.md`) |
| HHKB key works on one HHKB but not the other | Only one model has a device entry | Add the missing product ID to `hhkb_pids` and rerun the merge (`03-karabiner.md`) |
| Option+Tab does nothing | Shortcut not applied yet | Set it in the GUI once, or log out/in after the `defaults write` |
| MacBook's own US/ISO keyboard types wrong symbols after setup | Karabiner grabs the built-in keyboard and sends it through the JIS virtual keyboard | Set the built-in keyboard's device entry to `ignore: true` (`03-karabiner.md`, "Non-JIS built-in keyboard"). Don't switch the virtual keyboard back to ANSI |
