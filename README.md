# DevOps as Apps Platform

## Overview
Building DevOps automation as persistent Kubernetes applications using ConfigHub's actual features.

## Key Documents
- `docs/CONFIGHUB-ACTUAL-FEATURES.md` - What ConfigHub REALLY does (review this first!)
- `docs/DEVOPS-AS-APPS-MASTER-PLAN.md` - Master implementation plan
- `docs/DEVOPS-AS-APPS-PLAN.md` - Detailed implementation guide
- `docs/DEPLOY-FROM-BRANCH-PATTERN.md` - Branch deployment pattern
- `docs/UPGRADE-MANAGER-USE-CASE.md` - Upgrade management use case
- `docs/GITOPS-AS-APPS-COMPARISON.md` - GitOps comparison
- `docs/DEVOPS-AS-APPS-MARKET-STRATEGY.md` - Market positioning

## Quick Start
1. **Review `docs/CONFIGHUB-ACTUAL-FEATURES.md` first** - understand what's real
2. Use only documented ConfigHub APIs
3. Follow Dev Mode for quick iteration
4. Follow Enterprise Mode for production

## ConfigHub Real Features ✅
- **Spaces & Units** - Core configuration management
- **Sets** - Group related units for bulk operations
- **Filters** - Powerful WHERE clauses for queries
- **Upstream/Downstream** - Configuration inheritance
- **Push-upgrade** - Propagate changes with `BulkPatchUnits(Upgrade: true)`
- **Apply/Destroy** - Deploy and remove configurations
- **Live State** - Track deployment status (read-only)

## What NOT to Use ❌
- **Variants** - Don't exist (no aws-variant, gcp-variant)
- **Gates** - Don't exist (no promotion gates)
- **Dependency graphs** - Don't exist (no GetDependencyGraph)
- **UpdateStatus** - Can't update live status
- **CloneWithVariant** - Not a real operation
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
ConfigHub → Git → Flux/Argo → Kubernetes
```
- Full audit trail in Git
- Flux/Argo for compliance
- ConfigHub drives configuration

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

## Contributing
1. Always use real ConfigHub APIs
2. Test against actual ConfigHub instance
3. Document any new patterns discovered
4. Update `docs/CONFIGHUB-ACTUAL-FEATURES.md` if you discover new APIs

## License
Proprietary - ConfigHub, Inc.