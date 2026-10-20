# Architecture Canvas — Africa Delivery

Date: 2026-08-13 · One page · Sources: [README](../README.md#sources)

| Section | Content |
|---|---|
| **Business goals** | **G1 (15 Oct 2026, MVP):** pilot in Nigeria (Lagos, Abuja) and Kenya (Nairobi, Mombasa): catalogue 5,000 SKU / 100 sellers, checkout with M-Pesa + Flutterwave, SMS/WhatsApp notifications, simple buyer account, CEO dashboard (case). **G2 (12 months):** 5 countries incl. ZA, > 1M users, B2B, seller cabinet with analytics, track & trace, returns, PUDO anti-fraud, EN/FR + local languages (case). **G3 (36 months):** 12–15 countries, 40% share (A1, A7). Investor's success test for 16 Oct: "people buy, sellers get paid, nothing breaks, no minister calls" (A1). |
| **Key drivers** | D1 Fixed date — paid media $180k, store pages 1 Oct (A2). D2 Cash — $5k/mo infra, 9 months runway (A6). D3 Data sovereignty — NG data in NG, consent in ZA (A3, A7). D4 Mobile reality — < $150 Android, 3G, paid megabytes, USSD/SMS for 30–40% of the market (A2, case). D5 Trust — counterfeits and failed deliveries are the competitor's weakness (A2). |
| **Constraints (non-negotiable)** | Launch 15 Oct (A1). ≤ $5k/mo for compute, storage, DB, CDN, monitoring, SaaS; > $500/mo items need CFO sign-off; no prepay/RI; invoice billing, no card (A6 p.1–4). 6 contractors, hiring freeze to Dec (A6 p.5). No per-core licences (A6 p.7). Payments only via licensed PSPs (A3 p.4). No hyperscaler region in Nigeria (case/Task4). Logistics only via partner APIs (case). |
| **Quality attributes (to be quantified in Task3)** | Availability of checkout and payments; P99 latency on 3G; offline tolerance; throughput for launch-day bursts; RPO/RTO; portability between providers (case: "any compute provider"); cloud-native operability (CN checklist, A5). |
| **Principles** | P1 Buy/partner commodity, build differentiation (checkout, catalogue, seller onboarding over WhatsApp/USSD). P2 Late binding — defer what lawyers/traffic will tell us; keep it a configuration. P3 Portability through open interfaces (Kubernetes, Postgres protocol, OpenTelemetry), not through self-hosting. P4 Degrade, don't fail. P5 Personal data never leaves its country in clear form. |
| **Out of scope for MVP** | Crypto loyalty (A1, A4 #14 — "study, not for October"); BNPL with own scoring, wallet (licences); own logistics engine and warehousing (A4 notes); dynamic pricing via Jumia scraping (legal risk); native live streaming; voice search. |
| <a id="risks"></a>**Top risks** | R1 Partner onboarding (KYB at Flutterwave/Safaricom, WhatsApp BSP verification) takes longer than 6 weeks. R2 Legal opinion (5–19 Sep) changes data placement late. R3 Launch-day burst from radio/billboards. R4 Investor overrides decisions outside his competence (case). R5 Key-person risk: CTO and SA are the only permanent technical staff; no local staff (case). R6 Sendy/Sendbox coverage does not deliver "same day" promise (A2 promise 2). |
| **Assumptions** | See below. |
| **Decision rights** | See [stakeholder-matrix.md](stakeholder-matrix.md#who-actually-decides). Contradictions and owners: [contradictions.md](contradictions.md). |

## Assumptions

- AS-1 MVP launch traffic: ≤ 100k monthly active users, ≤ 20k orders/month, 10× burst after radio spots (vs "1M in month 1", A2) — see C-06.
- AS-2 Kenya and Nigeria first; South Africa no earlier than month 6, so POPIA consent tooling must exist before ZA launch, not on 15 Oct (A1).
- AS-3 SMS, WhatsApp, USSD per-message fees and PSP fees are cost of sales, outside the $5k infra cap (open question Q3).
- AS-4 Contractors are remote in 3 time zones (UTC+1 … UTC+5:30); no hire in Nairobi before December.
- AS-5 Pseudonymised order data (no name, phone, address) may be processed outside Nigeria (A3 p.1 "masking may be acceptable") until counsel says otherwise.
