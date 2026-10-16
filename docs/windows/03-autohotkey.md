# Windows 03 — AutoHotkey v2: one-way language keys

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
any existing file first: `Copy-Item f "f.bak-$(Get-Date -f yyyyMMddHHmmss)"`), and start it at
sign-in with a shortcut in `shell:startup` pointing to it.

## `hhkb-ekj-trilingual.ahk`

```autohotkey
#Requires AutoHotkey v2.0
#SingleInstance Force
; hhkb-ekj-trilingual (Windows): one-way language keys sent by the HHKB Studio, keymap Profile 4.
;   F18 -> Korean (한)   F19 -> English (Japanese IME off)   F20 -> Japanese hiragana, kana typing
;   F17 -> 한자 in Korean, nothing elsewhere   Pause (Fn1+P) -> Media Play/Pause

HKL_KO := 0x04120412, LANG_KO := 0x0412
HKL_JA := 0x04110411, LANG_JA := 0x0411

; IME conversion-mode bits (imm.h)
CM_NATIVE := 0x01, CM_KATAKANA := 0x02, CM_FULLSHAPE := 0x08, CM_ROMAN := 0x10

; Targets: language, IME open status, conversion mode to set, and the bits that must match.
KOREAN   := {hkl: HKL_KO, lang: LANG_KO, open: 1, cmode: CM_NATIVE, mask: CM_NATIVE}
ENGLISH  := {hkl: HKL_JA, lang: LANG_JA, open: 0}
JAPANESE := {hkl: HKL_JA, lang: LANG_JA, open: 1, cmode: CM_NATIVE | CM_FULLSHAPE
           , mask: CM_NATIVE | CM_KATAKANA | CM_FULLSHAPE | CM_ROMAN}

F18::SwitchTo(KOREAN)
F19::SwitchTo(ENGLISH)
F20::SwitchTo(JAPANESE)
F17::ContextAction()
Pause::Send("{Media_Play_Pause}")

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
    ImeCtl(hwnd, 0x006, t.open)                 ; IMC_SETOPENSTATUS
    if t.open
        ImeCtl(hwnd, 0x002, t.cmode)            ; IMC_SETCONVERSIONMODE
}

IsAt(hwnd, t) {
    if LangOf(hwnd) != t.lang || ImeCtl(hwnd, 0x005) != t.open   ; IMC_GETOPENSTATUS
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

## What is documented, inferred, or still to be measured

| Piece | Status |
| --- | --- |
| `WM_INPUTLANGCHANGEREQUEST` changes the focused window's input language | Documented (Win32) |
| `WM_IME_CONTROL` + `IMC_*` on the default IME window, from another process | Common practice, not formally documented for this use. **Measure** |
| Clearing `IME_CMODE_ROMAN` gives kana typing in the current Microsoft Japanese IME | **Inferred.** The IME's "かな入力" setting (`02-windows.md`) is the primary control; the script only avoids turning Romaji on |
| Korean 한 = conversion mode `HANGUL` (0x01) with the IME open | Inferred from imm.h. **Measure** |
| `VK_HANJA` (0x19) opens 한자 conversion in the Korean IME | Documented VK; behavior **to check**. Only sent while Korean is active, because on Japanese the same VK is `VK_KANJI` (IME toggle) |
| The Studio sends F17–F20 as VK `0x80`–`0x83` on every layer | **Measure** (`01-hardware.md`) |
| Elevated (admin) windows | A non-elevated AutoHotkey can't post messages to them (UIPI), so the keys won't switch there. Known limitation |

Compared with doing it without a remapper: PowerToys Keyboard Manager can remap F18 → a key or
shortcut, but can't select a language *and* set the IME state, so one-way behavior across Korean and
Japanese isn't reachable with it. The Microsoft IME key settings only act while Japanese is active.
