# Data Residency & Sovereignty Map: African Operations

* Last Updated: 2026-08-27
* Primary Infrastructure Providers: Google Cloud (africa-south1, Johannesburg) — core; AWS (af-south-1, Cape Town) — DR; local providers in Lagos and Nairobi — PII vaults
* Compliance Jurisdictions:
  * Nigeria: NDPR 2019 / Nigeria Data Protection Act (NDPA) 2023
  * South Africa: POPIA
  * Kenya: Data Protection Act 2019 (DPA) + Data Protection (General) Regulations 2021, read with KICA
* Inputs: A3 (counsel's letter), A1/A7 (investor: "data stays in the country"), [Task1/open-questions.md](../Task1/open-questions.md) Q2, [Task2/architecture-vision.md](../Task2/architecture-vision.md)

> **Deviation from the course template.** The template sample places Nigerian financial data in AWS Frankfurt under SCCs and treats "cryptographic adequacy" as a substitute for local hosting. We reject both: counsel reads NDPR as requiring storage in Nigeria (A3 p.1), and **encrypted personal data is still personal data** — encryption protects it, it does not relocate it. Europe is not used for any personal data.

## 1. Executive Summary

Clear-text personal data of Nigerian and Kenyan subjects is stored and processed **only in-country**, in a per-country PII vault on a local provider. The shared platform in South Africa processes **pseudonymised** records (tokens, SKU, amounts, pickup-point IDs). South African subjects' data stays in South Africa by default. Placement is a per-country configuration (ADR-003), so counsel's final opinion (expected 5–19 Sep) changes deployment, not code.

## 2. What is known vs assumed

| Topic | Nigeria | Kenya | South Africa |
|---|---|---|---|
| Must personal data stay in-country? | **Known (counsel):** yes, transfers abroad only in limited cases (A3 p.1) | **Known (counsel):** "close to localisation"; payment-related data stricter (A3 p.3). **Assumed:** a serving copy in Kenya satisfies it | **Known:** no localisation; cross-border allowed with adequate protection (POPIA s.72) |
| Is pseudonymised / masked data in scope? | **Assumed:** masking acceptable "in certain configurations" (A3 p.1) → processing abroad allowed if re-identification keys stay in NG — counsel Q2(a) | **Assumed:** same — counsel Q2(b) | Pseudonymised data is personal data if re-identifiable; we keep it in ZA anyway |
| Consent management mandatory | **Known:** for direct marketing (A3 p.2 "probably NG") | **Known:** DPA s.37 restricts commercial use without consent | **Known:** POPIA s.69 — opt-in, provable consent before direct electronic marketing (A3 p.2, case) |
| Registration / officer | **Known:** register with NDPC, appoint DPO, annual audit filing (A3 p.1) | **Known:** register with ODPC (A3 p.3) | **Known:** register Information Officer with the Information Regulator |
| Breach notification | **Assumed:** 72 h to NDPC | **Known:** 72 h to ODPC | **Known:** "as soon as reasonably possible" |
| Content takedown on regulator request | — | **Known:** KICA (case) | — |
| Retention periods | **Unknown** — counsel (A3 closing) | **Unknown** | **Unknown** |

## 3. Infrastructure Footprint Matrix

| Provider | Region / site | Code | Primary compliance role | Target data subjects |
|---|---|---|---|---|
| GCP | South Africa (Johannesburg) | africa-south1 | Core platform: catalogue, orders (pseudonymised), payments ledger (tokens), ZA personal data (from month 6) | All (tokens); ZA (clear) |
| AWS | Africa (Cape Town) | af-south-1 | DR pilot light: logical replica of core DB, backups | All (tokens); ZA (clear) |
| Local provider (shortlist: Layer3, Nobus Cloud, Kasi Cloud) | Lagos, 2nd site Abuja for backups | ng-lagos | Nigerian PII vault + consent ledger + in-country egress (SMS, courier booking) | Nigeria |
| Local provider (shortlist: Africa Data Centres, iXAfrica via local IaaS, Safaricom Cloud) | Nairobi, 2nd site Nairobi DC2 for backups | ke-nairobi | Kenyan PII vault + payment references + consent ledger | Kenya |
| Cloudflare | Global edge (PoPs in Lagos, Nairobi, Johannesburg) | — | TLS termination, cache of public catalogue only | None (no PII cached) |
| Grafana Cloud, Sentry | EU | — | Telemetry, PII scrubbed at source | None |

## 4. Data Categories & Residency Mapping

| # | Data category | Data subject country | Where stored | Where processed | Legal basis | Technical measure |
|---|---|---|---|---|---|---|
| D1 | Buyer identity & contact (name, phone, email) | NG | Lagos vault | Lagos vault (OTP, SMS, courier booking sent from NG) | Contract (NDPA s.25(1)(b)) | Tokenisation; field-level AES-256 envelope encryption, KEK in in-country key store; masked display (`+234 80* *** **67`) |
| D1 | — | KE | Nairobi vault | Nairobi vault | Contract (DPA s.30(1)(b)) | Same |
| D1 | — | ZA | GCP africa-south1 (vault schema) | GCP africa-south1 | Contract (POPIA s.11(1)(b)) | Same, KEK in Cloud KMS ZA |
| D2 | Delivery address, landmarks, GPS pin, pickup-point choice | NG / KE / ZA | Country vault (ZA: core) | Country vault → courier API from inside the country | Contract | Only pickup-point ID and city leave the vault |
| D3 | Seller KYC: ID document, business registration, payout account | NG / KE / ZA | Country vault (object store, encrypted) | Country vault; ops view via masked back-office | Legal obligation (KYC, counterfeit liability A3 p.5) | Documents encrypted per object; access logged; 2-person access |
| D4 | Payment references (PSP transaction ID, masked MSISDN, card last-4) | KE | Nairobi vault (stricter rule, A3 p.3) | PSP (Safaricom, in Kenya); token in core ledger | Contract; legal obligation (tax) | No card data at all (PSP hosted checkout/tokens) — PCI scope SAQ-A |
| D4 | — | NG / ZA | Core ledger (tokens + amounts) | Core | Contract | PSP tokens only; PAN never received |
| D5 | Orders & transactions (pseudonymised: buyer token, seller ID, SKU, amount, pickup-point ID, timestamps) | All | GCP africa-south1 (+ AWS af-south-1 replica) | GCP africa-south1 | Contract; legal obligation (tax records) | No direct identifiers; re-identification only via country vault |
| D6 | Consent records (purpose, channel, timestamp, evidence) | NG / KE | Country vault | Country vault; core receives `consent=true/false` per purpose | Legal obligation (demonstrable consent) | Append-only ledger, hash-chained |
| D6 | — | ZA | GCP africa-south1 | GCP africa-south1 | POPIA s.69 | Same; mandatory before ZA launch |
| D7 | Messaging content (WhatsApp, SMS, USSD sessions) | NG / KE | Country vault (90 days) | BSP/aggregator → webhook to `channels` → PII moved to vault, token kept in core | Contract | Webhook payload PII redacted before persistence in core |
| D8 | Catalogue, seller public profile, prices | — (business data) | GCP africa-south1, CDN | Anywhere | Legitimate interest | Public data |
| D9 | Telemetry, logs, traces | All | Grafana Cloud (EU), Sentry | EU | Legitimate interest | PII scrubbing in OTel Collector; weekly DLP scan (NFR-14) |
| D10 | CEO dashboard / analytics aggregates | All | GCP africa-south1 | GCP africa-south1 | Legitimate interest | Aggregates only (k ≥ 10) |
| D11 | Purchased marketing list (410k numbers) | NG / KE (assumed) | Quarantined, not loaded | Not processed | **None** — no consent (A3 p.2) | Held pending counsel; not used for SMS (C-05) |
| D12 | PUDO video evidence (month 12+) | NG / KE / ZA | Country vault / local NVR | In-country only | To be defined (DPIA required; legality open, A4 note) | Not built until counsel confirms |

## 5. Regional Data Flows & Sovereignty Controls

### South Africa (POPIA)
* Hosting: GCP africa-south1; DR AWS af-south-1 — both in South Africa.
* Rule: consent ledger and opt-in marketing must be live **before** the ZA launch (month 6). No SMS to purchased numbers.

### Kenya (DPA 2019, KICA)
* Hosting: Nairobi vault (local provider), backups to a second Nairobi site.
* Rule: Kenyan PII and payment references stay in Kenya; core gets tokens. Breach: processors → controller 48 h, controller → ODPC 72 h. KICA takedown: back-office function with ≤ 24 h SLA and audit log (NFR-16).

### Nigeria (NDPR / NDPA)
* Hosting: Lagos vault (local provider), backups to the provider's Abuja site — **never to Europe or South Africa**.
* Rule: only pseudonymised records leave Nigeria, re-identification keys never do. If counsel rejects this (Q2a), the full Nigerian cell moves to Lagos (ADR-003 reversibility).

## 6. Security & Technical Compliance Controls

* Isolation: separate cloud projects/accounts per environment; vaults in separate tenants per country; org policy restricts GCP resource locations to `africa-south1`, AWS SCP restricts to `af-south-1`.
* Keys: per-country key hierarchy; NG/KE keys never leave the country; crypto-shredding of a subject's data key implements erasure requests.
* Audit: access to vaults and back-office logged to an append-only table in the same country; quarterly access review by the DPO.

## Assumptions

- AS-1 Local providers in Lagos and Nairobi can provide 2 VMs in separate racks, a second site for backups, invoice billing and ISO 27001 — RFQ by 2026-09-04.
- AS-2 Counsel's opinion arrives 5–19 Sep (A3); defaults in §2 apply until then.
- AS-3 The 410k numbers come from Nigeria/Kenya; provenance asked in Task1 Q5.
