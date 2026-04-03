---
name: dashboard-api-server
version: 1.0.0
description: "Serves operational hotel data to Mission Control and explanation interfaces."
user-invocable: false
metadata:
 openclaw:
  requires:
   bins: [bash]
   env: [DATABASE_URL, AGENT_BASE_URL, VECTOR_DB_URL, VECTOR_DB_API_KEY, EMBEDDING_MODEL, LOG_LEVEL]
---
# Dashboard API Server
## What This Skill Does
This skill exposes the structured HTTP endpoints used by Mission Control and related operator interfaces. It reads from canonical SQLite tables, augments explanation routes with vector retrieval when needed, and returns dashboard-friendly JSON for metrics, charts, tables, and drill-downs.

## Responsibilities
- Serve overview KPIs and trends
- Serve filtered booking tables and reservation detail views
- Serve ARI health summaries and daily facts
- Serve alert listings and alert detail drill-downs
- Serve sync and audit views
- Serve explanation routes that combine SQL facts with vector memory

## Input Sources
- properties
- room_types
- rate_plans
- sync_runs
- reservation_messages
- reservations
- reservation_room_stays
- reservation_events
- ari_daily_facts
- occupancy_outlook_daily
- alerts
- property_notes
- audit_log
- vector collections

## Output Style
Responses should stay operational and dashboard-oriented. Avoid leaking parser internals unless the audit page explicitly needs them. Support filtering by property, date range, status, severity, and alert type.

## Steps
1. Validate query parameters.
2. Execute parameterized SQLite queries or aggregates.
3. For explanation routes, retrieve current structured evidence first.
4. Enrich with vector retrieval from incident-history, property-context, reservation-explanations, ari-anomalies, or memory when relevant.
5. Return compact JSON suited for metric cards, tables, charts, and activity feeds.

## Guardrails
- Use parameterized queries only.
- Never expose credentials, tokens, or sensitive payload fragments.
- Mask guest-sensitive fields where not operationally required.
- Keep response time predictable with indexes and bounded windows.

## Explanation Behavior
The explanation endpoint should answer questions such as:
- Why did occupancy outlook fall?
- Why is this property unbookable next month?
- Has this sync failure pattern happened before?
It should clearly separate current SQL facts from historical analogies retrieved from vectors.

## Collaboration
This skill depends on upstream normalization, outlook building, anomaly detection, and memory writing. It is the presentation layer for Mission Control.

## Done Condition
The skill is complete when all Mission Control pages can load from stable endpoints and explanation routes can combine structured evidence with relevant memory.