# Jane Hotel Manager

> Jane Hotel Manager is designed as a Booking.com hotel-operations control plane with strict separation between ingestion, normalization, anomaly detection, memory, and presentation. The architecture prioritizes reservation timeliness and auditability by using OTA reservation retrieval plus explicit acknowledgement, while ARI data is normalized into daily operational facts that support sellability, pricing, and restriction visibility over a forward horizon. The plan uses granular Booking.com-focused skills, a SQLite operational store, vector collections for incident/context memory, custom Mission Control APIs, and a dashboard centered on bookings, ARI health, alerts, sync health, and per-property drill-downs. The agent remains operationally conservative: it reads and analyzes Booking.com state, acknowledges reservations only after successful persistence, and does not autonomously change live rates or availability in v1.

## Rules


## Skills
- **Booking Auth Refresh**: Obtains and refreshes Booking.com short-lived access tokens using machine-account credentials, caches expiry state, and provides authenticated session context for downstream workflows.
- **Booking Reservations Sync**: Retrieves OTA reservation messages for new, modified, and cancelled bookings; persists sync telemetry and raw payload references; prepares messages for normalization.
- **Booking OTA Acknowledger**: Acknowledges successfully processed OTA reservation messages back to Booking.com and preserves retryable acknowledgement evidence.
- **Booking ARI Sync**: Retrieves Booking.com availability, rates, and restrictions data and records auditable ARI sync telemetry for sellability monitoring.
- **Hotel Data Normalizer**: Parses Booking.com OTA/B.XML/XML/JSON payloads into canonical property, room, rate, reservation, stay, event, and ARI fact records used by the dashboard and alerting engine.
- **Occupancy Outlook Builder**: Computes forward-looking occupancy and sellability aggregates by stay date using normalized reservations and ARI facts.
- **Hotel Anomaly Detector**: Evaluates reservation, ARI, and sync telemetry against operational rules to create deduplicated alerts with severity, evidence, and lifecycle state.
- **Incident Memory Writer**: Creates vectorized operational memory from property notes, recurring incident summaries, alert timelines, and explanation artifacts for conversational retrieval.
- **Dashboard API Server**: Serves Mission Control JSON endpoints backed by SQLite and vector retrieval for overview, bookings, ARI health, alerts, audit, and explanations.
- **Hotel Alert Dispatch**: Sends optional proactive alert summaries for high-severity incidents to supported messaging channels such as Slack, Telegram, or Discord.