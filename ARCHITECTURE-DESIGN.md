# TraderX ConfigHub Architecture Design

## Executive Summary

This document defines the technical architecture for deploying the FINOS TraderX trading platform using ConfigHub as the single source of truth. The architecture demonstrates all 12 canonical ConfigHub patterns across a multi-environment deployment (dev, staging, prod) with 8 microservices in strict dependency order.

**Key Architecture Principles**:
- ConfigHub-native deployment (zero kubectl commands in production code)
- Environment hierarchy with upstream/downstream relationships
- Event-driven automation via ConfigHub workers
- Drift detection and auto-correction
- Cost optimization with AI recommendations
- Full audit trail and compliance readiness

---

## 1. ConfigHub Space Topology

### 1.1 Space Hierarchy

```
┌─────────────────────────────────────────────────────────────┐
│                    ConfigHub Organization                    │
└─────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┴─────────────────────┬──────────┐
        │                     │                     │          │
┌───────▼────────┐  ┌────────▼────────┐  ┌────────▼────────┐ │
│ ${prefix}-     │  │ ${prefix}-      │  │ Base Spaces     │ │
│ traderx-base   │  │ traderx-filters │  │ (17 units)      │ │
│                │  │                 │  │                 │ │
│ Contains:      │  │ Contains:       │  │ Labels:         │ │
│ - 8 Deployment │  │ - 7 Filters     │  │ - project       │ │
│ - 7 Service    │  │ - 2 Sets        │  │ - environment   │ │
│ - 1 Namespace  │  │                 │  │ - layer         │ │
│ - 1 Ingress    │  │                 │  │ - order         │ │
└────────┬───────┘  └─────────────────┘  └─────────────────┘ │
         │                                                      │
         │ upstream/downstream relationships                   │
         │                                                      │
   ┌─────▼─────┐                                               │
   │ ${prefix}-│                                               │
   │ traderx-  │◄──────────────────────────────────────────────┘
   │ dev       │
   │           │
   │ Labels:   │
   │ - env=dev │
   │ - targetable=true │
   └─────┬─────┘
         │ push-upgrade propagation
         │
   ┌─────▼─────┐
   │ ${prefix}-│
   │ traderx-  │
   │ staging   │
   │           │
   │ Labels:   │
   │ - env=staging │
   │ - targetable=true │
   └─────┬─────┘
         │ push-upgrade propagation
         │
   ┌─────▼─────┐
   │ ${prefix}-│
   │ traderx-  │
   │ prod      │
   │           │
   │ Labels:   │
   │ - env=prod │
   │ - targetable=true │
   │ - critical=true │
   └───────────┘
```

### 1.2 Space Naming Convention

**Pattern**: `${prefix}-traderx-${environment}`

**Prefix Generation**:
```bash
# Canonical pattern from ConfigHub
prefix=$(cub space new-prefix)  # e.g., "fluffy-bunny", "happy-tree"
```

**Space Types**:

| Space | Purpose | Unit Count | Labels |
|-------|---------|-----------|--------|
| `${prefix}-traderx-base` | Master definitions | 17 | `project=${prefix}-traderx, environment=base` |
| `${prefix}-traderx-filters` | Filter and set definitions | 7 filters, 2 sets | `project=${prefix}-traderx, type=filters` |
| `${prefix}-traderx-dev` | Development environment | 17 (cloned from base) | `project=${prefix}-traderx, environment=dev, targetable=true` |
| `${prefix}-traderx-staging` | Staging environment | 17 (cloned from dev) | `project=${prefix}-traderx, environment=staging, targetable=true` |
| `${prefix}-traderx-prod` | Production environment | 17 (cloned from staging) | `project=${prefix}-traderx, environment=prod, targetable=true, critical=true` |

**Total ConfigHub Resources**:
- **Spaces**: 5
- **Units**: 85 (17 base + 17 dev + 17 staging + 17 prod + 17 in filters space)
- **Filters**: 7
- **Sets**: 2
- **Workers**: 3 (dev, staging, prod)

---

## 2. Service Topology

### 2.1 Service Dependency Graph

```
                     ┌──────────────────┐
                     │  reference-data  │
                     │  (order: 1)      │
                     │  Port: 18085     │
                     │  Tech: Java      │
                     └────────┬─────────┘
                              │
              ┌───────────────┼───────────────┐
              │               │               │
    ┌─────────▼────────┐ ┌───▼──────────┐ ┌─▼───────────────┐
    │  people-service  │ │  account-    │ │  position-      │
    │  (order: 2)      │ │  service     │ │  service        │
    │  Port: 18089     │ │  (order: 3)  │ │  (order: 4)     │
    │  Tech: Java      │ │  Port: 18091 │ │  Port: 18090    │
    └──────────────────┘ │  Tech: Node  │ │  Tech: Java     │
                         └───────┬──────┘ └─────────┬────────┘
                                 │                  │
                                 └────────┬─────────┘
                                          │
                                ┌─────────▼─────────┐
                                │  trade-service    │
                                │  (order: 5)       │
                                │  Port: 18092      │
                                │  Tech: .NET       │
                                │  **CRITICAL**     │
                                └─────────┬─────────┘
                                          │
              ┌───────────────────────────┼───────────────────────────┐
              │                           │                           │
    ┌─────────▼─────────┐      ┌─────────▼─────────┐      ┌─────────▼─────────┐
    │  trade-processor  │      │  trade-feed       │      │  web-gui          │
    │  (order: 6)       │      │  (order: 7)       │      │  (order: 8)       │
    │  No external port │      │  Port: 18088      │      │  Port: 18080      │
    │  Tech: Python     │      │  Tech: Java       │      │  Tech: Angular    │
    └───────────────────┘      └───────────────────┘      └───────────────────┘
                                                                     │
                                                           ┌─────────▼─────────┐
                                                           │  ingress          │
                                                           │  (order: 9)       │
                                                           │  nginx/traefik    │
                                                           └───────────────────┘
```

### 2.2 Service Communication Matrix

| Service | Depends On | Depended By | Protocol | Critical Path |
|---------|-----------|-------------|----------|---------------|
| **reference-data** | None | people, account, position, trade | HTTP REST | ✅ CRITICAL |
| **people-service** | reference-data | trade-service | HTTP REST | HIGH |
| **account-service** | reference-data | position, trade | HTTP REST | HIGH |
| **position-service** | reference-data, account | trade | HTTP REST | ✅ CRITICAL |
| **trade-service** | reference, people, account, position | processor, feed, web-gui | HTTP REST | ✅ CRITICAL |
| **trade-processor** | trade-service | None | Async (Message Queue) | HIGH |
| **trade-feed** | trade-service | web-gui | WebSocket/SSE | MEDIUM |
| **web-gui** | All services | ingress | HTTP REST | MEDIUM |
| **ingress** | web-gui | External users | HTTP/HTTPS | HIGH |

### 2.3 ConfigHub Unit Definitions

Each service consists of 1-2 ConfigHub units:

| Service | Deployment Unit | Service Unit | Total Units |
|---------|----------------|--------------|-------------|
| namespace | ✅ (order: 0) | N/A | 1 |
| reference-data | ✅ (order: 1) | ✅ | 2 |
| people-service | ✅ (order: 2) | ✅ | 2 |
| account-service | ✅ (order: 3) | ✅ | 2 |
| position-service | ✅ (order: 4) | ✅ | 2 |
| trade-service | ✅ (order: 5) | ✅ | 2 |
| trade-processor | ✅ (order: 6) | N/A | 1 |
| trade-feed | ✅ (order: 7) | ✅ | 2 |
| web-gui | ✅ (order: 8) | ✅ | 2 |
| ingress | ✅ (order: 9) | N/A | 1 |
| **Total** | | | **17** |

**Unit Labels** (for all deployment units):
```yaml
labels:
  service: <service-name>          # e.g., "trade-service"
  layer: <data|backend|frontend>   # Service layer
  tech: <technology-stack>         # e.g., "Java/Spring", ".NET"
  order: <0-9>                     # Deployment order
  port: <port-number>              # Service port (0 if none)
  critical: <true|false>           # Critical service flag
  environment: <base|dev|staging|prod>
```

---

## 3. Deployment Patterns

### 3.1 Ordered Deployment (Dependency-Aware)

**Deployment Sequence**:
```
0. namespace (traderx-dev)           ← Infrastructure
   ↓ wait for Active
1. reference-data-deployment         ← Foundation layer
   ↓ wait for Running (60s timeout)
2. people-service-deployment         ← User management
   ↓ wait for Running (60s timeout)
3. account-service-deployment        ← Account operations
   ↓ wait for Running (60s timeout)
4. position-service-deployment       ← Position tracking
   ↓ wait for Running (60s timeout)
5. trade-service-deployment          ← Trade execution (CRITICAL)
   ↓ wait for Running (60s timeout)
6. trade-processor-deployment        ← Async processing
   ↓ wait for Running (60s timeout)
7. trade-feed-deployment             ← Real-time feed
   ↓ wait for Running (60s timeout)
8. web-gui-deployment                ← User interface
   ↓ wait for Running (60s timeout)
9. ingress                           ← External access
```

**Implementation**:
```bash
#!/bin/bash
# bin/ordered-apply

SPACE=$1
PROJECT=$(bin/proj)

# Deploy in order 0-9
for order in {0..9}; do
  echo "📦 Deploying order $order units..."

  # Get units with this order
  units=$(cub unit list --space ${PROJECT}-${SPACE} \
    --filter "Labels.order = '$order'" --format json | jq -r '.[].Slug')

  for unit in $units; do
    echo "  Applying $unit..."
    cub unit apply $unit --space ${PROJECT}-${SPACE}

    # Health check with timeout
    timeout 60s bash -c "
      until kubectl get pods -n ${PROJECT}-${SPACE} \
        -l app=${unit%-*} \
        --field-selector=status.phase=Running | grep -q Running; do
        echo '    Waiting for $unit to be Running...'
        sleep 2
      done
    " || {
      echo "❌ ERROR: $unit failed to start within 60 seconds"
      exit 1
    }

    echo "  ✅ $unit is Running"
  done
done

echo "✅ All services deployed successfully in order!"
```

### 3.2 Blue-Green Deployment (Production)

**Architecture**:
```
                    ┌──────────────┐
                    │   Ingress    │
                    │  (LoadBalancer)│
                    └───────┬──────┘
                            │
              ┌─────────────┴─────────────┐
              │  Service Selector         │
              │  version: blue|green      │
              └─────────────┬─────────────┘
                            │
              ┌─────────────┴─────────────┐
              │                           │
    ┌─────────▼─────────┐      ┌─────────▼─────────┐
    │  Blue Environment │      │ Green Environment │
    │  (Current Prod)   │      │  (New Version)    │
    │                   │      │                   │
    │  trade-service:v1 │      │ trade-service:v2  │
    │  replicas: 3      │      │ replicas: 3       │
    │  Active           │      │ Standby           │
    └───────────────────┘      └───────────────────┘
```

**Implementation Pattern**:
```bash
# Phase 1: Deploy green environment
cub unit create trade-service-green \
  --space ${PROJECT}-prod \
  --upstream-unit ${PROJECT}-staging/trade-service \
  --label version=green \
  --label active=false

cub unit apply trade-service-green --space ${PROJECT}-prod

# Phase 2: Validate green environment
./test/validate-green-environment.sh

# Phase 3: Switch traffic to green
cub unit update ingress \
  --patch \
  --space ${PROJECT}-prod \
  --data '{"spec":{"rules":[{"backend":{"service":{"name":"trade-service-green"}}}]}}'

# Phase 4: Monitor for issues (keep blue warm)
sleep 300  # 5 minute soak test

# Phase 5: Decommission blue (or rollback if issues)
if health_check_passes; then
  cub unit destroy trade-service-blue --space ${PROJECT}-prod
else
  # Instant rollback
  cub unit update ingress \
    --patch \
    --space ${PROJECT}-prod \
    --data '{"spec":{"rules":[{"backend":{"service":{"name":"trade-service-blue"}}}]}}'
fi
```

### 3.3 Canary Deployment (Progressive Rollout)

**Traffic Split**:
```
┌─────────────────────────────────────────────────────┐
│                    Ingress                          │
│                                                     │
│  Traffic Split (managed by Istio/Linkerd):         │
│  - 90% → stable (v1)                                │
│  - 10% → canary (v2)                                │
└──────────────────────┬──────────────────────────────┘
                       │
         ┌─────────────┴─────────────┐
         │                           │
   ┌─────▼─────┐              ┌─────▼─────┐
   │  Stable   │              │  Canary   │
   │  v1.0     │              │  v2.0     │
   │  replicas:│              │  replicas:│
   │  3        │              │  1        │
   └───────────┘              └───────────┘
```

**Progressive Rollout Schedule**:
| Stage | Duration | Canary % | Stable % | Success Criteria |
|-------|----------|----------|----------|------------------|
| 1. Initial | 10 min | 10% | 90% | Error rate < 1% |
| 2. Expand | 20 min | 25% | 75% | Latency p95 < 200ms |
| 3. Majority | 30 min | 50% | 50% | No increase in errors |
| 4. Full | - | 100% | 0% | Complete migration |
| **Rollback** | 30 sec | 0% | 100% | Instant revert on failure |

### 3.4 Rollback Mechanisms

**Revision-Based Rollback**:
```bash
# Immediate rollback to previous revision
cub unit apply trade-service \
  --space ${PROJECT}-prod \
  --revision=N-1

# Verification
cub unit get trade-service --space ${PROJECT}-prod \
  | jq '.Revision'
```

**Changeset-Based Rollback** (atomic multi-service):
```bash
# Tag current state before deployment
cub changeset create release-v1.0 \
  --space ${PROJECT}-prod \
  --filter ${PROJECT}/all

# Deploy new version
bin/apply-all prod

# Rollback entire environment if issues
cub changeset apply release-v1.0 \
  --space ${PROJECT}-prod
```

**Emergency Recovery** (full environment rebuild):
```bash
# Destroy entire production
cub unit destroy --space ${PROJECT}-prod --filter ${PROJECT}/all

# Re-promote from staging
bin/promote staging prod
bin/apply-all prod
```

### 3.5 Health Check Integration

**Kubernetes Health Checks**:
```yaml
# Applied to all deployment units
spec:
  template:
    spec:
      containers:
      - name: trade-service
        livenessProbe:
          httpGet:
            path: /health/live
            port: 18092
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3

        readinessProbe:
          httpGet:
            path: /health/ready
            port: 18092
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 2
```

**ConfigHub Live State Monitoring**:
```bash
# Check live state of all services
cub unit get-live-state trade-service --space ${PROJECT}-prod

# Expected output:
# {
#   "Status": "Running",
#   "Replicas": 3,
#   "ReadyReplicas": 3,
#   "UpdatedReplicas": 3,
#   "Conditions": [...]
# }
```

---

## 4. Network Architecture

### 4.1 Service Mesh Readiness

**Istio Integration** (future):
```
┌─────────────────────────────────────────────────────────┐
│                    Istio Control Plane                  │
│  (pilot, citadel, galley, telemetry)                    │
└──────────────────────┬──────────────────────────────────┘
                       │
                       │ Service Discovery & mTLS
                       │
        ┌──────────────┴──────────────┐
        │                             │
┌───────▼────────┐          ┌─────────▼────────┐
│  Envoy Sidecar │          │  Envoy Sidecar   │
│  (trade-service)│         │  (position-svc)  │
│                │          │                  │
│  Features:     │          │  Features:       │
│  - mTLS        │          │  - mTLS          │
│  - Retry       │◄────────►│  - Circuit Break │
│  - Timeout     │          │  - Load Balance  │
│  - Observability│         │  - Observability │
└────────────────┘          └──────────────────┘
```

**Service Mesh Benefits**:
- **mTLS**: Automatic encryption between all services
- **Circuit Breaking**: Prevent cascading failures
- **Retry Logic**: Automatic retry with exponential backoff
- **Load Balancing**: Advanced routing (round-robin, least-request)
- **Observability**: Automatic tracing and metrics

**ConfigHub Integration**:
```yaml
# Add Istio labels to all deployment units
labels:
  istio-injection: enabled
  version: v1.0.0

# Istio VirtualService managed by ConfigHub
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: trade-service-route
spec:
  hosts:
  - trade-service
  http:
  - match:
    - headers:
        version:
          exact: v2
    route:
    - destination:
        host: trade-service
        subset: v2
  - route:
    - destination:
        host: trade-service
        subset: v1
```

### 4.2 Ingress Configuration

**Ingress Architecture**:
```
                    Internet
                       │
                       │ HTTPS (443)
                       ▼
            ┌──────────────────────┐
            │  Ingress Controller  │
            │  (nginx/traefik)     │
            │                      │
            │  TLS Termination     │
            │  Rate Limiting       │
            │  Path-based Routing  │
            └──────────┬───────────┘
                       │
         ┌─────────────┼─────────────┬─────────────┐
         │             │             │             │
    ┌────▼────┐  ┌────▼────┐  ┌────▼────┐  ┌────▼────┐
    │web-gui  │  │ trade   │  │position │  │reference│
    │:18080   │  │:18092   │  │:18090   │  │:18085   │
    └─────────┘  └─────────┘  └─────────┘  └─────────┘
```

**Ingress Rules** (from ConfigHub unit):
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: traderx-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/rate-limit: "100"
spec:
  tls:
  - hosts:
    - traderx.example.com
    secretName: traderx-tls
  rules:
  - host: traderx.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-gui
            port:
              number: 18080
      - path: /api/trades
        pathType: Prefix
        backend:
          service:
            name: trade-service
            port:
              number: 18092
      # ... additional paths
```

### 4.3 Network Policies

**Zero-Trust Network Model**:
```yaml
# Default deny all traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: traderx-prod
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

---
# Allow internal TraderX communication
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: traderx-internal
  namespace: traderx-prod
spec:
  podSelector:
    matchLabels:
      app: traderx
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: traderx
    ports:
    - protocol: TCP
      port: 18080  # web-gui
    - protocol: TCP
      port: 18085  # reference-data
    - protocol: TCP
      port: 18088  # trade-feed
    - protocol: TCP
      port: 18089  # people-service
    - protocol: TCP
      port: 18090  # position-service
    - protocol: TCP
      port: 18091  # account-service
    - protocol: TCP
      port: 18092  # trade-service
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: traderx
    ports:
    - protocol: TCP
      port: 18085  # reference-data
    # ... all service ports

---
# Allow ingress controller access
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-ingress-controller
  namespace: traderx-prod
spec:
  podSelector:
    matchLabels:
      app: web-gui
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - protocol: TCP
      port: 18080
```

**Network Policy Validation**:
```bash
# Test service-to-service connectivity
kubectl run test-pod --image=curlimages/curl -it --rm -- \
  curl -v http://reference-data.traderx-prod.svc.cluster.local:18085/health

# Expected: 200 OK (allowed by policy)

# Test external access (should be blocked)
kubectl run test-pod --image=curlimages/curl -it --rm -- \
  curl -v http://trade-service.traderx-prod.svc.cluster.local:18092/health

# Expected: Connection timeout (blocked by policy)
```

### 4.4 Load Balancing Strategy

**Internal Load Balancing** (Kubernetes Services):
```yaml
apiVersion: v1
kind: Service
metadata:
  name: trade-service
spec:
  type: ClusterIP
  sessionAffinity: None  # Round-robin load balancing
  ports:
  - port: 18092
    targetPort: 18092
    protocol: TCP
  selector:
    app: trade-service
```

**External Load Balancing** (Ingress Controller):
```yaml
# Weighted load balancing for canary deployments
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: trade-service-canary
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "10"  # 10% to canary
spec:
  rules:
  - host: traderx.example.com
    http:
      paths:
      - path: /api/trades
        backend:
          service:
            name: trade-service-canary
            port:
              number: 18092
```

**Load Balancing Strategies**:

| Strategy | Use Case | Implementation |
|----------|----------|----------------|
| **Round-robin** | Default for all services | Kubernetes Service (default) |
| **Least-connection** | High-load services (trade-service) | Istio DestinationRule |
| **Weighted** | Canary deployments | Nginx Ingress annotations |
| **Sticky sessions** | Stateful services (web-gui) | `sessionAffinity: ClientIP` |
| **Geo-routing** | Multi-region deployments | External DNS + Ingress |

---

## 5. Filter and Set Architecture

### 5.1 Filter Definitions

**Purpose**: Enable selective deployments, bulk operations, and targeted queries.

| Filter Name | Type | WHERE Clause | Use Case |
|------------|------|--------------|----------|
| **all** | Unit | `Space.Labels.project = '${project}'` | Select all TraderX units across all environments |
| **frontend** | Unit | `Labels.layer = 'frontend'` | Deploy/update only UI services |
| **backend** | Unit | `Labels.layer = 'backend'` | Deploy/update only backend services |
| **data** | Unit | `Labels.layer = 'data'` | Deploy/update only data layer services |
| **core-services** | Unit | `Labels.service IN ('reference-data', 'people-service', 'account-service')` | Core foundation services |
| **trading-services** | Unit | `Labels.service IN ('trade-service', 'trade-processor', 'trade-feed')` | Trading-specific services |
| **ordered** | Unit | `Labels.order IS NOT NULL` | Services with explicit deployment order |

**Filter Usage Examples**:
```bash
# Deploy only backend services to dev
cub unit apply --space ${PROJECT}-dev --filter ${PROJECT}/backend

# Update all trading services across environments
cub run set-image-reference \
  --container-name trade-service \
  --image-reference :v2.0.0 \
  --filter ${PROJECT}/trading-services \
  --space '*'

# View all services in deployment order
cub unit list --space ${PROJECT}-prod \
  --filter ${PROJECT}/ordered \
  --sort-by Labels.order
```

### 5.2 Set Definitions

**Purpose**: Logical grouping for drift detection, cost optimization, and monitoring.

| Set Name | Purpose | Members | Labels |
|----------|---------|---------|--------|
| **critical-services** | Services that must not fail (SLA 99.95%) | reference-data, trade-service, position-service | `tier=critical, monitor=true, alert=pagerduty` |
| **data-services** | Data layer services (database-dependent) | reference-data, people-service, account-service | `tier=data, layer=data, backup=required` |

**Set Membership** (dynamic via filters):
```bash
# Add all critical services to the set
cub set add-units critical-services \
  --space ${PROJECT}-base \
  --filter "Labels.service IN ('reference-data', 'trade-service', 'position-service')"

# View set members
cub set get critical-services --space ${PROJECT}-base

# Bulk operations on set
cub unit update --patch --space ${PROJECT}-prod \
  --where "SetID = 'critical-services'" \
  --data '{"spec":{"template":{"spec":{"priorityClassName":"high"}}}}'
```

**Set-Based Monitoring**:
```bash
# Monitor all critical services for drift
drift-detector --sets critical-services --space ${PROJECT}-prod

# Optimize costs for data services
cost-optimizer --sets data-services --space '*'
```

---

## 6. ConfigHub Canonical Patterns Implementation

### 6.1 Pattern Checklist

| # | Pattern | Implementation | Status |
|---|---------|---------------|--------|
| 1 | **Unique Project Naming** | `cub space new-prefix` in `bin/install-base` | ✅ |
| 2 | **Space Hierarchy** | base → dev → staging → prod with upstream relationships | ✅ |
| 3 | **Filter Creation** | 7 filters for layer-based and service-based targeting | ✅ |
| 4 | **Environment Cloning** | `cub unit create --dest-space --upstream-unit` | ✅ |
| 5 | **Version Promotion** | `cub run set-image-reference` + push-upgrade | ✅ |
| 6 | **Sets for Grouping** | critical-services, data-services sets | ✅ |
| 7 | **Event-Driven** | ConfigHub workers with 10s poll interval | ✅ |
| 8 | **ConfigHub Functions** | `cub run` commands for operational tasks | ✅ |
| 9 | **Changesets** | Atomic multi-service updates with rollback | ✅ |
| 10 | **Lateral Promotion** | Region-by-region rollout (future multi-region) | 🔄 |
| 11 | **Revision Management** | Full history tracking, rollback to revision N | ✅ |
| 12 | **Link Management** | Service-to-infrastructure links (DB, cache) | 🔄 |

### 6.2 Pattern Implementation Details

**Pattern 1: Unique Project Naming**
```bash
# bin/install-base
prefix=$(cub space new-prefix)  # e.g., "fluffy-bunny"
project="${prefix}-traderx"
echo "$project" > .cub-project
```

**Pattern 2: Space Hierarchy**
```bash
# bin/install-envs
cub unit create --dest-space ${PROJECT}-dev \
  --space ${PROJECT}-base \
  --filter ${PROJECT}/all \
  --label targetable=true
# Creates upstream relationship: dev units → base units
```

**Pattern 5: Version Promotion**
```bash
# bin/promote
cub run set-image-reference \
  --container-name web-gui \
  --image-reference :v1.2.3 \
  --space ${PROJECT}-staging

cub unit update --patch --upgrade \
  --space ${PROJECT}-prod
# Push-upgrade propagates changes from staging to prod
```

**Pattern 7: Event-Driven Workers**
```bash
# bin/setup-worker
cub worker create \
  --space ${PROJECT}-dev \
  --poll-interval 10s \
  --auto-apply true
# Worker continuously monitors ConfigHub for changes
```

**Pattern 9: Changesets**
```bash
# Atomic multi-service update
cub changeset create release-v2.0 \
  --space ${PROJECT}-prod \
  --filter ${PROJECT}/trading-services

cub changeset apply release-v2.0 \
  --space ${PROJECT}-prod
# All trading services updated atomically
```

---

## 7. Security Architecture

### 7.1 Security Boundaries

```
┌─────────────────────────────────────────────────────────┐
│                    Public Internet                      │
└───────────────────────┬─────────────────────────────────┘
                        │ HTTPS (TLS 1.3)
                        │ Rate Limited
                        ▼
            ┌───────────────────────┐
            │  Ingress Controller   │
            │  (DMZ)                │
            │  - TLS Termination    │
            │  - WAF                │
            │  - DDoS Protection    │
            └───────────┬───────────┘
                        │ Internal HTTP
                        │ Network Policy: Allow
                        ▼
            ┌───────────────────────┐
            │  Web-GUI Pods         │
            │  (Frontend Tier)      │
            │  - RBAC: read-only    │
            │  - No secrets         │
            └───────────┬───────────┘
                        │ Internal HTTP
                        │ Network Policy: Allow
                        ▼
            ┌───────────────────────┐
            │  Backend Services     │
            │  (Application Tier)   │
            │  - RBAC: limited      │
            │  - Secrets: K8s       │
            │  - mTLS (Istio)       │
            └───────────┬───────────┘
                        │ Database Protocol
                        │ Network Policy: Allow from backend only
                        ▼
            ┌───────────────────────┐
            │  Data Layer           │
            │  (Database Tier)      │
            │  - RBAC: strict       │
            │  - Encrypted at rest  │
            │  - Network isolation  │
            └───────────────────────┘
```

### 7.2 RBAC Configuration

**Kubernetes RBAC** (per environment):
```yaml
# Developer role (dev environment)
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: traderx-developer
  namespace: traderx-dev
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

---
# SRE role (staging environment)
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: traderx-sre
  namespace: traderx-staging
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps"]
  verbs: ["get", "list", "watch", "update", "patch"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "patch"]

---
# Read-only role (production environment)
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: traderx-viewer
  namespace: traderx-prod
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch"]
```

### 7.3 Secrets Management

**No Secrets in ConfigHub** (critical principle):
```yaml
# ✅ CORRECT: Reference Kubernetes secret in ConfigHub unit
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      containers:
      - name: trade-service
        envFrom:
        - secretRef:
            name: traderx-db-credentials  # Created separately in K8s

---
# ❌ WRONG: Hardcoded secrets in ConfigHub unit
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      containers:
      - name: trade-service
        env:
        - name: DB_PASSWORD
          value: "P@ssw0rd123"  # NEVER DO THIS!
```

**Secrets Creation** (outside ConfigHub):
```bash
# Create secrets in Kubernetes directly
kubectl create secret generic traderx-db-credentials \
  --from-literal=username=traderx_user \
  --from-literal=password=$(openssl rand -base64 32) \
  --namespace traderx-prod

# Reference in ConfigHub deployment unit
# (unit YAML only references the secret name, not values)
```

### 7.4 Audit Trail & Compliance

**ConfigHub Audit Trail**:
```bash
# View all changes to trade-service
cub unit revision list trade-service --space ${PROJECT}-prod

# Output:
# Rev | Author | Timestamp | Change
# 5   | alex@  | 2025-10-03 10:15 | Updated replicas 2→3
# 4   | alex@  | 2025-10-03 09:30 | Updated image :v1.1→:v1.2
# 3   | alex@  | 2025-10-02 14:20 | Updated env vars
# 2   | alex@  | 2025-10-01 11:10 | Initial deployment
```

**Compliance Requirements**:
- ✅ **SEC Rule 17a-4**: All configuration changes tracked with immutable audit log
- ✅ **FINRA 4511**: Complete audit trail for all trading platform changes
- ✅ **SOC 2**: Access controls, change management, and monitoring
- ✅ **PCI-DSS**: Secrets management, network segmentation, logging

---

## 8. Data Flow Diagrams

### 8.1 Trade Execution Flow

```
┌──────────┐
│ Web-GUI  │
│ (User)   │
└─────┬────┘
      │ 1. POST /api/trades/new
      │    {symbol: "AAPL", qty: 100, side: "BUY"}
      ▼
┌─────────────────┐
│ Trade-Service   │
│ (.NET)          │◄────────────────────────┐
└─────┬───────────┘                         │
      │                                     │
      │ 2. Validate trade                   │
      │ ↓ GET /api/reference-data/AAPL      │
      ▼                                     │
┌─────────────────┐                         │
│ Reference-Data  │                         │
│ (Java)          │                         │
└─────────────────┘                         │
      ↑                                     │
      │ 3. Returns: {price: $150, valid: true}
      └─────────────────────────────────────┘

      ┌─────────────────┐
      │ Trade-Service   │
      └─────┬───────────┘
            │ 4. Check account balance
            │ ↓ GET /api/accounts/{accountId}
            ▼
      ┌─────────────────┐
      │ Account-Service │
      │ (Node.js)       │
      └─────────────────┘
            ↑
            │ 5. Returns: {balance: $50000, available: true}
            └─────────────────────────────────────┐
                                                  │
      ┌─────────────────┐                         │
      │ Trade-Service   │◄────────────────────────┘
      └─────┬───────────┘
            │ 6. Execute trade
            │ ↓ Update position
            ▼
      ┌─────────────────┐
      │ Position-Service│
      │ (Java)          │
      └─────────────────┘
            ↑
            │ 7. Returns: {positionId: 123, updated: true}
            └─────────────────────────────────────┐
                                                  │
      ┌─────────────────┐                         │
      │ Trade-Service   │◄────────────────────────┘
      └─────┬───────────┘
            │ 8. Async: Publish trade event
            │
            ├─────────────────┐
            │                 │
            ▼                 ▼
      ┌──────────────┐  ┌──────────────┐
      │Trade-Processor│ │ Trade-Feed   │
      │(Python)      │  │ (Java)       │
      │              │  │              │
      │Settlement    │  │WebSocket→GUI │
      └──────────────┘  └──────────────┘
```

### 8.2 ConfigHub Deployment Flow

```
┌─────────────────────────────────────────────────────────┐
│              Developer Workflow                         │
└─────────────────────────────────────────────────────────┘
      │
      │ 1. Code change → Git push
      ▼
┌─────────────────┐
│ CI/CD Pipeline  │
│ (GitHub Actions)│
└─────┬───────────┘
      │ 2. Build image
      │    docker build -t trade-service:v2.0
      ▼
┌─────────────────┐
│ Container       │
│ Registry        │
└─────┬───────────┘
      │ 3. Push image
      │    docker push trade-service:v2.0
      ▼
┌─────────────────┐
│ ConfigHub       │
│ (Update unit)   │
└─────┬───────────┘
      │ 4. cub run set-image-reference :v2.0
      │
      ├────────────────┬────────────────┐
      │                │                │
      ▼                ▼                ▼
┌──────────┐    ┌──────────┐    ┌──────────┐
│  Worker  │    │  Worker  │    │  Worker  │
│  (Dev)   │    │ (Staging)│    │  (Prod)  │
└────┬─────┘    └────┬─────┘    └────┬─────┘
     │               │               │
     │ 5. Poll ConfigHub (10s)      │
     │               │               │
     ▼               ▼               ▼
┌──────────┐    ┌──────────┐    ┌──────────┐
│Kubernetes│    │Kubernetes│    │Kubernetes│
│  (Dev)   │    │ (Staging)│    │  (Prod)  │
└──────────┘    └──────────┘    └──────────┘
     │               │               │
     │ 6. Apply deployment          │
     │               │               │
     ▼               ▼               ▼
┌──────────┐    ┌──────────┐    ┌──────────┐
│  Pods    │    │  Pods    │    │  Pods    │
│  Running │    │  Running │    │  Running │
└──────────┘    └──────────┘    └──────────┘
```

### 8.3 Drift Detection & Correction Flow

```
┌─────────────────────────────────────────────────────────┐
│              Kubernetes Cluster (Prod)                  │
└─────────────────────────────────────────────────────────┘
      │
      │ 1. Manual kubectl change (drift introduced)
      │    kubectl scale deployment trade-service --replicas=10
      ▼
┌─────────────────┐
│ Deployment      │
│ trade-service   │
│ replicas: 10    │ ← Drift!
└─────┬───────────┘
      │
      │ 2. Informer detects change
      ▼
┌─────────────────┐
│ Drift Detector  │
│ (DevOps App)    │
└─────┬───────────┘
      │ 3. Compare with ConfigHub
      │ ↓ GET unit trade-service
      ▼
┌─────────────────┐
│ ConfigHub API   │
│ Expected: 3     │
│ Actual: 10      │
│ Drift: YES      │
└─────┬───────────┘
      │
      │ 4. Generate correction
      │    cub unit update --patch replicas=3
      ▼
┌─────────────────┐
│ Claude AI       │
│ (Risk Analysis) │
└─────┬───────────┘
      │ 5. Assess risk
      │    Risk: LOW (replica count)
      │    Action: AUTO-CORRECT
      ▼
┌─────────────────┐
│ ConfigHub       │
│ Update unit     │
└─────┬───────────┘
      │ 6. Trigger worker
      │ ↓ Worker polls, sees change
      ▼
┌─────────────────┐
│ Worker (Prod)   │
└─────┬───────────┘
      │ 7. Apply correction
      │    cub unit apply trade-service
      ▼
┌─────────────────┐
│ Kubernetes      │
│ trade-service   │
│ replicas: 3     │ ← Corrected!
└─────────────────┘
      │
      │ 8. Alert sent
      │    "Drift detected and corrected"
      ▼
┌─────────────────┐
│ Monitoring      │
│ Dashboard       │
└─────────────────┘
```

---

## 9. Disaster Recovery & Business Continuity

### 9.1 Recovery Time Objectives (RTO)

| Scenario | RTO | RPO | Recovery Method |
|----------|-----|-----|-----------------|
| **Single service failure** | 30 seconds | 0 | Kubernetes self-healing + worker reapply |
| **Environment failure (dev)** | 5 minutes | 0 | `bin/apply-all dev` |
| **Environment failure (prod)** | 15 minutes | 0 | `cub changeset apply` (last known good) |
| **Complete cluster failure** | 1 hour | 5 minutes | Rebuild cluster + `bin/apply-all prod` |
| **ConfigHub unavailable** | 4 hours | 0 | Manual kubectl apply + restore from backup |
| **Data corruption** | 4 hours | 15 minutes | Database restore from backup |

### 9.2 Backup Strategy

**ConfigHub Backups**:
```bash
# Daily backup of all ConfigHub units
cub unit export --space ${PROJECT}-prod --format yaml > traderx-prod-backup-$(date +%Y%m%d).yaml

# Weekly backup of entire project
for env in base dev staging prod; do
  cub unit export --space ${PROJECT}-${env} --format yaml > traderx-${env}-$(date +%Y%m%d).yaml
done

# Store in version control
git add traderx-*-backup-*.yaml
git commit -m "ConfigHub backup $(date +%Y%m%d)"
git push
```

**Kubernetes Backups**:
```bash
# Velero backup (all namespaces)
velero backup create traderx-prod-backup \
  --include-namespaces traderx-prod \
  --snapshot-volumes \
  --ttl 720h

# Verify backup
velero backup describe traderx-prod-backup
```

### 9.3 Disaster Recovery Runbook

**Scenario: Production Cluster Failure**

1. **Immediate Response** (0-5 min)
   ```bash
   # Verify cluster is down
   kubectl get nodes  # Expected: connection refused

   # Alert team
   slack-notify "#incidents" "🚨 Prod cluster DOWN - Initiating DR"
   ```

2. **Activate DR Cluster** (5-15 min)
   ```bash
   # Switch kubeconfig to DR cluster
   kubectl config use-context traderx-prod-dr

   # Verify DR cluster is healthy
   kubectl get nodes  # Expected: All nodes Ready
   ```

3. **Restore ConfigHub State** (15-30 min)
   ```bash
   # Apply ConfigHub units to DR cluster
   bin/apply-all prod --cluster traderx-prod-dr

   # Verify all services running
   kubectl get pods -n traderx-prod
   ```

4. **Restore Data** (30-60 min)
   ```bash
   # Restore databases from backup
   velero restore create --from-backup traderx-prod-backup

   # Verify data integrity
   ./test/verify-data-integrity.sh
   ```

5. **Switch Traffic** (60-65 min)
   ```bash
   # Update DNS to point to DR cluster
   aws route53 change-resource-record-sets \
     --hosted-zone-id Z123 \
     --change-batch file://dns-update-dr.json

   # Verify traffic flowing to DR
   curl https://traderx.example.com/health
   ```

6. **Post-Recovery** (65+ min)
   ```bash
   # Monitor for issues
   ./monitor/check-health.sh

   # Document incident
   ./incidents/create-postmortem.sh
   ```

---

## 10. Multi-Region Architecture (Future)

### 10.1 Multi-Region Topology

```
┌─────────────────────────────────────────────────────────┐
│                    ConfigHub Global                     │
│                  (Single Source of Truth)               │
└────────────────────┬────────────────────────────────────┘
                     │
        ┌────────────┴────────────┬────────────────────────┐
        │                         │                        │
┌───────▼────────┐      ┌─────────▼────────┐    ┌─────────▼────────┐
│  US-EAST-1     │      │   US-WEST-2      │    │   EU-CENTRAL-1   │
│  (Primary)     │      │   (Secondary)    │    │   (Tertiary)     │
│                │      │                  │    │                  │
│  Workers: 3    │      │   Workers: 3     │    │   Workers: 3     │
│  K8s Cluster   │      │   K8s Cluster    │    │   K8s Cluster    │
│                │      │                  │    │                  │
│  Latency: 0ms  │      │   Latency: 50ms  │    │   Latency: 100ms │
└────────────────┘      └──────────────────┘    └──────────────────┘
```

### 10.2 Lateral Promotion Pattern

```bash
# Pattern 10: Lateral Promotion (region-by-region rollout)

# Phase 1: Deploy to us-east-1
cub unit apply --space ${PROJECT}-prod-us-east-1 \
  --filter ${PROJECT}/all

# Validate in us-east-1
./test/validate-region.sh us-east-1

# Phase 2: Lateral promotion to us-west-2
cub unit create --dest-space ${PROJECT}-prod-us-west-2 \
  --space ${PROJECT}-prod-us-east-1 \
  --filter ${PROJECT}/all \
  --label region=us-west-2

# Phase 3: Lateral promotion to eu-central-1
cub unit create --dest-space ${PROJECT}-prod-eu-central-1 \
  --space ${PROJECT}-prod-us-west-2 \
  --filter ${PROJECT}/all \
  --label region=eu-central-1
```

---

## 11. Monitoring & Observability Architecture

### 11.1 Metrics Collection

```
┌─────────────────────────────────────────────────────────┐
│                  TraderX Services                       │
│  (All expose /metrics endpoint)                         │
└────────────────────┬────────────────────────────────────┘
                     │ Prometheus scrape (15s interval)
                     ▼
┌─────────────────────────────────────────────────────────┐
│                  Prometheus                             │
│  - Service discovery (K8s)                              │
│  - Time-series database                                 │
│  - Alerting rules                                       │
└────────────────────┬────────────────────────────────────┘
                     │
        ┌────────────┴────────────┬──────────────┐
        │                         │              │
        ▼                         ▼              ▼
┌───────────────┐      ┌──────────────────┐  ┌──────────┐
│   Grafana     │      │  Alert Manager   │  │  Thanos  │
│  (Dashboards) │      │  (Notifications) │  │  (Long-  │
│               │      │                  │  │  term)   │
└───────────────┘      └──────────────────┘  └──────────┘
```

### 11.2 Distributed Tracing

```
┌──────────────────────────────────────────────────────────┐
│  User Request: POST /api/trades/new                      │
└────────────────────┬─────────────────────────────────────┘
                     │ TraceID: abc123
                     ▼
          ┌──────────────────┐
          │  web-gui         │ Span 1: 50ms
          └────────┬─────────┘
                   │
                   ▼
          ┌──────────────────┐
          │  trade-service   │ Span 2: 150ms
          └────────┬─────────┘
                   │
       ┌───────────┴───────────┬───────────────────┐
       │                       │                   │
       ▼                       ▼                   ▼
┌─────────────┐    ┌─────────────────┐   ┌────────────────┐
│reference-   │    │ account-service │   │ position-      │
│data         │    │                 │   │ service        │
│Span 3: 20ms │    │ Span 4: 30ms    │   │ Span 5: 50ms   │
└─────────────┘    └─────────────────┘   └────────────────┘

Total latency: 50 + 150 + max(20, 30, 50) = 250ms
```

---

## Appendix A: Configuration Examples

### A.1 Complete Deployment Unit Example

```yaml
# confighub/base/trade-service-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: trade-service
  namespace: traderx-{{ .Environment }}
  labels:
    app: trade-service
    layer: backend
    tech: dotnet
    version: v1.0.0
spec:
  replicas: {{ if eq .Environment "prod" }}3{{ else }}1{{ end }}
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: trade-service
  template:
    metadata:
      labels:
        app: trade-service
        layer: backend
        version: v1.0.0
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "18092"
        prometheus.io/path: "/metrics"
    spec:
      serviceAccountName: trade-service
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 1000
      containers:
      - name: trade-service
        image: finos/traderx-trade-service:{{ .ImageTag | default "latest" }}
        imagePullPolicy: IfNotPresent
        ports:
        - name: http
          containerPort: 18092
          protocol: TCP
        env:
        - name: ASPNETCORE_ENVIRONMENT
          value: {{ .Environment | title }}
        - name: ASPNETCORE_URLS
          value: "http://+:18092"
        - name: REFERENCE_DATA_URL
          value: "http://reference-data:18085"
        - name: ACCOUNT_SERVICE_URL
          value: "http://account-service:18091"
        - name: POSITION_SERVICE_URL
          value: "http://position-service:18090"
        envFrom:
        - secretRef:
            name: trade-service-secrets
        livenessProbe:
          httpGet:
            path: /health/live
            port: 18092
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 18092
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 2
        resources:
          requests:
            cpu: {{ if eq .Environment "prod" }}500m{{ else }}250m{{ end }}
            memory: {{ if eq .Environment "prod" }}512Mi{{ else }}256Mi{{ end }}
          limits:
            cpu: {{ if eq .Environment "prod" }}1000m{{ else }}500m{{ end }}
            memory: {{ if eq .Environment "prod" }}1Gi{{ else }}512Mi{{ end }}
        volumeMounts:
        - name: config
          mountPath: /app/config
          readOnly: true
      volumes:
      - name: config
        configMap:
          name: trade-service-config
```

---

## Appendix B: Command Reference

### B.1 Essential ConfigHub Commands

```bash
# Infrastructure setup
cub space new-prefix                          # Generate unique prefix
cub space create ${PROJECT}-base              # Create space
cub filter create all Unit --where-field ...  # Create filter
cub set create critical-services              # Create set

# Unit operations
cub unit create my-unit --type k8s/Deployment # Create unit
cub unit apply my-unit --space ${PROJECT}-dev # Apply to cluster
cub unit update --patch --data '{"spec":{}}' # Update unit
cub unit destroy my-unit --space ${PROJECT}-dev # Remove from cluster

# Environment management
cub unit create --dest-space dev --space base # Clone units
cub run set-image-reference :v1.2.3          # Update image
cub unit update --patch --upgrade            # Push-upgrade

# Monitoring
cub unit get-live-state my-unit              # Check live status
cub unit tree --node=space --filter all      # View hierarchy
cub unit revision list my-unit               # View history
```

---

**Document Version**: 1.0
**Author**: Architecture Agent
**Date**: 2025-10-03
**Status**: Ready for Code Generator Agent Review
**Next Steps**: Implement service-dependency-map.json and confighub-topology.yaml
