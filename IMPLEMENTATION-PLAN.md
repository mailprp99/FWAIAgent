# IMPLEMENTATION PLAN — Enquiry Follow-Up Board with Fee at Risk

**Status: DRAFT.**
**This document answers one question: in what order, and what is demonstrable at each step?**
Inputs: `PRD.md` (what and for whom), `TECH-STACK.md` (what with, and why).

**The ordering rule, which overrides everything else:** every step below ends in something you can
open on a phone and look at. Not "the schema is migrated". Something on a screen. If a step cannot
end that way, it is two steps or it is in the wrong place.

**Verification is part of every step.** A step is not done when it is built; it is done when it is
built, tested on a phone, and the result has been seen. "Built" without "tested" does not count as
done, and there is a test named for each step.

**Two rules on order.** Expensive-to-undo decisions go early, while the codebase is nearly empty.
And every step is placed to catch its mistake before the next step builds on it — a mistake at step
two is cheap, the same mistake at step nine means unwinding everything above it.

Total: **10 half-days (5 days)** across 8 steps. That is a full week with no slack, which is exactly
why the cut list in the last section is decided now.

---

## Gates — answers needed before certain steps

These come from the PRD's unresolved contradictions (Section 2) and dependencies (Section 8). Missing
answers do not stop the build; they are built against the reversible default below and switched later.

| Gate | Needed before | Default if unanswered |
|---|---|---|
| C-02 What closes an enquiry | Step 4 | "Went elsewhere" and "signed" both close it; status is a field, so this is a setting |
| C-05 May the fee be blank | Step 4 | Yes; blank counts as zero in totals |
| C-04 Days since arrival, or since first contact | Step 5 | Store both dates; display since last contact, fall back to arrival |
| C-07 Money at risk = full fee or a share | Step 5 | Full expected fee of every open enquiry |
| The "gone quiet" threshold | Step 5 | 14 days, hardcoded for the first ship |
| C-03 Must everyone log contact | Step 7 | Only logged contact counts; anyone may log |
| C-08 Does the owner need a separate view | Step 8 | No separate view at first; the totals serve the owner |
| Practice-area list and fee values | Step 2 (seed only) | A short placeholder list, replaced before the pilot |

---

## Overview

| # | Step | Half-days | What you can see on a phone | Mistake this catches |
|---|---|---|---|---|
| 1 | Walking skeleton on a real URL | ½ | A live URL showing a board of fake enquiries and totals | Is the idea legible on a phone at all; does deploy work |
| 2 | Data model and a real read | 1 | The same board, driven by a database you can change | The expensive schema shape, fixed while free |
| 3 | Sign-in and firm isolation | 1½ | Log in, see only your firm; a second firm sees only its own | The most expensive decision in the stack (identity + tenancy) |
| 4 | Log an outcome | 1½ | Tap a row, log a call, watch it reorder and totals move | The hidden cost: is logging fast enough to actually happen |
| 5 | Derived truth: order, days, totals | 1 | A seeded board whose numbers you can check by hand | Wrong money numbers, before anything depends on them |
| 6 | Add an enquiry by hand | 1 | Add an enquiry on the phone and see it land | Whether the required fields are the right ones |
| 7 | Firm-wide logging and history | 1½ | A second user's log appears for the first; one enquiry's history | Whether more than one person will really log |
| 8 | Owner view and the production switch | 2 | The production URL on a phone, with backups confirmed | Shipping to real client data without backups or errors surfaced |

Phase 1 (the MVP) is steps 1–6. Phase 2 is steps 7–8.

---

## Phase 1 — The daily page (the MVP)

### Step 1 — Walking skeleton on a real URL  ·  ½ day
**What gets built.** A deployed page at a real URL with six hardcoded fake enquiries, the three
totals at the top, and one row that opens to show the three outcome buttons (they do nothing yet).
Phone-first layout. No database, no login.
**What you can see on the phone.** A live URL you can send yourself, showing the board as it will
look, on a real phone.
**Mistake caught.** The largest product risk is not technical: does this screen make sense on a
phone, and does the owner understand it in five seconds. Also proves hosting and deploy work.
**Verification.** Open the URL on an iPhone and an Android; confirm no horizontal scrolling, the
totals are visible without scrolling, and tap targets are finger-sized. Record the URL and a
screenshot as evidence.

### Step 2 — Data model and a real read  ·  1 day
**What gets built.** The database schema: firms; users (a canonical record keyed to the login id);
enquiries (firm id, arrived at, last contacted at, expected fee that may be blank, status); and
outcomes as a new row every time, never an overwritten field. One firm seeded. The page now reads
from the database through a server query.
**What you can see on the phone.** The same board, now driven by data. Change a value in the
database, refresh the phone, watch it change.
**Mistake caught.** The expensive schema shape is fixed here, on day two: outcomes as an append-only
log, and both dates stored. Getting this wrong later loses history that cannot be backfilled.
**Verification.** Confirm the phone shows exactly the seeded rows; confirm a blank expected fee
renders; confirm the outcomes table accepts a row.

### Step 3 — Sign-in and firm isolation  ·  1½ days
**What gets built.** Login by email and password. Each user maps to one firm. Every table carries the
firm and is protected twice: a mandatory filter in the code and a database rule as a backstop.
**What you can see on the phone.** A login screen; after logging in, your firm's board; log out; a
second seeded firm logs in on another phone and sees only its own rows.
**Mistake caught.** This is the most expensive decision in `TECH-STACK.md` (Section 10). It is
exercised on day two, while the codebase is small, instead of being discovered at step nine.
**Verification** (this one is adversarial; the easy version proves nothing). Sign in as firm A. Try to
open a firm B enquiry by changing its id in the URL. Confirm the request is refused. Run a query under
A's session and confirm no firm B row is readable. List every table and confirm the isolation rule is
on all of them. Record each result.

### Step 4 — Log an outcome  ·  1½ days
**What gets built.** Tap an enquiry, choose a call, a booked consultation, or went elsewhere. That
writes an outcome row, updates the last-contacted time, recomputes the days and the totals, and
reorders the chase list. "Went elsewhere" closes the enquiry and drops it from the active list.
**What you can see on the phone.** Tap a row, log a call, watch it fall down the list and the days
reset to zero. Log "went elsewhere", watch it leave and the money at risk fall.
**Mistake caught.** The PRD's single biggest hidden cost is "she logs what happened". If logging is
slow or fiddly, it must be found here, before the owner view and everything else is stacked on top.
**Verification.** Log one outcome on a phone while timing it; the PRD target is under 30 seconds.
Then confirm the outcome row exists with the right time and the right person, the days and totals
changed correctly, and the row moved.

### Step 5 — Derived truth: chase order, days since, threshold, totals  ·  1 day
**What gets built.** One query that produces the ordered chase list, the days-since number, the "gone
quiet" count, and the three totals. One query, so the list and the sums cannot disagree.
**What you can see on the phone.** A board seeded with known data, whose order and totals can be
checked by hand against the screen.
**Mistake caught.** A wrong money number is the silent way this product loses trust. Caught before
anything is built on the derivation.
**Verification.** Seed a fixed dataset; assert the displayed order and totals equal hand-computed
values. Write this as one automated test that runs on every change, and also read the same numbers off
the phone. Both, because the phone is the product.

### Step 6 — Add an enquiry by hand  ·  1 day
**What gets built.** A short form so reception can add an enquiry that arrived outside the normal
routes, with the expected fee allowed to be blank.
**What you can see on the phone.** Add an enquiry on the phone; it appears in the right place in the
chase order; a blank fee does not break the totals.
**Mistake caught.** The last scope item, and the point where the office manager discovers whether the
form asks the right questions. Cheaper to learn here than after launch.
**Verification.** Add one enquiry with a fee and one without. Confirm both appear, the blank does not
change the money at risk, and both survive a refresh and a second login.

### Phase 1 exit criteria — tick these
- [ ] A real URL opens on a phone and shows the board.
- [ ] The board reads from a real database, not hardcoded data.
- [ ] A login is required; two firms provably cannot see each other's rows (adversarial test passed).
- [ ] One outcome can be logged on a phone in under 30 seconds.
- [ ] The chase list orders by days since contact and reorders immediately after a log.
- [ ] The three totals match hand-computed values on a seeded dataset, and that check is an automated test.
- [ ] A blank expected fee does not break the totals.
- [ ] An enquiry added by hand survives a refresh and is visible to a second login.
- [ ] Every line above has a recorded test result, not just a build.

---

## Phase 2 — Team and history

### Step 7 — Firm-wide logging and per-enquiry history  ·  1½ days
**What gets built.** More than one user per firm; the log records who did it; a history view on an
enquiry that lists every outcome in order (the rows already exist from step 2).
**What you can see on the phone.** A second login in the same firm sees the first person's logged
outcome, and one enquiry opens to show its history.
**Mistake caught.** Whether anyone other than the office manager will log. This is PRD C-03, and it is
a behaviour risk, not a code risk. Find out here.
**Verification.** Two logins in one firm; each sees the other's entry; the history order matches the
timestamps; each entry shows who logged it.

### Step 8 — Owner view and the production switch  ·  2 days
**What gets built.** A money-focused view for the owner if the answer to C-08 says yes; empty states
and error messages; then the production switch: paid plans, daily backups on, domain, free error
tracker.
**What you can see on the phone.** The production URL on a phone, the owner view, and a list of
available database backups.
**Mistake caught.** Shipping real client data without backups, or with errors going nowhere. The
production switch is a step in the plan precisely so it cannot be forgotten on the last night.
**Verification.** The production URL loads on a phone; deliberately trigger one test error and confirm
it reaches the error tracker; list the backups and confirm the most recent one exists; confirm the
production app still refuses a firm B row.

### Phase 2 exit criteria — tick these
- [ ] A second user in the same firm sees the first user's outcome, and history shows who did what.
- [ ] The owner has a money view, or the totals are signed off as sufficient for the owner.
- [ ] The production site runs on paid plans (host and database) with daily backups listed.
- [ ] A deliberately triggered error reaches the error tracker.
- [ ] The person who will use this daily has opened the production URL on their own phone and approved it.

---

## Decisions most expensive to reverse, and where they happen

Placed early on purpose. At steps 1–3 the codebase is nearly empty and the change costs hours; at
step 8 the same change touches every table, every query and every login.

| # | Decision | Where it is made | Why there |
|---|---|---|---|
| D1 | Identity and firm isolation boundary (login vendor + firm field + database rule) | **Step 3** | Most expensive to undo (TECH-STACK Section 10). Exercised on day two, proven adversarially before any feature sits on it. |
| D2 | Outcomes stored as an append-only log, not an overwritten field | **Step 2 schema, exercised in Step 4** | History not captured cannot be backfilled. One table now, impossible later. |
| D3 | Many firms in one database, separated by a firm field | **Step 2** | Changing the tenancy shape later is a rewrite plus a data migration. |
| D4 | Both "arrived at" and "last contacted at" stored | **Step 2** | Makes PRD C-04 reversible for free; you can switch which date is shown without a migration. |
| D5 | Hosting, framework and runtime | **Step 1** | Cheap to change per TECH-STACK Section 9; proven on day one, not assumed. |
| D6 | What closes an enquiry (PRD C-02) | **Step 4** | A setting, but once a year of real outcomes exists, changing it rewrites history. Decide while the history is small. |

---

## What gets cut if the week slips

Decided now, calmly. Cut from the top. Each cut is safe because something else already covers the
need, and the client never sees a broken page.

1. **The owner view (Step 8).** The totals at the top already give the owner the number; they can read
   the same page. Saves about a half day. The owner sees the money, just without their own screen.
2. **The per-enquiry history screen (Step 7).** Keep writing outcome rows — nothing is lost — and drop
   the screen. Saves about a half day. History is still there the day it is wanted.
3. **The configurable "gone quiet" threshold (Step 5).** Hardcode 14 days for the first ship. Saves
   about a half day. One number, changed later as a setting.
4. **The manual add-enquiry form (Step 6).** Reception enters enquiries another way for now. Saves
   about a day. This one is high value, so it is cut last.
5. **All of Phase 2.** If the slip is larger than the cuts above, ship Phase 1 to a single pilot firm
   and stop. The daily page is the product; the rest is improvement.

**Never cut, at any cost:** firm isolation (Step 3), logging an outcome (Step 4), derived totals being
correct (Step 5), and backups at production (Step 8). A cut here does not make the week easier; it
makes the product unsafe or untrue.

---

## Why this order catches mistakes early

- **Step 1** catches the product risk — legibility on a phone — in half a day, before any code exists.
- **Step 2** catches the data shape while the schema holds one seeded firm.
- **Step 3** catches the security boundary before there are features worth breaching.
- **Step 4** catches the behavioural risk — will logging actually happen — before it is built upon.
- **Step 5** makes the numbers trustworthy before anything reads them.
- **Steps 6–8** are additive once the foundation is proven; by then a mistake is a change, not an
  unwinding.
