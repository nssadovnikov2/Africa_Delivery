# Team Structure, Location and 24/7 On-Call

Date: 2026-08-28 · Constraints: 6 contractors until December, hiring freeze (A6 p.5); nobody from the local market on staff (case); 99.95% SLO needs 24/7 response (Task3).

## Structure (MVP)

```
CTO (R. Menon) — engineering head, Payments & Fulfilment design review, logistics domain
└── Deputy CTO / Solution Architect — architecture, ADRs, PII vaults, Identity & Consent
    ├── Stream "Buy" (checkout → delivery)
    │   ├── C1 Go backend — Checkout, Orders, Payments, Payouts, Ledger          (UTC+2)
    │   ├── C3 Go integrations — USSD/SMS/WhatsApp channels, couriers, notifications (UTC+5:30)
    │   └── C4 Flutter mobile — buyer app, offline cache, localisation           (UTC-3)
    ├── Stream "Sell" (onboard → paid)
    │   ├── C2 Go backend — Catalogue, Search, Seller onboarding                 (UTC+5:30)
    │   └── C5 Full-stack — back office, CEO dashboard, test automation          (UTC+2)
    └── Platform
        └── C6 SRE — GKE, IaC, CI/CD, observability, DR                          (UTC-3)
CEO (M. Lefebvre) — interim product owner; ops team (KYC, moderation, support) hired outside the engineering freeze
```

Two stream-aligned pairs plus a platform engineer match the two value streams ([value-stream.png](value-stream.png)); each module has exactly one owner ([services-per-capability.md](services-per-capability.md)).

## Nairobi office or remote?

| | Office in Nairobi | Remote-first (chosen) |
|---|---|---|
| Hiring | Freeze: would mean relocating contractors | Keeps the 6 contractors we have |
| Cost | Office + relocation ≈ $8–10k/month | ≈ $600/month: 2 co-working desks (Lagos, Nairobi) |
| 24/7 coverage | One time zone → night shifts | Three time zones → follow-the-sun without nights |
| Market knowledge | Daily contact with sellers, partners, real 3G | Covered by a **field presence**: CEO, SA and a rotating contractor in Lagos/Nairobi; device lab of five < $150 Android phones on local SIMs |

**Decision:** remote-first engineering with a small Nairobi hub (CEO, CTO, SA) and a launch war-room in Lagos and Nairobi (two contractors on site 10–29 October). Revisit in December when the freeze lifts: the first hires (PM, local ops and QA) are made **in Lagos and Nairobi** to close the "no local staff" gap.

## 24/7 on-call with six contractors

| UTC window | Local time of on-call | Africa local time (WAT / EAT) | Primary / secondary |
|---|---|---|---|
| 00:00–08:00 | India 05:30–13:30 (C2, C3) | 01–09 / 03–11 | C2 / C3, swap weekly |
| 08:00–16:00 | Eastern Europe 10:00–18:00 (C1, C5) | 09–17 / 11–19 | C1 / C5 |
| 16:00–24:00 | Brazil 13:00–21:00 (C4, C6) | 17–01 / 19–03 | C6 / C4 |

- Everyone is on call only in their own working hours — no night shifts; stipend $1.5k/month in total (TCO).
- Pages only on SLO burn-rate and P1 partner alerts (Task3 observability); each alert links to a runbook.
- Escalation: on-call → module owner → SA (deputy CTO) → CTO; region DR decision by CTO/SA (Task4 failover plan).
- Launch fortnight (12–29 Oct): double coverage, war-room channel with CEO and partner contacts.

## Assumptions

- AS-1 Contractors are already distributed across three time zones; if not, the rota is rebuilt with a paid night stipend.
- AS-2 Ops staff (KYC, moderation, support) are not engineers and are budgeted by the CEO outside the IT TCO.
