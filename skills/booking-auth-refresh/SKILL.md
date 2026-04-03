---
name: booking-auth-refresh
version: 1.0.0
description: "Refreshes and validates Booking.com Connectivity API access tokens using machine-account credentials."
user-invocable: false
metadata:
 openclaw:
  requires:
   bins: [bash]
   env: [BOOKING_MACHINE_ACCOUNT_USERNAME, BOOKING_MACHINE_ACCOUNT_PASSWORD, BOOKING_CONNECTIVITY_AUTH_URL, BOOKING_REQUEST_TIMEOUT_MS, BOOKING_MAX_RETRIES, LOG_LEVEL, ENVIRONMENT]
---
# Booking Auth Refresh
## What This Skill Does
This skill creates and refreshes authenticated Booking.com Connectivity API session state for Jane Hotel Manager. Booking.com uses short-lived tokens, so Jane centralizes refresh behavior here instead of letting reservation, ARI, and acknowledgement workflows each invent their own auth logic.

## Operational Intent
Authentication is a shared dependency. If auth fails, downstream ingestion must not pretend the system is healthy. The failure should be reflected in sync telemetry, surfaced to anomaly detection, and kept traceable for operators.

## Inputs
- Machine account username
- Machine account password or secret
- Token endpoint URL
- Timeout and retry settings
- Cached token state if present

## Outputs
- Current authenticated token or session secret
- Token expiry timestamp
- Auth health metadata
- Audit-safe telemetry for refresh success or failure

## Guardrails
- Never log raw credentials.
- Never log full live tokens in plaintext.
- Redact secrets in all traces.
- Refresh before expiry rather than waiting for hard failure.
- On 401, invalidate cache and perform one controlled refresh.
- Do not enter infinite retry loops.

## Steps
1. Check for a cached token and confirm it is valid for the next required sync window.
2. Reuse the cached token if it is still healthy.
3. If refresh is required, call the Booking.com auth endpoint over HTTPS.
4. Parse the returned token, expiry, and any scope metadata.
5. Persist only minimal runtime metadata needed for safe reuse.
6. Emit success or failure telemetry to sync and audit layers.
7. If refresh fails after bounded retries, mark dependent workflows blocked and provide evidence for alerting.

## Retry Policy
- Retry transient network and 5xx failures with exponential backoff and jitter.
- Do not loop on credential-related 4xx failures, aside from one controlled stale-cache retry.

## Failure Modes To Handle
- Invalid credentials
- Expired or revoked machine account
- Auth endpoint timeout
- Unexpected auth response schema
- Clock skew causing incorrect expiry logic

## Collaboration
- booking-reservations-sync depends on this skill before OTA retrieval.
- booking-ota-ack depends on this skill before acknowledgement.
- booking-ari-sync depends on this skill before ARI retrieval.
- anomaly-detector should consider repeated auth failures when opening alerts.

## Done Condition
The skill is complete when it returns a valid token or a clearly classified failure state, with enough telemetry to explain the outcome without exposing secrets.