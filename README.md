# hhkb-ekj-trilingual

> [!WARNING]
> **This entire repo was AI-generated**: the docs, the Karabiner rules, the Hammerspoon Lua, and
> this README. If you don't want AI-written code on your machine, **skip the interactive setup**
> and read the files yourself before applying anything.

English / Korean / Japanese on a **JIS HHKB Professional HYBRID Type-S** under **macOS**, with
one key per language:

| Key | Goes to |
| --- | --- |
| **HHKB** (logo key) | Korean 2-Set (always, even if already Korean) |
| **英数** (Romaji) | English ABC (QWERTY) |
| **かな** (Kana) | Japanese, Kana typing |

Plus: ろ types `\` (Shift+ろ → `_`) outside Japanese, Fn+P = Play/Pause, Fn+I = Cmd+Shift+4, and
"Move focus to next window" on **Option+Tab**.

Bottom row (remapped with the HHKB Keymap Tool, Mac mode = DIP switches 1 and 5 ON, rest OFF,
**set before writing the keymap**):

```
Fn | HHKB | Option | Command | 英数 | Space | かな | Command | Option | Fn | ← | ↓ | →
```

Control sits left of A.

Built with [Karabiner-Elements](https://karabiner-elements.pqrs.org/) and
[Hammerspoon](https://www.hammerspoon.org/).

## Scope

- **macOS only.** This repo was designed for macOS. The interactive setup is **not officially
  supported on other operating systems**. The agent warns you and asks before going on, but it
  won't refuse if you confirm.
- **JIS only.** Built and tested on the HHKB Professional HYBRID Type-S **JIS**. **US ANSI
  support is not built in.** The ろ, ¥, 英数 and かな keys this setup depends on don't exist on
  ANSI boards.
- The defaults are the author's own preferences. To send the HHKB key to a different input
  source (QWERTZ, AZERTY, …), see [`docs/05-customization.md`](docs/05-customization.md).

## Interactive setup (for a coding agent)

Paste the prompt below into a coding agent running on the Mac you want to set up (Claude Code,
Codex, etc.). It asks you a few questions, then does the rest on its own. It stops only for steps
that need your hands: DIP switches, the HHKB Keymap Tool, macOS permission prompts, and key-press
checks.

The prompt is **pinned to the release tag `v1.0.0`**. The agent reads the docs exactly as they
were at that release, even if `main` changes later.

```text
Set up my HHKB JIS trilingual (English / Korean / Japanese) keyboard configuration on this
machine, following the hhkb-ekj-trilingual repo pinned at tag v1.0.0.

Source of truth (use this tag only — not main, not any other ref):
  Entry point: https://github.com/themrsung/hhkb-ekj-trilingual/blob/v1.0.0/AGENTS.md
  Get the repo:  git clone --depth 1 --branch v1.0.0 https://github.com/themrsung/hhkb-ekj-trilingual.git
  (If you can't clone, fetch each file from
   https://raw.githubusercontent.com/themrsung/hhkb-ekj-trilingual/v1.0.0/<path>, starting with
   AGENTS.md and then every file under docs/ that it links to.)
Clone into a temporary/scratch directory, read AGENTS.md and all of docs/ in full, and follow them.

Before changing anything:
1. Check the OS. This repo is designed for macOS only. If this isn't macOS, tell me the
   interactive setup isn't officially supported here and ask whether to continue anyway. Continue
   only if I confirm, adapting where you can and telling me what doesn't carry over.
2. Ask me, in one batch, and show the default for each:
   a. Which keyboard I have. The repo is built for the HHKB Professional HYBRID Type-S, JIS
      layout. Other JIS HHKBs may need different USB IDs. US ANSI is not built in. If I have
      ANSI, warn me and ask whether to continue.
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
by hand (DIP switches — which must be set BEFORE the HHKB Keymap Tool keymap — the Keymap Tool
itself, approving system extensions and permissions, and pressing keys for the checks). Give me
exact click-by-click instructions for those and wait for me to confirm. Finish by going through
the checklist in docs/06-verification.md with me, then summarize what changed and where the
backups are.
```

To set it up by hand instead, follow [`AGENTS.md`](AGENTS.md), then `docs/01` → `docs/06` in
order. There are no config files to install. The exact Karabiner JSON and Hammerspoon Lua are in
the docs.

## Maintenance

**This repo will not be serviced.** No issues, no support, no guarantees. Fork it and use it as
you see fit.

The author may update it from time to time, but only to match their own preferences, and only
tested on HHKB models they own. The plan is to add the **HHKB Studio JIS** (not ANSI) at some
point. Each update gets a new tag, so older prompts pinned to an earlier tag keep working as they
were.

## License

[CC0 1.0 Universal](LICENSE). Public domain dedication: no rights reserved, no attribution
required.
