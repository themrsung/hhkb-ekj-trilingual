# Inconsistent symbol typing on Windows

**Applies to:** the Windows setup, Path A (`../docs/windows/03-autohotkey.md`). Path B
(FKeyIME) types JIS symbols in every language and keeps one mode for all windows, so it shouldn't
happen there (not measured for this issue yet).
**Status:** known, with a one-key workaround.

## Symptom

Symbol keys sometimes type **US ANSI** symbols instead of the JIS keycap ones, even though you
think you're in English. For example, Shift+2 gives `@` instead of `"`.

## Cause

Windows chooses the hardware layout **per language**. Korean uses `kbd101a.dll` (US ANSI symbols),
and Japanese uses `kbd106.dll` (JIS). English in this setup is "Japanese with the IME off", so it
gets JIS symbols. That's the compromise in `../docs/windows/03-autohotkey.md`.

Windows also keeps the input language **per window**. So a window can still be in Korean, or switch
to Korean on its own (for example with Win+Space, or an app that sets its own language), while you
expect English. Its symbols then follow the Korean (US ANSI) layout. The exact trigger hasn't been
pinned down.

## What to do

Press **F19 (English)** or **F20 (Japanese かな)**. Both switch the focused window to the Japanese
language, which uses the JIS layout, and the symbols match the keycaps again. F18 (Korean) always
types US ANSI symbols. That part is the compromise, not this issue.
