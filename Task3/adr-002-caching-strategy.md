# ADR-002: Caching Strategy

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-08-24 |
| Deciders | CTO R. Menon (accountable), Solution Architect (responsible) |
| Consulted | Catalog, Payments leads; Head of Marketing (image quality) |
| Traces to | C-01 (availability within budget), C-10 (premium images vs 3G) in [contradictions](../Task1/contradictions.md); A1 "never down", A2 "< 2 s", "megabytes are counted"; NFR-02, NFR-05, NFR-06, NFR-12 |

## Context

We must serve 99.95% catalogue availability and < 2 s to the first product on 3G (A2) on a $4,100/month platform (ADR-001) with one PostgreSQL primary. Launch bursts are 10× (QAS-04). Prices and stock must never be wrong at checkout. A "Redis for everything" approach without TTLs and invalidation would serve stale prices and hide outages.

## Decision drivers

1. ≥ 90% of catalogue reads served without touching PostgreSQL.
2. Price/stock change visible ≤ 60 s for 99% of changes (catalog SLO).
3. Catalogue usable when origin or network is down (degrade, don't fail).
4. No personal data in shared caches (ADR-003).

## Considered options

| Option | Pros | Cons |
|---|---|---|
| A. No cache, scale DB | Simple, always fresh | Read replicas cost ≈ $350/mo each; no offline; fails QAS-03/04 |
| B. Redis only (server side) | Simple | Every 3G client still pays RTT to Johannesburg; no offline; Redis becomes a SPOF |
| **C. Multi-layer: app cache + CDN + Redis cache-aside, TTL + event invalidation** | Offline, low latency, DB protected, cheap | Invalidation logic in 3 layers; staleness window must be designed |

## Decision

Option C, with explicit rules:

| Layer / where | What is cached | TTL | Invalidation |
|---|---|---|---|
| L0 Mobile app (SQLite) | Category tree; product cards and lists viewed; thumbnails | Tree 24 h; cards 15 min; images until evicted (LRU 150 MB) | `ETag` revalidation on open; `catalog_version` in every API response forces refresh of changed categories |
| L1 CDN (Cloudflare) | Product images | 1 year, immutable | Content-hashed URL; new image = new URL (never purged) |
| L1 CDN | Anonymous `GET` category lists, product detail, top-100 search queries | 60 s + `stale-while-revalidate=300` + `stale-if-error=86400` | Purge-by-URL from `ProductUpdated`/`PriceChanged` events; TTL bounds the rest |
| L2 Redis (Memorystore HA, 2 GB) — cache-aside in `api` | Product detail (all languages) | 10 min ± 10% jitter | Key delete on `ProductUpdated`, `PriceChanged`, `StockChanged` from the outbox |
| L2 Redis | Category list pages | 5 min ± 10% | Delete category keys on product change in that category |
| L2 Redis | Search results (normalised query) | 2 min | TTL only |
| L2 Redis | Stock badge (in stock / low / out) | 60 s | `StockChanged` event |
| L2 Redis (not a cache) | OTP rate limits, idempotency keys | 10 min / 24 h | Expiry |

**Never cached:** cart totals, checkout prices and stock (read from PostgreSQL inside the order transaction), payment and payout state, personal data (Redis holds tokens only), seller balances.

**Stampede protection:** Go `singleflight` per key, TTL jitter, CDN `stale-while-revalidate`. **Failure mode:** Redis down → read-through to PostgreSQL with a concurrency limit (bulkhead) and CDN serving stale; app falls back to L0 (FMEA 10).

**Images (C-10):** three WebP renditions (thumb ≤ 25 KB, card ≤ 80 KB, zoom ≤ 300 KB loaded only on tap) — premium on demand, not by default.

## Consequences

**Positive:** ≈ 90% edge + Redis hit ratio removes the need for extra read replicas for buyer traffic (≈ $350/mo each); the single replica serves analytics only; catalogue survives origin outage for 24 h in stale mode; data use per session ≤ 1.5 MB (NFR-12).

**Negative:** up to 60 s stale price in lists (buyer sees the final price at checkout, with a "price updated" notice); three invalidation paths to test; Cloudflare Pro has no tag-based purge, so URL lists must be maintained by the Catalog module.

**Neutral:** cache keys include language and country (`ad:{country}:{lang}:product:{id}:v{schema}`).

## Reversibility

High. Each layer is controlled by headers and configuration; TTLs are environment variables; removing a layer degrades performance but not correctness, because checkout never trusts a cache. Redis is accessed via the standard protocol and can move to any provider.

## Revisit triggers

- Redis memory > 70% or hit ratio < 80% for a week.
- Catalogue > 200k SKU (move search to a dedicated engine, ADR to follow).
- Upgrade to Cloudflare Enterprise (enables cache-tag purge, TTL can grow to 10 min).

## Assumptions

- AS-1 5,000 SKU × 5 languages × ≈ 6 KB ≈ 150 MB of product payloads — fits 2 GB with search and lists.
- AS-2 Product edits are rare relative to reads (≈ 1:1,000).
