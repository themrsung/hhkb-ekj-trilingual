# Windows 04 — Path B (preview): FKeyIME

**FKeyIME** ([`themrsung/fkeyime`](https://github.com/themrsung/fkeyime), BSD-3-Clause) is one
Windows input method (a TSF text input processor) for English, Korean and Japanese, driven by the
same Profile 4 F-keys as Path A. It types every character from its own JIS table, so the symbol
keys match the JIS keycaps in **every** language, Korean included. Path A's compromise
(`03-autohotkey.md`, "The compromise") goes away.

> [!WARNING]
> **Preview.** FKeyIME has no installer or release yet. This page installs a development build
> from source, pinned to FKeyIME commit
> [`33209e2`](https://github.com/themrsung/fkeyime/tree/33209e2654fdaf933133502e85bb7e391f9817fb).
> Path A (`03-autohotkey.md`) stays the tested default. Tell the user, and ask before going on.

FKeyIME links on this page point at that commit. Read them there, not at FKeyIME's `main`.

## What it does

| Key (Profile 4) | FKeyIME | F17 in that mode |
| --- | --- | --- |
| F18 | Korean 2-Set, JIS symbols. Backspace removes one jamo | Nothing yet (no Hanja) |
| F19 | English, JIS keycap symbols macOS style: ろ `\` / `_`, ¥ `¥` / `\|` | Transient Caps Lock |
| F20 | Japanese, JIS kana input. F6 / F7 / F8 force hiragana / katakana / half-width katakana, like macOS. Space converts to kanji (bundled Mozc server, candidate window), Enter accepts, Escape / Backspace go back to kana | Transient romaji (not implemented yet: types the English table) |
| F16 | Japanese, romaji input (not implemented yet: types the English table). No key sends F16 on Profile 4 | Transient kana |

The mode is global: all windows switch together. Measured behavior and limits are in FKeyIME's
[`docs/spec.md`](https://github.com/themrsung/fkeyime/blob/33209e2654fdaf933133502e85bb7e391f9817fb/docs/spec.md),
sections 8 and 9.

## Dependencies (tell the user)

Path A needs AutoHotkey v2, a script from these docs and a logon task. Path B adds:

- **System-wide in-process COM DLLs** (x64 and x86) that Windows loads into every app that takes
  text input. Installing needs admin.
- **A conversion server**, `fkeyime_mozc_server.exe`: Google's Mozc (BSD-3-Clause), renamed and
  patched by FKeyIME (`third_party/patches/` there). About 23 MB of memory while running. It keeps
  what it learned from your conversions in `%LOCALAPPDATA%Low\FKeyIME Mozc`. **It runs with your
  normal user rights, without Mozc's sandbox** (FKeyIME spec, section 9, "Security").
- **Binaries built from a separate repo** instead of a script you can read here.
- **To build it:** Visual Studio 2022 Build Tools (C++ x64/x86 and ATL), the Windows SDK, CMake,
  Ninja, Python 3.12+, Bazelisk and the .NET 8 SDK. The first Mozc build downloads LLVM, MSYS2
  and Qt sources and takes about an hour and roughly 10 GB of disk.

## Install

Every step changes the system: 👤 ask the user first. Do Path A's hardware and Windows steps first
(`01-hardware.md`, `02-windows.md` step 3: the Japanese 106/109 layout). Path B uses them too.

1. **Get FKeyIME at the pinned commit:**

   ```powershell
   git clone https://github.com/themrsung/fkeyime.git
   git -C fkeyime checkout --detach 33209e2654fdaf933133502e85bb7e391f9817fb
   ```

2. **Toolchain:** `tools/bootstrap.ps1 -Install` installs what's missing with winget (UAC
   prompts) and sets up the repo. It's listed in FKeyIME's
   [`docs/toolchain.md`](https://github.com/themrsung/fkeyime/blob/33209e2654fdaf933133502e85bb7e391f9817fb/docs/toolchain.md).
3. **Build FKeyIME:** `tools/build.ps1` (x64 and x86, runs the unit tests), then
   `tools/build.ps1 -Config Release`.
4. **Build the conversion server:** `tools/fetch-mozc.ps1` (clones Mozc and applies FKeyIME's
   patches). Then follow the toolchain page's "Mozc" steps once, and run
   `tools/build-mozc-server.ps1`. Check it with `tools/test-mozc-server.ps1` (must say PASS).
5. **Install:** `tools/spike.ps1 -Install` (one UAC prompt). It registers both DLLs in
   `C:\Program Files\FKeyIME-spike`, adds **FKeyIME EN / KO / JA** to the language list next to
   the Microsoft IMEs (they stay, as a fallback), installs the server, and saves the language list
   for uninstalling.
6. **Windows setting:** Settings → Time & language → Typing → Advanced keyboard settings → "Let me
   use a different input method for each app window": **off**, so the mode stays global.
7. **AutoHotkey**, if Path A is set up (`Get-ScheduledTask hhkb-ekj-trilingual`): keep it with the
   FKeyIME-aware script in `03-autohotkey.md`, so Win+Space chooses between FKeyIME and the
   Microsoft IMEs, or stop it (`Disable-ScheduledTask hhkb-ekj-trilingual`). The old v1.5.x script
   takes F17–F20 before FKeyIME can see them. Fn1+P (Play/Pause) needs AutoHotkey on either path.
8. **Sign out and back in.** Apps that were running during the install may not use FKeyIME until
   they restart (`../../issues/windows-fkeyime-restart-apps.md`).
9. **Select FKeyIME** once with Win+Space (FKeyIME KO, EN or JA). From then on F16–F20 switch
   between its modes.

Coming from Path A on an existing PC: FKeyIME's
[`docs/migration-from-v1.5.2.md`](https://github.com/themrsung/fkeyime/blob/33209e2654fdaf933133502e85bb7e391f9817fb/docs/migration-from-v1.5.2.md)
has backups, the AutoHotkey options, and the rollback.

## Verify

In Notepad, with FKeyIME selected:

| Press | Expect |
| --- | --- |
| F19, then Shift+2, ろ, Shift+ろ, ¥, Shift+¥ | `"\_¥\|` |
| F19, F17, `a`, F17, `a` | `Aa` (Caps Lock on, then off) |
| F18, `dkssudgktpdy` Shift+2 | `안녕하세요"` (JIS symbol in Korean) |
| F18, `rhk` Backspace Backspace `l` Enter | `기` (Backspace removes one jamo) |
| F20, `3` `t` `@` Enter | `あが` |
| F20, `3` `e` F7 Enter | `アイ` |
| F20, `i` `-` `y` Space Enter | `日本` |
| F20, `i` `-` `y` Space Space | the candidate window opens; Escape goes back to `にほん` |
| AutoHotkey kept: Win+Space to a Microsoft IME, then F18 / F20 | Microsoft Korean / Japanese IME, as in Path A |

FKeyIME's test tools do the same by scan code: `build\x64\tools\spike\Release\fkeyime_probe.exe
--typing` (34 cases) and `--handoff 20` (the AutoHotkey hand-off).

## Uninstall

`tools/spike.ps1 -Uninstall` (UAC) unregisters FKeyIME, restores the saved language list and stops
the server. Re-enable Path A if it was disabled: `Enable-ScheduledTask hhkb-ekj-trilingual`.

## Known limits (preview)

- No Hanja (F17 in Korean does nothing), no romaji input, no mode indicator, no installer.
- Password fields, UAC prompts and the sign-in screen don't load input methods: F17–F20 do
  nothing there.
- Apps that read raw keys instead of text (games, some terminals) see Windows' layout, not
  FKeyIME's table.
- The first conversion after the server stopped starts it (about 100 ms). Store apps can't start
  it; they convert once another app has.
- Mozc learns the candidates you pick. Your Microsoft IME dictionary doesn't carry over.
- Elevated (admin) apps and Bluetooth: not tested with FKeyIME yet.
