# Security

## Why this app needs a keyboard hook and a mouse hook

The gesture is "hold a key, then scroll" - Windows has no API to ask "is key X currently
held down" without either polling constantly (expensive, laggy) or installing a
low-level hook that gets notified on every key event. This app uses the second
approach: `WH_KEYBOARD_LL` to track whether the modifier key is physically held, and
`WH_MOUSE_LL` to see wheel ticks and (if configured) mouse side-button state while it is.

**What the keyboard hook actually looks at:** only whether the event matches the one
physical key (by hardware scan code) or virtual key currently configured as the
modifier. Every other keystroke is passed through to Windows unchanged and is never
logged or transmitted. Since 2.9 the hook also keeps, in memory only, how many characters
you have typed since the caret last moved and which of them were spaces - never which keys
they were - so the gesture can fix the text you just typed when nothing is selected (see
[PRIVACY.md](PRIVACY.md)); a click, Enter, Tab, an arrow key, a shortcut or another window
discards it. The *content* of your typing is never visible to this
code except in the one case the whole app is for: text you explicitly select and
trigger the gesture on, which briefly passes through the clipboard (see below) before
being pasted back.

**What the mouse hook actually looks at:** wheel direction/magnitude and side-button
state, only while the modifier is held. It does not log clicks, movement, or track
cursor position for any other purpose.

Both hook callbacks swallow events only in the specific window where the gesture is
genuinely held and firing; everything else is passed to `CallNextHookEx` immediately so
other applications' input is not delayed or altered.

## What happens to your text when the gesture fires

1. The current clipboard contents are copied out in full and held in memory (not
   written to disk) so they can be restored afterward.
2. Your selection is copied to the clipboard via a synthetic Ctrl+C, read back, and run
   through the local conversion logic (no network call - see below).
3. If conversion changed anything, the result is written to the clipboard and pasted
   back via a synthetic Ctrl+V.
4. The original clipboard contents are restored - unless something else (you, or
   another application) wrote to the clipboard in the meantime, detected via the
   clipboard's sequence number, in which case the restore is skipped instead of
   silently overwriting whatever is there now.

Clipboard writes this app makes are tagged with the Windows-documented formats that
opt text out of clipboard history (Win+V), Cloud Clipboard roaming to your other
devices, and the convention most third-party clipboard-manager utilities respect
(`CanIncludeInClipboardHistory`, `CanUploadToCloudClipboard`,
`ExcludeClipboardContentFromMonitorProcessing`) - none of this app's automated
clipboard writes are something you chose to keep around.

## What is logged, and where

By default, the log (`%LOCALAPPDATA%\LayFlip\layflip.log`) records only
event types and character counts - e.g. `Fixed selection (6 -> 6 chars)` - never the
actual text. A "Подробный лог" (verbose logging) option exists in the tray menu for
debugging a specific conversion problem; turning it on logs full text content to that
same local file until turned back off. The log file is local only, capped at 512 KB
with one rollover (`layflip.log.old`), and is never transmitted anywhere.

Settings (`%APPDATA%\LayFlip\settings.ini`) and the list of words you've
told the app never to "fix" again (`%APPDATA%\LayFlip\exceptions.txt`) are
plain local files, never transmitted.

## What this app does not do

- **No automatic network access of any kind.** The program contains no socket, HTTP client
  or background update-check code. You can verify this without the source: block it in
  Windows Firewall or watch its connections in Resource Monitor (Network tab) - there are
  none, and blocking it changes nothing. The one exception, and the only network-adjacent thing this app can ever do,
  is the tray menu's "Проверить обновления" - it opens the project's releases page in
  your default browser, and only in direct response to you clicking it; nothing is ever
  fetched, checked, or reported automatically or on a schedule.
- **No telemetry, analytics, or crash reporting** to any external service.
- **No persistence mechanism of its own** beyond the optional startup shortcut the
  installer creates when its "Launch automatically when Windows starts" checkbox is
  ticked (a normal Windows startup shortcut) - the app does not modify the registry,
  schedule tasks, or install services on its own.
- **Does not read or act on password fields.** The focused UI element is checked via
  UI Automation's `IsPasswordProperty` immediately before the gesture would otherwise
  fire; if it reports a password field, the gesture does nothing there at all.
- **Does not act in console windows** (cmd.exe, PowerShell, Windows Terminal), since
  the synthetic Ctrl+C the gesture relies on would interrupt a running program there
  instead of copying.

## Why the synthetic input is tagged

Keystrokes and clicks this app injects itself (replaying a character you meant to type
normally, releasing the modifier before sending Ctrl+C/V) are marked with a private
`dwExtraInfo` value the app's own hook recognizes and ignores, so they are never
mistaken for new user input and re-processed as another gesture. This is purely an
internal bookkeeping mechanism, not a security boundary against other software.

## Scope of what this document covers

This covers the app's own behavior as shipped. It does not cover:

- The security of Windows' own low-level hook or `SendInput` APIs themselves.
- Third-party antivirus or endpoint-security software that may flag any app installing
  a global keyboard hook as suspicious by category, regardless of what it actually does
  (a false positive this app is especially likely to trigger - please verify behavior
  against this document, PRIVACY.md and your own firewall rather than relying on an AV verdict alone).
- The target application you paste into - if it is compromised, this app's paste is no
  more or less trusted than any other paste into it.

## Reporting a concern

This is a small independent product without a formal disclosure program. If you find a genuine security issue (not just "AV flagged
it" - see above), please open an issue at
https://github.com/Afonnis/LayFlip/issues with enough detail to reproduce it, or
reach the maintainer directly if the issue should not be public before a fix is
available.
