# ConfigHub Deployment Pattern for DevOps Apps

## Executive Summary

Following the global-app pattern, **ALL DevOps apps must deploy themselves through ConfigHub units**, not traditional kubectl/YAML files. This ensures consistency, enables environment management, and demonstrates that DevOps tools are first-class applications.

## The Pattern

### ❌ Old Way (Don't Do This)
```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/rbac.yaml
```

### ✅ New Way (ConfigHub-Driven)
```bash
bin/install-base      # Create units in ConfigHub
bin/install-envs      # Set up env hierarchy (dev → staging → prod)
bin/apply-all dev     # Deploy via ConfigHub
bin/promote dev staging  # Promote with push-upgrade
```

## Standard Directory Structure

Every DevOps app MUST follow this structure:

```
app-name/
├── confighub/
│   └── base/
│       ├── namespace.yaml           # K8s namespace
│       ├── app-deployment.yaml      # Deployment spec
│       ├── app-service.yaml         # Service definition
│       └── app-rbac.yaml           # RBAC configuration
├── bin/
│   ├── install-base                # Creates ConfigHub units
│   ├── install-envs                # Creates env hierarchy
│   ├── apply-all [env]             # Applies to K8s via ConfigHub
│   ├── promote [from] [to]         # Promotes between envs
│   ├── cleanup                     # Removes all resources
│   └── proj                        # Gets project name
├── main.go                         # App implementation
├── go.mod                          # Go module
└── README.md                       # Must include ConfigHub deployment
```

## ConfigHub Space Hierarchy

Each app creates this hierarchy in ConfigHub:

```
app-name-base                      # Base configurations
    └── app-name-dev               # Dev environment (cloned from base)
        └── app-name-staging       # Staging (cloned from dev)
            └── app-name-prod      # Production (cloned from staging)

app-name-filters                   # Filters for targeting
app-name                          # Main space for sets/metadata
```

## Implementation Guide

### Step 1: Create Base Kubernetes Manifests

Create standard Kubernetes manifests in `confighub/base/`:

**namespace.yaml**:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: devops-apps
  labels:
    managed-by: confighub
```

**app-deployment.yaml**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: your-app
  namespace: devops-apps
spec:
  replicas: 1
  selector:
    matchLabels:
      app: your-app
  template:
    spec:
      containers:
      - name: your-app
        image: ghcr.io/monadic/your-app:latest
```

### Step 2: Create Installation Scripts

**bin/install-base**:
```bash
#!/bin/bash
set -e

# Check for existing project
if [ -e ".cub-project" ]; then
  echo "Clean up previous usage with bin/cleanup or delete .cub-project"
  exit 1
fi

# Generate unique prefix using ConfigHub
if [ -z "$1" ]; then
  prefix=$(cub space new-prefix)  # Returns like "chubby-paws"
  project="${prefix}-your-app"    # Result: "chubby-paws-your-app"
else
  project=$1
fi

echo $project > .cub-project

# Create spaces
cub space create $project --label app=your-app
cub space create $project-base --label base=true
cub space create $project-filters --label type=filters

# Create filters
cub filter create all Unit --where-field "Space.Labels.project = '$project'" --space $project-filters
cub filter create critical Unit --where-field "Labels.tier='critical'" --space $project-filters

# Create units from base configs
cub unit create namespace confighub/base/namespace.yaml --space $project-base
cub unit create your-app-deployment confighub/base/app-deployment.yaml --space $project-base
cub unit create your-app-service confighub/base/app-service.yaml --space $project-base
cub unit create your-app-rbac confighub/base/app-rbac.yaml --space $project-base

# Create sets for grouping
cub set create your-app-set --space $project --label app=your-app

echo "✅ Base setup complete!"
```

**bin/install-envs**:
```bash
#!/bin/bash
set -e

project=$(cat .cub-project)

# Function to clone units with upstream
clone_units() {
  local from=$1
  local to=$2
  local env=$3

  for unit in namespace your-app-deployment your-app-service your-app-rbac; do
    cub unit create $unit \
      --space $to \
      --upstream-unit $unit \
      --upstream-space $from \
      --label environment=$env
  done
}

# Create environment hierarchy
cub space create $project-dev --upstream-space $project-base
clone_units $project-base $project-dev dev

cub space create $project-staging --upstream-space $project-dev
clone_units $project-dev $project-staging staging

cub space create $project-prod --upstream-space $project-staging
clone_units $project-staging $project-prod prod

echo "✅ Environment hierarchy created!"
```

**bin/apply-all**:
```bash
#!/bin/bash
set -e

project=$(cat .cub-project)
env=${1:-dev}
space=$project-$env

echo "🚀 Applying to $env environment..."

# Apply in correct order
cub unit apply namespace --space $space
cub unit apply your-app-rbac --space $space
cub unit apply your-app-service --space $space
cub unit apply your-app-deployment --space $space

# Or use bulk apply
cub bulk apply --space $space --where "Labels.app = 'your-app'"

echo "✅ Deployment complete!"
```

**bin/promote**:
```bash
#!/bin/bash
set -e

project=$(cat .cub-project)
from=$1
to=$2

echo "🔄 Promoting from $from to $to..."

# Use push-upgrade pattern
cub bulk patch \
  --space $project-$to \
  --where "UpstreamSpaceID = '$project-$from'" \
  --upgrade true

echo "✅ Promotion complete!"
```

### Step 3: Deploy Your App

```bash
# Initial setup (one time)
cd your-app/
bin/install-base
bin/install-envs

# Create secrets
kubectl create secret generic your-app-secrets \
  --from-literal=cub-token=$CUB_TOKEN \
  -n devops-apps

# Deploy to dev
bin/apply-all dev

# Test in dev...

# Promote to staging
bin/promote dev staging
bin/apply-all staging

# Test in staging...

# Promote to production
bin/promote staging prod
bin/apply-all prod
```

## Why This Pattern?

### 1. Consistency
All apps (DevOps and business) use the same deployment pattern.

### 2. Environment Management
- Clear hierarchy: base → dev → staging → prod
- Changes flow through push-upgrade
- Each environment can have local customizations

### 3. Version Control & Audit Trail
- ConfigHub provides full version history for all units
- Every change tracked with who/what/when/why
- Complete promotion history with push-upgrade
- ApplyGates for approval workflows
- No need for Git-based versioning (ConfigHub has it built-in)

### 4. Bulk Operations
- Deploy entire app stack with one command
- Apply filters for targeted updates
- Use sets for grouped operations

### 5. Dogfooding
DevOps apps use ConfigHub just like the business apps they manage.

## Verification Commands

```bash
# View space hierarchy
cub unit tree --node=space --filter your-app --space '*'

# List units in an environment
cub unit list --space your-app-dev

# Check deployment status
cub unit list --space your-app-dev --show-status

# View downstream relationships
cub unit tree --filter your-app --space '*' --show-downstream

# Check what needs promotion
cub unit list --space your-app-staging --filter "UpgradeNeeded = true"
```

## Common Patterns

### Multi-Region Production (Direct Editing!)
```bash
# Create regional variants by editing units directly in their spaces
for region in us eu asia; do
  # Step 1: Create regional space (if needed)
  cub space create $project-$region-prod --label region=$region

  # Step 2: Copy base units to regional space
  for unit in app-deployment app-service app-rbac; do
    cub unit create $unit --space $project-$region-prod \
      --upstream-unit $unit --upstream-space $project-prod
  done

  # Step 3: Edit units directly for region-specific variants
  if [ "$region" == "us" ]; then
    # US variant: higher resources
    echo '{"spec":{"replicas":5}}' | \
      cub unit update app-deployment --space $project-us-prod --patch --from-stdin \
      --change-desc "US variant: scale for higher traffic"
  elif [ "$region" == "eu" ]; then
    # EU variant: GDPR compliance
    echo '{"metadata":{"labels":{"compliance":"gdpr"}}}' | \
      cub unit update app-deployment --space $project-eu-prod --patch --from-stdin \
      --change-desc "EU variant: GDPR requirements"
  elif [ "$region" == "asia" ]; then
    # Asia variant: different storage class
    echo '{"spec":{"template":{"spec":{"nodeSelector":{"zone":"asia-east1"}}}}}' | \
      cub unit update app-deployment --space $project-asia-prod --patch --from-stdin \
      --change-desc "Asia variant: regional nodes"
  fi
done

# Each unit now has its own revision with regional customizations
# No complex cloning needed - just direct edits!
```

### Feature Branch Deployment
```bash
# Create feature branch environment
branch="feature-xyz"
cub space create $project-$branch --upstream-space $project-dev
clone_units $project-dev $project-$branch $branch
bin/apply-all $branch
```

### Emergency Hotfix
```bash
# Create hotfix from prod
cub space create $project-hotfix --upstream-space $project-prod
clone_units $project-prod $project-hotfix hotfix

# Apply fix
cub unit update your-app-deployment --space $project-hotfix --data "..."
bin/apply-all hotfix

# Test...

# Promote directly to prod
bin/promote hotfix prod
```

## Troubleshooting

### Issue: Space already exists
```bash
# Use existing space
cub space get $project-dev
```

### Issue: Unit already exists
```bash
# Update existing unit
cub unit update your-app-deployment --space $project-dev
```

### Issue: Promotion fails
```bash
# Check upstream relationship
cub unit get your-app-deployment --space $project-staging --json | jq '.UpstreamUnitID'

# Force refresh
cub unit refresh your-app-deployment --space $project-staging
```

## Examples

### Drift Detector
See `/Users/alexisrichardson/github-repos/devops-examples/drift-detector/` for a complete implementation.

### Global App Pattern
Reference: `/Users/alexisrichardson/examples-internal/global-app/` shows the original pattern we're following.

## Next Steps

1. **Implement this pattern** in all new DevOps apps
2. **Migrate existing apps** to use ConfigHub deployment
3. **Update SDK** to include deployment helpers
4. **Create templates** for common app types
5. **Build CI/CD pipelines** that work with ConfigHub

## Key Takeaways

- **ConfigHub manages everything** - not just app config but deployment itself
- **Follow the hierarchy** - base → dev → staging → prod
- **Use push-upgrade** - for promoting changes
- **Think in units** - everything is a ConfigHub unit
- **Leverage bulk operations** - for efficiency

This pattern ensures that DevOps tools are true first-class applications with proper lifecycle management, not just scripts or workflows.