# SLO Contract — Payments (checkout, refunds, seller payouts)

Owner: Payments module (Checkout & Payments) · Consumers: buyer app, USSD, seller payouts · Window: rolling 28 days · OpenSLO v1 + SLA.

## SLI — what and how we measure

| SLI | Good event | Valid event | Source |
|---|---|---|---|
| Payment API availability | `POST /v1/payments` returns 2xx or a business decline (insufficient funds, cancelled STK) | All payment initiations; client 4xx excluded | Ingress + OTel metrics `payment_initiations_total{outcome}` |
| Payment completion | Intent reaches terminal state (paid / declined) ≤ 90 s | All intents for which the PSP accepted the request | Payment state machine events |
| Correctness | Ledger matches PSP settlement report | All transactions of day D | Daily reconciliation job |
| Payout timeliness | Seller payout initiated ≤ 4 h after delivery confirmation | All payouts | Payout outbox timestamps |

## SLO — targets

| SLO | Target |
|---|---|
| Payment API availability | **99.95%** (composite incl. PSP failover) |
| Completion ≤ 90 s | 99.5% |
| Reconciliation | 100% matched by T+1 (no error budget: any mismatch is an incident) |
| Payouts ≤ 4 h | 99% |

```yaml
apiVersion: openslo/v1
kind: SLO
metadata: {name: payments-availability, displayName: Payment initiation availability}
spec:
  service: payments
  budgetingMethod: Occurrences
  timeWindow: [{duration: 28d, isRolling: true}]
  indicator:
    metadata: {name: payment-good-ratio}
    spec:
      ratioMetric:
        counter: true
        good:  {metricSource: {type: Prometheus, spec: {query: 'sum(rate(payment_initiations_total{outcome=~"accepted|declined_business"}[5m]))'}}}
        total: {metricSource: {type: Prometheus, spec: {query: 'sum(rate(payment_initiations_total{outcome!="client_error"}[5m]))'}}}
  objectives: [{displayName: availability, target: 0.9995}]
  alertPolicies: [payments-burn-rate-fast, payments-burn-rate-slow]
---
apiVersion: openslo/v1
kind: AlertPolicy
metadata: {name: payments-burn-rate-fast}
spec:
  alertWhenBreaching: true
  conditions: [{kind: AlertCondition, metadata: {name: fast-burn}, spec: {severity: page, condition: {kind: burnrate, op: gte, threshold: 14.4, lookbackWindow: 1h, alertAfter: 5m}}}]
```

Per-PSP SLIs (`psp="mpesa|flutterwave|paystack"`) drive the circuit breaker and failover ([FMEA](../fmea.md) 03, 09).

## SLA — what is guaranteed and what happens on breach

- **To buyers:** refund *initiated* within 1 minute of approval; M-Pesa reversal ≤ 1 h, cards per card-network timelines (communicated in-app). Breach → $1 voucher.
- **To sellers:** payout same day for deliveries confirmed by 18:00 local time. Breach → payout fee waived for that order; 3 breaches in a month → escalation to CFO.
- **To the business (CFO):** payment cost ≤ 2.5% of GMV all-in (A6 p.6, ADR-004) — reported monthly.
- **Error-budget policy:** fast burn (14.4× in 1 h) pages on-call; > 50% of budget burnt → freeze of checkout changes; reconciliation mismatch → P1 incident regardless of budget.

## Assumptions

- AS-1 PSPs do not offer contractual SLAs at our volume; partner availability ≈ 99.9% (Task3 NFR AS-3).
- AS-2 Same-day payout is paid only after delivery confirmation to avoid funding fraud (Task1 C-07).
