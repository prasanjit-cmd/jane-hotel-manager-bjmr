---
name: incident-memory-writer
version: 1.0.0
description: "Writes operational summaries and property context into vector collections for future explanation and retrieval."
user-invocable: false
metadata:
 openclaw:
  requires:
   bins: [bash]
   env: [VECTOR_DB_URL, VECTOR_DB_API_KEY, EMBEDDING_MODEL, DATABASE_URL]
---
# Incident Memory Writer
## What This Skill Does
This skill turns selected operational records into durable vector memory for Jane Hotel Manager. It focuses on context that improves future explanations: recurring incidents, property-specific quirks, pricing constraints, mapping notes, alert resolutions, and significant reservation anomalies.

## Collections Covered
- property-context
- incident-history
- reservation-explanations
- ari-anomalies
- memory

## Inputs
- alerts and alert state transitions
- property_notes
- sync_runs excerpts
- selected reservation_events
- selected ARI anomaly evidence
- confirmed durable user preferences for summary style or reporting behavior

## Outputs
- Embedded documents stored in vector collections
- Metadata pointers back to structured rows
- Retrieval-ready summaries for explanation endpoints

## Content Rules
- Keep summaries concise and evidence-based.
- Avoid unnecessary guest PII.
- Include property ID, date range, incident type, severity, and outcome in metadata.
- Link every vector entry to at least one structured record ID.

## Embedding Triggers
- New medium or high severity alert
- Alert resolution or root-cause note addition
- Property note change
- Significant reservation anomaly such as repeated ack failure or modification burst
- Noteworthy ARI anomaly such as horizon loss or broad missing-price range
- Confirmed durable user preference at conversation end

## Steps
1. Select newly created or updated records that deserve memory.
2. Generate a compact narrative covering what happened, evidence, impact, and outcome if known.
3. Remove or mask guest-sensitive detail not needed for retrieval.
4. Embed content using the configured embedding model.
5. Upsert into the appropriate vector collection with metadata.
6. Store back-references so explanation APIs can show provenance.

## Retrieval Use
This skill supports questions such as:
- Has this happened before?
- What usually causes this on this property?
- Which prior fix worked?
- Is this pattern typical or unusual?

## Collaboration
- anomaly-detector identifies what is worth remembering.
- dashboard-api-server uses vectors for explanation routes.
- hotel-data-normalizer and occupancy-outlook-builder provide factual anchors.

## Done Condition
The skill is complete when Jane can retrieve relevant prior incidents and property context to produce historically aware, evidence-backed explanations.