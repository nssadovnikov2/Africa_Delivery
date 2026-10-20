# Roadmap Risk Register

Date: 2026-08-28 · Probability / impact: H / M / L · Reviewed weekly by CTO and SA; top 5 shown to the CEO.

| ID | Risk | Probability | Impact | Mitigation | Owner | Trigger (early signal) |
|---|---|---|---|---|---|---|
| R1 | PSP production access late (Safaricom Daraja go-live, Flutterwave KYB) — and without Daraja-direct the fee rises to ≈ 2.8% (ADR-004) | H | H | Submit KYB in week 1; Paystack in parallel; COD as fallback; CFO escalates via account managers | CFO + C1 | No production keys by 2026-09-20 |
| R2 | App Store / Play review delays the 1 Oct store pages | M | H | Play internal testing from 09-15; iOS TestFlight 09-22; submit 09-24; web deep-link fallback in creatives | C4 | Rejection or no approval by 09-28 |
| R3 | Counsel's opinion requires full NG cell in Nigeria | M | M | ADR-003 reversibility runbook (≤ 10 days); local provider RFQ includes "full cell" option | SA | Opinion says pseudonymised data in scope |
| R4 | Scope creep from investor ("everything the buyer touches") | H | H | CEO owns scope; rule "one in, one out"; decisions logged with cost and date impact | CEO | > 2 unplanned items in a week |
| R5 | Launch-day load above plan (radio, 1M promise) | M | H | k6 at 2× target from 09-21; HPA; load shedding; CDN stale mode; queue for OTP | C6 | Load test > 70% of capacity, or real traffic > 50% of tested peak |
| R6 | Contractor attrition or key-person loss (CTO/SA; no local staff) | M | H | Pairing on Payments and vaults; ADRs and runbooks; contracts to December with notice period | CTO | Any resignation notice; velocity −30% two weeks running |
| R7 | Courier partner cannot meet "same day" or partner viability issue (Sendy publicly reported restructuring in 2023) | M | H | Second courier per city under the same adapter (e.g. Kwik in Lagos; Pickup Mtaani / Fargo in Kenya); SLA in contract | C3 + CEO | On-time delivery < 85% in launch week; partner API errors > 5% |
| R8 | Seller supply: 100 sellers / 5,000 SKU not ready | M | H | CEO-led field team in Lagos/Nairobi; ops-assisted bulk import from WhatsApp catalogues/CSV | CEO | < 60 sellers live by 09-30 |
| R9 | Cloud bill above $5k/month | L | M | Budgets and alerts at 80%; weekly bill review; scale-to-zero for non-prod at night | C6 + CFO | Forecast > $4,500 |
| R10 | Fraud at launch: COD refusals, fake sellers, refund abuse | M | M | COD limit per new buyer; KYC before first payout; payout after delivery; velocity rules | C1 | COD refusal > 15%; refund rate > 8% |
| R11 | SMS sender-ID registration delayed (NG/KE), OTP unreliable | M | H | Register sender IDs in week 1 via two aggregators; WhatsApp OTP fallback | C3 | Not approved by 09-25 |
| R12 | CTO does not endorse ADR-001 and escalates to the investor | M | H | Co-authored draft before Nairobi; CTO owns the extraction triggers and the M12 logistics decision | SA | ADR-001 unsigned by 2026-08-21 |
| R13 | Marketing promises not delivered as advertised (same day everywhere, instant refunds, 5 languages) | H | M | "Deliver differently" list approved by CEO; legal review of claims; FAQ in app | CEO + Marketing | Complaint rate on promises > 2% of orders |

## Assumptions

- AS-1 Probability and impact are team estimates on 2026-08-28; re-scored at each checkpoint.
- AS-2 Owners C1–C6 as defined in [team-structure.md](team-structure.md).
