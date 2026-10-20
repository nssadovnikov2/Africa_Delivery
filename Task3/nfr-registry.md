# Non-Functional Requirements Registry

Date: 2026-08-24 · Inputs: [Task2/architecture-vision.md](../Task2/architecture-vision.md), [ADR-001](../Task2/adr-001-architecture-style.md), A1, A2, A5 (incl. CN checklist), A6 · Sources legend: [README](../README.md#sources)

## 1. Availability economics — how many nines we buy

The investor's "never down, $12k per minute" (A1, A7) is converted into money. GMV at risk per minute: Year-1 average $25/min (GMV $13.2M), 5-year average $425/min; we use **$650/min** (×1.5 peak weighting, TCO `Assumptions!C19`). Incremental costs are on top of the ADR-001 baseline.

| SLO (monthly) | Downtime / year | What it takes | Incremental cost / yr | Loss / yr at $650/min | Loss / yr at investor's $12k/min |
|---|---|---|---|---|---|
| 99.5% | 2,628 min (43.8 h) | 1 zone, 1 DB, business-hours support | baseline | $1.71M | $31.5M |
| 99.9% | 526 min (8.8 h) | 3 zones, Postgres HA, Redis HA, ≥ 2 replicas, rolling deploys | +$5.4k (HA DB/Redis ≈ $450/mo) | $342k | $6.3M |
| **99.95% (chosen)** | **263 min (4.4 h)** | + PSP failover, multi-SMS, canary + auto-rollback, cross-provider pilot-light DR, 24/7 follow-the-sun on-call | +$21k (DR $3k + on-call $18k) | $171k | $3.15M |
| 99.99% | 53 min | Active-active in 2 regions — only cross-provider (one hyperscaler region per provider in Africa), multi-writer DB, +3 SRE | +$300k (infra $38k, SRE $216k, egress $6k, one-off $45k) | $34k | $0.63M |
| 99.999% | 5 min | Not attainable: partner rails are below it (M-Pesa, SMS carriers) | n/a | — | — |

**Decision.** Each step up to 99.95% saves more than it costs (99.9 → 99.95: $171k saved for $21k). The 4th nine saves $137k/yr at $650/min but costs ≈ $300k/yr → **break-even $1.43k/min (≈ $500M GMV/yr, the Year-5 forecast)**. We commit **99.95% for checkout & payments and catalogue, 99.9% for notifications**, and reopen at **> $1.5k/min measured GMV-at-risk** (ADR-001 trigger). At the investor's $12k/min the 4th nine would pay back 8× — the figure equals $6.3B GMV/yr, 480× the Year-1 forecast. Note: the CTO's full package (option C) breaks even only at $4.6k/min (Task2 Comparison), because it also adds Oracle, Kafka and 14 FTE.

**Composite availability.** Checkout = platform 99.95% × PSP. With Flutterwave ↔ Paystack failover (each ≈ 99.9%) the PSP leg is ≈ 99.9999%; M-Pesa in Kenya has no second rail of the same kind, so Kenya routes M-Pesa via Flutterwave as secondary. Without failover end-to-end checkout would be ≈ 99.85%.

## 2. Registry

| ID | Category | Requirement | Metric | Target | Verification | Source |
|---|---|---|---|---|---|---|
| NFR-01 | Availability | Checkout & payment API available | Good requests / valid requests, 28 days | ≥ 99.95% | SLO dashboard, [slo/payments.md](slo/payments.md) | A1, A7, §1 |
| NFR-02 | Availability | Catalogue browse available (incl. stale cache) | Good responses at edge / all | ≥ 99.95% | [slo/catalog.md](slo/catalog.md) | A1, A2 |
| NFR-03 | Availability | Transactional notifications delivered | Delivered ≤ 5 min / sent | ≥ 99.9% | [slo/notifications.md](slo/notifications.md) | case |
| NFR-04 | Degradation | No hard failure when a dependency fails: catalogue served stale, notifications queued, PSP switched | % of dependency-outage drills passed | 100% of quarterly game-days | Chaos drill (FMEA actions) | A1 "never down", case |
| NFR-05 | Latency | Server-side API latency | p99 per endpoint | ≤ 300 ms catalogue, ≤ 800 ms checkout (excl. PSP) | k6 load test, Prometheus histograms | A2 |
| NFR-06 | Latency (client) | First product visible after app start on 3G | RUM p75 / p95 | ≤ 2.0 s / ≤ 4.0 s (from cache) | Firebase Performance on Android Go device | A2 "< 2 s" |
| NFR-07 | Throughput | Launch-day burst after radio spots | Sustained without SLO breach | 20k concurrent sessions, 10k rps at edge, 1k rps origin, 300 orders/min | k6 test from Lagos/Nairobi, weekly before launch | A2, Task1 AS-1 |
| NFR-08 | Scalability | Scale out by configuration only | Time to double API capacity | ≤ 5 min (HPA), no code change | HPA test | A5 |
| NFR-09 | Recoverability | Zone failure | RTO / RPO | ≤ 2 min / 0 | Zone-drain drill monthly | A5 "RPO zero" (zone scope) |
| NFR-10 | Recoverability | Region / provider loss | RTO / RPO | ≤ 60 min / ≤ 5 min | DR drill quarterly ([Task4/failover-plan.md](../Task4/failover-plan.md)) | A5, case |
| NFR-11 | Data integrity | No double charge, no lost payment | Reconciliation mismatches | 0 unresolved after T+1 | Daily PSP reconciliation job | A6 p.6 |
| NFR-12 | Mobile data | Data per session | Median MB per 10-min session | ≤ 1.5 MB (thumbnails WebP ≤ 25 KB, full image on tap) | RUM, CDN logs | A2 "megabytes are counted" |
| NFR-13 | Offline | App works without network for browse and order draft | Screens usable offline | Catalogue, cart, order status | Airplane-mode test | case (3G, drops) |
| NFR-14 | Residency | Clear-text PII of NG/KE subjects only in-country | Records outside country | 0 (DLP scan of core DB/logs) | Weekly automated scan | A3, A7, ADR-003 |
| NFR-15 | Privacy | Provable consent before marketing (ZA, NG, KE) | Marketing messages without consent record | 0 | Consent ledger audit | A3 p.2, case POPIA |
| NFR-16 | Compliance | Regulator takedown executed | Time from request to removal | ≤ 24 h, audit-logged | Back-office report | case KICA |
| NFR-17 | Security | Secrets & keys | Secrets in Git / images | 0; KMS-managed keys, rotation ≤ 90 days | CI secret scan | CN.SEC.03 |
| NFR-18 | Cloud native | Containerised, on Kubernetes, requests/limits set | % workloads compliant | 100% | Policy check (Kyverno) in CI | CN.RPL.01, CN.RPL.03 |
| NFR-19 | Cloud native | ≥ 2 replicas, liveness/readiness/startup probes, auto-restart | % deployments compliant | 100% (except jobs) | Kyverno policy | CN.SEL.01, CN.FTL.02 |
| NFR-20 | Cloud native | Config-as-code, GitOps, zero-downtime rolling/canary, automatic rollback | Manual changes in prod; failed deploy rollback time | 0; ≤ 5 min | ArgoCD drift report | CN.ADL.01/04/05/06 |
| NFR-21 | Cloud native | Internal image registry with proxy cache, namespace per app | Images pulled from public registries in prod | 0 | Admission policy | CN.SEC.02, CN.SEC.04 |
| NFR-22 | Cloud native | Managed services preferred where they meet NFRs | Self-operated stateful systems | 0 at MVP | Architecture review | CN.MSV.01 |
| NFR-23 | Observability | Metrics, logs, traces to central platform | % services exporting OTel | 100% | Grafana coverage dashboard | CN.OBS.01/02 |
| NFR-24 | Portability | Deployable on GKE, EKS, AKS or on-prem Kubernetes | Provider-specific code paths | 0 in app; differences only in Terraform modules | Quarterly restore into AWS DR | case "any provider" |
| NFR-25 | Cost | Cloud + SaaS in first 6 months | $/month | ≤ $5,000 (plan $4,100) | Monthly billing review with CFO | A6 p.1 |
| NFR-26 | Localisation | UI strings in EN, FR, Swahili, Hausa, Yoruba | Untranslated keys | 0 at release | i18n lint in CI | A2 promise 5 |

## Assumptions

- AS-1 Loss per minute counts GMV at risk, not margin, and assumes no recovery of abandoned orders — deliberately pessimistic in the investor's favour.
- AS-2 SRE rate $72k/yr; 3 additional SRE needed for multi-region 24/7 (TCO).
- AS-3 Partner availability ≈ 99.9% (no public SLA from M-Pesa/Flutterwave; to be confirmed in contracts).
- AS-4 "CNCF certification" (case, A5) is interpreted as compliance with the CN checklist; CNCF certifies Kubernetes distributions (GKE is certified) and people, not applications.
