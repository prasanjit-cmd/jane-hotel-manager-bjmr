---
name: hotel-alert-dispatch
version: 1.0.0
description: "Dispatches concise high-severity operational alerts to configured channels without spamming operators."
user-invocable: true
metadata:
 openclaw:
  requires:
   bins: [bash]
   env: [ALERT_DEFAULT_CHANNEL, ALERT_SEVERITY_THRESHOLD]
---
# Hotel Alert Dispatch
## What This Skill Does
This skill delivers optional outbound notifications when Jane Hotel Manager identifies a material operational issue. The dashboard remains the source of truth; outbound messaging exists for fast awareness of high-severity events.

## Supported Channels
- Slack
- Telegram
- Discord

## Inputs
- Active alerts from anomaly-detector
- Routing or default channel configuration
- Severity threshold

## Outputs
- Outbound alert messages
- Audit records showing when a proactive alert was sent

## Dispatch Principles
- Send only when the alert materially changes operator awareness.
- Avoid repeating the same alert state unnecessarily.
- Include property, severity, issue, impact, and next best action.
- Prefer links back to Mission Control when available.

## Steps
1. Receive a candidate alert from anomaly-detector.
2. Check that severity meets threshold.
3. Confirm the alert is new or materially escalated.
4. Build a concise operator message.
5. Send to the configured channel.
6. Record dispatch timestamp and destination for audit.

## Guardrails
- Do not include raw guest PII.
- Do not send low-value repeated notices.
- Do not route around the dashboard; preserve Mission Control as canonical detail view.

## Example Message
"High severity: Property 4821 has 3 reservation messages pending acknowledgement for 18+ minutes after normalization. Last successful reservation sync: 20:05 UTC. Check Mission Control / Alerts for retry status and message details."

## Done Condition
The skill is complete when materially important incidents are surfaced to configured channels in a concise, non-spammy, traceable way.