# Inconsistent symbol typing on Windows

**Applies to:** the Windows setup (`../docs/windows/`).
**Status:** known, with a one-key workaround.

## Symptom

Symbol keys sometimes type **US ANSI** symbols instead of the JIS keycap ones, even though you
think you're in English. For example, Shift+2 gives `@` instead of `"`.

## Cause

Windows chooses the hardware layout **per language**. Korean uses `kbd101a.dll` (US ANSI symbols),
and Japanese uses `kbd106.dll` (JIS). English in this setup is "Japanese with the IME off", so it
gets JIS symbols. Korean is JIS Korean since `v1.9.0`, which types its own JIS symbols in 한글, so
the Korean layout's US symbols only come through when a key reaches that layout untouched:

- **JIS Korean is in ABC** (한/영 off): a click on the taskbar indicator, a laptop's 한/영 key, or
  Profile3's `LANG1`. The state is one for the whole session, so every window in Korean types US
  symbols until it's switched back (`../docs/windows/04-jis-korean.md`, "한/영").
- **The Microsoft Korean IME is still enabled** next to JIS Korean, and a window landed on it.
- **A window is in Korean while you expect English.** Windows keeps the input language **per
  window**, so a window can still be in Korean, or switch to Korean on its own (Win+Space, or an
  app that sets its own language). Through `v1.8.0` that alone gave US symbols; with JIS Korean in
  한글 you'd see Hangul instead. The exact trigger hasn't been pinned down.

## What to do

Press **F19 (English)** or **F20 (Japanese かな)**. Both switch the focused window to the Japanese
language, which uses the JIS layout, and the symbols match the keycaps again. If it happens in
Korean, press **F18**: it switches JIS Korean from ABC back to 한글, for every app. If the
Microsoft Korean IME shows up in Win+Space, remove it (`../docs/windows/04-jis-korean.md`, step 2).
