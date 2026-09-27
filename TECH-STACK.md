# TECH STACK — Enquiry Follow-Up Board with Fee at Risk

**Status: DRAFT.**
**This document answers one question: what do we build it with, and why that?**
Inputs: `PRD.md` (the requirements and features) and the scale: **150 users**.

Every choice below is paired with the constraint that forced it. A choice without a constraint is a
preference, and preferences get argued about later. Each layer also names what was rejected, so it
does not get re-litigated in three months.

Pricing and plan limits cited here were read from the vendors' own pricing pages on 2026-09-13.
Where a number is not from that reading, it is marked UNVERIFIED.

---

## 0. The scale, read literally

150 users. Not 150 concurrent sessions, 150 named people with logins. That is the whole number.

Reading it against the PRD (office manager, owner, one or two fee earners per firm):

- ~4 users per firm → **about 35 to 50 firms**.
- One page opened once each morning → **~150 page loads per day**, ~3,000 per month.
- Outcome logging a few times per firm per day → **~200 writes per day**, ~5,000 per month.
- Enquiry rows: even at an aggressive 1,000 enquiries per firm per year, 40 firms is ~40,000 rows a
  year. With outcomes, maybe a few hundred thousand rows over several years — tens of megabytes.

Two conclusions that shape everything below:

1. **Traffic is never the problem at this scale.** Nothing here comes within two orders of magnitude
   of a free tier's capacity limits. If a plan limit is going to bite, it will not be request count.
2. **"150 users" means many firms, not one.** The PRD says one firm, one office for v1, but 150 users
   only adds up across dozens of firms. So the data model must separate firms from day one even
   though the screen shows one firm. This is the seed of the most expensive decision (Section 10).

---

## 1. Tenancy and data model

| | |
|---|---|
| **Choice** | One shared relational database. Every business table carries a `firm_id`. Isolation is enforced twice: a mandatory tenant filter in application code, and row-level security in the database as a backstop. |
| **Constraint that forced it** | 150 users across dozens of firms need separation; one shared store is the cheapest thing that provides it. A database per firm multiplies provisioning, migrations and backups by 40+ for no benefit at this volume. |
| **Rejected** | **Database per firm** — operationally heavy, painful to migrate, 40× the backup and connection overhead. **Schema per firm** — same problem, worse tooling. **Separate app per firm** — nonsense at this size. **No isolation (single-firm product)** — breaks the moment there is a second customer, and is the one change that would force a data migration of everything. |
| **Reversal cost** | Changing the tenant key shape later is a rewrite with a data migration. See Section 10. |

The isolation is applied twice on purpose. Application filtering alone leaks if one query forgets its
`firm_id`; the database rule alone is easy to disable by accident. Two layers means one mistake is not
a breach.

---

## 2. Database

| | |
|---|---|
| **Choice** | **Managed Postgres** — Supabase on the free tier to start, its paid tier in production. |
| **Constraint that forced it** | The core screen is sums and orderings over rows ("money at risk", "days since", chase order) plus simple forms. That is exactly what a relational database is for. Postgres is the default choice with no vendor-specific query language to learn. |
| **Free tier** | 500 MB database, 50,000 monthly active users, unlimited API requests, 5 GB egress, 1 GB file storage. Free projects **pause after 1 week of inactivity**; limit of **2 active projects**. Sources: supabase.com/pricing, verified 2026-09-13. |
| **At 150 users** | Storage is low tens of MB per year. The 500 MB cap is years away. Pausing only bites if nobody opens the page for a week, which the daily-morning habit prevents. |
| **Rejected** | **SQLite / embedded file database** — fine for one firm, weak for concurrent multi-tenant writes and for managed backups. **MySQL / PlanetScale** — no meaningful free tier now, and no advantage here. **MongoDB** — no clean relational sums across tenants, and the money-at-risk totals become application code instead of a query. **DynamoDB / key-value** — you would bake every access pattern into the key design and lose ad-hoc queries; punitive to change. **Self-hosted Postgres** — pure ops cost with no upside at this scale. |
| **Reversal cost** | Staying on Postgres is the point. Moving between managed Postgres vendors is a dump and restore, not a rewrite. |

---

## 3. Authentication

| | |
|---|---|
| **Choice** | **Supabase Auth** (email plus password, and email magic links), reusing the same account already used for the database. |
| **Constraint that forced it** | Staff-only logins, ~150 people, sessions, password reset, and a `firm_id` attached to each user. The free tier covers 50,000 monthly active users — 300× the stated scale. Building login is a security liability and pure cost. |
| **Rejected** | **Clerk** — excellent, but a second vendor, a second bill, and one more place user identity lives. **Auth0** — enterprise pricing and configuration for 150 people. **Rolling our own** — never; password storage and reset are exactly where small products get breached. **Plain shared password per firm** — no per-user accountability, and "who logged this outcome" (PRD C-03) becomes unanswerable. |
| **Reversal cost** | This is half of the most expensive decision in this document. See Section 10. |

Note on email: Supabase allows a **custom SMTP server on the free tier**, but removing Supabase
branding from auth emails is paid-only (supabase.com/pricing, verified 2026-09-13). At this scale,
bring a free transactional email account for password resets, or accept the branding.

---

## 4. Frontend framework

| | |
|---|---|
| **Choice** | **Next.js (App Router) with React and TypeScript.** |
| **Constraint that forced it** | One page that both displays and writes (PRD C-01), forms for logging outcomes and adding enquiries, and a login gate. The framework pairs natively with the host in Section 5, so deploys are zero-config, and TypeScript stops the tenant-field mistakes that leak data between firms. |
| **Rejected** | **Plain HTML and a little JavaScript** — genuinely enough for one page, but you re-implement routing, session handling and form safety, and you pay it back on the first Phase 2 feature. **SvelteKit** — good tool, smaller hiring pool for a product someone else will maintain. **Remix / React Router framework** — no advantage over Next.js here. **Angular** — enterprise weight for one screen. **A batteries-included framework (Rails, Django, Laravel)** — collapses the server side usefully, but abandons the free serverless hosting that makes this cheap and the JavaScript skills the team is assumed to have (UNVERIFIED assumption; if the team is Python-first, this swaps to Django and the rest of the stack barely moves). |
| **Reversal cost** | Component and page rewrite if changed, but data, auth and hosting are untouched. Moderate, not severe. |

Server-rendering the one page also means the morning load does not wait on a client-side data fetch,
and the database is never queried from the browser.

---

## 5. Hosting

| | |
|---|---|
| **Choice** | **Vercel**, free Hobby plan for development, **Pro in production**. |
| **Constraint that forced it** | Next.js deploys with no configuration; 150 users is a rounding error against the free tier; cron and functions are included if a later feature needs them. |
| **Free tier (Hobby)** | 1M function invocations/month, 1M edge requests/month, 100 GB fast data transfer/month, 4 hours of active CPU/month, Cron Jobs included, runtime logs retained **1 hour**. Pro is **$20/month** per developer seat. Source: vercel.com/pricing, verified 2026-09-13. |
| **At 150 users** | ~3,000–15,000 invocations per month against a 1M allowance. Under 2% of the limit. Capacity will never force an upgrade. |
| **Rejected** | **Cloudflare Pages/Workers** — the strongest alternative: commercial use is allowed on the free tier and it is cheaper at scale. Rejected only because the Next.js server and session handling would need rework, and the saving is $20/month. Worth revisiting only if cost ever matters more than speed. **Netlify** — same shape as Vercel, no advantage. **Render / Railway / Fly.io** — containers and sleeping free tiers, more ops for nothing. **A cloud provider directly (AWS, GCP, Azure)** — enterprise infrastructure for 150 users; this is the instinct to resist. |
| **Forcing limit** | Not traffic. Vercel's own FAQ: *"Our Hobby plan is for personal, non-commercial use. Pro is designed for professional developers, freelancers, and businesses."* This product is sold to law firms for $1,200 a month, so it is commercial, so Hobby is out the moment it is used for real business. **$20/month, on day one of being commercial.** Source: vercel.com/pricing FAQ, verified 2026-09-13. |

---

## 6. Server functions and business logic

| | |
|---|---|
| **Choice** | **Next.js Route Handlers and Server Actions running on the host, talking to Postgres.** The derived numbers ("days since", chase order, money at risk) are computed **in the database** as a view or a single query, not in application code. |
| **Constraint that forced it** | The work is thin CRUD plus a handful of derived reads. There is no long-running job, no queue, no scheduled task in scope. Serverless functions are the cheapest thing that runs this, and there is nothing to keep warm. |
| **Rejected** | **A separate backend service** (standalone Node, NestJS, FastAPI) — a second deploy, second set of logs, second thing to break, zero benefit at this size. **Serverless functions from the database vendor instead of the host** — splits logic across two runtimes for no gain. **A message queue or background workers** — there is no asynchronous work in the PRD. **Stored totals** — totals that are written rather than derived drift the moment an outcome is logged; derive them every time. |
| **Reversal cost** | Moving to a separate backend later is a rewrite of the deploy and the request layer, but the database, schema and auth are unchanged. |

Why totals live in the database: the PRD says totals must "update the chase list and the money at
risk". If the sum is computed on read from the same rows the page shows, the two can never disagree.

---

## 7. What the PRD's features specifically demand

**Outcome history (PRD C-06, Phase 2).**
Store every logged outcome as a new row in an append-only table (`enquiry_id`, `firm_id`, outcome
type, `happened_at`, `logged_by`), even in Phase 1 where the screen only needs the latest one. The
constraint: history that was never captured cannot be backfilled. Doing this now costs one table;
retrofitting it means the first year of outcomes has no history. Rejected: a single mutable
"last contacted" field on the enquiry, which quietly forecloses Phase 2.

**Derived totals and chase order.**
One database view: open enquiries with `days_since_contact`, ordered by it, plus the sums. No extra
layer needed. Rejected: a cache, a report table, a search index — all premature at 150 users.

**Auth email (password resets).**
Use a custom SMTP provider on the free tier; Supabase includes custom SMTP on Free. Removing Supabase
branding from auth emails is Pro-only. Rejected: sending any product email — the PRD explicitly bars
sending messages to enquirers, so there is no outbound mail system, only password resets.

**No reminders, notifications, messaging, realtime, or files.**
The PRD puts automated reminders and enquirer messaging out of scope, so: no scheduler or cron, no
push, no SMS, no queue, and no realtime subscriptions. Skipping realtime also avoids Supabase's free
cap of 200 concurrent connections (supabase.com/pricing, verified 2026-09-13). No file uploads are in
scope, so no object storage, even though a small free allowance exists.

**Legal-grade data handling.**
The board holds enquirer names, contact details and matter hints. The constraint: managed
encryption at rest, least-privilege access, and **backups**. Supabase **Free includes no automatic
backups**; Pro includes daily backups kept 7 days. Source: supabase.com/pricing, verified
2026-09-13. This is the second reason the free tier ends on day one (Section 8).

**Observability.**
Vercel Hobby keeps runtime logs for **1 hour**; Pro keeps them for 1 day (vercel.com/pricing,
verified 2026-09-13). At 150 users, add a free-tier error tracker and rely on plan logs. Rejected:
log drains, metrics stacks and tracing — enterprise observability for a product with one page.

**Choices that survive the PRD's open contradictions (C-01 to C-08).**
The stack is deliberately shaped so the unresolved decisions do not force a rebuild:
- C-04 "days since what": store both `received_at` and `last_contact_at`; compute whichever is chosen.
- C-05 "fee may be unknown": `expected_fee` is nullable; totals treat null as zero.
- C-02 "what closes an enquiry": a status field; the set of closed values is configuration.
- C-06 "history in scope": outcomes are already stored as rows.
- C-08 "owner view": a filtered view over the same data, no schema change.
- C-01 "read versus edit": the same page can render and accept writes; no architectural fork.

---

## 8. Free tier: exactly what forces you off it, in order

The honest answer is that this product leaves the free tier almost immediately, and **not because of
traffic**.

1. **Day one of being commercial — Vercel.** Hobby is non-commercial by its own FAQ. Taking one
   paying client forces Pro: **$20/month**.
2. **Day one of holding a real client's data — Supabase.** Free has no automatic backups and pauses
   projects after a week of inactivity. Storing a law firm's enquiry data without backups is not
   acceptable, so this forces Pro: **$25/month**.
3. **Years later, if ever — storage.** 500 MB free. At 150 users the data grows by tens of MB a
   year, so this is not the forcing limit. Pro includes 8 GB, then $0.125/GB.
4. **Effectively never — compute and bandwidth.** Function invocations, egress and edge requests sit
   at a few percent of the free allowances. They will not force anything at 150 users.

So the floor for a real deployment is roughly **$45/month** (Vercel Pro $20 + Supabase Pro $25) plus
a domain. Against one client at $1,200/month that is under 4% of revenue for that client. The advice:
keep development on the free tiers, but do not contort the design to stay free, because the two
things that end free tier are terms and backups, and neither goes away by writing cleverer code.
For a pure prototype with no real client data, stay 100% free — the daily-morning habit prevents
Supabase's inactivity pause.

---

## 9. MVP versus production

| Layer | MVP (free, prototype) | Production | What kind of change |
|---|---|---|---|
| Frontend | Next.js + React + TypeScript | Same | **None** |
| Hosting | Vercel Hobby | Vercel Pro, $20/mo | **Config switch** (plan) |
| Database | Supabase Free Postgres | Supabase Pro, $25/mo | **Config switch** (plan) |
| Backups | None on Free (acceptable only for synthetic data) | Daily, 7-day retention | **Config switch** (plan) |
| Authentication | Supabase Auth | Supabase Auth (unchanged) | **None** |
| Tenancy | Shared DB, `firm_id`, RLS | Same | **None** |
| Server logic | Next.js route handlers / server actions | Same | **None** |
| Auth email | Custom SMTP or branded | Custom SMTP | **Config** |
| Observability | Host logs only | Host logs + free error tracker | **Config / add-on** |
| Realtime, queues, workers, file storage | Not present | Not present | **N/A** |

**The changes that are rewrites, not switches** — the ones worth caring about:

- **Leaving the auth vendor.** Every login, session and access rule is tied to it. Rewrite plus user
  migration.
- **Changing the tenancy model** (shared database → database per firm). Rewrite plus full data
  migration.
- **Moving off serverless to containers.** Rewrite of deployment and runtime.
- **Reworking the schema from mutable state to an outcome log.** Avoided entirely by storing outcomes
  as rows now.

Everything else in the table is a billing change or a setting. If a change appears in both lists,
it is a switch; the four above are the only rewrites.

---

## 10. The decision most expensive to reverse

**Coupling authentication to the database vendor — that is, letting the same vendor own both the
login identities and the row-level access rules that every table depends on.**

Why this one and not the others:

- It touches **every table, every query and every user**. Access rules reference the auth identity;
  identities reference the vendor.
- Leaving means re-issuing credentials to ~150 people, migrating identities, and rewriting every
  access rule at once. There is no incremental path.
- It is invisible until the day you want to leave. Unlike the schema, which you can version and
  migrate calmly, identity is entangled with real people's logins.

Cheaper-to-reverse decisions, for contrast: swapping hosting is a plan change; swapping frontend
framework leaves the data and auth untouched; moving managed Postgres vendors is a dump and restore.

**How to de-risk it now, cheaply:**
1. Keep a canonical user record in your own table (id, firm, role, email), keyed to the auth vendor's
   id — not only in the vendor's store.
2. Keep an application-layer tenant filter in addition to the database rules, so business logic is
   not expressed solely in vendor policy.
3. Put auth calls behind one small module so a future swap changes one file, not the whole codebase.

**Second place, and worth naming:** the shape of the enquiry record — whether outcomes are an
append-only log or a single overwritten "last contact" field. It is cheaper to fix than the auth
coupling, because the code change is additive; but the data you failed to capture is gone. That is why
Section 7 stores the log from day one.

---

## Road not taken, in one place

Enterprise infrastructure was deliberately not reached for: no Kubernetes, no container registry, no
message queue, no microservices, no data warehouse, no search cluster, no separate cache, no log
drain, no multi-region. None of it is justified by 150 users, one page, and one query pattern.

Rejected and why, condensed:

- **Database per firm / schema per firm** — 40× operational cost, no benefit at this scale.
- **No tenant isolation** — the one choice that guarantees a data migration later.
- **SQLite / Mongo / DynamoDB** — worse fit for relational sums or multi-tenant queries.
- **Self-hosted Postgres** — ops for nothing.
- **Clerk / Auth0 / custom auth** — a second vendor, enterprise pricing, or a security liability.
- **Plain HTML, SvelteKit, Angular, Rails/Django** — a rebuild of routing, auth or hosting for no gain.
- **Cloudflare, Netlify, Render, Railway, Fly, raw cloud** — equal or worse ergonomics; only Cloudflare is worth revisiting if $20/month ever matters.
- **Separate backend, queues, workers, stored totals** — no asynchronous work and totals must stay derived.
- **Realtime, notifications, SMS, object storage** — explicitly out of PRD scope.
