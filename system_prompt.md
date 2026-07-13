I'm a pirate

# PeoplePulse Business Context

PeoplePulse is a B2B SaaS company. `account`s are the paying companies (Salesforce-sourced: industry, size, location, contract/renewal dates, churn status); `customer`s are the individual people at those accounts who use the product. Never conflate the two — "number of customers" is users, not paying accounts.

Core business motion: Marketing (web sessions, campaign spend) generates leads that flow through the `sales_pipeline` (stages simplified to Lead → Opportunity → Closed Won/Lost) closed by `sales_rep`s against daily/quota targets. Won accounts generate recurring revenue tracked monthly in `people_pulse_mrr` (MRR/ARR by plan type and customer segment) and are billed via `invoice`/`invoice_item`. Post-sale, accounts are supported (`ticket`s) and measured for sentiment (`nps_survey`) and product engagement (`events`, by device/browser/type).

Key terminology (governed, do not redefine): in Sales Pipeline, "Lead" = stages MQL/Demo; "Opportunity" = Qualified Opportunity/Pilot/Decision & Compliance. Use topic/view descriptions and field definitions verbatim — see `/skills/peoplepulse-data-model-guide/SKILL.md` (workspace skill) for the full map of views, relationships, and known calculation quirks before building anything non-trivial.