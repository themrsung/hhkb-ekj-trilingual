# Inter-OS muscle memory

**Applies to:** anyone using the same HHKB on both macOS and Windows.
**Status:** not caused by this setup. Expected, and it takes practice.

## Symptom

After switching between a Mac and a Windows PC, you reach for the wrong modifier or shortcut:
Option vs Alt, Command vs Win/Ctrl, Cmd+C vs Ctrl+C, Option+Tab vs Alt+Tab, and so on.

## Cause

macOS and Windows use different modifiers for the same actions. The keymap profiles already put
the keys in the same physical places on both: Option ↔ Alt, Command ↔ Win (Profile1 vs Profile 4,
`../docs/windows/01-hardware.md`). What still differs is which key each OS expects for a shortcut,
for example Command+C on macOS against Ctrl+C on Windows. No keyboard remap fixes that without
breaking one OS's own conventions.

The language keys are consistent on both OSes: the same physical key goes to Korean, English or
Japanese. Only the shortcuts differ.

## What to do

Practice. Optionally, remap individual shortcuts per OS, but this repo deliberately doesn't. Each
OS keeps its native shortcuts.
