# DevOps as Apps: Master Plan with Actual ConfigHub Features

## Executive Summary

DevOps automation should be built as persistent Kubernetes applications that leverage ConfigHub's actual feature set: **spaces, units, sets, filters, and upstream/downstream relationships**. This plan provides two deployment modes:

- **Dev Mode**: Direct ConfigHub → Kubernetes (simpler, faster)
- **Enterprise Mode**: ConfigHub → Git → Flux/Argo → Kubernetes (audit trail, GitOps compliance)

## Two Deployment Modes

### Dev Mode (Direct ConfigHub)
```yaml
ConfigHub (source of truth)
    ↓ (direct apply)
Kubernetes Cluster
```
- Faster feedback loops
- No Git intermediary
- Perfect for development and small teams
- ConfigHub is the only source of truth

### Enterprise Mode (GitOps)
```yaml
ConfigHub (configuration source)
    ↓ (sync)
Git Repository (audit trail)
    ↓ (Flux/Argo)
Kubernetes Cluster
```
- Full audit trail in Git
- Flux/Argo for enterprise compliance
- ConfigHub still drives configuration
- Git provides versioning and approval workflows

## Chapter 1: Actual ConfigHub Feature Utilization

### 1.1 Spaces and Units
Every DevOps app uses spaces and units:
```yaml
# Spaces contain units
Space: drift-detector-prod
  └── Units:
      ├── drift-detector-config
      ├── drift-detector-deployment
      └── drift-detector-service

# Units can have upstream/downstream relationships
Upstream: base-drift-detector → Downstream: prod-drift-detector
```

### 1.2 Upstream/Downstream for Configuration Inheritance
```go
type DriftDetector struct {
    *sdk.DevOpsApp
    upstreamUnitID string // Base configuration
}

func (d *DriftDetector) CreateFromUpstream(environment string) {
    // Create unit with upstream relationship
    unit := d.Cub.CreateUnit(Unit{
        Slug: fmt.Sprintf("drift-detector-%s", environment),
        UpstreamUnitID: d.upstreamUnitID,
    })

    // Push-upgrade propagates changes from upstream
    d.Cub.PushUpgrade(d.upstreamUnitID)
}

// Multi-environment using upstream/downstream
func (d *DriftDetector) PropagateChanges() {
    // Changes flow from upstream to downstream
    d.Cub.BulkPatchUnits(BulkPatchParams{
        Where: fmt.Sprintf("UpstreamUnitID = '%s'", d.upstreamUnitID),
        Upgrade: true,
    })
}
```

### 1.3 Sets for Grouped Operations (REAL)
```go
// Sets group related units
func (u *UpgradeManager) OperateOnSet(setName string) {
    // Create a set
    set := u.Cub.CreateSet(u.spaceID, CreateSetRequest{
        Slug: setName, // e.g., "frontend-services"
    })

    // Add units to the set
    units := u.Cub.ListUnits(u.spaceID)
    for _, unit := range units {
        if strings.Contains(unit.Slug, "frontend") {
            u.Cub.UpdateUnit(unit.UnitID, UpdateUnitRequest{
                SetID: set.SetID,
            })
        }
    }

    // Bulk apply all units in the set
    u.Cub.BulkApplyUnits(BulkApplyParams{
        Where: fmt.Sprintf("SetID = '%s'", set.SetID),
    })
}

```

### 1.4 Upstream/Downstream Relationships
```go
// Units track upstream relationships for inheritance
func (d *DeployManager) ManageUpstream() {
    // Units have UpstreamUnitID and UpstreamRevisionNum
    units := d.Cub.ListUnits(d.spaceID)

    for _, unit := range units {
        if unit.UpstreamUnitID != nil {
            // This unit inherits from an upstream
            upstream := d.Cub.GetUnit(*unit.UpstreamUnitID)

            // Check if downstream needs updating
            if unit.UpstreamRevisionNum < upstream.HeadRevisionNum {
                // Push upgrade to sync downstream
                d.Cub.PushUpgrade(unit.UpstreamUnitID)
            }
        }
    }
}
```

### 1.5 Filters for Targeted Operations (REAL)
```go
// Filters enable powerful queries
func (app *DevOpsApp) UseFilters() {
    // Create a filter for specific units
    filter := app.Cub.CreateFilter(app.spaceID, CreateFilterRequest{
        Slug: "production-services",
        Where: "Slug LIKE 'prod-%' AND ToolchainType = 'Kubernetes/YAML'",
    })

    // Use filter to list specific units
    units := app.Cub.ListUnits(app.spaceID, ListUnitsParams{
        FilterID: filter.FilterID,
    })

    // Bulk operations with WHERE clauses
    app.Cub.BulkApplyUnits(BulkApplyParams{
        Where: "SetID = 'frontend-set' AND Tags['environment'] = 'production'",
    })
}
```

### 1.6 Apply and Destroy Operations
```go
// Apply units to targets (Kubernetes, cloud providers)
func (c *CostOptimizer) ApplyOptimizations() {
    // Apply a single unit
    c.Cub.ApplyUnit(c.spaceID, unitID, ApplyParams{
        TargetID: c.targetID,
    })

    // Bulk apply with filters
    c.Cub.BulkApplyUnits(BulkApplyParams{
        Where: "SetID = 'cost-optimized-set'",
    })

    // Destroy units when no longer needed
    c.Cub.DestroyUnit(c.spaceID, unitID)

    // Bulk destroy
    c.Cub.BulkDestroyUnits(BulkDestroyParams{
        Where: "Tags['lifecycle'] = 'temporary'",
    })
}
```

### 1.7 Live State Tracking
```go
// ConfigHub tracks deployment state (read-only)
func (app *DevOpsApp) CheckLiveState() {
    units := app.Cub.ListUnits(app.spaceID)

    for _, unit := range units {
        // Get live state of the unit
        liveState := app.Cub.GetUnitLiveState(app.spaceID, unit.UnitID)

        if liveState.Status != "Applied" {
            app.Alert("Unit not live: %s", unit.Slug)
        }

        // If drift detected, reapply
        if liveState.DriftDetected {
            app.Cub.ApplyUnit(app.spaceID, unit.UnitID)
        }
    }
}
```

## Chapter 2: DevOps Apps Using Actual Features

### 2.1 Drift Detector (Using Real ConfigHub)
```go
func (d *DriftDetector) DetectDrift() error {
    // 1. Get units in the set
    criticalSet := d.Cub.GetSet(d.spaceID, "critical-services")

    // 2. List units with filter
    filter := d.Cub.CreateFilter(d.spaceID, CreateFilterRequest{
        Where: fmt.Sprintf("SetID = '%s'", criticalSet.SetID),
    })
    units := d.Cub.ListUnits(d.spaceID, ListUnitsParams{
        FilterID: filter.FilterID,
    })

    // 3. Check each unit's live state
    for _, unit := range units {
        liveState := d.Cub.GetUnitLiveState(d.spaceID, unit.UnitID)

        if liveState.DriftDetected {
            // 4. Create fix by updating unit
            d.Cub.UpdateUnit(unit.UnitID, UpdateUnitRequest{
                Data: d.generateFix(liveState.Drift),
            })

            // 5. Reapply the unit
            d.Cub.ApplyUnit(d.spaceID, unit.UnitID)
        }
    }

    return nil
}
```

### 2.2 Cost Optimizer (Using Real Features)
```go
func (c *CostOptimizer) Optimize() error {
    // 1. Create filter for non-critical services
    filter := c.Cub.CreateFilter(c.spaceID, CreateFilterRequest{
        Slug: "cost-optimization-filter",
        Where: "Tags['tier'] = 'non-critical' AND Tags['optimize'] = 'true'",
    })

    // 2. List units matching filter
    units := c.Cub.ListUnits(c.spaceID, ListUnitsParams{
        FilterID: filter.FilterID,
    })

    // 3. Analyze and optimize each unit
    for _, unit := range units {
        optimization := c.analyzeUnit(unit)

        // 4. Update unit with optimized config
        c.Cub.UpdateUnit(unit.UnitID, UpdateUnitRequest{
            Data: optimization.NewConfig,
        })

        // 5. Apply the optimized unit
        c.Cub.ApplyUnit(c.spaceID, unit.UnitID)
    }

    return nil
}
```

### 2.3 Branch Deployer (Using Upstream/Downstream)
```go
func (b *BranchDeployer) DeployBranch(branch string) error {
    // 1. Create units for branch with upstream relationship
    baseUnitID := b.getBaseUnitID()

    branchUnit := b.Cub.CreateUnit(b.spaceID, CreateUnitRequest{
        Slug: fmt.Sprintf("branch-%s", branch),
        UpstreamUnitID: &baseUnitID,
        Data: b.getBranchConfig(branch),
    })

    // 2. Apply the branch unit
    b.Cub.ApplyUnit(b.spaceID, branchUnit.UnitID)

    // 3. When base changes, push upgrade to branches
    b.Cub.BulkPatchUnits(BulkPatchParams{
        Where: fmt.Sprintf("UpstreamUnitID = '%s'", baseUnitID),
        Upgrade: true, // This triggers push-upgrade
    })

    return nil
}
```

## Chapter 3: Dev Mode vs Enterprise Mode

### Dev Mode Implementation
```go
// Dev mode: Direct ConfigHub to Kubernetes
type DevModeApp struct {
    *sdk.DevOpsApp
    mode string // "dev"
}

func (d *DevModeApp) Deploy() error {
    // Direct from ConfigHub to Kubernetes
    units := d.Cub.GetUnits(d.space)

    for _, unit := range units {
        // Apply directly to cluster
        d.K8s.Apply(unit)

        // Unit is now applied
    }

    return nil
}

func (d *DevModeApp) Promote() error {
    // Simple clone upgrade in dev mode
    return d.Cub.CloneUpgrade("qa", "staging")
}
```

### Enterprise Mode Implementation
```go
// Enterprise mode: ConfigHub → Git → Flux/Argo → Kubernetes
type EnterpriseModeApp struct {
    *sdk.DevOpsApp
    mode    string // "enterprise"
    git     *GitClient
    flux    *FluxClient // or ArgoClient
}

func (e *EnterpriseModeApp) Deploy() error {
    // 1. Get units from ConfigHub
    units := e.Cub.GetUnits(e.space)

    // 2. Sync to Git for audit trail
    e.git.CommitUnits(units, "Synced from ConfigHub")

    // 3. Let Flux/Argo handle deployment
    e.flux.Reconcile()

    // 4. Check live state from ConfigHub
    for _, unit := range units {
        liveState := e.Cub.GetUnitLiveState(e.space, unit.UnitID)
        e.logState(liveState)
    }

    return nil
}

func (e *EnterpriseModeApp) Promote() error {
    // Enterprise flow with approvals
    pr := e.git.CreatePR("qa", "staging", "Promotion")

    // Wait for manual approval (outside ConfigHub)
    if !e.waitForManualApproval() {
        return fmt.Errorf("promotion rejected")
    }

    e.git.MergePR(pr)
    e.flux.Reconcile()

    return nil
}
```

## Chapter 4: SDK with Full ConfigHub Support

### Actual SDK Structure (Based on Real ConfigHub)
```go
// devops-sdk/confighub.go
type CubClient struct {
    baseURL string
    token   string
    mode    string // "dev" or "enterprise"
}

// REAL ConfigHub operations
type ConfigHubOperations interface {
    // Spaces
    CreateSpace(req CreateSpaceRequest) (*Space, error)
    GetSpace(spaceID uuid.UUID) (*Space, error)
    ListSpaces(params ListSpacesParams) ([]*Space, error)

    // Units
    CreateUnit(spaceID uuid.UUID, req CreateUnitRequest) (*Unit, error)
    GetUnit(spaceID, unitID uuid.UUID) (*Unit, error)
    UpdateUnit(unitID uuid.UUID, req UpdateUnitRequest) (*Unit, error)
    ListUnits(spaceID uuid.UUID, params ListUnitsParams) ([]*Unit, error)
    ApplyUnit(spaceID, unitID uuid.UUID) error
    DestroyUnit(spaceID, unitID uuid.UUID) error
    BulkApplyUnits(params BulkApplyParams) error
    BulkPatchUnits(params BulkPatchParams) error // For push-upgrade

    // Sets (REAL)
    CreateSet(spaceID uuid.UUID, req CreateSetRequest) (*Set, error)
    GetSet(spaceID, setID uuid.UUID) (*Set, error)
    UpdateSet(setID uuid.UUID, req UpdateSetRequest) (*Set, error)

    // Filters (REAL)
    CreateFilter(spaceID uuid.UUID, req CreateFilterRequest) (*Filter, error)
    ListUnits(spaceID uuid.UUID, params ListUnitsParams) ([]*Unit, error)

    // Live State (read-only)
    GetUnitLiveState(spaceID, unitID uuid.UUID) (*LiveState, error)
}
```

## Chapter 5: Market Positioning

### For Developers (Dev Mode)
- **Simple**: Direct ConfigHub → Kubernetes
- **Fast**: No Git delays
- **Powerful**: Full ConfigHub features
- **Perfect for**: Startups, dev teams, rapid iteration

### For Enterprises (Enterprise Mode)
- **Compliant**: Full GitOps audit trail
- **Governed**: Gates and approvals
- **Scalable**: Flux/Argo for large deployments
- **Perfect for**: Regulated industries, large teams

### Real Differentiators
1. **Sets**: Group related units for bulk operations
2. **Filters**: Powerful WHERE clauses for targeted operations
3. **Upstream/Downstream**: Configuration inheritance via push-upgrade
4. **Live State**: Track actual deployment status
5. **Bulk Operations**: Apply/destroy multiple units efficiently
6. **Dual Mode**: Dev simplicity OR enterprise governance
7. **AI Integration**: Claude analyzes and suggests optimizations

## Chapter 6: File Structure

```
/github-repos/
├── devops-sdk/              # Reusable SDK
│   ├── app.go              # Base framework with informers
│   ├── claude.go           # AI integration with constraints
│   ├── confighub.go        # Full ConfigHub features
│   ├── kubernetes.go       # K8s utilities with informers
│   └── modes.go            # Dev vs Enterprise modes
├── devops-examples/         # Example apps
│   ├── drift-detector/      # Uses sets, filters
│   ├── cost-optimizer/      # Uses sets, filters
│   ├── security-scanner/    # Uses filters, bulk ops
│   ├── compliance-auditor/  # Uses sets, filters
│   ├── upgrade-manager/     # Uses push-upgrade
│   └── branch-deployer/     # Uses upstream/downstream
└── claude-operator/         # Optional CRD
    ├── controllers/
    └── examples/
```

## Chapter 7: Implementation Strategy

### Phase 1: Build Core Example Apps (4-6 weeks)
- drift-detector (using sets and filters)
- cost-optimizer (using sets and filters)
- security-scanner (using filters and bulk apply)
- compliance-auditor (using sets)
- upgrade-manager (using push-upgrade)
- branch-deployer (using upstream/downstream)

### Phase 2: Extract SDK Components (2-3 weeks)
- Base DevOpsApp framework with informers
- ConfigHub client with REAL operations
- Mode-specific deployers (Dev vs Enterprise)
- Claude integration with constraints
- Proper error handling and retries

### Phase 3: Claude CRD (Optional, 3-4 weeks)
```yaml
apiVersion: claude.ai/v1
kind: ClaudeAnalysis
metadata:
  name: drift-analysis
  namespace: devops-apps
spec:
  schedule: "*/5 * * * *"
  prompt: |
    Analyze drift and suggest fixes
  mode: dev  # or enterprise
  spaceID: "abc-123"
  setID: "critical-services"  # Operate on this set
  filterWhere: "Tags['monitor'] = 'true'"  # Filter units
```

### Phase 4: Market Launch (2-3 weeks)
- Dev Mode for startups and small teams
- Enterprise Mode for large organizations
- Migration tools for vendor consolidation

### Success Metrics
- **Phase 1**: 6 apps using full ConfigHub features
- **Phase 2**: SDK reduces dev time by 75%
- **Phase 3**: CRD enables declarative DevOps
- **Phase 4**: Both modes production-ready

## Next Steps

1. Start with drift-detector in Dev Mode
2. Implement remaining apps with full features
3. Extract common patterns into SDK
4. Add Enterprise Mode with Git/Flux integration
5. Consider CRD for declarative definitions
6. Launch with vendor consolidation story