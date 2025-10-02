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
bin/setup-worker      # Install ConfigHub worker to execute applies
bin/apply-base        # Deploy via ConfigHub worker
bin/test-workflow     # Validate everything works

# For multi-environment (optional):
bin/install-envs      # Set up env hierarchy (dev → staging → prod)
bin/apply-all dev     # Deploy to dev
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
│   ├── setup-worker                # Installs ConfigHub worker
│   ├── apply-base                  # Sets targets and applies units
│   ├── test-workflow               # Validates deployment
│   ├── install-envs                # Creates env hierarchy (optional)
│   ├── apply-all [env]             # Applies to specific env (optional)
│   ├── promote [from] [to]         # Promotes between envs (optional)
│   ├── cleanup                     # Removes all resources
│   └── proj                        # Gets project name
├── main.go                         # App implementation
├── go.mod                          # Go module
├── README.md                       # Architecture and features
├── QUICKSTART.md                   # Step-by-step setup guide
└── WORKFLOW.md                     # ConfigHub → Kubernetes workflow
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

## ⚠️ CRITICAL: Worker Setup with --include-secret

**IMPORTANT**: When setting up ConfigHub workers, you MUST use the `--include-secret` flag to generate unique authentication credentials for each worker.

### The Issue

If you create multiple workers without `--include-secret`:
- Workers reuse existing `confighub-worker-env` secret
- New workers get WRONG credentials (from the first worker)
- Workers fail with: `[ERROR] Failed to get bridge worker slug: server returned status 404`
- Cannot connect or deploy units

### Correct Worker Setup Pattern

Every `bin/setup-worker` script MUST use this pattern:

```bash
#!/bin/bash
set -e

PROJECT=$(cat .cub-project)
SPACE="$PROJECT-base"
WORKER_NAME="devops-test-worker"

# Create confighub namespace
kubectl create namespace confighub --dry-run=client -o yaml | kubectl apply -f -

# Create worker in ConfigHub
if ! cub worker list --space $SPACE 2>/dev/null | grep -q $WORKER_NAME; then
    cub worker create $WORKER_NAME --space $SPACE
fi

# ⚠️ CRITICAL: Use --include-secret to generate unique credentials!
cub worker install $WORKER_NAME \
    --namespace confighub \
    --space $SPACE \
    --include-secret \
    --export > /tmp/worker-manifest.yaml

# Apply to cluster (includes deployment AND secret with correct credentials)
kubectl apply -f /tmp/worker-manifest.yaml

# Wait and verify
sleep 10
cub worker list --space $SPACE  # Should show: Condition=Ready
cub target list --space $SPACE  # Should show: k8s-<worker-name>
```

### What `--include-secret` Does

**Without `--include-secret`:**
```bash
cub worker install my-worker --export > worker.yaml
# Generates:
# - Deployment (references secret "confighub-worker-env")
# ❌ NO secret manifest
# ❌ Uses existing secret with WRONG worker credentials
```

**With `--include-secret`:**
```bash
cub worker install my-worker --include-secret --export > worker.yaml
# Generates:
# - Secret with CORRECT CONFIGHUB_WORKER_SECRET for THIS worker
# - Deployment (references the secret)
# ✅ Worker gets proper authentication
# ✅ Connects successfully
```

### Verification

After running `bin/setup-worker`, verify:

```bash
# 1. Worker shows as Ready
$ cub worker list --space your-space
NAME           CONDITION    SPACE        LAST-SEEN
your-worker    Ready        your-space   2025-10-01 18:31:55  ✅

# 2. Target was auto-created
$ cub target list --space your-space
NAME              WORKER        PROVIDERTYPE    PARAMETERS
k8s-your-worker   your-worker   Kubernetes      {"WaitTimeout":"2m0s"}  ✅

# 3. Worker pod is running
$ kubectl get pods -n confighub
NAME                         READY   STATUS    RESTARTS   AGE
your-worker-xxx-yyy          1/1     Running   0          2m  ✅

# 4. Worker logs show success
$ kubectl logs -n confighub -l app=your-worker --tail=5
[INFO] Successfully connected to event stream in 375ms, status: 200 200 OK  ✅
```

### Troubleshooting

**Worker shows "Disconnected" or 404 error?**
1. Delete the broken worker: `kubectl delete deployment <worker-name> -n confighub`
2. Recreate with `--include-secret`:
   ```bash
   cub worker install <worker-name> --space <space> --include-secret --export > worker.yaml
   kubectl apply -f worker.yaml
   ```
3. Verify: `cub worker list --space <space>` should show `Condition=Ready`

**For detailed troubleshooting**, see `/devops-examples/WORKER-SETUP.md`.

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

### Lateral Promotion (Conservative Rollout)
```bash
# Promote changes between environments without following hierarchy
# Useful for testing in one region before others

project=$(cat .cub-project)

# Step 1: Make change in us-staging
cub run set-env-var \
  --container-name backend \
  --env-var MODEL_VERSION \
  --env-value "v2.0" \
  --space $project-us-staging

# Apply and test
cub unit apply --space $project-us-staging

# Step 2: Get revision history
cub revision list backend --space $project-us-staging

# Step 3: Diff to see what changed
cub unit diff -u backend --space $project-us-staging --from=5

# Step 4: Promote specific revision range to eu-staging (bypassing hierarchy)
cub unit update backend \
  --space $project-eu-staging \
  --merge-unit $project-us-staging/backend \
  --merge-base=5 --merge-end=6

# Step 5: Verify the change
cub unit diff -u backend --space $project-eu-staging --from=4

# Step 6: Apply when confident
cub unit apply --space $project-eu-staging
```

**Why Lateral Promotion?**
- More conservative than hierarchical push-upgrade
- Test in one environment before rolling out everywhere
- Avoid accidentally promoting unrelated changes
- Better control over exactly which changes propagate

### Changesets for Atomic Multi-Unit Operations
```bash
# Use changesets when making multiple related changes that must be atomic
project=$(cat .cub-project)

# Step 1: Create changeset
cub changeset create --space $project memory-upgrade \
  --description "Increase memory for all backend services in staging"

# Step 2: Lock units to this changeset (prevents other changes)
cub unit update --patch \
  --space "*" \
  --changeset $project/memory-upgrade \
  --where "Slug = 'backend' AND Labels.role='staging'"

# Step 3: Make all changes under this changeset
cub run set-env-var \
  --space "*" \
  --changeset $project/memory-upgrade \
  --container-name backend \
  --env-var LARGE_MEMORY \
  --env-value "true" \
  --where "Slug = 'backend' AND Labels.role = 'staging'"

cub run set-container-resources \
  --space "*" \
  --changeset $project/memory-upgrade \
  --container-name backend \
  --memory 2Gi \
  --cpu 1 \
  --where "Slug = 'backend' AND Labels.role = 'staging'"

# Step 4: Unlock units (remove changeset lock)
cub unit update --patch --changeset - \
  --space "*" \
  --where "Slug = 'backend' AND Labels.role='staging'"

# Step 5: Apply all changes atomically by referencing the changeset
cub unit apply \
  --space "*" \
  --where "Slug = 'backend' AND Labels.role = 'staging'" \
  --revision "ChangeSet:$project/memory-upgrade"
```

**Why Changesets?**
- Prevent conflicts when multiple changes are in flight
- Ensure all related changes are applied together
- Rollback entire changeset if needed
- Clear audit trail for complex operations

### Revision Management and Rollback
```bash
project=$(cat .cub-project)

# List all revisions for a unit
cub revision list backend --space $project-qa

# Diff between specific revisions
cub unit diff -u backend --space $project-qa --from=5 --to=7

# Diff from revision to HEAD
cub unit diff -u backend --space $project-qa --from=5

# Rollback to previous revision
cub unit apply --space $project-qa --unit backend --revision=5

# View upgrade status across all environments
cub unit tree \
  --node=space \
  --filter $project/app \
  --space "*" \
  --columns Space.Slug,HeadRevisionNum,UpgradeNeeded,UnappliedChanges
```

**Rollback Script Example:**
```bash
#!/bin/bash
# bin/rollback
set -e

project=$(cat .cub-project)
env=$1
unit=$2
revision=$3

if [ -z "$env" ] || [ -z "$unit" ] || [ -z "$revision" ]; then
  echo "Usage: bin/rollback <env> <unit> <revision>"
  exit 1
fi

echo "🔄 Rolling back $unit in $env to revision $revision"

# Rollback
cub unit apply --space $project-$env --unit $unit --revision=$revision

# Verify
cub unit list --space $project-$env --unit $unit --columns Name,HeadRevisionNum,Status

echo "✅ Rollback complete!"
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