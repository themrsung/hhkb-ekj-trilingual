# FKeyIME: apps keep typing English after installing

**Applies to:** Windows, Path B (`../docs/windows/04-fkeyime.md`).
**Status:** known; restart the apps.

## Symptom

After installing, updating or reinstalling FKeyIME, F18 and F20 seem to work (the taskbar shows
FKeyIME KO or JA), but an app keeps typing English letters: US symbols in Korean, JIS symbols in
Japanese. Windows' plain keyboard layouts are typing, not FKeyIME.

## Cause

Apps that were already running when FKeyIME was installed may never load it. Measured on the
owner's PC (2026-10-17): Notepad, Claude, Opera, Edge and Windows Terminal, running since before
several reinstalls, switched to FKeyIME's language but never loaded its DLL. Freshly started apps
loaded it at once, and Notepad typed Korean and kana after a restart.

## What to do

Restart the app, or sign out and back in to restart them all.
