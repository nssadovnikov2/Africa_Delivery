# SLO Contract — Notifications (SMS, WhatsApp, push, OTP)

Owner: Notifications module (runs in `worker`; sends PII-bound messages through country vaults) · Window: rolling 28 days · OpenSLO v1 + SLA.

## SLI — what and how we measure

| SLI | Good event | Valid event | Source |
|---|---|---|---|
| OTP latency | OTP delivery report (DLR) ≤ 30 s after request | All OTP requests to reachable numbers | Aggregator DLR webhooks → `notification_delivery_seconds{type="otp"}` |
| Transactional delivery | Delivered ≤ 5 min on any channel (SMS → WhatsApp → push fallback) | All order/payment/delivery status messages | DLR + WhatsApp status callbacks |
| Queue health | Message dequeued ≤ 60 s after enqueue | All messages | Outbox lag metric |

## SLO — targets

| SLO | Target |
|---|---|
| OTP ≤ 30 s | 99% |
| Transactional ≤ 5 min | **99.9%** |
| Queue lag ≤ 60 s | 99.5% |

```yaml
apiVersion: openslo/v1
kind: SLO
metadata: {name: notifications-delivery, displayName: Transactional notifications delivered within 5 min}
spec:
  service: notifications
  budgetingMethod: Occurrences
  timeWindow: [{duration: 28d, isRolling: true}]
  indicator:
    metadata: {name: delivered-within-5m}
    spec:
      ratioMetric:
        counter: true
        good:  {metricSource: {type: Prometheus, spec: {query: 'sum(rate(notification_delivery_seconds_bucket{type="transactional",le="300"}[5m]))'}}}
        total: {metricSource: {type: Prometheus, spec: {query: 'sum(rate(notification_delivery_seconds_count{type="transactional"}[5m]))'}}}
  objectives: [{displayName: delivered-5m, target: 0.999}]
```

## SLA — what is guaranteed and what happens on breach

- **To buyers and sellers:** best effort with fallback across channels; no financial credit. Marketing messages are excluded and require a consent record (NFR-15).
- **To the business:** monthly report of delivery rate per country and provider; SMS sender-ID block (carrier filtering) is a P1 incident.
- **Error-budget policy:** budget burnt → switch default provider for the affected country and freeze template changes.

## Assumptions

- AS-1 Two SMS aggregators per country (e.g. Africa's Talking + Termii in NG) are contracted before launch.
- AS-2 Notifications tolerate minutes of delay, hence the lower 99.9% target than checkout.
