# 02 — macOS settings

Checked on macOS 27.0.1. Earlier releases with "System Settings" (macOS 13+) use the same panes.

## 1. Keyboard type = JIS

When the HHKB is first connected, macOS runs the **Keyboard Setup Assistant**. Pick **JIS
(Japanese)**. If it didn't appear, or the wrong type was picked:
System Settings → Keyboard → **Change Keyboard Type…** → follow the prompts → JIS.

Check:

```sh
defaults read /Library/Preferences/com.apple.keyboardtype
# the HHKB entry ("34-1278-0", and also "34-1278-15" if present) must be 42 (JIS)
```

Karabiner's virtual keyboard must be JIS too (`03-karabiner.md`). macOS stores that one as
`"593-1452-0" = 42`.

## 2. Input sources

System Settings → Keyboard → Text Input → Input Sources → **Edit…**

Enable exactly these three keyboard input sources (other non-keyboard ones like the Emoji &
Symbols palette can stay):

| Input source | ID | Notes |
| --- | --- | --- |
| **ABC** | `com.apple.keylayout.ABC` | English QWERTY. This is where 英数 lands. |
| **Korean → 2-Set Korean** (두벌식) | `com.apple.inputmethod.Korean.2SetKorean` | Target of the HHKB key |
| **Japanese → Kana** | `com.apple.inputmethod.Japanese` in bundle `com.apple.inputmethod.Kotoeri.KanaTyping` | Target of かな |

### Japanese must use Kana typing, not Romaji

In the Japanese input source settings:

- **Input** (入力方式) → **Kana** (かな入力). Don't use Romaji.
- **Input modes**: enable **Hiragana** only. **Turn off** "Romaji" / 英字 (and Katakana etc. if
  you like). With no Roman mode inside the Japanese IME, the **英数** key falls through to the
  **ABC** layout, so 英数 always means English QWERTY.

Check:

```sh
defaults read com.apple.inputmethod.Kotoeri JIMPrefTypingMethodKey   # 1 = Kana typing
defaults read com.apple.HIToolbox AppleEnabledInputSources
# must include: KeyboardLayout Name = ABC;
#               Input Mode = com.apple.inputmethod.Korean.2SetKorean;
#               Bundle ID = com.apple.inputmethod.Kotoeri.KanaTyping, Input Mode = com.apple.inputmethod.Japanese
# must NOT include a com.apple.inputmethod.Roman / Kotoeri "...Roman" input mode
```

Native macOS behavior used here (no Karabiner/Hammerspoon involved):

- **英数** (`japanese_eisuu`) → selects ABC.
- **かな** (`japanese_kana`) → selects the Japanese input source (Kana typing).

Leave "Use the Caps Lock key to switch to and from ABC" and similar options as they are. The
HHKB has no Caps Lock key anyway.

## 3. "Move focus to next window" → Option+Tab

The macOS default is **Cmd+`**. That's not usable on this setup: JIS has no dedicated `` ` ``
key, and the HHKB key is used for Korean. The repo default is **Option+Tab**. Users may choose a
different shortcut.

GUI: System Settings → Keyboard → **Keyboard Shortcuts…** → **Keyboard** →
"Move focus to next window" → double-click the shortcut → press **⌥⇥**.

CLI (same effect; symbolic hotkey 27 = move focus to next window; 48 = Tab keycode,
524288 = Option modifier mask):

```sh
defaults write com.apple.symbolichotkeys AppleSymbolicHotKeys -dict-add 27 \
  '{ enabled = 1; value = { parameters = (65535, 48, 524288); type = standard; }; }'
/System/Library/PrivateFrameworks/SystemAdministration.framework/Resources/activateSettings -u
```

If the shortcut doesn't work right after the CLI write, log out and back in, or set it once in the
GUI. Check:

```sh
plutil -extract AppleSymbolicHotKeys.27 json -o - ~/Library/Preferences/com.apple.symbolichotkeys.plist
# {"enabled":true,"value":{"type":"standard","parameters":[65535,48,524288]}}
```
