# Heimshe Cars — Server

Supabase backend for Heimshe Cars, a UK minicab firm. **Read `README.md` first**: it holds the business rules (short vs long distance, commission, job sources) and the open questions.

Frontends: `heimshecars/Web` (office, desktop) and `heimshecars/Phone` (drivers, Android tablets). Both are **static exports that call Supabase from the browser**.

## Rules
- **RLS on every table**, no exceptions. Drivers see only their own jobs and earnings, plus leaderboard aggregates. Staff see everything.
- **Business logic lives here, once.** That means long-distance classification, commission maths and scoring. Frontends display results; they never recalculate them.
- **Money in pence** (integer columns). Never floats.
- **Commission rates are versioned** (effective-from dates) so historic jobs keep the rate they were earned at.
- Integrations (hotline, card machine) come in through edge-function webhooks. Verify their signatures.
- Schema changes go through Supabase migrations in this repo.
