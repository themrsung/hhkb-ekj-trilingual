# Windows 01 — HHKB Studio JIS, keymap Profile 4

The Windows setup uses the same HHKB Studio JIS as the Mac (`docs/07-hhkb-studio.md`), but a
**different keymap profile**: the Mac keeps **Profile1** (native 英数 / かな), Windows uses
**Profile 4**. Both profiles live on the keyboard, so neither OS needs the other's keymap.

| Property | Value |
| --- | --- |
| USB vendor / product | `04FE` / `0015` (PFU Limited, HHKB Studio) |
| Windows sees (USB) | USB composite device: HID keyboard (`MI_00`), mouse (`MI_02`), consumer + system control (`MI_03`), two vendor-defined interfaces |
| DIP switches | All OFF (same as the Mac) |
| Keymap profile | **Profile 4** |

## Bottom row on Profile 4

Written with the HHKB Studio Keymap Tool (on another computer is fine; the keymap is stored on the
keyboard). Set the four F keys on **every layer** (Standard, Fn1, Fn2, Fn3), as the Mac doc does
for F18, so holding Fn1 or Fn2 never changes what they send.

```
Fn1 | F18 | Alt | Win | F19 | Space | F20 | F17 | RAlt | Fn1 | ← | ↓ | →
```

| # | Position (Mac Profile1 role) | Profile 4 | Windows meaning |
| --- | --- | --- | --- |
| 1 | Fn1 | Fn1 | HHKB Fn layer |
| 2 | 半/全 (F18 / HHKB key) | **F18** | → Korean, 한 (`03-autohotkey.md`) |
| 3 | Option | Alt | Left Alt |
| 4 | Command | Win | Left Windows |
| 5 | 英数 | **F19** | → English (Japanese IME off) |
| 6 | Space | Space | |
| 7 | かな | **F20** | → Japanese hiragana, kana typing |
| 8 | Command | **F17** | Context action: 한자 in Korean, nothing elsewhere yet |
| 9 | Option | Right Alt | |
| 10 | Fn1 | Fn1 | HHKB Fn layer |
| 11–13 | ← ↓ → | ← ↓ → | |

Fn-layer keys used (stock Fn1 layer): **Fn1+P → Pause** (→ Media Play/Pause in AutoHotkey),
**Fn1+I → Print Screen** (behavior on Windows: not decided yet).

## Measure, don't assume

Expected virtual-key codes: F17 `0x80`, F18 `0x81`, F19 `0x82`, F20 `0x83` (scan codes `0x68`–`0x6B`).
Check them with AutoHotkey's **KeyHistory** (tray icon → Open → View → Key history) after pressing
each bottom-row key, plain and with Fn1 held.

Measured on the owner's board: *pending.*
