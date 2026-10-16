# 08 — Migrating the HHKB Studio from the `` ` `` remap to F18 in the keymap

**Applies to the HHKB Studio only.** Before v1.4.0, the Studio's "HHKB key" (the 半/全 position)
sent `grave_accent_and_tilde`, and a Karabiner simple modification turned that into F18. From
v1.4.0 on, the key is set to **F18 in the HHKB Studio Keymap Tool**, so the Studio sends F18 by
itself. Hammerspoon stays the same, and Karabiner stops remapping that key on the Studio.

**The HYBRID Type-S doesn't migrate.** Its HHKB Keymap Tool can't assign F18, so the HYBRID keeps
the Karabiner `` ` `` → F18 remap. Leave its device entry (product ID `34`) alone.

Why bother: the key no longer depends on Karabiner. A Studio connection that Karabiner doesn't
grab, or a Mac without Karabiner, still sends F18. You only need Hammerspoon for the switch.

## 1. Check where you are

```sh
jq -c '.profiles[] | select(.selected) | .devices[]
       | select(.identifiers.vendor_id == 1278 and .identifiers.product_id == 21)
       | {ids: .identifiers, mods: [.simple_modifications[]?.from.key_code]}' \
   ~/.config/karabiner/karabiner.json
```

- Old setup: the Studio entries list `grave_accent_and_tilde` (and `pause`).
- Already migrated: they list only `pause`.

## 2. Keymap tool: set the key to F18 (on every layer)

👤 Needs the user's hands. Connect the Studio over **USB** and open the **HHKB Studio Keymap Tool**.

1. Pick the profile the user uses (the owner uses **Profile1**).
2. On the **Standard** layer, set the key in the 半/全 position (bottom row, between Fn1 and
   Opt) to **F18**.
3. Do the same on **Fn1**, **Fn2** and **Fn3**. On every layer the key should read F18 (compare
   with the screenshots in `07-hhkb-studio.md`).
4. Click **Write to HHKB**.

Changing only this key leaves the rest of the keymap alone. Flipping SW4 later doesn't undo it
either (`07-hhkb-studio.md`).

## 3. Karabiner: remove the `` ` `` remap from the Studio entries

Back up, then drop only the `grave_accent_and_tilde` modification from the Studio's device
entries (USB `is_keyboard`, and the Bluetooth `is_keyboard` + `is_pointing_device` one).
Fn1+P → Play/Pause stays, and the HYBRID Type-S entry (product `34`) isn't touched:

```sh
KJ=~/.config/karabiner/karabiner.json
cp "$KJ" "$KJ.bak-$(date +%Y%m%d%H%M%S)"
jq '.profiles |= map(if .selected then .devices |= map(
      if .identifiers.vendor_id == 1278 and .identifiers.product_id == 21
         and (.identifiers.is_keyboard // false)
      then .simple_modifications |= map(select(.from.key_code != "grave_accent_and_tilde"))
      else . end) else . end)' "$KJ" > "$KJ.tmp" && mv "$KJ.tmp" "$KJ"
```

Rerunning the merge script in `03-karabiner.md` (v1.4.0 or later) gives the same result, because
it writes the Studio entries without the `` ` `` remap.

**Order:** do step 2 first, or at least do steps 2 and 3 together. If you remove the remap while
the Studio still sends `` ` ``, that key types `` ` `` until the keymap is written. If you write the
keymap first and leave the remap in, nothing breaks: the Studio sends F18 and the `` ` `` remap
just never fires. Remove it anyway, so the config matches the docs.

## 4. Hammerspoon

Nothing to change. It binds bare F18 (`04-hammerspoon.md`), whichever keyboard sends it. If your
`init.lua` comment says the key is "remapped to F18 by Karabiner", you can update it, but that's
only a comment.

## 5. Verify

On the Studio, over USB and over Bluetooth if you use both:

| Check | Expected |
| --- | --- |
| Karabiner-EventViewer, press the F18 key | `f18` |
| From ABC, press F18 | Korean 2-Set, overlay shown once |
| Fn1+P with music playing | Play/Pause (still Karabiner) |
| Hold Fn1 or Fn2 and press F18 | Still `f18` (the key is F18 on every layer) |
| HYBRID Type-S HHKB key | Still `f18` → Korean (its `` ` `` remap is untouched) |

Then run the rest of `06-verification.md` as usual.

## Rolling back

Restore the backup (`cp "$KJ.bak-<timestamp>" "$KJ"`), and set the key back to 半/全 in the keymap
tool on every layer. That is the pre-v1.4.0 setup: the Studio sends `` ` `` and Karabiner remaps
it to F18.
