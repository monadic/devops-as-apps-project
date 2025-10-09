# TraderX ConfigHub Fix Plan

## Executive Summary

After thorough analysis, I've identified that TraderX's ConfigHub integration is fundamentally broken due to misunderstanding of the ConfigHub deployment pattern. The core issue: **we're missing the Worker → Target → Unit association chain** that enables ConfigHub to deploy to Kubernetes.

**Current State**: ConfigHub infrastructure exists (60 units) but cannot deploy
**Root Cause**: No worker installed, no targets created, units not associated with targets
**Fix Complexity**: Medium (2-4 hours of work)
**Success Probability**: High (95%) - pattern is well-documented

---

## 🔍 Root Cause Analysis

### The Fundamental Misunderstanding

We assumed: `cub unit apply --target k8s-dev` would work
Reality: ConfigHub requires this chain:
1. **Worker** installed in Kubernetes (bridge between ConfigHub and K8s)
2. **Target** created (represents the Kubernetes cluster)
3. **Units** associated with target via `set-target`
4. **Apply** uses the pre-associated target

### Critical Missing Components

1. **No Worker Installed**
   - Worker is the execution engine that applies units to Kubernetes
   - Without worker, ConfigHub can't communicate with cluster
   - Our setup-worker script creates ConfigMaps instead of using `cub worker install`

2. **No Target Created**
   - Target represents where units should be deployed
   - Scripts assume "k8s-dev" target exists but never created
   - Without target, units have nowhere to deploy

3. **No Unit-Target Association**
   - Units must be associated with targets before apply
   - We never run `cub unit set-target`
   - This is why we get "missing TargetID on Unit" errors

4. **Wrong Apply Syntax**
   - Scripts use non-existent `--target` flag
   - Should just be `cub unit apply unit-name` after association

---

## 🛠️ Comprehensive Fix Plan

### Phase 1: Fix Worker Installation (Critical)

#### 1.1 Create Proper Worker Installation Script
**File**: `/Users/alexis/traderx/bin/setup-worker-fixed`

```bash
#!/bin/bash
set -euo pipefail

PROJECT=$(cat .cub-project || echo "mellow-muzzle-traderx")
ENV=${1:-dev}
SPACE="${PROJECT}-${ENV}"
WORKER_NAME="${PROJECT}-worker-${ENV}"

echo "Installing ConfigHub worker for $SPACE..."

# Create namespace
kubectl create namespace confighub --dry-run=client -o yaml | kubectl apply -f -

# Install worker with proper authentication
# This creates both the worker AND a target automatically
cub worker install "$WORKER_NAME" \
  --space "$SPACE" \
  --namespace confighub \
  --include-secret \
  --wait \
  --deployment-name "${PROJECT}-worker-${ENV}"

echo "Worker installed successfully"

# Get the target that was created
TARGET_ID=$(cub target list --space "$SPACE" | grep "$WORKER_NAME" | awk '{print $1}')
echo "Target created: $TARGET_ID"

# Save target for other scripts
echo "$TARGET_ID" > ".cub-target-${ENV}"
```

#### 1.2 Associate Units with Target
**File**: `/Users/alexis/traderx/bin/associate-targets`

```bash
#!/bin/bash
set -euo pipefail

PROJECT=$(cat .cub-project)
ENV=${1:-dev}
SPACE="${PROJECT}-${ENV}"
TARGET=$(cat ".cub-target-${ENV}")

echo "Associating units with target $TARGET..."

# Associate all units in the space with the target
cub unit set-target "$TARGET" \
  --space "$SPACE" \
  --where "Slug LIKE '%'"

echo "All units associated with target"

# Verify association
cub unit list --space "$SPACE" | grep -c "$TARGET" || true
```

---

### Phase 2: Fix Deployment Scripts

#### 2.1 Fix ordered-apply Script
**Changes needed in** `/Users/alexis/traderx/bin/ordered-apply`

```bash
# REMOVE this line (line 156):
if ! retry_with_backoff $MAX_RETRIES "cub unit apply namespace --space $SPACE --target k8s-${ENV}"; then

# REPLACE with:
if ! retry_with_backoff $MAX_RETRIES "cub unit apply namespace --space $SPACE"; then

# Similarly fix all other apply commands (lines 199, 207, 237)
# Remove --target flag from all cub unit apply commands
```

#### 2.2 Fix rollback Script
**Changes in** `/Users/alexis/traderx/bin/rollback`

```bash
# REMOVE --target flag from apply commands
# Change from:
cub unit apply "$unit" --revision "$REVISION" --space "$SPACE" --target "k8s-${ENV}"

# To:
cub unit apply "$unit" --revision "$REVISION" --space "$SPACE"
```

---

### Phase 3: Fix Template Variables

#### 3.1 Create Template Processor
**File**: `/Users/alexis/traderx/bin/process-templates`

```bash
#!/bin/bash
set -euo pipefail

ENV=${1:-dev}
PROJECT=$(cat .cub-project)

echo "Processing templates for $ENV..."

# Process each YAML file
for file in confighub/base/*.yaml; do
  if grep -q "{{" "$file"; then
    echo "Processing $(basename $file)..."

    # Replace template variables
    sed -i.bak \
      -e "s/{{ .Namespace | default \"traderx-dev\" }}/traderx-${ENV}/g" \
      -e "s/{{ .Version | default \"latest\" }}/latest/g" \
      -e "s/{{ .ImageTag | default \"latest\" }}/latest/g" \
      -e "s/{{ .Replicas | default 1 }}/1/g" \
      -e "s/{{ .Environment | default \"dev\" }}/${ENV}/g" \
      -e "s/{{ .ResourceRequestMemory | default \"[^\"]*\" }}/256Mi/g" \
      -e "s/{{ .ResourceRequestCPU | default \"[^\"]*\" }}/250m/g" \
      -e "s/{{ .ResourceLimitMemory | default \"[^\"]*\" }}/512Mi/g" \
      -e "s/{{ .ResourceLimitCPU | default \"[^\"]*\" }}/500m/g" \
      "$file"
  fi
done

echo "Templates processed"
```

#### 3.2 Update Units in ConfigHub
**File**: `/Users/alexis/traderx/bin/update-units`

```bash
#!/bin/bash
set -euo pipefail

PROJECT=$(cat .cub-project)
SPACE="${PROJECT}-base"

# Process templates first
./bin/process-templates dev

# Update units with processed YAML
for file in confighub/base/*-deployment.yaml; do
  unit=$(basename "$file" .yaml)
  echo "Updating $unit..."
  cub unit update "$unit" --space "$SPACE" --data-file "$file"
done

echo "Units updated with processed templates"
```

---

### Phase 4: Fix Security Issues

#### 4.1 Create RBAC Manifests
**File**: `/Users/alexis/traderx/confighub/base/rbac.yaml`

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: traderx-service-account
  namespace: traderx-dev
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: traderx-role
  namespace: traderx-dev
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: traderx-rolebinding
  namespace: traderx-dev
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: traderx-role
subjects:
- kind: ServiceAccount
  name: traderx-service-account
  namespace: traderx-dev
```

#### 4.2 Create NetworkPolicies
**File**: `/Users/alexis/traderx/confighub/base/network-policy.yaml`

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: traderx-network-policy
  namespace: traderx-dev
spec:
  podSelector:
    matchLabels:
      app: traderx
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: traderx-dev
    ports:
    - protocol: TCP
      port: 18080
  egress:
  - to:
    - namespaceSelector: {}
    ports:
    - protocol: TCP
      port: 443
    - protocol: TCP
      port: 5432
```

---

### Phase 5: Remove kubectl Dependencies

#### 5.1 Create ConfigHub-Only Health Check
**File**: `/Users/alexis/traderx/bin/health-check-fixed`

```bash
#!/bin/bash
set -euo pipefail

PROJECT=$(cat .cub-project)
ENV=${1:-dev}
SPACE="${PROJECT}-${ENV}"

echo "Checking deployment health via ConfigHub..."

# Check unit status
echo "=== Unit Status ==="
cub unit list --space "$SPACE" | grep -E "deployment|service"

# Check live state
echo "=== Live State ==="
for unit in $(cub unit list --space "$SPACE" | grep deployment | awk '{print $1}'); do
  echo "Checking $unit..."
  cub unit livestate "$unit" --space "$SPACE" || echo "Not deployed"
done

# No kubectl commands!
```

---

## 📋 Implementation Sequence

### Critical Path (Must Do First)
1. **Install Worker** (30 min)
   - Run `bin/setup-worker-fixed dev`
   - Verify worker is running in confighub namespace

2. **Associate Units with Target** (15 min)
   - Run `bin/associate-targets dev`
   - Verify units show target association

3. **Fix Apply Commands** (30 min)
   - Remove --target flags from all scripts
   - Test with single unit first

### Secondary Fixes (Can Do Later)
4. **Process Templates** (30 min)
   - Run template processor
   - Update units in ConfigHub

5. **Add Security** (1 hour)
   - Create RBAC unit
   - Create NetworkPolicy unit
   - Apply to cluster

6. **Remove kubectl** (1 hour)
   - Replace health checks
   - Update blue-green script

---

## ✅ Validation Plan

### Test Sequence
```bash
# 1. Install worker
./bin/setup-worker-fixed dev

# 2. Associate targets
./bin/associate-targets dev

# 3. Test single unit apply
cub unit apply namespace --space mellow-muzzle-traderx-dev

# 4. Check deployment
kubectl get namespace traderx-dev

# 5. If successful, apply all
./bin/ordered-apply dev

# 6. Validate
kubectl get all -n traderx-dev
```

### Success Criteria
- [ ] Worker running in confighub namespace
- [ ] Target created and associated
- [ ] Namespace deploys successfully
- [ ] All 8 services deploy
- [ ] Health checks pass
- [ ] Can access web-gui on port 18080

---

## 🚀 Quick Fix Script

**File**: `/Users/alexis/traderx/bin/quick-fix`

```bash
#!/bin/bash
set -euo pipefail

echo "=== TraderX Quick Fix ==="
echo "This will fix the ConfigHub deployment issues"
echo ""

PROJECT="mellow-muzzle-traderx"
ENV="dev"
SPACE="${PROJECT}-${ENV}"

# Step 1: Install Worker
echo "Step 1: Installing ConfigHub worker..."
kubectl create namespace confighub --dry-run=client -o yaml | kubectl apply -f -

WORKER_NAME="${PROJECT}-worker-${ENV}"
cub worker install "$WORKER_NAME" \
  --space "$SPACE" \
  --namespace confighub \
  --include-secret \
  --wait

# Step 2: Get Target
echo "Step 2: Finding target..."
TARGET=$(cub target list --space "$SPACE" | grep -v NAME | head -1 | awk '{print $1}')
echo "Target: $TARGET"

# Step 3: Associate Units
echo "Step 3: Associating units with target..."
cub unit set-target "$TARGET" \
  --space "$SPACE" \
  --where "Slug LIKE '%'"

# Step 4: Apply namespace
echo "Step 4: Testing with namespace..."
cub unit apply namespace --space "$SPACE"

# Step 5: Check
kubectl get namespace traderx-dev

echo "✅ Fix applied! Now run: ./bin/ordered-apply dev"
```

---

## 💡 Key Insights

1. **ConfigHub Workers are Essential**
   - They're the bridge between ConfigHub and Kubernetes
   - Without workers, nothing can deploy

2. **Target Association is Required**
   - Units must know where to deploy
   - This happens through set-target, not apply flags

3. **The Pattern is Worker → Target → Units → Apply**
   - Worker creates target
   - Units associate with target
   - Apply uses pre-associated target

4. **Templates Need Pre-Processing**
   - ConfigHub doesn't process Go templates
   - Need to replace before creating units

---

## 📊 Risk Assessment

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|------------|
| Worker fails to install | Low | High | Check namespace, permissions |
| Target association fails | Low | High | Verify target exists first |
| Templates break YAML | Medium | Medium | Backup files, test individually |
| Services fail to start | Medium | Low | Fix one at a time |
| Network issues | Low | Medium | Check cluster connectivity |

---

## 🎯 Expected Outcome

After implementing these fixes:
- ✅ Worker installed and running
- ✅ All 60 units associated with target
- ✅ Deployment works via ConfigHub
- ✅ No --target flag errors
- ✅ Services accessible on expected ports
- ✅ Rollback and promotion functional
- ✅ 100% ConfigHub-driven deployment

**Estimated Time**: 2-4 hours for complete fix
**Success Probability**: 95% (pattern is proven in global-app)