# Services per Capability

Date: 2026-08-28 · MVP state. "Module" = module of the Go modular monolith (ADR-001); owners are the six contractors (C1–C6, see [team-structure.md](team-structure.md)) plus CTO/SA.

| Capability (L1 → L2) | Product / service | Owner | External dependency |
|---|---|---|---|
| 1.1 Buyer app & web | Flutter app; deep-link pages served by `api` | C4 Mobile | Google Play, App Store, FCM/APNs |
| 1.2 USSD/SMS commerce | `channels` deployment — USSD menu engine | C3 Integrations | SMS/USSD aggregator (short codes NG, KE) |
| 1.3 Notifications | Notifications module (in `worker`), provider router | C3 Integrations | 2 SMS aggregators per country, WhatsApp BSP, FCM |
| 1.4 Localisation | i18n bundles (app + API) | C4 Mobile | Translation vendor |
| 1.5 Consent-based marketing | Consent gate in Identity & Consent module | SA | Country vaults |
| 2.1 Catalogue & taxonomy | Catalog & Search module | C2 Catalog | Cloud Storage, Cloudflare (images) |
| 2.2 Search | Catalog & Search module (PostgreSQL FTS + trigram) | C2 Catalog | — |
| 2.4 Moderation & takedown, 4.4 Disputes, 7.5 Regulatory requests | Back Office module + React back-office SPA | C5 Full-stack | — |
| 3.1 Seller onboarding | Seller Onboarding module (WhatsApp conversation flows) | C2 Catalog | WhatsApp BSP (Meta) |
| 3.2 Seller KYC | Back Office KYC queue; documents in vault | C5 Full-stack | Country vaults |
| 3.3 Listings & inventory | Catalog module: WhatsApp/CSV import | C2 Catalog | WhatsApp BSP |
| 4.1 Cart & checkout | Cart & Checkout module | C1 Payments | Redis (idempotency), PSPs |
| 4.2 Order lifecycle | Orders module (state machine, outbox) | C1 Payments | Pub/Sub |
| 4.3 Returns & refunds (manual in MVP) | Orders + Payments modules, back-office action | C1 Payments | PSP refund APIs |
| 5.1 Payment acceptance | Payments module — PSP adapters | C1 Payments | M-Pesa Daraja, Flutterwave, Paystack |
| 5.2 PSP routing | Payments module — router (country, method, health, cost) | C1 Payments | PSP status/health metrics |
| 5.3 Seller payouts | Payouts job in `worker` | C1 Payments | M-Pesa B2C, Flutterwave transfers |
| 5.4 Ledger & reconciliation | Ledger tables + daily reconciliation job | C1 Payments | PSP settlement reports |
| 6.1 Shipment booking | Fulfilment module — courier adapters (executed via vault egress) | C3 Integrations | Sendbox (NG), Sendy (KE) |
| 6.2 Pickup points | Fulfilment module — pickup-point directory sync | C3 Integrations | Sendbox / PUDO partners |
| 6.4 Address & location | Fulfilment module — landmark + GPS pin capture | C4 Mobile + C3 | Device GPS (offline capable) |
| 7.1 Identity & access | Identity & Consent module (phone + OTP; staff SSO) | SA | SMS aggregators, Google Workspace SSO for staff |
| 7.2 Fraud prevention (rules) | Rules in Checkout/Payouts (velocity, COD limits, payout after delivery) | C1 Payments | — |
| 7.3 Data protection & residency | PII vault service (NG, KE) | SA + C6 Platform | Local providers Lagos, Nairobi |
| 7.4 Consent management | Consent ledger in vaults | SA | — |
| 8.1 Executive live dashboard | Grafana dashboard on business metrics | C6 Platform | Grafana Cloud |
| 8.2 Analytics | Metabase on read replica (M6) | C5 Full-stack | — |
| 9.1 Runtime & delivery | GKE, ArgoCD, GitHub Actions, Terraform | C6 Platform | GCP, AWS (DR), GitHub |
| 9.2 Observability | OTel Collector, Grafana Cloud, Sentry | C6 Platform | Grafana Labs, Sentry |
| 9.3 Customer support | Helpdesk with WhatsApp channel | CEO's ops lead | Helpdesk SaaS |

## Assumptions

- AS-1 One owner per module avoids shared ownership; CTO reviews Payments and Fulfilment designs (his logistics background).
- AS-2 Operations staff (moderation, KYC, support) are hired by the CEO outside the engineering freeze.
