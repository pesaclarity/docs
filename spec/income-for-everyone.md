# Income for everyone

**Status:** Accepted · **Date:** 2026-11-03

## Problem

Pesa Clarity assumes a monthly salary: Plan is a month with expected income, safe to spend divides
the month's day-to-day budget by the days left, and "payday" is a salary-sized deposit. Interviews
with students, jua kali and small-business people and gig workers showed most of them have no
payday. Money comes from parents at uneven gaps, from customers every day, from gigs now and then,
or at harvest. Many file nil tax returns. Today they see "—" where safe to spend should be.

## People and patterns

| Pattern | Who | How money comes |
| --- | --- | --- |
| Salary | Employees | Monthly, every two weeks or weekly |
| Business or hustle | Jua kali, mama mboga, boda, shops, tills | Many small payments, most days |
| Gigs and casual work | Freelance, mjengo, delivery | Now and then, amounts vary |
| Family or sponsor support | Students, dependants | From a few people, uneven gaps |
| Allowance, bursary or HELB | Students | A few times a term |
| Seasonal | Farmers | A few times a year |
| Nothing steady right now | Between jobs | Rarely or not at all |

A person can have **several** patterns at once (a student with support and gigs, a salaried person
with a hustle).

## Decision

### 1. Ask, then learn (stage 1)

- Setup asks "How does money usually come to you?" with the patterns above as a multi-select. It
  can be changed in Profile.
- From the messages, the app learns the income pattern: the typical gap between money arriving, how
  much it varies, and how many people send it. Borrowed money (Fuliza, M-Shwari, mobile loans,
  family loans), reversals and own transfers are never income.
- The learnt pattern is a suggestion shown next to the choice; the person's choice wins.

### 2. Purpose per transaction, not per payee

- A payee's usual category is a suggestion only. Each transaction keeps its own category, so money
  from the same person can be family support one week and a loan the next (this already works
  through the per-transaction category).
- Stage 2: when a sender's past money has had different purposes, a new transfer from them is not
  filled in automatically; it asks "What was this?" (family support, payment for work, a loan to
  you, paying you back, a gift, a sale).
- Stage 2, with pesaclarity-ml: income categories split into family support, allowance and bursary,
  and gifts, alongside salary, business and freelance, and money received. The categoriser's
  categories change together (categories.yaml version 3).

### 3. Plan until the next money (stage 1)

Safe to spend today:

> (money on hand − bills due before the next money − a small buffer) ÷ days until the next money

- Money on hand is the balance of day-to-day accounts (not Saccos or money market funds).
- The buffer is a tenth of the money on hand.
- When the next money is likely:

| Pattern | Next money | Counted |
| --- | --- | --- |
| Salary with a Plan | The pay date | The month's day-to-day budget (unchanged) |
| Regular support or allowance | Last arrival + the typical gap | Only what has arrived |
| Business, gigs, irregular | A rolling 7 days | Only what is on hand |
| Seasonal, nothing steady | No date: shows how long it lasts at today's pace | Only what is on hand |

- Home's wording follows: "Until about Fri 14 Nov, when money usually comes", "For the next 7
  days", and "Lasts until about … at this pace". "Payday" appears only for salaried people.
- A monthly Plan stays optional for everyone.

### 4. Plans for each pattern (stage 2)

- **Monthly plan:** today's Plan.
- **Make it last:** when money arrives, a split is suggested (rent and fees first, then a daily
  amount until the next money), with an early warning if it will run out first.
- **Week by week:** for business and gigs. Good days suggest a share into a smoothing pot
  (kibubu); lean days may draw from it. Its target is a typical bad week.

### 5. Spaces: personal and business (stage 3)

- One app holds several spaces (Personal, Business, more if needed). Each has its own encrypted
  database: accounts, categories, Plan, goals, insights and backup. The top of Home switches space.
- Each space can sign in to its own account for backups, or share one.
- Messages are routed per space: an account belongs to one space, and rules can send till and
  Pochi la Biashara payments on a shared M-PESA line to Business.
- Moving money from Business to Personal is the owner's pay: income in Personal, a draw in Business.
- Business gets sales, stock, supplier and wages categories, and a monthly profit view.

### 6. A fair health score (stage 4)

- Income is the last 90 days, not one month.
- The emergency cushion is counted in days of essentials covered.
- Plan-keeping is checked week by week for irregular earners.
- Income steadiness is shown for information only, never as points lost.

### 7. Tax

- The tax reserve is optional and off by default unless income is salary or business.
- People with no or low taxable income get a reminder to file a nil return on iTax before 30 June.

### 8. Simple use (stage 4)

- An optional simple Home: today's amount and the next important thing.
- Chama contributions and mobile-loan repayments are first-class categories.
- Local words stay where people use them (chama, Fuliza, kibubu, jua kali); the app itself stays in English, since it is used beyond East Africa.

## Stages

1. Income sources at setup and in Profile, the learnt pattern, safe to spend until the next money,
   and Home's wording.
2. "What was this?" for mixed senders, new income categories (with pesaclarity-ml), make-it-last
   and week-by-week plans, the smoothing pot.
3. Spaces for personal and business.
4. The fair health score, simple Home, the nil-return reminder.

## Privacy

Unchanged: patterns are learnt on the phone from the ledger; nothing new leaves it. Each space is
its own encrypted database.
