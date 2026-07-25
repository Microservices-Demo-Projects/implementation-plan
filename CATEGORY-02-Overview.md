# CATEGORY-02 — Microservice Demo Projects

A portfolio of focused demo repos. **Each repo demonstrates one architectural
capability** — a non-functional / cross-cutting concern — with a
production-grade implementation, clear documentation, and automated scenario
verification. Each demo pairs its pattern with a focused business scenario,
keeping the domain lean so the architectural concern stays front and centre.

Every repo is self-contained — a reader can study any single demo, understand
the pattern fully, and lift it directly into their own system.

**Primary Tech Stack:** Java + Spring Boot · Kubernetes / OpenShift

---

## 2.0 Foundations

### 2.0.1 Philosophy on Business Domain

Each demo uses the business context that best showcases the pattern — familiar
domains like orders, bookings, or tasks are used where they fit naturally.
The domain provides enough structure to make the non-functional concern real
and meaningful, while keeping the service code focused on demonstrating the
architectural pattern clearly.

### 2.0.2 Commons Libraries — `ms-demo-commons` repo

Infrastructure libraries shared across every demo service. Domain-agnostic —
they wire in standard Spring Boot plumbing so each demo service gets
consistent, production-grade behaviour with zero boilerplate.

**Repo:** `ms-demo-commons`

**Deployed namespace:** N/A (libraries — not deployed)

**Design rules:**
- Each library is its **own** Maven project with its **own** `pom.xml`, published as an independent JAR
- Group ID: `demos.ponangi.com` · Versioning: `major.minor.patch`
- A BOM (`ms-demo-commons-bom`) pins all library versions — demo services import the BOM and never specify individual library versions
- Every library ships a `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` — wiring is automatic; apps add zero `@Bean` configuration for these concerns
- No library depends on another library in this repo — no transitive pull-in

#### `ms-demo-commons-bom`

Version management only — a single `<dependencyManagement>` import gives any
service access to all library versions without specifying them individually.

#### `commons-datasource`

Auto-configured DataSource with **built-in graceful credential rotation** and
**mTLS** support for PostgreSQL. Every service that depends on this library
gets live credential rotation for free.

**Credential rotation design — double-pool pattern:**

The Spring `DataSource` bean is a stable `RotatingCredentialsDataSource`
wrapper (NOT `@RefreshScope`). It holds two HikariCP pools during a rotation
— the old one drains, the new one serves. The Spring context is never
disrupted.

```
On RefreshScopeRefreshedEvent
  └─ CredentialRotationListener reads new username/password from mounted files
        └─ credentials changed?
              1. Build NEW HikariDataSource with new credentials
              2. oldPool.softEvictConnections()    ← idle connections evicted immediately;
                                                     checked-out connections finish their tx
                                                     then evicted when returned — no interruption
              3. activePool = newPool              ← atomic volatile swap; all new
                                                     getConnection() calls go here instantly
              4. schedule oldPool.close()          ← safety net after drain-timeout (default 30s)
```

**mTLS for PostgreSQL** — cert-manager issues all certs from a common CA.
The library builds the JDBC URL SSL params from configured paths:

```yaml
commons:
  datasource:
    credentials-path: /etc/secrets/db    # dir with username + password files
    drain-timeout: 30s
    tls:
      enabled: true
      ca-cert-path: /etc/certs/ca.crt           # verify DB server cert
      client-cert-path: /etc/certs/db/tls.crt   # app presents this to DB
      client-key-path: /etc/certs/db/tls.key
    pool:
      maximum-pool-size: 10
      minimum-idle: 2
      connection-timeout: 30000
    flyway:
      enabled: true                      # auto-run migrations on startup
```

Consuming services can override `spring.datasource.url`, `spring.datasource.hikari.*`,
and individual `commons.datasource.*` properties via their own ConfigMap.

#### `commons-logging`

Auto-configured structured JSON logging. Consuming services get production-grade
logging with MDC and Logback hot-reload from day one — no `logback-spring.xml`
in the service itself.

**What the library provides automatically:**
- Structured JSON output via Logstash encoder — every log line is a JSON object
- `scan="true"` with configurable scan period — when the service's logging
  ConfigMap changes, Logback detects it within the scan period without any external trigger or restart
- MDC auto-populated on every request: `traceId`, `spanId`, `correlationId`,
  `userId`, `service`, `env`
- Console appender by default; file appender with rollover policy when enabled
- Log retention and max-file-size configurable

**Overridable in consuming service:**

```yaml
commons:
  logging:
    scan-period: 10s             # how often Logback rescans the config file
    format: json                 # json | plain  (plain useful in local dev)
    mdc:
      correlation-id-header: X-Correlation-Id
      extra-fields:              # additional MDC fields extracted from request headers
        - header: X-Tenant-Id
          mdc-key: tenantId
```

When the consuming service mounts its own `logback-spring.xml` (from a
ConfigMap) at a known path and sets `logging.config:` pointing to it, that
file overrides the library's bundled config entirely. This is intentional —
the library's config is the fallback for when no external config file is
present (e.g. local development).

#### `commons-web`

Auto-configured Spring MVC plumbing.

- **RFC 7807 `ProblemDetail`** — global `@ControllerAdvice` handles all
  standard exceptions: `MethodArgumentNotValidException` → 400,
  `NoSuchElementException` → 404, `AccessDeniedException` → 403, uncaught
  → 500. Apps add their own `@ExceptionHandler` only for domain-specific cases
- **OpenAPI / Springdoc auto-config** — `/v3/api-docs` and `/swagger-ui.html`
  available with service name + version from `spring.application.name` and
  `commons.web.api-version`. Disable with `commons.web.openapi.enabled: false`

#### `commons-security`

Auto-configured JWT / OAuth2 security plumbing. Added to services that need
authentication — not all demos use it.

- JWT claims extractor — typed access to roles, subject, tenant from
  `SecurityContext` without raw `SecurityContextHolder` calls in app code
- Token propagation interceptor — when a service calls another via
  `RestClient` / `WebClient`, the inbound JWT is forwarded automatically
- Client Credentials flow helper — for service-to-service calls where the
  service needs its own token (not the user's)
- OAuth2 Implementation with **Keycloak**

#### `commons-messaging`

Auto-configured event envelope and idempotency utilities. Added to services
that publish or consume events — not all demos use it.

- `EventEnvelope<T>` — standard wrapper for all published events:
  `eventId`, `eventType`, `aggregateId`, `aggregateType`, `occurredAt`,
  `correlationId`, `payload`
- `IdempotencyKeyGenerator` — produces deterministic keys for deduplication
- `ProcessedEventStore` interface — backing store for idempotent consumers
  (implementations: in-memory for tests, JDBC for production)

#### `commons-observability`

Auto-configured Micrometer / Prometheus metrics plumbing.

- Prometheus registry auto-configured — `/actuator/prometheus` ready
- Base classes for custom business metrics counters and timers — services
  extend these to register domain-specific metrics consistently
- Health indicator helpers for custom `HealthIndicator` implementations

---

**How a demo service imports the commons:**

```xml
<!-- pom.xml — BOM import, version managed in one place -->
<dependencyManagement>
    <dependency>
        <groupId>demos.ponangi.com</groupId>
        <artifactId>ms-demo-commons-bom</artifactId>
        <version>1.0.0</version>
        <type>pom</type><scope>import</scope>
    </dependency>
</dependencyManagement>

<dependencies>
    <!-- Pick only what this service needs — no version required -->
    <dependency><groupId>demos.ponangi.com</groupId><artifactId>commons-logging</artifactId></dependency>
    <dependency><groupId>demos.ponangi.com</groupId><artifactId>commons-datasource</artifactId></dependency>
    <dependency><groupId>demos.ponangi.com</groupId><artifactId>commons-web</artifactId></dependency>
    <dependency><groupId>demos.ponangi.com</groupId><artifactId>commons-observability</artifactId></dependency>
    <!-- commons-security: only services that need JWT / OAuth2 -->
    <!-- commons-messaging: only services that publish or consume events -->
</dependencies>
```

> `commons-test` (Testcontainers utilities) is part of `ms-demo-commons` and
> is the foundation for `ms-testcontainers-demo`.

### 2.0.3 Standard Repo Structure

Every demo repo follows the same layout. Each service is an independent Maven
project with its own `pom.xml`. The structure is identical across every repo —
a reader already knows where everything is after seeing one.

```
ms-{topic}-demo/
├── README.md                # What this demo shows, prerequisites, how to run,
│                            # how to read the Allure report
├── SCENARIOS.md             # Every scenario: given/when/then + how to verify
├── docs/
│   ├── architecture.png     # High-level architecture diagram (draw.io export)
│   ├── flow-{scenario}.png  # Sequence / flow diagram per key scenario
│   └── notes.md             # Non-obvious implementation decisions only;
│                            # most important code/config snippets
├── services/
│   └── {service-name}/      # Own Maven project — src/, Dockerfile, pom.xml
├── ui/
│   └── {ui-app}/            # Web app (Angular, Vue, React, etc.) — Dockerfile, deployment config
├── tests/
│   └── scenario-runner/     # Lean JUnit 5 project — runs as K8s Job in cluster
├── deploy/
│   ├── helm/                # One Helm chart per service
│   │   └── {service}/
│   │       ├── Chart.yaml
│   │       ├── values.yaml          # Shared defaults
│   │       ├── values-local.yaml
│   │       ├── values-dev.yaml
│   │       └── templates/
│   ├── kustomize/
│   │   ├── base/
│   │   └── overlays/{local,dev}/
│   └── argocd/
│       ├── namespace.yaml
│       └── app-{service}.yaml       # One ArgoCD Application CR per service
└── scripts/
    ├── setup.sh             # Seed data, pre-flight dependency checks
    ├── run-scenarios.sh     # Run all e2e tests, open Allure report
    └── teardown.sh
```

**Standard README sections (every repo):**

```markdown
## What This Demo Shows
## Architecture
## Prerequisites
## Deploying to a Cluster
## Running the Scenario Runner
## Scenarios  (link to SCENARIOS.md; brief table of what each covers)
## How to Read the Test Results  (Allure report guide)
## Key Implementation Notes  (link to docs/notes.md)
```

### 2.0.4 Testing Strategy

Each demo is validated end-to-end in a running cluster through a dedicated
**scenario runner**. The scenario runner is the primary proof artefact — it
executes every scenario in `SCENARIOS.md` against the deployed service,
asserts the expected outcomes, and produces an Allure HTML report a reviewer
can read without touching the cluster.

Each demo repo contains a `tests/scenario-runner/` Maven project — a focused
JUnit 5 scenario runner that runs as a **Kubernetes Job** in the demo
namespace against the deployed services.

**What it does:**
- Resets preconditions (patches ConfigMaps to baseline, resets known state)
- Executes each scenario trigger (ConfigMap patch, Vault API call, Secret
  delete, etc.) using real cluster APIs
- Asserts the expected outcome via HTTP calls to the deployed service
- Generates an **Allure HTML report** — the primary proof artefact

**Tool stack:**

| Concern | Tool |
|---|---|
| HTTP assertions | REST-assured |
| Async polling / timing | Awaitility |
| Kubernetes API (ConfigMap / Secret / Pod ops) | Fabric8 `KubernetesClient` |
| Vault API (credential rotation) | Vault Java driver |
| Reporting | Allure |
| Service stubbing (failure scenarios, some demos only) | WireMock |

### 2.0.5 ConfigMap & Secret Design

Every demo uses the same four-resource pattern. The names and contents vary
per service; the reload strategy is always one of these four buckets.

| ConfigMap | Key | Mounted At | Reload | Stakater |
|---|---|---|---|---|
| `{svc}` | `application.yaml` — all stable properties; imports the features file | `/etc/config/app/` | Stakater → rolling restart | ✅ |
| `{svc}-features` | `features.yaml` — hot-reloadable overrides (feature flags, tunable rules) | `/etc/config/features/` | Config Watcher → `/actuator/refresh` → `@RefreshScope` beans | ❌ |
| `{svc}-logging` | `logback-spring.xml` (`scan="true"`) | `/etc/config/logging/` | Logback native file-scan | ❌ |
| `{svc}-runtime-env` | `JAVA_TOOL_OPTIONS`, `TZ`, JVM / OS-level env vars | `envFrom` | Stakater → rolling restart | ✅ |

Secrets are always file-mounted — never env vars. Rotation strategy mirrors
the ConfigMap pattern: secrets the app uses for ongoing connections are
hot-reloaded; secrets that define the app's own identity require a rolling
restart.

| Secret | Contents | Source | Mounted At | Reload |
|---|---|---|---|---|
| `{svc}-db-credentials` | `username`, `password` | Vault via ESO (mTLS) | `/etc/secrets/db/` | Config Watcher → `commons-datasource` double-pool rotation |
| `{svc}-db-client-tls` | Client TLS cert + key for DB mTLS | cert-manager | `/etc/certs/db-client/` | Config Watcher → `commons-datasource` double-pool rotation |
| `{svc}-server-tls` | App's own HTTPS server cert + key | cert-manager | `/etc/certs/server/` | Stakater → rolling restart |
| `{svc}-api-credentials` | Third-party API keys (when needed) | Vault via ESO | `/etc/secrets/api/` | Config Watcher → `@RefreshScope` |

**Key rules:**

- One `application.yaml` in the `{svc}` ConfigMap — any change triggers a
  rolling restart via Stakater.
- `{svc}-features` is mounted as `features.yaml`, imported by `application.yaml`
  via `spring.config.import`. Config Watcher hot-reloads it; only
  `@RefreshScope` beans act on the refreshed values.
- Stakater watches `{svc}`, `{svc}-runtime-env`, and `{svc}-server-tls` —
  all resources whose rotation requires a clean process restart.
- Config Watcher watches `{svc}-features`, `{svc}-db-credentials`, and
  `{svc}-db-client-tls` — resources whose rotation is safe to absorb live.
- `SPRING_PROFILES_ACTIVE` is set once in the Helm values / Deployment spec
  per environment.

### 2.0.6 Naming Conventions

```
Repos:            {org}/ms-demo-commons              # shared infra libraries
                  {org}/ms-{topic}-demo              # one repo per demo
                  {org}/infra-setup                  # cluster creation — separate
                  {org}/gitops-platform-setup        # middleware setup — separate

Namespaces:       demo-{topic}    (e.g. demo-config, demo-security, ...)

Maven coords:     groupId    = demos.ponangi.com
                  artifactId = {short-descriptive-name}
                  version    = major.minor.patch

Docker images:    {registry}/{org}/{repo-name}/{service-name}:{version}
                  e.g. ghcr.io/ponangi/ms-resilience-demo/order-service:1.0.0
```

### 2.0.7 Pre-Requisite Infrastructure

Managed by `gitops-platform-setup` with ArgoCD. Each demo's own section lists
the specific subset it depends on.

| Component | Used By |
|---|---|
| Kubernetes / OpenShift + ArgoCD | All |
| cert-manager | Security, Mesh |
| HashiCorp Vault | Config, Security |
| External Secrets Operator | Config, Security |
| Spring Cloud Kubernetes Config Watcher | Config |
| Stakater Reloader | Config |
| Keycloak | Security, API Design |
| Prometheus + Grafana | Observability, Scaling |
| OpenTelemetry Collector | Observability |
| Jaeger / Grafana Tempo | Observability |
| Loki + Promtail | Observability |
| Kafka (Strimzi Operator) | Events, Streaming, Dist-Tx |
| Apicurio Schema Registry | Events, Streaming |
| Apache Artemis | Distributed Transactions |
| PostgreSQL (CloudNativePG) | Database, Dist-Tx, Batch |
| Redis | Database, Resilience, Scaling |
| KEDA | Scaling |
| Istio | Mesh |
| Kagent | AI agent for k8s |

### 2.0.8 Implementation Sequence

Build in this order — later demos reference patterns proven in earlier ones.
Commons libraries must be published first.

```
Foundation:
  ms-demo-commons              (all 7 libraries published)

Section 1: Foundations
  01. ms-config-demo
  02. ms-identity-demo

Section 2: API Gateway & Routing
  03. ms-api-gateway-demo

Section 3: Data Layer
  04. ms-database-migrations-demo
  05. ms-caching-demo

Section 4: Observability & Monitoring
  06. ms-observability-demo

Section 5: API Styles & Contracts
  07. ms-rest-api-demo
  08. ms-grpc-api-demo
  09. ms-graphql-api-demo
  10. ms-realtime-api-demo

Section 6: Platform Services
  11. ms-service-mesh-demo     (Istio)
  12. ms-scaling-demo

Section 7: Distributed Systems & Transactions
  13A. ms-two-phase-commit-demo
  13B. ms-saga-demo            (Temporal)

Section 8: Event-Driven Architecture
  14A. ms-rabbitmq-events-demo
  14B. ms-event-sourcing-demo
  14C. ms-kafka-events-demo
  15.  ms-streaming-demo

Section 9: Batch Processing & Scheduling
  16. ms-quartz-scheduler-demo
  17. ms-spring-batch-demo

Section 10: Distributed Coordination
  18. ms-leader-election-demo

Section 11: Integration Testing
  19. ms-testcontainers-demo
```

---

## 2.1 Demo Index — 21 Focused Demos (Grouped by Section)

Each repo is independent, deployable to its own namespace. Conceptually they
build on each other, but can be studied in isolation.

### **Section 1: Foundations**

| #  | Status    | Repo                           | Namespace           | Focus |
|----|-----------|--------------------------------|---------------------|-------|
| **01** | `[ planned ]` | `ms-config-demo`               | `demo-config`       | Configuration externalization, hot reload, secret rotation |
| **02** | `[ planned ]` | `ms-identity-demo`             | `demo-identity`     | OAuth2 (user + service), JWT, mTLS, RBAC, BOLA, K8s Network Policies |

### **Section 2: API Gateway & Routing**

| #  | Status    | Repo                           | Namespace           | Focus |
|----|-----------|--------------------------------|---------------------|-------|
| **03** | `[ planned ]` | `ms-api-gateway-demo`          | `demo-gateway`      | Rate limiting, CORS, secure headers, request validation, routing |

### **Section 3: Data Layer**

| #  | Status    | Repo                           | Namespace           | Focus |
|----|-----------|--------------------------------|---------------------|-------|
| **04** | `[ planned ]` | `ms-database-migrations-demo`  | `demo-migrations`   | Liquibase/Flyway: DDL/DML deployment, versioning, rollbacks, tracking |
| **05** | `[ planned ]` | `ms-caching-demo`              | `demo-caching`      | Redis/Memcached: cache-aside, write-through, write-behind, invalidation, stampede prevention |

### **Section 4: Observability & Monitoring**

| #  | Status    | Repo                           | Namespace           | Focus |
|----|-----------|--------------------------------|---------------------|-------|
| **06** | `[ planned ]` | `ms-observability-demo`        | `demo-observability`| Grafana, Prometheus, Loki, Tempo, OpenTelemetry: logs, traces, metrics, dashboards, alerts |

### **Section 5: API Styles & Contracts**

| #  | Status    | Repo                           | Namespace           | Focus |
|----|-----------|--------------------------------|---------------------|-------|
| **07** | `[ planned ]` | `ms-rest-api-demo`             | `demo-rest`         | Versioning, pagination, filtering, RFC 7807 errors, HATEOAS, OpenAPI |
| **08** | `[ planned ]` | `ms-grpc-api-demo`             | `demo-grpc`         | Protocol Buffers, RPC, streaming, service definitions, load balancing |
| **09** | `[ planned ]` | `ms-graphql-api-demo`          | `demo-graphql`      | Schema design, resolvers, N+1 prevention, subscriptions, federation |
| **10** | `[ planned ]` | `ms-realtime-api-demo`         | `demo-realtime`     | WebSocket vs SSE: connection management, backpressure, trade-offs |

### **Section 6: Platform Services**

| #  | Status    | Repo                           | Namespace           | Focus |
|----|-----------|--------------------------------|---------------------|-------|
| **11** | `[ planned ]` | `ms-service-mesh-demo`         | `demo-mesh`         | Istio (ambient mode): resilience + traffic mgmt (canary, A/B, mirroring) + zero-trust security + observability |
| **12** | `[ planned ]` | `ms-scaling-demo`              | `demo-scaling`      | Kubernetes HPA, KEDA, ArgoCD, Pod Disruption Budgets |

### **Section 7: Distributed Systems & Transactions**

| #  | Status    | Repo                           | Namespace           | Focus |
|----|-----------|--------------------------------|---------------------|-------|
| **13A** | `[ planned ]` | `ms-two-phase-commit-demo`     | `demo-2pc`          | 2PC: coordinator pattern, synchronous, all-or-nothing, rollback |
| **13B** | `[ planned ]` | `ms-saga-demo`                 | `demo-saga`         | Temporal: saga orchestration, async workflows, compensation |

### **Section 8: Event-Driven Architecture**

| #  | Status    | Repo                           | Namespace           | Focus |
|----|-----------|--------------------------------|---------------------|-------|
| **14A** | `[ planned ]` | `ms-rabbitmq-events-demo`      | `demo-rabbitmq`     | RabbitMQ: event communication, idempotency, DLQ, retry mechanisms, correlation IDs |
| **14B** | `[ planned ]` | `ms-event-sourcing-demo`       | `demo-event-sourcing` | Event Sourcing + CQRS: immutable event store, write/read separation, state reconstruction |
| **14C** | `[ planned ]` | `ms-kafka-events-demo`         | `demo-kafka`        | Kafka: event streaming, consumer groups, schema registry (Apicurio), ordering |
| **15** | `[ planned ]` | `ms-streaming-demo`            | `demo-streaming`    | Kafka Streams: stateful processing, windowing, aggregation, topology |

### **Section 9: Batch Processing & Scheduling**

| #  | Status    | Repo                           | Namespace           | Focus |
|----|-----------|--------------------------------|---------------------|-------|
| **16** | `[ planned ]` | `ms-quartz-scheduler-demo`     | `demo-scheduler`    | Quartz: clustered job scheduling, avoiding duplicate triggers, K8s CronJobs |
| **17** | `[ planned ]` | `ms-spring-batch-demo`         | `demo-batch`        | Spring Batch: large-scale job processing, chunking, parallel processing, retries |

### **Section 10: Distributed Coordination**

| #  | Status    | Repo                           | Namespace           | Focus |
|----|-----------|--------------------------------|---------------------|-------|
| **18** | `[ planned ]` | `ms-leader-election-demo`      | `demo-leader-election` | K8s Lease API + Redis locks + etcd + database locks; singleton jobs, failover |

### **Section 11: Integration Testing**

| #  | Status    | Repo                           | Namespace           | Focus |
|----|-----------|--------------------------------|---------------------|-------|
| **19** | `[ planned ]` | `ms-testcontainers-demo`       | `demo-testcontainers` | Testcontainers: integration testing, container dependencies, scenario patterns |

---

See **CATEGORY-02-Demos.md** for detailed plans of all 21 demos.
