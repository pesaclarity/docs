# Pesa Clarity brand

<img src="brand/logo-wordmark.svg" alt="Pesa Clarity" width="420">

Pesa Clarity is an independent project. It shares no brand, code or ownership with any employer or client.

## Name

- The name is two words, **Pesa Clarity**, everywhere people read it: in prose with both words capitalised, and in the wordmark as **pesa clarity**, lowercase, with "pesa" in the text colour and "clarity" in red. A visible name never joins the words: not "PesaClarity", "Pesaclarity" or "pesaclarity", and not in titles, alt text, store listings, app names or social cards either.
- The only exceptions are technical identifiers, which keep the joined form because changing them would break links, installs and builds:
  - the GitHub organisation and repository names (`pesaclarity`, `pesaclarity-app`, `pesaclarity/.github`) and other account handles;
  - domains and hosts, such as `pesaclarity.github.io`, `pesaclarity.cloud.run` and `api.pesaclarity.org`;
  - package and bundle ids, such as `com.pesaclarity.app`;
  - file names, such as `pesaclarity-play.apk`;
  - environment variables, such as `PESACLARITY_API_URL`;
  - the HTTP header `X-Pesaclarity-Integrity`;
  - class names, channel names, storage keys and other names in code, such as `PesaClarityApp` and `pesaclarity.db.key.v1`.
- Working name until after the proof of concept (see the spec's open questions). Check Kenyan trademark (KIPI) and Play Store availability before launch. An unrelated GitHub project, `Jenn-mawia/PesaClarity-Dashboard`, already uses the name.

## Promise and voice

**Promise:** your money, made clear, without giving your data away.

Voice rules, from the spec's principles:

- Plain answers: "You can spend 1,200 today", not "Daily discretionary allowance: KES 1,200".
- One number first, detail one tap deeper.
- Calm about money problems. No shaming, no alarm emojis, no "Oops!".
- Honest about privacy: say exactly what leaves the phone.
- Plain Kenyan English that anyone can read. No Swahili or Sheng terms; product names (M-Pesa, Fuliza, Pochi la Biashara) stay as they are.

## Logo

| File | Use |
| --- | --- |
| [`brand/logo-mark.svg`](brand/logo-mark.svg) | App icon, avatars, favicons. Red tile, white "p" with a centre dot. |
| [`brand/logo-mark-mono.svg`](brand/logo-mark-mono.svg) | One colour, takes `currentColor`. Print, notifications, watermarks. |
| [`brand/logo-wordmark.svg`](brand/logo-wordmark.svg) | Headers, README, website. |

The mark is a lowercase **p** whose bowl is both a coin (pesa) and a lens (clarity); the dot is the coin's centre.

- Keep clear space of a quarter of the mark's width on every side.
- Minimum size: 24 px for the mark, 120 px wide for the wordmark. The space between "pesa" and "clarity" is Space Grotesk's own word space, which still reads as a gap at 120 px; never close it up or widen it.
- Don't recolour, rotate, stretch, outline or add effects. Don't put the full-colour mark on busy photos.
- The wordmark SVG uses live text (Space Grotesk). Before using it outside a browser, outline the text in a vector editor.

## Colour

The brand colour is **Pesa Red**, the red used across the app designs. It is Pesa Clarity's own colour.

| Token | Hex | Use |
| --- | --- | --- |
| Pesa Red (`brand`) | `#C8102E` | Fills: the red lead card (Safe to spend, Debt-free by, Sales today), the + button, the main button, a filtering drop-down. White text on it: 5.9:1 |
| Link red (`brandLink`) | `#C8102E` | Text and thin strokes: the current tab, progress bars, charts, links. Pill buttons such as See all and Add, and the two-way switch, stay neutral |
| Soft fill (`brandSoft`) | `#EBEBEB` | The neutral fill behind a chosen chip, menu choice or side of a switch |
| Selected (`selected`) | `#EBEBEB` | Quiet neutral fills, such as the Ask bar |
| Background | `#F5F5F5` | Neutral screen ground, so white cards stand off it |
| Surface / border | `#FFFFFF` / `#E5E5E5` | Cards, lists, nav bar |
| Text / muted | `#171717` / `#525252` | Body, labels (muted 7.2:1) |
| Success | `#059669` (text `#047857`) | Income, on track, cleared |
| Warning | `#B45309` | In progress, overdue, something to check |
| Warning, stronger (`warningStrong`) | `#C2410C` | The worse of two warnings, such as Struggling beside Coping |

Amounts stay neutral; red on an amount means below zero only. Spending charts and the weakest Health Index pillar stay neutral too, so red never reads as a warning there.

**Red means brand, so "over budget" can't rely on colour alone.** Every over or failed state also carries a label or icon ("Over", ⚠ icon). Green stays a status colour only, which also keeps us visually distinct from M-Pesa.

Dark mode is a soft dark, grey rather than near black, and keeps the same red:

| Token | Hex |
| --- | --- |
| Background / surface / border | `#121212` / `#1C1C1C` / `#2E2E2E` |
| Fields and chips / floating parts (tab bar, sheets, menus) | `#222222` / `#262626` |
| Text / muted | `#E0E0E0` / `#A3A3A3` |
| Brand, a fill only (white text on it) | `#C8102E` |
| Danger (`danger`, fills only, such as a destructive button) | `#C8102E` |
| Danger in text and large figures below zero (`dangerText`) | `#FF4D6A` |
| Accent (`brandLink`: links, current tab, progress, chosen options, red edges on cards) | `#FF4D6A` |
| Soft fill (`brandSoft`) / selected | `#262626` / `#333333`, neutral greys, never pink |
| Success (text `#6EE7B7`) | `#34D399` |
| Warning / stronger | `#FBBF24` / `#FB923C` |

Every neutral, light and dark, is a pure grey with no warm or rose tint. Dark is soft: a `#121212` page and `#1C1C1C` cards rather than near black, and `#E0E0E0` text rather than near white, so it glares less and is easier on tired eyes. The phone's high-contrast setting keeps the deeper `#0A0A0A` page and its stronger text and edges. Brand red on dark is a fill only, with white text on it (5.9:1): on a card it is 2.9:1, too faint for text or an edge. So red text, large red figures (such as a tile below zero), red edges on cards and thin strokes use `#FF4D6A`, the same hue brightened. Dark has no drop shadows; floating parts sit on a lighter surface instead.

Dark contrast (WCAG 2.2):

| Pair | On page `#121212` | On card `#1C1C1C` | On sheet `#262626` |
| --- | --- | --- | --- |
| Text `#E0E0E0` | 14.2:1 | 12.9:1 | 11.5:1 |
| Muted `#A3A3A3` | 7.4:1 | 6.8:1 | 6.0:1 |
| Red `#FF4D6A` (text, large figures, edges) | 5.8:1 | 5.3:1 | 4.7:1 |
| Brand red `#C8102E` (fills only) | 3.2:1 | 2.9:1 | 2.6:1 |
| White on brand red | 5.9:1 | | |

Machine-readable values: [`brand/tokens.json`](brand/tokens.json) (W3C design-token format).

## Type

- **Space Grotesk** for numbers and titles. Tabular figures for amounts.
- **Inter** for body text and labels.
- Both are SIL Open Font License 1.1, so they can ship inside the app and the F-Droid build.

## Shape

20 px card radius, 44 px minimum touch target, floating pill navigation, one primary number per screen, no horizontal scrolling. Spacing, sizes and how each screen, sheet and dialog is built are in [brand/ui-rules.md](brand/ui-rules.md).

## Licence

The logo and name are trademarks of the project and are **not** covered by the CC BY 4.0 licence on this repository's text. Forks of the app must use a different name and logo, so clones cannot pose as Pesa Clarity and harvest SMS.
