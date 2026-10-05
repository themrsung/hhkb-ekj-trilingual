# 04 — Hammerspoon: HHKB key → Korean 2-Set

Checked with Hammerspoon **1.1.1**.

## Why Hammerspoon (and not Karabiner `select_input_source`)

Selecting a CJK input method from a hotkey is flaky: the first select sometimes doesn't take, and
the input-source overlay can show twice. Hammerspoon lets us select, check, and retry *only when
needed*. That's how this setup handles both problems. Karabiner only turns the HHKB key into
`F18`. Hammerspoon does the switching.

## Install and permissions

```sh
brew install --cask hammerspoon
```

Then the **user** must:

1. Open Hammerspoon. In **Preferences**, turn on **Launch Hammerspoon at login**.
2. Allow **Accessibility** for Hammerspoon when prompted (System Settings → Privacy & Security →
   Accessibility). The hotkey and input-source calls don't strictly need it, but Hammerspoon asks
   for it and other modules expect it.
3. Optional: install the `hs` command-line tool for checks. Run `hs.ipc.cliInstall()` in the
   Hammerspoon console. The config below already loads `hs.ipc`.

## `~/.hammerspoon/init.lua`

If the file doesn't exist, create it with exactly this. If it exists, back it up
(`cp ~/.hammerspoon/init.lua ~/.hammerspoon/init.lua.bak-$(date +%Y%m%d%H%M%S)`) and add the
block, keeping the user's other code. Don't load `hs.ipc` twice.

```lua
require("hs.ipc") -- lets the `hs` CLI query this instance

-- to_korean key (remapped to F18 by Karabiner) → Korean 2-Set
local KOREAN = "com.apple.inputmethod.Korean.2SetKorean"

local function toKorean()
  -- Always select, even if Korean is already reported as current: the reported source can be
  -- stale when a previous switch didn't take, and skipping here left the key dead until another
  -- input source was picked.
  hs.keycodes.currentSourceID(KOREAN)
  -- CJK input methods sometimes don't take on the first select; verify and retry once
  -- (only when needed — every select shows the input-source overlay)
  hs.timer.doAfter(0.15, function()
    if hs.keycodes.currentSourceID() ~= KOREAN then
      hs.keycodes.currentSourceID(KOREAN)
    end
  end)
end

hs.hotkey.bind({}, "f18", toKorean)
```

Reload: Hammerspoon menu-bar icon → **Reload Config** (or `hs -c 'hs.reload()'`).

## Required behavior (don't "simplify" these away)

1. **The HHKB key always selects Korean 2-Set, even when Korean is already active.**
   An earlier version started with
   `if hs.keycodes.currentSourceID() == KOREAN then return end`. That was **removed**: the
   reported current source can be stale after a switch that didn't take, and the early return left
   the HHKB key dead until the user picked another source by hand. Don't add it back.
2. **The retry is guarded by a check.** Known bug: the first switch from another language to
   Korean 2-Set could show the switch overlay **twice**, because every select shows the overlay.
   The fix is the guard: after 150 ms, re-select **only if** the current source is *not* already
   2-Set. Never retry without checking.
3. **Bare F18 only.** `hs.hotkey.bind({}, "f18", …)` has no modifiers. Modified presses of the
   HHKB key do nothing, on purpose.

## Check

```sh
hs -c 'hs.keycodes.currentSourceID()'          # after pressing the HHKB key
# com.apple.inputmethod.Korean.2SetKorean
hs -c 'hs.inspect(hs.keycodes.methods(true))'  # enabled input methods (IDs)
hs -c 'hs.inspect(hs.keycodes.layouts(true))'  # enabled keyboard layouts (IDs)
```

Press the HHKB key from ABC, from Japanese, and from Korean itself. Each time the result must be
2-Set Korean, with no double overlay when coming from another language.

## Known issue: the switch occasionally doesn't reach the focused text field (unfixed)

**Symptom.** About 1 press in 100, the HHKB key "works" (the menu bar and
`hs.keycodes.currentSourceID()` both say 2-Set Korean), but the text field you're typing in keeps
the previous source (English, or Kana) until focus changes. かな and 英数 never do this.

**Cause.** This is a macOS bug, not a bug in this config. `hs.keycodes.currentSourceID(id)` calls
`TISSelectInputSource` from Hammerspoon, a background process. For CJK input methods that call
sometimes changes the system-wide source without reattaching the focused app's text input client.
かな and 英数 are real key events that the focused app handles itself, through the system's own
switching path, so they can't get out of sync. The same bug is reported in
[Karabiner-Elements #1602](https://github.com/pqrs-org/Karabiner-Elements/issues/1602) and on the
[Apple Developer Forums (#748791)](https://developer.apple.com/forums/thread/748791).

**What was measured** (macOS 27.0.1, Hammerspoon 1.1.1, TextEdit, synthetic typing read back from
the document, about 550 switches):

- Lost switches were about 1%. A lost switch stayed in the old source for more than 1.5 s, so it
  isn't just a slow switch. Losses happened more often when typing started within about 50 ms
  of the switch. With a 400 ms gap there were 0 losses in 60 trials.
- It happens in native Cocoa text fields (TextEdit), not only in web pages.
- Restarting the app or the Korean input method didn't reproduce it, and "Automatically switch to
  a document's input source" is off.
- **The 150 ms retry guard can't see this failure.** In every lost switch it read 2-Set Korean and
  skipped the retry. The guard is still correct and still required: it handles selects that don't
  take at all, and it prevents the double overlay. It just doesn't cover this case.

**Workarounds tried and rejected:**

- **Focus bounce** (activate another app for about 30 ms, then come back, as macism and Input
  Source Pro's "Switching Focus" mode do): in 60 of 60 trials it dropped the first keystroke
  after the switch. That's worse than the bug.
- **Unconditional second select at 150 ms:** no measurable effect at this failure rate, and it
  brings back the double-overlay bug.
- **A Chrome extension:** extensions can't see or set macOS input sources, and the desync happens
  below the page, so a page can't detect it.

**Other tools** (surveyed October 2026, not tested here). Anything that calls
`TISSelectInputSource` has the same bug: im-select, issw, kawa, xkbswitch-macosx, and Karabiner's
`select_input_source`. Karabiner's docs warn about it for CJKV. The approaches that address it
post a key event that the system handles itself:
[CmdIME](https://github.com/ShunmeiCho/cmd-ime) ("kanaThenSelect": post かな, wait, then select)
and the "post the input-source shortcut" approach that Karabiner's docs recommend. Input Source
Pro's "Shortcut Simulation" mode uses the same idea but warns it is unreliable on macOS 26+.
Revisit this when one of these, or a macOS update, gives a reliable Korean switch. Test any
candidate the same way: a few hundred switches with typing right after each, then read back what
landed. A ~1% rate can't be judged from a handful of presses.

Until then, if Korean didn't take, press the HHKB key again, or click out of the field and back.
