# Canonical Patterns Summary: DevOps as Apps with ConfigHub

## Overview

This document summarizes the canonical patterns from the global-app reference implementation that ALL DevOps apps must follow.

## Top 10 Requirements

1. **ALWAYS** use `cub space new-prefix` for unique naming
2. **ALWAYS** use the ConfigHub unit model and cub CLI for managaing config
3. **ALWAYS** follow the global-app model for app structure, use of CLI, scripts
4. **ALWAYS** create environment hierarchy with upstream/downstream
5. **ALWAYS** use informers, not polling
6. **ALWAYS** deploy through ConfigHub, not kubectl
7. **ALWAYS** use Sets and Filters for bulk operations
8. **ALWAYS** use push-upgrade for propagation
9. **ALWAYS** use ConfigHub, cub and sdk features where possible. Extend sdk with common features.
10. **ALWAYS** check code to remove any hallucinated features of ConfigHub

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

### 8. ConfigHub Functions (cub run)
```bash
# Version management - Update container image tags
cub run set-image-reference \
  --container-name app \
  --image-reference :1.2.0 \
  --space $(bin/proj)-qa

# Environment variables - Set or update env vars
cub run set-env-var \
  --container-name backend \
  --env-var DATABASE_URL \
  --env-value "postgres://new-db:5432" \
  --space $(bin/proj)-qa

# Resource limits - Update CPU/memory
cub run set-container-resources \
  --container-name app \
  --memory 2Gi \
  --cpu 1 \
  --space $(bin/proj)-staging

# Hostname management - Update ingress subdomains
cub run set-hostname-subdomain \
  --subdomain new-app \
  --space $(bin/proj)-qa \
  --where "Slug IN ('backend', 'frontend')"

# Deep YAML editing - Modify any path in manifests
cub run set-string-path \
  --resource-type v1/Namespace \
  --path metadata.name \
  --attribute-value new-namespace \
  --unit ns-qa \
  --space $(bin/proj)-infra
```

### 9. Changesets for Atomic Operations
```bash
# Create changeset for grouping multiple changes
cub changeset create --space $(bin/proj) memory-upgrade \
  --description "Increase memory across all staging services"

# Lock units to changeset (prevents other changes)
cub unit update --patch \
  --space "*" \
  --changeset $(bin/proj)/memory-upgrade \
  --where "Slug = 'backend' AND Labels.role='staging'"

# Make all changes under the same changeset
cub run set-env-var \
  --space "*" \
  --changeset $(bin/proj)/memory-upgrade \
  --container-name backend \
  --env-var LARGE_MEMORY \
  --env-value "true" \
  --where "Slug = 'backend' AND Labels.role = 'staging'"

cub run set-container-resources \
  --space "*" \
  --changeset $(bin/proj)/memory-upgrade \
  --container-name backend \
  --memory 1Gi \
  --where "Slug = 'backend' AND Labels.role = 'staging'"

# Unlock units
cub unit update --patch --changeset - \
  --space "*" \
  --where "Slug = 'backend' AND Labels.role='staging'"

# Apply all changes atomically
cub unit apply \
  --space "*" \
  --where "Slug = 'backend' AND Labels.role = 'staging'" \
  --revision "ChangeSet:$(bin/proj)/memory-upgrade"
```

### 10. Lateral Promotion (Conservative Rollout)
```bash
# Promote directly between environments (bypassing hierarchy)
# Useful for testing changes in one region before others

# Get revision history to find the range
cub revision list ollama --space $(bin/proj)-us-staging

# Diff to see what changed
cub unit diff -u ollama --space $(bin/proj)-us-staging --from=5

# Promote specific revision range from us-staging to eu-staging
cub unit update ollama \
  --space $(bin/proj)-eu-staging \
  --merge-unit $(bin/proj)-us-staging/ollama \
  --merge-base=5 --merge-end=6

# Verify the change
cub unit diff -u ollama --space $(bin/proj)-eu-staging --from=4

# Apply when ready
cub unit apply --space $(bin/proj)-eu-staging
```

### 11. Revision Management
```bash
# List all revisions for a unit
cub revision list backend --space $(bin/proj)-qa

# Diff between revisions
cub unit diff -u backend --space $(bin/proj)-qa --from=5 --to=7

# Diff from specific revision to HEAD
cub unit diff -u backend --space $(bin/proj)-qa --from=5

# Rollback to previous revision
cub unit apply --space $(bin/proj)-qa --unit backend --revision=5

# View unit tree with upgrade status
cub unit tree \
  --node=space \
  --filter $(bin/proj)/app \
  --space "*" \
  --columns Space.Slug,UpgradeNeeded,UnappliedChanges
```

### 12. Link Management (Connect App to Infrastructure)
```bash
# Link app units to infrastructure (e.g., namespace)
cub link create \
  --dest-space $(bin/proj)-qa \
  --where-from "Slug LIKE '%'" \
  --where-to "Slug='ns-qa'" \
  --where-to-space "Slug = '$(bin/proj)-infra'"

# This connects all units in qa space to the ns-qa namespace unit
# Units will automatically use the linked infrastructure
```

## ConfigHub Features We Use 

Each example should make the maximum possible use of ConfigHub's specific features.  If the example works around ConfigHub by using eg kubectl, then we consider that a mistake that we should correct.

### ✅ Features:
- **Spaces**: Workspaces with unique prefixes
- **Units**: Configuration items and lifecycle eg. update, clone
- **Sets**: Groups for bulk operations
- **Filters**: WHERE clause queries
- **Upstream/Downstream**: Inheritance via UpstreamUnitID
- **Push-upgrade**: BulkPatchUnits(Upgrade: true)
- **Apply/Destroy**: Deploy and remove
- **Live State**: Read-only status
- **Changesets**: Atomic multi-unit operations
- **Links**: Connect app units to infrastructure
- **ConfigHub Functions**: cub run commands for common operations
- **Revisions**: Full version history with diff and rollback

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

## Recommended Patterns

1. **Persistent over Ephemeral**: Apps run continuously, not just when triggered
2. **Hierarchical over Flat**: Full environment inheritance (base → qa → staging → prod)
3. **Event-driven over Polling**: Kubernetes informers for immediate reaction
4. **Intelligent over Rule-based**: Claude AI for adaptive decisions
5. **Stateful over Stateless**: ConfigHub tracks all history

## Apps Completed with Canonical Patterns

1. **drift-detector**: Full Sets/Filters, event-driven, auto-correction
2. **cost-optimizer**: AI-powered, dashboard (:8081), Sets for grouping

## Next Apps to Build

3. **security-scanner**: Continuous CVE monitoring with auto-patch
4. **compliance-auditor**: Continuous compliance with drift detection
5. **upgrade-manager**: Intelligent version management across environments
6. **branch-deployer**: Better than Cased's "killer branch deploy"

## References

- **Canonical Implementation**: `/Users/alexisrichardson/examples-internal/global-app/`
- **ConfigHub Source**: `/Users/alexisrichardson/github-repos/confighub/`
- **Our SDK**: `/Users/alexisrichardson/github-repos/devops-sdk/`
- **Example Apps**: `/Users/alexisrichardson/github-repos/devops-examples/`

This is the definitive guide. When in doubt, follow the global-app patterns exactly.
