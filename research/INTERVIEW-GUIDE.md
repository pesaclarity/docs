# Phase 0 interview guide

**Goal:** decide whether to build. The gate from the roadmap: **10 of 15 people would grant SMS access** to an app like Pesa Clarity.

Secondary goals: learn which screen earns the first open, which pain is sharpest, and what would stop someone trusting the app.

## Who to talk to

15 people, Kenyan, 22–40, earning roughly 50k–300k KES a month, who use M-Pesa daily. Spread them across the four segments in the spec:

| Segment | Target | Screen for |
| --- | --- | --- |
| Salaried, M-Pesa heavy | 5 | Salary into a bank, most spending on M-Pesa |
| Freelancers and remote workers | 4 | Income in USD/EUR or irregular dates |
| Young families | 3 | Pays rent, school fees or supports relatives |
| People in debt | 3 | Used Fuliza, a mobile loan or a family loan in the last 3 months |

Avoid close friends and family where you can; they'll be polite. One referral from each interviewee is a good way to fill the list.

## Before the call

- 30–40 minutes, video or in person. Have the design canvas open on your phone to show screens.
- Say up front: *"I'm not selling anything. There are no wrong answers. I'm trying to learn how people manage money, not judge it."*
- **Privacy:** don't record their amounts, names of people they pay, or screenshots of their messages. Notes use a code (P01, P02…), not their name. Ask before recording audio; delete audio after notes are written.

## The conversation

Ask about the past, not the future. "What did you do last month?" beats "Would you use…?". Don't pitch until section 4.

### 1. How money moves today (8 min)

1. Walk me through the last time you got paid. Where did the money land, and where did it go in the first week?
2. How do you know how much you have left before the next payday?
3. When did you last run out of money before payday? What happened?
4. Have you used Fuliza, a mobile loan or borrowed from someone in the last 3 months? How do you keep track of what you owe?

### 2. What they've tried (6 min)

5. Have you ever tried to track spending or budget? What did you use (notebook, Excel, an app, bank app)?
6. What made you stop, or what keeps you going?
7. Have you ever downloaded your M-Pesa statement? What did you do with it?

### 3. The sharpest pain (4 min)

8. Of everything we've talked about, what is the most annoying or stressful part of managing money right now?
9. If that disappeared tomorrow, what would change for you?

### 4. Show the designs (10 min)

Show Home first, then let them tap around. Stay quiet; note where they hesitate.

10. What do you think this screen is telling you?
11. Which number here would you look at first? Is anything missing?
12. Show Insights (leakages). *"Would seeing this about your own money be useful, uncomfortable, or both?"*
13. Show Debts. *"Would this help with what you told me about earlier?"*

### 5. The SMS question (5 min)

Explain plainly: *"To work without you typing anything, the app reads M-Pesa and bank SMS on your phone. It only reads messages from the institutions you tick. Nothing leaves the phone; there's no server that sees your transactions."*

14. **Would you allow the app to read those messages?** (Yes / Yes with conditions / No)
15. What would make you more comfortable? What would make you say no?
16. If you said no to SMS: would you share a monthly M-Pesa statement PDF instead?

### 6. Close (2 min)

17. Would you try a test version in a few weeks? (Yes / No; get a contact if yes.)
18. Who else should I talk to?

## After each call

Fill in one row of a copy of [`SCORING-TEMPLATE.csv`](SCORING-TEMPLATE.csv) within an hour, while it's fresh, and note any quotes worth keeping under the participant code.

**Keep results out of this repo.** It will be public. Store the filled sheet and notes in a private place (your own Google Sheet or a private repo), and publish only the aggregated findings here.

## Reading the results

- **Gate:** count "Yes" plus "Yes with conditions" on Q14. 10 or more of 15 → go to Phase 1. Fewer → the product leads with statement import, and SMS becomes optional; revisit the spec before building.
- **Conditions matter:** if most yeses are conditional, the conditions become requirements (e.g. "only if it's open source", "only if I can see what it read").
- **Pain ranking:** tally Q8 answers by theme. The top theme should be what Home shows first.
- **Beta list:** everyone who said yes to Q17.
