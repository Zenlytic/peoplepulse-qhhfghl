---
name: peoplepulse-data-model-guide
description: Use when answering any PeoplePulse data question, building a dashboard/report, or auditing/editing the data model. Provides the map of views, how they relate, which topic to search for which subject area, and known calculation quirks to watch for (deal_size scaling, mrr/12 pattern, nps survey_date always = today, etc.) so answers stay consistent with governed definitions.
---

# PeoplePulse Data Model Guide

PeoplePulse is a B2B SaaS business. This skill is the map of the data model: what each view represents,
how views relate, and non-obvious calculation quirks baked into existing fields. Read this before
building anything beyond a single simple `data_question` pull, and before proposing data model changes.

## The business, end to end

Marketing (`marketing_spend`, `web_session`) drives awareness and top-of-funnel leads → Sales
(`sales_pipeline`, `sales_rep`, `people_pulse_plan`) works opportunities through stages to Closed
Won/Lost against rep quotas → Won deals become paying `account`s billed via `invoice`/`invoice_item`
and tracked as recurring revenue in `people_pulse_mrr` (MRR/ARR by plan type & customer segment) →
Post-sale, `customer`s (the humans at each account) use the product (`events`), get supported
(`ticket`), and give feedback (`nps_survey`). `employee` is internal headcount/payroll data, unrelated
to the customer lifecycle.

**Accounts vs. customers — never conflate:** `account` = the paying company (Salesforce fields:
industry, size, location, contract/renewal dates, churn). `customer` = an individual person/user at
that account. "How many customers do we have" means user count, not paying-account count; "how many
accounts churned" is an account-level question.

## View map and what each represents

| View | Grain | Represents | Key joins |
|---|---|---|---|
| `account` | 1 row/company | Paying company (Salesforce) | → `customer` (1:many), → `invoice`, `people_pulse_mrr` (1:many) |
| `customer` | 1 row/person | Product user at an account | → `account` (many:1), → `events`, `nps_survey`, `ticket` |
| `people_pulse_mrr` | 1 row/account/month/plan | Monthly recurring revenue snapshot | → `account` (many:1) |
| `invoice` | 1 row/invoice | Billing event | → `account` (many:1), → `invoice_item` (1:many) |
| `invoice_item` | 1 row/line item | Product line on an invoice | → `invoice` (many:1), → `product` (many:1) |
| `sales_pipeline` | 1 row/opportunity | Deal in the sales funnel | → `sales_rep` (many:1), → `product` (many:1) |
| `sales_rep` | 1 row/rep | Sales rep roster & quota | → `sales_pipeline`, `people_pulse_plan` (1:many) |
| `people_pulse_plan` | 1 row/rep/day | Daily quota plan | → `sales_rep` (many:1); merged cross-view with `sales_pipeline.closed_won_usd` for `quota_attainment` |
| `product` | 1 row/product | Product catalog | referenced by `sales_pipeline`, `invoice_item` |
| `events` | 1 row/event | Logged-in product usage event | → `customer` (many:1) → `account` |
| `nps_survey` | 1 row/survey | Customer sentiment score | → `customer` (many:1); allowed fanout join to `events` |
| `ticket` | 1 row/ticket | Support ticket | → `customer` (many:1) → `account` |
| `web_session` | 1 row/session | **Marketing website** session (anonymous, NOT logged-in product users) | standalone |
| `marketing_spend` | 1 row/campaign/day | Ad/campaign spend, reach, engagement | standalone, no join keys |
| `employee` | 1 row/employee | Internal headcount | standalone; `salary` is access-gated to `exec_only` |

Topics (legacy grouping) map roughly 1:1 to these subject areas and encode the valid join paths — use
`search_fields` with the matching topic when unsure which view has a field.

## Core governed metrics (use as-is, do not redefine)

- **Revenue:** `people_pulse_mrr.total_mrr`, `total_arr` — both use `non_additive_dimension` on the max
  date, so summing across many months double counts; let the measure handle it, don't hand-roll SUM(mrr).
- **Pipeline:** `sales_pipeline.total_deal_size`, `weighted_deal_value` (deal_size × probability/100),
  `closed_won_usd`, `count_won_opportunities`, `count_lost_opportunities`.
- **Quota attainment:** `people_pulse_plan.quota_attainment` = closed-won $ ÷ sum of daily quotas; it's
  a merged-result field spanning `sales_pipeline` and `people_pulse_plan` — don't rebuild manually.
- **Churn:** `account.number_of_churned_accounts` / `account.number_of_accounts`, filtered by
  `account.churned = 'Yes'`.
- **Support health:** `ticket.average_satisfaction_score`, `average_resolution_time`.
- **Sentiment:** `nps_survey.average_nps_survey_score`, bucketed via `sentiment` (good ≥8, ok 6-8, bad
  4-6, ugly <4).
- **Engagement:** `events.total_events`, `unique_customers`; `web_session` metrics are marketing-site
  only, not product usage.

## Known calculation quirks (do not "fix" without confirming with the user)

- `people_pulse_mrr.mrr` is defined as `MRR / 12` and `total_arr` multiplies back by 12 — so the raw
  `MRR` column in the warehouse is already annualized; the view field named `mrr` is actually a derived
  monthly figure. Trust `total_mrr`/`total_arr` measures rather than the raw column.
- `sales_pipeline.deal_size` is `DEAL_SIZE * 0.15` — a scaling factor is already baked in. Use the
  `deal_size` field (and measures built on it), never the raw `DEAL_SIZE` column.
- `sales_pipeline.stage` remaps raw warehouse stage values into funnel-friendly labels (e.g. raw
  'Proposal'/'Lead Generation' → 'MQL', raw 'Qualification' → 'Demo', raw 'Initial Contact' →
  'Qualified Opportunity', raw 'Negotiation' → 'Pilot', raw 'Meeting' → 'Decision & Compliance'). Always
  filter/group on the governed `stage` field, not the raw column.
- `nps_survey.survey_date` is hardcoded to `CURRENT_DATE` in SQL, not an actual historical survey date
  column — trend-by-date charts on this field will show every survey as "today." Flag this to the user
  if they ask for an NPS trend over time; there may not be a true historical date available.
- `employee.salary` requires the `exec_only` access grant (`user_attribute: "My Thing"`, allowed value
  `"Hey"`) — most users will not see this field.
- No `model-level relationships` are currently defined in `models/base_model.yml`; joins are carried by
  legacy `topics/` and view `identifiers`. When adding new cross-view logic, still search within a
  topic scope rather than assuming a join exists.

## When building dashboards/reports

Match subject area to topic: revenue → "Revenue and Customer Account Data"; churn/retention →
"Account Information Summary" or "Customer and Account Data"; funnel/quota → "Sales Pipeline Analysis" /
"Sales Performance Dashboard"; support → "Customer Support Ticket Analysis"; sentiment/usage →
"Customer Feedback and Interaction Data" / "Product Usage Data Model"; marketing → "Marketing Spend
Analysis" / "Web Session Analytics". Several PeoplePulse dashboards already exist covering these areas
(Business Overview, Executive, Product Engagement & Sentiment, Support Ticket Performance, Sales
Pipeline & Performance) — check existing artifacts before building a new one on the same subject.
