# Open Questions to Stakeholders

Date: 2026-08-13 · Only questions the architect cannot answer alone. Each has a default so work continues if no answer comes.

| # | Question | To | What it blocks | Answer by | Assumption if no answer |
|---|---|---|---|---|---|
| Q1 | The $12k/min figure equals $6.3B GMV per year. For launch we can fund 99.95% (≈ 4.4 h/yr) within $5k/mo; 99.99% active-active costs ≈ +$300k/yr (second provider region + 3 SRE). Which do you choose: 99.95% now with an automatic upgrade trigger, or extra budget? | Investor (K. Orlov), CFO in copy | SLO per service, topology (single region multi-AZ vs multi-region), ADR-001, ADR-002 | 2026-08-14 (before his Thursday flight) | 99.95% for checkout/payments, 99.9% for notifications; upgrade when GMV-at-risk > $1.5k/min (C-01) |
| Q2 | Two narrow answers, ahead of the full opinion: (a) may pseudonymised order data (token, SKU, amount, pickup-point ID; keys kept in Nigeria) be processed in South Africa? (b) must Kenyan payment-related data stay in Kenya when the payment itself is processed by a licensed PSP? | External counsel (A. Nwosu) | Placement of the shared platform, ADR-003, deployment diagram | 2026-08-28 (infra build starts 31 Aug) | (a) yes, but NG cell stays deployable fully in-country within 10 working days; (b) yes — Kenyan PII and payment references kept in a Nairobi vault |
| Q3 | Are per-message SMS/WhatsApp/USSD fees and PSP commissions inside the $5k/mo cap or cost of sales? Can we buy GCP (primary) and AWS (DR) through a local reseller on 45-day invoices? | CFO (A. Weber) | Provider choice and procurement timeline (3 weeks for card process, A6 p.4); TCO block split | 2026-08-17 | Cost of sales; GCP and AWS via reseller invoice; card process started in parallel |
| Q4 | Will you act as interim product owner and approve the MVP cut (9 of 29 features) and the "deliver differently" list for marketing promises? | CEO (M. Lefebvre) | Backlog for 6 contractors, Task5 roadmap | 2026-08-19 (Nairobi) | SA's cut from Task5 applies; CEO informed in writing |
| Q5 | Where do the 410k numbers come from (countries, consent provenance)? Can the launch blast become an opt-in USSD/SMS call to action? Is "1M buyers in month 1" a capacity target or only a PR claim? | Head of Marketing (S. Almeida) | Consent design, notification capacity, load test targets | 2026-08-20 | No blast; opt-in only; capacity planned for 100k MAU with 10× burst |
| Q6 | Who on the "Board" requires on-prem to stay on the table, and is it a 15 Oct requirement or a portability requirement? | CTO (R. Menon), confirm with Investor | Whether to run an on-prem cluster at launch (≈ $2.5k/mo, TCO-C) | 2026-08-19 | Portability requirement only: Kubernetes manifests + Terraform must run on-prem; no on-prem cluster at launch except the Nigerian PII vault |

## Assumptions

- AS-1 The investor answers in writing in one line; silence by the deadline is treated as acceptance of the default (communicated in the question).
- AS-2 Counsel can answer two closed questions in two weeks at limited hourly cost (≤ 5 hours).
