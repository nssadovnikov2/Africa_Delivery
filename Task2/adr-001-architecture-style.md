# ADR-001: Architecture Style — Modular Monolith with Partner Integrations

| Field | Value |
|---|---|
| Status | Proposed (for Nairobi meeting 2026-08-19) |
| Date | 2026-08-18 |
| Deciders | CTO R. Menon (accountable), Solution Architect (responsible); CFO A. Weber (budget sign-off) |
| Consulted | Investor K. Orlov, CEO M. Lefebvre |
| Traces to | C-01, C-02, C-06, C-08 ([contradictions](../Task1/contradictions.md)); A1, A5, A6 p.1/p.5/p.7, A7 |

## Context

The CTO proposes 40–50 microservices with Istio, Kafka, Oracle and three active-active clusters, built for 10M users from day 1, with in-house logistics and payment orchestration (A5). The CFO caps infrastructure at $5k/month, forbids per-core licences and freezes hiring at six contractors (A6 p.1, p.5, p.7). The investor fixes 15 October and wants it "simple, like Wildberries… very cheap" (A7) while "never down" (A1). The TCO model ([tco-model.xlsx](tco-model.xlsx)) and the [options comparison](options-comparison.md) quantify the trade-off.

## Decision drivers

1. Go-live on 2026-10-15 with ≤ 48 engineer-weeks of capacity.
2. Cloud + SaaS ≤ $5,000/month in the first 6 months.
3. Checkout availability 99.95% with graceful degradation (Task3).
4. Growth path to 5 countries and > 1M users in 12 months without a rewrite.
5. Portability across AWS/GCP/Azure/on-prem (case) and compliance with the CN checklist (A5 attachment).

## Considered options

| Option | Y1 cloud+SaaS/mo | Team Y1 | Go-live | 5-yr TCO direct | Verdict |
|---|---|---|---|---|---|
| A. Modular monolith, managed Postgres, partner logistics/payments | $4,100 | 6 | 15 Oct | $44.1M | **Chosen** |
| B. Service-based, 6 services + broker | $6,150 | 8 | ≈ 20 Nov | $46.4M | Rejected: misses date and cap; scale not needed before ≈ 3M MAU |
| C. Microservices + Istio + Kafka + Oracle, 3 clusters, build logistics/payments | $19,683 | 14 | ≈ Feb 2027 | $49.0M | Rejected: 3.9× cap, 2.3× team, $190k Oracle; pays back only above $4.6k/min downtime cost |

## Decision

We build a **Go modular monolith** with ten modules (Identity & Consent, Catalog & Search, Cart & Checkout, Orders, Payments, Fulfilment, Seller Onboarding, Notifications, Back Office, Analytics export). Modules own their PostgreSQL schema, talk through Go interfaces in-process and publish domain events via a **transactional outbox**; no module reads another module's tables (CN.LCP.05). One container image runs as three Kubernetes deployments — `api`, `worker`, `channels` (USSD/SMS/WhatsApp webhooks) — giving bulkheads without a distributed system. Data: **managed PostgreSQL HA + managed Redis** (open protocols). Logistics and payments are **partnered** (Sendbox, Sendy; M-Pesa Daraja, Flutterwave, Paystack) behind adapter ports. Runtime: managed Kubernetes in one region with three zones, GitOps (ArgoCD), OpenTelemetry.

**Not now (late binding):**

| Deferred decision | Until |
|---|---|
| Extracting modules into services; Kafka | A module breaches a trigger below (earliest ≈ month 6) |
| Multi-region active-active, 99.99% | Measured GMV-at-risk > $1.5k/min (Task3 NFR §1), or regulator demands it |
| Service mesh (Istio) | > 8 independently deployed services |
| Own logistics engine | Month 12 review with real volume and partner SLA data |
| Blockchain loyalty (A1, A5) | Listed in Architecture Vision as "evaluate after month 12"; no build |
| Final data placement per country | Counsel's opinion (≈ 5–19 Sep), see ADR-003 |

## Consequences

**Positive:** fits date, cap and team; one codebase for six people; 0 self-operated databases; cheapest 5-yr TCO; CN checklist level-1 items met (containers, K8s, requests/limits, probes, ≥ 2 replicas, GitOps, secrets, namespaces, metrics/logs).

**Negative:** single deployable — a bad release affects all modules (mitigated by canary via Argo Rollouts and feature flags); vertical limit of one Postgres ≈ 2k orders/min; module boundaries erode without discipline (mitigated by `go-arch-lint` in CI); CN.LCP.03 (independent component lifecycle) is only partly met until extraction; dependency on managed services of one provider (mitigated by open protocols and IaC); disagreement with the CTO's public position — he must own the message to the investor.

**Neutral:** Go and Kubernetes, the CTO's strengths, stay.

## Reversibility

Medium-low cost to reverse. Outbox events and per-module schemas make extraction a copy-and-route operation (≈ 4–6 engineer-weeks per module). Managed Postgres/Redis can be replaced by self-run or another provider's equivalents through Terraform modules and standard wire protocols; Kubernetes manifests run on GKE, EKS, AKS or on-prem.

## Revisit triggers

- p99 checkout latency > 800 ms at < 70% CPU for 2 weeks, or Postgres CPU > 60% sustained at peak.
- > 2M MAU or > 1,500 orders/min at peak.
- > 3 teams committing to one module, or deploy queue > 1 day.
- Measured GMV-at-risk > $1.5k/min — break-even of the 4th nine on top of A (Task3 NFR §1); option C as a whole breaks even at $4.6k/min (TCO Comparison sheet).

## Assumptions

- AS-1 Load at launch ≤ 100k MAU, 10× burst (Task1 canvas AS-1).
- AS-2 Managed Postgres in the chosen region supports HA across zones (verified in Task4).
- AS-3 The CTO accepts the trade-off if he presents it as "cloud native, extract-ready" — co-authored before Nairobi (A5 asks for a draft).
