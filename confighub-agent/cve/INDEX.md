# CCVE Database Index

Quick navigation for all 50 CCVEs. **[Full Catalog](README.md)** | **[Quick Reference](QUICK-REFERENCE.md)**

---

## By ID

| ID | Name | Tool | Category | Severity | Fixture |
|----|------|------|----------|----------|---------|
| [0001](CCVE-2025-0001.yaml) | GitRepository not ready | Flux | SOURCE | Critical | [✓](fixtures/CCVE-2025-0001-gitrepo-not-ready.yaml) |
| [0002](CCVE-2025-0002.yaml) | Kustomization build failed | Flux | RENDER | Critical | [✓](fixtures/CCVE-2025-0002-kustomization-build-failed.yaml) |
| [0003](CCVE-2025-0003.yaml) | HelmRelease chart not ready | Flux | SOURCE | Critical | |
| [0004](CCVE-2025-0004.yaml) | Application sync failed | Argo | APPLY | Critical | [✓](fixtures/CCVE-2025-0004-argo-sync-failed.yaml) |
| [0005](CCVE-2025-0005.yaml) | Application out of sync | Argo | DRIFT | Warning | [✓](fixtures/CCVE-2025-0005-argo-out-of-sync.yaml) |
| [0006](CCVE-2025-0006.yaml) | Application health degraded | Argo | STATE | Warning | |
| [0007](CCVE-2025-0007.yaml) | Helm release failed | Helm | STATE | Critical | |
| [0008](CCVE-2025-0008.yaml) | HelmRelease install retries exhausted | Flux | CONFIG | Warning | |
| [0009](CCVE-2025-0009.yaml) | Flux reconciliation suspended | Flux | CONFIG | Info | |
| [0010](CCVE-2025-0010.yaml) | Resource validation skipped - CRD missing | Argo | CONFIG | Warning | |
| [0011](CCVE-2025-0011.yaml) | Manual kubectl edit detected | Any | DRIFT | Warning | |
| [0012](CCVE-2025-0012.yaml) | Image tag drift | Any | DRIFT | Critical | [✓](fixtures/CCVE-2025-0012-image-tag-drift.yaml) |
| [0013](CCVE-2025-0013.yaml) | Helm release pending upgrade | Helm | STATE | Warning | |
| [0014](CCVE-2025-0014.yaml) | ConfigHub unit pending changes | ConfigHub | DRIFT | Warning | |
| [0015](CCVE-2025-0015.yaml) | ConfigHub worker disconnected | ConfigHub | DEPEND | Critical | |
| [0016](CCVE-2025-0016.yaml) | HelmRelease dependency not ready | Flux | DEPEND | Warning | |
| [0017](CCVE-2025-0017.yaml) | Argo Application resource too large | Argo | CONFIG | Warning | |
| [0018](CCVE-2025-0018.yaml) | Flux webhook validation failure | Flux | CONFIG | Warning | |
| [0019](CCVE-2025-0019.yaml) | Unmanaged resource in managed namespace | Any | ORPHAN | Warning | |
| [0020](CCVE-2025-0020.yaml) | Replica count drift | Any | DRIFT | Warning | |
| [0021](CCVE-2025-0021.yaml) | Grafana datasource connection failed | Grafana | DEPEND | Critical | [✓](fixtures/CCVE-2025-0021-grafana-datasource-failed.yaml) |
| [0022](CCVE-2025-0022.yaml) | Grafana persistent volume unbound | Grafana | DEPEND | Critical | [✓](fixtures/CCVE-2025-0022-grafana-pvc-unbound.yaml) |
| [0023](CCVE-2025-0023.yaml) | Grafana LDAP authentication failure | Grafana | CONFIG | Critical | |
| [0024](CCVE-2025-0024.yaml) | Grafana dashboard provisioning failed | Grafana | CONFIG | Warning | [✓](fixtures/CCVE-2025-0024-grafana-dashboard-provisioning-failed.yaml) |
| [0025](CCVE-2025-0025.yaml) | Grafana provisioned datasource not found | Grafana | CONFIG | Warning | |
| [0026](CCVE-2025-0026.yaml) | Grafana plugin installation failed | Grafana | DEPEND | Warning | |
| [0027](CCVE-2025-0027.yaml) | Grafana sidecar namespace whitespace error | Grafana | CONFIG | Warning | [✓](fixtures/CCVE-2025-0027-grafana-namespace-whitespace.yaml) |
| [0028](CCVE-2025-0028.yaml) | IngressRoute service not found | Traefik | DEPEND | Critical | [✓](fixtures/CCVE-2025-0028-traefik-service-not-found.yaml) |
| [0029](CCVE-2025-0029.yaml) | TLS passthrough using wrong resource type | Traefik | CONFIG | Critical | [✓](fixtures/CCVE-2025-0029-traefik-tls-passthrough-wrong-type.yaml) |
| [0030](CCVE-2025-0030.yaml) | Middleware not found | Traefik | DEPEND | Warning | |
| [0031](CCVE-2025-0031.yaml) | IngressRoute entryPoint not defined | Traefik | CONFIG | Critical | [✓](fixtures/CCVE-2025-0031-traefik-entrypoint-not-defined.yaml) |
| [0032](CCVE-2025-0032.yaml) | TLS certificate secret not found | Traefik | DEPEND | Critical | |
| [0033](CCVE-2025-0033.yaml) | Cross-namespace service reference blocked | Traefik | CONFIG | Warning | |
| [0034](CCVE-2025-0034.yaml) | Certificate Issuer not found | cert-manager | DEPEND | Critical | |
| [0035](CCVE-2025-0035.yaml) | ClusterIssuer not ready | cert-manager | CONFIG | Critical | |
| [0036](CCVE-2025-0036.yaml) | ACME DNS01 challenge failed | cert-manager | DEPEND | Critical | |
| [0037](CCVE-2025-0037.yaml) | ACME HTTP01 challenge 404 | cert-manager | CONFIG | Critical | |
| [0038](CCVE-2025-0038.yaml) | Invalid ACME account email | cert-manager | CONFIG | Warning | |
| [0039](CCVE-2025-0039.yaml) | Webhook not reachable from API server | cert-manager | DEPEND | Critical | |
| [0040](CCVE-2025-0040.yaml) | Certificate renewal rate limited | cert-manager | CONFIG | Warning | |
| [0041](CCVE-2025-0041.yaml) | ServiceMonitor not discovered by Prometheus | Prometheus | CONFIG | Critical | |
| [0042](CCVE-2025-0042.yaml) | Thanos object storage configuration invalid | Thanos | CONFIG | Critical | |
| [0043](CCVE-2025-0043.yaml) | Thanos sidecar not uploading blocks | Thanos | CONFIG | Critical | |
| [0044](CCVE-2025-0044.yaml) | Thanos Query StoreAPI endpoint unreachable | Thanos | DEPEND | Critical | |
| [0045](CCVE-2025-0045.yaml) | Alertmanager configuration invalid | Alertmanager | CONFIG | Critical | |
| [0046](CCVE-2025-0046.yaml) | Alert route not matching any alerts | Alertmanager | CONFIG | Warning | |
| [0047](CCVE-2025-0047.yaml) | Inhibit rules using deprecated syntax | Alertmanager | CONFIG | Warning | |
| [0048](CCVE-2025-0048.yaml) | Secrets Store CSI driver not registered on node | CSI | DEPEND | Critical | |
| [0049](CCVE-2025-0049.yaml) | SecretProviderClass namespace mismatch | CSI | CONFIG | Critical | |
| [0050](CCVE-2025-0050.yaml) | Secrets Store CSI mount failed - provider error | CSI | CONFIG | Critical | |

---

## By Severity

### Critical (29)
[0001](CCVE-2025-0001.yaml) • [0002](CCVE-2025-0002.yaml) • [0003](CCVE-2025-0003.yaml) • [0004](CCVE-2025-0004.yaml) • [0007](CCVE-2025-0007.yaml) • [0012](CCVE-2025-0012.yaml) • [0015](CCVE-2025-0015.yaml) • [0021](CCVE-2025-0021.yaml) • [0022](CCVE-2025-0022.yaml) • [0023](CCVE-2025-0023.yaml) • [0028](CCVE-2025-0028.yaml) • [0029](CCVE-2025-0029.yaml) • [0031](CCVE-2025-0031.yaml) • [0032](CCVE-2025-0032.yaml) • [0034](CCVE-2025-0034.yaml) • [0035](CCVE-2025-0035.yaml) • [0036](CCVE-2025-0036.yaml) • [0037](CCVE-2025-0037.yaml) • [0039](CCVE-2025-0039.yaml) • [0041](CCVE-2025-0041.yaml) • [0042](CCVE-2025-0042.yaml) • [0043](CCVE-2025-0043.yaml) • [0044](CCVE-2025-0044.yaml) • [0045](CCVE-2025-0045.yaml) • [0048](CCVE-2025-0048.yaml) • [0049](CCVE-2025-0049.yaml) • [0050](CCVE-2025-0050.yaml)

### Warning (20)
[0005](CCVE-2025-0005.yaml) • [0006](CCVE-2025-0006.yaml) • [0008](CCVE-2025-0008.yaml) • [0010](CCVE-2025-0010.yaml) • [0011](CCVE-2025-0011.yaml) • [0013](CCVE-2025-0013.yaml) • [0014](CCVE-2025-0014.yaml) • [0016](CCVE-2025-0016.yaml) • [0017](CCVE-2025-0017.yaml) • [0018](CCVE-2025-0018.yaml) • [0019](CCVE-2025-0019.yaml) • [0020](CCVE-2025-0020.yaml) • [0024](CCVE-2025-0024.yaml) • [0025](CCVE-2025-0025.yaml) • [0026](CCVE-2025-0026.yaml) • [0027](CCVE-2025-0027.yaml) • [0030](CCVE-2025-0030.yaml) • [0033](CCVE-2025-0033.yaml) • [0038](CCVE-2025-0038.yaml) • [0040](CCVE-2025-0040.yaml) • [0046](CCVE-2025-0046.yaml) • [0047](CCVE-2025-0047.yaml)

### Info (1)
[0009](CCVE-2025-0009.yaml)

---

## By Tool

### Flux (7)
- [0001](CCVE-2025-0001.yaml) GitRepository not ready ⚠️
- [0002](CCVE-2025-0002.yaml) Kustomization build failed ⚠️
- [0003](CCVE-2025-0003.yaml) HelmRelease chart not ready ⚠️
- [0008](CCVE-2025-0008.yaml) Install retries exhausted
- [0009](CCVE-2025-0009.yaml) Reconciliation suspended ℹ️
- [0016](CCVE-2025-0016.yaml) Dependency not ready
- [0018](CCVE-2025-0018.yaml) Webhook validation failure

### Argo CD (5)
- [0004](CCVE-2025-0004.yaml) Sync failed ⚠️
- [0005](CCVE-2025-0005.yaml) Out of sync
- [0006](CCVE-2025-0006.yaml) Health degraded
- [0010](CCVE-2025-0010.yaml) CRD missing
- [0017](CCVE-2025-0017.yaml) Resource too large

### Helm (2)
- [0007](CCVE-2025-0007.yaml) Release failed ⚠️
- [0013](CCVE-2025-0013.yaml) Pending upgrade

### ConfigHub (2)
- [0014](CCVE-2025-0014.yaml) Unit pending changes
- [0015](CCVE-2025-0015.yaml) Worker disconnected ⚠️

### Grafana (7)
- [0021](CCVE-2025-0021.yaml) Datasource connection failed ⚠️
- [0022](CCVE-2025-0022.yaml) Persistent volume unbound ⚠️
- [0023](CCVE-2025-0023.yaml) LDAP authentication failure ⚠️
- [0024](CCVE-2025-0024.yaml) Dashboard provisioning failed
- [0025](CCVE-2025-0025.yaml) Provisioned datasource not found
- [0026](CCVE-2025-0026.yaml) Plugin installation failed
- [0027](CCVE-2025-0027.yaml) Sidecar namespace whitespace error

### Traefik (6)
- [0028](CCVE-2025-0028.yaml) IngressRoute service not found ⚠️
- [0029](CCVE-2025-0029.yaml) TLS passthrough wrong resource type ⚠️
- [0030](CCVE-2025-0030.yaml) Middleware not found
- [0031](CCVE-2025-0031.yaml) IngressRoute entryPoint not defined ⚠️
- [0032](CCVE-2025-0032.yaml) TLS certificate secret not found ⚠️
- [0033](CCVE-2025-0033.yaml) Cross-namespace service reference blocked

### cert-manager (7)
- [0034](CCVE-2025-0034.yaml) Certificate Issuer not found ⚠️
- [0035](CCVE-2025-0035.yaml) ClusterIssuer not ready ⚠️
- [0036](CCVE-2025-0036.yaml) ACME DNS01 challenge failed ⚠️
- [0037](CCVE-2025-0037.yaml) ACME HTTP01 challenge 404 ⚠️
- [0038](CCVE-2025-0038.yaml) Invalid ACME account email
- [0039](CCVE-2025-0039.yaml) Webhook not reachable from API server ⚠️
- [0040](CCVE-2025-0040.yaml) Certificate renewal rate limited

### Prometheus (1)
- [0041](CCVE-2025-0041.yaml) ServiceMonitor not discovered ⚠️

### Thanos (3)
- [0042](CCVE-2025-0042.yaml) Object storage configuration invalid ⚠️
- [0043](CCVE-2025-0043.yaml) Sidecar not uploading blocks ⚠️
- [0044](CCVE-2025-0044.yaml) Query StoreAPI endpoint unreachable ⚠️

### Alertmanager (3)
- [0045](CCVE-2025-0045.yaml) Configuration invalid ⚠️
- [0046](CCVE-2025-0046.yaml) Alert route not matching
- [0047](CCVE-2025-0047.yaml) Inhibit rules deprecated syntax

### Secrets Store CSI Driver (3)
- [0048](CCVE-2025-0048.yaml) CSI driver not registered on node ⚠️
- [0049](CCVE-2025-0049.yaml) SecretProviderClass namespace mismatch ⚠️
- [0050](CCVE-2025-0050.yaml) Mount failed - provider error ⚠️

### Cross-Tool (4)
- [0011](CCVE-2025-0011.yaml) Manual kubectl edit
- [0012](CCVE-2025-0012.yaml) Image tag drift ⚠️
- [0019](CCVE-2025-0019.yaml) Unmanaged resource
- [0020](CCVE-2025-0020.yaml) Replica count drift

---

## By Category

### SOURCE (2)
[0001](CCVE-2025-0001.yaml) • [0003](CCVE-2025-0003.yaml)

### RENDER (1)
[0002](CCVE-2025-0002.yaml)

### APPLY (1)
[0004](CCVE-2025-0004.yaml)

### DRIFT (5)
[0005](CCVE-2025-0005.yaml) • [0011](CCVE-2025-0011.yaml) • [0012](CCVE-2025-0012.yaml) • [0014](CCVE-2025-0014.yaml) • [0020](CCVE-2025-0020.yaml)

### DEPEND (13)
[0015](CCVE-2025-0015.yaml) • [0016](CCVE-2025-0016.yaml) • [0021](CCVE-2025-0021.yaml) • [0022](CCVE-2025-0022.yaml) • [0026](CCVE-2025-0026.yaml) • [0028](CCVE-2025-0028.yaml) • [0030](CCVE-2025-0030.yaml) • [0032](CCVE-2025-0032.yaml) • [0034](CCVE-2025-0034.yaml) • [0036](CCVE-2025-0036.yaml) • [0039](CCVE-2025-0039.yaml) • [0044](CCVE-2025-0044.yaml) • [0048](CCVE-2025-0048.yaml)

### STATE (3)
[0006](CCVE-2025-0006.yaml) • [0007](CCVE-2025-0007.yaml) • [0013](CCVE-2025-0013.yaml)

### ORPHAN (1)
[0019](CCVE-2025-0019.yaml)

### CONFIG (24)
[0008](CCVE-2025-0008.yaml) • [0009](CCVE-2025-0009.yaml) • [0010](CCVE-2025-0010.yaml) • [0017](CCVE-2025-0017.yaml) • [0018](CCVE-2025-0018.yaml) • [0023](CCVE-2025-0023.yaml) • [0024](CCVE-2025-0024.yaml) • [0025](CCVE-2025-0025.yaml) • [0027](CCVE-2025-0027.yaml) • [0029](CCVE-2025-0029.yaml) • [0031](CCVE-2025-0031.yaml) • [0033](CCVE-2025-0033.yaml) • [0035](CCVE-2025-0035.yaml) • [0037](CCVE-2025-0037.yaml) • [0038](CCVE-2025-0038.yaml) • [0040](CCVE-2025-0040.yaml) • [0041](CCVE-2025-0041.yaml) • [0042](CCVE-2025-0042.yaml) • [0043](CCVE-2025-0043.yaml) • [0045](CCVE-2025-0045.yaml) • [0046](CCVE-2025-0046.yaml) • [0047](CCVE-2025-0047.yaml) • [0049](CCVE-2025-0049.yaml) • [0050](CCVE-2025-0050.yaml)

---

## Test Fixtures

12 fixtures available for critical patterns:

- [CCVE-0001](fixtures/CCVE-2025-0001-gitrepo-not-ready.yaml) — GitRepository auth failure
- [CCVE-0002](fixtures/CCVE-2025-0002-kustomization-build-failed.yaml) — Kustomization build error
- [CCVE-0004](fixtures/CCVE-2025-0004-argo-sync-failed.yaml) — Argo sync failure
- [CCVE-0005](fixtures/CCVE-2025-0005-argo-out-of-sync.yaml) — Argo drift/out-of-sync
- [CCVE-0012](fixtures/CCVE-2025-0012-image-tag-drift.yaml) — Image tag drift
- [CCVE-0021](fixtures/CCVE-2025-0021-grafana-datasource-failed.yaml) — Grafana datasource connection failed
- [CCVE-0022](fixtures/CCVE-2025-0022-grafana-pvc-unbound.yaml) — Grafana PVC unbound
- [CCVE-0024](fixtures/CCVE-2025-0024-grafana-dashboard-provisioning-failed.yaml) — Grafana dashboard provisioning failed
- [CCVE-0027](fixtures/CCVE-2025-0027-grafana-namespace-whitespace.yaml) — Grafana sidecar namespace whitespace (RBC incident)
- [CCVE-0028](fixtures/CCVE-2025-0028-traefik-service-not-found.yaml) — Traefik IngressRoute service not found
- [CCVE-0029](fixtures/CCVE-2025-0029-traefik-tls-passthrough-wrong-type.yaml) — Traefik TLS passthrough wrong resource type
- [CCVE-0031](fixtures/CCVE-2025-0031-traefik-entrypoint-not-defined.yaml) — Traefik IngressRoute entryPoint not defined

---

## Schema Reference

Each CCVE has:
- **YAML definition** — Machine-readable schema
- **Detection logic** — K8s API conditions for programmatic detection
- **Remediation steps** — Actionable fix commands
- **References** — Official documentation links
- **Confidence level** — High/Medium/Low

Example:
```yaml
id: CCVE-2025-0001
category: SOURCE
severity: critical
confidence: high
detection:
  resources: [GitRepository]
  condition: status.conditions[?type=='Ready'].status == 'False'
```

---

## Usage

### Scan for all CCVEs
```bash
cd ../test/atk
./scan
```

### Scan with JSON output
```bash
./scan --json > results.json
```

### Scan specific namespace
```bash
./scan --namespace production
```

### Filter by severity
```bash
./scan --severity critical
```

---

**Full Documentation:** [README.md](README.md)
**Field Guide:** [QUICK-REFERENCE.md](QUICK-REFERENCE.md)
**Execution Summary:** [../CCVE-EXECUTION-SUMMARY.md](../CCVE-EXECUTION-SUMMARY.md)
