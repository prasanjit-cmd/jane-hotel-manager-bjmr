---
name: booking-reservations-sync
version: 1.0.0
description: "Fetches OTA reservation messages from Booking.com and records message-level ingestion telemetry."
user-invocable: true
metadata:
 openclaw:
  requires:
   bins: [bash]
   env: [BOOKING_CONNECTIVITY_SECURE_BASE_URL, BOOKING_REQUEST_TIMEOUT_MS, BOOKING_MAX_RETRIES, BOOKING_RATE_LIMIT_PER_MINUTE, PROPERTY_SCOPE_ALLOWLIST, BOOKING_PROVIDER_ID]
---
# Booking Reservations Sync
## What This Skill Does
This skill pulls reservation messages from Booking.com using the OTA reservation flow. It is responsible for fetching new booking confirmations, modifications, and cancellations safely, idempotently, and with enough evidence for later acknowledgement.

## Purpose
Reservation timeliness is the highest operational priority. This skill is Jane's first line of defense against missed bookings, backlog, and fallback risk.

## Inputs
- Valid auth token from booking-auth-refresh
- Secure Booking.com reservation base URL
- Property scope or account scope
- Poll window parameters
- Local rate-limit settings

## Outputs
- sync_runs rows for each reservation sync attempt
- reservation_messages rows for each unique OTA message retrieved
- Raw payload references or checksums for auditability
- Retrieval metrics such as row counts, latency, and error state

## Core Rules
- Use the secure Booking.com endpoint for reservation retrieval.
- Keep retrieval and acknowledgement separate.
- De-duplicate on reservation identity plus message timing or checksum.
- Preserve enough identity to support exact acknowledgement later.
- If parsing is deferred or partial, still retain a raw reference so the message remains visible.

## Steps
1. Request valid auth from booking-auth-refresh.
2. Create a sync_runs row with run_type set to reservation_sync and status running.
3. Determine the scoped property set and request window.
4. Call OTA reservation retrieval endpoints for new, modified, and cancelled messages.
5. Capture HTTP status, latency, and retrieval counts.
6. For each returned message, compute a stable checksum, store reservation_messages metadata, and retain a raw payload reference.
7. Record duplicate observations without losing traceability.
8. Update sync_runs with final counts and status.
9. Hand off stored messages to hotel-data-normalizer.

## Rate Limiting
Use endpoint-aware throttling so reservation retrieval retains capacity even during heavy ARI polling. On 429:
- back off with jitter
- honor Retry-After if provided
- preserve partial state in sync_runs
- do not mark the property healthy if the pull was incomplete

## Failure Modes To Surface
- Secure endpoint unavailable after auth succeeded
- Partial payload retrieval
- Malformed OTA XML
- Catch-up duplication after outage
- Property-specific failure inside broader account sync

## Collaboration
- booking-auth-refresh supplies auth state.
- hotel-data-normalizer parses and canonicalizes retrieved messages.
- booking-ota-ack works from successfully normalized reservation_messages.
- anomaly-detector watches sync gaps, ack backlog, and queue age.

## Done Condition
The skill is complete when all retrievable OTA messages are safely recorded with run telemetry and raw evidence, without overstating downstream processing success.