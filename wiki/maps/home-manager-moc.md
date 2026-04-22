---
created: 2026-04-22
updated: 2026-04-22
type: moc
topic: Home Manager (personal business)
tags: [moc, home-manager, business, personal]
related:
  - wiki/maps/personal-moc.md
  - wiki/maps/root.md
---

# Home Manager — Map of Content

> Personal-business project kicked off 2026-04-20. Two tracks: (1) productized household-concierge service, (2) Claude-Code-powered DIY kit. Customer Zero is Harrison's own house, which is also the kit's reference implementation.

## Orientation

The idea: run a household the way a chief of staff runs an executive's day. Food, cleaning, repairs, kids, cars, fitness, yard — single point of contact, one invoice, weekly report. 13 service lines. Two tiers:

- **Kit** (DIY, $299–$499 one-time + ~$40–$90/mo runtime): for technical homeowners who'll operate it themselves via Claude Code.
- **Service** ($750–$3,500/mo): done-for-you tier, human Home Manager uses the same kit as their operating system.

The two tiers aren't in conflict. The kit is the tool; the service is the operator. Margin on the service tier improves over time because the kit keeps getting better.

## Project hub

- [[personal/projects/home-manager/index]] — **Start here.** Business overview, positioning, open questions, MVP plan.

## Planning & strategy

- [[personal/projects/home-manager/services-catalog]] — Full breakdown of the 13 service lines with scope-in/scope-out for each.
- [[personal/projects/home-manager/whitepaper-claude-code-kit]] — "Home Manager in a Box" whitepaper. Architecture, economics, GTM, risks, 90-day plan.

## Operational

- [[personal/projects/home-manager/home-repair-backlog]] — Customer Zero's 9 open house items, triaged with priorities and suggested sequencing. Doubles as case-study data.
- **Live implementation at `~/home/`** — Git-initialized operating repo (not in the HW vault tree). Contains `CLAUDE.md` agent manual, 9 repair files pre-seeded, 6 skills (intake / repair-open / chase / schedule / weekly-report / month-end), asset stubs, vendor/recurring READMEs, Google Sheets export pipeline.

## Reference architecture

The kit's agent-host pattern is copied from the trading-side OpenClaw install Harrison already runs:

- [[wiki/summaries/openclaw-install-audit-20260420]] — Point-in-time audit of the local OpenClaw setup that serves as reference architecture.
- [[wiki/summaries/openclaw-vps-setup-guide]] — VPS deployment guide (~$120/mo) for the always-on version.

The `~/home/` repo mirrors the `~/HW/` vault's "markdown files + CLAUDE.md operating manual" pattern, just pointed at a house instead of a research brain.

## Stakeholders

- **Harrison Willis** (principal) — Customer Zero, kit author, service-tier operator. Day job: Director of Asset Management at Haven Park (multifamily/MHC). Vendor-management-at-scale mindset is the transferable skill.
- See [[wiki/entities/harrison-haven-park]] for the day-job context.

## Open questions (from project index)

- **Revenue model** — flat retainer vs. retainer + pass-through vs. markup on vendor invoices?
- **Staffing** — W-2 employees vs. broker model (vendor bench) vs. hybrid?
- **Geography** — which metro/zip first?
- **Tech** — client app from day 1 or CRM + Slack + Sheets until volume forces it?
- **Entity** — LLC vs. S-corp? Sit under Haven-Park-adjacent or fully separate?
- **Naming** — "Home Manager" for the service tier; the kit may want a different tech-forward name (`hearth`, `housekit`, `the home os`).

## What's NOT in scope (yet)

- Pricing tiers with unit-economic model
- Vendor bench in target geography
- Customer One (first paying, non-Harrison customer)
- Any paid marketing or public landing page
- Autonomous always-on agent via VPS (Path B from the OpenClaw whitepaper) — currently Path A: user opens Claude Code locally

## Related MOCs

- [[wiki/maps/personal-moc|Personal]] — parent domain map
- [[wiki/maps/root|Root]] — vault top-level

## Log touchpoints

- 2026-04-20 — project-init (services-catalog, index, repair-backlog, whitepaper, openclaw audit)
- 2026-04-20 — project-build (`~/home/` Customer Zero scaffold)
- 2026-04-21 — Google Sheets export pipeline + iCloud mirror
- (This MOC added 2026-04-22 as part of vault organization pass)
