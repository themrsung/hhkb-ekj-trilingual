# Windows 03 — Path A (default): AutoHotkey v2, one-way language keys

> [!NOTE]
> Path B, FKeyIME (preview), is the alternative: JIS symbols in every language, but it installs
> dependencies. See `04-fkeyime.md` and AGENTS.md section 6 for the trade-offs.

The Windows counterpart of `docs/04-hammerspoon.md`. The HHKB Studio (keymap **Profile 4**) sends
**F18 / F19 / F20 / F17** from its own firmware (`01-hardware.md`). AutoHotkey catches those keys
and does all the switching. Windows itself never sees an IME key from the bottom row, so the IMEs'
own toggle behavior (半角/全角, 無変換, 한/영) can't fight the script.

| Key | Goes to | How |
| --- | --- | --- |
| **F18** | Korean, **한** mode | switch language to Korean (`0x0412`), IME open, conversion mode `HANGUL` |
| **F19** | English alphanumeric | switch language to Japanese (`0x0411`), **IME closed** |
| **F20** | Japanese hiragana, **kana typing** | switch language to Japanese, IME open, conversion mode `NATIVE \| FULLSHAPE` (no `ROMAN` bit) |
| **F17** | Context action | Korean: 한자 (sends `VK_HANJA`). Other languages: nothing yet |
| **Pause** (Fn1+P) | Media Play/Pause | |

English is "Japanese IME with the IME off", not a separate English layout. That keeps every
symbol key matching the JIS legends (the Windows US layout would type US symbols on JIS keys) and
keeps ろ native (`\` / `_`). The cost: the taskbar shows **JA** (or **A**) while you type English.

### The compromise (owner's default)

| Language | Key | Handled by | Symbol keys |
| --- | --- | --- | --- |
| Korean | F18 | Hand off to the **Microsoft Korean IME** (한 mode) | **US ANSI**: they don't match the JIS keycaps |
| English | F19 | **Japanese with the IME off** | **JIS**: they match the keycaps |
| Japanese | F20 | **Microsoft Japanese IME**, kana typing | **JIS** |

Why: the Microsoft Korean IME always uses US ANSI symbol mappings. Windows picks the hardware
layout per language, and Korean's is `kbd101a.dll` (`02-windows.md`, step 3). English and
Japanese both run on the Japanese language, so they get the JIS layout and every symbol matches
its keycap. The repo owner types English by default and uses Korean only when needed, mostly for
Hangul, so the US symbols in Korean cost little. For symbols, switch to English with F19.

**Results may vary with typing habits.** Someone who types many symbols in Korean may prefer one
of the other options in `02-windows.md`, step 3. Adjust it for the end user as they prefer.

## Two levels of switching

1. **Language** (Korean vs Japanese): `WM_INPUTLANGCHANGEREQUEST` posted to the focused window with
   the target layout. Documented: the message asks the window to change its input language, and
   `DefWindowProc` normally accepts it.
2. **IME state inside the language** (Japanese on/off and hiragana; Korean 한/영):
   `ImmGetDefaultIMEWnd` + `WM_IME_CONTROL` with `IMC_SETOPENSTATUS` / `IMC_SETCONVERSIONMODE`.
   Widely used (it's what most "IME status" AHK snippets do), but **not formally documented for
   cross-process use**. It's the part to measure.

The language change is asynchronous, so the script waits (up to 150 ms) until the focused thread
reports the new language before touching the IME state. Otherwise the IME command would go to the
old IME.

## Rules carried over from the Mac (don't "simplify" these away)

1. **Always apply.** No early return when the target already looks active. Pressing F18 in Korean
   still forces 한 mode, so the key is never dead and never toggles out (`04-hammerspoon.md`, rule 1).
2. **Retry is guarded by a check.** 150 ms later the script reads language, open status and
   conversion mode back, and re-applies **once**, only if something is wrong.
3. **Bare keys only.** `F18::` etc. have no modifiers; Shift+F18 does nothing.

## Install

👤 needs the user's OK: `winget install --id AutoHotkey.AutoHotkey -e` (AutoHotkey **v2**).

Then save the script below as `%USERPROFILE%\Documents\AutoHotkey\hhkb-ekj-trilingual.ahk` (back up
any existing file first: `Copy-Item f "f.bak-$(Get-Date -f yyyyMMddHHmmss)"`). Then start it at
sign-in with a **Task Scheduler logon task**. Creating it needs admin (one UAC prompt), but the
task itself runs as the user, **not** elevated:

```powershell
# Run in an elevated PowerShell. $user must be the account that signs in (DOMAIN\name).
$user   = "$env:USERDOMAIN\$env:USERNAME"
$script = "$env:USERPROFILE\Documents\AutoHotkey\hhkb-ekj-trilingual.ahk"
$a = New-ScheduledTaskAction -Execute 'C:\Program Files\AutoHotkey\v2\AutoHotkey64.exe' -Argument "`"$script`""
$t = New-ScheduledTaskTrigger -AtLogOn -User $user
$p = New-ScheduledTaskPrincipal -UserId $user -LogonType Interactive -RunLevel Limited
$s = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries `
       -ExecutionTimeLimit ([TimeSpan]::Zero) -MultipleInstances IgnoreNew
Register-ScheduledTask -TaskName 'hhkb-ekj-trilingual' -Action $a -Trigger $t -Principal $p -Settings $s -Force
```

The settings matter. By default a task doesn't start on battery, and it's stopped after 3 days.
`-RunLevel Limited` keeps AutoHotkey at the same level as normal apps. Start it right away with
`Start-ScheduledTask hhkb-ekj-trilingual`. `LastTaskResult` `267009` (`0x41301`) means it's running.
Remove it with `Unregister-ScheduledTask hhkb-ekj-trilingual`.

**Don't use `shell:startup` or a Run key.** On the owner's PC (`01-hardware.md`, "Tested on"),
neither ran AutoHotkey at sign-in, over three restarts. The Shell-Core event log
(`Microsoft-Windows-Shell-Core/Operational`, events 9705–9708) showed Windows working through
every other HKCU Run entry. It skipped this one without even logging an attempt, although the
value was present and readable. It also logged no startup-folder entries at all. The cause wasn't
found, and the logon task avoids both mechanisms. **Verified** on the owner's PC (2026-10-17):
after a restart, the task started AutoHotkey and the F keys worked. On a new PC, check that the
green **H** tray icon is there after a restart.

## `hhkb-ekj-trilingual.ahk`

**FKeyIME-aware** (since 2.0.0-preview). Without FKeyIME installed it behaves exactly like the
v1.5.x script. With FKeyIME (Path B) installed too, **Win+Space decides**:

- FKeyIME selected: the script never takes F16–F20. A window that isn't on FKeyIME yet is moved
  to it, and the key is sent again for FKeyIME.
- A Microsoft IME selected: the switching below, as in v1.5.x. If a language switch lands on
  FKeyIME's profile for that language, the script selects the Microsoft IME instead.

It asks FKeyIME through its public API (`FKeyImeQueryWindow` in `fkeyime.dll`, found through
FKeyIME's CLSID; [`docs/api.md`](https://github.com/themrsung/fkeyime/blob/33209e2654fdaf933133502e85bb7e391f9817fb/docs/api.md)). The script is FKeyIME's
[`examples/autohotkey/hhkb-ekj-trilingual.ahk`](https://github.com/themrsung/fkeyime/blob/33209e2654fdaf933133502e85bb7e391f9817fb/examples/autohotkey/hhkb-ekj-trilingual.ahk)
at the pinned commit.

```autohotkey
#Requires AutoHotkey v2.0
#SingleInstance Force
; hhkb-ekj-trilingual (Windows): one-way language keys sent by the HHKB Studio, keymap Profile 4.
;   F18 -> Korean (한)   F19 -> English (Japanese IME off)   F20 -> Japanese hiragana, kana typing
;   F17 -> 한자 in Korean, nothing elsewhere   Pause (Fn1+P) -> Media Play/Pause
; FKeyIME-aware (https://github.com/themrsung/fkeyime docs/api.md). Win+Space picks who handles
; the language keys:
;   - FKeyIME is the session's input method: this script steps aside. A window that isn't on
;     FKeyIME yet is moved to it, and the key is sent again for FKeyIME to handle.
;   - A Microsoft IME is the session's input method: the keys work as in v1.5.2.
; Without FKeyIME installed, this behaves exactly like the hhkb-ekj-trilingual v1.5.2 script.

HKL_KO := 0x04120412, LANG_KO := 0x0412
HKL_JA := 0x04110411, LANG_JA := 0x0411
LANG_EN := 0x0409

; IME conversion-mode bits (imm.h)
CM_NATIVE := 0x01, CM_KATAKANA := 0x02, CM_FULLSHAPE := 0x08, CM_ROMAN := 0x10

; Targets: language, IME open status, conversion mode to set, and the bits that must match.
KOREAN   := {hkl: HKL_KO, lang: LANG_KO, open: 1, cmode: CM_NATIVE, mask: CM_NATIVE}
ENGLISH  := {hkl: HKL_JA, lang: LANG_JA, open: 0}
JAPANESE := {hkl: HKL_JA, lang: LANG_JA, open: 1, cmode: CM_NATIVE | CM_FULLSHAPE
           , mask: CM_NATIVE | CM_KATAKANA | CM_FULLSHAPE | CM_ROMAN}

FKEYIME_CLSID := "{66749D2A-1770-43D3-A269-A051B5BC1AAA}"
FKEYIME_EN := "{8F61A2C0-75BC-4042-B739-CC7D63B542E5}"
FKEYIME_KO := "{28C201BF-390E-482F-8259-CD26E05844A9}"
FKEYIME_JA := "{1AC13F3E-07AC-4B03-A606-967BD48BFD99}"
; The Microsoft IMEs these targets mean (language, CLSID, profile), for UseMicrosoftIme().
KOREAN.tip   := [LANG_KO, "{A028AE76-01B1-46C2-99C4-ACD9858AE02F}", "{B5FE1F02-D5F2-4445-9C03-C568F23C99A1}"]
ENGLISH.tip  := [LANG_JA, "{03B5835F-F03C-411B-9CE2-AA23E1171E36}", "{A76C93D9-5523-4E90-AAFA-4DB112F9AC76}"]
JAPANESE.tip := ENGLISH.tip

; FKeyIME chosen, but this window isn't on it yet: move it there and let FKeyIME have the key.
#HotIf !FKeyImeActive() && FKeyImeSession()
F16::ToFKeyIme("F16", LANG_JA, FKEYIME_JA)
F18::ToFKeyIme("F18", LANG_KO, FKEYIME_KO)
F19::ToFKeyIme("F19", LANG_EN, FKEYIME_EN)
F20::ToFKeyIme("F20", LANG_JA, FKEYIME_JA)
; A Microsoft IME chosen: v1.5.2's switching.
#HotIf !FKeyImeActive() && !FKeyImeSession()
F18::SwitchTo(KOREAN)
F19::SwitchTo(ENGLISH)
F20::SwitchTo(JAPANESE)
F17::ContextAction()
#HotIf
Pause::Send("{Media_Play_Pause}")

; True while FKeyIME is the input method of the focused window. The DLL is found through FKeyIME's
; permanent CLSID (32-bit AutoHotkey gets the x86 DLL through registry redirection) and loaded
; only for the call, so it never stays locked.
FKeyImeActive() {
    try dll := RegRead("HKLM\SOFTWARE\Classes\CLSID\" FKEYIME_CLSID "\InprocServer32")
    catch
        return false                            ; FKeyIME isn't installed
    try return DllCall(dll "\FKeyImeQueryWindow", "ptr", FocusHwnd(), "int") >= 0
    catch
        return false                            ; DLL missing or without the API: act as before
}

; ITfInputProcessorProfileMgr, for reading and choosing the session's input method.
ProfileMgr() {
    static mgr := ComObject("{33C53A50-F456-4884-B049-85FD643ECFED}", "{71C6E74C-0F28-11D8-A82A-00065B84435C}")
    return mgr
}

Guid(text) {
    g := Buffer(16)
    DllCall("ole32\CLSIDFromString", "str", text, "ptr", g)
    return g
}

; True when the session's input method (what Win+Space last chose) is FKeyIME. This thread has no
; text input of its own, so it sees the session's choice (measured: it follows session changes).
FKeyImeSession() {
    static keyboard := Guid("{34745C63-B2F0-4784-8B67-5E12C8701A31}")  ; GUID_TFCAT_TIP_KEYBOARD
    static fkeyime := Guid(FKEYIME_CLSID)
    profile := Buffer(96, 0)                    ; TF_INPUTPROCESSORPROFILE; clsid at offset 8
    try ComCall(10, ProfileMgr(), "ptr", keyboard, "ptr", profile)  ; GetActiveProfile
    catch
        return false
    return DllCall("ntdll\RtlCompareMemory", "ptr", profile.Ptr + 8, "ptr", fkeyime, "uptr", 16
                 , "uptr") = 16
}

; Selects one of FKeyIME's profiles for the whole session (which moves this window too), waits for
; FKeyIME to be active here, and sends the key again so FKeyIME applies its own rules to it.
ToFKeyIme(key, lang, profile) {
    try ComCall(3, ProfileMgr(), "uint", 1, "ushort", lang, "ptr", Guid(FKEYIME_CLSID)  ; ActivateProfile
              , "ptr", Guid(profile), "ptr", 0, "uint", 0x20000004)  ; FORSESSION | DONTCARECURRENTINPUTLANGUAGE
    deadline := A_TickCount + 300
    while !FKeyImeActive() && A_TickCount < deadline
        Sleep(10)
    if FKeyImeActive()
        SendEvent("{" key "}")
}

; A language change picks that language's last-used input method, which can be FKeyIME's. Select
; the Microsoft IME explicitly instead (ITfInputProcessorProfileMgr::ActivateProfile, whole session).
UseMicrosoftIme(t) {
    try ComCall(3, ProfileMgr(), "uint", 1, "ushort", t.tip[1], "ptr", Guid(t.tip[2])
              , "ptr", Guid(t.tip[3]), "ptr", 0, "uint", 0x20000000)  ; FORSESSION
}

SwitchTo(t) {
    hwnd := FocusHwnd()
    if !hwnd
        return
    ; Always apply, even if t already looks active: a stale state must not leave the key dead.
    Apply(hwnd, t)
    ; Verify once, and retry only if the switch didn't take.
    SetTimer(() => (IsAt(hwnd, t) || Apply(hwnd, t)), -150)
}

Apply(hwnd, t) {
    DllCall("PostMessageW", "ptr", hwnd, "uint", 0x50, "ptr", 0, "ptr", t.hkl)  ; WM_INPUTLANGCHANGEREQUEST
    deadline := A_TickCount + 150
    while LangOf(hwnd) != t.lang && A_TickCount < deadline
        Sleep(10)
    if FKeyImeActive() {                        ; landed in FKeyIME's profile for this language
        UseMicrosoftIme(t)
        deadline := A_TickCount + 150
        while FKeyImeActive() && A_TickCount < deadline
            Sleep(10)
    }
    ImeCtl(hwnd, 0x006, t.open)                 ; IMC_SETOPENSTATUS
    if t.open
        ImeCtl(hwnd, 0x002, t.cmode)            ; IMC_SETCONVERSIONMODE
}

IsAt(hwnd, t) {
    if FKeyImeActive() || LangOf(hwnd) != t.lang || ImeCtl(hwnd, 0x005) != t.open   ; IMC_GETOPENSTATUS
        return false
    return !t.open || (ImeCtl(hwnd, 0x001) & t.mask) = t.cmode    ; IMC_GETCONVERSIONMODE
}

ContextAction() {
    hwnd := FocusHwnd()
    if hwnd && LangOf(hwnd) = LANG_KO
        Send("{vk19}")                          ; VK_HANJA: Korean IME's 한자 conversion
    ; Other languages: nothing yet.
}

; The window that has keyboard focus (a child control, if the app uses them).
FocusHwnd() {
    if !(active := WinExist("A"))
        return 0
    tid := DllCall("GetWindowThreadProcessId", "ptr", active, "ptr", 0, "uint")
    gti := Buffer(A_PtrSize = 8 ? 72 : 48, 0)   ; GUITHREADINFO
    NumPut("uint", gti.Size, gti, 0)
    if DllCall("GetGUIThreadInfo", "uint", tid, "ptr", gti)
        && (focus := NumGet(gti, A_PtrSize = 8 ? 16 : 12, "ptr"))
        return focus
    return active
}

LangOf(hwnd) {
    tid := DllCall("GetWindowThreadProcessId", "ptr", hwnd, "ptr", 0, "uint")
    return DllCall("GetKeyboardLayout", "uint", tid, "ptr") & 0xFFFF
}

ImeCtl(hwnd, cmd, value := 0) {
    if !(ime := DllCall("imm32\ImmGetDefaultIMEWnd", "ptr", hwnd, "ptr"))
        return -1
    result := 0
    DllCall("SendMessageTimeoutW", "ptr", ime, "uint", 0x283, "ptr", cmd, "ptr", value  ; WM_IME_CONTROL
          , "uint", 0x2, "uint", 200, "ptr*", &result)                                  ; SMTO_ABORTIFHUNG
    return result
}
```

## Measured: IME keys (2026-10-17)

AutoHotkey **KeyHistory** (keyboard hook installed), HHKB over **USB**, on the PC in
`01-hardware.md` ("Tested on"). The JIS hardware layout (`02-windows.md`, step 3) was set in the
registry but **not active yet** (no restart since it was set), and the Korean layout file is
`kbd101a.dll`. The HHKB was on Profile3 while it still had PFU's stock Windows (JIS) keycodes, before its 変換 key
was changed to `LANG1` (`01-hardware.md`), not on Profile 4.

| Key pressed | VK | SC | Meaning |
| --- | --- | --- | --- |
| Laptop's built-in **한/영** | `15` | `138` | `VK_HANGUL`. SC `138` is the extended `E0 38` code (the Right Alt position), so 한/영 rides on Right Alt's scan code |
| HHKB **半/全** | `C0` | `029` | Plain `` ` `` (`VK_OEM_3`); no IME meaning in this layout |
| HHKB **無変換** | `EB` | `07B` | No standard VK name; `NONCONVERT` (`1D`) would need the JIS layout |
| HHKB **変換** | `FF` | `079` | **Unmapped** (`FF` = no virtual key) |
| HHKB **かな** | `FF` | `070` | **Unmapped** |

Takeaways:

- **What the Korean IME acts on for 한/영 is `VK_HANGUL` (`0x15`).** To toggle it from a script,
  `Send("{vk15sc138}")` should work (**not tested yet**). The script doesn't need this: F18 sets 한 directly and never toggles.
- Without the JIS layout active, the HHKB's own IME keys are dead or wrong on Windows. That's why
  Profile 4 replaces them with F17–F20.
- `0x15` is also `VK_KANA`, the same number under another name. So in Japanese, the same key
  would toggle Kana lock instead. Send it only while Korean is active, the same rule as
  `VK_HANJA` / `VK_KANJI` below.

### Native 한/영 without AutoHotkey: map a key to Mac かな (`LANG1`)

Checked by the owner (2026-10-17): a key set to **Mac かな** in the HHKB Studio Keymap Tool toggles
**한/영** in the Microsoft Korean IME.

- Mac かな sends USB HID `LANG1` (usage `0x90`), which is also the code a Korean USB keyboard's
  한/영 key sends. Windows turns it into `VK_HANGUL` (`0x15`), the same VK as the laptop's 한/영 key
  above. Mac 英数 sends `LANG2` (`0x91`), the code a Korean keyboard's 한자 key sends.
- The stock Windows JIS かな key sends a different code (`International2`, SC `070`), which the
  Korean layout leaves unmapped. That's why it does nothing.
- **It works only while Korean is the active language.** In Japanese, `0x15` is `VK_KANA` and goes
  to the Japanese IME instead, so the key seems dead. Switch to Korean first (F18 or Win+Space).
  This is why the key "stopped working" once Japanese was added.
- Expected codes for Mac かな: `VK 15 / SC 0F2`, and `VK 19 / SC 0F1` for Mac 英数. **Not
  measured yet.** Check them with KeyHistory.

Profile 4 doesn't use this. F18 sets 한 directly and never toggles, and F17 sends 한자. The
fallback profile without AutoHotkey, **Profile3** (`01-hardware.md`), replaces the stock 変換 key, right of
Space, with `LANG1`.

To repeat the test, run a two-line script (`InstallKeybdHook` then `KeyHistory`), press the keys,
and press F5 in the window to refresh.

## What is documented, inferred, or still to be measured

| Piece | Status |
| --- | --- |
| `WM_INPUTLANGCHANGEREQUEST` changes the focused window's input language | Documented (Win32) |
| `WM_IME_CONTROL` + `IMC_*` on the default IME window, from another process | Common practice, not formally documented for this use. **Measure** |
| Clearing `IME_CMODE_ROMAN` gives kana typing in the current Microsoft Japanese IME | **Inferred.** The IME's "かな入力" setting (`02-windows.md`) is the primary control; the script only avoids turning Romaji on |
| Korean 한 = conversion mode `HANGUL` (0x01) with the IME open | Inferred from imm.h. **Measure** |
| `VK_HANJA` (0x19) opens 한자 conversion in the Korean IME | Documented VK; behavior **to check**. Only sent while Korean is active, because on Japanese the same VK is `VK_KANJI` (IME toggle) |
| The Studio sends F17–F20 from the bottom row | **Checked** on the owner's board with a separate key-test tool (`01-hardware.md`) |
| Elevated (admin) windows | A non-elevated AutoHotkey can't post messages to them (UIPI), so the keys won't switch there. Known limitation |

Compared with doing it without a remapper: PowerToys Keyboard Manager can remap F18 → a key or
shortcut, but can't select a language *and* set the IME state, so one-way behavior across Korean and
Japanese isn't reachable with it. The Microsoft IME key settings only act while Japanese is active.
