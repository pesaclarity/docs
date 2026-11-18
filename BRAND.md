# Pesa Clarity brand

<img src="brand/logo-wordmark.svg" alt="Pesa Clarity" width="420">

Pesa Clarity is an independent project. It shares no brand, code or ownership with any employer or client.

## Name

- **Pesa Clarity** in prose, two words, both capitalised.
- **pesaclarity** in the wordmark, handles and identifiers (`pesaclarity` org, `com.pesaclarity.app`).
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
- Minimum size: 24 px for the mark, 120 px wide for the wordmark.
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

Dark mode keeps the same red:

| Token | Hex |
| --- | --- |
| Background / surface / border | `#0A0A0A` / `#171717` / `#2E2E2E` |
| Fields and chips / floating parts (tab bar, sheets, menus) | `#1F1F1F` / `#242424` |
| Text / muted | `#E5E5E5` / `#A3A3A3` |
| Brand (white text on it) | `#C8102E` |
| Danger (`danger`, fills and large figures) | `#C8102E` |
| Danger in small text (`dangerText`) | `#FF4D6A` |
| Accent (`brandLink`: links, current tab, progress, chosen options) | `#FF4D6A` |
| Soft fill (`brandSoft`) / selected | `#262626` / `#383838`, neutral greys, never pink |
| Success (text `#6EE7B7`) | `#34D399` |
| Warning / stronger | `#FBBF24` / `#FB923C` |

Every neutral, light and dark, is a pure grey with no warm or rose tint. Dark text is moderate rather than maximal (14.2:1 on a card), easier on tired eyes. Brand red is a fill and a large figure on dark (3.0:1 on a card); small red text and thin strokes, such as the accent or a field's error, use `#FF4D6A`, the same hue brightened only as far as it needs (4.8:1 on a sheet, 5.6:1 on a card). Dark has no drop shadows; floating parts sit on a lighter surface instead.

Machine-readable values: [`brand/tokens.json`](brand/tokens.json) (W3C design-token format).

## Type

- **Space Grotesk** for numbers and titles. Tabular figures for amounts.
- **Inter** for body text and labels.
- Both are SIL Open Font License 1.1, so they can ship inside the app and the F-Droid build.

## Shape

20 px card radius, 44 px minimum touch target, floating pill navigation, one primary number per screen, no horizontal scrolling. Spacing, sizes and how each screen, sheet and dialog is built are in [brand/ui-rules.md](brand/ui-rules.md).

## Licence

The logo and name are trademarks of the project and are **not** covered by the CC BY 4.0 licence on this repository's text. Forks of the app must use a different name and logo, so clones cannot pose as Pesa Clarity and harvest SMS.
