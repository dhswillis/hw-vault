---
date: 2026-04-20
status: decided
deciders: [harrison]
project: home-manager
tags: [decision, home-manager, positioning, business-model]
links:
  - personal/projects/home-manager/index.md
  - personal/projects/home-manager/services-catalog.md
  - personal/projects/home-manager/home-repair-backlog.md
  - wiki/maps/home-manager-moc.md
  - wiki/entities/harrison-haven-park.md
---

# Home Manager: Retainer Tier + Broker-vs-W2 + Single-Metro Start

## Context

Monday 4-20 kicked off Home Manager — a household-concierge service scaffolded against Harrison's own house as Customer Zero (9 open repair items in `home-repair-backlog.md`). Initial generation produced 13 service lines, a services catalog, a whitepaper for the Claude-Code "kit" version, and a local Customer-Zero implementation (`~/home/` git-init, 6 skills, CLAUDE.md operating manual).

What it did **not** produce: a business model, a staffing model, or a geography. Three open questions surfaced from the generation phase:

1. **Gas-orders scope** — is recurring errand-running (fuel, household supplies) inside or outside the core offer?
2. **Fitness-equipment scope** — is in-home gym maintenance (calibration, belt replacement, lubrication) a service line or a distraction?
3. **Target geography / comparable pricing** — which metro, and priced against what benchmark (property-management companies, lifestyle-concierge firms, executive-assistant services)?

Day-job overlap is non-trivial: Harrison is Director of Asset Management at Haven Park Communities (see [[wiki/entities/harrison-haven-park]]) running vendor coordination across a 30k→60–80k unit manufactured-housing portfolio. Home Manager is structurally the single-asset version of that same vendor-management muscle — generalizing down, not up.

## Options considered

### Option 1: Flat per-visit pricing, national launch, W2 staff

**Pros:**
- Simplest pricing, fastest to communicate to customers.
- W2 gives the most control over quality.
- National positions for scale from day one.

**Cons:**
- Per-visit pricing doesn't capture the actual value (ongoing peace-of-mind), and makes unit economics brutal — every visit has to individually pencil.
- National launch with W2 means hiring before demand is proven, in markets the founder can't personally oversee.
- No moat. Any concierge can do per-visit. Retainer is what defines "manager."

### Option 2: Retainer tier + broker-vs-W2 staffing + single-metro start **(chosen)**

**Pros:**
- Retainer captures recurring value: customer pays monthly for "this house is handled," visits are usage within the tier. Same structure PE uses for property management — Harrison has run this model from the owner side.
- Broker-vs-W2 split lets the business scale variable load (brokered specialists: HVAC, plumbing, gym-equipment tech) while keeping core touchpoint (the "manager" person the customer knows) on W2 or founder-operator. This is the Haven Park pattern at household scale.
- Single-metro start means the founder can walk every house in the first cohort. Customer Zero is already in-metro.

**Cons:**
- Retainer requires real service delivery from day one — can't fake it with a lead-gen wrapper.
- Broker relationships take 60–90 days to establish in a new metro; that's the binding timeline, not marketing.
- Single-metro means the business is trivially copyable by a local competitor for the first year; the moat is customer density + broker network, both time-bound.

### Option 3: Software-first (Claude-Code "kit" from the whitepaper), no physical service

**Pros:**
- Zero operational risk. Pure software margins.
- Already have the scaffolding.

**Cons:**
- Sells the wrong thing. Customers don't want "a kit to manage their home" — they want their home managed. The kit is a reference implementation, not the product.
- Whitepaper stays as an artifact / credibility piece / open-source companion — that's its right role, not the primary offer.

## Decision

**Home Manager is a retainer-tiered household-concierge service with a broker-vs-W2 staffing split, launching in a single metro with Harrison's own house as the first account.**

Open questions get provisional answers for the first cohort:

1. **Gas-orders scope:** **IN**, bundled in retainer — it's the kind of weekly-touchpoint task that keeps the relationship warm and generates incidental intel on the house.
2. **Fitness-equipment scope:** **IN as brokered** — gym calibration/belt/lube is real for the target customer segment (high-income households with in-home gyms), but it's brokered to a specialist, not W2.
3. **Target geography:** single metro (Harrison's). Comparable pricing benchmarked against property-management retainers (0.5–1% of home value per month for luxury single-family) **not** per-visit concierge (which underprices recurring value).

## Why

Three reinforcing reasons:

- **Model match.** Retainer + brokered specialists + direct-operator core is the property-management playbook Harrison runs at Haven Park. Porting a known-working model to a new asset class (single-family luxury) is lower-risk than inventing a business model from the services catalog alone.
- **Founder-density match.** Single-metro means the founder can personally walk every house in the first 10–20 accounts, which is what customers are actually buying in a white-glove retainer — the founder's attention, not an app.
- **Honest scope.** Gas-orders IN and fitness-equipment IN-but-brokered answers the scope questions in the direction of "serve the real high-value customer well" rather than "build the broadest catalog." The services catalog gets trimmed accordingly.

All generation-phase work (13 service lines, whitepaper, Customer-Zero kit) remains in the vault as context and reference. The whitepaper becomes an open-source companion / credibility artifact, not the product.

## Consequences

**Enables:**
- Concrete next steps become obvious: (a) define the retainer tier pricing against comparable property-management benchmarks, (b) identify 3–5 broker relationships needed for launch metro, (c) build customer-interview pipeline — Customer Zero's neighbors and peer group are the first 5 conversations.
- Day-job knowledge transfer is real, not aspirational — vendor-management playbook, contract templates, QA cadence all port.
- Forces a scope cut on the services catalog: anything that can't be either W2-delivered in-metro or reliably brokered gets deferred.

**Closes off:**
- National-from-day-one positioning. If the business ever wants it, it earns it via the second and third metro expansions — explicit gating.
- Pure-software positioning. The Claude-Code kit is an artifact, not a product line.
- Flat per-visit pricing as the headline offer. Ad-hoc visits may still exist as an onboarding product, but they're not the retainer business.

## Review trigger

- **After 5 customer interviews** with target-segment neighbors/peers (target: before end of W19). If <3 of 5 express interest in a retainer at benchmarked pricing, revisit **Option 1** (per-visit pricing) as a possible wedge before returning to retainer.
- **After first 3 broker relationships established** (target: 60–90 days). If broker availability/reliability is worse than day-job experience predicts, revisit W2 scope — some "brokered" service lines may need to move in-house earlier than planned.
- **At 10 paying accounts** OR end of Q3 2026, whichever first: full business-model review — unit economics, retention, NPS. This is the real promotion-to-serious-business checkpoint.
