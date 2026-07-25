# CATEGORY-02 — Detailed Demo Plans

Detailed implementation plans for each of the 21 microservice architecture
demos, grouped into 11 sections. Read in order (Demo 01 → 19) for the full
architectural journey, or jump to the [Demo Index](#demo-index--21-focused-demos-grouped-by-section)
for a quick overview.


## Demo Index — 21 Focused Demos (Grouped by Section)

Each repo is independent, deployable to its own namespace. They build on each other conceptually but can be studied in isolation.

### **Section 1: Foundations**

| #  | Status    | Repo                           | Namespace           | Focus |
|----|-----------|--------------------------------|---------------------|-------|
| **01** | `[ planned ]` | `ms-config-demo`               | `demo-config`       | Configuration externalization, hot reload, secret rotation |
| **02** | `[ planned ]` | `ms-identity-demo`             | `demo-identity`     | OAuth2 (user + service), JWT, mTLS, RBAC, BOLA, K8s Network Policies |

### **Section 2: API Gateway & Routing**

| #  | Status    | Repo                           | Namespace           | Focus |
|----|-----------|--------------------------------|---------------------|-------|
| **03** | `[ planned ]` | `ms-api-gateway-demo`          | `demo-gateway`      | Rate limiting, CORS, secure headers, validation, routing |

### **Section 3: Data Layer**

| #  | Status    | Repo                           | Namespace           | Focus |
|----|-----------|--------------------------------|---------------------|-------|
| **04** | `[ planned ]` | `ms-database-migrations-demo`  | `demo-migrations`   | Liquibase/Flyway: DDL/DML deployment, versioning, rollbacks |
| **05** | `[ planned ]` | `ms-caching-demo`              | `demo-caching`      | Redis/Memcached: cache-aside, write-through, write-behind, invalidation, stampede |

### **Section 4: Observability & Monitoring**

| #  | Status    | Repo                           | Namespace           | Focus |
|----|-----------|--------------------------------|---------------------|-------|
| **06** | `[ planned ]` | `ms-observability-demo`        | `demo-observability`| Grafana, Prometheus, Loki, Tempo, OpenTelemetry: logs, traces, metrics, alerts |

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
| **11** | `[ planned ]` | `ms-service-mesh-demo`         | `demo-mesh`         | Istio (ambient mode): resilience + traffic mgmt + zero-trust security + observability |
| **12** | `[ planned ]` | `ms-scaling-demo`              | `demo-scaling`      | HPA, KEDA, ArgoCD, Pod Disruption Budgets |

### **Section 7: Distributed Systems & Transactions**

| #  | Status    | Repo                           | Namespace           | Focus |
|----|-----------|--------------------------------|---------------------|-------|
| **13A** | `[ planned ]` | `ms-two-phase-commit-demo`     | `demo-2pc`          | 2PC: coordinator pattern, synchronous, all-or-nothing, rollback |
| **13B** | `[ planned ]` | `ms-saga-demo`                 | `demo-saga`         | Temporal: saga orchestration, async workflows, compensation |

### **Section 8: Event-Driven Architecture**

| #  | Status    | Repo                           | Namespace           | Focus |
|----|-----------|--------------------------------|---------------------|-------|
| **14A** | `[ planned ]` | `ms-rabbitmq-events-demo`      | `demo-rabbitmq`     | RabbitMQ: event communication, idempotency, DLQ, retry, correlation IDs |
| **14B** | `[ planned ]` | `ms-event-sourcing-demo`       | `demo-event-sourcing` | Event Sourcing + CQRS: event store, write/read separation, snapshots |
| **14C** | `[ planned ]` | `ms-kafka-events-demo`         | `demo-kafka`        | Kafka: event streaming, consumer groups, schema registry, ordering |
| **15** | `[ planned ]` | `ms-streaming-demo`            | `demo-streaming`    | Kafka Streams: stateful processing, windowing, aggregation, topology |

### **Section 9: Batch Processing & Scheduling**

| #  | Status    | Repo                           | Namespace           | Focus |
|----|-----------|--------------------------------|---------------------|-------|
| **16** | `[ planned ]` | `ms-quartz-scheduler-demo`     | `demo-scheduler`    | Quartz: clustered scheduling, duplicate prevention, K8s CronJobs |
| **17** | `[ planned ]` | `ms-spring-batch-demo`         | `demo-batch`        | Spring Batch: large-scale processing, chunking, parallel, retries |

### **Section 10: Distributed Coordination**

| #  | Status    | Repo                           | Namespace           | Focus |
|----|-----------|--------------------------------|---------------------|-------|
| **18** | `[ planned ]` | `ms-leader-election-demo`      | `demo-leader-election` | K8s Lease API + Redis locks + etcd + database locks; singleton jobs |

### **Section 11: Integration Testing**

| #  | Status    | Repo                           | Namespace           | Focus |
|----|-----------|--------------------------------|---------------------|-------|
| **19** | `[ planned ]` | `ms-testcontainers-demo`       | `demo-testcontainers` | Testcontainers: integration testing, container dependencies, scenarios |


---

## Demo 01 — Configuration & Secret Management

**Repo:** `ms-config-demo` · **Namespace:** `demo-config`

**Focus:** Demonstrate the three reload strategies for a Spring Boot service on
Kubernetes — Logback file-scan, Config Watcher + `@RefreshScope`, and Stakater
Reloader rolling restart — with a clear, justified boundary between them. DB
credential rotation without restart is the centrepiece secret-management
scenario, implemented via `commons-datasource`'s double-pool pattern.

**What a reader takes away:** Exactly which category each property belongs in,
why the ConfigMap split enforces that boundary, and how to implement live
credential rotation without dropping a single request.

### Service

`order-service` — a focused Order CRUD API that provides the right surface
area to demonstrate every config and secret management scenario.

```
POST /orders              → create an order (enforces max-quantity rule)
GET  /orders/{id}         → fetch a single order
GET  /orders              → list orders (paginated)
GET  /orders/bulk         → gated by bulk-orders-enabled feature flag
```

Commons used: `commons-logging` `commons-datasource` `commons-web`
`commons-observability`

### Pre-Requisite Infrastructure

| Component | Role in this demo |
|---|---|
| **cert-manager** | Common internal CA; issues TLS cert for Vault, PostgreSQL server cert, app server cert (Ingress/Route HTTPS), app DB-client cert |
| **HashiCorp Vault** | Authoritative secret store — DB credentials live here; TLS-terminated (cert-manager cert) |
| **External Secrets Operator** | Connects to Vault over mTLS; syncs DB credentials to a Kubernetes Secret |
| **Spring Cloud K8s Config Watcher** | Watches K8s API; POSTs `/actuator/refresh` to each pod IP on ConfigMap / Secret change |
| **Stakater Reloader** | Watches `order-runtime-env` ConfigMap; triggers rolling restart on any change |
| **PostgreSQL (CloudNativePG)** | Deployed with TLS + client-cert auth; app connects with mTLS |

**cert-manager CA chain:**

```
cert-manager ClusterIssuer (internal CA)
  ├── vault-tls           ← Vault server cert         (ESO verifies when connecting to Vault)
  ├── eso-client-tls      ← ESO client cert           (ESO presents this to Vault — mTLS)
  ├── postgres-tls        ← PostgreSQL server cert    (app verifies this on every connection)
  ├── order-server-tls    ← app HTTPS server cert     (Ingress/Route; Stakater rolling restart on rotation)
  └── order-db-client-tls ← app DB client cert        (app presents this to PostgreSQL; hot-reload via commons-datasource)
```

### ConfigMap & Secret Layout

```
ConfigMap: order-service            key: application.yaml
  mounted at: /etc/config/app/application.yaml
  Stakater: ✅  — any change triggers rolling restart
  annotation: reloader.stakater.com/search: "order-service,order-runtime-env"

ConfigMap: order-features           key: features.yaml
  mounted at: /etc/config/features/features.yaml
  imported by application.yaml via spring.config.import
  Config Watcher → /actuator/refresh → @RefreshScope beans update live

ConfigMap: order-logging            key: logback-spring.xml
  mounted at: /etc/config/logging/logback-spring.xml
  Logback scan="true" — detects file change within scanPeriod, no restart

ConfigMap: order-runtime-env        envFrom: configMapRef
  JAVA_TOOL_OPTIONS, TZ
  Stakater: ✅  — rolling restart on any change

Secret: order-db-credentials        keys: username, password
  mounted at: /etc/secrets/db/      (ESO-managed; Vault source; mTLS)
  Config Watcher → /actuator/refresh
  → commons-datasource double-pool rotation (zero disruption to in-flight requests)

Secret: order-db-client-tls         keys: tls.crt, tls.key
  mounted at: /etc/certs/db-client/ (cert-manager; same internal CA)
  Config Watcher → /actuator/refresh
  → commons-datasource double-pool rotation (new pool connects with new client cert)

Secret: order-server-tls            keys: tls.crt, tls.key
  mounted at: /etc/certs/server/    (cert-manager; same internal CA)
  Stakater: ✅  — cert rotation triggers rolling restart
  annotation: reloader.stakater.com/search: "order-service,order-runtime-env,order-server-tls"
```

### ConfigMap Contents

**`order-service` — `application.yaml` (rolling restart on change):**

```yaml
spring:
  config:
    import:
      - optional:file:/etc/config/features/features.yaml   # hot-reloadable overrides

  datasource:
    url: jdbc:postgresql://postgres.demo-config.svc:5432/orders
  jpa:
    open-in-view: false
  flyway:
    enabled: true

server:
  port: 8080
  shutdown: graceful
  ssl:
    enabled: true
    bundle: order-service-tls

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,configprops,refresh,prometheus
  endpoint:
    health:
      show-details: always

commons:
  datasource:
    credentials-path: /etc/secrets/db
    tls:
      ca-cert-path: /etc/certs/ca.crt
      client-cert-path: /etc/certs/db-client/tls.crt
      client-key-path: /etc/certs/db-client/tls.key
  logging:
    scan-period: 10s

logging:
  config: /etc/config/logging/logback-spring.xml

# Defaults — overridden by features.yaml at runtime
app:
  features:
    bulk-orders-enabled: false
  orders:
    max-quantity: 10
```

**`order-features` — `features.yaml` (hot-reload via Config Watcher):**

```yaml
# Overrides the app.* defaults in application.yaml.
# @RefreshScope beans re-bind to these values on /actuator/refresh.
# Only properties whose beans carry @RefreshScope have any effect here.
app:
  features:
    bulk-orders-enabled: false     # S03 — toggle to true to enable GET /orders/bulk
  orders:
    max-quantity: 10               # S04 — reduce to 3 to tighten the business rule
```

The service JAR's `src/main/resources/application.yaml` contains only
`spring.application.name` and the Spring Cloud K8s bootstrap configuration.
Everything else is externalised to the cluster.


### Key Implementation Notes

**`@RefreshScope` on the two live-reloadable config beans:**

```java
@ConfigurationProperties(prefix = "app.features")
@RefreshScope                              // re-bound on /actuator/refresh
public class FeatureFlagConfig {
    private boolean bulkOrdersEnabled;
}

@ConfigurationProperties(prefix = "app.orders")
@RefreshScope                              // re-bound on /actuator/refresh
public class OrderRulesConfig {
    private int maxQuantity;
}
// All other @ConfigurationProperties: no @RefreshScope — stable at startup
```

**Gating an endpoint with a feature flag:**

```java
@GetMapping("/orders/bulk")
public ResponseEntity<?> getBulkOrders() {
    if (!featureFlagConfig.isBulkOrdersEnabled()) {
        return ResponseEntity.notFound().build();   // 404, not 403 — don't confirm existence
    }
    return ResponseEntity.ok(orderService.getBulkOrders());
}
```

**Logback config (`order-logging` ConfigMap):**

```xml
<configuration scan="true" scanPeriod="10 seconds">
  <!-- commons-logging provides JSON encoder, appender, and MDC filter.
       This file only declares log levels. -->
  <springProperty name="LOG_LEVEL_ROOT" source="logging.level.root"              defaultValue="INFO"/>
  <springProperty name="LOG_LEVEL_APP"  source="logging.level.demos.ponangi.com" defaultValue="INFO"/>
  <root level="${LOG_LEVEL_ROOT}"><appender-ref ref="JSON_CONSOLE"/></root>
  <logger name="demos.ponangi.com" level="${LOG_LEVEL_APP}"/>
</configuration>
```



### Scenarios

| # | Scenario | Reload mechanism | What changes | Validation |
|---|---|---|---|---|
| **S01** | Service starts — all config from ConfigMaps, secrets from Vault, DB connected via mTLS | — | Startup | `GET /actuator/configprops` shows expected values; DB query succeeds; TLS handshake logs confirm client cert used |
| **S02** | Log level changed INFO → DEBUG | Logback `scan="true"` (no Config Watcher, no restart) | `logback-spring.xml` in `order-logging` | Wait ≤ 15s; next request produces DEBUG logs; pod age unchanged |
| **S03** | `bulk-orders-enabled` toggled false → true | Config Watcher → `/actuator/refresh` → `@RefreshScope` | `features.yaml` in `order-features` ConfigMap | `GET /orders/bulk` 404 before; 200 after; pod age unchanged; no restart |
| **S04** | `max-quantity` changed 10 → 3 | Config Watcher → `/actuator/refresh` → `@RefreshScope` | `features.yaml` in `order-features` ConfigMap | `POST /orders` qty=5 returns 200 before; 400 after; pod age unchanged |
| **S05** | DB credentials rotated in Vault — zero failed requests | Vault → ESO syncs Secret → Config Watcher → refresh → `commons-datasource` double-pool rotation | `order-db-credentials` Secret | Load gen at 100 req/s throughout; assert zero 5xx during full rotation window; `/actuator/health` confirms new pool active |
| **S06** | Server TLS cert rotated by cert-manager — rolling restart, zero downtime | Stakater → rolling restart (watches `order-server-tls` Secret) | `order-server-tls` Secret deleted → cert-manager re-issues → Stakater detects new secret | Load gen running throughout; new cert serial confirmed after rollout; zero 5xx |
| **S07** | JVM heap resized 512m → 1g — rolling restart, zero downtime | Stakater → rolling restart (watches `order-runtime-env`) | `JAVA_TOOL_OPTIONS` in `order-runtime-env` | Before: `jvm.memory.max` = 536870912; change applied; load gen running; after rollout: `jvm.memory.max` = 1073741824; zero 5xx throughout |



### Scenario Runner

Runs as a Kubernetes Job in `demo-config` against the already-deployed
`order-service`. One JUnit 5 test class per scenario. Report fetched after
job completes.

```bash
kubectl apply -f deploy/scenario-runner-job.yaml -n demo-config
kubectl wait --for=condition=complete job/scenario-runner -n demo-config --timeout=10m
# copy Allure results from pod and serve
```

---

## Demo 02 — Identity & Access Control

**Repo:** `ms-identity-demo` · **Namespace:** `demo-identity`

**Focus:** Production-grade Identity and Access Control (IAC) architecture
demonstrating three core concepts: **Authentication** (user identity via OAuth2
Authorization Code + PKCE), **Authorization** (RBAC + BOLA prevention), and
**mTLS** (service-to-service cryptographic identity). Two independent OAuth2
flows: user tokens for browser authentication, service credentials for
inter-service calls. No API gateway — direct service calls with proper
authentication at each boundary.

**What a reader takes away:** Why services never forward user tokens to peers
(principle of least privilege), how to prevent Broken Object Level
Authorization (BOLA), and why service-to-service calls use separate identity
(Client Credentials) — not the original user's token. This is staff-architect
thinking: each service is independently responsible for validating incoming
tokens and enforcing access rules in its domain.



### Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│ USER AUTHENTICATION FLOW (Browser / UI)                             │
│ Authorization Code + PKCE ← industry standard for SPAs               │
└─────────────────────────────────────────────────────────────────────┘

[Angular UI]
    │ (1) User clicks Login
    ▼
[Keycloak OIDC Provider]
    │ (2) Redirect to login page + code_challenge
    ├─→ User authenticates
    │ (3) Redirect back with auth_code
    ▼
[Angular UI — exchanges code for tokens]
    │ access_token (short-lived JWT, user claims)
    │ refresh_token (HTTP-only secure cookie)
    ▼
[product-service]  ── validates JWT (Keycloak JWKS)
                   ── RBAC + BOLA: buyer sees products, seller manages own
                   ── calls inventory-service with mTLS + Client Credentials
                   │
                   ├─→ [inventory-service]  ── validates Client Credentials JWT
                   │                         ── (never accepts user JWT)

[order-service]    ── validates JWT (Keycloak JWKS)
                   ── RBAC + BOLA: buyer creates/views own orders,
                   │                seller views orders for own products
                   ├─→ [product-service]    ── mTLS + Client Credentials
                   │                         ── (service identity, not user)
                   └─→ [inventory-service]  ── mTLS + Client Credentials

┌─────────────────────────────────────────────────────────────────────┐
│ SERVICE-TO-SERVICE AUTHENTICATION (mTLS + Client Credentials)       │
│ Each service holds its own long-lived identity at Keycloak          │
│ No user tokens cross service boundaries                             │
└─────────────────────────────────────────────────────────────────────┘
```

**Three independent auth domains:**
1. **User realm** — OAuth2 Authorization Code + PKCE. User logs in once;
   browser holds tokens. Every API call includes user's access token.
2. **Service realm** — OAuth2 Client Credentials. Each service holds a
   client_id + client_secret. Service fetches its own token to call peers.
3. **Transport security** — mTLS (cert-manager). Every TCP connection is
   mutually authenticated at the TLS layer — independent of OAuth2.



### Services & UI

| Component | Role | Auth requirement |
|---|---|---|
| **Angular UI** | Browser app hosted via Ingress HTTPS. User-driven flows: browse products, create orders. Initiates OAuth2 Authorization Code + PKCE flow with Keycloak. | User JWT (access_token) in Authorization header |
| **product-service** | Public API. Buyers browse products; sellers manage their own. Direct HTTPS from UI. | User JWT. Validates token; enforces RBAC (buyer vs seller) + BOLA (seller sees only own products) |
| **order-service** | Public API. Buyers create/view orders; sellers view orders for their products. Direct HTTPS from UI. | User JWT. Validates token; enforces RBAC + BOLA (buyer/seller access control per order) |
| **inventory-service** | Internal-only API. Called by product-service and order-service to check/reserve stock. Never called directly from UI. | Client Credentials JWT (service-to-service identity). Never accepts user tokens. mTLS required. |



### Angular UI — OAuth2 Authorization Code + PKCE

**Hosting:** Static SPA served via Kubernetes Ingress (HTTPS). No backend —
all authentication logic runs in the browser. Same-origin as product-service
and order-service for secure cookie mode.

**Why Authorization Code + PKCE (industry standard for SPAs):**
- Browser apps cannot securely store client secrets → PKCE replaces the secret
  with a one-time `code_verifier` generated in the browser
- `code_challenge` (hash of verifier) sent in the authorization request;
  `code_verifier` sent only in the token exchange — prevents interception
- Refresh tokens stored in secure HTTP-only cookies; access tokens kept in
  memory only — prevents XSS from stealing the access token

**Token storage strategy (security best practice):**
- **access_token** — in-memory only (JS variable). Short-lived (5–15 min).
  Expires and forces re-authentication if the browser tab is closed.
- **refresh_token** — secure HTTP-only cookie set by Keycloak. Long-lived
  (days/weeks). Browser sends it automatically; JS code cannot access it.
  Survives tab closure.
- **localStorage** — empty. No tokens stored here (vulnerable to XSS).

**Key user flows:**

1. **Login (S01-Initial)**
   - User clicks "Login" button
   - App generates random `code_verifier` and stores in sessionStorage
   - App computes `code_challenge = SHA256(code_verifier)` (PKCE)
   - Redirects to Keycloak: `/auth?client_id=web-client&code_challenge=...&redirect_uri=http://localhost:4200`
   - User enters credentials at Keycloak
   - Keycloak redirects back: `/callback?code=AUTH_CODE`
   - App exchanges AUTH_CODE + code_verifier for tokens:
     ```
     POST /token
     code=AUTH_CODE
     code_verifier=ORIGINAL_RANDOM_STRING
     client_id=web-client
     ```
   - Keycloak validates code_verifier against code_challenge; returns
     `access_token` + `refresh_token` (cookie)
   - App stores access_token in memory; redirects to dashboard

2. **API call (S02, S03, S04, etc.)**
   - App intercepts every HTTP request to product-service / order-service
   - HTTP Interceptor adds: `Authorization: Bearer {access_token}`
   - Service validates JWT signature against Keycloak JWKS
   - Service extracts user claims (subject, roles) and enforces RBAC + BOLA
   - Response returned to UI

3. **Token refresh (implicit, transparent)**
   - If API returns 401 Unauthorized (token expired)
   - HTTP Interceptor catches error
   - Browser automatically sends refresh_token cookie in next request
   - App exchanges refresh_token for new access_token
   - HTTP Interceptor retries original request with new token
   - User sees no interruption

4. **Logout (S10-UI logout)**
   - User clicks "Logout" button
   - App clears in-memory access_token
   - App sends request to Keycloak logout endpoint
   - Keycloak invalidates session server-side + clears cookies
   - Redirect to login page
   - Browser "back" button cannot restore session (refresh_token invalidated)



### Keycloak Setup — Two Separate OAuth2 Realms

**User Realm (Human-driven flows):**

```
Realm: demo-users

Client: web-client
  Flow: Authorization Code + PKCE (no client secret)
  Redirect URIs: https://ui.demo-iac.svc, http://localhost:4200
  Refresh token: Enabled (cookie, HTTP-only, Secure)
  Access token TTL: 5 minutes

Roles: buyer, seller

Test Users:
  alice        role: buyer     can browse products; can create/view own orders
  bob          role: seller    can create/manage products; can view orders for own products
  charlie      role: buyer     tests authorization rejection scenarios
```

**Service Realm (Machine-to-machine flows):**

```
Realm: demo-services

Service Clients:
  product-service
    Flow: Client Credentials
    Scopes: inventory:read, inventory:write
    Client ID: product-service
    Client secret: (Vault-managed, rotated, file-mounted as /etc/secrets/keycloak/client-secret)

  order-service
    Flow: Client Credentials
    Scopes: inventory:read, inventory:write
    Client ID: order-service
    Client secret: (Vault-managed, rotated, file-mounted)

  inventory-service
    NO CLIENT — internal service; only accepts inbound Client Credentials calls
    Does NOT initiate calls to other services

Scopes Defined:
  inventory:read   — check stock levels
  inventory:write  — reserve / release stock
```

**Why two realms?**
- **demo-users:** OAuth2 Authorization Code flow. Keycloak issues user tokens
  with user claims (subject=username, roles=[buyer|seller]). Browser refreshes
  these tokens automatically via refresh_token.
- **demo-services:** OAuth2 Client Credentials flow. Service calls Keycloak
  with client_id + client_secret; receives service token with service claims
  (subject=service-name, scopes=[...]). No refresh logic — service fetches new
  token when current one expires (Spring handles this automatically).

This reflects **industry best practice:** user and service tokens are issued
from separate logical namespaces to enforce different security policies.



### API Endpoints

**product-service** (Public — HTTPS only, no mTLS requirement for UI calls)
```
GET    /products              user JWT (buyer or seller)   → list all products
GET    /products/{productId}  user JWT                     → get one product
POST   /products              user JWT + seller role       → create product
PUT    /products/{productId}  user JWT + seller role +     → update own product only (BOLA)
                              verify product.seller_id == user.id
DELETE /products/{productId}  user JWT + seller role +     → delete own product only
                              verify product.seller_id == user.id
GET    /products/seller/{seller_id}  user JWT             → list seller's products (public)
```

**order-service** (Public — HTTPS only)
```
GET    /orders                user JWT                     → list own orders (buyer) or all
GET    /orders/{orderId}      user JWT                     → get order (BOLA: only if buyer
                                                              OR seller of a product in order)
POST   /orders                user JWT + buyer role        → create new order
PUT    /orders/{orderId}/status  user JWT + seller role    → update order status
                                (only for orders with own products)
GET    /orders/seller/{seller_id}  user JWT + seller role  → list orders for own products (BOLA)
```

**inventory-service** (Internal-only — mTLS required + Client Credentials JWT)
```
GET    /stock/{productId}     Client Credentials JWT       → check available stock
POST   /stock/{productId}/reserve  Client Credentials      → reserve N units (idempotent)
POST   /stock/{productId}/release  Client Credentials      → release N units (on order cancel)
```

**Key principle:** UI calls product-service and order-service directly.
Both public services call inventory-service internally with their own
Client Credentials — the original user's token is NOT forwarded.



### Pre-Requisite Infrastructure

| Component | Role in this demo |
|---|---|
| **cert-manager** | Common internal CA; issues server certs for all services + Keycloak. Client certs for inter-service mTLS. |
| **Keycloak** | OIDC + OAuth2 provider. Two realms: `demo-users` (Authorization Code + PKCE for browsers) and `demo-services` (Client Credentials for services). |
| **HashiCorp Vault** | Stores Client Credentials secrets (client_id + client_secret for each service). Authoritative source — secrets never checked into git. |
| **External Secrets Operator** | Syncs Vault secrets to Kubernetes Secrets; watched by Spring Cloud Config Watcher or Stakater Reloader. |
| **Stakater Reloader** | Watches service ConfigMaps and server TLS secrets; rolling restart on change. |

**cert-manager CA chain:**

```
cert-manager ClusterIssuer (internal CA)
  ├── keycloak-tls              ← Keycloak server cert (HTTPS, JWKS endpoint verification)
  │
  ├── product-service-tls       ← product-service server cert (HTTPS from UI)
  ├── product-service-client-tls ← client cert (product → inventory mTLS call)
  │
  ├── order-service-tls         ← order-service server cert (HTTPS from UI)
  ├── order-service-client-tls   ← client cert (order → inventory mTLS call)
  │
  └── inventory-service-tls     ← inventory-service server cert (mTLS only; never public HTTPS)
```

**Why mTLS for inventory-service?**
- Inventory is internal-only (not called directly from UI).
- Product-service and order-service identify themselves cryptographically
  at the TLS layer (independent of OAuth2 tokens).
- Even if a Client Credentials token is forged, the TLS handshake fails
  if the client cert is invalid.
- This is **defence in depth:** two independent authentication layers.



### ConfigMap & Secret Layout

Each service follows the standard four-resource ConfigMap pattern (from Demo 01).
Secrets split by rotation strategy: stable secrets (server TLS, trust stores)
trigger rolling restart; connection credentials (Client Credentials) use Config
Watcher hot-reload.

```
─── product-service ────────────────────────────────────────────────────────
ConfigMap: product-service          key: application.yaml       Stakater: ✅
ConfigMap: product-service-features key: features.yaml          Config Watcher
ConfigMap: product-service-logging  key: logback-spring.xml     Logback scan
ConfigMap: product-service-runtime-env  envFrom: JVM vars       Stakater: ✅

Secret: product-service-server-tls  cert-manager    Stakater: ✅ (rolling restart)
Secret: product-service-client-tls  cert-manager    Config Watcher (new mTLS to inventory)
Secret: product-keycloak-secret     Vault via ESO   Config Watcher (Client Credentials secret)
Secret: keycloak-ca                 cert-manager    Stakater: ✅ (JWT validation)

─── order-service ────────────────────────────────────────────────────────
ConfigMap: order-service            key: application.yaml       Stakater: ✅
ConfigMap: order-service-features   key: features.yaml          Config Watcher
ConfigMap: order-service-logging    key: logback-spring.xml     Logback scan
ConfigMap: order-service-runtime-env    envFrom: JVM vars       Stakater: ✅

Secret: order-service-server-tls    cert-manager    Stakater: ✅ (rolling restart)
Secret: order-service-client-tls    cert-manager    Config Watcher (new mTLS to inventory)
Secret: order-keycloak-secret       Vault via ESO   Config Watcher (Client Credentials secret)
Secret: keycloak-ca                 cert-manager    Stakater: ✅ (JWT validation)

─── inventory-service ──────────────────────────────────────────────────────
ConfigMap: inventory-service        key: application.yaml       Stakater: ✅
ConfigMap: inventory-service-logging key: logback-spring.xml    Logback scan
ConfigMap: inventory-service-runtime-env  envFrom: JVM vars     Stakater: ✅

Secret: inventory-service-tls       cert-manager    Stakater: ✅ (server cert for mTLS)
Secret: keycloak-ca                 cert-manager    Stakater: ✅ (validate inbound CC tokens)
```

**Vault-managed secrets (Client Credentials):**
- `product-service/client-id` and `product-service/client-secret`
- `order-service/client-id` and `order-service/client-secret`
- ESO syncs these to Kubernetes Secrets at fixed intervals
- Config Watcher fires on change → Spring re-initializes OAuth2 client
- No service restart needed (different from Demo 01's double-pool DB rotation)

**Key principle:** Keycloak trust store (`keycloak-ca`) changes rarely — goes
in rolling-restart bucket. Client secrets change frequently — hot-reload bucket.



### Key Implementation Notes — Framework-Agnostic Concepts

All code shown in **pseudo-code / architecture language**, applicable to any
OAuth2-compliant framework (Spring Boot, Express, FastAPI, Go, etc.).



#### 1. User JWT Validation (Authentication)

**Concept:** Every public service validates user tokens independently.
Keycloak exposes a JWKS endpoint; services cache and use it to verify
signatures.

```
Pseudocode — product-service receiving user request:

  1. Extract Authorization header: "Bearer {JWT}"
  2. Decode JWT header → get key ID (kid)
  3. Fetch JWKS from Keycloak (cached, refreshed periodically)
  4. Find public key in JWKS matching kid
  5. Verify JWT signature with public key
  6. If valid:
       Extract claims: subject (username), roles, iat, exp
       Check exp > now (not expired)
       Continue to business logic with authenticated user context
  7. If invalid:
       Return 401 Unauthorized
```

**Why signature verification?**
- Prevents token forgery (attacker cannot create a valid JWT without Keycloak's private key)
- Keycloak is the only issuer; all public keys in JWKS are Keycloak's

```yaml
# Framework config example (Spring Boot / Python FastAPI / etc.)
# All frameworks: point to Keycloak JWKS endpoint
keycloak:
  jwks-uri: https://keycloak.demo-identity.svc/realms/demo-users/protocol/openid-connect/certs
  realm: demo-users
```



#### 2. Authorization — RBAC + BOLA Prevention (Broken Object Level Authorization)

**RBAC (Role-Based Access Control):** User's JWT contains `roles` claim.
Service checks if user has required role before executing business logic.

```
Pseudocode — seller can only edit their own products:

  PUT /products/{productId}
    1. Validate user JWT (from step 1 above)
    2. Extract roles from JWT claims
    3. Check: user.roles.contains("seller") ?
         If no → return 403 Forbidden
    4. Fetch product from database
    5. Check BOLA: product.seller_id == user.id ?
         (user can only edit products they created)
         If no → return 403 Forbidden
    6. Update product
```

**BOLA prevention (Object Level Authorization):**
- Database entities have ownership fields (e.g., `product.seller_id`, `order.buyer_id`)
- Before returning or modifying data, service ALWAYS verifies the requesting
  user owns/can access that object
- This prevents attackers from guessing IDs and accessing other users' data
  (e.g., `/orders/999` should 403 if user didn't create order 999)

```
Pseudocode — buyer viewing orders (with BOLA):

  GET /orders/{orderId}
    1. Validate user JWT
    2. Extract subject (username) and roles
    3. Fetch order from database
    4. Authorize based on role:
         If role == "buyer" → check order.buyer_id == user.id
         If role == "seller" → check any product in order.products
                               has product.seller_id == user.id
         Otherwise → return 403 Forbidden
    5. Return order details
```

**Why not rely on 404 for access denial?**
- Returning 404 confirms the resource doesn't exist
- Returning 403 confirms existence but denies access
- Security best practice: if access is denied, always return 403 (not 404)
  to avoid confirming whether the object exists



#### 3. Client Credentials — Service-to-Service Identity

**Concept:** Services never forward user tokens to other services. Instead,
each service obtains its own long-lived token using Client Credentials flow.

```
Pseudocode — product-service calling inventory-service:

  When product-service starts:
    1. Load client_id and client_secret from mounted secrets
    2. Store them (in-memory, secured)

  When product-service needs to check inventory:
    1. Check if cached token is still valid (exp > now)
    2. If expired, request new token from Keycloak:
         POST /token
         client_id=product-service
         client_secret=SECRET
         grant_type=client_credentials
         scope=inventory:read
    3. Keycloak returns: {access_token: CC_TOKEN, expires_in: 3600}
    4. Cache token for 3600 seconds
    5. Call inventory-service with mTLS + Authorization: Bearer {CC_TOKEN}
```

**Why Client Credentials instead of forwarding user token?**

| Concern | User Token (Bad) | Client Credentials (Good) |
|---|---|---|
| **Scope** | User has full permissions on their data. Forwarding gives inventory-service access to all user's orders globally. | Service token has minimal scopes: only `inventory:read`, not user's full permissions. |
| **Audit** | Logs show "user alice did X" even though alice never called inventory directly. Confusing. | Logs show "product-service (on behalf of alice) did X". Clear. |
| **Revocation** | Revoking user's token revokes them everywhere, breaking service-to-service calls. | User and service tokens are independent. Revoking user doesn't affect service calls. |
| **TTL** | User token is short-lived (5–15 min). Services would exhaust refresh rate. | Service token is long-lived (60+ min). Services cache it efficiently. |

```
Architecture principle:

[User Token (from browser)]
  ↓
[product-service]  ← validates user token; extracts user context
  ↓
[Calls inventory-service]  ← sends CC_TOKEN, not user token
  ↓
[inventory-service]  ← validates CC_TOKEN; sees "product-service" not "alice"
                       checks scope (inventory:read), not user roles
```



#### 4. mTLS — Transport Layer Authentication

**Concept:** Every TCP connection between services is mutually authenticated
at the TLS layer. Certificates are issued by cert-manager and bound to
service identity.

```
Pseudocode — product-service initiating mTLS to inventory-service:

  When product-service starts:
    1. Load server certificate: /etc/certs/server/tls.crt, tls.key
    2. Load CA certificate: /etc/certs/ca.crt (to verify peer)

  When product-service connects to inventory-service:
    1. TLS handshake initiates
    2. Product-service presents its certificate (signed by internal CA)
    3. Inventory-service validates: certificate chain valid? CN=product-service?
    4. Inventory-service presents its certificate
    5. Product-service validates: certificate chain valid? CN=inventory-service?
    6. Both sides compute shared secret using certificates
    7. If either certificate is invalid or forged → TLS handshake fails;
       connection refused before HTTP layer
    8. Only after successful TLS handshake: HTTP layer (OAuth2 CC token validation)
```

**Why two layers (mTLS + OAuth2)?**
- **mTLS (transport):** Cryptographic identity. Proves "you are the inventory-service".
- **OAuth2 (application):** Authorization. Proves "you have permission to inventory:read".
- **Defence in depth:** If one layer is compromised (e.g., token forged), the other still protects.

```yaml
# Framework config example
mtls:
  server:
    cert-file: /etc/certs/server/tls.crt
    key-file: /etc/certs/server/tls.key
    port: 8443
  client:
    ca-cert: /etc/certs/ca.crt
    verify-hostname: true  # CN must match service name
```

**Angular UI — OAuth2 & JWT handling (auth guard + interceptor):**

```typescript
// auth.guard.ts — protects routes, redirects to login if no valid token
export class AuthGuard implements CanActivate {
  constructor(private authService: AuthService, private router: Router) {}

  canActivate(route: ActivatedRouteSnapshot): boolean {
    const token = this.authService.getAccessToken();
    if (token && !this.isTokenExpired(token)) {
      return true;
    }
    // Token missing or expired — initiate PKCE login flow
    this.authService.login();
    return false;
  }
}

// http.interceptor.ts — adds Bearer token to all api-facade requests
export class HttpInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<any> {
    const token = this.authService.getAccessToken();
    if (token) {
      req = req.clone({
        setHeaders: { Authorization: `Bearer ${token}` }
      });
    }
    return next.handle(req).pipe(
      catchError(error => {
        if (error.status === 401) {
          // Token expired — refresh and retry
          return this.authService.refreshToken().pipe(
            switchMap(() => {
              const newToken = this.authService.getAccessToken();
              const retried = req.clone({
                setHeaders: { Authorization: `Bearer ${newToken}` }
              });
              return next.handle(retried);
            }),
            catchError(() => {
              // Refresh failed — redirect to login
              this.authService.logout();
              return EMPTY;
            })
          );
        }
        return throwError(error);
      })
    );
  }
}

// auth.service.ts — PKCE flow with Keycloak
export class AuthService {
  private accessToken: string | null = null;

  login() {
    const codeVerifier = this.generateCodeVerifier();
    const codeChallenge = this.generateCodeChallenge(codeVerifier);
    sessionStorage.setItem('oidc_code_verifier', codeVerifier);

    const params = new URLSearchParams({
      response_type: 'code',
      client_id: 'web-client',
      redirect_uri: window.location.origin,
      scope: 'openid profile email',
      code_challenge: codeChallenge,
      code_challenge_method: 'S256'
    });

    window.location.href =
      `https://keycloak.demo-security.svc/realms/demo/protocol/openid-connect/auth?${params}`;
  }

  exchangeAuthCode(code: string) {
    const codeVerifier = sessionStorage.getItem('oidc_code_verifier');
    const body = new URLSearchParams({
      grant_type: 'authorization_code',
      code,
      client_id: 'web-client',
      redirect_uri: window.location.origin,
      code_verifier: codeVerifier
    });

    return this.http.post('/token-endpoint', body).pipe(
      tap(response => {
        this.accessToken = response.access_token;
        // Refresh token stored by Keycloak in secure HTTP-only cookie automatically
      })
    );
  }

  refreshToken(): Observable<any> {
    // Refresh token is in secure HTTP-only cookie — browser sends automatically
    return this.http.post('/token-endpoint', { grant_type: 'refresh_token' }).pipe(
      tap(response => {
        this.accessToken = response.access_token;
      })
    );
  }

  logout() {
    this.accessToken = null;
    // Keycloak invalidates server-side session + clears cookies
    window.location.href =
      `https://keycloak.demo-security.svc/realms/demo/protocol/openid-connect/logout?redirect_uri=${window.location.origin}`;
  }

  getAccessToken(): string | null {
    return this.accessToken;
  }
}
```

**Role-based UI rendering:**

```html
<!-- dashboard.component.html -->
<div *ngIf="user$ | async as user">
  <h1>Orders Dashboard — {{ user.name }}</h1>

  <!-- All users can create orders -->
  <button (click)="openCreateOrderDialog()" *ngIf="user.roles.includes('customer')">
    Create Order
  </button>

  <!-- Only order-manager can see all orders -->
  <div *ngIf="user.roles.includes('order-manager')">
    <button (click)="loadAllOrders()">View All Orders</button>
    <table *ngIf="allOrders">...</table>
  </div>

  <!-- All users see their own orders -->
  <div *ngIf="user.roles.includes('customer')">
    <button (click)="loadMyOrders()">My Orders</button>
    <table *ngIf="myOrders">...</table>
  </div>

  <button (click)="logout()">Logout</button>
</div>
```



**Kubernetes RBAC — minimum permissions, one ServiceAccount per service:**

```yaml
# Each service's Helm chart creates its own ServiceAccount and Role
# Example: api-facade Role in demo-security
rules:
  - apiGroups: [""]
    resources: ["configmaps", "secrets"]
    verbs: ["get", "list", "watch"]   # Spring Cloud K8s Config reads only
# No pod management, no Deployment access, no cross-namespace permissions
```



### Scenarios (12 core scenarios)

| # | Scenario | Concept | Expected outcome |
|---|---|---|---|
| **S01** | UI: User clicks "Login" → redirects to Keycloak → authenticates → redirected back with tokens | OAuth2 Authorization Code + PKCE | Browser stores access_token (memory), refresh_token (HTTP-only cookie); redirects to dashboard |
| **S02** | UI: Buyer calls GET /products → authorization header contains user JWT | User Authentication | HTTP 200; product list returned; JWT signature valid; user claims extracted |
| **S03** | UI: Seller creates product (POST /products with seller role) | User Authentication + RBAC | HTTP 201; product created with seller_id = user.id; BOLA enforced |
| **S04** | UI: Buyer creates order for seller's product (POST /orders) | User Authentication + RBAC | HTTP 201; order created with buyer_id = user.id; product_id linked |
| **S05** | Anonymous request: no Authorization header on GET /orders | Authentication failure | HTTP 401 Unauthorized; response includes `WWW-Authenticate: Bearer` |
| **S06** | Buyer alice calls GET /orders/{bob_order_id} (order created by bob) | BOLA prevention (authorization failure) | HTTP 403 Forbidden; order exists but user alice cannot access it |
| **S07** | Seller bob calls GET /orders (seller endpoint; should only see orders for own products) | RBAC + BOLA | HTTP 200; returns list of orders containing only bob's products (not all orders) |
| **S08** | Tampered JWT: attacker modifies token claims (e.g., role: seller) and resends | Token integrity verification | HTTP 401; signature validation fails; token rejected |
| **S09** | Expired JWT: access_token has exp < now | Token expiry check | HTTP 401; `error: invalid_token, error_description: Token expired` |
| **S10** | product-service calls inventory-service with Client Credentials JWT (mTLS + OAuth2) | Service-to-service identity (Client Credentials) | HTTP 200; inventory-service validates CC token, not user token; scopes checked (inventory:read) |
| **S11** | product-service → inventory-service WITHOUT valid client cert (TLS layer rejection) | mTLS transport authentication | TLS handshake fails before HTTP; connection refused at socket level |
| **S12** | Client Credentials secret rotated in Vault → ESO syncs → Config Watcher fires | Service credential rotation | Spring OAuth2 discards cached token; next inventory call fetches new token with new secret; zero failed requests |



### Scenario Runner

Runs as a Kubernetes Job in `demo-identity`. Obtains test tokens from Keycloak
(user tokens via Resource Owner Password grant for testing; service tokens via
Client Credentials flow). Verifies authentication and authorization at each layer.

```bash
kubectl apply -f deploy/scenario-runner-job.yaml -n demo-identity
kubectl wait --for=condition=complete job/scenario-runner -n demo-identity --timeout=12m
# copy Allure results from pod and serve
```

**Scenario execution strategy:**

- **S01:** UI test — Selenium/Cypress in-cluster; simulates browser login flow
- **S02–S04, S07:** REST calls with user JWT from test tokens
- **S05–S06:** Negative tests (missing/invalid/expired tokens)
- **S08–S09:** Token tampering (modify claims, set exp to past)
- **S10:** Direct REST call from scenario runner to product-service with CC token
- **S11:** Raw TLS socket (no mTLS cert) → assert handshake failure at transport layer
- **S12:** Patch Vault secret → wait for ESO/Config Watcher → retry inventory call

---

## Demo 03 — API Gateway

**Repo:** `ms-api-gateway-demo` · **Namespace:** `demo-gateway`

**Focus:** Move cross-cutting API concerns (rate limiting, CORS, secure headers,
request validation, routing) out of application code and into a platform-level
gateway. Applications stay focused on business logic; the gateway owns the edge
security posture and traffic-shaping policies.

**What a reader takes away:** Which concerns belong at the edge vs in the app,
how to configure a gateway declaratively (GitOps-friendly), and why every
service in a real system sits behind one.



### Services

`echo-service` — minimal HTTP echo backend. Returns request headers, path,
method, and body as JSON. Proves the gateway is transforming requests correctly.

`product-service-lite` — REST service with 3 endpoints. Demonstrates
route-specific policies (stricter rate limits, request validation, JWT enforcement).

Commons used: `commons-logging` `commons-web` `commons-observability`



### Pre-Requisite Infrastructure

| Component | Role |
|---|---|
| **Kong Gateway** (OSS, Kubernetes Ingress Controller) | The gateway itself; declarative config via CRDs |
| **cert-manager** | TLS cert for gateway HTTPS listener |
| **Redis** | Distributed rate-limit counter store (multi-replica safe) |
| **Keycloak** (from Demo 02) | JWT issuer for the JWT enforcement scenarios |

Kong deployed via Helm. Route + plugin config lives in Git; applied via ArgoCD.
Alternative gateway (equally valid): **Traefik** — simpler config, K8s-native.
Kong chosen for its richer plugin ecosystem.



### Gateway Configuration Layout

```
Route: /api/echo           → echo-service
  Plugins:
    - rate-limiting        100 req/min per client IP (Redis-backed)
    - cors                 allowed-origins: https://ui.example.com
    - response-transformer add security headers (HSTS, CSP, X-Frame-Options, X-Content-Type-Options)

Route: /api/products       → product-service-lite
  Plugins:
    - rate-limiting        20 req/min per client IP (stricter)
    - jwt                  validate JWT via Keycloak JWKS
    - request-validator    OpenAPI schema on request body
    - cors, response-transformer  (same as /api/echo)
```



### Key Implementation Notes

**Plugins are declarative — zero application code:**

```yaml
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: rate-limit-100-per-min
plugin: rate-limiting
config:
  minute: 100
  policy: redis
  redis_host: redis.demo-gateway.svc

apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: secure-headers
plugin: response-transformer
config:
  add:
    headers:
      - "Strict-Transport-Security:max-age=31536000; includeSubDomains"
      - "Content-Security-Policy:default-src 'self'"
      - "X-Frame-Options:DENY"
      - "X-Content-Type-Options:nosniff"
```

**Attach plugins to a route via Ingress annotation:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: echo-service
  annotations:
    konghq.com/plugins: rate-limit-100-per-min,secure-headers,cors
spec:
  ingressClassName: kong
  rules:
    - host: api.demo-gateway.local
      http:
        paths:
          - path: /api/echo
            pathType: Prefix
            backend:
              service:
                name: echo-service
                port: { number: 8080 }
```

**Request validation via OpenAPI schema (gateway rejects before app sees it):**

```yaml
plugin: request-validator
config:
  body_schema: |
    { "type": "object",
      "required": ["name","price"],
      "properties": {
        "name":  { "type": "string", "maxLength": 100 },
        "price": { "type": "number", "minimum": 0 }
      } }
```



### Scenarios

| # | Scenario | Concept | Expected outcome |
|---|---|---|---|
| **S01** | `GET /api/echo` — check response headers | Security header injection | Response contains `Strict-Transport-Security`, `X-Frame-Options: DENY`, `Content-Security-Policy`, `X-Content-Type-Options: nosniff` |
| **S02** | Cross-origin request from `https://evil.com` | CORS deny | Preflight fails; `Access-Control-Allow-Origin` header absent; browser blocks |
| **S03** | Cross-origin request from `https://ui.example.com` | CORS allow | Response includes `Access-Control-Allow-Origin: https://ui.example.com` |
| **S04** | One client sends 101 requests within 60 s to `/api/echo` | Per-IP rate limiting | First 100 → HTTP 200; 101st → HTTP 429 with `Retry-After: 60` |
| **S05** | Two clients each send 100 requests within 60 s | Rate-limit isolation | Both succeed; buckets tracked independently in Redis |
| **S06** | 3 Kong replicas serving traffic; run S04 across all pods | Distributed rate limiting | Aggregate counter consistent across pods; still exactly 100 succeed |
| **S07** | `POST /api/products` body: `{"name":"x","price":-5}` | Request validation | HTTP 400 from gateway; product-service-lite never invoked (verified in logs) |
| **S08** | `POST /api/products` body: valid | Request validation happy path | HTTP 201; product created |
| **S09** | `GET /api/products` without `Authorization` header | JWT enforcement at edge | HTTP 401 from gateway; backend never reached |
| **S10** | `GET /api/products` with valid JWT from Keycloak | JWT enforcement happy path | HTTP 200; backend receives request with JWT claims propagated in headers |



### Scenario Runner

Runs as a Kubernetes Job in `demo-gateway`. Uses REST-assured for HTTP
assertions; parallel HTTP clients drive the rate-limit scenarios. Rate-limit
verification queries Redis directly to prove counter state.

```bash
kubectl apply -f deploy/scenario-runner-job.yaml -n demo-gateway
kubectl wait --for=condition=complete job/scenario-runner -n demo-gateway --timeout=8m
```

---

## Demo 04 — Database Migrations

**Repo:** `ms-database-migrations-demo` · **Namespace:** `demo-migrations`

**Focus:** Structured schema evolution with Liquibase. Track DDL/DML changes as versioned migration scripts, enforce migration-before-startup via init containers, and handle rollbacks and checksum violations.

**What a reader takes away:** How to version database changes alongside application deployments, why init containers enforce ordering, and how Liquibase prevents partial schema state.



### Service

`product-catalog-service` — CRUD API over a product table that evolves through 5 schema versions during the demo.

```
GET  /products        → list products
GET  /products/{id}   → get product
POST /products        → create product
```

Commons used: `commons-datasource` `commons-logging` `commons-web`



### Pre-Requisite Infrastructure

| Component | Role |
|---|---|
| **PostgreSQL** (CloudNativePG) | Target database; cert-manager TLS (same pattern as Demo 01) |



### Migration Layout

```
db/changelog/
  db.changelog-master.yaml          ← root changelog (includes all versioned files)
  v1__create_products.sql           ← initial schema
  v2__add_description_column.sql    ← add nullable column
  v3__add_category_index.sql        ← performance index
  v4__seed_reference_data.sql       ← DML: insert product categories
  v5__rename_price_column.sql       ← rename (requires rollback script)
  v5__rename_price_column.rollback.sql
```

**Init container pattern (migration before app start):**

```yaml
initContainers:
  - name: migrate
    image: liquibase/liquibase:latest
    command:
      - liquibase
      - --changelog-file=db/changelog/db.changelog-master.yaml
      - --url=jdbc:postgresql://postgres:5432/catalog
      - update
    envFrom:
      - secretRef: { name: db-credentials }
containers:
  - name: product-catalog-service
    # starts only after init container exits 0
```

**Concurrent safety:** Liquibase uses `DATABASECHANGELOGLOCK` table — concurrent init containers block on DB row lock until one finishes, then others skip already-applied changesets.



### Scenarios

| # | Scenario | Concept | Expected outcome |
|---|---|---|---|
| **S01** | Fresh deploy — all 5 migrations run in order | Initial migration | `DATABASECHANGELOG` shows 5 entries with author, checksum, executed timestamp |
| **S02** | Deploy v2 app only — only new migrations run | Incremental migration | Only migrations not yet applied execute; previously applied entries untouched (checksum match) |
| **S03** | 3 pods start simultaneously at fresh deploy | Concurrent migration locking | Exactly one pod acquires lock and runs migrations; others wait and proceed after; no duplicate execution |
| **S04** | Rollback v5 (column rename) to tag `v4` | Schema rollback | `DATABASECHANGELOG` shows v5 rolled back; original column name restored |
| **S05** | Modify v1 changeset SQL after it was already applied | Checksum violation | Liquibase refuses to start (`ValidationFailedException`); init container fails; app never starts |
| **S06** | Seed reference data via DML migration (v4) | Data migration | Categories table populated; verified via GET /products/categories |



### Scenario Runner

```bash
kubectl apply -f deploy/scenario-runner-job.yaml -n demo-migrations
kubectl wait --for=condition=complete job/scenario-runner -n demo-migrations --timeout=10m
```

---

## Demo 05 — Caching

**Repo:** `ms-caching-demo` · **Namespace:** `demo-caching`

**Focus:** Redis caching strategies: cache-aside, write-through, TTL-based invalidation, and thundering herd (stampede) prevention. Observable cache effectiveness via Grafana.

**What a reader takes away:** Which caching pattern suits each access mode, how to prevent stampede under high concurrency, and how to observe cache hit/miss ratios.



### Service

`product-service` — read-heavy product catalog with configurable caching strategy per endpoint.

```
GET  /products/{id}         → cache-aside: check Redis → miss → DB → populate
GET  /products/featured     → write-through cached: always consistent
PUT  /products/{id}         → write-through: update DB + invalidate cache
GET  /products/hot          → demonstrates stampede: artificial 200ms DB delay
```

Commons used: `commons-datasource` `commons-logging` `commons-web` `commons-observability`



### Pre-Requisite Infrastructure

| Component | Role |
|---|---|
| **Redis** | Cache layer |
| **PostgreSQL** | Source of truth |
| **Prometheus + Grafana** | Cache hit/miss ratio dashboard (Micrometer counters) |



### Key Implementation Notes

**Cache-aside:**

```
Read:
  1. GET Redis key product:{id}
  2. HIT  → return cached value; increment cache_hit counter
  3. MISS → query DB → SET Redis key product:{id} EX 60 → return; increment cache_miss counter

Write:
  1. UPDATE PostgreSQL
  2. DEL Redis key product:{id}    ← invalidate (not update — avoids write race)
```

**Stampede prevention (distributed lock on miss):**

```
On MISS:
  SETNX lock:product:{id} {pod-id} PX 5000    ← atomic; only one winner
  If acquired → fetch DB → SET cache → DEL lock
  If not acquired → sleep 50ms → re-read cache (populated by lock holder)
```



### Scenarios

| # | Scenario | Concept | Expected outcome |
|---|---|---|---|
| **S01** | Cold cache: 100 GETs for same product ID | Cache miss path | All 100 hit PostgreSQL on first request; Redis populated after first; Grafana: 0% → rising hit ratio |
| **S02** | Warm cache: repeat same 100 requests | Cache hit path | All 100 served from Redis; zero DB queries; lower latency observable in Grafana |
| **S03** | PUT /products/{id} → GET /products/{id} | Cache invalidation | First GET post-PUT is a cache miss; returns updated value |
| **S04** | Wait 60s after warming cache; GET /products/{id} | TTL expiry | Cache miss; DB queried; cache repopulated |
| **S05** | 100 concurrent GETs for same uncached product (200ms DB delay) | Stampede prevention | Without lock: 100 DB queries. With lock: ≤ 2 DB queries; rest wait and read from cache. Assert DB query count ≤ 2. |
| **S06** | Kill Redis pod → GET /products/{id} | Graceful degradation | Service falls back to PostgreSQL; returns 200 (not 500); `cache_errors_total` metric increments |
| **S07** | Grafana dashboard | Observability | Hit ratio, miss ratio, DB query count, per-operation latency comparison visible |



### Scenario Runner

```bash
kubectl apply -f deploy/scenario-runner-job.yaml -n demo-caching
kubectl wait --for=condition=complete job/scenario-runner -n demo-caching --timeout=8m
```

---

## Demo 06 — Observability & Monitoring

**Repo:** `ms-observability-demo` · **Namespace:** `demo-observability`

**Focus:** Three observability pillars (metrics, logs, traces) with OpenTelemetry as the collection layer and the Grafana stack (Prometheus, Loki, Tempo) as backends. Zero additional instrumentation code in services — OpenTelemetry Java agent handles it automatically. Alerting on SLO breach.

**What a reader takes away:** How to correlate a log line → trace → metric without touching service code, and how to define alerts that fire before users notice degradation.



### Services

`order-service` — creates orders; calls notification-service.
`notification-service` — downstream dependency; failures injected on demand.

Commons used: `commons-logging` `commons-web` `commons-observability`



### Pre-Requisite Infrastructure

| Component | Role |
|---|---|
| **OpenTelemetry Collector** | Receives OTLP from services; routes metrics → Prometheus, traces → Tempo, logs → Loki |
| **Prometheus** | Metrics storage + alerting rules |
| **Loki** | Log aggregation (structured JSON from `commons-logging`) |
| **Tempo** | Distributed trace storage |
| **Grafana** | Unified dashboard: metrics + logs + traces with correlation links |
| **Alertmanager** | Routes fired alerts to notification channel |

**OpenTelemetry Java agent** attached via `JAVA_TOOL_OPTIONS` in the `order-runtime-env` ConfigMap — no source code change.



### Key Implementation Notes

**Zero-code auto-instrumentation:**

```
ConfigMap: order-runtime-env
  JAVA_TOOL_OPTIONS: -javaagent:/opt/otel/opentelemetry-javaagent.jar
  OTEL_SERVICE_NAME: order-service
  OTEL_EXPORTER_OTLP_ENDPOINT: http://otel-collector.demo-observability.svc:4317
  OTEL_LOGS_EXPORTER: otlp
  OTEL_METRICS_EXPORTER: otlp
  OTEL_TRACES_EXPORTER: otlp
```

**Trace correlation:** OTel agent injects `traceparent` header on outbound calls and extracts on inbound — spans linked across services automatically. `commons-logging` includes `traceId` in every JSON log line.

**Grafana correlation:** Click a log line's `traceId` field → Grafana navigates directly to the matching Tempo trace.

**Custom business metric (3 lines, optional):**

```java
meterRegistry.counter("orders.created", "status", "success").increment();
```

**SLO alert (Prometheus rule):**

```yaml
- alert: HighErrorRate
  expr: >
    rate(http_server_requests_seconds_count{status=~"5.."}[5m])
    / rate(http_server_requests_seconds_count[5m]) > 0.05
  for: 2m
  annotations:
    summary: "Error rate > 5% for 2 minutes"
```



### Scenarios

| # | Scenario | Concept | Expected outcome |
|---|---|---|---|
| **S01** | POST /orders → happy path | Trace propagation | Tempo shows span tree: order-service → notification-service; latency per span |
| **S02** | Inject 2s delay in notification-service | Latency trace | Tempo: notification-service span is 2s of total; order-service waiting span visible |
| **S03** | notification-service returns 500 | Error in all three pillars | Tempo: span ERROR; Loki: error log with same traceId; Prometheus: error counter increments |
| **S04** | Error rate > 5% sustained for 2 min | Alert firing | Alertmanager fires `HighErrorRate`; alert visible in Grafana Alerting panel |
| **S05** | Click log line traceId in Loki | Log → trace correlation | Grafana navigates to Tempo trace for that exact request |
| **S06** | Load test: 500 req/min for 5 min | Business metric + SLI | `orders.created` counter rising; p50/p95/p99 latency heatmap in Grafana |
| **S07** | GET /actuator/prometheus | Metrics exposure | JVM, HTTP, DB pool, and custom metrics visible; scraped by Prometheus |



### Scenario Runner

```bash
kubectl apply -f deploy/scenario-runner-job.yaml -n demo-observability
kubectl wait --for=condition=complete job/scenario-runner -n demo-observability --timeout=10m
```

---

## Demo 07 — REST API Design

**Repo:** `ms-rest-api-demo` · **Namespace:** `demo-rest`

**Focus:** Production REST API design: URL versioning, cursor-based pagination, filtering/sorting, RFC 7807 problem details, HATEOAS links, and OpenAPI contract-first development.

**What a reader takes away:** The right patterns for REST API evolution — versioning that doesn't break existing clients, pagination that works at scale, and error responses that clients can programmatically handle.



### Service

`product-api` — two API versions. v1 is the baseline; v2 introduces a breaking change (field rename) while v1 stays working.

```
GET  /v1/products                            → paginated list (page/size)
GET  /v1/products/{id}                       → field: "unitPrice"
GET  /v2/products                            → paginated list (cursor-based)
GET  /v2/products/{id}                       → field renamed: "price" (breaking change)
POST /v1/products                            → create; RFC 7807 on validation errors
GET  /v1/products?category=X&sort=price:asc  → filter + sort
```

Commons used: `commons-web` `commons-logging` `commons-observability`



### Pre-Requisite Infrastructure

| Component | Role |
|---|---|
| **PostgreSQL** | Product data |
| **OpenAPI Generator** (build time) | Generates server stubs from `openapi.yaml` — contract-first |



### Key Implementation Notes

**Cursor-based pagination (v2):**

```json
GET /v2/products?limit=20&cursor=eyJpZCI6NTB9
{
  "data": [...],
  "pagination": {
    "nextCursor": "eyJpZCI6NzB9",
    "hasMore": true
  }
}
```

Cursor = base64-encoded last-seen ID. Stable even when rows are inserted between pages (unlike offset pagination).

**RFC 7807 error (commons-web provides automatically):**

```json
{
  "type": "https://api.example.com/errors/validation-failed",
  "title": "Validation Failed",
  "status": 422,
  "detail": "price must be positive",
  "instance": "/v1/products",
  "errors": [{ "field": "price", "message": "must be greater than 0" }]
}
```

**HATEOAS links in response:**

```json
{
  "id": 1, "name": "Widget", "price": 9.99,
  "_links": {
    "self":     { "href": "/v1/products/1" },
    "update":   { "href": "/v1/products/1", "method": "PUT" },
    "category": { "href": "/v1/categories/widgets" }
  }
}
```



### Scenarios

| # | Scenario | Concept | Expected outcome |
|---|---|---|---|
| **S01** | GET /v1/products?page=1&size=10 | Offset pagination | 10 products; `X-Total-Count` header present |
| **S02** | GET /v2/products?limit=10 → follow nextCursor | Cursor pagination | Consistent pages even if rows inserted between requests |
| **S03** | GET /v1/products/{id} vs GET /v2/products/{id} | API versioning | v1 returns `unitPrice`; v2 returns `price`; both active simultaneously; no breakage |
| **S04** | POST /v1/products with missing required field | RFC 7807 error | HTTP 422; response matches RFC 7807 schema; `errors[]` has field-level detail |
| **S05** | GET /v1/products?category=electronics&sort=price:desc | Filter + sort | Only electronics returned; sorted descending; SQL WHERE + ORDER BY correct |
| **S06** | GET /v1/products/{id} response | HATEOAS | `_links.self`, `_links.update` present; clients navigate without hardcoded URLs |
| **S07** | GET /v3/products | Unknown API version | HTTP 404 with clear error; not 500 |



### Scenario Runner

```bash
kubectl apply -f deploy/scenario-runner-job.yaml -n demo-rest
kubectl wait --for=condition=complete job/scenario-runner -n demo-rest --timeout=8m
```

---

## Demo 08 — gRPC API

**Repo:** `ms-grpc-api-demo` · **Namespace:** `demo-grpc`

**Focus:** Protocol Buffers schema-first service design. Unary, server streaming, and bidirectional streaming RPC. Client-side load balancing via Kubernetes headless service. Proto backward compatibility.

**What a reader takes away:** When gRPC is the right choice over REST (typed contracts, streaming, high-throughput internal services), how load balancing differs from HTTP/1.1, and how to evolve proto schemas safely.



### Services

`inventory-service` — gRPC server (proto-defined).
`order-service` — gRPC client; calls inventory-service.

Commons used: `commons-logging` `commons-observability`



### Pre-Requisite Infrastructure

| Component | Role |
|---|---|
| **cert-manager** | TLS for gRPC (HTTP/2 + TLS) |
| **Buf CLI** (CI) | Proto linting + breaking change detection |



### Proto Schema

```protobuf
syntax = "proto3";
package inventory.v1;

service InventoryService {
  rpc GetStock(GetStockRequest)      returns (StockLevel);                              // Unary
  rpc WatchStock(WatchStockRequest)  returns (stream StockUpdate);                     // Server streaming
  rpc ReserveItems(stream ReservationRequest) returns (stream ReservationResult);      // Bidirectional
}

message GetStockRequest    { string product_id = 1; }
message StockLevel         { string product_id = 1; int32 available = 2; }
message WatchStockRequest  { string product_id = 1; }
message StockUpdate        { string product_id = 1; int32 available = 2; int64 timestamp = 3; }
message ReservationRequest { string product_id = 1; int32 quantity = 2; string order_id = 3; }
message ReservationResult  { string order_id = 1; bool success = 2; string reason = 3; }
```



### Scenarios

| # | Scenario | Concept | Expected outcome |
|---|---|---|---|
| **S01** | order-service calls GetStock(productId) | Unary RPC | StockLevel returned; binary Protobuf on the wire (not JSON) |
| **S02** | order-service calls WatchStock → receives 5 stock updates | Server streaming | Stream stays open; updates pushed as they occur; closed by server |
| **S03** | order-service streams 3 ReservationRequests; inventory streams results | Bidirectional streaming | Full-duplex; responses interleaved with requests |
| **S04** | inventory-service pod killed mid-stream | Retry on UNAVAILABLE | gRPC client retries with backoff; succeeds on new pod |
| **S05** | `grpcurl` reflection — list methods without proto file | gRPC server reflection | Methods and shapes enumerable from live server |
| **S06** | 3 inventory-service pods (headless K8s service); 100 RPCs | Client-side load balancing | Requests distributed across all 3 pods; per-pod counters confirm balance |
| **S07** | Add optional field to StockLevel proto; old client reads response | Backward compatibility | Old client ignores unknown field; no deserialization error |



### Scenario Runner

```bash
kubectl apply -f deploy/scenario-runner-job.yaml -n demo-grpc
kubectl wait --for=condition=complete job/scenario-runner -n demo-grpc --timeout=8m
```

---

## Demo 09 — GraphQL API

**Repo:** `ms-graphql-api-demo` · **Namespace:** `demo-graphql`

**Focus:** Schema-first GraphQL design. N+1 prevention via DataLoader, subscriptions over WebSocket, field-level authorization, and partial error responses.

**What a reader takes away:** When GraphQL reduces over-fetching and round trips vs REST, why DataLoader is mandatory for nested entities, and how partial responses with field-level errors keep clients resilient.



### Service

`storefront-api` — GraphQL API over products and orders domain.

Commons used: `commons-logging` `commons-web` `commons-observability`



### Pre-Requisite Infrastructure

| Component | Role |
|---|---|
| **PostgreSQL** | Products + orders |



### GraphQL Schema (SDL)

```graphql
type Product {
  id: ID!
  name: String!
  price: Float!
  seller: Seller!          # nested → N+1 without DataLoader
}

type Seller {
  id: ID!
  name: String!
  marginPercent: Float     # @auth(role: SELLER) — hidden from buyers
}

type Query {
  products(filter: ProductFilter, limit: Int, cursor: String): ProductConnection!
  order(id: ID!): Order
}

type Mutation { createProduct(input: CreateProductInput!): Product! }

type Subscription { orderStatusChanged(orderId: ID!): Order! }
```



### Scenarios

| # | Scenario | Concept | Expected outcome |
|---|---|---|---|
| **S01** | Query 20 products with `seller { name }` — DataLoader OFF | N+1 problem | 21 DB queries (1 products + 20 seller lookups); visible in query log |
| **S02** | Same query — DataLoader ON | N+1 prevention | 2 DB queries (1 products batch + 1 sellers batch); identical response |
| **S03** | Query with `{ products { unknownField } }` | Schema validation | HTTP 400 before execution; unknown field listed in error; no DB query |
| **S04** | Mutation: createProduct; immediately query it | Mutation + query | Product created and queryable in same request cycle |
| **S05** | Subscription: orderStatusChanged → update order status | Real-time subscription | WebSocket client receives update within 500ms of status change |
| **S06** | Buyer queries `seller { marginPercent }` | Field-level authorization | Field returns `null`; error in `errors[]` array; rest of response intact (partial response) |
| **S07** | Introspection query | Schema introspection | Full schema enumerable; flag demo: introspection disabled in "prod" config |



### Scenario Runner

```bash
kubectl apply -f deploy/scenario-runner-job.yaml -n demo-graphql
kubectl wait --for=condition=complete job/scenario-runner -n demo-graphql --timeout=8m
```

---

## Demo 10 — Real-time API

**Repo:** `ms-realtime-api-demo` · **Namespace:** `demo-realtime`

**Focus:** WebSocket (full-duplex) vs SSE (server-push) trade-offs for real-time order status updates. Multi-pod fan-out via Redis pub/sub. Backpressure handling and reconnection with event replay.

**What a reader takes away:** When to use SSE vs WebSocket, why stateful connections need a pub/sub fan-out layer in multi-pod deployments, and how to handle slow consumers without OOM.



### Service

`realtime-service` — both WebSocket and SSE endpoints for order status.

```
GET /sse/orders/{id}/status   → SSE stream: text/event-stream
WS  /ws/orders/{id}/status    → WebSocket: bidirectional, client acks each update
```

Commons used: `commons-logging` `commons-web` `commons-observability`



### Pre-Requisite Infrastructure

| Component | Role |
|---|---|
| **Redis** | Pub/sub fan-out across realtime-service pods |
| **cert-manager** | WSS/HTTPS TLS |



### Key Implementation Notes

**Why Redis pub/sub for multi-pod:**

```
Without Redis:
  Client A → Pod 1.  Event arrives at Pod 2.  Client A never sees it.

With Redis:
  Event → Redis channel → all pods subscribed → each forwards to its connected clients
```

**SSE reconnection (Last-Event-ID):**

```
Server: id: 42\ndata: {"status":"SHIPPED"}\n\n
Client disconnects. Reconnects with header: Last-Event-ID: 42
Server: replays events > 42 from in-memory ring buffer
```

**Backpressure:** Bounded buffer per subscriber (100 events). On overflow: drop oldest + increment `slow_consumer_drops` metric.



### Scenarios

| # | Scenario | Concept | Expected outcome |
|---|---|---|---|
| **S01** | SSE: subscribe; order transitions PENDING → CONFIRMED → SHIPPED | SSE server push | 3 events received in order; no polling; connection stays open |
| **S02** | SSE: disconnect at event 3; reconnect with `Last-Event-ID: 3` | Reconnection + replay | Events 4+ received; no duplicates; no missed events |
| **S03** | WebSocket: client acks each event; server sends next on ack | Bidirectional ack flow | Full-duplex; server respects backpressure via explicit ack protocol |
| **S04** | Fill subscriber buffer to 100; continue publishing | Backpressure | After 100 buffered: oldest dropped; `slow_consumer_drops` counter increments |
| **S05** | Publish event to Pod A; subscriber connected to Pod B | Multi-pod fan-out | Subscriber on Pod B receives event; Redis bridges the pods |



### Scenario Runner

```bash
kubectl apply -f deploy/scenario-runner-job.yaml -n demo-realtime
kubectl wait --for=condition=complete job/scenario-runner -n demo-realtime --timeout=8m
```

---

## Demo 11 — Service Mesh (Istio Ambient Mode)

**Repo:** `ms-service-mesh-demo` · **Namespace:** `demo-mesh`

**Focus:** Istio ambient mode (ztunnel for L4 mTLS, waypoint proxy for L7 policies — no sidecars). Covers four cross-cutting concerns in one mesh: resilience (retries, circuit breaking, timeouts), traffic management (canary, A/B routing), zero-trust security (AuthorizationPolicy, mTLS enforcement), and code-agnostic observability. Zero application code changes.

**What a reader takes away:** How a single Istio deployment eliminates entire categories of app-level code, and why ambient mode is operationally preferred over sidecar injection.



### Services

`product-service` v1 and v2 (canary/A/B traffic split).
`order-service` — calls product-service and inventory-service.
`inventory-service` — internal; AuthorizationPolicy restricts callers.

Commons used: `commons-logging` `commons-web`



### Pre-Requisite Infrastructure

| Component | Role |
|---|---|
| **Istio** (ambient mode) | ztunnel (per-node L4) + waypoint proxy (per-namespace L7) |
| **Kiali** | Service topology + traffic visualization |
| **Prometheus + Grafana** | Metrics from Istio telemetry (no app code) |
| **Tempo / Jaeger** | Distributed traces from Istio span propagation |



### Key Implementation Notes

**Ambient mode — no sidecars:**

```
Sidecar mode: Envoy proxy injected into each pod (memory overhead per pod)
Ambient mode:
  ztunnel: per-node DaemonSet; handles L4 mTLS + basic telemetry
  waypoint proxy: per-namespace; handles L7 routing, retries, authorization
  App pods: no annotation, no restart — mesh policies applied transparently
```

**Zero-trust: deny-all then allow-list:**

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata: { name: deny-all }
spec: {}          # no rules = deny everything by default

apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata: { name: allow-order-to-inventory }
spec:
  selector:
    matchLabels: { app: inventory-service }
  rules:
    - from:
        - source:
            principals: ["cluster.local/ns/demo-mesh/sa/order-service"]
```

**Canary routing (no app code):**

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata: { name: product-service }
spec:
  http:
    - route:
        - destination: { host: product-service, subset: v1 }
          weight: 90
        - destination: { host: product-service, subset: v2 }
          weight: 10
```



### Scenarios

| # | Scenario | Concept | Expected outcome |
|---|---|---|---|
| **S01** | 100 requests to product-service; check distribution | Canary 90/10 | ~90 served by v1, ~10 by v2; VirtualService weight controls split; no app changes |
| **S02** | Request with `X-Version: v2` header | A/B header-based routing | 100% routed to v2 regardless of weight |
| **S03** | inventory-service returns 500; observe order-service | Circuit breaker | DestinationRule outlier detection ejects unhealthy pod; traffic rerouted automatically |
| **S04** | inventory-service sleeps 5s; order-service has 2s VirtualService timeout | Timeout enforcement | order-service returns 504 after 2s; zero app code change |
| **S05** | Flaky GET /products (idempotent) | Automatic retry | Istio retries up to 3×; client sees 200; retry count in Kiali |
| **S06** | product-service (not order-service) calls inventory-service | AuthorizationPolicy deny | HTTP 403 at mesh layer; order-service still permitted |
| **S07** | Non-mesh pod calls inventory-service (no SPIFFE identity) | mTLS + PeerAuthentication | Connection refused before HTTP layer; no identity = no access |
| **S08** | Inject 50% HTTP 500 faults on product-service | Fault injection (chaos) | 50% of product requests return 500; Kiali error rate graph shows it; zero code change |
| **S09** | End-to-end: UI → order-service → product-service → inventory-service | Code-agnostic distributed trace | Tempo/Jaeger shows span tree across all 3 hops; no instrumentation code in apps |



### Scenario Runner

```bash
kubectl apply -f deploy/scenario-runner-job.yaml -n demo-mesh
kubectl wait --for=condition=complete job/scenario-runner -n demo-mesh --timeout=12m
```

---

## Demo 12 — Scaling & High Availability

**Repo:** `ms-scaling-demo` · **Namespace:** `demo-scaling`

**Focus:** Kubernetes-native scaling: CPU-based HPA, event-driven KEDA (scale from zero), Pod Disruption Budgets, and ArgoCD Rollout with automated analysis (auto-rollback on regression).

**What a reader takes away:** The right scaling tool per trigger (CPU vs event-driven), how PDBs protect availability during node maintenance, and how progressive delivery prevents bad deployments from reaching full traffic.



### Services

`api-service` — scales on CPU (HPA).
`order-processor` — scales on RabbitMQ queue depth (KEDA, zero-to-N capable).

Commons used: `commons-logging` `commons-web` `commons-messaging` `commons-observability`



### Pre-Requisite Infrastructure

| Component | Role |
|---|---|
| **KEDA** | Event-driven autoscaler; ScaledObject CRD |
| **metrics-server** | CPU/memory metrics for HPA |
| **RabbitMQ** | Queue depth drives KEDA |
| **Argo Rollouts** | Progressive delivery with analysis |
| **Prometheus** | Error rate metric for rollout analysis |



### Key Implementation Notes

**KEDA ScaledObject:**

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
spec:
  scaleTargetRef: { name: order-processor }
  minReplicaCount: 0        # scale to zero when queue empty
  maxReplicaCount: 10
  triggers:
    - type: rabbitmq
      metadata:
        queueName: orders
        queueLength: "5"    # 1 pod per 5 queued messages
```

**ArgoCD Rollout with analysis:**

```yaml
# If error rate > 5% during canary phase → automatic rollback
analysis:
  templates: [{ templateName: error-rate-check }]
  args: [{ name: error-rate-threshold, value: "0.05" }]
```



### Scenarios

| # | Scenario | Concept | Expected outcome |
|---|---|---|---|
| **S01** | Load test api-service to 80% CPU | HPA scale-out | Pods scale 2 → 5 within ~60s; CPU drops as load spreads |
| **S02** | Remove load; wait cooldown window | HPA scale-in | Pods scale back to 2 after stabilization (default 5 min) |
| **S03** | Publish 50 messages to orders queue | KEDA scale-from-zero | order-processor scales 0 → 10 as queue grows; processes messages |
| **S04** | Queue drains completely | KEDA scale-to-zero | order-processor scales back to 0; no idle pods |
| **S05** | `kubectl drain` node running api-service pod | PDB protection | Eviction blocked until another pod is healthy; `minAvailable: 1` enforced |
| **S06** | Deploy api-service v2 with injected 10% error rate | ArgoCD auto-rollback | Error rate > 5% detected; rollback to v1 triggers automatically within 3 min |



### Scenario Runner

```bash
kubectl apply -f deploy/scenario-runner-job.yaml -n demo-scaling
kubectl wait --for=condition=complete job/scenario-runner -n demo-scaling --timeout=15m
```

---

## Demo 13A — Two-Phase Commit (2PC)

**Repo:** `ms-two-phase-commit-demo` · **Namespace:** `demo-2pc`

**Focus:** Synchronous distributed transaction via 2PC coordinator pattern. Two independent databases commit atomically or both roll back. Demonstrates prepare/commit/rollback phases and coordinator crash recovery.

**What a reader takes away:** When 2PC provides strong consistency guarantees, its failure modes (blocking, coordinator SPOF), and why it's generally avoided in microservices in favour of eventual consistency / Saga.



### Services

`transaction-coordinator` — orchestrates 2PC protocol; writes durable decision log.
`inventory-service` — participant; Postgres DB A.
`billing-service` — participant; Postgres DB B.

Commons used: `commons-logging` `commons-datasource` `commons-web`



### Pre-Requisite Infrastructure

| Component | Role |
|---|---|
| **PostgreSQL A** | inventory-service database |
| **PostgreSQL B** | billing-service database |
| **PostgreSQL C** | coordinator transaction log (durable recovery) |



### Key Implementation Notes

**2PC protocol:**

```
Phase 1 — Prepare:
  Coordinator → inventory: "Prepare: reserve 5 units of X"   → VOTE_COMMIT (row locked)
  Coordinator → billing:   "Prepare: charge $50"             → VOTE_COMMIT (row locked)

Phase 2 — Commit (all voted COMMIT):
  Coordinator writes COMMIT to durable log
  Coordinator → inventory: "Commit"   → unlocks, commits
  Coordinator → billing:   "Commit"   → unlocks, commits

Phase 2 — Rollback (any ABORT or timeout):
  Coordinator → all: "Rollback"  → undo, unlock
```

**Recovery:** On coordinator restart, it reads its durable log. If COMMIT recorded → re-send COMMIT to all participants. If no decision → send ROLLBACK (safe: participants still holding locks wait for coordinator).



### Scenarios

| # | Scenario | Concept | Expected outcome |
|---|---|---|---|
| **S01** | Reserve stock + charge billing — both succeed | 2PC happy path | Both DBs updated; coordinator log shows COMMIT |
| **S02** | Billing votes ABORT (insufficient credit) | Phase 1 abort | Coordinator sends ROLLBACK to inventory; stock reservation undone; both DBs consistent |
| **S03** | Coordinator crashes after Phase 1 COMMIT log, before sending Phase 2 | Coordinator crash recovery | Coordinator restarts; reads durable log; resends COMMIT; transaction completes |
| **S04** | inventory-service unreachable during Phase 1 | Participant timeout | Coordinator times out; sends ROLLBACK to billing; aborted safely |
| **S05** | Concurrent conflicting transactions on same product | Locking | One waits on prepare lock; no deadlock; serialized correctly |



### Scenario Runner

```bash
kubectl apply -f deploy/scenario-runner-job.yaml -n demo-2pc
kubectl wait --for=condition=complete job/scenario-runner -n demo-2pc --timeout=10m
```

---

## Demo 13B — Saga Pattern (Temporal)

**Repo:** `ms-saga-demo` · **Namespace:** `demo-saga`

**Focus:** Saga orchestration with Temporal OSS. Long-running workflows with explicit compensation steps for partial failures. Durable execution survives worker crashes.

**What a reader takes away:** Why sagas are the preferred distributed transaction pattern for microservices (async, resilient, no distributed locks), and how Temporal makes them production-grade with built-in durability and visibility.



### Services

`order-workflow-service` — Temporal workflow (saga orchestrator).
`inventory-service`, `payment-service`, `shipping-service` — Temporal activities.

Commons used: `commons-logging` `commons-messaging` `commons-web`



### Pre-Requisite Infrastructure

| Component | Role |
|---|---|
| **Temporal** (OSS, K8s) | Workflow engine; durable execution state |
| **PostgreSQL** | Temporal persistence backend |



### Saga Design

```
PlaceOrderSaga(orderId, items, customerId):
  Step 1: reserveStock(items)         → compensate: releaseStock(items)
  Step 2: chargePayment(customerId)   → compensate: refundPayment(customerId)
  Step 3: createShipment(orderId)     → compensate: cancelShipment(orderId)

On any step failure:
  Execute compensations in reverse order of completed steps
  Mark order CANCELLED
```



### Scenarios

| # | Scenario | Concept | Expected outcome |
|---|---|---|---|
| **S01** | Full saga: reserve → charge → ship | Saga happy path | All 3 activities complete; order CONFIRMED; Temporal UI shows workflow history |
| **S02** | Payment fails (insufficient funds) | Saga compensation | `releaseStock` compensation runs automatically; order CANCELLED; inventory restored |
| **S03** | Shipping fails after payment succeeds | Multi-step compensation | `cancelShipment` + `refundPayment` run; stock released; all 3 DBs consistent |
| **S04** | Temporal worker pod crashes mid-workflow (after Step 1) | Durable execution | Worker restarts; workflow resumes from Step 2; Step 1 not re-executed |
| **S05** | inventory-service slow; activity timeout | Temporal retry | Retries with exponential backoff; eventual success; retry count in Temporal UI |
| **S06** | GET /orders/{id}/workflow-status | Workflow query | Returns current step, history of completed steps, compensation status |



### Scenario Runner

```bash
kubectl apply -f deploy/scenario-runner-job.yaml -n demo-saga
kubectl wait --for=condition=complete job/scenario-runner -n demo-saga --timeout=12m
```

---

## Demo 14A — RabbitMQ Event-Driven Communication

**Repo:** `ms-rabbitmq-events-demo` · **Namespace:** `demo-rabbitmq`

**Focus:** Event-driven communication with RabbitMQ. Topic exchange fan-out, work queues (competing consumers), idempotent consumers, DLQ with retry backoff, and correlation IDs.

**What a reader takes away:** How to design reliable event-driven flows — idempotency prevents duplicate side effects, DLQ prevents message loss on errors, and correlation IDs make cross-service debugging tractable.



### Services

`order-service` — publishes `OrderPlaced` events.
`notification-service` — topic exchange subscriber (idempotent consumer).
`inventory-service` — work queue consumer (3 pods competing).

Commons used: `commons-messaging` `commons-logging` `commons-observability`



### Pre-Requisite Infrastructure

| Component | Role |
|---|---|
| **RabbitMQ** (K8s Operator) | Topic exchange + work queue + DLX |
| **PostgreSQL** | Idempotency dedup table (message IDs) |



### Key Implementation Notes

**Exchange topology:**

```
topic exchange: orders.events
  routing: orders.# → notification-queue    (fan-out)
  routing: orders.placed → inventory-queue  (work queue, competing consumers)

Dead Letter Exchange: orders.dlx → orders.dlq (after maxRetries=3)
```

**Retry with backoff (TTL dead-lettering):**

```
On consumer failure:
  NACK → retry-queue (TTL=5s, 15s, 30s per attempt)
  After TTL → DLX routes back to original queue
  After maxRetries → DLQ (no re-route)
```

**Idempotent consumer:**

```
On receive:
  Check dedup_table for messageId
  If found → ACK and discard
  If not found → process → INSERT messageId → ACK
```



### Scenarios

| # | Scenario | Concept | Expected outcome |
|---|---|---|---|
| **S01** | Publish `OrderPlaced` | Topic fan-out | Both notification-service and inventory-service receive the event independently |
| **S02** | Publish same message twice (same messageId) | Idempotent consumer | Second delivery deduplicated via dedup table; processed exactly once |
| **S03** | notification-service throws on process | DLQ + retry backoff | Retried 3× with 5s/15s/30s backoff; on 4th failure → DLQ |
| **S04** | Inspect DLQ | Dead letter queue | Failed message visible in RabbitMQ management UI with original headers + failure reason |
| **S05** | Publish 30 events to inventory-queue with 3 consumer pods | Competing consumers | Each message consumed exactly once; ~10 per pod; parallel processing confirmed |
| **S06** | Trace correlationId from producer log to consumer log | Correlation ID | Same correlationId in both publisher and consumer spans/logs |



### Scenario Runner

```bash
kubectl apply -f deploy/scenario-runner-job.yaml -n demo-rabbitmq
kubectl wait --for=condition=complete job/scenario-runner -n demo-rabbitmq --timeout=10m
```

---

## Demo 14B — Event Sourcing + CQRS

**Repo:** `ms-event-sourcing-demo` · **Namespace:** `demo-event-sourcing`

**Focus:** Event Sourcing (append-only event store) + CQRS (separate command and query models). State rebuilt by replaying events; read projections updated asynchronously. Snapshot optimization for long-lived aggregates.

**What a reader takes away:** How event sourcing provides a complete audit trail and enables time-travel queries, and why CQRS lets each model be optimized independently (read vs write).



### Services

`order-command-service` — write side; validates commands, persists events.
`order-query-service` — read side; consumes event stream, maintains query-optimized projection.

Commons used: `commons-logging` `commons-datasource` `commons-messaging` `commons-web`



### Pre-Requisite Infrastructure

| Component | Role |
|---|---|
| **PostgreSQL** (event store) | Append-only `events` table |
| **PostgreSQL** (read model) | Separate DB; schema optimized for reads |
| **RabbitMQ** | Event propagation: command side → query side |



### Event Store Schema

```sql
CREATE TABLE events (
  id            BIGSERIAL PRIMARY KEY,
  aggregate_id  UUID NOT NULL,
  sequence      INT NOT NULL,          -- monotonic per aggregate
  event_type    VARCHAR(100) NOT NULL,
  payload       JSONB NOT NULL,
  created_at    TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(aggregate_id, sequence)       -- prevents duplicate events
);
```

**Aggregate replay:** SELECT events WHERE aggregate_id=? ORDER BY sequence ASC; apply each event to rebuild state. **Snapshot:** store state at sequence N; replay only events > N.



### Scenarios

| # | Scenario | Concept | Expected outcome |
|---|---|---|---|
| **S01** | POST /orders → `OrderCreated` appended → GET /orders/{id} (query side) | Write + eventual read | Event store: 1 event; query side reflects order within 500ms |
| **S02** | PATCH /orders/{id}/status → `OrderStatusChanged` appended | Event append | Event store: 2 events for aggregate; query side updates |
| **S03** | Truncate read model DB; rebuild from event store | Rebuild from events | Query service replays all events; read model identical to pre-truncate state |
| **S04** | Create aggregate with 50 events; trigger snapshot | Snapshot optimization | Snapshot at sequence 50; next replay starts from snapshot + events 51+ only |
| **S05** | GET /orders/{id}/history?at=2025-01-15T10:00:00Z | Time-travel query | Returns aggregate state as of given timestamp |
| **S06** | Scale query service to 3 replicas; command service at 1 | CQRS separation | Each service independently deployable, scalable, and with its own DB schema |



### Scenario Runner

```bash
kubectl apply -f deploy/scenario-runner-job.yaml -n demo-event-sourcing
kubectl wait --for=condition=complete job/scenario-runner -n demo-event-sourcing --timeout=10m
```

---

## Demo 14C — Kafka Event Streaming

**Repo:** `ms-kafka-events-demo` · **Namespace:** `demo-kafka`

**Focus:** Kafka for event streaming: partitioning for ordering guarantees, consumer groups (independent fan-out + competing consumers within a group), Avro schema with Apicurio Schema Registry for evolution, and dead letter topics for poison messages.

**What a reader takes away:** When Kafka is the right choice over RabbitMQ (high throughput, replay, independent consumer groups at scale), how partitioning guarantees ordering, and how schema registry enforces contracts safely.



### Services

`order-service` — Kafka producer.
`fulfillment-service` — consumer group A.
`analytics-service` — consumer group B (reads same events; independent offset).

Commons used: `commons-messaging` `commons-logging` `commons-observability`



### Pre-Requisite Infrastructure

| Component | Role |
|---|---|
| **Kafka** (Strimzi Operator) | Topic `orders` (6 partitions); topic `orders.dlq` |
| **Apicurio Schema Registry** | Avro schema storage + compatibility enforcement |



### Key Implementation Notes

**Avro schema — backward-compatible evolution:**

```json
{
  "type": "record", "name": "OrderPlaced",
  "fields": [
    { "name": "orderId",     "type": "string" },
    { "name": "customerId",  "type": "string" },
    { "name": "totalAmount", "type": "double" },
    { "name": "region",      "type": ["null","string"], "default": null }
  ]
}
```

`region` is a new optional field (v2). Old consumers read v2 messages — missing field uses default `null`. Schema registry rejects incompatible changes.

**Partition key = orderId** → all events for the same order go to the same partition → ordering guaranteed per order.

**Poison message:** Consumer catches `DeserializationException` → publishes to `orders.dlq` → original partition continues unblocked.



### Scenarios

| # | Scenario | Concept | Expected outcome |
|---|---|---|---|
| **S01** | Publish 60 OrderPlaced events; check both consumer groups | Independent consumer groups | Both groups receive all 60 events; each maintains own offset independently |
| **S02** | orderId as partition key; verify ordering per orderId | Partition ordering | All events for same orderId at same partition; consumed in sequence |
| **S03** | 3 fulfillment-service pods; 6 partitions | Consumer group partition assignment | Each pod assigned 2 partitions; rebalance on 4th pod addition |
| **S04** | Publish with v1 schema; send v2 message (new optional field) | Backward-compatible schema evolution | v1 consumers still process v2 messages; schema registry validates compatibility |
| **S05** | Publish malformed Avro message | Poison message → DLQ | Message routed to `orders.dlq`; healthy messages continue uninterrupted |
| **S06** | High-load publish; monitor consumer lag | Consumer lag metric | Grafana `kafka_consumer_group_lag` rises under load; drops as consumers catch up |



### Scenario Runner

```bash
kubectl apply -f deploy/scenario-runner-job.yaml -n demo-kafka
kubectl wait --for=condition=complete job/scenario-runner -n demo-kafka --timeout=10m
```

---

## Demo 15 — Kafka Streams

**Repo:** `ms-streaming-demo` · **Namespace:** `demo-streaming`

**Focus:** Stateful stream processing with Kafka Streams. Tumbling window aggregation, KStream-KTable join for enrichment, RocksDB state store with rebuild on restart, and interactive queries.

**What a reader takes away:** How stream processing differs from batch (continuous, stateful, real-time), when Kafka Streams is the right tool vs a separate framework, and how state stores survive pod restarts via Kafka changelog replication.



### Service

`order-analytics-service` — Kafka Streams application consuming `orders` topic.

Commons used: `commons-logging` `commons-observability`



### Pre-Requisite Infrastructure

| Component | Role |
|---|---|
| **Kafka** (Strimzi) | Input: `orders`; output: `order-metrics`; changelog topics for state replication |



### Topology

```
Input: orders topic
  KStream: filter valid orders
    groupBy(productId)
      windowedBy(TumblingWindow 5min)
        count() → KTable: order-counts-by-product

Input: products topic → KTable: products
  KStream (orders) join KTable (products) → enriched-orders (output topic)
```

**Interactive query (HTTP endpoint over local RocksDB):**

```
GET /analytics/orders/by-product?productId=X&window=2025-07-25T10:00
→ served from local RocksDB state store; no Kafka query round-trip
```



### Scenarios

| # | Scenario | Concept | Expected outcome |
|---|---|---|---|
| **S01** | Publish 50 orders across 10 products; query window counts | Tumbling window aggregation | GET /analytics/orders/by-product returns count per product per 5-min window |
| **S02** | OrderPlaced enriched with product name from products KTable | KStream-KTable join | Enriched-orders topic contains product name; no separate DB call |
| **S03** | Kill analytics-service pod; restart; query same window | State store rebuild | RocksDB rebuilt from changelog topic; identical query results post-restart |
| **S04** | Query state store via REST while stream is processing | Interactive queries | Returns current aggregation from local RocksDB without pausing stream |
| **S05** | Call GET /analytics/topology | Topology inspection | Returns processor topology description (input → filter → groupBy → window → output) |



### Scenario Runner

```bash
kubectl apply -f deploy/scenario-runner-job.yaml -n demo-streaming
kubectl wait --for=condition=complete job/scenario-runner -n demo-streaming --timeout=10m
```

---

## Demo 16 — Quartz Clustered Scheduler

**Repo:** `ms-quartz-scheduler-demo` · **Namespace:** `demo-scheduler`

**Focus:** Clustered Quartz scheduler ensuring jobs fire exactly once across multiple pod replicas. Misfire handling, dynamic job management at runtime, and K8s CronJob comparison.

**What a reader takes away:** Why application-level clustering (Quartz + JDBC) is needed for exactly-once execution with multiple replicas, and when K8s CronJobs are sufficient vs insufficient.



### Service

`scheduler-service` — 3 replicas sharing the same Quartz JDBC job store (PostgreSQL).

Commons used: `commons-logging` `commons-observability`



### Pre-Requisite Infrastructure

| Component | Role |
|---|---|
| **PostgreSQL** | Quartz JDBC tables (`QRTZ_*`); shared lock across all pods |



### Key Implementation Notes

**Quartz JDBC clustering:**

```yaml
spring:
  quartz:
    job-store-type: jdbc
    properties:
      org.quartz.jobStore.isClustered: true
      org.quartz.jobStore.clusterCheckinInterval: 10000
      org.quartz.scheduler.instanceId: AUTO   # unique per pod
```

All 3 pods read from `QRTZ_TRIGGERS`. DB row-level lock ensures exactly one pod acquires each trigger. If the executing pod dies, another picks it up after `clusterCheckinInterval`.

**K8s CronJob comparison:**

| | K8s CronJob | Quartz JDBC |
|---|---|---|
| Exactly-once | No (duplicate if pod slow) | Yes (DB lock) |
| Dynamic jobs | No (YAML only) | Yes (API at runtime) |
| Job history | No built-in | Yes (QRTZ tables) |
| External dep | None | PostgreSQL |



### Scenarios

| # | Scenario | Concept | Expected outcome |
|---|---|---|---|
| **S01** | 3 pods; trigger every 30s for 5 min | Clustered exactly-once | Exactly 1 execution per interval; logs show which pod acquired each trigger |
| **S02** | Pod executing job is killed mid-run | Job recovery | Another pod detects missed checkin; picks up job; execution completes |
| **S03** | All pods offline at scheduled time; restart | Misfire policy (`FIRE_NOW`) | Job fires immediately on pod restart; total execution count preserved in QRTZ tables |
| **S04** | POST /jobs: create new cron job at runtime | Dynamic job management | Job persisted to DB; fires on schedule; survives full pod restart |
| **S05** | K8s CronJob deployed for same interval with 2 replicas | CronJob duplicate problem | Two pods start → two executions; demonstrates why app-level clustering is needed |



### Scenario Runner

```bash
kubectl apply -f deploy/scenario-runner-job.yaml -n demo-scheduler
kubectl wait --for=condition=complete job/scenario-runner -n demo-scheduler --timeout=12m
```

---

## Demo 17 — Spring Batch Processing

**Repo:** `ms-spring-batch-demo` · **Namespace:** `demo-batch`

**Focus:** Large-scale data processing with Spring Batch. Chunk-based processing (transactional safety at scale), partitioned parallel steps (~4× throughput), retry/skip policies, and checkpoint-based restart (resume from failure, not from the beginning).

**What a reader takes away:** How chunk processing provides transactional safety without loading entire dataset into memory, why partitioned steps multiply throughput, and how restart-from-checkpoint eliminates duplicate work.



### Service

`batch-service` — Spring Batch application. Processes large CSV product import files from MinIO.

Commons used: `commons-logging` `commons-datasource` `commons-observability`



### Pre-Requisite Infrastructure

| Component | Role |
|---|---|
| **PostgreSQL** | Spring Batch metadata tables + target product table |
| **MinIO** (S3-compatible) | Input CSV files |



### Job Design

```
Job: ProductImportJob
  Step 1: ValidateFileStep      (single; check file exists + parse headers)
  Step 2: ImportProductsStep    (partitioned chunk step)
    Partitioner:  splits file into N ranges by line offset (1 partition per 10k rows)
    Reader:       FlatFileItemReader (reads CSV by offset range)
    Processor:    validates row; enriches with category lookup
    Writer:       JdbcBatchItemWriter (batch INSERT; chunk size=100)
    Retry:        transient DB errors → up to 3 retries (chunk-level rollback + retry)
    Skip:         validation errors → skip row; write to skip file; job continues
```



### Scenarios

| # | Scenario | Concept | Expected outcome |
|---|---|---|---|
| **S01** | Process 100k-row CSV | Chunk-based processing | 1000 chunks of 100 rows; all rows inserted; Spring Batch metadata shows chunk count + timing |
| **S02** | Transient DB error at chunk 500 | Chunk-level retry | Chunk 500 rolled back and retried (up to 3×); other chunks unaffected |
| **S03** | Invalid row (negative price) in chunk 700 | Skip policy | Row appended to skip file; `BATCH_STEP_EXECUTION.skipCount` increments; job continues |
| **S04** | Job fails at chunk 750; restart | Checkpoint restart | Job resumes from chunk 751; chunks 1–750 not reprocessed |
| **S05** | 100k rows: 1 partition vs 4 partitions | Partitioned step throughput | 4-partition run completes ~4× faster; each partition on separate thread |
| **S06** | Inspect `BATCH_JOB_EXECUTION` + `BATCH_STEP_EXECUTION` | Job metadata | Full execution history; start/end times; read/write/skip/commit counts per step |



### Scenario Runner

```bash
kubectl apply -f deploy/scenario-runner-job.yaml -n demo-batch
kubectl wait --for=condition=complete job/scenario-runner -n demo-batch --timeout=15m
```

---

## Demo 18 — Leader Election

**Repo:** `ms-leader-election-demo` · **Namespace:** `demo-leader-election`

**Focus:** Exactly-one-active leader election across multiple pod replicas using three mechanisms: Kubernetes Lease API (native, no external dep), Redis distributed lock (SETNX + TTL + fencing token), and PostgreSQL advisory lock. Failover detection and automatic re-election.

**What a reader takes away:** When each mechanism is appropriate, what split-brain means and how fencing tokens prevent it, and why leader election is essential for singleton background jobs in horizontally scaled deployments.



### Service

`worker-service` — 3 replicas. Only the elected leader processes tasks. Others remain in standby.

Commons used: `commons-logging` `commons-observability`



### Pre-Requisite Infrastructure

| Component | Role |
|---|---|
| **Kubernetes Lease API** | Built-in K8s leader election (no external dep) |
| **Redis** | SETNX + TTL distributed lock |
| **PostgreSQL** | `pg_try_advisory_lock` advisory lock |



### Key Implementation Notes

**K8s Lease API:**

```yaml
# Lease resource (created/renewed by elected pod)
apiVersion: coordination.k8s.io/v1
kind: Lease
metadata: { name: worker-leader, namespace: demo-leader-election }
spec:
  holderIdentity: pod-a          # current leader's pod name
  leaseDurationSeconds: 15
  renewTime: 2025-07-25T10:00:00Z
```

Pod renews Lease every 5s. If renewal fails for > `leaseDurationSeconds` → another pod wins next acquire.

**Redis lock with fencing token:**

```
SET leader:lock {pod-id} NX PX 10000       # atomic; NX=only-if-not-exists; PX=TTL ms
If OK → this pod is leader; renew every 5s; fencing token = INCR leader:token
If NIL → another pod holds; retry after TTL
```

Fencing token is an incrementing counter returned on acquisition. Stale leader (crashed, then resumed) will have an older token — downstream services reject writes with older token.

**PostgreSQL advisory lock:**

```sql
SELECT pg_try_advisory_lock(12345);   -- true if acquired; false if held elsewhere
-- Released on connection close or pg_advisory_unlock
```



### Scenarios

| # | Scenario | Concept | Expected outcome |
|---|---|---|---|
| **S01** | 3 pods start; K8s Lease election | K8s Lease | Exactly 1 pod acquires Lease; processes tasks; 2 in standby; `holderIdentity` in Lease object |
| **S02** | Leader pod killed; wait for lease expiry | K8s Lease failover | One standby pod wins election within `leaseDurationSeconds` (15s); no tasks missed |
| **S03** | 3 pods race for Redis SETNX | Redis distributed lock | Exactly 1 wins; others poll; lock TTL visible in Redis |
| **S04** | Redis leader pod crashes | Redis lock failover | After 10s TTL: another pod acquires lock; fencing token incremented; stale writes rejected |
| **S05** | 3 pods attempt `pg_try_advisory_lock` | DB advisory lock | 1 returns true; 2 return false; pod with lock processes; others skip iteration |
| **S06** | Compare all three: acquisition speed, failover time, external dependency | Mechanism comparison | Allure report: K8s Lease ≈ 15s failover; Redis ≈ 10s; DB advisory ≈ connection close |



### Scenario Runner

```bash
kubectl apply -f deploy/scenario-runner-job.yaml -n demo-leader-election
kubectl wait --for=condition=complete job/scenario-runner -n demo-leader-election --timeout=12m
```

---

## Demo 19 — Integration Testing with Testcontainers

**Repo:** `ms-testcontainers-demo` · **Namespace:** `demo-testcontainers`

**Focus:** Integration testing with real containers (PostgreSQL, Redis, Kafka, Keycloak) instead of mocks. Singleton container pattern for performance. Parallel test class isolation. No Kubernetes involved — runs locally and in CI.

**What a reader takes away:** Why Testcontainers eliminates the mock/production divergence problem, how the singleton pattern keeps suites fast, and when to use `@DynamicPropertySource` to wire container ports into Spring context.



### Test Subject

`order-service` — tested end-to-end against real dependencies.

Tests in `tests/integration/` — separate Maven module.



### Key Implementation Notes

**Singleton container pattern (shared across all test classes):**

```java
// AbstractIntegrationTest.java — all IT classes extend this
public abstract class AbstractIntegrationTest {

  static final PostgreSQLContainer<?> postgres =
      new PostgreSQLContainer<>("postgres:16").withReuse(true);
  static final GenericContainer<?> redis =
      new GenericContainer<>("redis:7").withExposedPorts(6379).withReuse(true);
  static final KafkaContainer kafka =
      new KafkaContainer(DockerImageName.parse("confluentinc/cp-kafka:7.6")).withReuse(true);

  static {
    Startables.deepStart(postgres, redis, kafka).join(); // parallel start; once per JVM
  }

  @DynamicPropertySource
  static void configure(DynamicPropertyRegistry registry) {
    registry.add("spring.datasource.url", postgres::getJdbcUrl);
    registry.add("spring.data.redis.host", redis::getHost);
    registry.add("spring.data.redis.port", () -> redis.getMappedPort(6379));
    registry.add("spring.kafka.bootstrap-servers", kafka::getBootstrapServers);
  }
}
```

**Wait strategies:**

```java
new GenericContainer<>("keycloak:25")
    .waitingFor(Wait.forLogMessage(".*Keycloak.*started.*", 1));

new PostgreSQLContainer<>()
    .waitingFor(Wait.forHealthcheck());
```



### Scenarios

| # | Scenario | Concept | Expected outcome |
|---|---|---|---|
| **S01** | `OrderServiceIT`: create order → verify DB row via JdbcTemplate | Real DB integration | No mocks; actual JDBC; Liquibase runs same migration scripts as prod |
| **S02** | `SecurityIT`: Keycloak container issues JWT → secured endpoint validates it | Real auth integration | JWT from real Keycloak OIDC endpoint; Spring Security validates against container JWKS |
| **S03** | `EventIT`: order created → Kafka event consumed within 5s | Real event integration | Producer + consumer use real Kafka; Awaitility assertion within 5s |
| **S04** | 50 test methods across 5 test classes; measure startup | Singleton pattern performance | 3 total container starts (not 15); suite completes in under 90s |
| **S05** | 4 test classes execute in parallel | Parallel test isolation | Each class uses isolated DB schema; no cross-class contamination |
| **S06** | `DockerComposeIT`: start full stack from `docker-compose.yaml` | Compose module | All services start; full end-to-end flow validated |



### Scenario Runner

Testcontainers tests run locally or in CI — not as a Kubernetes Job:

```bash
mvn test -pl tests/integration -Dtest=*IT
allure serve target/allure-results/
```

---
