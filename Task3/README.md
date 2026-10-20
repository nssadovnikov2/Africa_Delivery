# Task 3 — Non-Functional Requirements and Availability Strategy

| File | Content |
|---|---|
| [nfr-registry.md](nfr-registry.md) | Cost of each nine (99.5% → 99.999%), chosen SLO and break-even; 26 NFRs incl. CN checklist mapping |
| [slo/catalog.md](slo/catalog.md) | SLI / SLO / SLA contract — catalogue (OpenSLO) |
| [slo/payments.md](slo/payments.md) | SLI / SLO / SLA contract — payments, refunds, payouts (OpenSLO) |
| [slo/notifications.md](slo/notifications.md) | SLI / SLO / SLA contract — OTP and transactional notifications (OpenSLO) |
| [quality-attribute-scenarios.md](quality-attribute-scenarios.md) | 7 scenarios: availability ×3, P99 latency, throughput, RTO/RPO, deployability |
| [fmea.md](fmea.md) | 13 failure modes incl. 7 partner integrations, RPN, protection patterns |
| [adr-002-caching-strategy.md](adr-002-caching-strategy.md) | ADR-002: what is cached where, TTLs, invalidation |
| [observability.md](observability.md) | Golden signals, SLI mapping, logs, traces, tools |
| [observability.puml](observability.puml) / [.png](observability.png) | Telemetry collection scheme |

**Bottom line:** 99.95% for checkout, payments and catalogue, 99.9% for notifications. Each nine up to 99.95% pays for itself; the 4th nine costs ≈ $300k/yr and pays back only above ≈ $1.43k/min of GMV at risk (forecast average $650/min) — that is the written trigger to move to active-active.
