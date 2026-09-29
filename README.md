# Funnel review

One page with every outbound funnel we run, in order, each with a verdict (working,
watch, not working), the reason in one line, and the one thing to do about it. Built
for the Monday GTM meeting, but it's live: open it whenever.

Live at https://rishabhpandey-hash.github.io/funnel-review/ (Rishabh and Kinner only).

## How it works

- This repo is only the page (`index.html`). It holds no data and no secrets; the key
  in it is the Supabase anon key, which is public by design.
- After email sign-in, the page calls the `funnel-review` edge function in the ops
  Supabase project. That function checks the email against `fr_users` and returns
  `fr_review()`.
- The numbers are built inside the database, every hour at :41 and on "Refresh now":
  `fr_queue_run()` queues the calls, `fr_worker()` (pg_cron, every minute) sends them
  a few at a time, and `fr_build()` writes the verdicts.
- Source of truth for the backend is the control-tower repo:
  `backend/ops/supabase/functions/funnel-review/` and
  `backend/ops/supabase/migrations/20260929_*_funnel_review_*.sql`.

## Changing things

- **Add or remove a campaign, rename a funnel, change the order:** edit the
  `fr_funnels` row (email campaigns by SmartLead id, LinkedIn campaigns by the name
  the LinkedIn inbox uses). A LinkedIn campaign must be mapped in the LinkedIn inbox
  admin panel first, or it shows as "not connected".
- **Change a benchmark:** edit `fr_benchmarks`.
- **Give someone access:** `insert into fr_users (email, role) values ('name@company.com', 'member');`
- **Preview locally without signing in:** save a `fr_review()` result as
  `payload.json`, run `python3 -m http.server 8899` here, and open
  `http://localhost:8899/?preview=payload.json`.
