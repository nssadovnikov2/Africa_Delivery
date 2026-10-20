# Quality Attribute Scenarios

Date: 2026-08-24 · Template: [templates/qas-template.md](../templates/qas-template.md) (6 fields) · Each scenario is a test case for the launch-readiness review on 2026-10-08.

## QAS-01 Availability — zone failure during checkout

| Field | Value |
|---|---|
| Source | Cloud provider infrastructure |
| Stimulus | One of three zones in GCP africa-south1 becomes unavailable |
| Artifact | `api` deployment, Cloud SQL primary, Memorystore |
| Environment | Normal operation, 300 orders/hour |
| Response | Kubernetes reschedules pods to the remaining zones; Cloud SQL fails over to the standby; in-flight payment intents resume from the outbox |
| Response measure | Checkout unavailable ≤ 2 min; 0 committed orders lost (RPO 0); no double charge (NFR-09, NFR-11) |

## QAS-02 Availability — payment provider outage

| Field | Value |
|---|---|
| Source | Flutterwave (Nigeria) |
| Stimulus | 50% of charge requests time out for 10 minutes |
| Artifact | Payments module, PSP adapter |
| Environment | Evening peak, launch week |
| Response | Circuit breaker opens after 20 failures in 30 s; new card/transfer intents route to Paystack; pending intents are polled with retry + exponential backoff; buyer sees "use another method" instead of an error |
| Response measure | Payment API availability ≥ 99.95% over the 28-day window; breaker opens ≤ 30 s; payment success rate back to ≥ 95% within 2 min |

## QAS-03 Latency P99 — browsing on 3G at peak

| Field | Value |
|---|---|
| Source | 20,000 concurrent buyers after a radio spot |
| Stimulus | Category and product-detail requests, 10k rps at the edge |
| Artifact | Cloudflare CDN, `api` catalogue endpoints, Redis |
| Environment | Peak, 3G clients (300 ms RTT, 1 Mbit/s) |
| Response | 90% of requests served by CDN/Redis (ADR-002); origin scales via HPA |
| Response measure | Origin p99 ≤ 300 ms; client first product ≤ 2 s p75 from cache (NFR-05, NFR-06) |

## QAS-04 Throughput — launch-day burst

| Field | Value |
|---|---|
| Source | Marketing campaign start, 15 Oct 08:00 WAT/EAT |
| Stimulus | Traffic jumps 10× in 5 minutes: 3,000 registrations/min, 300 orders/min |
| Artifact | Whole platform incl. OTP via SMS |
| Environment | Launch day |
| Response | HPA adds `api` replicas (3 → 15); OTP requests are queued and rate-limited per number; non-critical jobs (analytics export, recommendations) are shed |
| Response measure | 300 orders/min sustained for 30 min with checkout p99 ≤ 800 ms and error rate < 0.5% (NFR-07) |

## QAS-05 Recoverability (RTO/RPO) — loss of the primary region

| Field | Value |
|---|---|
| Source | Provider-wide incident or regulatory order |
| Stimulus | GCP africa-south1 unavailable for > 30 min |
| Artifact | Entire core platform |
| Environment | Any time; pilot light in AWS af-south-1 |
| Response | Incident commander declares DR; Terraform scales the pilot-light EKS cluster; logical replica in RDS is promoted; Cloudflare switches origin; PII vaults (Lagos, Nairobi) are unaffected ([Task4/failover-plan.md](../Task4/failover-plan.md)) |
| Response measure | RTO ≤ 60 min, RPO ≤ 5 min; catalogue keeps serving from CDN stale cache during the switch (NFR-10) |

## QAS-06 Availability — SMS provider or sender-ID block

| Field | Value |
|---|---|
| Source | Nigerian carrier filtering |
| Stimulus | Delivery reports for OTP drop below 80% |
| Artifact | Notifications module |
| Environment | Normal operation |
| Response | Bulkhead keeps OTP queue separate from bulk status messages; router switches to the second aggregator; offers WhatsApp OTP to app users |
| Response measure | OTP ≤ 30 s for 99% restored within 10 min; transactional delivery SLO 99.9% kept |

## QAS-07 Deployability (cloud native) — release during business hours

| Field | Value |
|---|---|
| Source | Developer merges to `main` |
| Stimulus | New monolith version with a defect in checkout |
| Artifact | CI/CD (GitHub Actions → ArgoCD → Argo Rollouts) |
| Environment | Peak hours |
| Response | Canary 5% → 25% → 100%; analysis on payment SLI; automatic rollback on SLO burn |
| Response measure | Rollback ≤ 5 min, < 1% of checkouts affected, zero manual kubectl changes (NFR-20, CN.ADL.05/06) |

## Assumptions

- AS-1 Load profile from Task1 canvas AS-1; burst factor 10× from radio campaigns (A2).
- AS-2 Pilot-light DR with logical replication from Cloud SQL to RDS is in place by 2026-10-05 (Task5 roadmap).
