# hhkb-ekj-trilingual

> [!WARNING]
> **This entire repo was AI-generated**: the docs, the Karabiner rules, the Hammerspoon Lua, and
> this README. If you don't want AI-written code on your machine, **skip the interactive setup**
> and read the files yourself before applying anything.

> [!NOTE]
> **Using a US (ANSI) HHKB?** Use [`themrsung/hhkb-us-bilingual`](https://github.com/themrsung/hhkb-us-bilingual)
> instead. This repo is for the **JIS** layout only.
>
> A US board has no spare keys next to the space bar like JIS's 英数 and かな, so it's physically
> very hard to give every language its own one-way button there. A toggle key is the realistic
> compromise on US.

English / Korean / Japanese on a **JIS HHKB Professional HYBRID Type-S** and/or a **JIS HHKB
Studio** under **macOS**, with one key per language:

| Key | Goes to |
| --- | --- |
| **HHKB** (logo key) | Korean 2-Set (always, even if already Korean) |
| **英数** (Romaji) | English ABC (QWERTY) |
| **かな** (Kana) | Japanese, Kana typing |

Plus: ろ types `\` (Shift+ろ → `_`) outside Japanese, Fn+P = Play/Pause, Fn+I = Cmd+Shift+4, and
"Move focus to next window" on **Option+Tab**.

Bottom row, the same on both models (remapped with PFU's keymap tool where needed; DIP switches
**set before writing the keymap**: HYBRID Type-S 1 and 5 ON, rest OFF; Studio all OFF):

```
Fn | HHKB | Option | Command | 英数 | Space | かな | Command | Option | Fn | ← | ↓ | →
```

On the Studio, the HHKB-key role is the key in the **半/全** position, set to **F18** in the Studio's
keymap tool so it sends F18 itself. The HYBRID Type-S's keymap tool can't do that, so its HHKB key
is remapped to F18 in Karabiner. Control sits left of A. The full Studio
keymap and pointing-stick / gesture-pad settings are in [`docs/07-hhkb-studio.md`](docs/07-hhkb-studio.md).

Built with [Karabiner-Elements](https://karabiner-elements.pqrs.org/) and
[Hammerspoon](https://www.hammerspoon.org/).

> [!NOTE]
> **Windows 11: experimental, HHKB Studio only, with a compromise.** The Studio uses keymap
> Profile 4 (F18 / F19 / F20 / F17), and [AutoHotkey v2](https://www.autohotkey.com/) does the
> switching:
>
> | Key | Goes to | Symbol keys |
> | --- | --- | --- |
> | F18 | Korean: handed off to the Microsoft Korean IME | **US ANSI** (the Korean IME enforces them) |
> | F19 | English: Japanese with the IME off | JIS |
> | F20 | Japanese: Microsoft Japanese IME, kana typing | JIS |
>
> The author types English by default and Korean only when needed, so the US symbols in Korean
> cost little. Results may vary with typing habits. Tested on one PC, over USB and Bluetooth. See [`docs/windows/`](docs/windows/01-hardware.md).

## Scope

- **macOS first.** This repo was designed for macOS.
- **Windows 11 is experimental:** HHKB Studio JIS only, with the compromise above (Korean types
  US ANSI symbols). The agent tells you about the compromise and asks before going on. The HYBRID
  Type-S has no Windows setup.
- **Other operating systems are not supported.** The agent warns you and asks before going on, but
  it won't refuse if you confirm.
- **JIS only.** Built and tested on the HHKB Professional HYBRID Type-S **JIS** and the HHKB
  Studio **JIS**. **US ANSI
  support is not built in.** The ろ, ¥, 英数 and かな keys this setup depends on don't exist on
  ANSI boards. For a US HHKB, use
  [`themrsung/hhkb-us-bilingual`](https://github.com/themrsung/hhkb-us-bilingual), which uses a
  toggle instead of one key per language.
- The defaults are the author's own preferences. To send the HHKB key to a different input
  source (QWERTZ, AZERTY, …), see [`docs/05-customization.md`](docs/05-customization.md).

## Interactive setup (for a coding agent)

Paste the prompt below into a coding agent running on the Mac (or, experimentally, the Windows 11
PC) you want to set up (Claude Code, Codex, etc.). It asks you a few questions, then does the rest
on its own. It stops only for steps that need your hands: DIP switches, the keymap tool,
permission prompts, settings that change the system, and key-press checks.

The prompt is **pinned to the release tag `v1.5.1`**. The agent reads the docs exactly as they
were at that release, even if `main` changes later.

```text
Set up my HHKB JIS trilingual (English / Korean / Japanese) keyboard configuration on this
machine, following the hhkb-ekj-trilingual repo pinned at tag v1.5.1.

Source of truth (use this tag only — not main, not any other ref):
  Entry point: https://github.com/themrsung/hhkb-ekj-trilingual/blob/v1.5.1/AGENTS.md
  Get the repo:  git clone --depth 1 --branch v1.5.1 https://github.com/themrsung/hhkb-ekj-trilingual.git
  (If you can't clone, fetch each file from
   https://raw.githubusercontent.com/themrsung/hhkb-ekj-trilingual/v1.5.1/<path>, starting with
   AGENTS.md and then every file under docs/ that it links to.)
Clone into a temporary/scratch directory, read AGENTS.md and all of docs/ in full, and follow them.

Before anything else, ask me this one question on its own, word for word, and wait for my answer:

  Which layout is your HHKB, or whatever keyboard you are setting up?
  - US ANSI (no arrow keys, the classic "HHKB" look)
  - JIS (arrow keys on the bottom-right, big enter key spanning two rows, Yen (¥) key directly
    left of Backspace (BS))
  - Not sure, help me figure it out.

Then, depending on my answer:
  - If US ANSI: this repo is for JIS only. Point me to
    https://github.com/themrsung/hhkb-us-bilingual instead and stop, unless I explicitly ask to
    continue with this repo anyway (then warn me that the ろ, ¥, 英数 and かな keys it depends
    on don't exist on ANSI).
  - If JIS: continue below.
  - If not sure: help me tell them apart using the features listed above. Ask me to look at the
    board, or to send a photo if I can. You may also check which keyboard is connected (for
    example with Karabiner's karabiner_cli --list-connected-devices, or System Information),
    but the model name alone doesn't prove the layout, so confirm with me before going on. Then
    follow the US ANSI or JIS branch.

Before changing anything:
1. Check the OS.
   - macOS: continue below.
   - Windows 11: Windows is supported experimentally, for the HHKB Studio JIS only, with a
     compromise. Korean is handed off to the Microsoft Korean IME, which types US ANSI symbols.
     English is Japanese with the IME off, so its symbols match the JIS keycaps. Japanese is the
     Microsoft Japanese IME with kana typing. Explain this to me, and ask whether to continue. If
     I do, follow the Windows section of AGENTS.md and docs/windows/ instead of the macOS steps
     below. Ask before installing anything or changing any system setting.
   - Anything else: tell me the interactive setup isn't supported here and ask whether to continue
     anyway. Continue only if I confirm, adapting where you can and telling me what doesn't carry
     over.
2. Ask me, in one batch, and show the default for each:
   a. Which JIS keyboard model(s) I have. The repo is built for the HHKB Professional HYBRID
      Type-S and the HHKB Studio, both JIS layout (one or both). Other JIS HHKBs may need
      different USB IDs.
   b. Which input source the HHKB key should switch to (default: Korean 2-Set).
   c. Where 英数 and かな should go (default: English ABC / Japanese with Kana typing).
   d. The "Move focus to next window" shortcut (default: Option+Tab).
   e. Whether to keep the extras: ろ → \ outside Japanese, Fn+P → Play/Pause,
      Fn+I → Cmd+Shift+4 (default: all on).
   If I just say "defaults", use the repo defaults for everything.
3. Look at what is already installed and configured (Karabiner-Elements, Hammerspoon,
   ~/.config/karabiner/karabiner.json, ~/.hammerspoon/init.lua, enabled input sources) and tell
   me what you'll add or change. Back up any file before editing it. Merge, never overwrite my
   other rules.

Then do the setup on your own in the order AGENTS.md gives. Pause only for things I must do
by hand (DIP switches — which must be set BEFORE the keymap tool keymap — the keymap tool
itself, approving system extensions and permissions, and pressing keys for the checks). Give me
exact click-by-click instructions for those and wait for me to confirm. Finish by going through
the checklist in docs/06-verification.md with me, then summarize what changed and where the
backups are.
```

To set it up by hand instead, follow [`AGENTS.md`](AGENTS.md), then `docs/01` → `docs/06` in
order (plus `docs/07` for a Studio, and `docs/08` to migrate an older Studio setup to F18). On
Windows, follow `docs/windows/01` → `03` instead. The exact Karabiner JSON, Hammerspoon Lua and
AutoHotkey script are in the docs. The only
config file is the HHKB Studio keymap export
([`docs/windows/hhkb-studio-setting-all.hks`](docs/windows/hhkb-studio-setting-all.hks)), which you
can load in the Studio's keymap tool instead of setting each key by hand.

## Maintenance

**This repo will not be serviced.** No issues, no support, no guarantees. Fork it and use it as
you see fit.

The author may update it from time to time, but only to match their own preferences, and only
tested on HHKB models they own (currently the HYBRID Type-S JIS and the Studio JIS). Each update gets a new tag, so older prompts pinned to an earlier tag keep working as they
were.

## License

[CC0 1.0 Universal](LICENSE). Public domain dedication: no rights reserved, no attribution
required.
