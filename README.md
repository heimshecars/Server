# Heimshe Cars — Server

The backend for Heimshe Cars: **Supabase** (Postgres database, auth, realtime, edge functions), plus integrations with the phone hotline and the card machine.

> **Status:** not built yet. This README records how the business works, as understood so far, so the database is designed around the real rules. **Please correct anything that's wrong**, especially the open questions at the bottom.

## The business

- A UK **minicab** firm. **4 drivers** to start.
- Everything is **staff only**: the office uses the **Web** app and drivers use the **Phone** app. There is no customer app yet; one may come later as a Play Store app for drivers and customers.

### Where jobs come from
1. **Phone hotline.** A customer calls. The menu lets the caller **press for long distance**.
2. **Card machine.** When a driver takes a payment, it should show up as a job automatically.
3. **Drivers** log jobs themselves in the Phone app.
4. **The office** creates or edits jobs in the Web app.

### Short vs long distance
A job is **long distance** if either:
- the fare is **over £20**, or
- the caller chose **long distance** on the hotline.

Otherwise it is **short distance**.

### Commission
The company takes a **percentage of each fare**. **The rate differs for short and long-distance jobs.** For every job, the driver sees:
- the fare
- the commission rate (%) and the amount taken
- what they earned

### Scoring / leaderboard
Drivers are ranked against each other on jobs done, takings and performance. Drivers and the office see the same numbers.

### Driver devices
The company buys one **Android tablet per driver**. Each tablet is enrolled as a company-managed device by **scanning a QR code**, using Android Enterprise (built into Android) or Samsung Knox. The home screen is locked to **Google Maps, Waze and the Heimshe driver app**. The driver app is the Phone repo, delivered as a web app at our URL.

## How the frontends talk to this backend

Both frontends (**Web**, **Phone**) are **static sites**: every page is pre-rendered HTML with no server of their own. They talk to Supabase **directly from the browser**, which means:

- **Row-level security (RLS) on every table is mandatory.** It is the only thing stopping a driver from reading another driver's data.
- Business rules (long-distance classification, commission maths, scoring) live **here**, in one place: Postgres functions, triggers or edge functions. The frontends never recalculate them.
- New jobs reach drivers through **Supabase Realtime**.
- Integrations (hotline, card machine) post to **edge functions** via webhooks.

## Planned building blocks (first draft, not final)

| Area | What |
| --- | --- |
| People | `staff`, `drivers` (linked to Supabase auth users), `devices` (tablet ↔ driver) |
| Work | `jobs` (source, pickup, drop-off, fare in pence, distance type, status, driver), `shifts` |
| Money | `payments` (card machine transactions, matched to jobs), `commission_rates` (short/long, with effective dates so past jobs keep their rate) |
| Scoring | a view or function that ranks drivers over a period |
| Functions | `hotline-webhook`, `card-machine-webhook`, commission calculation, job assignment |

**Money is stored in pence** (integers), never as floating-point pounds.

## Open questions

1. **Commission rates:** what % for short and for long distance? The same for every driver?
2. **Exactly £20:** short or long? (Currently assumed: *over* £20 is long, £20.00 is short.)
3. **Conflicts:** if a caller presses "long distance" but the fare is under £20, which wins? (Currently assumed: long.)
4. **Card machine:** which provider (SumUp, Dojo, Zettle, Square, Stripe Terminal, …)? It must offer an API or webhooks so payments reach us.
5. **Phone hotline:** which provider runs the call menu (Twilio, …)? It needs to tell us when a caller presses long distance.
6. **Scoring:** which factors count, and how much (jobs, takings, acceptance rate, customer rating, …)?
7. **Shifts and payouts:** do drivers clock in and out? How often are drivers paid, and does cash collected by drivers need reconciling?
8. **Tablets:** which model, and which management route: Android Enterprise (Google's built-in, QR enrollment) or Samsung Knox?
