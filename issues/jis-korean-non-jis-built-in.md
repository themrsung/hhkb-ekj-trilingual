# JIS Korean with a non-JIS built-in keyboard

**Applies to:** macOS, JIS Korean (`../docs/09-jis-korean.md`) on a MacBook whose built-in keyboard
is US ANSI or ISO, used both with and without the HHKB.
**Status:** by design. There's a known catch with macOS's own switching keys.

## The setup

Two Korean input sources are enabled on purpose:

| Keyboard | Switches with | Korean source |
| --- | --- | --- |
| HHKB (JIS) | HHKB key → F18 → Hammerspoon | **JIS Korean** (`me.sjun.inputmethod.JISKorean.2Set`) |
| Built-in (US ANSI / ISO, ignored by Karabiner) | Caps Lock, Ctrl+Space, Ctrl+Option+Space | **2-Set Korean** (`com.apple.inputmethod.Korean.2SetKorean`) |

JIS Korean reads physical JIS key codes. On an ANSI board, the number row would type double
consonants and compound vowels instead of digits, and the symbol keys wouldn't match the keycaps.
So the built-in keyboard keeps standard 2-Set. Don't remove 2-Set Korean on these Macs, even
though `docs/09` says to remove it on a JIS-only setup.

## Symptom

On the built-in keyboard, Caps Lock or Ctrl+Space lands on **JIS Korean** instead of 2-Set Korean.

## Cause

macOS's Caps Lock switch toggles between ABC and the **last used** non-ABC source, and Ctrl+Space
selects the **previous** source. After the HHKB key has selected JIS Korean, that's the source
these keys go back to.

## What to do

Look at the menu-bar input icon. Press Ctrl+Option+Space (Select next source) to step to 2-Set
Korean. Once 2-Set has been used on the built-in keyboard, Caps Lock / Ctrl+Space go back to it
until the HHKB key is pressed again.

Not fixed in the configs. Karabiner can't help, because it ignores the built-in keyboard (see
`../docs/03-karabiner.md`, "Non-JIS built-in keyboard"), and un-ignoring it breaks the built-in
keyboard's symbols. A Hammerspoon binding that selects 2-Set explicitly would work, but it would
need a key the built-in keyboard has spare. Nothing has been measured yet.

Checked on the owner's MacBook (US ANSI built-in) with an HHKB Studio over Bluetooth, macOS
27.0.1, JIS Korean 0.1.0, Hammerspoon 1.1.1, Karabiner-Elements 16.3.0 (2026-10-19).
