---
name: booking-ota-ack
version: 1.0.0
description: "Acknowledges Booking.com OTA reservation messages only after successful persistence and validation."
user-invocable: true
metadata:
 openclaw:
  requires:
   bins: [bash]
   env: [BOOKING_CONNECTIVITY_SECURE_BASE_URL, BOOKING_REQUEST_TIMEOUT_MS, BOOKING_MAX_RETRIES, BOOKING_RATE_LIMIT_PER_MINUTE]
---
# Booking OTA Acknowledger
## What This Skill Does
This skill sends acknowledgement responses for OTA reservation messages that have already been retrieved, parsed, validated, and persisted. It exists to enforce Jane's strict rule that Booking.com messages are only acknowledged after durable local processing succeeds.

## Purpose
Acknowledgement is operationally meaningful. Sending it too early risks data loss. Sending it too late increases fallback and service risk. This skill manages that boundary carefully.

## Inputs
- reservation_messages with ack_required true
- Normalization success state for each message
- Valid auth token
- Secure Booking.com acknowledgement endpoint

## Outputs
- Updated ack_status, ack_attempted_at, and ack_confirmed_at values
- Audit entries for acknowledgement attempts
- Retryable failure state when acknowledgement does not complete

## Core Rules
- Never acknowledge a message that failed normalization.
- Never acknowledge based on retrieval alone.
- Keep pending, succeeded, and failed ack states operator-visible.
- Support manual re-ack from the dashboard.
- Preserve message identity for exact retry targeting.

## Steps
1. Query reservation_messages that require acknowledgement and are safe to acknowledge.
2. Confirm linked reservation and reservation_event rows exist.
3. Request valid auth from booking-auth-refresh.
4. Build the acknowledgement payload matching the original message type.
5. Send the acknowledgement request to the secure Booking.com endpoint.
6. Mark ack_status succeeded on confirmed success.
7. On errors, mark the message error or pending_retry with diagnostic metadata.
8. Emit audit_log rows for each attempt.

## Retry Logic
- Retry transient failures with exponential backoff and jitter.
- Honor Booking.com rate limiting behavior.
- Keep retries bounded and visible.
- Escalate persistent failures into alerting evidence.

## Failure Modes
- Message no longer valid for acknowledgement
- Token expired before ack submission
- Secure endpoint timeout
- Schema mismatch in ack payload
- Duplicate acknowledgement attempt after success

## Collaboration
- booking-reservations-sync populates reservation_messages queue state.
- hotel-data-normalizer determines whether a message is safe to acknowledge.
- anomaly-detector evaluates ack age and repeated ack failures.
- dashboard-api-server exposes message details and manual retry actions.

## Done Condition
The skill is complete when safe-to-ack messages are acknowledged, unsafe messages remain unacknowledged, and every attempt remains traceable for audit.