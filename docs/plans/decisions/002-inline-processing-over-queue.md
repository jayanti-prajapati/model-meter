# ADR-002 — Process events inline in API (not via separate worker queue)

**Status:** ✅ Decided & implemented  
**Date:** October 2026  
**Commit:** `37dbca3`

---

## Context

The original design had a two-process architecture:

1. `apps/api` validates an event and pushes it to an **in-process array** (`eventQueue`)
2. `apps/worker` polls a **separate in-process `InProcessQueue`** and writes to the database

The intent was to decouple ingestion latency from processing latency, and to make it easy to swap the queue backend to SQS later.

## Problem discovered

Both processes maintain their own in-memory arrays. Node.js processes do not share memory. Events pushed into the API's `eventQueue` are invisible to the worker's `sharedQueue`. The result: every event was accepted with HTTP 202 but **never written to the database**. The bug was silent — no errors, just an empty `usage_events` table.

## Decision

Move `persistEvent()` directly into the API ingestion router. Events are now written to PostgreSQL synchronously as part of the ingestion request, before the 202 response is returned.

## Reasons

- **Correctness**: Events now actually land in the database.
- **Simplicity**: One process, one data path, no inter-process coordination.
- **Observability**: If persistence fails, the ingestion request returns an error in the `rejected` array. The SDK can log or retry.
- **MVP scope**: At current expected event volumes (hundreds per day per workspace), synchronous persistence adds negligible latency (<5 ms per event).

## Trade-offs

- **Latency coupling**: A slow database write now blocks the ingestion response. Acceptable at MVP scale.
- **No retry queue**: If persistence fails, the event is lost (reported in `rejected`). The SDK's own retry/flush logic provides a first line of defence.
- **Worker is now unused**: `apps/worker` remains in the repo and is still a valid deployment target for future async processing or aggregation jobs.

## When to revisit

- Ingestion latency P99 exceeds 200 ms under load
- Event volume exceeds ~10 K events/minute per workspace
- SQS or a durable queue is introduced for retry guarantees

At that point, move `persistEvent` back into the worker and connect the queue via an SQS adapter (the `IQueue<T>` interface in `apps/worker/src/queue/` already supports this).
