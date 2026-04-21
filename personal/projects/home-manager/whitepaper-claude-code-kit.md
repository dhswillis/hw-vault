---
created: 2026-04-20
updated: 2026-04-20
type: synthesis
tags: [whitepaper, home-manager, claude-code, productize, business, draft]
related:
  - "[[personal/projects/home-manager/index]]"
  - "[[personal/projects/home-manager/services-catalog]]"
  - "[[wiki/summaries/openclaw-install-audit-20260420]]"
  - "[[wiki/summaries/openclaw-vps-setup-guide]]"
status: draft
version: 0.1
---

# Whitepaper — "Home Manager in a Box"

### An opinionated kit for running your household with Claude Code

**Author:** Harrison Willis
**Status:** Draft v0.1 — 2026-04-20
**Audience:** Technically curious high-income households, and operators who want to productize the same thing.

---

## Thesis

Running a household is a coordination problem, not a labor problem. The labor is cheap and plentiful (cleaners, handymen, detailers, nannies). What's expensive is the executive function — deciding who to call, remembering when the filter was last changed, chasing the plumber's quote, reconciling the six invoices at month-end.

[[personal/projects/home-manager/index|Home Manager]] (the human-services company) sells that executive function as a service. This paper describes a second, complementary product: **a kit** that lets a technically-inclined homeowner run the same playbook themselves using Claude Code as the operating layer — without hiring a human Home Manager.

The two are not in conflict. The kit is the DIY tier; the service is the done-for-you tier. Together they cover the full market from "hacker who wants automation" to "family that just wants it handled."

---

## Part 1 — The job to be done

A household has roughly 13 service lines (see [[personal/projects/home-manager/services-catalog]]). Each one generates:

- **State** — the current status of assets (which appliance under warranty, last service date, open issues)
- **Events** — invoices, quotes, appointments, bills, receipts
- **Decisions** — repair vs. replace, which vendor, when to schedule

A homeowner accumulates this in email, text threads, post-it notes, and memory. At scale it breaks. The household has the same operational problem a small business has — just without the tooling.

Historical attempts to solve this:
- **Lifestyle-management firms** (human concierge) — works, but $5k–$15k/mo, designed for UHNW
- **Vertical apps** (TaskRabbit, Thumbtack, Angi, Handy) — fragmented, no memory across jobs
- **Generic PM tools** (Notion, Trello, Asana) — all state, no automation; the homeowner still does the coordination
- **Smart-home platforms** (Home Assistant, SmartThings) — great for devices, useless for the vendor-coordination problem

The gap: **an agent that remembers your house, coordinates with vendors, and produces weekly reports**, at a price point below human concierge.

That agent is now feasible because of a specific stack: Claude Code + local markdown files + MCP tools + cheap messaging channels.

---

## Part 2 — Reference architecture

The kit mirrors the pattern Harrison already runs for his trading business, adapted for the home. See [[wiki/summaries/openclaw-install-audit-20260420]] and [[wiki/summaries/openclaw-vps-setup-guide]] for the existing trading-side implementation — this is the same shape pointed at a different domain.

```
┌────────────────────────────────────────────────────────────┐
│                  YOUR PHONE / DESKTOP                      │
│     Telegram / SMS / iMessage → single chat w/ "Home"      │
└────────────────────────────────────────────────────────────┘
                           ▲  ▼
┌────────────────────────────────────────────────────────────┐
│                   GATEWAY / AGENT HOST                     │
│    (Claude Code or OpenClaw on a $6/mo VPS, or a Mac)      │
│  ┌──────────────┐  ┌─────────────┐  ┌──────────────────┐   │
│  │ Claude API   │  │ Skills (MD) │  │ MCP servers      │   │
│  │ (reasoning)  │  │ • intake    │  │ • gmail          │   │
│  │              │  │ • schedule  │  │ • calendar       │   │
│  │              │  │ • chase     │  │ • sms provider   │   │
│  │              │  │ • report    │  │ • file store     │   │
│  └──────────────┘  └─────────────┘  └──────────────────┘   │
│         ▲              ▲                    ▲              │
└─────────┼──────────────┼────────────────────┼──────────────┘
          │              │                    │
          ▼              ▼                    ▼
┌──────────────────────────────────────────────────────────────┐
│              HOUSEHOLD KNOWLEDGE BASE (markdown)             │
│   property.md  vendors/  repairs/  recurring/  reports/      │
│              (git-versioned, synced nightly)                 │
└──────────────────────────────────────────────────────────────┘
```

**Three rules that make this work**

1. **State lives in markdown, not a database.** Every decision, vendor, appointment, and asset is a markdown file. Claude Code reads files natively. This is the same principle behind the [[wiki/CLAUDE|HW vault]].
2. **One chat, one agent.** The homeowner talks to one thread ("Home"). The agent decides whether to read, schedule, email a vendor, or escalate.
3. **Human-in-the-loop on money and on behalf-of-you actions.** Any spend over a threshold, any outbound message to a vendor, any scheduled appointment — the agent drafts and the human confirms with a thumbs-up. No rogue emails to the plumber.

---

## Part 3 — What ships in the kit

A concrete, opinionated starter pack. Not a framework — a pre-assembled setup that's running the day you buy it.

### 3.1 Install (10 minutes)

```bash
# one-liner gets you:
#  - Claude Code installed
#  - ~/home/ repo cloned with the starter structure
#  - Claude authenticated to your account
#  - MCP servers for gmail + calendar + file store configured
#  - iMessage/Telegram bot paired to your phone

curl -fsSL home-manager-kit.sh | bash
```

### 3.2 Starter knowledge base

```
~/home/
├── CLAUDE.md                # the operating manual (agent reads this first)
├── property.md              # address, year built, sq ft, systems inventory
├── people.md                # household members, pets, emergency contacts
├── vendors/
│   ├── plumber-acme.md      # contact, rate, last job, rating
│   ├── hvac-reliant.md
│   └── ...
├── assets/
│   ├── hvac-upstairs.md     # make/model, install date, filter spec, warranty
│   ├── roof.md              # age, material, last inspection
│   └── ...
├── repairs/
│   ├── open/
│   │   └── 2026-04-12-kids-bath-drywall.md
│   └── done/
├── recurring/
│   ├── cleaning.md          # vendor, cadence, cost, access instructions
│   ├── lawn.md
│   └── ...
├── reports/
│   └── 2026-W16.md          # agent-generated weekly summary
└── inbox/                   # quick capture for triage
```

This is **exactly** the shape of Harrison's HW vault and OpenClaw workspace, rewritten for a home instead of a trading firm. Not a coincidence — it's the pattern that works.

### 3.3 Opinionated skills (pre-installed)

- **intake** — first-run wizard that walks the homeowner through populating `property.md`, `vendors/`, `assets/` from a photo of their electrical panel, a year's worth of email receipts, and 20 minutes of Q&A
- **repair-open** — when the homeowner texts "the kids' bath has a weird ceiling stain", the agent: files `repairs/open/<slug>.md`, cross-references any related asset, identifies the right vendor tier, drafts the vendor email for approval, and schedules a follow-up check in 48h
- **chase** — every weekday morning, the agent scans `repairs/open/` for items whose last-touch is >3 days and drafts a nudge to the relevant vendor
- **schedule** — reads `recurring/` and the household calendar; avoids double-booking (e.g. don't schedule a deep clean the morning the plumber is coming)
- **month-end** — reconciles all invoices, produces one PDF summary, flags anomalies (e.g. lawn bill jumped 40%)
- **weekly-report** — posts a Friday recap to the chat: what got done, what's open, what needs a human decision, spend-to-date

### 3.4 Channels

- **iMessage + Telegram** out of the box
- **Email send/receive** via a dedicated `home@yourname.com` inbox that the agent reads and drafts from
- **Calendar** two-way sync (Google/Apple)

---

## Part 4 — Economics

### DIY kit (end-user)

| Line item | Cost/mo |
|---|---|
| VPS (or run on home Mac mini) | $6–$20 |
| Anthropic API usage (moderate) | $20–$60 |
| Twilio / SMS provider | $5 |
| Email inbox (Google Workspace) | $6 |
| **Total recurring** | **~$40–$90/mo** |
| Kit price (one-time) | $299–$499 |

Compare to: hiring a human Home Manager at $1,500–$3,500/mo, or a luxury lifestyle firm at $5k–$15k/mo.

The DIY tier wins on cost but loses on "I don't want to think about it." It's a real market, but not the whole market.

### Done-for-you tier (Home Manager the service)

| Tier | Price/mo | Includes |
|---|---|---|
| **Lite** | $750 | Kit + 4 coordinated hours/mo + weekly report |
| **Standard** | $1,800 | Kit + 15 hrs/mo + full repair coordination + vendor bench |
| **Premium** | $3,500 | Kit + dedicated point-of-contact + unlimited coordination + monthly in-home walk |

The service tier runs on the same kit, operated by a human Home Manager. The kit is both the product (for DIYers) and the internal operating system (for the service).

This is the crucial insight: **the tool you sell is the same tool you use.** Your cost to serve the top tier goes down over time because the kit gets better.

---

## Part 5 — Go-to-market

### Wedge 1: Kit to hacker-homeowners (proof of demand)
- Distribution: Hacker News, technical Twitter, the r/selfhosted crowd, the Claude Code subreddit
- Price: $299 one-time + $0 support (community forum)
- Goal: 200 units in 90 days to validate product + generate testimonials + seed a vendor-rating dataset

### Wedge 2: Service to Harrison's own network (proof of margin)
- Customer Zero: Harrison's own house (already under way — see [[personal/projects/home-manager/home-repair-backlog]])
- Customer One–Three: friends-and-family at Lite tier
- Goal: prove that a part-time human + the kit can run 5 households at $1,500/mo each ≈ $7,500/mo revenue on <30 hrs/wk of labor

### Wedge 3: White-label to existing operators
- Real-estate brokers who close on a $2M house and want to hand the buyer a year of household ops
- Property managers of short-term rentals who need to coordinate turnover
- Employee-benefit providers at F500 companies (executive perks)

---

## Part 6 — Why this works now (and didn't in 2023)

1. **Claude Code** is the first agent host where the loop of "read a folder of markdown → decide → call a tool → write back" is genuinely production-grade. Earlier agent frameworks (LangChain et al.) couldn't reliably do this without babysitting.
2. **MCP** (Model Context Protocol) standardized tool access. One-line integrations for Gmail, Calendar, filesystem, HTTP APIs — no glue code.
3. **Anthropic pricing** on Sonnet/Haiku tiers makes 50–100 daily household actions economically trivial (<$2/day).
4. **Persistent markdown stores** (the Karpathy LLM-wiki pattern) are a proven pattern for giving agents long-term memory without a vector DB.
5. **OpenClaw / similar gateways** solve the "how do I let an agent live on a server and talk to my phone" problem in <$10/mo. See [[wiki/summaries/openclaw-vps-setup-guide]].

None of these five pieces were mature 18 months ago. All five are mature now.

---

## Part 7 — Risks and unknowns

- **Trust**: agents sending email/SMS on behalf of a homeowner is the blast-radius risk. Mitigation: default to draft-only for all outbound, require human thumbs-up.
- **Vendor adoption**: a plumber isn't going to respond to an AI-drafted email differently than a human-drafted one, but may balk at booking tools. Start with email/SMS only; don't force a portal.
- **Regulatory**: childcare, healthcare, financial-admin adjacencies touch regulated areas. Stay out of those in v1.
- **Model drift**: prompt changes across Claude releases could break skills. Mitigation: version-pin model IDs, version-control the skill files, weekly regression test.
- **"Just Notion"**: the kit must demonstrably do more than a well-organized Notion workspace or it won't justify the price. The agentic-action layer is the differentiator, not the file structure.
- **Churn**: home operations are episodic (busy season around repairs, quiet in between). Monthly billing on a fluctuating workload — consider annual prepay to smooth.

---

## Part 8 — 90-day plan

| Week | Milestone |
|---|---|
| 1–2 | Close Customer Zero repair backlog. Document every email, call, and vendor interaction as a test case for the kit's skills. |
| 3–4 | Build the first draft of the kit: starter repo template, `CLAUDE.md`, 6 skills, install script. |
| 5–6 | Dogfood — Harrison's own household runs fully on the kit for 30 days. Fix everything that breaks. |
| 7–8 | Recruit 3 friends as service-tier customers. Onboard them on the kit, operate them personally. |
| 9–10 | Kit landing page. Pre-order list. |
| 11–12 | Public launch of the kit (DIY tier). Soft-launch service tier to Harrison's network. |

---

## Appendix A — Relationship to existing work

- **OpenClaw install on Harrison's Mac** today: see [[wiki/summaries/openclaw-install-audit-20260420]]. It's configured for the Willis-Holdings trading use case, not home. The Home-Manager kit would be a parallel workspace with its own `IDENTITY.md`, skills, and knowledge base — not a retrofit of the trading one.
- **HW vault** (`~/HW`): the markdown-operating-manual pattern (`CLAUDE.md` + structured folders + `[[wikilinks]]`) is exactly what the kit ships. The kit is, in effect, `HW vault for a house`.
- **Haven Park day-job context** ([[wiki/entities/harrison-haven-park]]): vendor-management-at-scale is the same muscle. Home Manager is the single-asset case of multifamily asset management.

## Appendix B — Open questions

- Sell the kit as a **SaaS** (hosted), as **software** (self-install), or as **docs + consulting** (you implement, we teach)? Three different businesses.
- Does the kit include an **LLM subscription** (like Cursor bundles usage) or does the customer bring their own Anthropic API key? BYO is simpler but a worse funnel.
- Is "Home Manager" the right brand for both the DIY kit and the service? Or does the kit want a tech-forward name (e.g. "`hearth`", "`housekit`", "`the home os`") and the service keep "Home Manager"?
- Partnership with Anthropic for distribution? Claude Code has a growing power-user base; a home-ops kit could be a lighthouse example.
