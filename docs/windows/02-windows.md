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
| Korean (`ko`) | **JIS Korean** (`04-jis-korean.md`), the only Korean keyboard | `0412:{D10651AD-51EA-4DBA-8BFF-1905261B26A1}{669E0E87-FA58-46F6-96E1-0890A0224404}` | F18 |
| Japanese (`ja`) | Microsoft IME (Japanese) | `0411:{03B5835F-F03C-411B-9CE2-AA23E1171E36}{A76C93D9-5523-4E90-AAFA-4DB112F9AC76}` | F19 (IME off), F20 (IME on) |

Add Japanese without removing Korean (back up the list first). Or use Settings → Time &
language → Language & region → Add a language → 日本語. JIS Korean's installer adds it to Korean;
then remove the Microsoft Korean IME (`0412:{A028AE76-01B1-46C2-99C4-ACD9858AE02F}{B5FE1F02-D5F2-4445-9C03-C568F23C99A1}`)
from Korean, as `04-jis-korean.md` step 2 does.

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
| `LayerDriver KOR` | `kbd101a.dll` (Windows' default for Korean) | Keys that reach Windows' Korean layout type **US ANSI symbols** (for example, Shift+2 gives `@` instead of the keycap's `"`) |

Profile 4's English is "Japanese with the IME off", so English typing gets JIS symbols. Korean is
**JIS Korean**, which reads scan codes and types its Mode A (JIS) legends itself, so the Korean
layout's US ANSI symbols don't apply to it. They still show in two places:

- **JIS Korean's ABC state** (한/영 off): keys go to Windows' Korean layout untouched. The setup
  never uses ABC (`04-jis-korean.md`, "한/영"), and F18 switches it back to 한글.
- **The Microsoft Korean IME**, if it's still enabled next to JIS Korean. Remove it
  (`04-jis-korean.md`, step 2).

Leave `LayerDriver KOR` at `kbd101a.dll`. Through `v1.8.0` this section offered pointing Korean at
the JIS layout, or a per-keyboard driver, to work around the Microsoft Korean IME's US symbols;
with JIS Korean neither is needed, and neither was tested.

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

## 5. Korean: JIS Korean

Install and set it up with `04-jis-korean.md`: build, install, make it the only Korean keyboard,
Mode A. F18 selects it in 한글, so the 한/영 toggle is never needed; F17 sends F7 (hanja) in Korean.

## 6. Input-language hotkeys

Leave the defaults (Win+Space). The HHKB keys don't depend on them.
