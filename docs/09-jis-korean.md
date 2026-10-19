# 09 — Optional: JIS Korean instead of Korean 2-Set (macOS)

**Korean 2-Set stays the default.** This page is the alternative for users who pick it.

[JIS Korean](https://github.com/team-mirae/jis-korean) is a Korean 2-Set (두벌식) input method
built for JIS keyboards. Every standard 2-Set key still types what it always did. The JIS keys
2-Set leaves unused get extras:

- Double consonants and compound vowels on the number row: 1–5 = ㅃ ㅉ ㄸ ㄲ ㅆ, 6–0 = ㅟ ㅝ ㅚ ㅢ ㅘ.
  Shift types the digit.
- Korean punctuation on the symbol keys: `·` on `/`, `「」` on Shift+`[` / `]`, `—` on ¥.
- Japanese-IME-style F6–F10: F6 commits, F7 hanja, F8 a temporary escape to standard 2-Set with
  stock JIS symbols (Shift+F8 returns), F9 / F10 full- and half-width letters.

Try it out. If you use an HHKB, you'll love it!

Checked with **JIS Korean 0.1.0** on macOS 27.0.1, Hammerspoon 1.1.1, Karabiner-Elements 16.3.0,
the HYBRID Type-S JIS, and the HHKB Studio JIS over Bluetooth. **macOS only**: there's no Windows build yet, so the Windows setup
(`docs/windows/`) keeps the Microsoft Korean IME.

The default setup carries over unchanged except for four steps:

| Step | Default (2-Set) | With JIS Korean |
| --- | --- | --- |
| Input source (`02-macos.md`) | Korean → 2-Set Korean | **JIS Korean**, installed from source (step 1 below) |
| Karabiner rule C (`03-karabiner.md`) | skipped for `ja` | skipped for `ja` **and JIS Korean** (step 3) |
| Hammerspoon (`04-hammerspoon.md`) | `KOREAN = "com.apple.inputmethod.Korean.2SetKorean"` | `KOREAN = "me.sjun.inputmethod.JISKorean"` (step 4) |
| Function keys | as you like | **F1, F2, etc. as standard function keys** (step 5) |

Everything else stays as it is: the always-select plus guarded-retry structure, bare F18, 英数 → ABC,
かな → Kana, and JIS keyboard types for each HHKB and for Karabiner's virtual keyboard. JIS Korean
reads physical key codes, and ¥ and ろ only arrive correctly with a JIS keyboard type.

## 1. Install JIS Korean 0.1

Pinned to tag **`v0.1.0`**. It's built from source (it's only ad-hoc signed, so a downloaded
build would be blocked by Gatekeeper). It needs the Xcode Command Line Tools and Rust with both Mac
targets:

```sh
xcode-select --install                     # if the Command Line Tools aren't there yet
brew install rustup                        # or the installer from rustup.rs
export PATH="$(brew --prefix rustup)/bin:$PATH"   # Homebrew's rustup is keg-only, see below
rustup default stable
rustup target add aarch64-apple-darwin x86_64-apple-darwin

git clone --depth 1 --branch v0.1.0 https://github.com/team-mirae/jis-korean.git
make -C jis-korean/mac install             # builds JISKorean.app, copies it to ~/Library/Input Methods, registers it
plutil -extract CFBundleShortVersionString raw ~/Library/Input\ Methods/JISKorean.app/Contents/Info.plist   # 0.1.0
```

Ask the user before installing Rust or the Command Line Tools.

- **Homebrew's `rustup` is keg-only and has no `rustup-init`** (Homebrew 7.0.7, rustup 1.29.1).
  Its `rustup`, `cargo` and `rustc` live in `$(brew --prefix rustup)/bin`, which isn't on PATH.
  `rustup default stable` installs the toolchain. The Makefile finds cargo with
  `command -v cargo` (falling back to `~/.cargo/bin/cargo`), so keep that PATH for `make`, and
  for any later rebuild or `make … uninstall`. With the rustup.rs installer, `~/.cargo/bin` is used instead.
- `git clone --branch v0.1.0` warns `refs/tags/v0.1.0 … is not a commit!`. That's harmless:
  v0.1.0 is an annotated tag and git checks out the commit it points to (`28d7d8e`, "Release 0.1.0").
- `make install` prints `TISFileInterrogator … old data invalid` and "duplicate keyboard layout
  identifier … replaced" lines about unrelated system layouts. Also harmless. It ends with "Registered.".

## 2. Input sources

The **user** adds it: System Settings → Keyboard → Text Input → Input Sources → Edit… → **+** →
Korean → **JIS Korean**. If it isn't in the list, log out and back in once.

Enable exactly ABC, Japanese – Kana and JIS Korean. Don't enable 2-Set Korean next to it. If it's
already enabled, remove both the 2-Set Korean mode and its parent "Korean" entry (Edit… → select →
**−**), or an empty "Korean" item can linger. **Exception:** a Mac whose built-in keyboard isn't
JIS keeps 2-Set Korean (next section).

Check:

```sh
defaults read com.apple.inputsources AppleEnabledThirdPartyInputSources
# Bundle ID = "me.sjun.inputmethod.JISKorean"; InputSourceKind = "Keyboard Input Method"
```

Third-party input methods are recorded there, **not** in `com.apple.HIToolbox
AppleEnabledInputSources`. On macOS 27.0.1, JIS Korean shows up in HIToolbox only under
`AppleInputSourceHistory`, so grepping `AppleEnabledInputSources` gives a false "not enabled".

`hs.keycodes.methods(true)` **won't** list JIS Korean either. It lists input modes only, and JIS
Korean is an input method without modes. Use the check above, or select it and read it back (step 4).

### Non-JIS built-in keyboard: keep 2-Set Korean too

On a MacBook with a US (ANSI) or ISO built-in keyboard, the usual setup lets Karabiner **ignore**
the built-in keyboard (`03-karabiner.md`, "Non-JIS built-in keyboard"). macOS then reads it as a
plain ANSI/ISO board. JIS Korean reads physical JIS key codes, so it doesn't fit that keyboard:
ㅃ ㅉ ㄸ ㄲ ㅆ / ㅟ ㅝ ㅚ ㅢ ㅘ take over the number row (`1` types ㅃ, and Shift+`1` types `1`
instead of `!`), the symbol keys get Korean punctuation the keycaps don't show, and ¥ and ろ don't exist.

So, when the Mac is also used without the HHKB:

- Enable **both** JIS Korean and 2-Set Korean (with its parent "Korean" entry). The input sources
  are ABC, Japanese – Kana, JIS Korean and 2-Set Korean.
- The **HHKB key** (F18 → Hammerspoon) always selects **JIS Korean**. Nothing else changes in step 4.
- The **built-in keyboard** keeps using macOS's own switching (Caps Lock toggles ABC, Ctrl+Space
  selects the previous source, Ctrl+Option+Space the next one) and types Korean in **2-Set**.
- The user adds JIS Korean in the same Edit… sheet that lists 2-Set. Removing 2-Set out of habit
  is easy because of the step above. If 2-Set was removed by accident,
  `hs.keycodes.currentSourceID("com.apple.inputmethod.Korean.2SetKorean")` returns `false` and the
  source doesn't change. Add 2-Set Korean back (+ → Korean → 2-Set Korean).

Checked on a MacBook with a US ANSI built-in keyboard (ignored by Karabiner) plus an HHKB Studio
over Bluetooth, macOS 27.0.1. Known catch: Caps Lock and Ctrl+Space return to the *most recently
used* source, which can be JIS Korean. See
[`../issues/jis-korean-non-jis-built-in.md`](../issues/jis-korean-non-jis-built-in.md).

## 3. Karabiner: keep rule C away from JIS Korean

Rule C (ろ → Option+¥) matches every source whose language isn't `ja`. JIS Korean's language is
`ko`, so without this change ろ would reach the input method as **Option+¥**. JIS Korean treats
Option as a shortcut: it commits the syllable being composed and lets the key through. JIS Korean
already types `\` on ろ and `_` on Shift+ろ by itself.

Use this condition list in rule C, both in the fresh-install JSON and after a merge:

```json
"conditions": [
  {
    "type": "input_source_unless",
    "input_sources": [
      { "language": "^ja$" },
      { "input_source_id": "^me\\.sjun\\.inputmethod\\.JISKorean$" }
    ]
  }
]
```

On an existing install, back up `karabiner.json`, then add the entry idempotently:

```sh
KJ=~/.config/karabiner/karabiner.json
cp "$KJ" "$KJ.bak-$(date +%Y%m%d%H%M%S)"
jq '(.profiles[] | select(.selected==true) | .complex_modifications.rules[]
     | select(.description | startswith("JIS ろ")) | .manipulators[].conditions[]
     | select(.type=="input_source_unless") | .input_sources)
    |= (if any(.[]; .input_source_id? == "^me\\.sjun\\.inputmethod\\.JISKorean$") then .
        else . + [{"input_source_id": "^me\\.sjun\\.inputmethod\\.JISKorean$"}] end)' \
  "$KJ" > "$KJ.tmp" && mv "$KJ.tmp" "$KJ" && chmod 600 "$KJ"
```

## 4. Hammerspoon: F18 → JIS Korean

In the `~/.hammerspoon/init.lua` block from `04-hammerspoon.md`, change only the constant:

```lua
-- to_korean key → JIS Korean. F18 comes from the HHKB Studio's keymap, or from Karabiner
-- (HYBRID Type-S HHKB key, remapped from `)
local KOREAN = "me.sjun.inputmethod.JISKorean"
```

Keep everything else, including the guarded retry, and reload from the menu bar.
`hs -c 'hs.reload()'` doesn't return, because the reload drops the connection the CLI waits on.
If an agent has to reload from a script, put a timeout on it:
`perl -e 'alarm 3; exec @ARGV' hs -c 'hs.reload()'`. It then exits 69 with `message port was
invalidated`, but the reload goes through (the Hammerspoon console shows `Enabled hotkey F18`).
Put the same timeout on scripted `hs -c` source selections: one hung for minutes while input
sources were being edited in System Settings. Hammerspoon itself stayed responsive.

```sh
hs -c 'hs.keycodes.currentSourceID()'     # after pressing the HHKB key → me.sjun.inputmethod.JISKorean
```

**The ~1% lost switch** (`04-hammerspoon.md`, "Known issue") wasn't seen with JIS Korean in
day-to-day use on the owner's Mac, but it hasn't been measured. Keep the known-issue section and the
retry guard as they are until a few hundred switches with typing read back say otherwise.

## 5. Function keys

JIS Korean uses F6–F10. They must arrive as F-keys, not media keys: System Settings → Keyboard →
Keyboard Shortcuts… → Function Keys → **Use F1, F2, etc. keys as standard function keys** = on.

```sh
defaults read -g com.apple.keyboard.fnState    # 1
```

On the HYBRID Type-S, F6–F10 are Fn+6…0. On the Studio, they're in its Fn layer. Check them in
Karabiner-EventViewer.

## 6. Verification

Run `06-verification.md` with **JIS Korean** wherever it says Korean 2-Set (rows 5–7), plus:

| # | Action (in JIS Korean) | Expected |
| --- | --- | --- |
| 11 | 👤 ろ / Shift+ろ | `\` / `_`. No `¥` caveat: JIS Korean types these itself |
| J1 | 👤 `1`, then `k` | 빠 (dedicated ㅃ); Shift+`1` types `1` |
| J2 | 👤 `gks`, then F6 | 한 is committed (no underline left) |
| J3 | 👤 `gkrry` (학교), then F7, then `1` | 學校 |
| J4 | 👤 F8, then `1` | `1` (stock JIS). Shift+F8, then `1` → ㅃ again |
| J5 | 👤 `gks`, then F10 | `gks`. F10 again → `GKS` |

Don't verify by typing in an app where Karabiner remaps letter keys (for example, a rule scoped to
TextEdit): Karabiner changes the keys before JIS Korean sees them.

## Known limits of JIS Korean 0.1

- **No hanja candidate window on macOS.** F7 shows the selected candidate inline. F7 or ↓ moves to
  the next one, and 1–9 pick, but the list isn't drawn yet.
- It's built from source and ad-hoc signed.
- Behavior can change between JIS Korean versions. This page is checked against 0.1.0 only.

## Switching back to 2-Set

Change `KOREAN` back to `com.apple.inputmethod.Korean.2SetKorean`, add 2-Set Korean in Input Sources
and remove JIS Korean. You can leave rule C's extra condition in place; it does nothing without JIS
Korean. To remove the app too: `make -C jis-korean/mac uninstall`.
