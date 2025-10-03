# TraderX ConfigHub Implementation Plan

## Executive Summary

This implementation plan outlines the comprehensive deployment of the FINOS TraderX trading platform (8 microservices) using ConfigHub as the single source of truth. The deployment will demonstrate all 12 canonical ConfigHub patterns and establish TraderX as a reference implementation for financial services platforms.

**Timeline**: 10 hours with multi-agent orchestration (vs 16+ hours sequential)
**Outcome**: Production-ready, multi-environment trading platform deployed exclusively through ConfigHub
**Risk Level**: Medium (financial trading requirements + service dependencies)

---

## Architecture Analysis

### Service Inventory

| # | Service | Tech Stack | Port | Dependencies | Criticality |
|---|---------|-----------|------|--------------|-------------|
| 1 | reference-data | Java/Spring | 18085 | None | **CRITICAL** - Master data foundation |
| 2 | people-service | Java/Spring | 18089 | reference-data | **HIGH** - User management |
| 3 | account-service | Node.js/NestJS | 18091 | reference-data | **HIGH** - Account operations |
| 4 | position-service | Java/Spring | 18090 | reference-data, account-service | **CRITICAL** - Position tracking |
| 5 | trade-service | .NET/C# | 18092 | All above | **CRITICAL** - Trade execution |
| 6 | trade-processor | Python | N/A | trade-service | **HIGH** - Async settlement |
| 7 | trade-feed | Java/Spring | 18088 | trade-service | **MEDIUM** - Real-time feed |
| 8 | web-gui | Angular/React | 18080 | All services | **MEDIUM** - User interface |

### Dependency Graph

```
reference-data (order 1) ← Foundation layer
    ├── people-service (order 2)
    ├── account-service (order 3)
    └── position-service (order 4) ← account-service
            └── trade-service (order 5)
                    ├── trade-processor (order 6)
                    ├── trade-feed (order 7)
                    └── web-gui (order 8)
```

### Current State Analysis

**Existing Implementation** (at `/Users/alexis/traderx/`):
- ✅ 17 YAML manifests created and validated
- ✅ 9 ConfigHub scripts implemented and tested
- ✅ Dependency ordering system (order 0-9) in place
- ✅ Filters and Sets architecture defined
- ⚠️ **BLOCKER**: ConfigHub quota (97/100 spaces, need 5 for deployment)

**Technical Constraints**:
- Financial trading platform requires high availability (99.95% target)
- Sub-second latency requirements for trading services
- Strict ordering of service startup (cascading failures if violated)
- Regulatory compliance for audit trails (SEC, FINRA requirements)

---

## Deployment Phases

### Phase 1: ConfigHub Infrastructure Setup (1.5 hours)

**Objective**: Establish ConfigHub foundation with unique naming and hierarchy

**Tasks**:
1. **Resolve Quota Issues** (30 min)
   - Remove BridgeWorkers from 25 test spaces
   - Verify 5 spaces available (base + filters + dev + staging + prod)
   - Document quota management best practices

2. **Execute Base Installation** (30 min)
   ```bash
   cd /Users/alexis/traderx
   bin/install-base  # Creates unique prefix, spaces, filters, units
   ```
   - Generates unique project prefix (e.g., "fluffy-bunny-traderx")
   - Creates base space with all 17 units
   - Creates filter space with 7 filters
   - Creates 2 sets (critical-services, data-services)

3. **Environment Hierarchy Creation** (30 min)
   ```bash
   bin/install-envs  # Creates dev → staging → prod hierarchy
   ```
   - Creates 3 environment spaces with upstream relationships
   - Establishes inheritance chain for push-upgrade
   - Configures environment-specific labels

**Success Criteria**:
- [ ] 5 spaces created and visible in ConfigHub
- [ ] All 17 units present in base space
- [ ] All 7 filters operational
- [ ] Environment hierarchy visible via `cub unit tree`

**Outputs**:
- ConfigHub spaces: `{prefix}-traderx-base`, `{prefix}-traderx-filters`, `{prefix}-traderx-{dev,staging,prod}`
- Units: 8 deployments, 7 services, 1 namespace, 1 ingress
- Filters: all, frontend, backend, data, core-services, trading-services, ordered

---

### Phase 2: Development Environment Deployment (2 hours)

**Objective**: Deploy all 8 services to dev environment in correct dependency order

**Tasks**:
1. **Ordered Deployment Execution** (45 min)
   ```bash
   bin/ordered-apply $(bin/proj)-dev
   ```
   - Deploys services order 0-9 sequentially
   - Health check validation after each service
   - Timeout handling (60s per service)

2. **Service Validation** (30 min)
   - Verify all 8 deployments are Running
   - Check service endpoints are accessible
   - Validate inter-service communication
   - Confirm database connections

3. **Worker Setup** (45 min)
   ```bash
   bin/setup-worker dev
   ```
   - Deploy ConfigHub worker to dev cluster
   - Configure auto-apply on unit changes
   - Test worker responsiveness (10s poll interval)
   - Validate worker logs and status

**Success Criteria**:
- [ ] All 8 services showing "Running" status in Kubernetes
- [ ] Service health checks passing (via kubectl or ConfigHub live state)
- [ ] Worker deployed and auto-applying changes
- [ ] Reference-data API responding on port 18085

**Expected Deployment Order**:
```
0. namespace (traderx-dev) → 1. reference-data → 2. people-service →
3. account-service → 4. position-service → 5. trade-service →
6. trade-processor → 7. trade-feed → 8. web-gui → 9. ingress
```

**Outputs**:
- Running dev environment with all 8 services
- ConfigHub worker active in cluster
- Live state tracking in ConfigHub

---

### Phase 3: Environment Promotion & Testing (2 hours)

**Objective**: Establish promotion pipeline and validate push-upgrade pattern

**Tasks**:
1. **Staging Promotion** (45 min)
   ```bash
   bin/promote dev staging
   bin/apply-all staging
   ```
   - Push-upgrade changes from dev to staging
   - Apply all units to staging cluster
   - Verify upstream relationships preserved

2. **Production Promotion** (45 min)
   ```bash
   bin/promote staging prod
   bin/apply-all prod
   ```
   - Push-upgrade changes from staging to prod
   - Apply with zero-downtime strategy
   - Validate production readiness

3. **Rollback Testing** (30 min)
   ```bash
   # Simulate failure and rollback
   cub unit apply --space $(bin/proj)-prod --unit trade-service --revision=N-1
   ```
   - Test revision rollback mechanism
   - Verify service recovery
   - Document rollback procedures

**Success Criteria**:
- [ ] Staging environment matches dev (via ConfigHub diff)
- [ ] Production environment deployed successfully
- [ ] Rollback tested and verified working
- [ ] Environment tree shows proper hierarchy

**Outputs**:
- 3 fully deployed environments (dev, staging, prod)
- Tested promotion pipeline
- Documented rollback procedures

---

### Phase 4: Worker Automation & CI/CD (1.5 hours)

**Objective**: Replace Tilt with ConfigHub workers for continuous deployment

**Tasks**:
1. **Worker Configuration** (45 min)
   - Set up workers in staging and prod
   - Configure environment-specific poll intervals
   - Establish security boundaries (RBAC, network policies)

2. **CI/CD Integration** (45 min)
   - Create GitHub Actions workflow (optional)
   - Automate image build → ConfigHub update
   - Test automated deployment flow

   ```yaml
   # Example: .github/workflows/dev-deploy.yml
   on: push
   jobs:
     deploy:
       - docker build -t traderx/web-gui:${{ github.sha }}
       - cub run set-image-reference --container-name web-gui --image-reference :${{ github.sha }}
   ```

**Success Criteria**:
- [ ] Workers active in all environments
- [ ] Automated deployment tested (code change → deployment < 2 min)
- [ ] CI/CD pipeline documented

**Outputs**:
- Workers in dev, staging, prod
- Automated deployment pipeline (optional)

---

### Phase 5: DevOps Apps Integration (2 hours)

**Objective**: Demonstrate drift detection and cost optimization on TraderX

**Tasks**:
1. **Drift Detector Integration** (1 hour)
   ```bash
   # Deploy drift-detector watching TraderX spaces
   cd /path/to/drift-detector
   # Configure to monitor $(bin/proj)-* spaces
   # Test by introducing intentional drift
   kubectl scale deployment trade-service --replicas=10
   # Verify detection and auto-correction
   ```

2. **Cost Optimizer Integration** (1 hour)
   ```bash
   # Deploy cost-optimizer analyzing TraderX
   cd /path/to/cost-optimizer
   # Analyze resource usage across all 8 services
   # Generate Claude AI recommendations
   # Test optimization application
   ```

**Success Criteria**:
- [ ] Drift detector identifies configuration drift within 30 seconds
- [ ] Cost optimizer provides AI-driven recommendations
- [ ] Combined dashboard shows drift + cost metrics
- [ ] Auto-correction tested and verified

**Outputs**:
- Drift detector monitoring TraderX
- Cost optimizer with AI recommendations
- Combined observability dashboard

---

### Phase 6: Monitoring & Observability (1 hour)

**Objective**: Establish comprehensive monitoring and alerting

**Tasks**:
1. **Health Check Configuration** (20 min)
   - Configure liveness/readiness probes
   - Set up service-level objectives (SLOs)
   - Define alerting thresholds

2. **Metrics Collection** (20 min)
   - Enable Prometheus metrics export
   - Configure Grafana dashboards
   - Set up log aggregation

3. **Alert Configuration** (20 min)
   - Critical: Trade service downtime > 30s
   - High: Any service restart > 3 times/hour
   - Medium: Resource usage > 80%

**Success Criteria**:
- [ ] All services exposing health endpoints
- [ ] Dashboards showing real-time metrics
- [ ] Alerts configured and tested

**Outputs**:
- Monitoring dashboards (Grafana/ConfigHub UI)
- Alert rules and notification channels
- SLO definitions

---

## Resource Requirements

### ConfigHub Resources

| Resource Type | Quantity | Purpose |
|--------------|----------|---------|
| Spaces | 5 | base, filters, dev, staging, prod |
| Units | 17 per env | 8 deployments, 7 services, 1 namespace, 1 ingress |
| Filters | 7 | Layer-based and service-based filtering |
| Sets | 2 | critical-services, data-services |
| Workers | 3 | One per environment (dev, staging, prod) |

**Current Quota**: 97/100 spaces used
**Required**: 5 spaces
**Action**: Remove 25 BridgeWorker-blocked test spaces

### Kubernetes Resources

| Environment | Nodes | CPU | Memory | Storage |
|------------|-------|-----|--------|---------|
| Dev | 2 | 8 cores | 16 GB | 50 GB |
| Staging | 3 | 12 cores | 24 GB | 100 GB |
| Prod | 5 | 20 cores | 40 GB | 200 GB |

### Compute Estimates (per environment)

| Service | Replicas | CPU (req/limit) | Memory (req/limit) | Monthly Cost* |
|---------|----------|-----------------|-------------------|---------------|
| reference-data | 3 | 500m/1000m | 512Mi/1Gi | $25 |
| people-service | 2 | 200m/500m | 256Mi/512Mi | $15 |
| account-service | 2 | 200m/500m | 256Mi/512Mi | $15 |
| position-service | 2 | 500m/1000m | 512Mi/1Gi | $25 |
| trade-service | 3 | 1000m/2000m | 1Gi/2Gi | $75 |
| trade-processor | 1 | 500m/1000m | 512Mi/1Gi | $10 |
| trade-feed | 2 | 500m/1000m | 512Mi/1Gi | $25 |
| web-gui | 1 | 200m/500m | 256Mi/512Mi | $8 |
| **Total** | **16 pods** | **~5 cores** | **~8 GB** | **$198/month** |

*Estimated for AWS EKS t3.medium nodes

---

## Integration Points

### 1. ConfigHub ↔ Kubernetes
- **Method**: ConfigHub workers with auto-apply
- **Direction**: ConfigHub (source of truth) → Kubernetes (runtime)
- **Frequency**: 10-second poll interval
- **Fallback**: Manual `cub unit apply` commands

### 2. ConfigHub ↔ Git (Optional - Enterprise Mode)
- **Method**: ConfigHub → Git commits → Flux/Argo → Kubernetes
- **Use Case**: Compliance requirements, change approval workflows
- **Implementation**: Phase 7 (future enhancement)

### 3. TraderX Services ↔ DevOps Apps
- **Drift Detector**: Monitors TraderX units for configuration drift
- **Cost Optimizer**: Analyzes resource usage, provides AI recommendations
- **Integration**: Labels and filters for cross-app visibility

### 4. CI/CD Pipeline Integration
- **Trigger**: Git push to main branch
- **Flow**: Build image → Push to registry → Update ConfigHub unit → Worker auto-applies
- **Tools**: GitHub Actions, Docker, ConfigHub CLI

---

## Timeline & Milestones

### Detailed Schedule (10-hour implementation)

| Hour | Phase | Milestone | Agent Responsible |
|------|-------|-----------|-------------------|
| 0-1.5 | Phase 1 | ConfigHub infrastructure ready | Architecture Agent |
| 1.5-3.5 | Phase 2 | Dev environment deployed | Code Generator + Deployment Agent |
| 3.5-5.5 | Phase 3 | Staging and prod deployed | Deployment Agent |
| 5.5-7 | Phase 4 | Workers and automation configured | Code Generator Agent |
| 7-9 | Phase 5 | DevOps apps integrated | Integration Agent |
| 9-10 | Phase 6 | Monitoring and documentation complete | Monitoring + Documentation Agents |

### Parallel Execution Opportunities

**Hours 6-7** (Quality Gate 1):
- Security Review Agent → Validates RBAC, secrets management
- Code Review Agent → Validates scripts, idempotency
- Testing Agent → Runs integration tests

**Hours 8-9** (Continuous):
- Monitoring Agent → Sets up dashboards while services deploy
- Documentation Agent → Updates docs while deployment proceeds

---

## Success Metrics

### Deployment Success
- [ ] All 8 services deployed to 3 environments (24 deployments total)
- [ ] Zero kubectl commands used (100% ConfigHub-driven)
- [ ] All 12 canonical patterns implemented and documented
- [ ] Workers successfully auto-applying changes

### Performance Metrics
- **Deployment Time**: < 2 minutes per environment (target)
- **Promotion Time**: < 5 minutes dev→staging→prod (target)
- **Rollback Time**: < 30 seconds (target)
- **Drift Detection**: < 30 seconds from drift to correction (target)

### Quality Metrics
- **Service Availability**: 99.95% (critical services)
- **Health Check Success Rate**: 100%
- **Configuration Drift Events**: 0 (auto-corrected)
- **Failed Deployments**: 0%

### Business Metrics
- **Cost Visibility**: Per-service breakdown available
- **Audit Trail**: 100% of changes tracked in ConfigHub
- **Compliance**: All regulatory requirements met
- **Developer Experience**: Deployment complexity reduced 70%

---

## Risk Mitigation Strategy

### Technical Risks
1. **Service Dependency Failures**: Mitigated by ordered deployment with health checks
2. **Resource Constraints**: Mitigated by pre-deployment capacity planning
3. **Network Policies**: Mitigated by thorough testing in dev environment
4. **Data Persistence**: Mitigated by volume mount validation

### Operational Risks
1. **ConfigHub Quota**: Mitigated by immediate cleanup of test spaces
2. **Worker Failures**: Mitigated by fallback to manual apply
3. **Rollback Requirements**: Mitigated by tested rollback procedures
4. **Multi-cluster Sync**: Mitigated by environment isolation

### Financial Risks
1. **Cost Overruns**: Mitigated by cost optimizer recommendations
2. **Resource Over-provisioning**: Mitigated by AI-driven right-sizing

---

## Quality Gates

### Quality Gate 1: Pre-Deployment (Before Phase 2)
**Criteria**:
- [ ] ConfigHub infrastructure validated
- [ ] All YAML manifests syntax-checked
- [ ] Dependency ordering verified
- [ ] Security review passed
- [ ] Code review passed

**Go/No-Go Decision**: All criteria must pass

### Quality Gate 2: Post-Dev Deployment (Before Phase 3)
**Criteria**:
- [ ] All dev services Running
- [ ] Health checks passing
- [ ] Worker operational
- [ ] Integration tests passed

**Go/No-Go Decision**: All criteria must pass

### Quality Gate 3: Pre-Production (Before prod deployment)
**Criteria**:
- [ ] Staging environment stable for 24 hours
- [ ] Performance tests passed
- [ ] Security scan clean
- [ ] Rollback tested successfully

**Go/No-Go Decision**: All criteria must pass or executive override required

---

## Rollback Procedures

### Immediate Rollback (< 1 minute)
```bash
# Rollback single service to previous revision
cub unit apply --space $(bin/proj)-prod --unit trade-service --revision=N-1

# Verify rollback
cub unit get trade-service --space $(bin/proj)-prod
```

### Full Environment Rollback (< 5 minutes)
```bash
# Rollback entire environment using changesets
cub unit apply --space $(bin/proj)-prod --revision "ChangeSet:previous-release"

# Verify all services
bin/ordered-apply $(bin/proj)-prod
```

### Emergency Recovery (< 15 minutes)
```bash
# Destroy current deployment
cub unit destroy --space $(bin/proj)-prod --filter $(bin/proj)/all

# Re-apply from staging
bin/promote staging prod
bin/apply-all prod
```

---

## Next Steps After Completion

### Immediate (Week 1)
1. Performance tuning based on real usage
2. Optimization of resource allocations
3. Fine-tuning of monitoring alerts
4. Documentation of lessons learned

### Short-term (Month 1)
1. Implement Enterprise Mode (ConfigHub → Git → Flux/Argo)
2. Add multi-region support
3. Enhance cost optimization with ML models
4. Create self-service developer portal

### Long-term (Quarter 1)
1. Extend to additional FINOS projects
2. Create TraderX-specific DevOps apps (trade analytics, compliance)
3. Publish case study to FINOS community
4. Present at industry conferences

---

## Appendix: Command Reference

### Essential Commands
```bash
# View project structure
cub unit tree --node=space --filter $(bin/proj)/all --space '*'

# Check deployment status
cub unit list --space $(bin/proj)-dev --format json | jq '.[] | {name: .Slug, status: .LiveState.Status}'

# View live state
cub unit get-live-state trade-service --space $(bin/proj)-prod

# Promote changes
bin/promote dev staging

# Apply changes
bin/apply-all staging

# Rollback
cub unit apply --space $(bin/proj)-prod --unit UNIT --revision=N
```

### Troubleshooting Commands
```bash
# Debug worker
kubectl logs -n $(bin/proj)-dev deployment/confighub-worker

# Check service health
kubectl get pods -n $(bin/proj)-dev

# View ConfigHub unit diff
cub unit diff -u trade-service --space $(bin/proj)-staging --from=5
```

---

**Document Version**: 1.0
**Last Updated**: 2025-10-03
**Owner**: Planning Agent
**Status**: Ready for Architecture Agent Review
