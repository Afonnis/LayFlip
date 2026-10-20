# Changelog

What changed for people using LayFlip, newest first. Installers for every version are on the
[releases page](https://github.com/Afonnis/LayFlip/releases); accuracy numbers come from the benchmark
described in [HOW-IT-WORKS.md](HOW-IT-WORKS.md#accuracy).

## 2.9.1 - 2026-10-20

- **New license:** free for personal use, a commercial license for work. Versions up to 2.9.0 stay
  under the MIT License. The installer now shows the agreement, and there are privacy notes
  (PRIVACY.md) and a list of third-party data (NOTICE.md).

## 2.9.0 - 2026-10-14

- **Fixing without a selection.** Typed a phrase on the wrong layout? Just do the gesture: LayFlip selects
  what you typed since the last click or arrow key and fixes it. With nothing typed, it takes the line
  from its start up to the caret. The caret stays where it was, text after it is not touched, and the
  undo gesture works the same. Checked in plain and rich text boxes and in the Windows 11 Notepad.
- **Times, units and quotes in a fixed phrase:** "15^30" comes back as "15:30" (the Shift+6 key), "70 ru" as
  "70 кг", "1500 he,/" as "1500 руб." and "1500 uhy/" as "1500 грн.", and an opening quote typed as @
  becomes " like the closing one already did.
- **Caps Lock left on:** "пРИВЕТ" and "gHBDTN" (a capital typed with Shift while Caps Lock was on) come back
  as "Привет". Names like "iPhone", "GitHub", "mRNA" are left alone.
- The Windows 11 Notepad sometimes ignored the paste after a fix; LayFlip now checks that the text
  arrived and pastes again if it did not.
- A README for people who just want to use the app, with a recorded demo; the details moved to
  HOW-IT-WORKS.md and DEVELOPMENT.md.
- The landing page in English as well.

## 2.8.2 - 2026-10-13

- **Kazakh, Kyrgyz and Mongolian (Cyrillic)** - 38 languages in all. Kazakh 99.4% of sentences fixed
  when typed on the English layout and 99.9% on the Russian one, Kyrgyz 98.6% on the Russian one,
  Mongolian 97.9% on the English one.
- **Armenian and Georgian punctuation** typed on the English layout comes back: the Armenian Eastern
  layout has ":" on the 1 key and "," on the 5 key, so a fixed sentence used to end in "1".
- **Fixed:** with a Czech, French, Hungarian or another layout that has a letter on Shift+/ installed,
  a question typed with English punctuation ended in "," instead of "?".
- English typed on the Russian layout is read from the one layout the whole phrase reads best from -
  with Bulgarian (Phonetic) installed 95.6% -> 99.2%, with Mongolian 94.0% -> 99.1%.
- The second scroll keeps offering the Ukrainian (or Russian) reading with a Kazakh or Kyrgyz layout
  installed.

## 2.8.0 - 2026-10-13

- **Settings window**: Settings... in the tray menu, or a double-click on the tray icon - interface
  language, start with Windows, gesture key, undo and how long it stays possible, layout switching,
  programs where the gesture is off, the detailed log. The tray menu is shorter.
- Latvian typed on the Ukrainian layout: "ū" (AltGr+U, "ґ" on the Ukrainian layout) comes back as "ū",
  not "\" - 75.7% -> 89.7%.

## 2.7.0 - 2026-10-12

- 13 more languages: Dutch, Swedish, Danish, Norwegian, Finnish, Hungarian, Romanian, Slovak,
  Lithuanian, Latvian, Icelandic, Croatian and Estonian (35 in all).

## 2.6.1 - 2026-10-10

- The installer's artwork says LayFlip.

## 2.6.0 - 2026-10-10

- **Renamed from Layout Fixer Desktop to LayFlip.** Installing over the old program moves your
  settings and learned exceptions across and removes it.
- Spanish, Italian, Portuguese, Turkish, Czech, Tatar, Arabic and Persian (22 languages).

## 2.5.0 - 2026-10-10

- Greek, German, French, Polish, Bulgarian, Serbian (Cyrillic), Macedonian, Hebrew, Armenian and
  Georgian (14 languages).
- Dead keys and AltGr: Greek accents, French AZERTY apostrophes, Polish letters typed with AltGr.

## 2.4.1 - 2026-10-09

- The gesture works in Codex and the new Windows 11 Notepad.
- The letter model knows common first names, patronymics and surnames (Russian, Ukrainian,
  Belarusian), so names convert on the first scroll.
- Quotes typed the English way stay quotes.

## 2.4.0 - 2026-10-09

- Belarusian.
- Ukrainian "ґ" converts; conversion is 2-3 times faster.
- A Ukrainian name typed on the Russian layout ("Андрый") becomes "Андрій" on the second scroll.

## 2.3.0 - 2026-10-07

- A window listing the learned exceptions (words you undid a fix of), to remove any of them.

## 2.2.1 - 2026-10-07

- Ukrainian typed on the Russian layout, and Russian typed on the Ukrainian one, are fixed.

## 2.2.0 - 2026-10-07

- A new letter model learned from real sentences; the phrase is decided as a whole, not word by word.
- The second scroll switches a phrase between Russian and Ukrainian.
- Punctuation keys that are letters on the other layout (б ю ж э х ъ) are read from context.

## 2.1.1 - 2026-10-07

- The installer's finish-page checkboxes are visible and work.

## 2.1.0 - 2026-10-07

- First release: hold the key under Esc and scroll up to fix the selection, down to undo; Russian,
  Ukrainian and English; interface in Russian, Ukrainian and English; per-app exclusion; installer.
