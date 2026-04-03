---
name: occupancy-outlook-builder
version: 1.0.0
description: "Builds date-grain occupancy outlook and sellability aggregates for Mission Control dashboards."
user-invocable: false
metadata:
 openclaw:
  requires:
   bins: [bash]
   env: [DATABASE_URL, LOG_LEVEL]
---
# Occupancy Outlook Builder
## What This Skill Does
This skill computes forward-looking occupancy and sellability aggregates for each property by combining normalized reservations with ARI daily facts. The result is the daily property outlook used in Mission Control charts and operational explanations.

## Inputs
- reservations and reservation_room_stays
- ari_daily_facts
- Property and mapping metadata

## Outputs
- occupancy_outlook_daily rows
- Audit evidence when the computation is partial or uncertain

## Calculation Principles
- One row per property per stay_date.
- reserved_rooms reflects booked room nights for that date.
- total_sellable_rooms reflects ARI-backed sellable inventory when available.
- estimated_occupancy_pct stays within logical bounds unless explicitly marked uncertain.
- avg_daily_rate uses available price facts for the date and property.
- sold_out_flag distinguishes true sold out from structurally unbookable where possible.

## Steps
1. Select the forward horizon, ideally at least 90 days and supporting 365 days.
2. Aggregate booked room nights from normalized reservation stays by property and date.
3. Aggregate sellable inventory and price context from ari_daily_facts.
4. Join the two datasets by property and stay_date.
5. Compute occupancy percentage and ADR.
6. Mark sold_out_flag only when evidence supports it.
7. Upsert occupancy_outlook_daily rows.
8. Log partial-confidence dates where missing ARI data weakens the estimate.

## Guardrails
- Do not invent occupancy where neither reservations nor ARI support it.
- If ARI is stale, preserve last known evidence but let alerts express reduced confidence.
- If mappings are incomplete, prefer conservative property-level aggregates over false precision.

## Collaboration
- hotel-data-normalizer supplies canonical reservation and ARI rows.
- anomaly-detector uses occupancy outlook for sellability and trend rules.
- dashboard-api-server serves the resulting aggregates.
- incident-memory-writer summarizes notable outlook shifts.

## Done Condition
The skill is complete when each active property has a current date-grain occupancy outlook with enough provenance to support dashboard and explanation use.