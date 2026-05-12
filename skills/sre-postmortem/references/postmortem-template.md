# Postmortem template

Fill every section from the extracted narrative. Use roles, never names. Unknown fields → `[TBD — not captured during incident]`.

## Summary

2–3 sentences. What broke, how long it lasted, what the customer experienced.

## Customer Impact

- **Services affected:** Each service and its degradation type (down, degraded, slow).
- **Duration:** HH:MM in PT.
- **Estimated financial impact:** Dollar figure if stated in the incident channel. Otherwise `[TBD — not captured during incident]`.

## Timeline

All times in PT. Each entry: timestamp, event, acting role.

| Time (PT) | Event | Actor |
|---|---|---|
| HH:MM | What happened | Role (primary/secondary/IC) |

Mark approximate timestamps as `HH:MM (approximate)`.

## Root Cause

One paragraph. The specific technical failure — a commit, a config change, a dependency failure, a capacity threshold crossed. This is the trigger, not the context. "The on-call was slow to respond" belongs in Contributing Factors.

## Contributing Factors

Conditions that let the root cause become a customer-visible outage. Examples: no alert on the metric, deploy gate was bypassed, runbook was stale, the change landed on a Friday.

## 5 Whys

1. **Why did the incident occur?** (root cause from above)
2. **Why was that possible?**
3. **Why didn't existing defenses catch it?**
4. **Why were the defenses missing or insufficient?**
5. **Why wasn't this gap known before?**

The fifth answer must point to a systemic fix. If any answer names a person ("the on-call didn't notice"), restart from that step — ask why the person didn't notice (no alert? alert fatigue? wrong threshold?) until the answer is a system or process gap.

## What Went Well

Process strengths during the incident. Examples: alert fired quickly, IC escalated cleanly, runbook was accurate, rollback was fast, comms were clear.

## What Went Wrong

Process gaps. Examples: no one was paged for 8 minutes, runbook had stale commands, incident channel was created late, handoff between roles was unclear.

## Action Items

Each item is a single concrete task with a clear "done" state. No "investigate" — instead "determine X and fix it."

| # | Action | Owner (role) | Priority |
|---|---|---|---|
| 1 | Add alert on auth-latency p99 > 500ms | Platform | High |

Link each item to its Linear issue created in Stage 6.
