# Windows 04 — JIS Korean: the Korean input method (Windows)

**JIS Korean is the default Korean input method on Windows too**, and F18's target. It replaces
the Microsoft Korean IME in the Korean language's keyboards. The macOS side is in
[`../09-jis-korean.md`](../09-jis-korean.md); what JIS Korean types is the same on both.

[JIS Korean](https://github.com/team-mirae/jis-korean) on Windows is a TSF text input processor
(`jiskorean.dll`). It reads **scan codes**, not Windows' keyboard layout, so in Mode A (JIS, the
default) the HHKB's symbol keys type JIS Korean's JIS legends while Korean is active, although
Windows' hardware layout for Korean stays `kbd101a.dll` (`02-windows.md`, step 3). That ends the
old compromise (Korean typing US ANSI symbols), except in ABC (below).

Pinned to JIS Korean commit **`90cc765`** (after `v0.2.0-preview`, not a tagged release). **Not
checked by this repo on Windows yet**: the macOS build at that commit was checked on macOS 27.0.1;
the Windows steps below follow JIS Korean's own `windows/README.md` at that commit. Measure each
row in [Checks](#6-checks) before calling it done.

## 1. Build and install

Built from source (there's no signed installer). Needs Visual Studio 2022 Build Tools with the C++
x64/x86 tools, CMake 3.28+, Ninja, and Rust (rustup) with both MSVC targets. 👤 Ask the user before
installing any of them:

```powershell
winget install --id Microsoft.VisualStudio.2022.BuildTools -e --override "--quiet --wait --add Microsoft.VisualStudio.Workload.VCTools --includeRecommended"
winget install --id Kitware.CMake -e
winget install --id Ninja-build.Ninja -e
winget install --id Rustlang.Rustup -e
rustup default stable
rustup target add x86_64-pc-windows-msvc i686-pc-windows-msvc

git clone https://github.com/team-mirae/jis-korean.git
git -C jis-korean checkout 90cc765
jis-korean\windows\tools\build.ps1      # x64 + x86 (32-bit apps load the x86 DLL), runs the shell tests
jis-korean\windows\tools\install.ps1    # UAC: Program Files\JIS Korean, registers x64 + x86, adds JIS Korean to Korean
```

Open a new terminal after installing the tools, so `cargo`, `cmake` and `ninja` are on PATH.

- `install.ps1` adds JIS Korean to the Korean language's keyboards and leaves everything else in
  the language list alone (the Microsoft Korean IME stays; step 2 removes it).
- **Restart apps after installing or updating.** A running app keeps the old DLL, or none, until
  it restarts.
- Updating later: rebuild, then `install.ps1 -Update` (no UAC). Don't uninstall first.

## 2. Korean's keyboards: JIS Korean only

👤 Ask first: the user may type with the Microsoft Korean IME today. Then make JIS Korean the only
keyboard under Korean. F18 selects the Korean **language** (`03-autohotkey.md`), and Windows then
activates that language's keyboard, so a second Korean keyboard can make F18 land on the wrong one.
Win+Space also lists one entry fewer.

```powershell
$list = Get-WinUserLanguageList
$list | ForEach-Object { "$($_.LanguageTag): $($_.InputMethodTips -join ', ')" } |
  Set-Content "$env:USERPROFILE\languagelist.bak-$(Get-Date -f yyyyMMddHHmmss).txt"
$jk = '0412:{D10651AD-51EA-4DBA-8BFF-1905261B26A1}{669E0E87-FA58-46F6-96E1-0890A0224404}'
$ko = $list | Where-Object LanguageTag -eq 'ko'
if ($ko.InputMethodTips -contains $jk) {
    foreach ($t in @($ko.InputMethodTips)) { if ($t -ne $jk) { [void]$ko.InputMethodTips.Remove($t) } }
    Set-WinUserLanguageList $list -Force
}
Get-WinUserLanguageList | ForEach-Object { "$($_.LanguageTag): $($_.InputMethodTips -join ', ')" }
```

Expect `ko: 0412:{D10651AD-…}{669E0E87-…}` and `ja: 0411:{03B5835F-…}{A76C93D9-…}`. Or in
Settings → Time & language → Language & region → 한국어 → Language options → Keyboards: keep
**JIS Korean**, remove **Microsoft IME**.

## 3. Mode A (JIS)

Mode A is the default. Check it: `Get-ItemProperty 'HKCU:\Software\JIS Korean' Mode` is `0` or
missing. Otherwise right-click the 한글 / ABC indicator in the taskbar → **Mode A (JIS)**. It's one
setting for every app. Don't use Mode B (US ANSI) or Mode C (UK ISO) with the HHKB.

## 4. 한/영: why the setup doesn't use it

JIS Korean has a 한/영 key that works exactly like the Microsoft IME's, but JIS Korean's own notes
(`windows/README.md`, "한/영 and English"; spec §8.4) say what it is and isn't. Plan around these:

- **It's a plain on/off switch.** 한/영 commits what's unconfirmed and then hands every key to the
  app (**ABC**). It can't turn the last word into English, and pressing it mid-word splits the word.
- **One state for the whole session.** 한글 / ABC is shared by every app and every field, isn't
  stored, and starts on 한글 after a restart (or a crash). Switching to ABC in one window switches
  every window.
- **English typed in ABC is invisible to JIS Korean**: no predictions, no F9 / F10, no hanja, no
  learning.
- **ABC types Windows' Korean layout (`kbd101a.dll`)**, so the HHKB's symbol keys type **US ANSI**
  symbols there, not the JIS keycaps. (Mode A only covers keys that reach JIS Korean.)
- **It ignores outside open/close requests.** When an app or script closes the IME
  (`ImmSetOpenStatus`, `IMC_SETOPENSTATUS`), JIS Korean puts its state back. Only 한/영
  (`VK_HANGUL`), a click on the indicator, or its menu changes it. So AutoHotkey can't *set*
  한글; it can only press 한/영 when the state is ABC.

What this repo does instead:

| You want | Use | Why |
| --- | --- | --- |
| Korean, or Korean with English words mixed in | **F18** (JIS Korean, 한글) | JIS Korean's recommended way: stay in it and let the predictions read English words (`hello` shows as hello, `server를` as server를). Shift+Z (F10) forces the English letters of a word |
| English-only text, code, commands, passwords, games | **F19** (English: Japanese with the IME off) | JIS symbols that match the keycaps, no composition at all. Takes the place 한/영 has in JIS Korean's notes |
| Japanese | **F20** | |

So Profile 4 has no 한/영 key, and nothing in the setup puts JIS Korean in ABC. If it ends up in
ABC anyway (a click on the indicator, a laptop's own 한/영 key), F18 switches it back to 한글
(`03-autohotkey.md`), for every app at once.

Unmarked password boxes (banking and certificate programs often draw their own) can't be detected
and compose like text. Switch to **F19** there, or right-click the indicator → **Off in
<program>**, which passes every key through in that program only.

**Train the predictions on your English words** (optional): list them in
`%APPDATA%\JIS Korean\training.json` (indicator menu → Learned Weights → Open Training Folder), in
the format of JIS Korean's `docs/training-contract.md`. Its README's `.co.kr` example is a good
first file. It's applied the next time JIS Korean activates and stays on this PC.

## 5. F-keys on the Studio

JIS Korean uses F6–F10 (F6 commit, F7 hanja, F8 a temporary escape to standard 2-Set, F9 / F10
full- and half-width letters). On Profile 4 they're in the stock **Fn1** layer (Fn1 + number row).
F17 (the right Command position) sends **F7** in Korean (`03-autohotkey.md`), so hanja has its own
key as before. Browsers take F10 for their menu: use Shift+Z there.

## 6. Checks

👤 rows need the user. Type in Notepad.

| # | Action | Expected |
| --- | --- | --- |
| W1 | `Get-WinUserLanguageList` | Korean has only JIS Korean; Japanese has the Microsoft IME |
| W2 | 👤 F18, then `gksrmf` | 한글 (the taskbar shows 한글) |
| W3 | 👤 F18, then `1`, `k` | 빠 (dedicated ㅃ); Shift+`1` types `1` |
| W4 | 👤 F18, then ろ / Shift+ろ, ¥, `^` | `\` / `_`, `—`, `(`: JIS Korean's Mode A legends, not US ANSI |
| W5 | 👤 F18, `gkrry`, then F17 | the hanja list for 학교 (學校 with `1`) |
| W6 | 👤 Click the indicator so it shows ABC, then F18, then `gks` | 한 (F18 pressed 한/영 back to 한글) |
| W7 | 👤 F18 while already in 한글, then `gks` | 한: F18 never toggles to ABC. **If it alternates 한글 / ABC, stop**: the open-status read isn't reliable here (`03-autohotkey.md`, "What is documented, inferred, or still to be measured") |
| W8 | 👤 F19, then Shift+2 | `"` (JIS) |
| W9 | 👤 F18 in one app, F19, switch to another app, F18 | 한글 in both |

## Known limits

- Built from source, not signed. Program Files install, registered for x64 and x86.
- Password fields and the secure desktop (UAC, sign-in) don't use input methods.
- Games that read raw keys see Windows' own layout, not JIS Korean.
- Store apps running sandboxed may not read JIS Korean's settings (JIS Korean: not measured).
- The ~1% lost switch on macOS (`../04-hammerspoon.md`) is a macOS bug; it hasn't been looked for
  on Windows.
