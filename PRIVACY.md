# Privacy

LayFlip works on your computer only. It has no account, no server, no telemetry and no crash
reporting, and it never sends your text, your keystrokes or anything else anywhere.

## What it reads, and when

- **The gesture.** The program watches whether the key you chose for the gesture is held and whether
  the mouse wheel is turned. Other keystrokes are not recorded, logged or stored.
- **Text you fix.** When the gesture fires, the program copies your selection through the clipboard,
  converts it on your machine and pastes the result back, then puts back what the clipboard held
  before (unless you or another program copied something else in the meantime). Its own clipboard
  writes are marked so Windows clipboard history and cloud clipboard skip them.
- **Fixing without a selection (2.9 and later).** To fix the text you have just typed, the program
  remembers how many characters you typed and which of them were spaces - never which keys they
  were. This is kept in memory only and is dropped as soon as you click, press Enter, Tab, an arrow
  key or a shortcut, or move to another window or field.
- **Password fields** are never read or changed.

## What it stores

On your computer only, in your user profile:

- `%APPDATA%\LayFlip\settings.ini` - your settings.
- `%APPDATA%\LayFlip\exceptions.txt` - words you told it not to change again.
- `%LOCALAPPDATA%\LayFlip\layflip.log` - a log with event types and character counts, never the
  text itself, unless you turn on the verbose log in the settings; it is limited to 512 KB.

Uninstalling removes the program and its shortcuts. The files above stay until you delete them.

## Network

The program makes no network connections. "Check for updates" opens the releases page in your
browser, only when you click it. You can confirm this with your firewall or Resource Monitor:
blocking the program changes nothing about how it works.

## Contact

Questions about privacy: open an issue at https://github.com/Afonnis/LayFlip/issues.
