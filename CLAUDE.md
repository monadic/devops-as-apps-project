# Claude Code Context for DevOps as Apps Project

## Critical Information
This project uses ConfigHub as its configuration management backend. Many features that might seem obvious DO NOT EXIST in ConfigHub. This file serves as your reference for what's real.

## REAL ConfigHub Features ✅
These are the ONLY features that actually exist:
- **Spaces** - Workspaces for configuration organization
- **Units** - Individual configuration items
- **Sets** - Groups of units for bulk operations
- **Filters** - WHERE clause queries
- **Upstream/Downstream** - Inheritance relationships via UpstreamUnitID
- **Push-upgrade** - Propagate changes with BulkPatchUnits(Upgrade: true)
- **Apply/Destroy** - Deploy and remove configurations
- **Live State** - Read-only deployment status

## HALLUCINATED Features ❌
DO NOT USE these - they don't exist:
- **Variants** - No aws-variant, gcp-variant functionality
- **Gates** - No promotion gates between environments
- **Dependency graphs** - No GetDependencyGraph() API
- **UpdateStatus** - Cannot update live status
- **CloneWithVariant** - This operation doesn't exist
- **UpgradeSet** - Use push-upgrade pattern instead

## Correct Patterns

### Multi-Environment (NOT variants)
```go
// ✅ CORRECT - Use upstream/downstream
unit := CreateUnit(Unit{
    UpstreamUnitID: &baseUnitID,
})

// ❌ WRONG - Variants don't exist
variant := GetVariant("aws-variant")
```

### Bulk Operations
```go
// ✅ CORRECT - Use Sets and Filters
set := CreateSet("critical-services")
filter := CreateFilter(CreateFilterRequest{
    Where: fmt.Sprintf("SetID = '%s'", set.SetID),
})

// ❌ WRONG - UpgradeSet doesn't exist
UpgradeSet(setID)
```

### Event-Driven Architecture
```go
// ✅ CORRECT - Use Kubernetes informers
app.RunWithInformers(func() error {
    detectDrift()
    return nil
})

// ❌ WRONG - Polling
for {
    detectDrift()
    time.Sleep(5 * time.Minute)
}
```

## Project Rules
1. ALWAYS check docs/CONFIGHUB-ACTUAL-FEATURES.md before using any ConfigHub API
2. Use Go, not Python
3. Use Kubernetes informers, not polling
4. Prefer Sets and Filters for bulk operations
5. Use push-upgrade pattern for propagation
6. Dev Mode: ConfigHub → Kubernetes (direct)
7. Enterprise Mode: ConfigHub → Git → Flux/Argo → Kubernetes

## Testing Commands
When implementing features, run these commands:
```bash
# Build
go build ./...

# Test
go test ./...

# Lint (if available)
golangci-lint run

# Format
go fmt ./...
```

## Key Documents
- `docs/CONFIGHUB-ACTUAL-FEATURES.md` - API reference (ALWAYS consult this)
- `docs/DEVOPS-AS-APPS-MASTER-PLAN.md` - Master implementation plan
- `docs/DEVOPS-AS-APPS-PLAN.md` - Detailed guide

## Remember
If you're about to use a ConfigHub feature, VERIFY it exists in the actual features document first. When in doubt, use only the confirmed operations: CreateSpace, CreateUnit, ApplyUnit, DestroyUnit, CreateSet, GetSet, CreateFilter, BulkPatchUnits, BulkApplyUnits.