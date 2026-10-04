# CLAUDE.md — FrictionIQ (JPM hackathon)

Context for Claude when working in this repo. Read this first.

## The project

**FrictionIQ** measures what fraud rules cost *good* clients. Today rules are tuned only on what they catch
(hit rate, precision, recall). FrictionIQ adds the other axis:

1. **Friction score per client** — every intervention a client received (deny, hold, settlement limit,
   restriction, block), weighted by severity and recency, each attributed to the rule that caused it.
   Shown on screen as an **A–F friction grade** with a range.
2. **Tradeoff curve per rule** — sweep the rule's threshold, re-run the whole population, plot
   **friction removed** vs **fraud still caught**. The flat stretch (friction falls, fraud caught doesn't) is the finding.

It recommends; it never changes a live rule. Production path: run a copy of the rule at the relaxed threshold
in shadow mode (logs what it would do, touches no client).

- Submission: **Oct 6, 2026**. Category: Decision Intelligence.
- Team of two: **Anisha** owns the screens, friction index, good-client bands and threshold sweep.
  **Partner** owns the synthetic data generator (clients, decision events, rule hits, fraud labels).
- Blind generation: the generator and the analysis share **only the schema below**. Don't tune the generator
  to make the analysis look good, and don't renegotiate the schema after seeing the other side's output.

## Hard rules

- **Everything is synthetic.** Fictional client names and IDs that can't be mistaken for real ones. No client
  data, PII, transaction records or live rule content, ever.
- **This repo is public.** Never commit the PRD or any internal system, team or tool names. Use generic terms
  ("shadow mode", "manual review").
- **Seed the generator and record the seed** (screens currently say seed `4127`). The demo must reproduce exactly.
- Keep numbers consistent with [`data/frictioniq-mock.json`](data/frictioniq-mock.json) — every figure on the
  Figma screens comes from there. If generator output changes a number, update the JSON and tell Anisha so the
  screens can be updated.

## Repo map

| Path | What |
|---|---|
| `README.md` | Human-facing overview and demo story |
| `design/README.md` | Screen-by-screen design notes and decisions |
| `design/screens/*.png` | Exports of the 3 Figma screens |
| `data/frictioniq-mock.json` | All placeholder values shown on screen (target shape for generator output) |

Figma (invite only): https://www.figma.com/design/iXDUVq6xs7RFJiSkXe7Ovk — page "⭐ FrictionIQ — Screens".
Screens: **1 Home** (portfolio) → **2 Client detail** (Acme Supplies) → **3 Rule tradeoff** (`payout_limit_100`).

## Schema (the contract)

| Object | Attributes |
|---|---|
| Rule | rule_id, rule_name, entity, request_type, category, sub_category, decision, rule_expression, shadow_setting |
| Ruleset binding | ruleset_id, rule_id, checkpoint, order |
| Rule hit | hit_id, rule_id, client_id, triggered_at, resolved_at, final_decision, action_taken |
| Client | client_id, entity, segment, tenure_months, peer_group |
| Incident | incident_id, start, end, affected_clients, severity |

- `rule_expression` carries the threshold as a literal (e.g. `payout_amount > 100`) so a sweep is a number
  change. Sweepable rules need a **single numeric comparison**; list-membership rules have two states, not a curve.
- `decision` gives the event type (a DENY rule firing = a denial event). Ledger = hits joined to rules.
- `order` decides attribution: when several rules hit one event, only the rule whose decision prevailed
  scores; the others are recorded as contributing hits.
- `shadow_setting = On` hits are logged but score **zero** (no client paid for them).
- Incidents are scored in their own band and **never** added to rule friction.

## Scoring (for reference)

```
friction(client) = Σ over prevailing, non-shadow hits of
                   decision_weight × duration_factor × recency_decay
duration_factor  = log(1 + hours_held / reference_hours)   # holds only; 1 for everything else
recency_decay    = 0.5 ^ (days_since / half_life)          # half_life 30, swept 14–60
```

Placeholder weights: shadow 0 · settlement limit 10 · product restriction 15 · hold/manual review 25 ·
deny 40 · block account 70 · termination 100. Weights are swept across orderings that respect this rank, so
results are always reported as a range.

## Generator volumes

| Dataset | Volume |
|---|---|
| Client profiles | 200–400 (screens assume 312) |
| Rules | 8–12 (screens show 8) |
| Decision events | 20,000–50,000 |
| Friction events (rule hits that acted) | 4,000–8,000 |
| Incidents | 2–4 |
| Confirmed fraud cases | 150–400 |

The world must be coherent: client profile → behaviour → rule hits → friction; fraud labels from controlled
scenarios. **Rules must differ in quality on purpose**: at least one with a long flat stretch (over-fires on good
clients) and at least one with none (tightly targeted). Include incident-driven clients, good-client/high-friction
and bad-client/low-friction cases, and a little controlled missingness.

## Seeded scenarios (build these first)

| # | Client | Pattern | Expected | Proves |
|---|---|---|---|---|
| 1 | **Acme Supplies**, Direct SMB, 38 mo, established, no fraud | 9 prevailing hits in 30 days, 7 from `payout_limit_100` (deny/hold payouts > $100; Acme's typical payout ≈ $150; fraud payouts mostly $2,000+) | High friction, mostly one rule | Hero case: raise to $500, friction gone, fraud caught unchanged |
| 2 | Direct SMB, 26 mo, established (e.g. Delta Bakehouse) | 2 hits + 40 h inside an incident | 85% of friction in the incident band | Separate incident band matters |
| 3 | Payfac, 7 mo, limited history, 2 confirmed fraud (Northwind Payfac) | 11 hits, 4 rules, 3 checkpoints | High friction, earned | Tool declines to relax |
| 4 | Enterprise, 52 mo, established | 3 rules hit one event: order 10 hold, 20 deny, 30 settlement limit | Only the deny scores | Attribution by order |
| 5a / 5b | Direct SMB, 30 mo | 5a: 1 hold, 96 h · 5b: 6 holds, ~25 min each | 5a > 5b | Duration factor works |
| 6 | Direct SMB, 19 mo, developing | 40 shadow hits + 1 live hit | Only the live hit scores | Shadow = free control group |
| 7 | Enterprise, 44 mo, established | 6 hits from a tightly targeted boarding rule (`boarding_doc_mismatch`) | Relaxing loses fraud immediately | The honest negative finding |

Acme's intervention dates and the other on-screen figures are in `data/frictioniq-mock.json` → `client_acme`.

## Outputs the screens need

Precompute everything; the slider reads a file, never recomputes live.
- Per rule × threshold: interventions removed, clients affected, fraud cases caught, friction-removed % with
  min/max across weightings.
- Per client: score + range, grade, interventions list (rule, action, checkpoint, hold hours, outcome),
  good-client evidence (tenure, reviews cleared, confirmed fraud, payout stability, disputes).
- Portfolio: totals, grade counts, clients interrupted, good clients with heavy friction, incident counts.
