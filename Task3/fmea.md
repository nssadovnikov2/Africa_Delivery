# Failure Mode and Effects Analysis (FMEA)

## Overview
* **System/Process:** Africa Delivery MVP — checkout, catalogue, notifications, partner integrations
* **Date:** 2026-08-24
* **Prepared By:** Solution Architect with CTO
* **Version:** 1.0

---

## Scoring Scale (1-10)
* **Severity (S):** 1 (No effect) to 10 (Catastrophic impact)
* **Occurrence (O):** 1 (Very unlikely) to 10 (Almost certain/frequent)
* **Detection (D):** 1 (Certain to detect) to 10 (Impossible to detect)
* **Risk Priority Number (RPN) = Severity × Occurrence × Detection**

---

## FMEA Register (sorted by RPN; partner integrations marked ★)

| ID | Component / Process Step | Potential Failure Mode | Potential Effect of Failure | S | Potential Cause | O | Current Controls | D | RPN | Recommended Action (pattern) | Owner | Status |
|----|--------------------------|------------------------|-----------------------------|---|-----------------|---|------------------|---|-----|--------------------|-------|--------|
| 01 | ★ PSP callbacks (M-Pesa, Flutterwave) | Callback lost, delayed or delivered twice | Order stuck "pending" or paid twice; seller paid for unpaid order | 9 | PSP retries, network drops, our pod restart | 5 | None yet | 5 | **225** | Idempotency key per intent; inbox table dedup; status polling with **Retry + exponential backoff** (5 s → 5 min); daily reconciliation | Payments lead | Open |
| 02 | ★ SMS aggregator / carrier | OTP not delivered, sender ID blocked | Nobody can register or log in on launch day | 9 | Carrier spam filter after bulk traffic; provider outage | 4 | DLR webhooks | 4 | **144** | **Bulkhead** (OTP queue separate from bulk); 2 aggregators per country with **Circuit Breaker** routing; WhatsApp OTP fallback; no marketing blast (C-05) | Notifications lead | Open |
| 03 | ★ M-Pesa Daraja (Kenya) | STK push timeouts, API down | Kenyan buyers cannot pay | 8 | Safaricom maintenance or overload | 6 | HTTP client timeouts | 3 | **144** | **Circuit Breaker** per PSP; secondary route M-Pesa via Flutterwave; cash on delivery offered; status banner | Payments lead | Open |
| 04 | Launch traffic | Origin overloaded by radio-driven burst | Slow app, failed checkouts | 8 | 10× traffic in minutes (A2) | 6 | HPA | 3 | **144** | **Cache-Aside** + CDN (ADR-002); load shedding of non-critical jobs; rate limits; k6 test at 2× target | SRE | Open |
| 05 | ★ Sendbox / Sendy API | Shipment booking fails | Orders not dispatched; "same day" broken | 6 | Partner API outage or quota | 5 | None | 4 | **120** | Booking via outbox with **Retry + backoff**; **Circuit Breaker**; ops fallback: manual booking in partner portal from back-office queue | Fulfilment lead | Open |
| 06 | PII vault Lagos (local provider) | Vault unreachable | No NG registration, no NG SMS/courier booking | 8 | Lower SLA of local provider, power/network | 5 | Health check | 3 | **120** | 2 vault replicas on 2 hosts; requests queued (**Bulkhead**: vault calls in `worker` pool, not `api`); tokens cached so browse/checkout continue; detokenise asynchronously | SRE | Open |
| 07 | ★ WhatsApp BSP / Meta | Number quality rating drops, templates paused | Seller onboarding and order confirmations stop | 6 | Sellers report spam; template rejection | 4 | None | 5 | **120** | Quality-rating alert; second verified number; SMS/USSD fallback for sellers | Seller onboarding lead | Open |
| 08 | Release pipeline | Defective monolith release | All modules affected | 9 | Single deployable (ADR-001) | 4 | CI tests | 3 | **108** | Argo Rollouts canary with SLO analysis, automatic rollback; feature flags | SRE | Open |
| 09 | ★ Flutterwave (Nigeria) | Charges fail for minutes | NG card/transfer payments fail | 8 | Provider incident | 4 | Error metrics | 2 | 64 | **Circuit Breaker** → failover to Paystack; **Retry** only for idempotent status queries | Payments lead | Open |
| 10 | Redis (Memorystore) | Cache unavailable or flushed | Cache stampede on Postgres | 6 | Failover, eviction | 3 | Memorystore HA | 3 | 54 | **Cache-Aside** with fallback to DB; singleflight request coalescing; TTL jitter; stale-if-error at CDN | Catalog lead | Open |
| 11 | Cloud SQL primary | Zone loss / DB failover | 30–60 s of write errors | 9 | Zone incident | 2 | HA standby | 2 | 36 | **Retry + backoff** on transient errors; outbox keeps events | SRE | Open |
| 12 | Cloudflare edge | CDN/WAF outage | App cannot reach API | 8 | Provider incident | 2 | Status page | 2 | 32 | DNS fallback to origin load balancer (pre-tested); app keeps offline cache | SRE | Open |
| 13 | GCP region africa-south1 | Region unavailable | Platform down | 10 | Provider-wide incident | 1 | — | 2 | 20 | Pilot-light DR in AWS af-south-1 (Task4 failover plan) | CTO | Open |

---

## Action Tracking & Review
* **High Priority Threshold:** RPN > 100 requires immediate mitigation — IDs 01–08 must be closed before launch-readiness review on 2026-10-08.
* **Review Date:** 2026-10-08, then monthly and after every P1 incident.

## Assumptions

- AS-1 Occurrence scores are expert estimates for the first 3 months of operation; recalibrate from incident data at month 3.
- AS-2 Paystack and a second SMS aggregator are contracted before launch (ADR-004, Task5).
