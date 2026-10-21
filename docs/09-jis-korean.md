# 09 — JIS Korean: the Korean input source (macOS)

**JIS Korean is the default Korean input source** on macOS, and the HHKB key's target. Apple's
**Korean 2-Set is no longer maintained** in this repo, also not next to JIS Korean on a MacBook
whose built-in keyboard isn't JIS. The last release that set up 2-Set is `v1.7.1`.

[JIS Korean](https://github.com/team-mirae/jis-korean) is a Korean 2-Set (두벌식) input method
built for JIS keyboards. Every standard 2-Set key still types what it always did. The JIS keys
2-Set leaves unused get extras:

- Double consonants and compound vowels on the number row: 1–5 = ㅃ ㅉ ㄸ ㄲ ㅆ, 6–0 = ㅟ ㅝ ㅚ ㅢ ㅘ.
  Shift types the digit.
- Korean punctuation on the symbol keys: `·` on `/`, `「」` on Shift+`[` / `]`, `—` on ¥.
- Japanese-IME-style F6–F10: F6 commits, F7 hanja, F8 a temporary escape to standard 2-Set with
  stock JIS symbols (Shift+F8 returns), F9 / F10 full- and half-width letters.
- `(` and `)` on the `^` key (Shift+¥ types `^`).
- Predictions with a suggestion list, learning from your choices (local only), and per-app switches
  in the input menu to stop learning or predicting in an app. Prediction starts off in terminals,
  file-transfer apps, Finder and other sensitive apps, and in Minecraft.
- Keyboard Viewer shows JIS Korean's legends while it's active.

Try it out. If you use an HHKB, you'll love it!

Checked with **JIS Korean at commit `90cc765`** (after `v0.2.0-preview`, not a tagged release) on macOS 27.0.1, Hammerspoon 1.1.1,
Karabiner-Elements 16.3.0, upgrading in place from 0.1.0 on a MacBook with a JIS built-in keyboard. The HHKB checks below were
first done at `a1a38e4` with the HYBRID Type-S JIS and the HHKB Studio JIS over Bluetooth. **Windows** uses JIS Korean too,
since `v1.9.0`: [`windows/04-jis-korean.md`](windows/04-jis-korean.md).

Where JIS Korean plugs into the other steps:

| Step | With JIS Korean |
| --- | --- |
| Input source (`02-macos.md`) | **JIS Korean**, installed from source (step 1 below), added by the user (step 2) |
| Karabiner rule C (`03-karabiner.md`) | skipped for `ja` **and JIS Korean** (step 3) |
| Hammerspoon (`04-hammerspoon.md`) | `KOREAN = "me.sjun.inputmethod.JISKorean.2Set"` (step 4) |
| Function keys | **F1, F2, etc. as standard function keys** (step 5) |

The rest of the setup doesn't depend on it: the always-select plus guarded-retry structure, bare
F18, 英数 → ABC, かな → Kana, and JIS keyboard types for each HHKB and for Karabiner's virtual
keyboard. JIS Korean reads physical key codes, and ¥ and ろ only arrive correctly with a JIS
keyboard type.

## 1. Install JIS Korean

Pinned to commit **`90cc765`**. It's built from source (it's only ad-hoc signed, so a downloaded
build would be blocked by Gatekeeper). It needs the Xcode Command Line Tools and Rust with both Mac
targets:

```sh
xcode-select --install                     # if the Command Line Tools aren't there yet
brew install rustup                        # or the installer from rustup.rs
export PATH="$(brew --prefix rustup)/bin:$PATH"   # Homebrew's rustup is keg-only, see below
rustup default stable
rustup target add aarch64-apple-darwin x86_64-apple-darwin

git clone https://github.com/team-mirae/jis-korean.git
git -C jis-korean checkout 90cc765
make -C jis-korean/mac install             # builds JISKorean.app, copies it to ~/Library/Input Methods, registers it
```

Ask the user before installing Rust or the Command Line Tools.

- **Homebrew's `rustup` is keg-only and has no `rustup-init`** (Homebrew 7.0.7, rustup 1.29.1).
  Its `rustup`, `cargo` and `rustc` live in `$(brew --prefix rustup)/bin`, which isn't on PATH.
  `rustup default stable` installs the toolchain. The Makefile finds cargo with
  `command -v cargo` (falling back to `~/.cargo/bin/cargo`), so keep that PATH for `make`, and
  for any later rebuild or `make … uninstall`. With the rustup.rs installer, `~/.cargo/bin` is used instead.
- `make install` prints `TISFileInterrogator … old data invalid` and "duplicate keyboard layout
  identifier … replaced" lines about unrelated system layouts. Harmless. It ends with
  "Registered (with its Keyboard Viewer layout).", after "Re-enabled the input source." on an
  upgrade.
- **Upgrading or reinstalling: run `make install` only, never `make uninstall` first.** If the app is
  missing for a while (an uninstall followed by a rebuild), apps that were already running can't
  start JIS Korean again until they're restarted: the menu shows JIS Korean but Latin letters come
  out. Quit and reopen the app, or log out and back in. Details:
  [`docs/troubleshooting/macos-input-method-not-launching.md`](https://github.com/team-mirae/jis-korean/blob/90cc765/docs/troubleshooting/macos-input-method-not-launching.md)
  in the JIS Korean repo.
- **Upgrading from 0.1.0:** the input source is now an input mode, `me.sjun.inputmethod.JISKorean.2Set`.
  Update the Hammerspoon constant (step 4) and rule C's condition (step 3). If JIS Korean isn't in
  the input menu after the upgrade, remove it in Input Sources and add it again.

## 2. Input sources

The **user** adds it: System Settings → Keyboard → Text Input → Input Sources → Edit… → **+** →
Korean → **JIS Korean**. If it isn't in the list, log out and back in once.

Enable exactly ABC, Japanese – Kana and JIS Korean (its JIS mode, `….JISKorean.2Set`; leave its US ANSI
and UK ISO modes off). Don't enable 2-Set Korean next to it, on any
Mac. If it's already enabled, remove both the 2-Set Korean mode and its parent "Korean" entry
(Edit… → select → **−**), or an empty "Korean" item can linger. Ask the user first: on a Mac set
up with an earlier release, 2-Set may be what they type with on the built-in keyboard (next
section).

Check:

```sh
defaults read com.apple.inputsources AppleEnabledThirdPartyInputSources
# Bundle ID = "me.sjun.inputmethod.JISKorean"; InputSourceKind = "Keyboard Input Method"
```

Expect two entries, the input method and its mode (`"Input Mode" =
"me.sjun.inputmethod.JISKorean.2Set"`). Third-party input methods are recorded there, **not** in
`com.apple.HIToolbox AppleEnabledInputSources`. On macOS 27.0.1, JIS Korean shows up in HIToolbox only under
`AppleInputSourceHistory`, so grepping `AppleEnabledInputSources` gives a false "not enabled".

`hs.keycodes.methods(true)` lists it as `me.sjun.inputmethod.JISKorean.2Set`.

### Non-JIS built-in keyboard

On a MacBook with a US (ANSI) or ISO built-in keyboard, the usual setup lets Karabiner **ignore**
the built-in keyboard (`03-karabiner.md`, "Non-JIS built-in keyboard"). macOS then reads it as a
plain ANSI/ISO board, and in Korean it types through JIS Korean as well. JIS Korean reads physical
JIS key codes, so it doesn't fit that keyboard: ㅃ ㅉ ㄸ ㄲ ㅆ / ㅟ ㅝ ㅚ ㅢ ㅘ take over the number
row (`1` types ㅃ, and Shift+`1` types `1` instead of `!`), and the symbol keys get Korean
punctuation the keycaps don't show. Letters type standard 2-Set.

This is accepted: 2-Set Korean is no longer kept next to JIS Korean for the built-in keyboard (it
was, from `v1.7.0` to `v1.7.1`). Tell the user, and don't add 2-Set back. See
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
      { "input_source_id": "^me\\.sjun\\.inputmethod\\.JISKorean(\\..*)?$" }
    ]
  }
]
```

The pattern covers the input mode (`….JISKorean.2Set`). With only `…JISKorean$`, as written for
0.1.0, ろ types Option+¥ in JIS Korean.

On an existing install, back up `karabiner.json`, then add the entry idempotently (it also replaces
the 0.1.0 pattern):

```sh
KJ=~/.config/karabiner/karabiner.json
cp "$KJ" "$KJ.bak-$(date +%Y%m%d%H%M%S)"
jq '(.profiles[] | select(.selected==true) | .complex_modifications.rules[]
     | select(.description | startswith("JIS ろ")) | .manipulators[].conditions[]
     | select(.type=="input_source_unless") | .input_sources)
    |= (map(select((.input_source_id? // "") | startswith("^me\\.sjun\\.inputmethod\\.JISKorean") | not))
        + [{"input_source_id": "^me\\.sjun\\.inputmethod\\.JISKorean(\\..*)?$"}])' \
  "$KJ" > "$KJ.tmp" && mv "$KJ.tmp" "$KJ" && chmod 600 "$KJ"
```

## 4. Hammerspoon: F18 → JIS Korean

The `~/.hammerspoon/init.lua` block in `04-hammerspoon.md` already targets JIS Korean. On a Mac
set up with an earlier release, change only the constant:

```lua
-- to_korean key → JIS Korean. F18 comes from the HHKB Studio's keymap, or from Karabiner
-- (HYBRID Type-S HHKB key, remapped from `)
local KOREAN = "me.sjun.inputmethod.JISKorean.2Set"
```

Keep everything else, including the guarded retry, and reload from the menu bar.
`hs -c 'hs.reload()'` doesn't return, because the reload drops the connection the CLI waits on.
If an agent has to reload from a script, put a timeout on it:
`perl -e 'alarm 3; exec @ARGV' hs -c 'hs.reload()'`. It then exits 69 with `message port was
invalidated`, but the reload goes through (the Hammerspoon console shows `Enabled hotkey F18`).
Put the same timeout on scripted `hs -c` source selections: one hung for minutes while input
sources were being edited in System Settings. Hammerspoon itself stayed responsive.

```sh
hs -c 'hs.keycodes.currentSourceID()'     # after pressing the HHKB key → me.sjun.inputmethod.JISKorean.2Set
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

`06-verification.md` checks JIS Korean in rows 5–7 and 11. Then run these:

| # | Action (in JIS Korean) | Expected |
| --- | --- | --- |
| J1 | 👤 `1`, then `k` | 빠 (dedicated ㅃ); Shift+`1` types `1` |
| J2 | 👤 `gks`, then F6 | 한 is committed (no underline left) |
| J3 | 👤 `gkrry` (학교), then F7, then `1` | 學校 |
| J4 | 👤 F8, then `1` | `1` (stock JIS). Shift+F8, then `1` → ㅃ again |
| J5 | 👤 `gks`, then F10 | `gks`. F10 again → `GKS` |
| J6 | 👤 `^`, then Shift+`^` | `()`. Shift+¥ → `^` |
| J7 | 👤 Input menu → Show Keyboard Viewer | Korean legends (ㅂ on Q, `(` on `^`) |

Don't verify by typing in an app where Karabiner remaps letter keys (for example, a rule scoped to
TextEdit): Karabiner changes the keys before JIS Korean sees them.

## Known limits of JIS Korean

- It's built from source and ad-hoc signed.
- Its menu bar badge (한) is a shade lighter than Apple's Korean badges in System Settings.
- Behavior can change between JIS Korean versions. This page is checked against commit `90cc765`.

## Korean 2-Set (no longer maintained)

This repo no longer sets up or tests Apple's Korean 2-Set. A user who wants it anyway can use
release `v1.7.1`, the last one that did, or on this release: set `KOREAN` back to
`com.apple.inputmethod.Korean.2SetKorean`, add 2-Set Korean in Input Sources and remove JIS Korean.
You can leave rule C's extra condition in place; it does nothing without JIS Korean. To remove
the app too: `make -C jis-korean/mac uninstall`. Tell them it's untested here.
