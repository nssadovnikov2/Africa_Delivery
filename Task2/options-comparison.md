# Architecture Options Comparison

Date: 2026-08-18 · Inputs: [Task1/contradictions.md](../Task1/contradictions.md) (C-01, C-02, C-08), A5 (CTO), A6 (CFO), [tco-model.xlsx](tco-model.xlsx). All options use the same partner integrations (M-Pesa, Flutterwave, Paystack, Sendbox, Sendy, WhatsApp BSP, SMS/USSD aggregator) and the same per-country PII vaults (Task4) — they differ in how the core is built and run.

## Options

| | **A. Modular monolith** | **B. Service-based** | **C. Microservices (CTO, A5)** |
|---|---|---|---|
| Shape | One Go codebase, 10 modules with hard boundaries (schema per module, internal APIs, outbox events); 3 deployments of one image: `api`, `worker`, `channels` | 6 coarse services (Catalog, Checkout+Orders, Payments, Fulfilment, Channels, Identity) + managed broker | 40–50 Go services, Istio mTLS, Kafka backbone, ArgoCD, 3 clusters (2 clouds + on-prem), Oracle core |
| Data | Managed PostgreSQL HA + Redis | 2 managed PostgreSQL clusters + Redis + RabbitMQ | Self-run Oracle (Data Guard) + Postgres operators + Kafka ×3 |
| Build vs buy | Partner logistics & payments behind adapter interfaces | Same | Build logistics engine and payment orchestration |
| Topology | 1 region (GCP africa-south1), 3 zones + pilot-light DR in AWS af-south-1 | Same | Active-active GCP + AWS + on-prem Lagos |

## Comparison on equal criteria

| Criterion | A. Modular monolith | B. Service-based | C. Microservices (CTO) |
|---|---|---|---|
| **Time to market** — effort vs capacity (6 contractors × 8 wks = 48 engineer-weeks) | 44 ew → **15 Oct** | 56 ew → **≈ 20 Nov** with 6 people | ≈ 234 ew (incl. logistics 104 ew by CTO's own estimate) → **≈ Feb 2027** |
| **Risk of missing 15 Oct** | ≈ 20% (scope cut is the buffer) | ≈ 70% | > 95% |
| **Cloud + SaaS, $/month, Year 1** (CFO cap $5,000) | **$4,100** ✅ | $6,150 ❌ (+23%) | $19,683 ❌ (×3.9) |
| **One-off licences** | $0 | $0 | $190,000 Oracle EE + $41.8k/yr support |
| **Team in Year 1** (freeze: 6) | **6** | 8 | 14 (10 dev + 3 SRE + DBA) |
| **5-yr TCO, direct** (all 5 blocks + 15% contingency) | **$44.1M** | $46.4M | $49.0M |
| **5-yr architecture-dependent cost** (blocks 1, 2, 4 + contingency) | **$7.1M** | $9.3M | $16.5M |
| **NPV of costs** (20%) | **$26.6M** | $28.1M | $29.8M |
| **Cost per order** — launch month (20k orders) / month 12 (100k) | **$3.32 / $1.25** | $4.00 / $1.42 | $6.36 / $2.02 |
| **Scalability without re-architecture** | ≈ 2–3M MAU (stateless API ×20 replicas; Postgres vertical + read replicas, ≈ 2k orders/min) | ≈ 5–10M MAU | 10M+ MAU (designed for it) |
| **Operational complexity** — deployables / stateful systems we operate ourselves | 3 / 0 (managed) | 6 / 1 | 40–50 / 4 (Oracle, Kafka, Postgres operators, Istio) |
| **Target SLO / downtime budget** | 99.95% / 263 min/yr | 99.95% / 263 min/yr | 99.99% / 53 min/yr |
| **Expected downtime loss / yr** at $650/min (GMV at risk, TCO Assumptions) | $171k | $171k | $34k |

Effort breakdown for A (engineer-weeks): mobile app 8, core modules 12, integrations 10, back office + CEO dashboard 6, platform (GKE, IaC, CI/CD, observability, vaults) 6, load test & hardening 2.

## The CTO's position, on its merits

- **Accepted:** Go for all backend code; Kubernetes; GitOps with ArgoCD; OpenTelemetry/Prometheus/Grafana; event-driven integration *inside* the monolith (transactional outbox) so modules can be extracted later; the CN checklist as the compliance yardstick.
- **Rejected with numbers:** (1) 10M users on day 1 — the wasted cost if 50k users come is **$858k in the 9-month runway** (C − A, blocks 1, 2, 4, 5 × 0.75 yr), plus **$180k** of paid media if the launch slips to February — that equals ≈ 20 years of option A's Year-1 cloud + SaaS bill ($49k/yr). (2) Oracle — $190k up-front breaks A6 p.7 and the runway. (3) Own logistics & payment orchestration — 24 engineer-months; even crediting his claim of −0.2 pp on payment fees and −30% on logistics fees, C remains **$4.9M more expensive over 5 years**. (4) Active-active 99.99% — pays back only if a minute of downtime costs **> $4,629**; our forecast GMV-at-risk is $650/min (Y1 average $25/min).
- **Investor's $12k/min, taken literally:** TCO incl. downtime becomes A $59.9M vs C $52.2M, i.e. C wins. But $12k/min = $6.3B GMV per year — 480× the Year-1 forecast and 12× Year 5. We therefore treat it as a **trigger**, not a launch requirement: ADR-001 is reopened for multi-region when measured GMV-at-risk exceeds ≈ $1.5k/min — the break-even of adding only the 4th nine to option A (Task3 NFR §1), which is cheaper than the whole option C.

## Cost of being wrong (CFO question, A6)

| Scenario | Cost |
|---|---|
| Build for 10M (C), only 50k users come | **≈ $1.04M** wasted in the runway: $858k excess run cost + $180k media wasted by a late launch |
| Build small (A), it takes off to 10M | **≈ $0.25M** in the year of growth: extract 2–3 hot modules (3 eng × 4 months ≈ $60k), +2 SRE ($144k/yr), active-active infra (+$4k/mo ≈ $48k) — paid from the growth revenue |

## Conclusion

Option A is the only one that meets all three hard constraints at once — 15 October, $5k/month, six people — and it is also the cheapest over five years ($44.1M direct, $7.1M architecture-dependent vs $16.5M for C). Option B is a legitimate step, but it needs 2 more engineers and 23% more budget for scalability we will not need before ≈ 3M MAU. Option C is the right target only if the investor's $12k/min becomes real (GMV > $2.4B/yr); A keeps that door open through module boundaries and outbox events, so the move is extraction, not a rewrite. **Recommendation: A, recorded in [ADR-001](adr-001-architecture-style.md).**

## Assumptions

- AS-1 Contractor rate $60k/yr dev, $72k/yr SRE; Oracle DBA $90k; recruitment agency fee 18% (TCO sheets).
- AS-2 Drivers: users 0.5M → 6M, orders 0.6M → 20M, AOV $22 → $26 over 5 years (TCO Assumptions).
- AS-3 Provider list prices Aug 2026 without commitments (A6 p.3); Task4 compares regions.
- AS-4 Effort estimates are architect's top-down estimates ±25%; C uses the CTO's own 6 months × 4 engineers for logistics (A5).
- AS-5 Fees and SMS/USSD costs are equal for A and B; C is credited with the CTO's claimed savings.
