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
