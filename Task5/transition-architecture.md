# Transition Architecture

Date: 2026-08-28 · Now → MVP → 6 months → 12 months. Each step adds or extracts; none rewrites. Traces to ADR-001 (triggers), ADR-003 (cells), ADR-004 (PSP adapters), [roadmap.md](roadmap.md).

## Invariants — what never changes, so nothing has to be rewritten

1. **Module boundaries** of the monolith: own schema, own Go interface, events via outbox (ADR-001).
2. **Event contracts** (`OrderPlaced`, `PaymentSucceeded`, `ShipmentBooked`, `ProductUpdated` …) versioned in Git — the same events later go over a broker.
3. **`country` partition key** in every table and every event; **vault API** is the only path to personal data (ADR-003).
4. **Adapter ports** for PSPs, couriers, messaging (ADR-004) — partners are swapped, not re-integrated.
5. **Kubernetes manifests + Terraform per cell**, OpenTelemetry, open DB/cache protocols — any provider.

## States

| | **Now (Aug 2026)** | **MVP (15 Oct 2026)** | **6 months (Apr 2027)** | **12 months (Oct 2027)** |
|---|---|---|---|---|
| Business | No system, no team; ads booked | NG + KE pilot, 100 sellers, 5k SKU | + ZA; seller cabinet; returns; track & trace | 5 countries, > 1M users, B2B |
| Application | — | Go modular monolith, 10 modules; `api` / `worker` / `channels` | + modules: Returns, Reviews, Promotions (flash sales), Seller Cabinet API | + B2B module; **first extractions only if triggers fire** — candidates: Catalog & Search (read-heavy), Notifications (volume) |
| Integration | — | In-process calls + Postgres outbox + Pub/Sub jobs | Courier webhooks → track & trace events | Extracted services consume the **same** outbox events through Pub/Sub (managed); Kafka only if > 10k events/s |
| Data | — | Cloud SQL HA + 1 replica; Redis; vaults NG, KE | + ZA vault schema & consent ledger; Metabase on replica | Per-country cells; pseudonymised analytics in BigQuery africa-south1; search engine if > 200k SKU |
| Clients | — | Flutter app (Android, iOS), USSD, WhatsApp seller flows, back office | + seller web cabinet, USSD seller onboarding, 5 UI languages | + B2B web portal, French content, public partner API (REST) |
| Infrastructure | Accounts, invoice billing applied | GKE regional (3 zones), AWS pilot-light DR, Cloudflare, local vault hosts | + ZA consent; DR drills quarterly | + Ghana, Côte d'Ivoire vault cells; if GMV-at-risk > $1.5k/min: active-active with Azure SA North or AWS (Task3) |
| Availability | — | 99.95% checkout / catalogue, 99.9% notifications | Same, measured | 99.95% → 99.99% only by trigger |
| Team | Investor, CEO, CFO, CTO, SA | 6 contractors | + PM, ops team | 10 engineers + PM (2 stream-aligned teams + platform) |
| Cost (cloud + SaaS) | $0 | $4,100/mo | ≈ $6,500/mo | ≈ $10,800/mo |

## What changes at each step and why it is not a rewrite

- **Now → MVP:** greenfield build of the monolith, vaults and adapters. Risk is time, not design — scope cut to 9 features (roadmap).
- **MVP → 6 months:** new country = new cell configuration (Terraform + Helm values + vault schema), not new code. New capabilities are new modules behind the same API gateway path. Track & trace consumes courier webhooks through the existing Fulfilment adapter.
- **6 → 12 months:** extraction pattern (strangler): (1) the module already owns its schema → copy schema to its own Cloud SQL instance with logical replication; (2) deploy the module as its own service from the same repository; (3) route its API paths at the ingress and switch event consumption from in-process to Pub/Sub; (4) remove the in-process call. ≈ 4–6 engineer-weeks per module. Logistics: if volume ≥ 2.7–3.6M orders/yr and partner SLA is poor, an own dispatch engine replaces the courier adapter behind the same port (capability map 6.1).

## Assumptions

- AS-1 Triggers in ADR-001 are measured from month 1 on the observability stack (Task3).
- AS-2 Managed Pub/Sub is sufficient up to ≈ 10k events/s; above that a managed Kafka is evaluated (CTO's backbone, by evidence).
