# How LayFlip works

The user-facing summary is in [README.md](README.md); this page has the details.

## The gesture

Lightweight Windows tray utility for fixing text typed with the wrong keyboard layout.

Gestures — hold the key under `Esc` (the one left of `1`) and scroll:

```text
key under Esc + mouse wheel up    -> fix the selected text
key under Esc + mouse wheel down  -> undo the last fix (works even mid-hold, right after cycling)
```

That key is labelled differently around the world - `ё` on Russian, `` ` `` on US/UK,
`²` on French, `^` on German - but it is the same physical button everywhere, and the
app names it correctly for whatever layout you are using.

Workflow:

1. Select mistyped text in any app.
2. Hold the modifier key (`ё` / `` ` `` by default).
3. Scroll the mouse wheel up once.
4. The app copies the selection, converts it, pastes the fixed text, and restores the previous clipboard.

If it converted something that was actually fine, hold the same key and scroll **down**
to put the original text back (within 30 seconds by default). This matters because the
language detection has no dictionary - it works off a statistical letter model (see "How it
decides," below) - so correct text can occasionally get "fixed" into nonsense. Undo makes
that recoverable instead of destructive.

## Trying another reading for an ambiguous word

Some single words read equally plausibly in more than one language (e.g. a word that
could be unremarkable English or could be a different language typed in the wrong
layout). When that happens, the first scroll-up leaves that word alone, and - without
releasing the key - a further scroll-up cycles through
the other plausible readings for that word, one per tick. Scrolling down at any point
undoes the most recent change and goes back to the original word, without permanently
blacklisting it (see below) - you were just looking, not rejecting a fix.

In a multi-word selection this works when exactly one of its words is ambiguous; with
two or more, the selection behaves like a normal one-shot fix. Short all-caps words
(abbreviations such as "RYG" for "КНП") get the same offer when their conversion is not a
clear win.

The same second scroll also switches a whole selection between Russian and Ukrainian when
the choice between them was a judgement call: "J rjnhsq ujlbys rjywthn&" first becomes
"О котрый годины концерт?", and one more scroll gives "О котрій годині концерт?" - every
word at once. When a letter only one of the two languages has was typed, the choice is
certain and there is nothing to switch to.

## Undo remembers what you rejected - but only real rejections

Undoing a normal (non-cycled) fix teaches the app: "that word, that specific
conversion, don't offer it automatically again." That word is then left alone by
future fixes, in any context, until you tell it otherwise: the tray menu's
"Learned exceptions (N)..." window lists these words and lets you remove any of them
(select and press Delete, or "Remove all"), after which they get fixed again. The list
itself is `%APPDATA%\LayFlip\exceptions.txt`.

Undoing a cycle step (see above) does **not** teach this - backing out of "let me see
the other reading" isn't the same as rejecting a fix outright.

## Where the gesture does nothing on purpose

- **Console windows** (cmd.exe, PowerShell, Windows Terminal): Ctrl+C without a
  selection would interrupt whatever is running there, so the gesture is disabled -
  copy via the console's own right-click menu instead.
- **Fullscreen applications** (games, video, presentations): the gesture pauses itself
  automatically, since a bumped modifier while scrolling to aim would be the worst
  possible place for an accidental fix.
- **Password fields**: detected via UI Automation (`IsPasswordProperty`) and skipped -
  triggering the gesture there would briefly put the password on the clipboard.
- **Excluded processes**: `mstsc` (Remote Desktop) is excluded by default, because its
  Ctrl+C/V goes through a redirection channel this app can't see reliably. Add more via
  `ExcludedProcesses=name1,name2` in `settings.ini` (no `.exe` suffix).

## Settings and the tray menu

**Настройки...** in the tray menu (or a double-click on the tray icon) opens the settings window.
Every change applies and is saved at once:

| Setting | Does |
| --- | --- |
| **Язык интерфейса** | Auto (follows Windows), Russian, Ukrainian or English. |
| **Запускать при входе в Windows** | The same Startup-folder shortcut the installer's checkbox makes. |
| **Клавиша жеста** | The modifier: the key under `Esc` (default), CapsLock, right Ctrl, right Alt, or a mouse side button. |
| **Отмена прокруткой вниз** and how many seconds it stays possible | Whether scrolling down undoes at all, and for how long after a fix (30 s by default). |
| **Переключать раскладку после исправления** | After a fix, also switch the OS keyboard layout to match the language it just fixed into (like Punto Switcher). On by default. |
| **Программы, где жест выключен** | Process names (no `.exe`), picked from the open programs or typed; `mstsc` by default (see above). |
| **Подробный лог (с текстом, для отладки)** | Off by default - see "Privacy" below. Next to it: the learned exceptions and the log folder. |

The tray menu itself keeps what is used on the go:

| Item | Does |
| --- | --- |
| **Слова-исключения (N)...** | Opens the list of words you undid a fix of (see "Undo remembers what you rejected" above); remove any to have them fixed again. |
| **Выключить для «app»** | Adds or removes the app that was in front to the excluded programs. |
| **Пауза** / global hotkey `Ctrl+Alt+Shift+P` | Pause/resume the gesture without opening the menu. |
| **Проверить преобразование** | Sanity-checks that conversion works at all (needs at least one non-Latin layout installed). |
| **Версия** | Shows the build's version number (also visible in the `.exe`'s own Properties > Details in Explorer). |
| **Проверить обновления** | Opens the GitHub releases page in your browser - only when you click it. No background checks, ever (see "Privacy" below). |

Settings persist to `%APPDATA%\LayFlip\settings.ini`.

**How the key under `Esc` can be both a modifier and a normal key:** the app swallows
the key press, and if you release it without scrolling, it replays the character at that
moment. So typing `ё` / `` ` `` still works - the character just appears on key release
instead of key press. Holding the key never repeats the character either. If you start
typing something else while the key is held (without ever scrolling), that keystroke's
own character is delivered first, in the order you actually typed, then the hold exits
hold-mode and the rest of your typing passes through untouched.

**Why scan codes, not virtual keys:** Microsoft documents the `VK_OEM_*` virtual-key
codes as layout-dependent. `VK_OEM_3` is the key under `Esc` only on a US layout - on a
German QWERTZ it is `Ö`, in the middle of the letter block. This app therefore matches
the physical key by hardware scan code (`0x29`), and builds its conversion tables from
scan codes too (including the Shift-held table, for uppercase Cyrillic and digit-row
symbols like Russian's `№`), so key positions and shifted symbols line up correctly on
every layout.

Examples (assuming Russian is one of your installed Windows keyboard languages):

```text
Ghbdtn, rfr ltkf? -> Привет, как дела?
руддщ -> hello
crf;b xnj -> скажи что     (Russian letters that share a key with Latin punctuation, like ; and ', convert too)
```

## Which languages it converts between

The app does **not** use a hardcoded language list. On startup it reads whatever
keyboard layouts are actually installed in Windows (Settings > Time & Language >
Language & region) and builds its conversion tables from those - so it works with
whatever the person running it has installed, not just English/Russian/Ukrainian.
Right-click the tray icon to see "Найденные раскладки: ..." and confirm what it found.

If only one layout is installed (e.g. just English), there is nothing to convert
between and "Проверить преобразование" will report no conversion - that's expected,
not a bug.

## How it decides

For every word in the selection it compares the word as typed with its readings in the other
installed layouts and keeps the most plausible one (so "Hello ghbdtn" only touches the second
word). Plausibility comes from a built-in letter model for English, Russian, Ukrainian,
Belarusian, Bulgarian, Serbian (Cyrillic), Macedonian, Tatar, Greek, German, French, Spanish, Italian, Portuguese, Polish, Czech, Dutch, Swedish, Danish, Norwegian, Finnish, Hungarian, Romanian, Slovak, Lithuanian, Latvian, Icelandic, Croatian, Estonian, Turkish, Kazakh, Kyrgyz, Mongolian (Cyrillic), Hebrew, Arabic, Persian, Armenian and Georgian - the probability of each letter given the two before it, learned from real
sentences - and from alphabet coverage for any other installed language. Then it looks at the
selection as a whole:

- The whole selection is meant in one language too: with the phonetic Serbian layout installed,
  "resistance" in English typed on the Russian layout is not read as Serbian "ресистанце", and an
  English word in correct Russian text is not read as a Serbian transliteration.
- The whole selection was typed on one layout, so a letter only one layout has ("щ") decides
  where the other words came from too ("іуу" is "see" from the Ukrainian layout, not "bee" from
  the Belarusian one, where "і" sits on the B key).
- Russian, Ukrainian and Belarusian are decided once for the selection, not word by word (they
  share most keys), and letters that leave only one of them settle it; text that already reads
  as one of them is kept unless another reading is clearly better; otherwise the second scroll
  offers the runner-up.
- Words that are nothing but punctuation keys on an English layout - Ukrainian "є" ("'"),
  "її" ("]]"), the particle "б" (",") - are read from the language around them, and a short
  word left over in an otherwise converted selection follows it.
- Lone letters ("r" for "к", "Z" for "Я") are converted only when the rest of the selection
  says which language they belong to.
- A plausible word written like the rest of a correctly typed sentence is left alone ("руку"
  is not turned into "here"); a name with a capital inside ("GitHub", "iPhone") is never touched.
- Keys that are punctuation on one layout and a letter on the other (б ю ж э х ъ ё) are read
  from context: "levf. jy" is "думаю он", but "Ghbdtn, rfr ltkf." stays "Привет, как дела.".
  A Russian sentence typed key for key ends in "/" and puts its commas on "?" - both are fixed
  when the selection shows it was typed that way.

When it cannot decide, it leaves the word alone rather than guessing (see "Trying another
reading," above, for the case where it instead offers a choice).

**Other languages.** The letter model covers English, Russian, Ukrainian, Belarusian, Bulgarian,
Serbian (Cyrillic), Macedonian, Tatar, Greek, German, French, Spanish, Italian, Portuguese, Polish, Czech, Dutch, Swedish, Danish, Norwegian, Finnish, Hungarian, Romanian, Slovak, Lithuanian, Latvian, Icelandic, Croatian, Estonian, Turkish, Kazakh, Kyrgyz, Mongolian (Cyrillic), Hebrew, Arabic, Persian, Armenian and Georgian. With
another layout installed as well (Hindi, Thai, ...), fixing those still works as described, but
that language is judged by its alphabet alone: correct text in it is mostly left alone yet
occasionally "corrected", and text in it typed on the wrong layout tends to come out in the
English layout instead (before German had its model, umlauts came out as "[" and "{"). More
languages are being added one by one: every Windows layout is known to the program, and
a model can be built for any language with enough text.

## Accuracy

Measured by `tests\eval\run-eval.cmd` on about 2000 real sentences per language that the model
never saw (Tatoeba), typed on the wrong layout key for key:

| | Sentences fixed completely | Correct sentences changed by mistake |
|---|---|---|
| Russian typed on English | 99.5% (97.5% with punctuation) | 0.1% |
| Ukrainian typed on English | 98.3% (96.7% with punctuation) | 0.0% |
| English typed on Russian | 99.2% (99.0% with punctuation) | 0.0% |
| Ukrainian typed on Russian | 94.9% (99.9% with the second scroll) | 0.0% |
| Russian typed on Ukrainian | 97.8% (99.9% with the second scroll) | 0.1% |

The other languages, each measured with its layout installed next to Russian, Ukrainian and
English (the numbers above stay within a few tenths with any of them):

| Language (layout) | Typed on English | Typed on Russian | Correct text changed |
|---|---|---|---|
| Belarusian | 99.4% | 98.0% | 0.1% |
| Bulgarian (Phonetic) | 94.7% | 98.3% | 0.2% |
| Serbian (Cyrillic) | 93.1% | 97.6% | 0.3% |
| Macedonian | 93.2% | 96.0% | 0.3% |
| Greek | 96.3% | 92.4% | 0.0% |
| German | 97.2% | 99.3% | 0.0% |
| French (AZERTY) | 99.2% | 99.5% | 0.0% |
| Polish (Programmers) | 98.3% | 90.9% | 0.1% |
| Spanish | 99.2% | 95.7% | 0.0% |
| Italian | 94.8% | 99.6% | 0.1% |
| Portuguese (Brazilian ABNT) | 99.1% | 98.1% | 0.0% |
| Czech | 98.9% | 99.2% | 0.0% |
| Dutch | 99.8% | 99.5% | 0.0% |
| Swedish | 99.3% | 98.9% | 0.1% |
| Danish | 98.8% | 99.2% | 0.1% |
| Norwegian (Bokmål) | 98.5% | 99.3% | 0.0% |
| Finnish | 99.2% | 99.8% | 0.0% |
| Hungarian | 98.7% | 99.1% | 0.0% |
| Romanian (Legacy) | 99.8% | 99.2% | 0.0% |
| Slovak | 98.8% | 99.4% | 0.0% |
| Lithuanian | 99.6% | 99.9% | 0.0% |
| Latvian (Standard) | 98.9% | 87.4% | 0.0% |
| Icelandic | 99.2% | 99.0% | 0.0% |
| Croatian | 96.5% | 99.0% | 0.0% |
| Estonian | 99.7% | 99.8% | 0.0% |
| Turkish (Q) | 96.7% | 98.1% | 0.1% |
| Tatar | 98.9% | 99.9% | 0.1% |
| Kazakh | 99.4% | 99.9% | 0.0% |
| Kyrgyz | 48.9%* | 98.6% | 0.1% |
| Mongolian (Cyrillic) | 97.9% | 90.2% | 0.1% |
| Arabic (101) | 95.6% | 96.4% | 0.2% |
| Persian (Standard) | 93.5% | 97.0% | 0.2% |
| Hebrew | 93.5% | 96.2% | 0.1% |
| Armenian (Eastern) | 92.4% | 97.0% | 1.6% |
| Armenian (Phonetic) | 94.2% | 96.8% | 0.8% |
| Georgian | 98.9% | 99.4% | 0.0% |
| Georgian (QWERTY) | 94.3% | 95.8% | 0.3% |

\* Kyrgyz ң ө ү have no key on the English layout, so most Kyrgyz sentences cannot be typed there
in full; on the Russian layout, where Kyrgyz is usually typed, 98.6%. Kazakh, Kyrgyz and Mongolian
are trained with Wikipedia sentences as well (Tatoeba has few).

Greek accents are typed with a dead key, and so they are read: "Kane;iw" is "Κανείς". German
typed on English differs only in the words with Y, Z, umlauts or ß ("yu" -> "zu", "f[r" ->
"für", "neu_" -> "neu?"). French AZERTY keeps the apostrophe on the 4 key ("О4йш" is "J'ai").
Polish (Programmers) types its letters with AltGr, which gives nothing on the Russian layout - so
"ą", "ż"... typed there are lost and only the rest can come back. Some Armenian words typed on
the phonetic layout are English words too ("դիմես" / "dimes"), which is where most of its
mistakes on correct text come from.

A selection that already reads as correct Russian, Ukrainian or Belarusian is left as it is on
the first scroll, and the second scroll offers it in the closest other language: a Ukrainian
name typed on the Russian layout, "Андрый", is "Андрій" one scroll later (a single short word is
too little for the letter model to overrule what was typed). The model was also trained on lists
of common names, patronymics and surnames (compiled for this project), so longer runs of names - "Агафонова
Крыстына Выталыъвна" - convert on the first scroll.

Counting the second scroll as well: 99.7% / 99.3% / 99.7%. Mixed selections - a sentence on
the wrong layout with a correctly typed English name in it, or one typed right at the start and
wrong at the end - are fixed completely in 98-99.9% of cases. The history of every
change is in [CHANGELOG.md](CHANGELOG.md).

## Privacy

- **The log does not contain your text by default** - only event types and character
  counts (e.g. "Fixed selection (6 -> 6 chars)"). Turn on "Подробный лог" in the settings
  window to log full text content, for debugging a specific conversion problem only.
- **Log location:** `%LOCALAPPDATA%\LayFlip\layflip.log`, capped at
  512 KB and rolled over to `layflip.log.old`.
- **Clipboard writes are tagged to opt out of clipboard history, Cloud Clipboard
  roaming, and most third-party clipboard managers** (`CanIncludeInClipboardHistory`,
  `CanUploadToCloudClipboard`, `ExcludeClipboardContentFromMonitorProcessing`) - this
  app's automated clipboard use is never something you chose to keep around.
- **The clipboard is copied out in full before the app touches it, and restored
  afterwards** - but only if nothing else wrote to the clipboard in the meantime
  (checked via the clipboard's sequence number); if you (or another app) copied
  something else mid-gesture, that is left alone instead of being silently overwritten.
- **No automatic network access, no telemetry, no background update checks.** The only
  network request this app can ever make is opening the releases page in your browser,
  and only when you click "Проверить обновления" in the tray menu yourself.

## Notes on behaviour

- Only one copy can run at a time; launching a second one just tells you it is already running.
- If pasting the fix fails (commonly: the target app is running elevated/as administrator
  and this app isn't), a tray notification says so instead of silently doing nothing.
