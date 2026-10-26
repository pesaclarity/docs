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
- Kenyan English by default; Swahili and Sheng-friendly wording after beta.

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
| Pesa Red (`brand`) | `#C8102E` | Hero cards, active tab, primary buttons. White text on it: 5.9:1 |
| Link red (`brandLink`) | `#B81D2F` | Links and alerts on white (6.5:1) |
| Red soft (`brandSoft`) | `#FDECEE` | Selected rows, chips |
| Background | `#FBF8F8` | Screen ground |
| Surface / border | `#FFFFFF` / `#EFE6E7` | Cards, lists, nav bar |
| Text / muted | `#1C1215` / `#6F6064` | Body, labels (muted 5.6:1) |
| Success | `#059669` (text `#047857`) | Income, on track, cleared |
| Warning | `#B45309` | In progress |

**Red means brand, so "over budget" can't rely on colour alone.** Every over or failed state also carries a label or icon ("Over", ⚠ icon). Green stays a status colour only, which also keeps us visually distinct from M-Pesa.

Dark mode:

| Token | Hex |
| --- | --- |
| Background / surface / border | `#1A1214` / `#251B1E` / `#3D2E32` |
| Text / muted | `#EDE4E6` / `#B3A5A9` |
| Brand (text on it `#1A1214`) | `#FF6B7A` |
| Link red | `#FF8592` |
| Red soft | `#3A1D23` |
| Success (text `#6EE7B7`) | `#34D399` |
| Warning | `#FBBF24` |

All text pairs are 6:1 or better on the dark surface. On dark, text on brand fills is dark (`#1A1214`), not white.

Machine-readable values: [`brand/tokens.json`](brand/tokens.json) (W3C design-token format).

## Type

- **Space Grotesk** for numbers and titles. Tabular figures for amounts.
- **Inter** for body text and labels.
- Both are SIL Open Font License 1.1, so they can ship inside the app and the F-Droid build.

## Shape

20–28 px card radius, 44 px minimum touch target, floating pill navigation, one primary number per screen, no horizontal scrolling.

## Licence

The logo and name are trademarks of the project and are **not** covered by the CC BY 4.0 licence on this repository's text. Forks of the app must use a different name and logo, so clones cannot pose as Pesa Clarity and harvest SMS.
