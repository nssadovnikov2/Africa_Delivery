# Capability Map — Build / Buy / Partner

Date: 2026-08-28 · Inputs: all artifacts, [Task2 TCO](../Task2/tco-model.xlsx), [Task3](../Task3/README.md), [Task4](../Task4/README.md). Capabilities describe *what the business must be able to do*, independent of the PM's feature list (A4); features are mapped to capabilities in [roadmap.md](roadmap.md#the-pms-29-features-what-was-cut-and-why).

**Cost basis:** build = engineer-weeks × $1,250 (contractor $5k/month); buy = SaaS list price; partner = per-transaction fee (TCO Block 3). **Decision rule:** *Build* only what differentiates us against Jumia (trust, reach to WhatsApp/USSD sellers and rural buyers, low cost per order); *Buy* commodity software; *Partner* where a licence or a physical network is required (A3 p.4, case).

## L1 / L2 hierarchy

```
1 Customer Engagement        1.1 Buyer app & web  1.2 USSD/SMS commerce  1.3 Notifications  1.4 Localisation  1.5 Consent-based marketing & referrals  1.6 Live commerce
2 Catalogue & Merchandising  2.1 Catalogue & taxonomy  2.2 Search & discovery  2.3 Pricing & promotions  2.4 Moderation & takedown  2.5 Recommendations
3 Seller Management          3.1 Seller onboarding (WhatsApp/USSD)  3.2 Seller KYC & verification  3.3 Listings & inventory  3.4 Seller cabinet & analytics
4 Order Management           4.1 Cart & checkout  4.2 Order lifecycle  4.3 Returns & refunds  4.4 Disputes  4.5 B2B ordering
5 Payments & Finance         5.1 Payment acceptance  5.2 PSP routing  5.3 Seller payouts  5.4 Ledger & reconciliation  5.5 Credit / BNPL / wallet
6 Fulfilment & Logistics     6.1 Shipment booking  6.2 Pickup-point network  6.3 Track & trace  6.4 Address & location resolution  6.5 PUDO fraud evidence  6.6 Warehousing
7 Trust, Risk & Compliance   7.1 Identity & access  7.2 Fraud prevention  7.3 Data protection & residency  7.4 Consent management  7.5 Regulatory requests
8 Data & Insight             8.1 Executive live dashboard  8.2 Analytics & BI
9 Platform & Operations      9.1 Runtime & delivery  9.2 Observability  9.3 Customer support
```

## Decision table

| Capability (L2) | When | Decision | Rationale | Cost (Y1) | Risk |
|---|---|---|---|---|---|
| 1.1 Buyer app & web | MVP | **Build** (Flutter, one codebase) | Offline cache and data thrift are our edge on 3G (A2) | 8 ew = $10k | App-store review (R2) |
| 1.2 USSD/SMS commerce | MVP | **Build** flows + **Partner** aggregator | 30–40% of market (A2); aggregator holds short codes | 4 ew = $5k + $12k/yr sessions | Short-code approval time |
| 1.3 Notifications | MVP | **Buy** (SMS aggregators ×2, WhatsApp BSP, FCM) | Commodity; we build only routing and fallback | $36k SMS + $6k WA + $1.2k BSP /yr | Sender-ID block (FMEA 02) |
| 1.4 Localisation | MVP UI / M6 content | **Buy** translation service + i18n in app | 5 languages promised (A2 #5) | ≈ $3k one-off | Quality of Hausa/Yoruba |
| 1.5 Consent marketing & referrals | M6 | **Build** consent gate (MVP); **Buy** campaign tool at M6 | POPIA/NDPA (A3 p.2) | 2 ew = $2.5k | Pressure to use 410k list (C-05) |
| 1.6 Live commerce | M12+ | **Partner** (TikTok/Facebook Live + deep links) | Native streaming ≈ $1.7–5k per stream (C-11) | $0 | Lower conversion than in-app |
| 2.1 Catalogue & taxonomy | MVP | **Build** | Core of the marketplace; seller-friendly attributes | 5 ew = $6k | Category sprawl |
| 2.2 Search & discovery | MVP basic / M6 | **Build** on PostgreSQL FTS + trigram; reassess a search engine > 200k SKU | 5,000 SKU need no search cluster | 2 ew = $2.5k | Multilingual relevance |
| 2.3 Pricing & promotions | M6 | **Build** (flash sales with stock reservation) | Launch week uses fixed "deals" carousel | 3 ew at M6 | Oversell |
| 2.4 Moderation & takedown | MVP | **Build** in back office | Legal: KICA, counterfeit liability (A3 p.5, C-09) | 2 ew = $2.5k | Ops workload |
| 2.5 Recommendations | M12 | **Buy** (managed recommender) when data exists | No data at launch | — | — |
| 3.1 Seller onboarding WhatsApp/USSD | MVP (WA) / M6 (USSD) | **Build** flows on **Partner** BSP | Differentiator: sellers without a cabinet (case) | 4 ew = $5k | Meta template policy (FMEA 07) |
| 3.2 Seller KYC & verification | MVP | **Build** ops queue; **Buy** ID verification API at M6 | "Verified" badge promised (A2 #6) | 1 ew + ≈ $0.5/check at M6 | Fake documents |
| 3.3 Listings & inventory | MVP | **Build** (WhatsApp/CSV import) | | in 2.1 | |
| 3.4 Seller cabinet & analytics | M6 / M12 | **Build** web cabinet; analytics on **Buy** BI (Metabase) | Case 12-month goal | 6 ew at M6 | |
| 4.1 Cart & checkout | MVP | **Build** | Core flow, trust and conversion | 4 ew = $5k | — |
| 4.2 Order lifecycle | MVP | **Build** | State machine + outbox | 3 ew = $4k | — |
| 4.3 Returns & refunds | MVP manual / M6 | **Build** (refund via PSP APIs) | "Free 30-day returns" promise (A2 #4) | 1 ew MVP, 4 ew M6 | Return fraud |
| 4.4 Disputes | MVP | **Build** in back office | | in 2.4 | |
| 4.5 B2B ordering | M12 | **Build** (quotes, invoices); credit terms via **Partner** | Case 12-month goal; credit needs licence | 8 ew at M12 | Credit risk |
| 5.1 Payment acceptance | MVP | **Partner** (M-Pesa, Flutterwave, Paystack) | Licence required (A3 p.4); ADR-004 | 1.9% of GMV = $251k | Fee cap 2.5% (A6 p.6) |
| 5.2 PSP routing | MVP | **Build thin** adapter (not an orchestration platform) | CTO's orchestration: 12 eng-months ≈ $60k + $60k/yr upkeep to save 0.2 pp = $26k in Y1 — pays back only at GMV > $50M/yr | 2 ew = $2.5k | — |
| 5.3 Seller payouts | MVP | **Partner** (M-Pesa B2C, Flutterwave transfers) | Same-day after delivery (A1, A7) | in 5.1 | Float/fraud (C-07) |
| 5.4 Ledger & reconciliation | MVP | **Build** | Money correctness is ours (NFR-11) | 3 ew = $4k | — |
| 5.5 Credit / BNPL / wallet | Not planned | **Partner** later (licensed BNPL) | E-money / credit licences (A3 p.4); A4 #12, #13 | — | Regulatory |
| 6.1 Shipment booking | MVP | **Partner** (Sendbox NG, Sendy KE) | CTO's engine: 24 eng-months ≈ $120k + 2 FTE upkeep ($120k/yr) vs partner margin ≈ $0.045/order → break-even at ≈ 2.7–3.6M orders/yr (Year 2, incl. amortising the build over 3 years) → **revisit at M12** with real data | 3 ew adapters = $4k + $90k/yr fees | Partner reliability (R7) |
| 6.2 Pickup-point network | MVP | **Partner** (Sendbox points, local PUDO networks) | Physical network | in 6.1 | Coverage |
| 6.3 Track & trace | M6 | **Build** on partner webhooks | Case 12-month goal | 3 ew at M6 | Partner data quality |
| 6.4 Address & location resolution | MVP | **Build** landmark + pin + pickup-point choice; **Buy** geocoding later | No address system (case) | 2 ew = $2.5k | Failed deliveries |
| 6.5 PUDO fraud evidence (video) | M12 | **Partner** (PUDO operator records); store in-country | Legality open (A4 note) → DPIA first | — | Privacy |
| 6.6 Warehousing | Year 2+ | **Partner** (3PL) | Investor: "maybe year 2" (A4 notes) | — | — |
| 7.1 Identity & access | MVP | **Build** phone + OTP; **Buy** SMS | | in 1.3 | — |
| 7.2 Fraud prevention | MVP rules / M12 ML | **Build** rules; **Buy** fraud scoring at M12 | No training data at launch | 1 ew | COD abuse (R10) |
| 7.3 Data protection & residency | MVP | **Build** PII vaults on **Partner** local hosting | ADR-003 | 3 ew + $8.4k/yr hosting | Local provider SLA (FMEA 06) |
| 7.4 Consent management | MVP | **Build** ledger | Simple, residency-bound | in 7.3 | — |
| 7.5 Regulatory requests | MVP | **Build** in back office | KICA, data-subject rights | in 2.4 | — |
| 8.1 Executive live dashboard | MVP | **Buy** (Grafana Cloud) | Investor request (A2), PM #28 | in observability $2.4k/yr | — |
| 8.2 Analytics & BI | M6 | **Buy** (Metabase OSS self-hosted) | | $0 licence | — |
| 9.1 Runtime & delivery | MVP | **Buy** managed (GKE, Cloud SQL, GitHub, ArgoCD OSS) | CN.MSV.01; 6 people | $40.8k/yr | Provider lock-in (mitigated by open protocols) |
| 9.2 Observability | MVP | **Buy** (Grafana Cloud, Sentry) | | $4.2k/yr | — |
| 9.3 Customer support | MVP | **Buy** helpdesk with WhatsApp channel | | ≈ $100/mo | Volume at launch |

**Summary:** of 41 L2 capabilities, 25 are built — all as modules of the monolith, mostly thin flows, rules and back-office screens on top of partners — 8 are bought and 8 partnered. The two items the CTO wants to build (logistics, payment orchestration) are partnered now with explicit break-even triggers, not rejected forever.

## Assumptions

- AS-1 Partner margin on delivery ≈ 30% of the platform-paid $0.15/order (TCO); own engine still needs couriers.
- AS-2 Engineer-week estimates ±25%; total MVP build ≈ 44 ew (Task2).
