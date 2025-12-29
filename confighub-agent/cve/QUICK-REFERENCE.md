# CCVE Quick Reference Card

**ConfigHub Common Vulnerabilities and Errors** — Field Guide

---

## Critical Issues (Fix Immediately)

| Code | Pattern | First Check |
|------|---------|-------------|
| **0001** | GitRepository not ready | `flux get sources git -A` |
| **0002** | Kustomization build failed | `flux logs --kind=kustomize-controller` |
| **0003** | HelmRelease chart not ready | `flux get sources chart -A` |
| **0004** | Argo sync failed | `argocd app get <name>` |
| **0007** | Helm release failed | `helm list -A --failed` |
| **0012** | Image tag drift | `kubectl get deploy -o wide` |
| **0015** | ConfigHub worker offline | `cub worker status --space <space>` |
| **0021** | Grafana datasource failed | `kubectl logs -l app=grafana \| grep datasource` |
| **0022** | Grafana PVC unbound | `kubectl get pvc -l app=grafana` |
| **0023** | Grafana LDAP auth failure | `kubectl logs -l app=grafana \| grep LDAP` |

---

## Warning Issues (Fix Soon)

| Code | Pattern | First Check |
|------|---------|-------------|
| **0005** | Argo out of sync | `argocd app list --sync-status OutOfSync` |
| **0006** | Argo health degraded | `argocd app list --health-status Degraded` |
| **0008** | HelmRelease retries exhausted | `kubectl describe helmrelease <name>` |
| **0010** | CRD missing | `kubectl get crds` |
| **0011** | Manual kubectl edit | Check last-applied-configuration |
| **0013** | Helm pending upgrade | `helm list -A --pending` |
| **0014** | ConfigHub unit pending | `cub unit diff --space <space> <unit>` |
| **0016** | HelmRelease dependency | `kubectl describe helmrelease <name>` |
| **0017** | Argo resource too large | Enable ServerSideApply |
| **0018** | Webhook validation | Update webhook sideEffects |
| **0019** | Unmanaged resource | Search for ownership labels |
| **0020** | Replica count drift | Check for HPA |
| **0024** | Grafana dashboard provisioning | `kubectl get cm -l grafana_dashboard=1` |
| **0025** | Grafana datasource not found | Check dashboard datasource UIDs |
| **0026** | Grafana plugin install failed | `kubectl logs -l app=grafana \| grep plugin` |
| **0027** | Grafana sidecar namespace whitespace | Check NAMESPACE env var for spaces |

---

## Info Issues (Cleanup)

| Code | Pattern | First Check |
|------|---------|-------------|
| **0009** | Flux suspended | `flux get all -A` |

---

## Triage by Symptom

### "Nothing is updating"
1. Check **0001** - GitRepository ready?
2. Check **0003** - HelmChart ready?
3. Check **0009** - Is reconciliation suspended?
4. Check **0015** - Worker connected?

### "Deployment failed"
1. Check **0004** - Argo sync error?
2. Check **0007** - Helm release failed?
3. Check **0002** - Kustomization build error?
4. Check **0010** - Missing CRD?

### "State is drifted"
1. Check **0005** - Argo out of sync?
2. Check **0011** - Manual kubectl edit?
3. Check **0012** - Image tag changed?
4. Check **0020** - Replica count changed?
5. Check **0014** - ConfigHub pending?

### "Resources unhealthy"
1. Check **0006** - Argo health degraded?
2. Check **0008** - HelmRelease timeout?
3. Check **0013** - Helm pending upgrade?

### "Unknown resources found"
1. Check **0019** - Unmanaged resources?

### "Grafana not working"
1. Check **0021** - Datasource connection failed?
2. Check **0022** - PVC unbound (pod pending)?
3. Check **0023** - LDAP auth failure?
4. Check **0024** - Dashboard not appearing?
5. Check **0025** - Dashboard shows "datasource not found"?
6. Check **0026** - Plugin missing?
7. Check **0027** - Sidecar not watching namespaces (whitespace in NAMESPACE env)?

---

## Quick Diagnostics

### Flux
```bash
flux check                         # System health
flux get all -A                    # All resources
flux get sources all -A            # Check sources
flux logs --all-namespaces --level=error
```

### Argo CD
```bash
argocd app list                    # All applications
argocd app list --refresh          # With sync check
argocd app get <name>              # Detailed view
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller
```

### Helm
```bash
helm list -A                       # All releases
helm list -A --failed              # Failed only
helm list -A --pending             # Stuck releases
helm history <release> -n <ns>     # Release history
```

### ConfigHub
```bash
cub unit list --where "HeadRevisionNum != LiveRevisionNum"  # Drifted
cub worker status --space <space>  # Worker health
cub unit diff --space <space> <unit>  # Show drift
```

### Grafana
```bash
kubectl get pods -l app=grafana          # Pod status
kubectl logs -l app=grafana              # Recent logs
kubectl get pvc -l app=grafana           # Storage status
kubectl get cm -l grafana_dashboard=1    # Provisioned dashboards
kubectl get secret -l app=grafana        # Datasource configs
```

---

## Common Fix Patterns

### Fix authentication (0001, 0003)
```bash
# Recreate secret
flux create secret git deploy-key \
  --url=ssh://git@github.com/org/repo \
  --ssh-key-algorithm=ecdsa

# For Helm/OCI
flux create secret oci <name> \
  --url=<registry> \
  --username=<user> \
  --password=<token>
```

### Fix build errors (0002)
```bash
# Test locally
git clone <repo> && cd <repo>
kustomize build <path>

# Check for invalid patches or missing resources
```

### Fix drift (0005, 0011, 0012, 0020)
```bash
# Argo: Sync to desired state
argocd app sync <name>

# Argo: Accept live state
argocd app diff <name>  # Review changes
# Update Git to match live

# Flux: Reconcile
flux reconcile kustomization <name>
```

### Fix CRD issues (0010)
```bash
# Install CRD first, then CR
kubectl apply -f crd.yaml
kubectl apply -f cr.yaml

# Or enable skip validation
kubectl patch application <name> -n argocd --type merge -p '
metadata:
  annotations:
    argocd.argoproj.io/sync-options: SkipDryRunOnMissingResource=true
'
```

### Fix resource size (0017)
```bash
# Enable server-side apply
kubectl patch application <name> -n argocd --type merge -p '
metadata:
  annotations:
    argocd.argoproj.io/sync-options: ServerSideApply=true
'
```

### Fix Grafana datasource (0021)
```bash
# Check datasource config
kubectl get configmap grafana-datasources -o yaml

# Test datasource connectivity
kubectl exec -it deploy/grafana -- wget -O- http://prometheus-server:80/api/v1/query?query=up

# Fix URL in datasource config
kubectl edit configmap grafana-datasources
# Update url field, then restart Grafana
kubectl rollout restart deployment grafana
```

### Fix Grafana PVC (0022)
```bash
# Check PVC status
kubectl get pvc grafana -o yaml

# Check available StorageClasses
kubectl get sc

# Fix: Add storageClassName
kubectl patch pvc grafana -p '{"spec":{"storageClassName":"gp2"}}'

# Or create matching PV manually for local/static provisioning
```

### Fix Grafana dashboard provisioning (0024)
```bash
# Add required label
kubectl label configmap my-dashboard grafana_dashboard=1

# Verify namespace (Rancher requires cattle-dashboards)
kubectl get cm my-dashboard -n cattle-dashboards

# Fix securityContext for file permissions
kubectl patch deployment grafana --type merge -p '
spec:
  template:
    spec:
      securityContext:
        fsGroup: 472
'
```

### Fix Grafana sidecar namespace whitespace (0027)
```bash
# Check current NAMESPACE env var value
kubectl get deployment grafana -o jsonpath='{.spec.template.spec.containers[?(@.name=="grafana-sc-dashboard")].env[?(@.name=="NAMESPACE")].value}'

# WRONG: "monitoring, grafana, observability" (spaces after commas)
# RIGHT: "monitoring,grafana,observability" (no spaces)

# Fix by removing all spaces from comma-separated list
kubectl set env deployment/grafana NAMESPACE="monitoring,grafana,observability" -c grafana-sc-dashboard

# Verify fix
kubectl rollout status deployment/grafana
kubectl logs -l app=grafana -c grafana-sc-dashboard | grep "Watching namespace"
```

---

## Detection Quick Hits

### Flux Resource Status
```bash
kubectl get <resource> -o jsonpath='{.status.conditions[?(@.type=="Ready")]}'
```

### Argo Application Status
```bash
kubectl get application -n argocd <name> -o jsonpath='{.status.sync.status}'
kubectl get application -n argocd <name> -o jsonpath='{.status.health.status}'
```

### Helm Release Status
```bash
helm status <release> -n <namespace> -o json | jq '.info.status'
```

### Check for Drift
```bash
# Compare annotation
kubectl get deployment <name> -o json | \
  jq '.metadata.annotations["kubectl.kubernetes.io/last-applied-configuration"]' | \
  jq -r | diff - <(kubectl get deployment <name> -o json | jq '.spec')
```

### Grafana Status Checks
```bash
# Check datasource health via API
kubectl port-forward svc/grafana 3000:80 &
curl -u admin:password http://localhost:3000/api/datasources | jq '.[].url'

# Check PVC binding
kubectl get pvc -l app=grafana -o jsonpath='{.items[*].status.phase}'

# Check dashboard provisioning logs
kubectl logs -l app=grafana | grep provisioning

# Check sidecar NAMESPACE env var for whitespace (CCVE-0027)
kubectl get deployment grafana -o jsonpath='{.spec.template.spec.containers[?(@.name=="grafana-sc-dashboard")].env[?(@.name=="NAMESPACE")].value}' | grep -E ', '
# If match found, you have spaces after commas (WRONG)
```

---

## Severity Triage

**Critical (10):** 0001, 0002, 0003, 0004, 0007, 0012, 0015, 0021, 0022, 0023
**Warning (16):** 0005, 0006, 0008, 0010, 0011, 0013, 0014, 0016, 0017, 0018, 0019, 0020, 0024, 0025, 0026, 0027
**Info (1):** 0009

---

## Category Reference

- **SOURCE** (2): 0001, 0003
- **RENDER** (1): 0002
- **APPLY** (1): 0004
- **DRIFT** (5): 0005, 0011, 0012, 0014, 0020
- **DEPEND** (5): 0015, 0016, 0021, 0022, 0026
- **STATE** (3): 0006, 0007, 0013
- **ORPHAN** (1): 0019
- **CONFIG** (9): 0008, 0009, 0010, 0017, 0018, 0023, 0024, 0025, 0027

---

## Scanner Usage

```bash
# Quick scan
cd confighub-agent/test/atk
./scan

# JSON output
./scan --json > scan-results.json

# Filter by severity
./scan --severity critical

# Specific namespace
./scan --namespace production
```

---

**Full Documentation:** See [cve/README.md](README.md)
**Report Issues:** https://github.com/confighub/confighub-agent/issues
