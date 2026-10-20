# ADR-004: Payment Providers and Routing

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-08-28 |
| Deciders | CFO A. Weber (accountable — fee cap and contracts), CTO R. Menon, Solution Architect |
| Consulted | Counsel (licensing, A3 p.4), CEO |
| Traces to | C-02 (build payment orchestration vs date), C-07 (same-day payouts, instant refunds, wallet vs licensing) in [contradictions](../Task1/contradictions.md); A1, A3 p.4, A5, A6 p.4/p.6, A7; case MVP: "payment via M-Pesa and Flutterwave" |

## Context

MVP must accept M-Pesa and Flutterwave (case). Only ≈ 20% of buyers use cards (case); M-Pesa dominates Kenya, bank transfer/USSD-pay and cards dominate Nigeria; marketing promised "M-Pesa, card, bank transfer or cash to the courier" (A2 #3). We cannot hold a payment licence by October (A3 p.4). The CFO caps all-in payment cost at **2.5% of GMV incl. conversion** (A6 p.6). The CTO wants to build payment orchestration and talk to M-Pesa and Flutterwave directly (A5). The investor wants sellers paid the same day (A1, A7).

## Decision drivers

1. Licensed partner holds the money (A3 p.4).
2. Blended cost ≤ 2.5% GMV with ≥ 0.5 pp headroom.
3. Checkout availability 99.95% → at least two rails per country (Task3 §1).
4. Coverage of the 5 countries planned for month 12.
5. No card on file / invoice: fees netted from settlements (A6 p.4).

## Provider geography and indicative fees

| Provider | Countries relevant to the roadmap | Methods | Indicative fee (public tariffs, Aug 2026; to be confirmed by quotes, A6 p.2) | Payouts |
|---|---|---|---|---|
| M-Pesa — Safaricom Daraja (direct) | Kenya (Vodacom M-Pesa: Tanzania, DRC, Mozambique, Lesotho, Egypt — separate contracts) | STK push (Lipa na M-Pesa), C2B | ≈ 0.55% merchant tariff | B2C ≈ KES 15–35 per transfer |
| Flutterwave | Nigeria, Kenya, Ghana, South Africa, Uganda, Tanzania, Rwanda, Côte d'Ivoire, Senegal, Cameroon, Egypt (30+) | Cards, bank transfer, USSD-pay, M-Pesa, mobile money | NG local ≈ 1.4% (capped); KE M-Pesa via Flutterwave ≈ 2.9% | Transfers ≈ NGN 10–50 |
| Paystack (Stripe) | Nigeria, Ghana, Kenya, South Africa, Côte d'Ivoire | Cards, bank transfer, USSD, M-Pesa (KE), EFT (ZA) | NG 1.5% + NGN 100 (capped NGN 2,000); KE M-Pesa 1.5%, cards 2.9%; ZA cards ≈ 2.9% | Transfers ≈ NGN 10–50 |
| Cash on delivery | NG, KE | Courier collects (Sendbox/Sendy) | ≈ 1.5% handling | Remitted by courier T+1–2 |

## Considered options

| Option | Blended cost, % GMV | Availability | Time to launch | Verdict |
|---|---|---|---|---|
| A. Single aggregator (Flutterwave) everywhere, incl. M-Pesa | NG 1.43%, **KE ≈ 2.83%** → weighted 2.06% + payouts/reserve/FX 0.7 pp ≈ **2.8%** | Single rail | Fastest | Rejected: breaks the 2.5% cap |
| B. CTO: own payment orchestration platform, direct contracts everywhere | ≈ 1.7% (−0.2 pp) | Depends on our build | +12 eng-months ($60k) + $60k/yr upkeep; not by 15 Oct | Rejected now: saving $26k in Y1 < cost; pays back at GMV > $50M/yr (≈ Year 2) |
| **C. Direct M-Pesa in KE + Flutterwave primary / Paystack standby in NG, thin routing adapter in our monolith** | **≈ 1.9%** (see below) | 2 rails per country | Fits MVP (2 ew for the router) | **Chosen** |

## Decision

- **Kenya:** M-Pesa via **Safaricom Daraja direct** (STK push for checkout, B2C for payouts) — primary; **Flutterwave** for cards and as the secondary M-Pesa route when Daraja is down (circuit breaker, FMEA 03).
- **Nigeria:** **Flutterwave** primary (cards, bank transfer, USSD-pay, payouts); **Paystack** hot standby for cards/transfers (FMEA 09); **cash on delivery** via Sendbox with order limits.
- **South Africa (month 6):** Paystack primary (cards, EFT) + Flutterwave secondary; ZA mix to be priced before launch — ZA cards alone exceed the cap, so EFT/instant-payment share is a launch condition.
- **Countries 4–5 (month 12):** Flutterwave/Paystack both cover Ghana and Côte d'Ivoire; choose by quote.
- **Routing:** a thin `PaymentRouter` in the Payments module chooses PSP by country, method, health (SLI) and cost; adapters behind one port interface. No card data touches us (hosted fields/redirect → PCI SAQ-A).
- **Payouts:** same day **after delivery confirmation**, aggregated per seller per day (one transfer fee instead of one per order).
- **Refunds:** initiated instantly via PSP refund API; M-Pesa reversal minutes, cards per network timelines. **No wallet, no BNPL** (licences, C-07).

**Blended cost (Year 1 mix: NG 55% / KE 45% of GMV):** NG = 75% digital × 1.4% + 25% COD × 1.5% = 1.43%; KE = 80% M-Pesa × 0.55% + 15% cards × 2.9% + 5% COD × 1.5% = 0.95% → weighted **1.21%** + payouts 0.2% + refunds/chargeback reserve 0.2% + FX on treasury sweeps 0.3% (all collections and payouts in local currency; only margin is converted) = **≈ 1.9% ≤ 2.5%** — the figure used in the TCO (Block 3).

**Guardrail:** cost per country is a monthly CFO metric; if Daraja-direct is not live, KE M-Pesa goes via Flutterwave (≈ 2.9%), KE rises to ≈ 2.8% and the blended rate to ≈ 2.8% — above the cap — **Daraja production approval is therefore a launch-critical path item (R1)**.

## Consequences

**Positive:** within the cap with 0.6 pp headroom; two rails per country; licences stay with partners; fees netted from settlements (no invoices, no card); covers the month-12 countries.

**Negative:** three PSP contracts and KYB processes in 6 weeks (R1); reconciliation across three settlement formats; COD carries cash-handling and refusal risk (R10); the 0.2 pp the CTO targets is left on the table until volume justifies renegotiation.

## Reversibility

High: adapters behind one interface; routing rules are configuration; adding or dropping a PSP ≈ 2–3 engineer-weeks. Building a full orchestration platform later reuses the same ports.

## Revisit triggers

- GMV > $50M/yr (≈ Year 2) → renegotiate rates / reconsider orchestration (CTO option B).
- Blended cost > 2.2% for two consecutive months.
- PSP availability < 99.9% in a month or a PSP exits a country.

## Assumptions

- AS-1 Fees are indicative public tariffs; binding numbers come from commercial proposals before contract signature (A6 p.2).
- AS-2 Payment mix: NG 25% COD, KE 80% M-Pesa — to be measured from week 1.
- AS-3 Average order $22 (TCO), so NGN caps rarely apply.
