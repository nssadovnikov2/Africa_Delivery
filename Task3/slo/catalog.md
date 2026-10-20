# SLO Contract — Catalogue (browse & search)

Owner: Catalog & Search module team · Consumers: buyer app, web deep links, USSD menus · Window: rolling 28 days · Structure: OpenSLO v1 (Service → SLI → SLO) + SLA.

## SLI — what and how we measure

| SLI | Good event | Valid event | Source |
|---|---|---|---|
| Availability | HTTP 2xx/3xx/404 response (incl. stale served by CDN or Redis) | All `GET /v1/catalog/*`, `/v1/search` at the edge | Cloudflare logs → Prometheus (`edge_requests_total{status}`) |
| Latency | Origin response < 300 ms | All origin catalogue requests | OTel histogram `http_server_duration_seconds` |
| Freshness | Price/stock change visible at edge ≤ 60 s | All `PriceChanged`/`StockChanged` events | Synthetic probe compares API vs DB event timestamp |

## SLO — targets

| SLO | Target | Error budget / 28 d |
|---|---|---|
| Availability | 99.95% | 20 min-equivalent of failed requests |
| Latency | 99% of origin requests < 300 ms | 1% slow requests |
| Freshness | 99% of changes ≤ 60 s | 1% late |

```yaml
apiVersion: openslo/v1
kind: SLO
metadata: {name: catalog-availability, displayName: Catalogue availability}
spec:
  service: catalog
  budgetingMethod: Occurrences
  timeWindow: [{duration: 28d, isRolling: true}]
  indicator:
    metadata: {name: catalog-good-ratio}
    spec:
      ratioMetric:
        counter: true
        good:  {metricSource: {type: Prometheus, spec: {query: 'sum(rate(edge_requests_total{route=~"catalog|search",status!~"5.."}[5m]))'}}}
        total: {metricSource: {type: Prometheus, spec: {query: 'sum(rate(edge_requests_total{route=~"catalog|search"}[5m]))'}}}
  objectives: [{displayName: availability, target: 0.9995}]
  alertPolicies: [catalog-burn-rate]
```

## SLA — what is guaranteed and what happens on breach

- **To the business (CEO):** monthly availability ≥ 99.9% (SLO minus 0.05 pp margin). Breach → written post-mortem within 3 working days and a reliability-only sprint for the module.
- **To sellers:** approved listing visible to buyers within 15 minutes, 99% of the time. Breach → listing boosted in "new arrivals" for 24 h.
- **Error-budget policy:** > 50% budget burnt in 7 days → feature freeze for catalogue changes; 100% burnt → only reliability work until the budget recovers.

## Assumptions

- AS-1 Catalogue reads are 90% served from CDN/Redis (ADR-002), so origin load ≈ 10% of edge traffic.
- AS-2 404 for delisted products counts as good (correct behaviour).
