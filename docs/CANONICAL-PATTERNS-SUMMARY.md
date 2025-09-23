# Canonical Patterns Summary: DevOps as Apps with ConfigHub

## Overview

This document summarizes the canonical patterns from the global-app reference implementation that ALL DevOps apps must follow.

## Core Principles

1. **Persistent over Ephemeral**: Apps run continuously, not just when triggered
2. **Hierarchical over Flat**: Full environment inheritance (base → qa → staging → prod)
3. **Event-driven over Polling**: Kubernetes informers for immediate reaction
4. **Intelligent over Rule-based**: Claude AI for adaptive decisions
5. **Stateful over Stateless**: ConfigHub tracks all history

## Canonical Patterns (MUST FOLLOW)

### 1. Unique Project Naming
```bash
# ALWAYS use this for new projects
prefix=$(cub space new-prefix)  # Returns e.g., "chubby-paws"
project="${prefix}-{app-name}"
```

### 2. Space Hierarchy
```yaml
{prefix}-{app}-base              # Base configuration, no target
    └── {prefix}-{app}-qa       # QA with cluster target
        └── {prefix}-{app}-staging   # Staging
            └── {prefix}-{app}-prod  # Production

# Separate infrastructure space
{prefix}-infra                   # Platform components
```

### 3. Filter Creation
```bash
# Standard filters every app needs
cub filter create all Unit --where-field "Space.Labels.project = '$project'"
cub filter create app Unit --where-field "Labels.type='app'"
cub filter create infra Unit --where-field "Labels.type='infra'"
```

### 4. Environment Cloning
```bash
# Clone with upstream relationship (NOT variants)
cub unit create --dest-space $project-qa \
  --space $project-base \
  --filter $project/app \
  --label targetable=true
```

### 5. Version Promotion
```bash
# Set version in environment
cub run set-image-reference \
  --container-name app \
  --image-reference :1.1.3 \
  --space $(bin/proj)-qa

# Promote with push-upgrade
cub unit update --patch --upgrade --space $(bin/proj)-staging

# Apply changes
cub unit apply --space $(bin/proj)-staging
```

### 6. Bulk Operations with Sets
```go
// Create set for grouping
set := CreateSet("critical-services")

// Add units to set
UpdateUnit(unitID, UpdateUnitRequest{
    SetID: set.SetID,
})

// Bulk apply
BulkApplyUnits(BulkApplyParams{
    Where: fmt.Sprintf("SetID = '%s'", set.SetID),
})
```

### 7. Event-Driven Architecture
```go
// ALWAYS use informers, not polling
app.RunWithInformers(func() error {
    // React to changes immediately
    return processChanges()
})
```

## ConfigHub Features We Use (REAL)

### ✅ Features That Exist:
- **Spaces**: Workspaces with unique prefixes
- **Units**: Configuration items
- **Sets**: Groups for bulk operations
- **Filters**: WHERE clause queries
- **Upstream/Downstream**: Inheritance via UpstreamUnitID
- **Push-upgrade**: BulkPatchUnits(Upgrade: true)
- **Apply/Destroy**: Deploy and remove
- **Live State**: Read-only status

### ❌ Features That DON'T Exist:
- **Variants**: No aws-variant, gcp-variant
- **Gates**: No promotion gates
- **Dependency graphs**: Use Sets/Filters instead
- **UpdateStatus**: Cannot update live status
- **CloneWithVariant**: Edit after cloning
- **UpgradeSet**: Use push-upgrade pattern

## Competitive Advantages vs Cased.com

| Feature | Cased (Workflows) | Our Apps (Persistent) |
|---------|------------------|----------------------|
| **Execution** | Ephemeral, exits | Continuous with informers |
| **Environment Cloning** | "Killer branch deploy" (temp) | Permanent hierarchy |
| **Promotion** | Manual copy | Push-upgrade propagation |
| **Drift Correction** | None | Auto-correction with Sets |
| **Cost Analysis** | One-time script | Continuous AI optimization |
| **State** | Stateless | Full ConfigHub history |
| **Rollback** | Redeploy | Instant revision switch |

## Standard App Structure

Every DevOps app MUST have:

```
{app-name}/
├── confighub/
│   └── base/
│       ├── namespace.yaml
│       ├── {app}-deployment.yaml
│       ├── {app}-service.yaml
│       └── {app}-rbac.yaml
├── bin/
│   ├── install-base        # Uses cub space new-prefix
│   ├── install-envs        # Creates hierarchy
│   ├── apply-all           # Deploys via ConfigHub
│   ├── promote             # Push-upgrade pattern
│   └── proj               # Returns project name
├── main.go                # Uses SDK with informers
├── demo.go                # Demo mode for testing
└── README.md              # Documents ConfigHub deployment
```

## Deployment Commands

```bash
# Step 1: Setup (canonical pattern)
cd {app-name}
bin/install-base      # Creates unique prefix, spaces, filters
bin/install-envs      # Creates environment hierarchy

# Step 2: Deploy
bin/apply-all dev     # Deploy to dev
bin/promote dev staging  # Promote to staging
bin/apply-all staging    # Apply staging

# View structure
cub unit tree --node=space --filter $(bin/proj)/app --space '*'
```

## SDK Integration Pattern

```go
func main() {
    app := sdk.NewDevOpsApp(sdk.DevOpsAppConfig{
        Name: "my-devops-app",
        Version: "1.0.0",
    })

    // Get unique prefix (canonical)
    projectPrefix := app.Cub.RunCommand("cub space new-prefix")

    // Run with informers (event-driven)
    app.RunWithInformers(func() error {
        // Your logic here
        return nil
    })
}
```

## Apps Completed with Canonical Patterns

1. **drift-detector**: Full Sets/Filters, event-driven, auto-correction
2. **cost-optimizer**: AI-powered, dashboard (:8081), Sets for grouping

## Next Apps to Build

3. **security-scanner**: Continuous CVE monitoring with auto-patch
4. **compliance-auditor**: Continuous compliance with drift detection
5. **upgrade-manager**: Intelligent version management across environments
6. **branch-deployer**: Better than Cased's "killer branch deploy"

## Key Takeaways

1. **ALWAYS** use `cub space new-prefix` for unique naming
2. **ALWAYS** create environment hierarchy with upstream/downstream
3. **ALWAYS** use informers, not polling
4. **ALWAYS** deploy through ConfigHub, not kubectl
5. **ALWAYS** use Sets and Filters for bulk operations
6. **ALWAYS** use push-upgrade for propagation
7. **NEVER** use hallucinated features (variants, gates, etc.)

## References

- **Canonical Implementation**: `/Users/alexisrichardson/examples-internal/global-app/`
- **ConfigHub Source**: `/Users/alexisrichardson/github-repos/confighub/`
- **Our SDK**: `/Users/alexisrichardson/github-repos/devops-sdk/`
- **Example Apps**: `/Users/alexisrichardson/github-repos/devops-examples/`

This is the definitive guide. When in doubt, follow the global-app patterns exactly.