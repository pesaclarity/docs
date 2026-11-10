# One look: UI rules

Every screen does the same job the same way, in every space. The shared pieces live in the app's `lib/src/shared` and `lib/src/theme/layout.dart`; a screen uses them instead of its own copy.

## Spacing and shape

| What | Value |
|---|---|
| Screen and sheet gutter | 20 |
| Gap between cards, and between tiles side by side | 12 |
| Inside a card | 16 |
| Below a tab's title row | 16 |
| End of a tab's list (clear of the + button) | 96 |
| End of a pushed screen's list | 24 |
| Card corner | 24 |
| Note, selectable tile, badge panel | 16 |
| Tag | 8 |
| Touch target | 44 at least |

## Amounts

| Role | Size |
|---|---|
| The one figure on a red lead card | 40 |
| The main figure on a white card | 32 |
| A figure in a tile | 24 |
| An amount at the end of a row | 18 |

Amounts use the display font with even-width figures. While amounts are hidden, every amount on a browse screen shows dots. The counter (Sell, Pay, tabs, Close the day, record sheets), forms and documents to share or print always show amounts.

## Screens

- **Tabs** start with one title row: the title, links to the tab's other screens as text buttons, then the eye last. Home keeps its own header.
- **Pushed screens** use a titled app bar, and keep it while loading. Actions: text buttons for other screens, icon buttons with a tooltip, then the eye, then 8.
- **Intro text** under an app bar is body large in the muted colour; an intro heading is headline small.
- **Section headings:** small capitals over a group of rows ("TODAY", "HISTORY"); a sentence-case title for a section, with its action at the right ("Goals" · Add a goal).
- **Red lead cards** are one card: label and eye, the figure, up to two caption lines, an optional bar.
- **Tiles** are one tile: label, figure, an optional note.
- **Rows** in a card are split by full-width lines. An action word at the end ("Edit", "Change") looks like a text button. A row with no value that opens another screen ends with ›; rows with a value do not.
- **Tags** ("Running low", "Paid") and **notes** come in three tones: information, good news and something to check. Overdue is amber everywhere.

## States

- **Loading:** the spinner under the title.
- **Could not load:** "<Thing> could not load." and Try again. Never a spinner that never ends.
- **Empty:** one muted line, "<Things> show here."

## Sheets and dialogs

- Every sheet opens the same way, as tall as it needs, with a handle. It has a headline-small title, full-width content and the main button last. There is no Cancel button in a sheet; the handle and tapping outside close it.
- Two buttons side by side: the main one first.
- Deleting anything asks first: "Delete <name>?", one sentence, then [Keep] [Delete].
- Other dialogs: text buttons, Cancel then the action ("Save", "Switch to USD").

## Choices and forms

- More than two options: the one drop-down. Two options: the segmented switch. On/off settings: switches. Ticks only for agreeing to something.
- Categories: the one category picker.
- Typed amounts and dates use the shared fields. Fields follow the person's field style (underlined or boxed).
- A check reads "Enter <what>, such as <example>." and shows under the field.

## Words

- Sentence case. "Add a <thing>", "Edit <thing>", "Save", "Choose", "Delete", "Record a payment".
- A short message after an action: past tense with a full stop ("Sale saved."). A failure says what to do: "Try again."
- The currency code comes from the base currency, never written into the app.
- Plain English, warm and brief, as in [BRAND.md](../BRAND.md).
