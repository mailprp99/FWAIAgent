# DRAFT — PRD: Enquiry Follow-Up Board with Fee at Risk

**Status: DRAFT. Not signed, not agreed, not priced.**
**This document answers one question only: what are we building, and for whom? It does not answer how.**
**No technology choices appear in this document on purpose.**

## READ THESE FIRST — unresolved contradictions

Section 2 lists the places where the client's own words do not add up. I have not resolved
them. At least four of them change what gets built, so decide these before any build starts.

- **C-01** A page she only reads each morning, and a page she types into. One page or two?
- **C-02** "Not yet a signed client" versus "went elsewhere". A lost enquiry is also not signed,
  so today it can never leave the list. What closes an enquiry?
- **C-03** "Days since anyone last spoke to them" when only she logs contact. Who counts as "anyone"?
- **C-04** A brand-new enquiry has never been spoken to. Days since what?
- **C-05** Every enquiry carries a fee, but the fee is not known when the enquiry first arrives.
- **C-06** "One page" sounds small; "every enquiry", a fee, a last-contact date and totals is a
  full enquiry record. Is history in scope?
- **C-07** Is money at risk the whole expected fee, or a share of it?
- **C-08** The office manager's chase list and the owner's money view may not be the same page.

**Source note (assumption A-01):** the only written input supplied was one paragraph describing
the idea. No full interview transcript was attached. Anything not literally in that paragraph is
listed as an assumption at the end of this document.

---

## 1. What was said, and what it means for the build

Each quote below is the client's own wording. Under it is what that wording actually commits us to.

| Their words | What it actually commits us to |
|---|---|
| "Enquiry follow-up board with fee at risk" | The primary object is an enquiry, not a customer or a matter. Money is attached to every enquiry. The product is about follow-up, not intake, not billing. |
| "One page the office manager opens each morning" | One page, one main user, one daily habit. It must be readable in a single sitting. "Each morning" sets the freshness bar: current as of yesterday's close, not live-to-the-second. |
| "lists every enquiry that has not yet become a signed client" | The list is defined by a missing outcome, not by a date or a stage. We must be able to tell "signed client" apart from everything else, which means someone must mark it. It also implies all enquiries, from every route in, land here. |
| "with the fee the firm expects from that matter" | Every enquiry carries a money value. That value is an expectation, not a quote and not an invoice. Someone must set it, and it must be revisable. |
| "the days since anyone last spoke to them" | Every enquiry carries a last-contact moment, and a number is derived from it. If contact is not logged, the number is wrong. "Anyone" implies more than one person may make contact. |
| "She logs what happened" | She is an editor, not a viewer. The page must accept input, not only display. This is the single biggest hidden cost in the idea. |
| "a call, a booked consultation, or went elsewhere" | A small, closed set of outcomes. Each entry has to change the enquiry's state, not just add a note. The set is short enough to be a fixed list, which keeps logging fast. |
| "the totals at the top update the chase list and the money at risk" | Totals are derived, never typed. They must always reflect the latest entries. "Chase list" and "money at risk" are two views of the same open enquiries, sorted by urgency and summed by expected fee. |

---

## 2. What does not add up

The contradictions. I am not resolving these. For each one, the decision you must make is stated.

- **C-01 — Read-only page versus editing page.**
  "One page the office manager opens each morning" describes something she reads. "She logs what
  happened" describes something she types into. A page built for scanning is not the same page as
  one built for fast entry.
  **Decision to make:** is the same page both the morning view and the entry point, or is there a
  separate place to log outcomes?

- **C-02 — "Not yet a signed client" versus "went elsewhere".**
  If an enquiry goes elsewhere, it is still not a signed client, so under the literal rule it stays
  on the list forever. Booked consultations sit in the same grey area.
  **Decision to make:** what exactly closes an enquiry? Signed only, or also went elsewhere, or
  also consulted and completed?

- **C-03 — "Anyone last spoke to them" versus "she logs what happened".**
  If an attorney calls an enquirer and does not log it, the days-since number keeps climbing and the
  chase list becomes false. The idea assumes firm-wide logging discipline but describes one logger.
  **Decision to make:** must everyone log contact, or do we only count contact the office manager
  records?

- **C-04 — Days since what, for a new enquiry.**
  A fresh enquiry has never been spoken to. "Days since anyone last spoke to them" has no starting
  value.
  **Decision to make:** count from the day the enquiry arrived, or start counting only after first
  contact?

- **C-05 — A fee on every enquiry, before the fee is known.**
  "Every enquiry" must carry "the fee the firm expects from that matter", but at enquiry time a
  matter is often unqualified, or out of scope, or unknown.
  **Decision to make:** must every enquiry have a fee before it appears, or may the fee be blank
  until someone sets it?

- **C-06 — "One page" versus a full enquiry record.**
  The phrase sounds light. Counting every enquiry, holding a fee, a last-contact moment, an outcome
  history and running totals is a record-keeping job, not a single view.
  **Decision to make:** is history in scope, or is a single "last contact" field enough to start?

- **C-07 — What "money at risk" actually sums.**
  "The fee the firm expects" and "the money at risk" are used as if identical. A whole expected fee
  is not truly at risk once a consultation is booked.
  **Decision to make:** is money at risk the full expected fee for every open enquiry, or a share
  that changes with stage?

- **C-08 — Two audiences, one page.**
  The office manager wants a chase list: who to call today. The owner wants a money view: how much
  is exposed. The idea gives both to one page, seen by one person.
  **Decision to make:** does the owner get their own view, or do they read over the office
  manager's shoulder on the same page?

---

## 3. Who this is for

The real people who will open this, and what each is trying to get done.

- **The office manager (primary daily user).**
  Tries to get to the end of the day without a single enquirer slipping. Wants to know, in order,
  who to call first, and wants logging an outcome to take seconds so it does not become a chore.
  Also does not want to be the one blamed for a missed follow-up.

- **The firm owner or senior partner (buyer and viewer).**
  Tries to see how much money is sitting in unresolved enquiries and whether the office is working
  them. Wants confidence that nothing quiet is being dropped, and a number they can point to.

- **Fee earners and attorneys (occasional users).**
  Try to know who is waiting on them before a consultation. They will resist extra logging unless it
  is trivial, which is why C-03 matters.

- **Reception and the answering service (source, not really users).**
  They create enquiries. They want a simple way for a new enquiry to reach the page.

- **The enquirer (indirect, never a user).**
  Wants a fast, personal call back. Never opens the page, but every design choice about speed of
  follow-up is ultimately for them.

- **You, the seller (the person selling this).**
  Need the page to show visible value every month so the monthly price stays justified.

---

## 4. Scope, locked

What is in.

1. **One page, for one firm, opened daily** by the office manager as the morning view of open
   enquiries. [A-05, A-08]
2. **The open enquiry list.** Every enquiry that has not yet reached whatever the client defines
   as closed (see C-02). Each row shows the enquirer, how they reached the firm, the practice area
   or matter type, the expected fee, the date of last contact, and the days since last contact.
3. **Outcome logging** in a short fixed set: a call, a booked consultation, went elsewhere (final
   set to be confirmed — see C-02). Logging an outcome records the time and moves the enquiry's
   state. It must be fast enough to do in seconds.
4. **A chase list.** The open enquiries ordered so the ones waiting longest sit at the top, so the
   order of calls is obvious without thinking.
5. **Totals at the top**, derived and never typed: the number of open enquiries, the total expected
   fee at risk, and how many enquiries have gone untouched past a threshold (threshold to be set —
   see dependencies).
6. **Adding an enquiry** by hand when one arrives outside the normal routes, so nothing is invisible
   to the board. [A-06]
7. **Persistence.** What she logs is still there tomorrow morning, and visible to whoever else has
   access, without re-entering anything.
8. **Basic access for the firm's own staff only.** Enquirers never see the page. [A-17]

---

## 5. Not building, and why

What is out. This is the section to read back to the client line by line.

- **Not intake or conflict checking.** Deciding whether the firm can take a matter is a legal and
  risk decision, not a follow-up one.
- **Not case or matter management.** The firm already runs its matters somewhere; this page does
  not replace that.
- **Not sending messages to enquirers.** No calls, texts or emails go out from this. The firm makes
  contact personally; the page only tells them who to contact.
- **Not automated reminders to staff.** The morning page is the reminder. Adding nudges changes the
  product and the price.
- **Not a phone system or call logging.** Contact is recorded by hand (see C-03). We do not capture
  calls automatically.
- **Not billing, invoicing or time recording.** The fee here is an expectation, not a charge.
- **Not lead-source or marketing reporting.** We may record how an enquiry reached the firm, but we
  do not analyse campaigns or spend.
- **Not broad reporting, trends or exports.** Beyond the totals at the top, none, until a later
  phase proves it is wanted.
- **Not multiple offices or multiple firms.** One firm, one office to start. [A-08]
- **Not for use away from the office.** It is opened at the office during the working day. [A-09]
- **Not roles and permissions beyond basic access.** Who may see and who may edit is kept simple at
  first. [A-05]
- **Not using enquirer data for anything except follow-up.** No marketing, no resale, no analysis
  beyond the chase list.

---

## 6. Phasing

What ships first, what waits.

**Phase 0 — Decisions and inputs, before any build.**
Resolve the Section 2 contradictions. Agree the definition of a signed client. Collect the
practice-area list with expected fee values. Name the people who will have access. Get one month of
baseline numbers.

**Phase 1 — The daily page (first ship).**
One page. The open enquiry list. The row fields: enquirer, source, practice area, expected fee, last
contact, days since contact. The fixed outcome logging set. Totals at the top for open count and
money at risk. Chase ordering by days since contact. One editor: the office manager, one firm, one
office.

**Phase 2 — Team and history.**
Firm-wide contact logging so "days since" stays true. A history per enquiry instead of one
last-contact date. A separate owner view. A configurable threshold for "gone quiet". Settled
handling of lost and booked enquiries.

**Phase 3 — Less manual work.**
Reducing duplicate entry by drawing enquiries from where the firm already records them. Support for
a second office. Reporting and trends, if the firm asks.

**Waits:** every item in Section 5, and everything in Phase 2 and Phase 3.

---

## 7. Success metrics

How we will know it worked, in numbers. All targets are proposed, not agreed. [A-15]

| What we measure | Proposed target | Why it matters |
|---|---|---|
| Mornings the page is opened | 4 of 5 weekdays | If it is not opened, nothing else counts. |
| Open enquiries with a last-contact date recorded | 95% or more | The chase list is only as good as this. |
| Open enquiries contacted within the last 7 days | 80% or more | Proves enquiries are actually being worked. |
| Open enquiries older than 14 days with no contact | Zero | This is the exact failure the client described. |
| Time to log one outcome | Under 30 seconds | Slow logging is why systems like this get abandoned. |
| Enquiries reaching a logged outcome within 48 hours | 90% or more | Measures how fast the office responds. |
| Money at risk | Falls week on week as enquiries close | The number the owner will actually watch. |
| Signed clients per 100 enquiries | Up on the firm's own baseline | The business result. Baseline currently unknown. |

---

## 8. Open dependencies

What I am waiting on from someone else. Most of this is the client.

1. **The definition of a signed client** — needed before anything is built (C-02).
2. **Whether lost enquiries stay on the list or drop off** (C-02).
3. **The practice-area list**, and the expected fee value for each. [A-14]
4. **The overdue threshold** — how many days of no contact counts as "gone quiet".
5. **Who may log outcomes**, and an honest view on whether attorneys will (C-03).
6. **The list of people who need access**, and what each may see.
7. **Baseline numbers:** enquiries per month, average expected fee, current signed-client rate.
8. **Where enquiries are recorded today**, and whether this page sits alongside it or replaces it.
   [A-06]
9. **One named decision-maker** on the client side for scope sign-off. [A-18]
10. **Confirmation of the interview record.** The only source provided was one idea paragraph; a
    fuller interview transcript would either confirm or change assumptions A-01 through A-18.

---

## Assumptions

Every place where I had to assume something a real client would have told us. These are mine, not
the client's. Each needs a yes, a no, or a number.

- **A-01** — The single idea paragraph is the complete interview input; no fuller transcript exists.
- **A-02** — The buyer is a solo or small law firm, 2 to 10 attorneys, in the United States.
- **A-03** — "Signed client" means a signed engagement accepted and the matter opened.
- **A-04** — "The fee the firm expects" is one estimated amount per matter, set at or near enquiry
  time, revisable later.
- **A-05** — The office manager is a non-lawyer staff member and the only daily editor.
- **A-06** — The firm already records enquiries somewhere; this page sits alongside that record
  rather than replacing it.
- **A-07** — Enquiries reach the firm by phone, through the firm's website, and through its
  answering service.
- **A-08** — One firm, one office, for the first version.
- **A-09** — The page is used at the office during the working day, not away from the office.
- **A-10** — It is acceptable for "days since last contact" to be based only on contact that was
  logged.
- **A-11** — A booked consultation is progress, not a closed enquiry.
- **A-12** — "Went elsewhere" means the enquiry is lost and should stop appearing on the active
  list. The client must confirm (C-02).
- **A-13** — "Each morning" means one check at the start of the working day, Monday to Friday.
- **A-14** — The firm can supply a practice-area list with an expected fee for each.
- **A-15** — No budget, deadline or success targets were given; the phasing and metrics above are
  proposals.
- **A-16** — Enquiry volume is small enough for one person to manage by hand. A number is needed.
- **A-17** — Only the firm's own staff see the page; enquirers never do.
- **A-18** — The client will name a single decision-maker for scope sign-off.
