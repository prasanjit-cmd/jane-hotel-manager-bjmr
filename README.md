# Jane Hotel Manager

> Jane Hotel Manager is designed as a Booking.com hotel-operations control plane with strict separation between ingestion, normalization, anomaly detection, memory, and presentation. The architecture prioritizes reservation timeliness and auditability by using OTA reservation retrieval plus explicit acknowledgement, while ARI data is normalized into daily operational facts that support sellability, pricing, and restriction visibility over a forward horizon. The plan uses granular Booking.com-focused skills, a SQLite operational store, vector collections for incident/context memory, custom Mission Control APIs, and a dashboard centered on bookings, ARI health, alerts, sync health, and per-property drill-downs. The agent remains operationally conservative: it reads and analyzes Booking.com state, acknowledges reservations only after successful persistence, and does not autonomously change live rates or availability in v1.

Built with [Ruh.ai](https://ruh.ai)
