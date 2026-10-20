# Cloud Regions Comparison

Date: 2026-08-27 · Inputs: case (no hyperscaler region in Nigeria; host on Amazon, Google, Microsoft or on-prem), A6 (≤ $5k/mo, no commitments, invoice billing), [ADR-001](../Task2/adr-001-architecture-style.md) footprint, [data-residency-map.md](data-residency-map.md).

## Comparison

| Provider / region | Services we need (managed K8s, Postgres HA, Redis, object storage, KMS, queue) | Latency, RTT ms (Lagos / Nairobi / Johannesburg) | Price — ADR-001 core footprint, $/mo | SLA (relevant) | Commercial fit (A6) | Residency fit | Verdict |
|---|---|---|---|---|---|---|---|
| **GCP africa-south1** (Johannesburg, 3 zones, since 2024) | All: GKE (regional), Cloud SQL PostgreSQL HA, Memorystore, GCS, Cloud KMS, Pub/Sub | 95–130 / **45–65** / 2–5 | **≈ $3,400** (sustained-use discounts apply automatically, no commitment) | GKE regional control plane 99.95%; Cloud SQL HA 99.95%; multi-zone VMs 99.99% | Invoiced billing on application; resellers available | ZA ✅; NG/KE clear PII ❌ | **Primary** for the core |
| AWS af-south-1 (Cape Town, 3 AZ, since 2020) | All: EKS, RDS Multi-AZ, ElastiCache, S3, KMS, SQS; opt-in region | 110–140 / 65–90 / 18–25 | ≈ $3,800 (+12%; discounts need Savings Plans = commitment, forbidden by A6 p.3) | EKS 99.95%; RDS Multi-AZ 99.95%; EC2 multi-AZ 99.99% | Invoicing after credit check; resellers | ZA ✅; NG/KE clear PII ❌ | **DR pilot light** (≈ $250/mo); 2nd provider and 2nd city |
| Azure South Africa North (Johannesburg, 3 AZ; paired with SA West, Cape Town) | All: AKS, PostgreSQL Flexible (zone-redundant HA), Azure Cache for Redis, Blob, Key Vault, Service Bus | 100–135 / 50–70 / 2–5 | ≈ $3,600 (AKS Standard tier for SLA +$73) | AKS (Standard tier) 99.95%; PG Flexible zone-redundant 99.99%; VMs across AZ 99.99% | CSP partner invoice — easiest for 45-day terms | ZA ✅; NG/KE clear PII ❌ | **Named alternative** (CFO rule A6 p.2); only provider with 2 in-country regions — the path to 99.99% inside ZA |
| Local Nigerian provider (Lagos; e.g. Layer3, Nobus, Kasi) | VMs, block/object storage, colocation; managed K8s/DB immature or absent → self-operated | **5–15** / 120–180 (often via Europe) / 95–130 | Vault only: ≈ $350; full core cell: ≈ $2,500 + 0.5 SRE (≈ $3,000) | Typically 99.9% service SLA; Tier III facility | Local invoice in NGN; no card needed | NG ✅ (only compliant option for NG clear PII) | **NG PII vault**; full NG cell only if counsel requires (ADR-003) |
| *Reference: AWS eu-central-1 Frankfurt* | All | 90–110 / 110–140 / 150–170 | ≈ $2,900 | — | — | ❌ the "minister call" scenario (A1) | Rejected |

Kenya: no hyperscaler region either; a Nairobi local provider (≈ $350/mo) hosts the KE vault, same pattern as Lagos.

## How the numbers were obtained

- **Latency:** indicative ranges from public RTT measurements (submarine routes WACS/Equiano/2Africa on the west coast, EASSy/SEACOM on the east coast). Week-1 action: blackbox probes on two $10/mo VPS in Lagos and Nairobi measure all three regions on 2026-08-20…27; the region choice is confirmed on 2026-08-28, before the GKE cluster is created on 2026-08-31 (probes later move to the vault VMs).
- **Price:** the same bill of materials as TCO Block 1 (3 × 4 vCPU/16 GB nodes, Postgres HA 4 vCPU/16 GB + replica, 2 GB HA Redis, 200 GB object storage, 1 TB egress behind Cloudflare) at on-demand list prices, Aug 2026, ±15%.
- **SLA:** providers' public SLA pages for the listed services.

## Conclusion

GCP africa-south1 is chosen as the primary region: full managed stack (CN.MSV.01), best latency to Nairobi and comparable to Lagos, the lowest on-demand price, and automatic sustained-use discounts without the commitments the CFO forbids (A6 p.3). AWS af-south-1 takes the pilot-light DR role: a different provider and a different city in the same country, so a provider-wide failure or account suspension does not stop us, and POPIA is respected. Azure South Africa North is the named alternative for the CFO and the path to an in-country active-active pair later. No hyperscaler region is acceptable for clear-text Nigerian or Kenyan personal data — local providers host the vaults.

## Assumptions

- AS-1 Region service availability as published by providers in Aug 2026; re-verified in the week-1 spike.
- AS-2 Cloudflare absorbs ≥ 85% of egress, so egress-price differences between providers are secondary.
- AS-3 Invoiced billing approval takes ≤ 3 weeks (A6 p.4); started on 2026-08-20 in parallel for GCP and AWS.
