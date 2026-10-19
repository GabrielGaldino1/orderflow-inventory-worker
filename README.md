# OrderFlow Inventory Worker

Quarkus application that will reserve, reject, and release inventory for the OrderFlow saga.

## F02 foundation

- Java 17 and Maven Wrapper.
- Quarkus Health, Hibernate ORM with Panache, PostgreSQL, Flyway, and Kafka client.
- Owned PostgreSQL database configured only through environment variables.
- Liveness at `/q/health/live` and readiness at `/q/health/ready`.
- Foundation health tests backed by an in-memory PostgreSQL-compatible H2 profile.

No Kafka consumer, reservation logic, or inventory domain table is implemented in F02.

## F05 transactional outbox

The worker owns an outbox table and a scheduled publisher prepared for the inventory events created
by later domain cards. Rows are claimed concurrently with `FOR UPDATE SKIP LOCKED`, published with
Kafka acknowledgements and idempotent producer settings, and marked only after acknowledgement.
Expired claims are retried with at-least-once delivery. `/q/metrics` exposes backlog size,
oldest-message age, attempts, failures, and publication latency without logging message payloads.

## Build and test

```powershell
./mvnw.cmd clean verify
```

Local runtime variables and orchestration commands are documented in `orderflow-platform`.
