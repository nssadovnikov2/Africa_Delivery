# ADR-003: Data Residency Strategy — Country PII Vaults with a Pseudonymised Shared Core

| Field | Value |
|---|---|
| Status | Accepted (conditional — reviewed on receipt of counsel's opinion, expected 2026-09-05…19) |
| Date | 2026-08-27 |
| Deciders | CTO R. Menon (accountable), Solution Architect (responsible); Investor informed as risk owner |
| Consulted | A. Nwosu (counsel), CFO (cost of local providers) |
| Traces to | C-03 (NG data vs hyperscaler-only cloud), C-04 (do not finalise vs 15 Oct), C-05 (consent vs 410k numbers) in [contradictions](../Task1/contradictions.md); A1, A3 p.1–3, A5, A7 l.6 |

## Context

Counsel reads NDPR as requiring Nigerian personal data to be stored in Nigeria; masking "may be acceptable in certain configurations"; Kenya is "close to localisation" with stricter rules for payment data; POPIA demands provable consent (A3). Specifics (retention, transfer mechanisms, status of pseudonymised order data) arrive in 4–6 weeks, and counsel advises not to finalise the architecture before that — while launch is fixed for 15 October (A1). No hyperscaler has a region in Nigeria or Kenya ([cloud-regions-comparison.md](cloud-regions-comparison.md)). The investor's red line: no minister calling about Nigerian data abroad (A1).

## Decision drivers

1. Zero clear-text NG/KE personal data outside its country from day 1 (most conservative reading).
2. Launch on 15 Oct within $5k/month.
3. Counsel's answer must change configuration, not code (late binding).
4. Managed services for the core (ADR-001, CN.MSV.01).

## Considered options

| Option | Cost / mo | Compliance risk | Effort to 15 Oct | Verdict |
|---|---|---|---|---|
| A. Everything in a ZA hyperscaler region, encryption as "protection" | $3.4k | High — encryption does not relocate data (A3 p.1) | Low | Rejected |
| B. Full country cells from day 1 (whole platform in Lagos and Nairobi on local providers) | ≈ $8.5k + 1 SRE | Lowest | High: self-operated K8s/Postgres in 3 places | Rejected: breaks cap and team (A6) |
| **C. Country PII vaults in Lagos and Nairobi + pseudonymised shared core in ZA; each country deployable as a full cell** | $4.1k (vaults $0.7k) | Low for clear PII; residual: pseudonymised data abroad (counsel Q2) | Medium | **Chosen** |

## Decision

1. **Vaults.** A small Go service + PostgreSQL per country (Lagos, Nairobi; two hosts each, backups to a second in-country site) is the **only** place where clear-text personal data of that country lives: identity, contact, address, KYC documents, consent ledger, and (Kenya) payment references. ZA subjects use the same vault schema inside the ZA core.
2. **Tokens in the core.** The core (GCP africa-south1) stores random tokens; re-identification needs the vault. Outbound actions needing PII (SMS, WhatsApp, courier booking) are executed **by the vault from inside the country** ("in-country egress").
3. **Masking.** UI and back-office show masked values (`Ade*** O.`, `+234 80* *** **67`); unmasking is a logged, role-gated vault call.
4. **Encryption & keys.** At rest: provider encryption + field-level AES-256-GCM envelope encryption in vaults; one data key per subject, wrapped by a per-country KEK held in an in-country key store (HashiCorp Vault Transit on the vault hosts; ZA: Cloud KMS). Keys never leave the country. In transit: TLS 1.3; core ↔ vault mTLS. **Encryption is a security control, not a residency control.**
5. **Replication.** Vault DB: synchronous standby on the second host + encrypted backups to the second in-country site every 15 min; **never** across borders. Core DB: HA in 3 zones + logical replication to AWS af-south-1 (ZA → ZA).
6. **Retention (until counsel specifies):** identity/contact — account life + 24 months; orders and payment records — 7 years (longest tax rule of the three countries) in pseudonymised form; KYC — 5 years after the relationship ends; consent records — account life + 3 years; messaging content — 90 days; logs 14 days, traces 7 days. Erasure = crypto-shredding the subject's data key; tokens in the core become anonymous.
7. **Consent.** Purpose-level consent ledger from day 1; marketing sends are blocked without a consent record; the 410k purchased numbers are quarantined, not used (C-05).
8. **Cell readiness.** Every table carries `country` as part of its key; no cross-country joins; each country is a Terraform + Helm "cell" definition (`country → data_plane: shared|local`).

## Consequences

**Positive:** no clear personal data abroad on launch day; total extra cost $700/month; erasure and breach scope limited per country; the same pattern serves 12–15 countries.

**Negative:** an extra network hop (core → vault, 50–150 ms) on registration, notifications and courier booking — kept off the browse/checkout path by caching tokens; two more small environments to operate on less mature providers (FMEA 06); local-provider SLA ≈ 99.9% makes NG/KE registration slightly less available than checkout; analytics needing personal attributes must run inside each vault.

## Reversibility

The decision is designed to be **reversed or tightened in ≤ 10 working days without rewriting code**:

| Counsel says… | What we do | Effort / cost |
|---|---|---|
| Pseudonymised data may leave NG/KE (expected) | Nothing — status quo | 0 |
| All NG personal data incl. pseudonymised orders must stay in NG | Switch NG cell to `data_plane: local`: deploy the same Helm charts on the Lagos provider (RKE2 + CloudNativePG), export NG partitions by `country`, geo-steer NG traffic in Cloudflare | ≈ 10 working days; +$1.8k/mo, +0.5 SRE (CFO approval) |
| Even transit of NG data outside NG is prohibited | As above + NG API endpoints terminate in Lagos | +2 days |
| KE payment data rules looser than assumed | Move payment references from KE vault to core ledger (config flag) | 1 day |
| Rules relax or a hyperscaler opens a region in NG/KE | Migrate the vault into that region (same container + Postgres) | ≈ 3 days |

**What makes it reversible:** country partition key everywhere, vault API as the single PII path, Kubernetes + Postgres protocol portable to local providers, IaC per cell, quarterly restore test.

## Revisit triggers

- Counsel's opinion received (2026-09-05…19) — mandatory review within 3 working days.
- Launch of a new country (each gets a cell decision).
- Vault availability < 99.9% in a month or p99 detokenisation > 300 ms.

## Assumptions

- AS-1 Pseudonymised order data with keys held in-country may be processed in ZA (counsel Q2a); if not, the NG cell moves (table above).
- AS-2 Kenyan payment references are treated as "stricter" data and kept in Nairobi (counsel Q2b).
- AS-3 Retention periods above are placeholders, configurable per category and country.
