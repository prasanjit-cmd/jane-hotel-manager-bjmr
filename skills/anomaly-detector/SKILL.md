---
name: anomaly-detector
version: 1.0.0
description: "Detects hotel operations anomalies across reservations, ARI health, and sync telemetry, then creates actionable alerts."
user-invocable: true
metadata:
 openclaw:
  requires:
   bins: [bash]
   env: [DATABASE_URL, ALERT_SEVERITY_THRESHOLD, ALERT_DEFAULT_CHANNEL, LOG_LEVEL]
---
# Hotel Anomaly Detector
## What This Skill Does
This skill evaluates normalized hotel operations data and creates actionable alerts for operational issues. It watches reservations, acknowledgement state, sync telemetry, ARI daily facts, and occupancy outlook to identify conditions that could affect service, sellability, or operator trust.

## Alert Categories
- Unacknowledged reservation backlog
- Stale reservation sync
- Stale ARI sync
- Missing prices for active room-rate-date combinations
- Unexpected closed inventory or horizon drop-off
- Abrupt ADR change
- Mapping gaps causing partial visibility
- Repeated normalization failures
- Partial portfolio outage due to auth or endpoint issues

## Outputs
- alerts rows with severity, status, title, description, impacted dates, and fingerprint
- audit evidence for create, update, acknowledge, and resolve transitions
- optional dispatch input for hotel-alert-dispatch

## Principles
- Fingerprint duplicates so recurring checks update one alert instead of spamming many.
- Severity should reflect operational impact.
- Preserve first_seen_at and last_seen_at for MTTR analysis.
- Auto-resolve only after repeated healthy validation.
- Distinguish true anomaly from uncertainty caused by missing data.

## Steps
1. Query fresh normalized reservation, ARI, sync, and occupancy tables.
2. Evaluate rule groups by property and date range.
3. Compute severity and alert fingerprint.
4. Upsert alerts by creating new, updating existing, or resolving current ones.
5. Write audit_log evidence for state transitions.
6. Pass notifiable alerts to hotel-alert-dispatch if configured.
7. Send notable incidents to incident-memory-writer for future retrieval.

## Example Rules
- Reservation ack pending beyond threshold after normalization -> high severity.
- No successful reservation sync within expected interval -> high severity.
- Missing ARI future dates inside required horizon -> medium or high based on scope.
- ADR spike beyond property norm -> medium unless paired with closures.
- Closed inventory across all active rates on future dates -> high severity.

## False Positive Control
Use property notes and historical context where available. Known quirks can reduce severity or suppress duplicate noise, but critical reservation risk should never be hidden without explicit logic.

## Collaboration
- booking-ota-ack provides acknowledgement state.
- occupancy-outlook-builder provides future-date visibility.
- incident-memory-writer stores notable narratives.
- hotel-alert-dispatch handles optional outbound messaging.

## Done Condition
The skill is complete when current operational risks are represented as deduplicated alerts with evidence, lifecycle state, and enough context for a human to act without opening raw logs first.