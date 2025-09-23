# DevOps as Apps Platform

## The Big Idea
**We make every operational concern work like a modern app - continuously verified, persistently running, and able to use AI.**

## Overview
DevOps tools should be real applications, not just scripts or workflows. We build DevOps automation as persistent Kubernetes applications that run continuously, maintain state, use AI for decisions, and react to events in real-time.

## Start Here
1. **[docs/README.md](docs/README.md)** - Complete documentation guide
2. **[docs/EXECUTIVE-SUMMARY.md](docs/EXECUTIVE-SUMMARY.md)** - 5-minute overview
3. **[docs/DEVOPS-AS-APPS-MASTER-PLAN.md](docs/DEVOPS-AS-APPS-MASTER-PLAN.md)** - Core vision

## Why DevOps as Modern Apps?

### Traditional DevOps (Problems)
- **Scripts/Workflows**: Run and exit, forget everything
- **No Intelligence**: Rule-based, can't adapt
- **Fragmented**: 15+ tools that don't integrate
- **Reactive**: Wait for triggers or schedules

### DevOps as Apps (Solution)
- **Modern Applications**: Run continuously like any app
- **AI-Powered**: Claude provides intelligence
- **Unified Platform**: One pattern for everything
- **Event-Driven**: React immediately to changes

## Quick Start
1. **Review `docs/CONFIGHUB-ACTUAL-FEATURES.md` first** - understand what's real
2. Use only documented ConfigHub APIs
3. Follow Dev Mode for quick iteration
4. Follow Enterprise Mode for production

## ConfigHub Real Features ✅
- **Spaces & Units** - Core configuration management with full version history
- **Sets** - Group related units for bulk operations
- **Filters** - Powerful WHERE clauses for queries
- **Upstream/Downstream** - Configuration inheritance
- **Push-upgrade** - Propagate changes with `BulkPatchUnits(Upgrade: true)`
- **Apply/Destroy** - Deploy and remove configurations
- **Live State** - Track deployment status (read-only)
- **Version Control** - Every unit change is versioned (no Git needed for config versioning)
- **ApplyGates** - Approval workflows for deployments
- **Audit Trail** - Complete history of who changed what, when, and why

## ConfigHub Patterns That Work Differently Than Expected

### ✅ Variants DO Exist! (Through Clone + Edit)
- Create variants by cloning units and editing them
- Each clone can have different customizations = variants
- Clone upgrade preserves local changes
- Example: `aws-prod`, `gcp-prod`, `azure-prod` spaces with different configs

### ❌ What NOT to Use
- **CloneWithVariant API** - Not a specific operation (use clone + edit instead)
- **Gates** - Don't exist (no promotion gates, but ApplyGates do exist)
- **Dependency graphs** - Don't exist (no GetDependencyGraph)
- **UpdateStatus** - Can't update live status
- **UpgradeSet** - Use push-upgrade instead

## Two Deployment Modes

### Dev Mode (Direct)
```
ConfigHub → Kubernetes
```
- Direct apply from ConfigHub
- Faster feedback loops
- No Git intermediary

### Enterprise Mode (GitOps)
```
ConfigHub (versioning + approvals) → Git (bridge) → Flux/Argo → Kubernetes
```
- ConfigHub provides versioning and approval workflows
- Git serves as bridge to GitOps tools (Flux/Argo)
- ConfigHub maintains full audit trail
- No duplicate versioning (ConfigHub already has it)

## Project Structure
```
devops-as-apps-project/
├── .claude-code/         # Claude Code configuration
├── docs/                 # Planning and design documents
├── devops-sdk/          # Reusable SDK components
│   ├── app/             # Base DevOps app framework
│   ├── confighub/       # ConfigHub client (REAL APIs only)
│   ├── kubernetes/      # K8s utilities with informers
│   ├── claude/          # Claude AI integration
│   └── modes/           # Dev vs Enterprise modes
└── devops-examples/     # Example implementations
    ├── drift-detector/   # Using sets and filters
    ├── cost-optimizer/   # Using sets and filters
    ├── upgrade-manager/  # Using push-upgrade
    ├── security-scanner/ # Using filters and bulk ops
    ├── compliance-auditor/ # Using sets
    └── branch-deployer/  # Using upstream/downstream

## Development Guidelines

### Always Check Real APIs
Before implementing any ConfigHub operation, verify it exists in:
- `docs/CONFIGHUB-ACTUAL-FEATURES.md`
- The actual ConfigHub source code

### Use Informers, Not Polling
```go
// ❌ BAD - Polling
for {
    detectDrift()
    time.Sleep(5 * time.Minute)
}

// ✅ GOOD - Informers
app.RunWithInformers(func() error {
    detectDrift()
    return nil
})
```

### Multi-Environment Pattern
```go
// Use upstream/downstream, NOT variants
unit := CreateUnit(Unit{
    UpstreamUnitID: &baseUnitID,
})

// Push changes downstream
BulkPatchUnits(BulkPatchParams{
    Where: "UpstreamUnitID = X",
    Upgrade: true,
})
```

### Bulk Operations Pattern
```go
// Use Sets for grouping
set := CreateSet("frontend-services")

// Use Filters for targeting
filter := CreateFilter(CreateFilterRequest{
    Where: "Tags['tier'] = 'critical'",
})

// Bulk apply
BulkApplyUnits(BulkApplyParams{
    Where: fmt.Sprintf("SetID = '%s'", set.SetID),
})
```

## Testing Protocol

### Step 1: Local Tests First ⚡
```bash
cd devops-examples/drift-detector
go test -v              # Unit tests with mocks
./drift-detector demo   # Simulated workflow
```

### Step 2: Real Integration Tests 🔗
```bash
# Setup real infrastructure
export CUB_TOKEN="your-confighub-token"
kind create cluster --name devops-test

# Test with real services
go test -tags=integration -v
./drift-detector  # Real ConfigHub + Kind cluster
```

## Contributing
1. Always use real ConfigHub APIs
2. Follow 2-step testing protocol (local → real)
3. Test against confighub.com + local Kind cluster
4. Document any new patterns discovered
5. Update `docs/CONFIGHUB-ACTUAL-FEATURES.md` if you discover new APIs

## License
Proprietary - ConfigHub, Inc.