# hhkb-ekj-trilingual

English / Korean / Japanese on a **JIS HHKB Professional HYBRID Type-S** under macOS, with one key
per language:

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

## How to use this repo

This repo is written for **coding agents** (Claude Code, Codex, etc.) to rebuild the setup on a
fresh Mac. Point your agent at [`AGENTS.md`](AGENTS.md) and ask it to set this up. Doing it by
hand works too: follow `docs/01` → `docs/06` in order.

There are no config files to install. The exact Karabiner JSON and Hammerspoon Lua are in the
docs. The defaults are the author's own setup. To send the HHKB key to a different input source
(QWERTZ, AZERTY, …), see [`docs/05-customization.md`](docs/05-customization.md).
