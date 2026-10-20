# Task 4 — Regulatory Constraints and Data Strategy

| File | Content |
|---|---|
| [data-residency-map.md](data-residency-map.md) | Known vs assumed by country; 12 data categories: subject country, where stored, where processed, legal basis, technical measure (course template structure) |
| [cloud-regions-comparison.md](cloud-regions-comparison.md) | GCP africa-south1, AWS af-south-1, Azure South Africa North, local Nigerian provider: services, latency, price, SLA, verdict |
| [adr-003-data-residency.md](adr-003-data-residency.md) | ADR-003: country PII vaults + pseudonymised core; masking, keys, replication, retention; **Reversibility** section |
| [c4-container.puml](c4-container.puml) / [.png](c4-container.png) | C4 Container diagram grouped by deployment region |
| [deployment-diagram.puml](deployment-diagram.puml) / [.png](deployment-diagram.png) | Deployment diagram (PlantUML) |
| [failover-plan.md](failover-plan.md) | HA mechanisms, region/vault/CDN failover scenarios: who decides, steps, time, data loss |

**Conservative assumption:** no clear-text personal data of Nigerian or Kenyan subjects leaves its country from day 1; only pseudonymised records are processed in South Africa. **Reversibility:** if counsel rules otherwise, a country becomes a full local cell in ≤ 10 working days by configuration, not by rewrite.
