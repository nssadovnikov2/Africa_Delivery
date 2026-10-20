# Africa Delivery — Solution Architecture

Project work for the Solution Architect course: a multi-category marketplace launching in Nigeria and Kenya on **15 October 2026**, then South Africa, 5 countries in 12 months. Starting point: no team, no documentation, eight conflicting artifacts, $5k/month, six contractors.

## The decision in one paragraph

A **Go modular monolith** on managed Kubernetes in **GCP Johannesburg** (3 zones), with **partner** payments (M-Pesa direct, Flutterwave, Paystack) and logistics (Sendbox, Sendy), **in-country PII vaults** in Lagos and Nairobi, and a pilot-light DR in **AWS Cape Town**. It launches on 15 October for **$4,100/month** with the six contractors we have, targets **99.95%** for checkout, and has written triggers for everything deferred: 99.99% when downtime costs > $1.5k/min, service extraction at > 2M users, own logistics at the month-12 review, final data placement after counsel's opinion.

## Navigation

| Task | Directory | Key files |
|---|---|---|
| 1. Stakeholders & Architecture Canvas | [Task1/](Task1/README.md) | [stakeholder matrix](Task1/stakeholder-matrix.md) · [canvas](Task1/architecture-canvas.md) · [contradictions](Task1/contradictions.md) · [open questions](Task1/open-questions.md) |
| 2. Architecture Vision & style (ADR-001) | [Task2/](Task2/README.md) | [options comparison](Task2/options-comparison.md) · [TCO model](Task2/tco-model.xlsx) · [ADR-001](Task2/adr-001-architecture-style.md) · [C4 Context](Task2/c4-context.png) · [use cases](Task2/use-cases.png) · [vision](Task2/architecture-vision.md) · [slide](Task2/nairobi-slide.pptx) · [script](Task2/nairobi-script.md) |
| 3. NFRs & availability (ADR-002) | [Task3/](Task3/README.md) | [NFR registry & cost of nines](Task3/nfr-registry.md) · [SLOs](Task3/slo/) · [QAS](Task3/quality-attribute-scenarios.md) · [FMEA](Task3/fmea.md) · [ADR-002](Task3/adr-002-caching-strategy.md) · [observability](Task3/observability.md) |
| 4. Regulation & data (ADR-003) | [Task4/](Task4/README.md) | [data residency map](Task4/data-residency-map.md) · [regions](Task4/cloud-regions-comparison.md) · [ADR-003](Task4/adr-003-data-residency.md) · [C4 Container](Task4/c4-container.png) · [deployment](Task4/deployment-diagram.png) · [failover plan](Task4/failover-plan.md) |
| 5. Roadmap & Build/Buy/Partner (ADR-004) | [Task5/](Task5/README.md) | [capability map](Task5/capability-map.md) · [value stream](Task5/value-stream.png) · [services](Task5/services-per-capability.md) · [roadmap](Task5/roadmap.md) · [ADR-004](Task5/adr-004-payment-provider.md) · [transition](Task5/transition-architecture.md) · [risks](Task5/risk-register.md) · [team](Task5/team-structure.md) |
| Templates | [templates/](templates/README.md) | [ADR template](templates/adr-template.md) · [QAS template](templates/qas-template.md) |

**ADRs (one template, continuous numbering):** ADR-001 architecture style → ADR-002 caching → ADR-003 data residency → ADR-004 payment providers.

## Sources

Every decision cites the input it comes from. Codes used throughout:

| Code | Artifact |
|---|---|
| A1 | Transcript of the meeting with the investor (10 Aug 2026) |
| A2 | Marketing brief (S. Almeida, 6 Aug 2026) |
| A3 | Legal letter (Nwosu & Partners, 8 Aug 2026), p.1–5 = numbered paragraphs |
| A4 | Product Manager's list of 29 features (T. Bradley, 3 Aug 2026), #N = feature number |
| A5 | CTO letter (R. Menon, 11 Aug 2026) incl. attached CN checklist (CN.xxx.NN) |
| A6 | CFO letter (A. Weber, 11 Aug 2026), p.1–7 = numbered points |
| A7 | Investor's note, l.N = line |
| A8 | System sketch (one-slide diagram) |
| case | Course case description (company, geography, MVP goals) |
| C-NN | Contradiction ID in [Task1/contradictions.md](Task1/contradictions.md) |

## Conventions

Markdown for text; diagrams as code (PlantUML, incl. C4-PlantUML) with the source `.puml` and the rendered `.png` side by side; kebab-case file names; explicit `## Assumptions` in every document that uses them. Diagrams are rendered with PlantUML 1.2025.4.
