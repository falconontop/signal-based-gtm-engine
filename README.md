# Signal-Based GTM Engine — Designed & Implemented

A system designed to replace manual outbound with signal-driven, AI-assisted pipeline generation.

Built for B2B SaaS teams (10–200 employees) that need predictable pipeline without scaling headcount. The agent handles the intelligence-heavy work — scoring, research, personalization — while purpose-built tools handle sending, CRM, and enrichment at scale.

→ **[View interactive architecture](./docs/gtm-agent-architecture.html](https://falconontop.github.io/signal-based-gtm-engine/))** · **[One-page system overview (PDF)](./docs/gtm-engine-onepager.pdf)**

---

## My role in this system

- Designed full system architecture and tool boundary decisions
- Built scoring + filtering logic (Fit + Intent model)
- Implemented AI-assisted signal research layer
- Engineered sequence generation prompts (few-shot, structured output)
- Iterated scoring weights based on reply and conversion data

---

## What it does

Takes a raw Apollo export and returns two campaign-ready files:

- A scored, filtered lead list with a signal brief per qualified lead
- A 4-touch email sequence per lead, grounded in actual company signals
- HubSpot import CSV (field-mapped, lifecycle stage set)
- Instantly campaign CSV (sequences pre-loaded, custom vars mapped)

Human review takes ~15 minutes for a 200-lead batch. Then import and launch.

---

## System flow

```
Apollo export (CSV)
      ↓
[01] Lead intake       — parse, validate, deduplicate
      ↓
[02] Signal detection  — job boards, LinkedIn, funding (agent-assisted)
[02b] Contact enrich   — verified emails, phones (Clay, external)
      ↓
[03] Scoring engine    — Fit score + Intent score → filter to qualified
      ↓
[04] Signal brief      — structured per-lead context (JSON)
      ↓
[05] Sequence gen      — 4-touch email copy, AI-assisted, signal-grounded
      ↓
[06] Export formatter  → HubSpot CSV + Instantly CSV
      ↓
Human review (15 min) → import → launch
```

Optional: wrap in n8n for daily scheduled runs — trigger → agent → exports → Slack notification.

---

## Realistic example

**Target:** Mid-size B2B SaaS, 50–150 employees, DACH region  
**Signal:** Hiring 2 SDRs + Series A announced 6 weeks prior  
**Hook:** "Scaling your outbound team — without a playbook yet?"

**Observed output:**
- Reply rate well above cold email baseline
- Faster SQL conversion — context already established at first touch
- Reps spending time on warm leads, not manual research

---

## Scoring logic

**Fit score — threshold: ≥60**

| Criteria | Points |
|---|---|
| Company size 10–200 employees | +20 |
| Industry: SaaS / Tech | +20 |
| Hiring for sales roles | +15 |
| Existing CRM detected (HubSpot / Salesforce) | +15 |
| Subscription / high ACV revenue model | +10 |

**Intent score — threshold: ≥40**

| Signal | Points |
|---|---|
| Hiring signal (SDR / AE / RevOps / GTM) | +30 |
| Recent funding or expansion news | +25 |
| Pricing / demo page activity | +20 |
| Tech stack change detected | +15 |

Both thresholds must pass → lead is qualified → enters sequence generation.

---

## Sequence structure

| Touch | Day | Angle |
|---|---|---|
| Email 1 | Day 1 | Signal hook — references specific, verifiable company signal |
| Email 2 | Day 3 | Value add — relevant insight for their segment |
| Email 3 | Day 6 | Case angle — proof matched to ICP situation |
| Email 4 | Day 10 | Break-up — concise, no hard sell |

Personalization is grounded in the signal brief, not injected ad-hoc. Quality is bounded by the system prompt: few-shot examples from real winning sequences, explicit anti-patterns, word count per touch enforced.

---

## Stack

| Layer | Tool | Role |
|---|---|---|
| Agent runtime | Claude Code (CLI) | Orchestration, scoring, AI-assisted generation |
| Enrichment | Clay | Verified contacts at scale — waterfall across 50+ sources |
| Email sending | Instantly / Smartlead | Deliverability, domain warm-up, bounce handling |
| CRM | HubSpot | Pipeline stages, lifecycle tracking, deal routing |
| Automation | n8n | Optional scheduled workflow wrapper |
| Lead sourcing | Apollo | Raw export input |

---

## When NOT to use this system

- **Low-ticket, high-volume B2C** — signal research cost outweighs value per lead
- **Markets without digital signals** — no job posts, no funding data, no traceable intent
- **Undefined ICP** — scoring logic needs a clear target to score against
- **No human review step** — AI-assisted generation still needs a final check before sending

---

## Expected impact

| Metric | Direction | Note |
|---|---|---|
| Reply rate | ↑ significantly | Relevance drives replies, not volume |
| Unqualified leads entering pipeline | ↓ ~60% | Scoring filters noise before it hits a sender |
| Manual research time per lead | ↓ ~80% | Signal detection + brief generation is automated |
| Pipeline predictability | ↑ | Systematic process replaces random outbound |

Estimates based on observed patterns. Your numbers will vary based on ICP clarity, prompt quality, and domain health.

---

## Where this breaks (honest)

**Signal noise early on** — Job board data is laggy. First 2–3 runs need manual brief validation before fully trusting automation. Calibrate signal weights on real reply data, not assumptions.

**Deliverability is not the agent's job** — Great copy doesn't matter on a cold domain. DMARC, SPF, warm-up are prerequisites. The agent hands off to Instantly; domain setup is on you.

**Clay cost at scale** — Waterfall enrichment costs rise linearly. Under 100 leads/week, agent-only research is viable. Above that, Clay ROI is clearly positive — but it's a real line item.

**AI-assisted copy still needs human review** — Hallucinated context, wrong seniority, tone mismatch — rare but real. The 15-minute review at export isn't optional. That's the actual cost of running this responsibly.

---

## Project structure

```
gtm-agent/
├── README.md
├── agent/
│   ├── main.py              # Entry point — orchestrates full pipeline
│   ├── intake.py            # Lead parsing + validation
│   ├── signals.py           # Web research + signal detection
│   ├── scoring.py           # Fit + Intent scoring logic
│   ├── brief.py             # Signal brief generator
│   ├── sequence.py          # Email sequence generation (Claude API)
│   └── export.py            # HubSpot + Instantly CSV formatter
├── prompts/
│   ├── system_sequence.md   # System prompt for email generation
│   └── system_brief.md      # System prompt for brief generation
├── data/
│   ├── input/               # Drop Apollo exports here
│   └── output/              # Scored leads + campaign CSVs
├── docs/
│   ├── gtm-agent-architecture.html
│   └── gtm-engine-onepager.pdf
└── examples/
    └── sample_output.json   # Example brief + sequence (fictional data)
```

---

## Status

Core scoring logic, signal detection, and export formatting implemented. Sequence generation prompt in active iteration — calibrating on reply data.

---

*Built on real implementation experience across B2B and D2C GTM systems.*
