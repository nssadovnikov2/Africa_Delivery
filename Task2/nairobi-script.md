# Nairobi Meeting — 10-Minute Talk Track

Audience: K. Orlov (investor), M. Lefebvre (CEO), R. Menon (CTO), A. Weber (CFO) · 19 Aug 2026 · One slide: [nairobi-slide.pptx](nairobi-slide.pptx) (preview: [nairobi-slide.png](nairobi-slide.png)) · ≈ 1,400 words, ≈ 130 words/minute.

---

**[0:00 — Opening, the answer first]**

Good morning. I will give you the answer first and then the reasons.

We launch on the fifteenth of October, in Nigeria and Kenya, for four thousand one hundred dollars a month of cloud and software, with the six contractors we have. Sellers get paid the same day the order is delivered. Personal data of Nigerians stays in Nigeria, and of Kenyans in Kenya. And we do it with an architecture that Rajesh and I have written together, which can grow into the platform he described in his letter without a rewrite.

Everything I say is backed by one model. Anna, you already have the spreadsheet: five-year total cost of ownership for three options, people included, not only the cloud bill.

**[0:50 — What the business asked for, and what cannot be true at once]**

Mr Orlov, you gave us four requirements: the date does not move, the system does not fall, it is very cheap, and data stays in the country. Anna gave us a ceiling of five thousand dollars a month and six people. Sofia's campaign promises a million buyers in the first month. Rajesh asked for ten million users from day one.

I took all eight documents we have and looked for places where two requirements cannot both be true. There are eleven. Four of them matter today.

First: ninety-nine point ninety-nine percent, active-active across regions, costs about twenty thousand dollars a month — four times Anna's ceiling.

Second: building our own logistics engine is, by Rajesh's own estimate, six months for four engineers. We have eight weeks and six people for everything.

Third: none of Amazon, Google or Microsoft has a region in Nigeria. So "everything in the hyperscaler cloud" and "Nigerian data in Nigeria" cannot both be literally true.

Fourth: a million buyers in month one, served by a hundred sellers, is ten thousand buyers per seller. Stock runs out on day two, and our whole positioning is trust.

So the job is not to find a perfect architecture. The job is to decide which of these we give up, how much it costs, and when we take it back.

**[2:20 — What we build: left column of the slide]**

On the left of the slide is what we build.

One mobile app, Flutter, for Android and iPhone from one codebase. It keeps the catalogue in a local cache, so a buyer on a cheap phone, paying for every megabyte, sees the first product in under two seconds, even when the network drops. For people without smartphones — Sofia says that is thirty to forty percent of the market — we offer ordering by USSD and SMS. Sellers register and list products through WhatsApp, because that is where they already sell.

Behind that is one core written in Go — Rajesh's language — as a modular monolith: ten modules with strict boundaries, each with its own data, talking through events. It runs on managed Kubernetes in Johannesburg across three data centres, with managed Postgres and Redis. That meets the cloud-native checklist Rajesh brought from KubeCon: containers, Kubernetes, health checks, GitOps, OpenTelemetry. Interestingly, the same checklist says to prefer managed services — so we do.

Payments and delivery we do not build. We connect M-Pesa directly in Kenya, Flutterwave in Nigeria with Paystack as an automatic backup, and Sendbox and Sendy for delivery. They hold the licences — our lawyer says a direct licence by October is unrealistic.

And in orange: small data vaults in Lagos and Nairobi, with local providers. Names, phones and addresses live there. The central system only sees tokens. When we send an SMS or book a courier, the vault does it from inside the country. That is how we answer the minister.

**[4:20 — The numbers: middle column]**

Now the numbers, Anna.

Our option costs four thousand one hundred dollars a month for cloud and software in year one — inside your five thousand. The design in Rajesh's letter costs nineteen thousand seven hundred, needs fourteen engineers instead of six, and an Oracle licence of one hundred and ninety thousand dollars up front. Realistically it goes live in February, not October.

Cost per order: three dollars thirty-two at launch, when volumes are small, falling to one twenty-five at month twelve. For the microservices design: six thirty-six falling to two dollars.

Over five years the part of the cost that architecture actually controls — cloud, licences and the team — is seven point one million for our option and sixteen and a half million for the alternative. Payment and delivery fees are the biggest line for both, about seventy percent, and they are the same either way, so they do not decide this.

You asked what it costs to be wrong. If we build for ten million users and fifty thousand come, we waste about one million dollars in our nine months of runway — including the media we already paid for, if we launch late. If we build small and grow to ten million, we spend about a quarter of a million to split the busiest modules into services. That second mistake is the cheap one, and we would pay it out of growth.

**[6:00 — Availability: the twelve-thousand-dollar minute]**

Mr Orlov, you said a minute of downtime costs twelve thousand dollars. I took that seriously and put it in the model. If it is true today, Rajesh's design wins — the fourth nine pays for itself.

But twelve thousand dollars a minute means six point three billion dollars of sales a year. Our year-one forecast is thirteen million — about twenty-five dollars a minute on average. So for launch we commit to ninety-nine point ninety-five percent for checkout and payments: about four hours a year, with no single point of failure inside one region, automatic fail-over between zones and between payment providers. When a payment provider fails, we switch to the other. When a region fails, we restore in Cape Town within an hour.

And we write down the trigger: when the real cost of a minute crosses fifteen hundred dollars, we move to active-active in two regions. That is not a refusal. That is your requirement, on a date the business can afford.

**[7:15 — What we do not do now, and when: right column]**

The right column is the most important part of the slide. It is what we are deliberately not doing, and when we will.

Ninety-nine ninety-nine: when the minute costs more than fifteen hundred dollars — that is when the extra nine pays for itself.

Microservices, Kafka, Istio: when we pass two million users or fifteen hundred orders a minute. The module boundaries make that an extraction, not a rewrite.

Our own logistics engine: at the twelve-month review, with real delivery volumes and partner performance. Rajesh's experience will matter then.

Final data placement: the lawyers answer between the fifth and nineteenth of September. Each country is a separate deployment unit, so if they say the Nigerian part must move entirely to Lagos, we move it in ten working days.

The blockchain loyalty programme: we evaluate it after month twelve, as you asked, Mr Orlov — studied, not built for October.

On scope: Tom's list has twenty-nine features, all marked top priority. Nine go into the launch, most of them cut to the minimum; the rule was "a buyer cannot complete a purchase, or a seller cannot get paid, without it — or the law requires it". Live shopping goes out on TikTok and Facebook Live with links into our app. And Sofia — we should not send SMS to the four hundred and ten thousand purchased numbers; the lawyer flagged it, and carriers would block our sender name on day one, which also blocks our login codes. We replace it with "dial this code to join" on the radio, which collects consent.

**[8:45 — Decisions needed today]**

So I need four decisions today, one from each of you.

Mr Orlov: ninety-nine point ninety-five percent at launch, with the written trigger for the fourth nine.

Anna: the four-thousand-one-hundred-dollar envelope, with AWS Cape Town as the named alternative for disaster recovery — your rule for anything over five hundred dollars.

Marc: you own the scope — nine features of twenty-nine — until we hire a product manager in December.

Rajesh: co-sign the decision record, and own the triggers that take us to your target architecture.

**[9:30 — Close]**

Launched, affordable, compliant, and ready to grow. If we agree today, the contractors start on Monday, store listings go up on the first of October, and on the sixteenth people buy, sellers get paid, nothing breaks, and nobody calls about Nigerian data in Frankfurt.

Thank you. Questions.

---

## Assumptions

- AS-1 The CTO has reviewed and co-signed the ADR-001 draft before the meeting (A5 asks for it).
- AS-2 Figures are from [tco-model.xlsx](tco-model.xlsx) and [options-comparison.md](options-comparison.md).
