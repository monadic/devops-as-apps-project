# DevOps as Apps: Comprehensive Implementation Plan

## Executive Summary

Based on our exploration of drift-detector and cost-optimizer applications, we've discovered a powerful pattern: **DevOps tools should be persistent Kubernetes applications, not ephemeral workflows**. This plan outlines how to scale this pattern into a comprehensive DevOps automation platform using ALL ConfigHub features:

- **Variants**: Multi-cloud/region configurations (aws-variant, gcp-variant)
- **Sets**: Grouped operations (critical-services, frontend-set)
- **Dependencies**: Topological ordering for safe deployments
- **Gates**: Control flow (business-hours, manual-approval)
- **Filters**: Targeted operations on configuration subsets
- **Two Deployment Modes**: Dev (direct) and Enterprise (GitOps)

## Key Insights from Our Journey

### 1. The Application Pattern Works
Both drift-detector and cost-optimizer follow the exact same pattern as business applications:
- **Kubernetes Deployments**: Run continuously like web services
- **External Integration**: Connect to Kubernetes, ConfigHub, and Claude AI
- **State Management**: Maintain context between runs
- **Full Lifecycle**: Versioning, deployment, monitoring, rollback

### 1.5 ConfigHub Self-Deployment Pattern (Critical!)
Following global-app pattern, **DevOps apps deploy themselves through ConfigHub**:
- **All K8s manifests are ConfigHub units** - not raw YAML files
- **Environment hierarchy** - base → dev → staging → prod
- **Push-upgrade for promotion** - not kubectl apply
- **Bulk operations** - deploy entire app stack with one command

```bash
# The NEW way (ConfigHub-driven):
bin/install-base      # Creates units in ConfigHub
bin/install-envs      # Sets up env hierarchy
bin/apply-all dev     # Deploys via ConfigHub
bin/promote dev staging  # Promotes with push-upgrade

# NOT the old way:
kubectl apply -f k8s/
```

### 2. Claude Integration is Event-Driven
The Claude integration uses informers, not polling:
```go
// Event-driven with Kubernetes informers
app.RunWithInformers(func() error {
    state := gatherSystemState()
    analysis := claudeClient.Analyze(state)
    createFixes(analysis)
    return nil
})
```

### 3. Two Deployment Modes

#### Dev Mode (Direct ConfigHub)
```yaml
ConfigHub (source of truth)
    ↓ (direct apply)
Kubernetes Cluster
```
- Faster feedback loops
- No Git intermediary
- Perfect for development and small teams

#### Enterprise Mode (GitOps)
```yaml
ConfigHub (source of truth + versioning + approvals)
    ↓ (sync)
Git Repository (GitOps bridge)
    ↓ (Flux/Argo)
Kubernetes Cluster
```
- ConfigHub maintains version history and approvals
- Git serves as bridge to GitOps tooling
- Flux/Argo for GitOps-specific compliance needs
- ConfigHub is still the source of truth

### 4. The SDK Pattern Accelerates Development
Common patterns extracted into `/Users/alexisrichardson/github-repos/devops-sdk/` dramatically reduce boilerplate and ensure consistency.

## Step-by-Step Implementation Plan

### Phase 1: Build Core Example Apps (4-6 weeks)

Each app MUST follow the standard structure with ConfigHub deployment:

```
security-scanner/
├── confighub/base/         # K8s manifests as ConfigHub units
├── bin/                    # Standard deployment scripts
│   ├── install-base        # Creates ConfigHub structure
│   ├── install-envs        # Sets up env hierarchy
│   ├── apply-all           # Deploys via ConfigHub
│   └── promote            # Promotes between envs
└── main.go                # App implementation
```

#### App 1: Security Scanner (Enhanced with ConfigHub Features)
```go
// /Users/alexisrichardson/github-repos/devops-examples/security-scanner/
func main() {
    app := sdk.NewDevOpsApp(sdk.DevOpsAppConfig{
        Name: "security-scanner",
        Environment: os.Getenv("ENV"),
        Mode: os.Getenv("MODE"), // "dev" or "enterprise"
    })

    app.RunWithInformers(func() error {
        // 1. Create filter for critical services
        filter := app.Cub.CreateFilter(app.Space, CreateFilterRequest{
            Where: "Tags['tier'] = 'critical' AND Tags['scan'] = 'required'",
        })

        // 2. Filter to only critical services
        filter := Filter{
            Labels: map[string]string{
                "tier": "critical",
                "scan": "required",
            },
            Status: "Live",
        }
        units := app.Cub.GetUnitsWithFilter(app.Space, filter)

        // 3. List units matching the filter
        units := app.Cub.ListUnits(app.Space, ListUnitsParams{
            FilterID: filter.FilterID,
        })

        // 4. Scan units for vulnerabilities
        vulnerabilities := scanUnits(units)

        // 5. Claude analyzes with constraints
        analysis := app.Claude.AnalyzeStructured(
            prompt: "Analyze security vulnerabilities",
            data: SecurityContext{
                Vulnerabilities: vulnerabilities,
                Units: units,
            },
            responseSchema: SecurityAnalysis{},
        )

        // 6. Apply fixes based on mode
        if app.Mode == "dev" {
            // Direct apply in dev mode
            app.K8s.ApplyFixes(analysis.Fixes)
            // Apply fixes
            for _, fix := range analysis.Fixes {
                app.Cub.ApplyUnit(app.Space, fix.UnitID)
            }
        } else {
            // Enterprise mode: sync to Git
            app.Git.CreatePR(analysis.Fixes, "Security fixes")
            app.Flux.Reconcile()
        }
        return nil
    })
}
```

**Purpose**: Scan container images for vulnerabilities, analyze with Claude, propose patches
**Value**: Automated security posture management

#### App 2: Compliance Auditor (Using Sets and Dependencies)
```go
// /Users/alexisrichardson/github-repos/devops-examples/compliance-auditor/
func main() {
    app := sdk.NewDevOpsApp(sdk.DevOpsAppConfig{
        Name: "compliance-auditor",
        Mode: os.Getenv("MODE"),
    })

    app.RunWithInformers(func() error {
        // 1. Use sets to audit grouped services
        sets := []string{"critical-services", "financial-services", "pii-services"}

        // 2. Create sets for grouped auditing
        for _, setName := range sets {
            app.Cub.CreateSet(app.Space, CreateSetRequest{
                Slug: setName,
            })
        }

        // 3. Check compliance by set
        for _, setName := range sets {
            // List units in the set
            units := app.Cub.ListUnits(app.Space, ListUnitsParams{
                Where: fmt.Sprintf("SetID = '%s'", setName),
            })

            for _, unit := range units {
                violations := auditUnit(unit, getComplianceRules(setName))

                if len(violations) > 0 {
                    // 4. Create and apply fixes
                    app.createComplianceFix(unit, violations)
                    app.Cub.ApplyUnit(app.Space, unit.UnitID)
                }
            }
        }

        // 5. Mark completion
        app.logComplianceStatus("Audit complete")
        return nil
    })
}
```

**Purpose**: Continuous compliance checking (SOC2, PCI-DSS, etc.)
**Value**: Automated compliance management and reporting

#### App 3: Capacity Planner (NEW)
```go
// /Users/alexisrichardson/github-repos/devops-examples/capacity-planner/
func main() {
    app := sdk.NewDevOpsApp(sdk.DevOpsAppConfig{
        Name: "capacity-planner",
        RunInterval: 4 * time.Hour,
    })

    app.Run(func() error {
        // Gather capacity metrics
        nodeMetrics := app.K8s.GetNodeMetrics()
        podMetrics := app.K8s.GetPodMetrics()
        trends := calculateTrends(nodeMetrics, podMetrics)

        // Claude predicts capacity needs
        forecast := app.Claude.AnalyzeJSON("Capacity planning", trends)

        // Create scaling recommendations
        createScalingPlan(app.Cub, forecast)
        return nil
    })
}
```

**Purpose**: Predict capacity needs and automate scaling decisions
**Value**: Prevent outages, optimize costs through intelligent scaling

### Phase 2: Extract Powerful SDK Components (2-3 weeks)

Based on patterns from 5 apps (drift-detector, cost-optimizer + 3 new), extract:

#### 2.1 Enhanced Base App Framework with All ConfigHub Features
```go
// /Users/alexisrichardson/github-repos/devops-sdk/app.go (enhanced)

type DevOpsApp struct {
    Name        string
    Environment string             // Current environment (qa, staging, prod)
    Mode        string             // "dev" or "enterprise"
    Space       string             // ConfigHub space
    K8s         *K8sClients        // Kubernetes integration
    Claude      *ClaudeClient      // AI analysis
    Cub         *ConfigHubClient   // Full ConfigHub features
    Git         *GitClient         // For enterprise mode
    Flux        *FluxClient        // For enterprise mode
    Metrics     *MetricsCollector  // Built-in metrics
    Alerting    *AlertManager      // Built-in alerting
}

// ConfigHub client with REAL features
type ConfigHubClient struct {
    // Spaces & Units
    CreateSpace(req CreateSpaceRequest) (*Space, error)
    CreateUnit(spaceID uuid.UUID, req CreateUnitRequest) (*Unit, error)
    ListUnits(spaceID uuid.UUID, params ListUnitsParams) ([]*Unit, error)
    ApplyUnit(spaceID, unitID uuid.UUID) error
    DestroyUnit(spaceID, unitID uuid.UUID) error
    // Sets (REAL)
    CreateSet(spaceID uuid.UUID, req CreateSetRequest) (*Set, error)
    GetSet(spaceID, setID uuid.UUID) (*Set, error)
    // Filters (REAL)
    CreateFilter(spaceID uuid.UUID, req CreateFilterRequest) (*Filter, error)
    // Push-upgrade
    BulkPatchUnits(params BulkPatchParams) error
    // Live State (read-only)
    GetUnitLiveState(spaceID, unitID uuid.UUID) (*LiveState, error)
}

// Run with informers (not polling!)
func (app *DevOpsApp) RunWithInformers(handler func() error) error {
    informer := app.K8s.NewInformer()
    informer.OnChange(func() {
        handler()
    })
    return informer.Start()
}
```

#### 2.2 Mode-Specific Deployment
```go
// Dev Mode: Direct ConfigHub → Kubernetes
type DevModeDeployer struct {
    cub *ConfigHubClient
    k8s *K8sClients
}

func (d *DevModeDeployer) Deploy(spaceID uuid.UUID) error {
    units := d.cub.ListUnits(spaceID)
    // Direct apply
    for _, unit := range units {
        d.cub.ApplyUnit(spaceID, unit.UnitID)
    }
    return nil
}

// Enterprise Mode: ConfigHub → Git → Flux → Kubernetes
type EnterpriseModeDeployer struct {
    cub  *ConfigHubClient
    git  *GitClient
    flux *FluxClient
}

func (e *EnterpriseModeDeployer) Deploy(spaceID uuid.UUID) error {
    units := e.cub.ListUnits(spaceID)
    // Sync to Git for audit trail
    e.git.CommitUnits(units, "Synced from ConfigHub")
    // Let Flux handle deployment
    e.flux.Reconcile()
    // Check live state
    for _, unit := range units {
        liveState := e.cub.GetUnitLiveState(spaceID, unit.UnitID)
        e.logState(liveState)
    }
    return nil
}
```

#### 2.3 Advanced Kubernetes Utilities
```go
// /Users/alexisrichardson/github-repos/devops-sdk/kubernetes.go (enhanced)

type K8sClients struct {
    // Current clients...
    PolicyClient  *PolicyV1Client    // NEW: Policy API
    NetworkClient *NetworkingClient  // NEW: Network policies
    StorageClient *StorageClient     // NEW: Storage classes, PVs
}

// Resource change tracking
func (k *K8sClients) WatchResourceChanges(gvr schema.GroupVersionResource, handler func(event)) {
    // Implement watch API for real-time change detection
}

// Bulk operations
func (k *K8sClients) BulkUpdate(resources []ResourceUpdate) error {
    // Efficient bulk updates
}
```

#### 2.4 Enhanced ConfigHub Integration
```go
// /Users/alexisrichardson/github-repos/devops-sdk/confighub.go (enhanced)

// Space hierarchy operations (following global-app pattern)
type SpaceOperations struct {
    // Create hierarchy like global-app does
    CreateHierarchy(base string, envs []string) error
    // Promote through environments
    PromoteThrough(app string, path []string) error
    // Clone for environment creation
    CloneForEnvironment(source, env string) (string, error)
}

// Config composition (key for "config as data")
func (c *CubClient) Compose(configs ...Config) Config {
    // Merge app + infra configs like global-app
    result := Config{}
    for _, cfg := range configs {
        result = c.mergeConfigs(result, cfg)
    }
    return result
}

// Query operations for "config as data"
func (c *CubClient) Diff(space1, space2 string) []Difference {
    // Compare configs between spaces
}

func (c *CubClient) GetHistory(unit string, duration time.Duration) []Change {
    // Query config history
}

// Promote DevOps app through hierarchy
func (c *CubClient) PromoteDevOpsApp(app string, from, to string) error {
    fromSpace := fmt.Sprintf("%s-%s", app, from)
    toSpace := fmt.Sprintf("%s-%s", app, to)
    return c.CloneUpgrade(fromSpace, toSpace)
}
```

### Phase 3: Consider Claude CRD for Kubernetes (3-4 weeks)

Create a native Kubernetes way to define Claude-powered automation:

```yaml
# /Users/alexisrichardson/github-repos/claude-operator/examples/drift-analysis.yaml
apiVersion: claude.ai/v1
kind: ClaudeAnalysis
metadata:
  name: drift-analysis
  namespace: devops-apps
spec:
  schedule: "*/5 * * * *"  # Every 5 minutes
  prompt: |
    Analyze Kubernetes configuration drift and suggest fixes.
    Look for differences between desired and actual state.

  inputs:
    - type: configHub
      space: "{{.ConfigHubSpace}}"
    - type: kubernetes
      resources: ["deployments", "services", "configmaps"]
      namespace: "{{.TargetNamespace}}"

  analysis:
    model: "claude-3-opus-20240229"
    temperature: 0
    responseFormat: "json"
    schema: |
      {
        "has_drift": "boolean",
        "items": [{"resource": "string", "issue": "string"}],
        "fixes": [{"action": "string", "target": "string"}]
      }

  actions:
    - type: configHubSpace
      when: "analysis.has_drift"
      space: "{{.ConfigHubSpace}}-drift-fix-{{.Timestamp}}"
      content: "{{.analysis.fixes}}"

    - type: alert
      when: "analysis.has_drift"
      severity: "warning"
      message: "Configuration drift detected: {{.analysis.summary}}"

---
apiVersion: claude.ai/v1
kind: ClaudeAnalysis
metadata:
  name: cost-optimization
spec:
  schedule: "0 */6 * * *"  # Every 6 hours
  prompt: |
    Analyze resource usage and suggest cost optimizations.
    Focus on rightsizing and identifying waste.

  inputs:
    - type: kubernetes
      resources: ["deployments", "pods"]
      includeMetrics: true
    - type: prometheus
      queries:
        - "container_cpu_usage_seconds_total"
        - "container_memory_working_set_bytes"

  analysis:
    model: "claude-3-opus-20240229"
    responseFormat: "json"

  actions:
    - type: report
      format: "markdown"
      destination: "s3://reports/cost-optimization/"

    - type: configHubSpace
      when: "analysis.potential_savings > 100"
      space: "cost-optimization-{{.Date}}"
```

**Benefits of Claude CRD Approach:**
- **Declarative**: Define analysis as code
- **Kubernetes-native**: Fits existing GitOps workflows
- **Reusable**: Template-based analysis definitions
- **Observable**: Standard Kubernetes monitoring/logging

### Phase 4: Dual-Mode Support (2-3 weeks)

Support both Direct ConfigHub and GitOps modes:

#### 4.1 Direct ConfigHub Mode (Current)
```go
// /Users/alexisrichardson/github-repos/devops-sdk/modes.go

type DirectMode struct {
    cub    *CubClient
    k8s    *K8sClients
    claude *ClaudeClient
}

func (m *DirectMode) DetectDrift() (*DriftAnalysis, error) {
    desired := m.cub.GetUnits(space)
    actual := m.k8s.GetResources()
    return m.claude.CompareSources(desired, actual)
}

func (m *DirectMode) CreateFix(analysis *DriftAnalysis) error {
    fixSpace := fmt.Sprintf("%s-fix-%d", space, time.Now().Unix())
    return m.cub.CreateSpace(fixSpace, analysis.Fixes)
}
```

#### 4.2 GitOps Mode (Enhanced)
```go
type GitOpsMode struct {
    flux   *FluxClient   // or ArgoClient
    git    *GitClient
    cub    *CubClient
    claude *ClaudeClient
}

func (m *GitOpsMode) DetectDrift() (*DriftAnalysis, error) {
    // Read Flux/Argo status for sophisticated drift detection
    fluxStatus := m.flux.GetKustomizationStatus()
    return m.claude.AnalyzeFluxDrift(fluxStatus)
}

func (m *GitOpsMode) CreateFix(analysis *DriftAnalysis) error {
    // Create fix in ConfigHub, then sync to Git
    m.cub.CreateSpace(fixSpace, analysis.Fixes)
    return m.syncToGit(fixSpace)
}
```

#### 4.3 Unified Interface
```go
type DevOpsMode interface {
    DetectDrift() (*DriftAnalysis, error)
    CreateFix(*DriftAnalysis) error
    GetStatus() (*ModeStatus, error)
}

func NewDevOpsApp(config DevOpsAppConfig) (*DevOpsApp, error) {
    var mode DevOpsMode

    if config.GitOpsEnabled {
        mode = NewGitOpsMode(config)
    } else {
        mode = NewDirectMode(config)
    }

    return &DevOpsApp{Mode: mode}, nil
}
```

## Technical Implementation Details

### Core Architecture Principles

1. **Apps, Not Scripts**: Every DevOps tool is a long-running Kubernetes application
2. **Claude-Native**: AI analysis is a first-class citizen, not an afterthought
3. **State-Aware**: Apps maintain context and learn over time
4. **ConfigHub-Centric**: Treat ConfigHub as the source of truth
5. **Mode-Flexible**: Support both direct and GitOps workflows

### Development Standards

#### Space Hierarchy Structure
Every DevOps app follows the global-app pattern:
```
drift-detector-base
    └── drift-detector-qa
        └── drift-detector-staging
            └── drift-detector-prod

cost-optimizer-base
    └── cost-optimizer-qa
        └── cost-optimizer-staging
            └── cost-optimizer-prod
```

#### File Structure
```
/Users/alexisrichardson/github-repos/
├── devops-sdk/                    # Reusable SDK
│   ├── app.go                     # Base app framework
│   ├── claude.go                  # AI integration
│   ├── confighub.go              # ConfigHub client with hierarchy
│   ├── kubernetes.go             # K8s utilities
│   ├── space-ops.go              # NEW: Space hierarchy operations
│   ├── modes.go                  # Direct vs GitOps modes
│   └── crds/                     # Claude CRD definitions
├── devops-examples/               # Example applications (each with hierarchy)
│   ├── drift-detector/           # base → qa → staging → prod
│   ├── cost-optimizer/           # base → qa → staging → prod
│   ├── security-scanner/         # base → qa → staging → prod
│   ├── compliance-auditor/       # base → qa → staging → prod
│   └── capacity-planner/         # base → qa → staging → prod
└── claude-operator/               # Claude CRD operator
    ├── controllers/
    ├── webhooks/
    └── examples/
```

#### Code Patterns
```go
// Every app follows this pattern (with hierarchy)
func main() {
    config := sdk.DevOpsAppConfig{
        Name:        "my-devops-app",
        Version:     "1.0.0",
        Environment: os.Getenv("ENV"), // qa, staging, or prod
        Hierarchy:   []string{"base", "qa", "staging", "prod"},
        Mode:        sdk.DetectMode(), // Auto-detect Direct vs GitOps
    }

    app, err := sdk.NewDevOpsApp(config)
    if err != nil {
        log.Fatal(err)
    }

    // Implement your specific logic
    app.RunWithState(func(state *sdk.AppState) error {
        // 1. Gather data
        data := gatherData(app)

        // 2. Analyze with Claude
        analysis, err := app.Claude.AnalyzeWithContext(
            "Analyze this data for issues",
            data,
            state.Context,
            &MyAnalysis{},
        )

        // 3. Take action
        if analysis.HasIssues {
            app.Mode.CreateFix(analysis)
        }

        // 4. Update state for next run
        state.Context.LastAnalysis = analysis
        return nil
    })
}
```

### Deployment Strategy

#### GitOps Deployment
```yaml
# All DevOps apps deploy via ConfigHub → Git → Flux/Argo
apiVersion: apps/v1
kind: Deployment
metadata:
  name: drift-detector
  namespace: devops-apps
spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: drift-detector
        image: devops-apps/drift-detector:v1.0.0
        env:
        - name: CLAUDE_API_KEY
          valueFrom:
            secretKeyRef:
              name: claude-credentials
              key: api-key
        - name: CUB_TOKEN
          valueFrom:
            secretKeyRef:
              name: confighub-credentials
              key: token
        - name: MODE
          value: "gitops"  # or "direct"
```

#### Monitoring & Observability
```yaml
# Prometheus metrics for all DevOps apps
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: devops-apps
spec:
  selector:
    matchLabels:
      app.kubernetes.io/component: devops-app
  endpoints:
  - port: metrics
    path: /metrics
    interval: 30s
```

## Success Metrics

### Phase 1 Success Criteria
- [ ] 3 new DevOps apps deployed and running
- [ ] Each app demonstrates Claude integration
- [ ] Each app creates ConfigHub spaces for fixes
- [ ] All apps follow consistent patterns

### Phase 2 Success Criteria
- [ ] SDK reduces new app development time by 75%
- [ ] Common patterns extracted and reusable
- [ ] Zero boilerplate for basic DevOps app
- [ ] SDK has comprehensive test coverage

### Phase 3 Success Criteria
- [ ] Claude CRD operator running in cluster
- [ ] Declarative analysis definitions working
- [ ] CRD integrates with existing GitOps workflows
- [ ] Performance better than polling approach

### Phase 4 Success Criteria
- [ ] Both Direct and GitOps modes fully functional
- [ ] Mode switching is seamless
- [ ] ConfigHub integration consistent across modes
- [ ] Enterprise customers can use GitOps mode

## Risk Mitigation

### Technical Risks
1. **Claude API Rate Limits**: Implement intelligent batching and caching
2. **Kubernetes API Load**: Use watch APIs instead of polling where possible
3. **State Management**: Use persistent volumes for critical app state
4. **Secret Management**: Integrate with Kubernetes secrets and external secret stores

### Operational Risks
1. **App Reliability**: Implement comprehensive health checks and auto-restart
2. **Monitoring**: Full observability from day one
3. **Rollback**: All apps support blue/green deployment via ConfigHub
4. **Security**: Regular security scans and least-privilege access

## Conclusion

This plan transforms DevOps automation from scripts and workflows into a **platform of intelligent applications**. By treating DevOps tools as first-class applications with Claude AI integration, we create:

1. **Intelligent Automation**: AI-powered analysis and decision making
2. **Operational Excellence**: Standard deployment, monitoring, and lifecycle management
3. **Developer Velocity**: SDK dramatically reduces time-to-value
4. **Enterprise Ready**: Supports both simple (Direct) and enterprise (GitOps) workflows

The pattern we discovered with drift-detector and cost-optimizer is not just a proof of concept—it's the foundation for a new approach to DevOps automation that is more reliable, intelligent, and maintainable than traditional approaches.

**Next Step**: Begin Phase 1 by building the security-scanner application using the existing SDK patterns.