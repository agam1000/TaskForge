# TaskForge (working name)

A fault-tolerant distributed job queue and scheduler built on Node.js, Express and PostgreSQL. Clients submit background jobs over a REST API; a pool of worker processes claims and runs them with leases, retries, a dead-letter queue and idempotent submission.

> **Status:** Work in progress. Sections marked `TODO` are filled in as features are built and measured.

## Problem

Web apps need to run work outside the request cycle: sending emails, resizing images, calling slow third-party APIs. Doing this naively (a `setTimeout`, an in-memory array, or a cron script) loses work when a process crashes, runs jobs twice when scaled to several instances, and gives no visibility into failures.

TaskForge explores how to make background execution **durable, safe under concurrency, and observable** without adding infrastructure beyond a single Postgres database.

## Goals

- No job is lost if a worker or the API crashes (durability).
- Multiple workers never process the same job at the same time (safe concurrency).
- Failed jobs retry sensibly and eventually land somewhere inspectable (reliability).
- Clients can safely retry submissions (idempotency).
- Behavior is measured, not assumed (benchmarks and chaos tests).

### Non-goals

Multi-region deployment, authentication and multi-tenant billing, a rich UI, and exactly-once delivery (see Delivery guarantees).

## Features

| Feature | Status |
|---|---|
| Submit, fetch and cancel jobs via REST | TODO |
| Atomic job claiming with `FOR UPDATE SKIP LOCKED` | TODO |
| Lease / visibility timeout and reaper | TODO |
| Retries with exponential backoff and jitter | TODO |
| Dead-letter queue with requeue endpoint | TODO |
| Idempotency keys | TODO |
| Worker heartbeats / lease renewal | TODO |
| Delayed jobs and priorities | TODO |
| Prometheus metrics and dashboard | TODO |

## Architecture

TODO: add diagram (Mermaid or draw.io).

```
Client --> Express API --> PostgreSQL (jobs table) <-- Worker 1..N
                                  ^
                                  |-- Reaper (requeues expired leases)
```

### Job lifecycle

```
queued --> running --> succeeded
              |
              +--> failed --(attempts left)--> queued (run_at = now + backoff)
              |
              +--> dead (attempts exhausted, visible in DLQ)
queued --> cancelled
```

## Key design decisions

TODO: for each decision, write what you chose, why, and the alternative you rejected.

1. **Postgres as the queue** (vs Redis, Kafka, SQS): TODO
2. **`SKIP LOCKED` claiming** (vs advisory locks, optimistic versioning): TODO
3. **Polling** (vs LISTEN/NOTIFY): TODO
4. **Lease duration and heartbeat interval**: TODO
5. **Retry policy** (backoff base, cap, jitter): TODO

## Delivery guarantees

TODO: explain at-least-once delivery, why exactly-once is not achievable across a network, and how idempotency keys and idempotent handlers give an exactly-once *effect*.

## Failure modes

| Failure | Behavior | Verified by |
|---|---|---|
| Worker crashes mid-job | TODO | TODO chaos test |
| Worker is slow (lease expires while still running) | TODO | TODO |
| API instance crashes | TODO | TODO |
| Database unavailable | TODO | TODO |
| Client retries a submit | TODO | TODO |
| Poison job (always fails) | TODO | TODO |

## API

TODO: request and response examples.

| Method | Path | Purpose |
|---|---|---|
| POST | `/jobs` | Submit a job |
| GET | `/jobs/:id` | Job status |
| POST | `/jobs/:id/cancel` | Cancel a queued job |
| GET | `/dlq` | List dead jobs |
| POST | `/dlq/:id/requeue` | Requeue a dead job |
| GET | `/metrics` | Prometheus metrics |
| GET | `/health` | Health check |

## Data model

TODO: paste final schema and index definitions, and explain the partial index used by the claim query.

## Getting started

TODO: confirm commands once the project runs.

```bash
git clone <repo-url>
cd queueforge
docker compose up --build
docker compose up --scale worker=3
```

## Testing

TODO: unit, integration and chaos test instructions.

## Benchmarks

TODO: fill with **measured** results only. Record the machine, Postgres settings, job type and test script used.

| Workers | Jobs | Throughput (jobs/s) | p50 (ms) | p99 (ms) | Lost jobs |
|---|---|---|---|---|---|
| 1 | TODO | TODO | TODO | TODO | TODO |
| 2 | TODO | TODO | TODO | TODO | TODO |
| 5 | TODO | TODO | TODO | TODO | TODO |

## What breaks at 100x scale

TODO: bottleneck analysis (hot table and index bloat, vacuum pressure, polling load, single primary) and the scaling path (partitioning, sharded queues, read replicas, moving to Kafka or SQS).

## Tech stack

Node.js, Express, PostgreSQL, node-postgres, Docker Compose, Jest and Supertest, Prometheus and Grafana, Locust (Python) for load tests.

## Roadmap

TODO: ideas not built yet (per-tenant rate limiting with a Redis token bucket, job dependencies, cron-style recurring jobs).

## License

TODO
