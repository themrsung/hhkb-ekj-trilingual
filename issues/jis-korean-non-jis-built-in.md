# JIS Korean with a non-JIS built-in keyboard

**Applies to:** macOS, on a MacBook whose built-in keyboard is US ANSI or ISO, used both with and
without the HHKB.
**Status:** accepted. Since `v1.8.0`, JIS Korean is the only Korean input source, and Korean 2-Set
is no longer maintained, also not for the built-in keyboard.

## Symptom

In Korean, the **built-in keyboard** doesn't type what its keycaps show: the number row types
ㅃ ㅉ ㄸ ㄲ ㅆ / ㅟ ㅝ ㅚ ㅢ ㅘ (`1` types ㅃ, and Shift+`1` types `1` instead of `!`), and the
symbol keys type Korean punctuation (`·` on `/`, `「」` on Shift+`[` / `]`). Letters type standard
2-Set, and the HHKB is fine.

## Cause

JIS Korean reads physical **JIS** key codes. Karabiner ignores the built-in keyboard (its JIS
virtual keyboard would break the built-in's symbols, `../docs/03-karabiner.md`, "Non-JIS built-in
keyboard"), so macOS reads it as a plain ANSI / ISO board, and JIS Korean maps those keys as if
they were JIS.

## What to do

- Type Korean on the HHKB. On the built-in keyboard, Korean letters work. For digits and symbols,
  switch to ABC (Caps Lock, or 英数 on the HHKB).
- Don't add Korean 2-Set back to the setup. From `v1.7.0` to `v1.7.1` it was kept next to JIS
  Korean for the built-in keyboard (with the catch that Caps Lock and Ctrl+Space could land on the
  wrong Korean source); that's no longer maintained. A user who wants it can follow `v1.7.1`.

Seen on the owner's MacBook (US ANSI built-in) with an HHKB Studio over Bluetooth, macOS 27.0.1,
Hammerspoon 1.1.1, Karabiner-Elements 16.3.0 (2026-10-19, JIS Korean 0.1.0).
