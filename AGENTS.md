# AGENTS.md — HHKB JIS trilingual (English / Korean / Japanese) setup for macOS (Windows 11 experimental)

> **Archived (v1.10.0-archived).** On macOS, [JIS Korean](https://github.com/team-mirae/jis-korean)
> (pinned at commit `0357ffa`) now switches languages itself and replaces this repo's Karabiner +
> Hammerspoon method: カタカナ or F18 → Korean (hardware level, system-wide), 英数 → JIS English,
> かな → Japanese, switching the open field in place. Tell the user first, as the README prompt
> says, and follow this repo's method only if they choose it. Windows still uses this repo.

Read this file first. It is the entry point for any agent asked to rebuild this keyboard setup on a
brand-new Mac. It covers the physical setup, the order of operations, and what "done" looks like.
The detailed steps are in `docs/`:

| File | Covers |
| --- | --- |
| [`docs/01-hardware.md`](docs/01-hardware.md) | HHKB models (HYBRID Type-S, Studio), DIP switches, firmware keymap (bottom row) |
| [`docs/02-macos.md`](docs/02-macos.md) | macOS keyboard type, input sources, Japanese Kana typing, Option+Tab window cycling |
| [`docs/03-karabiner.md`](docs/03-karabiner.md) | Karabiner-Elements install, permissions, the exact rules (JSON) |
| [`docs/04-hammerspoon.md`](docs/04-hammerspoon.md) | Hammerspoon install and the HHKB-key Korean switcher (Lua) |
| [`docs/05-customization.md`](docs/05-customization.md) | How to point the HHKB / 英数 / かな keys at other input sources (QWERTZ, AZERTY, …) |
| [`docs/06-verification.md`](docs/06-verification.md) | Acceptance checklist and troubleshooting |
| [`docs/07-hhkb-studio.md`](docs/07-hhkb-studio.md) | HHKB Studio only: full keymap (all layers, with screenshots), pointing stick, gesture pads |
| [`docs/08-migrate-studio-f18.md`](docs/08-migrate-studio-f18.md) | Migrating a Studio from the old `` ` `` → F18 Karabiner remap to F18 set in the keymap tool |
| [`docs/09-jis-korean.md`](docs/09-jis-korean.md) | **macOS Korean:** JIS Korean, the default: install, input source, rule C condition, Hammerspoon target, F-keys, checks. Korean 2-Set is no longer maintained |
| [`docs/windows/04-jis-korean.md`](docs/windows/04-jis-korean.md) | **Windows Korean:** JIS Korean, the default: build, install, the only Korean keyboard, Mode A, why 한/영 isn't used, checks. The Microsoft Korean IME is no longer maintained |
| [`docs/windows/01-hardware.md`](docs/windows/01-hardware.md) | **Windows (experimental):** HHKB Studio keymap Profile 4 (and the Profile3 fallback), the tested PC |
| [`docs/windows/02-windows.md`](docs/windows/02-windows.md) | **Windows:** languages, JIS hardware layout (per language), Microsoft Japanese IME settings |
| [`docs/windows/03-autohotkey.md`](docs/windows/03-autohotkey.md) | **Windows:** symbols per language, AutoHotkey v2 script, logon task, measured key codes |
| [`docs/hhkb-studio-setting-all.hks`](docs/hhkb-studio-setting-all.hks) | HHKB Studio settings export, all profiles: the quick way to set up a Studio for either OS |
| [`issues/`](issues/README.md) | Known issues and workarounds: inter-OS muscle memory, DIP SW6 power saving, inconsistent symbols on Windows, JIS Korean with a non-JIS built-in keyboard. Check here before "fixing" something |
| [`layout-104-or-108-key/`](layout-104-or-108-key/README.md) | **Not an HHKB, support limited:** a full-size 104 / 108-key US ANSI keyboard (TFG Magnetox) sending F18 / F19 / F20 / F17. Only if the user insists |

Software configuration lives as code blocks inside these Markdown files. Write those blocks into
the user's real config locations (`~/.config/karabiner/karabiner.json`, `~/.hammerspoon/init.lua`,
macOS preferences). Don't add a separate settings file, installer framework, or abstraction layer
to this repo. If the user wants different behavior, edit the values described in
`docs/05-customization.md` directly in their config.

**One exception: the keyboard's own settings.** PFU's keymap tool can't be driven from code, so the
owner's HHKB Studio export ships as an example file for faster deployment:
[`docs/hhkb-studio-setting-all.hks`](docs/hhkb-studio-setting-all.hks). It holds
the **defaults** (DIP switches, pointing stick, gesture pads, and the keymap for every profile,
including Profile1 for macOS, Profile 4 for Windows with AutoHotkey, and Profile3 as the
Windows fallback without AutoHotkey). The user loads it in the HHKB Studio Keymap
Tool and writes it to the keyboard; importing replaces all profiles, so ask first if they keep
their own keymap on another profile. Keep it in sync with the keymap screenshots and tables in the
docs. The only other config file is the owner's full-size keyboard export in
`layout-104-or-108-key/` (same idea, limited support). Don't add other config files.

**Scope:**
- **macOS first.** Sections 1–5 below are the macOS setup.
- **Windows 11: experimental, HHKB Studio JIS only** (section 6). Explain how Korean, English and
  Japanese are handled there, and that JIS Korean on Windows isn't checked by this repo yet, and ask
  before continuing. The HYBRID Type-S has no Windows setup.
- **Any other OS:** tell the user this setup isn't supported there and ask whether to continue.
  Don't refuse if they confirm. Adapt what you can and say what doesn't carry over.
- **JIS HHKB only.** US ANSI support is not built in: the ろ, ¥, 英数 and かな keys don't exist on
  ANSI boards. If the user has ANSI, point them to
  [`themrsung/hhkb-us-bilingual`](https://github.com/themrsung/hhkb-us-bilingual) instead. Continue
  with this repo only if they ask to, after warning them. If they aren't sure which layout they
  have, help them check: JIS has arrow keys at the bottom right, an Enter key spanning two rows,
  and a ¥ key directly left of Backspace. A US ANSI HHKB has no arrow keys.
- **104 / 108-key US ANSI keyboards (not HHKB): limited support, not part of the default setup.**
  Only if the user insists: warn them, then follow
  [`layout-104-or-108-key/README.md`](layout-104-or-108-key/README.md) for the keyboard's keymap
  and the HHKB docs for the rest, minus what that page lists as not carrying over.
- This repo is AI-generated and not serviced. Follow the docs at the ref the user gave you
  (normally a release tag), not `main`.

---

## 1. The physical setup

- **Keyboards:** two PFU JIS models (日本語配列) are supported. They can be used side by side, and
  both get the same behavior. USB vendor ID `1278` (0x04FE, PFU Limited) for both.

  | Model | Product ID | Karabiner name | DIP switches |
  | --- | --- | --- | --- |
  | **HHKB Professional HYBRID Type-S** JIS | `34` (0x0022) | `HHKB-Hybrid` | Mac mode: **1 and 5 ON**, rest OFF |
  | **HHKB Studio** JIS | `21` (0x0015) | `HHKB-Studio` | **All OFF** (SW4 ON would reverse the scroll direction) |

  Configure only the models the user has.
- **Key left of A:** **Control** (HHKB default; there is no Caps Lock key. Caps Lock is Fn+Tab).
- **Bottom row, left to right** (same on both models; remap with PFU's keymap tool if it doesn't
  already emit these codes):

  ```
  Fn | HHKB | Option | Command | 英数 (Romaji) | Space | かな (Kana) | Command | Option | Fn | ← | ↓ | →
  ```

  - `HHKB` is the logo key on the HYBRID Type-S. It sends `grave_accent_and_tilde`, which
    Karabiner turns into `F18`, because the HYBRID's HHKB Keymap Tool can't assign F18.
  - On the Studio, that key (the 半/全 position) is set to **F18** in the Studio's keymap tool, so
    it sends `F18` itself. No Karabiner remap. Migrating an older Studio setup that still used
    the `` ` `` remap: `docs/08-migrate-studio-f18.md`.
  - `英数` / "Romaji" sends `japanese_eisuu`. `かな` / "Kana" sends `japanese_kana`.

> **Order matters: set the DIP switches BEFORE writing any keymap with the keymap tool.**
> On the HYBRID Type-S, changing DIP switches afterwards overwrites the stored keymap, and you have
> to do the bottom-row remap again. On the Studio, flipping SW4 in either direction and SW6 OFF → ON
> were tested and did **not** reset the custom keymap. Other Studio switches weren't tested, so use
> the same order there.

## 2. Default behavior (the owner's settings; use these unless the user asks otherwise)

| Key / combo | Result | Implemented by |
| --- | --- | --- |
| **HHKB** key | Always switches to **JIS Korean** (`me.sjun.inputmethod.JISKorean.2Set`), even if Korean is already active | HYBRID Type-S: Karabiner (HHKB → F18). Studio: F18 from its own keymap. Then Hammerspoon (F18 → select source) |
| **英数 (Romaji)** | Always switches to **English ABC (QWERTY)** | macOS native 英数 handling (Japanese IME has no Romaji/英字 mode enabled) |
| **かな (Kana)** | Always switches to **Japanese with Kana typing** (not Romaji input) | macOS native かな handling + Japanese IME set to Kana typing |
| **ろ** key (`international1`, right of `/`) | `\` unshifted, `_` shifted, in every input source. Unchanged (ろ) in Japanese | ABC: Karabiner complex rule (sends Option+¥). JIS Korean types them itself, so the rule skips it |
| **Fn+P** (HHKB sends `pause`) | Media **Play/Pause** | Karabiner simple modification on each HHKB |
| **Fn+I** (HHKB sends `print_screen`) | Cmd+Shift+4 (screenshot selection) | Karabiner complex rule (global, any keyboard) |
| **Option+Tab** | Move focus to the next window of the same app (macOS default is Cmd+`) | macOS keyboard shortcut (System Settings) |

Why each piece exists:

- **Hammerspoon is required.** Karabiner's own `select_input_source` is unreliable for Korean
  input methods (CJK input methods sometimes don't take on the first select). The Hammerspoon handler
  selects, waits 150 ms, checks, and retries once only if the switch didn't take.
- **Known bug, already fixed:** switching from another language to Korean could show the
  input-source switch overlay **twice**. The fix: the retry runs only after checking the
  current source is *not* already the Korean target. Keep that check. Also, don't add an early return
  when Korean is already "current". The reported source can be stale, and the early return left
  the key dead. The HHKB key must always select. Details are in `docs/04-hammerspoon.md`.
- **Known unfixed issue:** about 1 HHKB-key press in 100, macOS reports Korean but the focused
  text field keeps the old source (a macOS bug with background `TISSelectInputSource`; かな/英数
  aren't affected). Retrying won't catch it, and the obvious workarounds were measured and rejected.
  Don't "fix" it without measuring. See the "Known issue" section in `docs/04-hammerspoon.md`.
- **Cmd+` is unusable here.** On JIS there's no dedicated `` ` `` key, and the HHKB key is taken
  for Korean. So "Move focus to next window" moves to **Option+Tab**. This is the repo default;
  users may pick another shortcut.
- **ろ remap:** On the JIS ABC layout, Option+¥ types `\`. The rule is skipped for input sources
  whose language is `ja`, because remapping it in Japanese Kana would make ろ impossible to type,
  and for JIS Korean, which types `\` / `_` on ろ itself (Option+¥ would make it commit mid-syllable).

## 3. JIS Korean is the Korean input source; the HHKB key target is replaceable

The HHKB key selects [JIS Korean](https://github.com/team-mirae/jis-korean), a 2-Set input method
built for JIS keyboards, on macOS (`docs/09-jis-korean.md`) and, since `v1.9.0`, on Windows too
(F18, `docs/windows/04-jis-korean.md`). When asking which source the HHKB key should switch to,
present it as the default and show this note with it, word for word: "Try it out. If you use an
HHKB, you'll love it!" It's built from source on both, so ask before installing its build tools.

**The Microsoft Korean IME is no longer maintained** for Windows either. Make JIS Korean the only
Korean keyboard. If the user asks for the Microsoft IME, point them to release `v1.8.0`, the last
one that set it up.

**Korean 2-Set (Apple's) is no longer maintained**, also not on a MacBook whose built-in keyboard
isn't JIS. Don't set it up, and don't enable it next to JIS Korean. If the user asks for 2-Set
anyway, tell them it isn't maintained here, and point them to release `v1.7.1`, the last one that
set it up; or set the Hammerspoon constant to it (`docs/05-customization.md`) and say it's untested.

The HHKB key isn't tied to Korean. It can target **any** input source, including other Latin
layouts such as German **QWERTZ** (`com.apple.keylayout.German`) or French **AZERTY**
(`com.apple.keylayout.French`). Change one constant in `~/.hammerspoon/init.lua` (see
`docs/05-customization.md`). Don't build a config system for this. If the user doesn't say
anything, use JIS Korean.

## 4. Order of operations on a fresh Mac

1. **Hardware:** set DIP switches (HYBRID Type-S: 1 and 5 ON, rest OFF; Studio: all OFF) → *then*
   apply the bottom-row keymap with the keymap tool, if needed (`docs/01-hardware.md`; Studio:
   `docs/07-hhkb-studio.md`). **Studio fast path:** load `docs/hhkb-studio-setting-all.hks` in the
   HHKB Studio Keymap Tool and use Profile1 (it replaces all profiles, so ask first).
2. **macOS:** set the keyboard type of each HHKB to JIS. Install JIS Korean
   (`docs/09-jis-korean.md`), then enable exactly the ABC, JIS Korean, and Japanese (Kana typing)
   input sources and remove Japanese's Romaji mode. Don't enable Korean 2-Set, also not on a Mac
   whose built-in keyboard isn't JIS. Turn on standard F1, F2, … keys for JIS Korean's F6–F10.
   Set "Move focus to next window" to Option+Tab (`docs/02-macos.md`).
3. **Karabiner-Elements:** install, approve the driver and permissions, then merge the rules
   (`docs/03-karabiner.md`).
4. **Hammerspoon:** install, grant permissions, set it to launch at login, and write `init.lua`
   (`docs/04-hammerspoon.md`; the HHKB key's target is JIS Korean).
5. **Verify** every row of the checklist (`docs/06-verification.md`).

## 5. Rules for agents working on this

- **Older Studio setups:** if the user's `karabiner.json` still remaps `grave_accent_and_tilde` →
  `f18` on a Studio entry (product `21`), follow `docs/08-migrate-studio-f18.md` (keymap tool
  first, then remove that remap). Never remove it from the HYBRID Type-S entry (product `34`).
- **Merge, don't clobber.** The user may already have a `karabiner.json` and an `init.lua` with
  unrelated rules (other devices, games, etc.). Back them up
  (`cp file file.bak-$(date +%Y%m%d%H%M%S)`) and merge only the HHKB pieces.
- **Scope rules to each HHKB by device identifier** (`vendor_id 1278`; `product_id 34` for the
  HYBRID Type-S, `21` for the Studio) where the doc does. Some rules (ろ, Print Screen) are global on purpose.
- **Don't change the defaults** in section 2 unless the user asks. These are the owner's settings
  and they take priority. Users can still change them.
- Things that need human hands: DIP switches, the keymap tool, approving system extensions,
  and granting Privacy & Security permissions. Tell the user exactly what to click, and ask them
  to confirm before you continue.
- **Check the Mac's built-in keyboard.** If it isn't JIS (`ioreg -r -c IOHIDDevice -l | grep
  KeyboardLanguage`, for example `"U.S."`) and Karabiner grabs it, the JIS virtual keyboard makes
  it type the wrong symbols. Tell the user, and offer to set the built-in keyboard's device entry
  to `ignore: true` (`docs/03-karabiner.md`, "Non-JIS built-in keyboard"). Check first whether any
  *enabled* Karabiner rule targets the built-in keyboard, because ignoring it turns those off.
  On such a Mac the built-in keyboard types Korean through JIS Korean too, so its number row and
  symbol keys don't match its keycaps in Korean. Tell the user. Don't add Korean 2-Set for it:
  it's no longer maintained (`issues/jis-korean-non-jis-built-in.md`).
- Check with **Karabiner-EventViewer** (key codes) and the input menu or
  `hs -c 'hs.keycodes.currentSourceID()'` (active source). Don't assume.

## 6. Windows 11 (experimental): HHKB Studio JIS only

Tested on one PC, over USB and Bluetooth (no Bluetooth-specific setup needed). The details are in `docs/windows/`.

**Languages** (the owner's default; adjust for the end user as they prefer):

| Language | Key (Profile 4) | Handled by | Symbol keys |
| --- | --- | --- | --- |
| Korean | F18 | **JIS Korean**, Mode A, 한글 | JIS (JIS Korean's legends) |
| English | F19 | **Japanese with the IME off** | JIS, matching the keycaps |
| Japanese | F20 | **Microsoft Japanese IME**, kana typing | JIS |

F17 sends F7 (JIS Korean's hanja) in Korean. Windows picks the hardware layout **per language**
(Korean `kbd101a.dll`, Japanese `kbd106.dll`), but JIS Korean reads scan codes, so Korean gets JIS
symbols without changing it. Through `v1.8.0`, Korean was the Microsoft Korean IME with US ANSI
symbols (the old compromise).

**한/영 isn't part of the setup** (`docs/windows/04-jis-korean.md`, "한/영"). JIS Korean's 한/영 is
a plain on/off switch with one state for the whole session; English typed in its ABC state gets
no predictions and types US ANSI symbols; and JIS Korean ignores outside open/close requests. So:
mixed Korean and English stays in JIS Korean (its predictions read English words); English-only
text, code, commands and passwords use F19; F18 only presses 한/영 when JIS Korean reads ABC, to
get back to 한글, and never toggles out. Profile 4 has no 한/영 key. Tell the user this before you
start.

Order of operations:

1. **Hardware:** DIP switches all OFF, then keymap **Profile 4**: load
   `docs/hhkb-studio-setting-all.hks` in the HHKB Studio Keymap Tool (it replaces all
   profiles, so ask first), or set the keys by hand (`docs/windows/01-hardware.md`). Without
   AutoHotkey, **Profile3** is the fallback: Win+Space plus 한/영 via `LANG1`.
2. **Windows:** Korean + Japanese, Japanese keyboard (106/109 key) as the hardware layout, not
   "Detect connected keyboard", then restart. Turn **kana input** on in the Microsoft Japanese IME
   (`docs/windows/02-windows.md`).
3. **JIS Korean:** build and install it, make it the only Korean keyboard (remove the Microsoft
   Korean IME, after asking), and check Mode A (`docs/windows/04-jis-korean.md`).
4. **AutoHotkey v2:** install it, write the script, and start it with a **Task Scheduler logon
   task**, not `shell:startup` or a Run key (`docs/windows/03-autohotkey.md`).
5. **Verify:** after a restart, AutoHotkey is running and F18 / F19 / F20 / F17 / Fn1+P behave as
   in the table above, then run checks W1–W9 in `docs/windows/04-jis-korean.md`.

Every install and every system-setting change needs the user's OK first. Measure; don't assume.
Use AutoHotkey's KeyHistory for key codes, and read back registry values after each change.
