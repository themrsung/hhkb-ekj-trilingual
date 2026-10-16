# Windows 02 — Windows 11 settings

Checked against Windows 11 25H2 (build 26200). Every step here changes a system setting:
👤 ask the user first, and back up what you change.

## 1. Survey first (read-only)

```powershell
Get-WinUserLanguageList | ForEach-Object { "$($_.LanguageTag): $($_.InputMethodTips -join ', ')" }
Get-ItemProperty 'HKLM:\SYSTEM\CurrentControlSet\Services\i8042prt\Parameters' |
  Select-Object 'LayerDriver JPN', 'LayerDriver KOR', OverrideKeyboardIdentifier, OverrideKeyboardType, OverrideKeyboardSubtype
Get-PnpDevice -PresentOnly | Where-Object InstanceId -match 'VID_04FE'
```

## 2. Languages: Korean + Japanese

Two languages, two IMEs, nothing else:

| Language | Keyboard | Input method tip | Used by |
| --- | --- | --- | --- |
| Korean (`ko`) | Microsoft IME (Korean) | `0412:{A028AE76-01B1-46C2-99C4-ACD9858AE02F}{B5FE1F02-D5F2-4445-9C03-C568F23C99A1}` | F18 |
| Japanese (`ja`) | Microsoft IME (Japanese) | `0411:{03B5835F-F03C-411B-9CE2-AA23E1171E36}{A76C93D9-5523-4E90-AAFA-4DB112F9AC76}` | F19 (IME off), F20 (IME on) |

Add Japanese without removing Korean (back up the list first). Or use Settings → Time &
language → Language & region → Add a language → 日本語.

```powershell
$backup = Get-WinUserLanguageList
$backup | ForEach-Object { "$($_.LanguageTag): $($_.InputMethodTips -join ', ')" } |
  Set-Content "$env:USERPROFILE\languagelist.bak-$(Get-Date -f yyyyMMddHHmmss).txt"
$list = Get-WinUserLanguageList
$list.Add('ja-JP')
Set-WinUserLanguageList $list -Force
```

## 3. Hardware keyboard layout: Japanese 106/109

Settings → Time & language → Language & region → 日本語 → Language options → **Change layout** →
**Japanese keyboard (106/109 key)** → restart. Windows keeps **one** hardware layout for all
keyboards, so every other keyboard on the PC (laptop, other USB boards) is read as JIS too.

Check after the restart: `LayerDriver JPN = kbd106.dll`, `OverrideKeyboardType = 7`,
`OverrideKeyboardSubtype = 2`.

**JIS applies only while Japanese is the active language.** Measured 2026-10-17 after forcing
106/109 and restarting. The hardware layout is chosen **per language**, through one
`LayerDriver <LANG>` value each under `HKLM\SYSTEM\CurrentControlSet\Services\i8042prt\Parameters`:

| Value | Set to | Effect |
| --- | --- | --- |
| `LayerDriver JPN` | `kbd106.dll` | Japanese (F19 English, F20 kana) types the JIS keycap symbols |
| `LayerDriver KOR` | `kbd101a.dll` (Windows' default for Korean) | Korean (F18) types **US ANSI symbols** (for example, Shift+2 gives `@` instead of the keycap's `"`) |

The good part: Profile 4's English is "Japanese with the IME off", so English typing gets JIS
symbols. The bad part: while Korean is active, the HHKB's symbol keys don't match their keycaps.
The opposite applies to a US built-in keyboard: right in Korean, wrong in Japanese and English.
Hangul itself is unaffected, because the letter keys sit in the same places on both layouts.

Options for Korean, none tested yet:

1. **Leave it as is.** Type Hangul in Korean and switch to F19 for symbols. No changes, no risk.
2. **Point Korean at the JIS layout too:** set `LayerDriver KOR` to `kbd106.dll` (admin, restart;
   back up the value first). Korean would then use JIS symbols as well, so the HHKB is consistent
   everywhere. Costs: the built-in US keyboard becomes JIS in every language. The Korean IME on a
   JIS layout isn't a configuration Microsoft documents, so check 한/영, 한자 and Hangul input
   after the change.
3. **Per-keyboard remapping** with a driver that tells keyboards apart: AutoHotInterception, on the
   Interception kernel driver. AutoHotkey would rewrite the HHKB's symbol keys only while Korean is
   active, leaving the built-in keyboard alone. Costs: a third-party kernel driver, and it may
   conflict with keyboard-security software (common on Korean PCs for online banking).

Not possible: plain AutoHotkey can't tell which keyboard a key came from, so it can't remap only
the HHKB. The HHKB keymap tool maps keys, not Shift+key pairs, so it can't fix the shifted
symbols either.

**Don't pick "Detect connected keyboard"** (offered next to the 106/109 and 101/102 choices). It
does not detect the HHKB as JIS. Tested on 25H2 build 26200 (2026-10-17), HHKB Studio on USB,
Profile 4: after choosing it and restarting, the HHKB typed **US ANSI** symbols. In the registry,
choosing it **removes** `LayerDriver JPN` and all the `Override*` values, so Windows falls back to
its default (101-key) layout. So it doesn't give you a US built-in keyboard alongside a JIS
HHKB either: whatever it detects, the HHKB came out US. Choose **Japanese keyboard (106/109 key)** and restart.

## 4. Microsoft Japanese IME

Settings → Time & language → Language & region → 日本語 → Language options → Microsoft IME →
**General**:

- **かな入力 (Kana input): On.** F20 lands in hiragana with kana typing.

The IME's own key settings (無変換 / 変換 / 半角/全角) don't matter: the HHKB sends none of those
keys on Profile 4.

## 5. Microsoft Korean IME

Defaults. F18 forces 한 mode, so the 한/영 toggle is never needed. F17 sends 한자 (`VK_HANJA`) in Korean.

## 6. Input-language hotkeys

Leave the defaults (Win+Space). The HHKB keys don't depend on them.
