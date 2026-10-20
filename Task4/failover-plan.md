# High Availability and Failover Plan

Date: 2026-08-27 · Traces to: NFR-09/10 ([Task3](../Task3/nfr-registry.md)), QAS-01/05, FMEA 06/12, [ADR-003](adr-003-data-residency.md), [deployment-diagram.png](deployment-diagram.png).

## High availability within the primary region (automatic)

| Layer | Mechanism | Failure absorbed | RTO / RPO |
|---|---|---|---|
| Edge | Cloudflare anycast; `stale-if-error` 24 h for public catalogue | Origin outage for browse | 0 for cached browse |
| Compute | GKE regional cluster, ≥ 3 `api` replicas spread across 3 zones (topology spread constraints), PodDisruptionBudget, HPA | Pod, node, zone loss | ≤ 2 min / — |
| Database | Cloud SQL HA: synchronous standby in a second zone, automatic failover | Primary or zone loss | ≤ 2 min / 0 |
| Cache | Memorystore Standard (replica in second zone) | Cache node loss; on full loss → DB read-through (ADR-002) | ≤ 1 min / n.a. |
| Vaults (Lagos, Nairobi) | 2 hosts in separate racks, synchronous standby, keepalived VIP | Host loss | ≤ 5 min / 0 |
| Partners | Circuit breakers, PSP failover Flutterwave ↔ Paystack, 2 SMS aggregators | Partner outage | ≤ 30 s switch |

## Scenarios that need a human decision

### S1 — Loss of GCP africa-south1 (or GCP account suspended)

- **Trigger:** region-wide errors > 30 min, or provider status "major outage", or account/billing suspension (note: invoice terms of 45 days, A6 p.4 — the CFO is alerted on any payment reminder).
- **Who decides:** Incident Commander (on-call SRE lead) proposes; **CTO or deputy CTO (Solution Architect) approves** within 15 min; CEO and investor informed by the IC with a one-line status every 30 min. CFO approves the temporary cost increase (AWS scaled up ≈ $150/day) after the fact.
- **Data loss:** RPO ≤ 5 min (logical replication lag, alert if > 60 s). Orders in flight during the lag are re-created from PSP callbacks and the daily reconciliation.

| Step | Action | Who | t (min) |
|---|---|---|---|
| 1 | Burn-rate page; IC confirms region-wide impact; status page "degraded" (catalogue still served from CDN) | On-call | 0–10 |
| 2 | Declare DR; freeze deploys; notify CTO, CEO | IC | 10–20 |
| 3 | `terraform apply -var dr_active=true` — EKS node group 1 → 6 nodes; Redis (ElastiCache) created | SRE | 20–30 |
| 4 | Stop replication, promote RDS replica to primary; run consistency check (last LSN, outbox counts) | SRE | 25–35 |
| 5 | ArgoCD syncs the `dr-aws` overlay (same images and manifests, different endpoints) | SRE | 30–40 |
| 6 | Switch Cloudflare origin pool to AWS load balancer; vault VPN tunnels re-pointed (pre-provisioned) | SRE | 40–45 |
| 7 | Smoke tests: browse, OTP, real M-Pesa payment of KES 10, courier quote | QA on-call | 45–55 |
| 8 | Status "operational"; reconcile PSP callbacks received during the switch | IC, Payments lead | 55–60 |

**RTO ≤ 60 min.** Failback: after 24 h of GCP stability, reverse replication RDS → Cloud SQL, planned switch in a low-traffic window (02:00 WAT), same steps reversed.

### S2 — Loss of the Lagos vault site (local provider data-centre outage)

- **Principle: compliance over availability.** Nigerian personal data is **never** failed over abroad, not even temporarily (ADR-003). We accept degraded service in Nigeria instead.
- **Degraded mode (automatic):** browse, cart and checkout for already-registered NG buyers continue (tokens are cached in the core); new registrations, SMS and courier bookings are queued in `worker` (bulkhead) for up to 24 h; app shows "confirmation SMS delayed".
- **Who decides:** IC with the **DPO** and CTO; investor informed (his red line, A1).
- **Steps:** (1) confirm site loss ≤ 15 min; (2) if > 2 h expected — restore vault from the latest in-country backup (Abuja site, ≤ 15 min old) onto pre-provisioned VMs at the backup site; (3) re-point VPN; (4) drain queues in order (OTP first, then courier bookings, then status SMS).
- **RTO ≤ 4 h, RPO ≤ 15 min** for vault data; 0 loss of orders (they live in the core).

### S3 — Loss of the Nairobi vault site

Same as S2 with Nairobi DC2 as the backup site; M-Pesa payment references are re-read from Safaricom statements if the last 15 minutes are lost.

### S4 — Cloudflare outage

IC switches DNS (low-TTL record at a secondary DNS provider) directly to the GCP load balancer; WAF rules replicated as Cloud Armor policy (pre-tested). RTO ≤ 15 min, RPO 0. The app keeps working offline for browsing.

### S5 — Regulator orders data moved or a cell isolated (e.g. counsel's opinion)

Not an outage but planned failover of a country cell: execute the ADR-003 reversibility runbook (≤ 10 working days), decision by CTO + counsel, approved by the investor.

## Drills and readiness

| Drill | Frequency | Pass criterion |
|---|---|---|
| Zone drain (GKE, Cloud SQL failover) | Monthly | QAS-01: ≤ 2 min, 0 lost orders |
| Full region DR to AWS (S1) | Before launch (2026-10-05), then quarterly | QAS-05: RTO ≤ 60 min, RPO ≤ 5 min |
| Vault host loss / site restore (S2, S3) | Before launch, then quarterly | ≤ 5 min host; ≤ 4 h site |
| PSP and SMS failover | Before launch, then monthly | FMEA 02, 03, 09 actions verified |

## Assumptions

- AS-1 Logical replication from Cloud SQL to RDS keeps lag < 60 s at launch volumes.
- AS-2 The pilot-light AWS account has invoice billing approved by 2026-09-15 (A6 p.4).
- AS-3 Local providers in Lagos/Nairobi offer a second site within the same country for backups.
