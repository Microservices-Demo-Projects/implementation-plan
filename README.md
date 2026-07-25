# Architecture Showcase Portfolio

Senior architect-level demonstration of microservice patterns, infrastructure
design, and production-grade implementations. Each repository is self-contained
and showcases one key architectural concern.

---

## Overview

This portfolio documents a comprehensive collection of microservice demos across
21 architectural capabilities, grouped into 11 logical sections. Each demo pairs
a clear non-functional concern with a focused business scenario, production-grade
code, automated scenario verification, and Allure HTML reports.

**Stack:** Java + Spring Boot · Kubernetes / OpenShift · Cloud-native patterns

---

## Structure

This documentation is split into three parts:

### [CATEGORY-01.md](CATEGORY-01.md) — Simple Changes

Quick POCs and single-repo refinements. Four focused items:

1. **Serverless expense-tracker** — Terraform setup, automated backup/restore via Lambda
2. **SFTP server variants** — Root vs non-root security postures
3. **Instrumentation agent** — Byte-code time travel for Date/Clock mocking
4. **AWS LightSail demo** — Documentation & presentation finish

---

### [CATEGORY-02-Overview.md](CATEGORY-02-Overview.md) — Foundations

Shared infrastructure and common patterns across all 21 microservice demos.

- **Philosophy:** Business domain is minimal and intentional; focus stays on
  the non-functional concern
- **Commons libraries** (7 libraries in `ms-demo-commons` repo) providing
  auto-configured Spring Boot behaviour
- **Standard repo structure** — identical layout across all demos
- **Testing strategy** — scenario runner pattern (JUnit 5 as Kubernetes Job)
- **ConfigMap & Secret design** — four-resource pattern (hot-reload vs rolling restart)
- **Naming conventions** — repos, namespaces, Maven coordinates
- **Infrastructure prerequisites** — cert-manager, Vault, ESO, Spring Cloud K8s, etc.
- **Implementation sequence** — build in sections; later demos reference earlier patterns

---

### [CATEGORY-02-Demos.md](CATEGORY-02-Demos.md) — Detailed Demo Plans

Complete specifications for each of the 21 demos:

- **Section 1 — Foundations** (01–02): Configuration & Secrets · Identity & Access Control
- **Section 2 — API Gateway & Routing** (03): Kong Gateway
- **Section 3 — Data Layer** (04–05): Database Migrations · Caching
- **Section 4 — Observability & Monitoring** (06): Grafana stack + OpenTelemetry
- **Section 5 — API Styles & Contracts** (07–10): REST · gRPC · GraphQL · Real-time
- **Section 6 — Platform Services** (11–12): Istio (ambient) · Scaling & HA
- **Section 7 — Distributed Transactions** (13A–13B): 2PC · Saga (Temporal)
- **Section 8 — Event-Driven Architecture** (14A–14C, 15): RabbitMQ · Event Sourcing + CQRS · Kafka · Kafka Streams
- **Section 9 — Batch Processing & Scheduling** (16–17): Quartz · Spring Batch
- **Section 10 — Distributed Coordination** (18): Leader Election (K8s Lease + Redis + PostgreSQL)
- **Section 11 — Integration Testing** (19): Testcontainers

---

## Quick Navigation

| You want to... | Read this |
|---|---|
| Understand the overall vision | This README |
| See quick POCs | [CATEGORY-01.md](CATEGORY-01.md) |
| Learn the shared foundations | [CATEGORY-02-Overview.md](CATEGORY-02-Overview.md) |
| Review a specific demo in detail | [CATEGORY-02-Demos.md](CATEGORY-02-Demos.md) |
| Understand commons libraries | [CATEGORY-02-Overview.md § 2.0.2](CATEGORY-02-Overview.md#202-commons-libraries--ms-demo-commons-repo) |
| See the scenario runner strategy | [CATEGORY-02-Overview.md § 2.0.4](CATEGORY-02-Overview.md#204-testing-strategy) |
| Review Config/Secret patterns | [CATEGORY-02-Overview.md § 2.0.5](CATEGORY-02-Overview.md#205-configmap--secret-design) |
| Dive into Config Demo details | [CATEGORY-02-Demos.md § Demo 01](CATEGORY-02-Demos.md#demo-01--configuration--secret-management) |
| Dive into Identity Demo details | [CATEGORY-02-Demos.md § Demo 02](CATEGORY-02-Demos.md#demo-02--identity--access-control) |
| See the full demo index | [CATEGORY-02-Demos.md § Demo Index](CATEGORY-02-Demos.md#demo-index--21-focused-demos-grouped-by-section) |

---

## Key Concepts

**Three reload strategies** (Demo 01):
- **Logback file-scan** — native Logback polling; no app restart
- **Config Watcher + @RefreshScope** — Spring Cloud K8s watches ConfigMaps; selected beans re-bind live
- **Stakater rolling restart** — watch ConfigMaps/Secrets; rolling restart when they change

**Identity in depth** (Demo 02):
- User auth: OAuth2 Authorization Code + PKCE (Keycloak `demo-users` realm)
- Service auth: OAuth2 Client Credentials (Keycloak `demo-services` realm)
- Every layer validates JWT independently — no blind forwarding
- mTLS at the transport layer (cert-manager) — independent of OAuth2
- RBAC + BOLA prevention at every endpoint

**Platform-level cross-cutting concerns:**
- Rate limiting, CORS, secure headers → **Kong Gateway** (Demo 03)
- Resilience, traffic management, zero-trust, observability → **Istio ambient** (Demo 11)
- Scaling → **HPA + KEDA + Argo Rollouts** (Demo 12)
- Observability → **Grafana stack + OpenTelemetry** (Demo 06)

**Shared libraries** (all demos):
- `commons-datasource` — graceful credential rotation, mTLS, connection pooling
- `commons-logging` — structured JSON, MDC, Logback file-scan hot-reload
- `commons-security` — JWT validation, OAuth2, Client Credentials, mTLS WebClient
- `commons-web` — RFC 7807 error handling, OpenAPI integration
- `commons-observability` — Prometheus, Micrometer, health indicators
- `commons-messaging` — event envelopes, idempotency
- `commons-test` — Testcontainers utilities (used by Demo 19)

**ConfigMap & Secret split** (all demos):
- `{svc}` ConfigMap — rolling restart (Stakater)
- `{svc}-features` ConfigMap — hot-reload via Config Watcher
- `{svc}-logging` ConfigMap — Logback native file-scan
- `{svc}-runtime-env` ConfigMap — JVM options, rolling restart
- Secrets mirrored: stable secrets trigger rolling restart; connection secrets trigger hot-reload

---

## Philosophy

This portfolio demonstrates **how a senior architect thinks about microservices**:

### Use proven open-source tools
- Keycloak for auth, not custom OAuth2
- Kafka for events, not home-built pub/sub
- Istio for resilience, not language-specific libraries
- Temporal for sagas, not choreography scripts
- Grafana stack for observability, not DIY logging

### Prefer platform-level solutions over app-level complexity
- API Gateway (Kong) handles rate limiting, CORS, headers, request validation
- Service mesh (Istio ambient) handles retries, circuit breaking, timeouts, mTLS, tracing
- Kubernetes handles scaling, leader election, health checks
- App code focuses on **business logic only**

### Architecture-first, framework-second
- Each demo teaches the **concept**, not the code
- Implementation notes in pseudocode where the concept is language-agnostic
- A developer should learn "why JWT + Client Credentials" before "how Spring OAuth2"
- Patterns apply to Python, Go, Node.js, Rust — not just Java

### Each demo solves exactly one concern
- No monolithic "security" demo covering IAM + TLS + rate limiting + secrets
- No "API design" that tries to teach REST + gRPC + GraphQL in one repo
- Split into 21 focused demos instead of 5 sprawling ones
- Scope: ~2–4 week engineering effort per demo

### Independent but conceptually connected
- A team can study Demo 03 (API Gateway) without Demo 02 (Identity)
- Each repo is deployable to its own namespace
- But reading them in order builds a complete mental model

---

**See also:** [CATEGORY-01.md](CATEGORY-01.md) · [CATEGORY-02-Overview.md](CATEGORY-02-Overview.md) · [CATEGORY-02-Demos.md](CATEGORY-02-Demos.md)
