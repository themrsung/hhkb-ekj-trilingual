# AGENTS.md — HHKB JIS trilingual (English / Korean / Japanese) setup for macOS (Windows 11 experimental)

Read this file first. It is the entry point for any agent asked to rebuild this keyboard setup on a
brand-new Mac. It covers the physical setup, the order of operations, and what "done" looks like.
The detailed steps are in `docs/`:

| File | Covers |
| --- | --- |
| [`docs/01-hardware.md`](docs/01-hardware.md) | HHKB models (HYBRID Type-S, Studio), DIP switches, firmware keymap (bottom row) |
| [`docs/02-macos.md`](docs/02-macos.md) | macOS keyboard type, input sources, Japanese Kana typing, Option+Tab window cycling |
| [`docs/03-karabiner.md`](docs/03-karabiner.md) | Karabiner-Elements install, permissions, the exact rules (JSON) |
| [`docs/04-hammerspoon.md`](docs/04-hammerspoon.md) | Hammerspoon install and the Korean 2-Set switcher (Lua) |
| [`docs/05-customization.md`](docs/05-customization.md) | How to point the HHKB / 英数 / かな keys at other input sources (QWERTZ, AZERTY, …) |
| [`docs/06-verification.md`](docs/06-verification.md) | Acceptance checklist and troubleshooting |
| [`docs/07-hhkb-studio.md`](docs/07-hhkb-studio.md) | HHKB Studio only: full keymap (all layers, with screenshots), pointing stick, gesture pads |
| [`docs/08-migrate-studio-f18.md`](docs/08-migrate-studio-f18.md) | Migrating a Studio from the old `` ` `` → F18 Karabiner remap to F18 set in the keymap tool |
| [`docs/windows/01-hardware.md`](docs/windows/01-hardware.md) | **Windows (experimental):** HHKB Studio keymap Profile 4 (and the Profile3 fallback), the tested PC |
| [`docs/windows/02-windows.md`](docs/windows/02-windows.md) | **Windows:** languages, JIS hardware layout (per language), Microsoft IME settings |
| [`docs/windows/03-autohotkey.md`](docs/windows/03-autohotkey.md) | **Windows:** the compromise, AutoHotkey v2 script, logon task, measured key codes |

Software configuration lives as code blocks inside these Markdown files. Write those blocks into
the user's real config locations (`~/.config/karabiner/karabiner.json`, `~/.hammerspoon/init.lua`,
macOS preferences). Don't add a separate settings file, installer framework, or abstraction layer
to this repo. If the user wants different behavior, edit the values described in
`docs/05-customization.md` directly in their config.

**One exception: the keyboard's own settings.** PFU's keymap tool can't be driven from code, so the
owner's HHKB Studio export ships as an example file for faster deployment:
[`docs/windows/hhkb-studio-setting-all.hks`](docs/windows/hhkb-studio-setting-all.hks). It holds
the **defaults** (DIP switches, pointing stick, gesture pads, and the keymap for every profile,
including Profile1 for macOS, Profile 4 for Windows with AutoHotkey, and Profile3 as the
Windows fallback without AutoHotkey). The user loads it in the HHKB Studio Keymap
Tool and writes it to the keyboard; importing replaces all profiles, so ask first if they keep
their own keymap on another profile. Keep it in sync with the keymap screenshots and tables in the
docs. Don't add other config files.

**Scope:**
- **macOS first.** Sections 1–5 below are the macOS setup.
- **Windows 11: experimental, HHKB Studio JIS only, with a compromise** (section 6). Explain the
  compromise and ask before continuing. The HYBRID Type-S has no Windows setup.
- **Any other OS:** tell the user this setup isn't supported there and ask whether to continue.
  Don't refuse if they confirm. Adapt what you can and say what doesn't carry over.
- **JIS HHKB only.** US ANSI support is not built in: the ろ, ¥, 英数 and かな keys don't exist on
  ANSI boards. If the user has ANSI, point them to
  [`themrsung/hhkb-us-bilingual`](https://github.com/themrsung/hhkb-us-bilingual) instead. Continue
  with this repo only if they ask to, after warning them. If they aren't sure which layout they
  have, help them check: JIS has arrow keys at the bottom right, an Enter key spanning two rows,
  and a ¥ key directly left of Backspace. US ANSI has no arrow keys.
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
> to do the bottom-row remap again. On the Studio, flipping SW4 in either direction was tested and did
> **not** reset the custom keymap. Other Studio switches weren't tested, so use the same order there.

## 2. Default behavior (the owner's settings; use these unless the user asks otherwise)

| Key / combo | Result | Implemented by |
| --- | --- | --- |
| **HHKB** key | Always switches to **Korean 2-Set** (`com.apple.inputmethod.Korean.2SetKorean`), even if Korean is already active | HYBRID Type-S: Karabiner (HHKB → F18). Studio: F18 from its own keymap. Then Hammerspoon (F18 → select source) |
| **英数 (Romaji)** | Always switches to **English ABC (QWERTY)** | macOS native 英数 handling (Japanese IME has no Romaji/英字 mode enabled) |
| **かな (Kana)** | Always switches to **Japanese with Kana typing** (not Romaji input) | macOS native かな handling + Japanese IME set to Kana typing |
| **ろ** key (`international1`, right of `/`) | `\` unshifted, `_` shifted, in every **non-Japanese** input source. Unchanged (ろ) in Japanese | Karabiner complex rule (sends Option+¥) |
| **Fn+P** (HHKB sends `pause`) | Media **Play/Pause** | Karabiner simple modification on each HHKB |
| **Fn+I** (HHKB sends `print_screen`) | Cmd+Shift+4 (screenshot selection) | Karabiner complex rule (global, any keyboard) |
| **Option+Tab** | Move focus to the next window of the same app (macOS default is Cmd+`) | macOS keyboard shortcut (System Settings) |

Why each piece exists:

- **Hammerspoon is required.** Karabiner's own `select_input_source` is unreliable for Korean
  2-Set (CJK input methods sometimes don't take on the first select). The Hammerspoon handler
  selects, waits 150 ms, checks, and retries once only if the switch didn't take.
- **Known bug, already fixed:** switching from another language to Korean 2-Set could show the
  input-source switch overlay **twice**. The fix: the retry runs only after checking the
  current source is *not* already 2-Set. Keep that check. Also, don't add an early return
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
  whose language is `ja`, because remapping it in Japanese Kana would make ろ impossible to type.

## 3. Korean 2-Set is plug-and-play; the HHKB key target is replaceable

With the steps above, Korean 2-Set works out of the box. The HHKB key isn't tied to Korean. It
can target **any** input source, including other Latin layouts such as German **QWERTZ**
(`com.apple.keylayout.German`) or French **AZERTY** (`com.apple.keylayout.French`). Change one
constant in `~/.hammerspoon/init.lua` (see `docs/05-customization.md`). Don't build a config
system for this. If the user doesn't say anything, use Korean 2-Set.

## 4. Order of operations on a fresh Mac

1. **Hardware:** set DIP switches (HYBRID Type-S: 1 and 5 ON, rest OFF; Studio: all OFF) → *then*
   apply the bottom-row keymap with the keymap tool, if needed (`docs/01-hardware.md`; Studio:
   `docs/07-hhkb-studio.md`).
2. **macOS:** set the keyboard type of each HHKB to JIS. Enable the ABC, Korean 2-Set, and Japanese
   (Kana typing) input sources and remove Japanese's Romaji mode. Set "Move focus to next
   window" to Option+Tab (`docs/02-macos.md`).
3. **Karabiner-Elements:** install, approve the driver and permissions, then merge the rules
   (`docs/03-karabiner.md`).
4. **Hammerspoon:** install, grant permissions, set it to launch at login, and write `init.lua`
   (`docs/04-hammerspoon.md`).
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
- Check with **Karabiner-EventViewer** (key codes) and the input menu or
  `hs -c 'hs.keycodes.currentSourceID()'` (active source). Don't assume.

## 6. Windows 11 (experimental): HHKB Studio JIS only

Tested on one PC, over USB and Bluetooth (no Bluetooth-specific setup needed). The details are in `docs/windows/`.

**The compromise** (the owner's default; adjust for the end user as they prefer):

| Language | Key (Profile 4) | Handled by | Symbol keys |
| --- | --- | --- | --- |
| Korean | F18 | Hand off to the **Microsoft Korean IME** (한 mode) | **US ANSI**: the Korean IME enforces them |
| English | F19 | **Japanese with the IME off** | JIS, matching the keycaps |
| Japanese | F20 | **Microsoft Japanese IME**, kana typing | JIS |

F17 sends 한자 in Korean. Windows picks the hardware layout **per language** (Korean `kbd101a.dll`,
Japanese `kbd106.dll`), so Korean can't have JIS symbols without the trade-offs listed in
`docs/windows/02-windows.md`, step 3. The owner types English by default and Korean only when
needed, so this costs little. Results may vary with typing habits, so tell the user before you
start.

Order of operations:

1. **Hardware:** DIP switches all OFF, then keymap **Profile 4**: load
   `docs/windows/hhkb-studio-setting-all.hks` in the HHKB Studio Keymap Tool (it replaces all
   profiles, so ask first), or set the keys by hand (`docs/windows/01-hardware.md`). Without
   AutoHotkey, **Profile3** is the fallback: Win+Space plus native 한/영 via `LANG1`.
2. **Windows:** Korean + Japanese, Japanese keyboard (106/109 key) as the hardware layout, not
   "Detect connected keyboard", then restart. Turn **kana input** on in the Microsoft Japanese IME
   (`docs/windows/02-windows.md`).
3. **AutoHotkey v2:** install it, write the script, and start it with a **Task Scheduler logon
   task**, not `shell:startup` or a Run key (`docs/windows/03-autohotkey.md`).
4. **Verify:** after a restart, AutoHotkey is running and F18 / F19 / F20 / F17 / Fn1+P behave as
   in the table above.

Every install and every system-setting change needs the user's OK first. Measure; don't assume.
Use AutoHotkey's KeyHistory for key codes, and read back registry values after each change.
