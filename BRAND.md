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
| Pesa Red (`brand`) | `#C8102E` | A fill only: the lead card, the + button, the open tab, the one main button on a screen, and danger. White text on it: 5.9:1 |
| Links (`brandLink`) | `#1C1215` | Links and other buttons use the text colour, with a chevron |
| Red soft (`brandSoft`) | `#FDECEE` | Subtle fills |
| Selected (`selected`) | `#ECE4E1` | A selected chip or menu choice: a neutral tint, so red stays where it counts |
| Background | `#F6F1EE` | Warm screen ground, so white cards stand off it |
| Surface / border | `#FFFFFF` / `#EFE6E7` | Cards, lists, nav bar |
| Text / muted | `#1C1215` / `#6F6064` | Body, labels (muted 5.6:1) |
| Success | `#059669` (text `#047857`) | Income, on track, cleared |
| Warning | `#B45309` | In progress, overdue, something to check |
| Warning, stronger (`warningStrong`) | `#C2410C` | The worse of two warnings, such as Struggling beside Coping |

**Red means brand, so "over budget" can't rely on colour alone.** Every over or failed state also carries a label or icon ("Over", ⚠ icon). Green stays a status colour only, which also keeps us visually distinct from M-Pesa.

Dark mode keeps the same red:

| Token | Hex |
| --- | --- |
| Background / surface / border | `#140E10` / `#1E1618` / `#352B2E` |
| Fields and chips / floating parts (tab bar, sheets, menus) | `#241B1E` / `#2A2124` |
| Text / muted | `#E3D9DB` / `#A3969A` |
| Brand (white text on it) | `#C8102E` |
| Danger (`danger`, fills and large figures) | `#C8102E` |
| Danger in small text (`dangerText`) | `#FF3C5B` |
| Links | `#E3D9DB`, the text colour |
| Red soft (`brandSoft`) / selected | `#2F2D2C` / `#383534`, neutral tints, never pink |
| Success (text `#6EE7B7`) | `#34D399` |
| Warning / stronger | `#FBBF24` / `#FB923C` |

Dark text is moderate rather than maximal (12.5:1 on a card), easier on tired eyes. Brand red is a fill and a large figure on dark (3.0:1 on a card); small red text, such as a field's error, uses `#FF3C5B`, the same hue brightened only as far as it needs (4.5:1 on a sheet, 5.1:1 on a card). Dark has no drop shadows; floating parts sit on a lighter surface instead.

Machine-readable values: [`brand/tokens.json`](brand/tokens.json) (W3C design-token format).

## Type

- **Space Grotesk** for numbers and titles. Tabular figures for amounts.
- **Inter** for body text and labels.
- Both are SIL Open Font License 1.1, so they can ship inside the app and the F-Droid build.

## Shape

20 px card radius, 44 px minimum touch target, floating pill navigation, one primary number per screen, no horizontal scrolling. Spacing, sizes and how each screen, sheet and dialog is built are in [brand/ui-rules.md](brand/ui-rules.md).

## Licence

The logo and name are trademarks of the project and are **not** covered by the CC BY 4.0 licence on this repository's text. Forks of the app must use a different name and logo, so clones cannot pose as Pesa Clarity and harvest SMS.
