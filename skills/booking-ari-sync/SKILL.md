---
name: booking-ari-sync
version: 1.0.0
description: "Retrieves and stages Booking.com rates, availability, and restrictions data for normalization and health analysis."
user-invocable: true
metadata:
 openclaw:
  requires:
   bins: [bash]
   env: [BOOKING_CONNECTIVITY_BASE_URL, BOOKING_REQUEST_TIMEOUT_MS, BOOKING_MAX_RETRIES, BOOKING_RATE_LIMIT_PER_MINUTE, BOOKING_ARI_POLL_INTERVAL_SECONDS, BOOKING_PRICING_TYPE_DEFAULT]
---
# Booking ARI Sync
## What This Skill Does
This skill retrieves Availability, Rates, and Inventory-related data from Booking.com and records fresh operational snapshots for sellability, rate coverage, restriction state, and horizon health.

## Scope
This skill is read-only in v1. It observes Booking.com ARI state and prepares auditable, date-grain data for normalization and downstream analysis. It does not push rate or availability changes.

## Inputs
- Valid Booking.com token
- Property scope
- ARI endpoint configuration
- Pricing model context where available
- Poll horizon and retry settings

## Outputs
- sync_runs rows with run_type ari_sync
- Staged or directly normalized ARI facts
- Evidence for missing data, stale syncs, and unexpected closures

## Why It Matters
A property can look open while still being unbookable because of missing prices, aggressive restrictions, or horizon drop-off. This skill provides the source material needed to catch those failures.

## Steps
1. Obtain a valid auth token.
2. Create a sync_runs record for the ARI pull.
3. Determine property, room, rate, and date scope.
4. Call Booking.com ARI-related endpoints with local throttling.
5. Persist response references and retrieval metrics.
6. Hand payloads to hotel-data-normalizer or write canonical staging rows.
7. Update sync_runs with row counts, latency, and status.
8. Emit audit evidence for missing payload segments or parse anomalies.

## Pricing-Type Awareness
The skill must not assume uniform pricing fields across properties. It should:
- detect or inherit pricing_type per property and rate plan
- preserve occupancy-based variants when present
- avoid flattening away important pricing semantics

## Failure Modes To Handle
- 429 rate limiting during broad portfolio pulls
- Partial date-range coverage
- Room-rate combinations disappearing between runs
- Pricing fields that differ by certification level
- Transient stale data after Booking.com-side delay

## Collaboration
- booking-auth-refresh provides auth.
- hotel-data-normalizer canonicalizes ARI payloads.
- occupancy-outlook-builder combines ARI and reservations.
- anomaly-detector looks for missing rates, closures, stale state, and horizon loss.

## Done Condition
The skill is complete when ARI retrieval produces auditable, date-grain operational facts suitable for sellability and inventory health analysis.