# One look: UI rules

Every screen does the same job the same way, in every space. The look below was approved from the mockups in the design canvas ("One look across the app"). The shared pieces live in the app's `lib/src/shared` and `lib/src/theme/layout.dart`; a screen uses them instead of its own copy.

## Spacing and shape

| What | Value |
|---|---|
| Screen and sheet gutter | 20 |
| Gap between cards, and between tiles side by side | 12 |
| Inside a card | 16 |
| Below a tab's title row | 16 |
| End of a tab's list (clear of the + button) | 96 |
| End of a pushed screen's list | 24 |
| Card corner | 20 |
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

Amounts use the display font with even-width figures. An amount is never split across lines, shrunk or cut: the only break allowed is between the currency code and the number, with the code going above. Tiles side by side (`FigureRow`) go two to a row when a figure would not fit whole, an odd last one at full width, and one to a row only when two do not fit either. Use the shared `FigureText`, `FigureTile` and `FigureRow` for every amount in a tile or row.

While amounts are hidden, every amount on a browse screen shows dots. The counter (Sell, Pay, tabs, Close the day, record sheets), forms and documents to share or print always show amounts.

## The look

- **Red only where it counts.** Red is for the lead card, the one main button on a screen (+, Pay, Sell, Save) and the open tab. Every other button, link and action is neutral: text colour, light border. Home's profile button is an outlined circle.
- **The + button.** A solid red circle with a white +, beside the tab bar on every tab, never over the list. It is the main action: Sell in a business, add a transaction or import a statement elsewhere.
- **Each number once.** A figure shows in one place per screen. Only a red lead card's figure carries the currency code (KES 1,250); every other amount is a plain number (1,250), with + for money in. Amounts in another currency keep their code. Receipts, invoices, statements, reports and the counter keep the code.
- **One shape for every card.** A title line, at most one detail line, and one action: tap the card (› at the end) or one button.
- **One way to add.** "+ Add" sits at the right of the title of the list it adds to: the screen's title for a screen's list, the section's title for a section. Actions beside a title are small outlined pills ("Count stock", "See all", "Remind").
- **Helpful empty states.** Never a dash in a big figure or a tile. A red card with nothing yet says what will show and when; a tile says "Soon" and when. Only what needs history waits: the best day shows after two weeks of sales, while the other insights show at once.

## Screens

- **Tabs** start with one title row: the title (the tab's name, on one line), pills for the tab's other screens and "+ Add", then the eye last. Home keeps its own header.
- **Pushed screens** use a titled app bar, and keep it while loading. Actions: pills for other screens and "+ Add", icon buttons with a tooltip, then the eye, then 8.
- **Intro text** under an app bar is body large in the muted colour; an intro heading is headline small.
- **Section headings:** small capitals over a group of rows ("TODAY", "HISTORY"); a sentence-case title for a section, with its action at the right ("Goals" · Add a goal).
- **Red lead cards** are one card: label and eye, the figure, up to two caption lines, an optional bar.
- **Tiles** are one tile: label, figure, an optional note.
- **Rows** in a card are split by full-width lines. An action word at the end ("Edit", "Change") is in the text colour. A row with no value that opens another screen ends with ›; rows with a value do not.
- **Tags** ("Running low", "Paid") and **notes** come in three tones: information, good news and something to check. Overdue is amber everywhere.
- **Marked messages.** When the app shows which part of a message is which, each part has its own colour: amount red, who blue, date and time green, reference amber, balance purple, fee teal. These mark colours are for that and nothing else.
- **Keep what informs.** A redesign never drops a detail that helps: what is on hand, the weakest part of the health score, each payment that may not go through, why a setting matters. Long details wrap to a second line before they shorten, and the account stays whole.

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
