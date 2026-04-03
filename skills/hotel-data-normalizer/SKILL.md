---
name: hotel-data-normalizer
version: 1.0.0
description: "Normalizes Booking.com reservation and ARI payloads into canonical SQLite records."
user-invocable: false
metadata:
 openclaw:
  requires:
   bins: [bash]
   env: [DATABASE_URL, BOOKING_PRICING_TYPE_DEFAULT, LOG_LEVEL]
---
# Hotel Data Normalizer
## What This Skill Does
This skill translates Booking.com source payloads into the canonical data model used by Jane Hotel Manager. It handles both reservation payloads and ARI payloads so the rest of the system can reason over one operational truth model.

## Scope
For reservations, the skill creates or updates normalized reservation rows, reservation room stays, and reservation events while preserving linkage back to the raw message. For ARI, it produces date-grain ari_daily_facts rows with availability, restriction, and pricing attributes that support dashboard queries and anomaly rules.

## Inputs
- reservation_messages awaiting normalization
- Raw ARI responses or staged ARI payloads
- Current property, room, and rate mappings when available
- Pricing-type defaults and property metadata

## Outputs
- properties, room_types, and rate_plans upserts
- reservations
- reservation_room_stays
- reservation_events
- ari_daily_facts
- audit evidence for quarantined or rejected records

## Core Principles
- Preserve source identity and timestamps.
- Never discard raw linkage.
- Use canonical status values for dashboard consistency.
- Validate dates, occupancy, currency, and price fields before insert.
- Support standard, derived, OBP, and LOS pricing structures.
- Prefer explicit source fields over inference.

## Reservation Steps
1. Read staged reservation_messages that are not yet normalized.
2. Parse OTA fields into canonical reservation identity, dates, room and rate assignments, guest counts, amounts, and statuses.
3. Upsert reservations by property and reservation ID.
4. Insert reservation_events for new, modified, or cancelled messages.
5. Insert reservation_room_stays for per-room allocations.
6. Update normalization state and emit audit rows.

## ARI Steps
1. Read staged ARI payloads or retrieval results from booking-ari-sync.
2. Resolve property, room, and rate identities.
3. Write one ari_daily_facts row per property-room-rate-date combination.
4. Preserve closed, CTA, CTD, and LOS restriction fields.
5. Preserve standard and occupancy-based price values where present.
6. Use upsert semantics so repeat syncs refresh facts instead of duplicating them.

## Validation Rules
Quarantine or reject records if:
- departure_date is not after arrival_date
- availability is negative
- price values are malformed
- derived occupancy would become impossible
- source references do not map to known property scope

## Mapping Behavior
Hydrate properties, room_types, and rate_plans opportunistically. If a new room or rate appears, store it instead of failing the whole run. If mapping is ambiguous, normalize what is known and emit evidence for anomaly detection.

## Collaboration
- booking-reservations-sync provides raw reservation messages.
- booking-ari-sync provides ARI retrieval data.
- booking-ota-ack depends on successful reservation normalization.
- occupancy-outlook-builder and anomaly-detector depend on the quality of this output.

## Done Condition
The skill is complete when raw Booking.com payloads have been converted into validated, canonical SQLite rows with audit traceability intact.