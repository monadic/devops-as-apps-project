# ConfigHub Common Vulnerabilities and Errors (CCVE) Database

The CCVE database catalogs known GitOps misconfiguration patterns — like MITRE CVE for code vulnerabilities, but for Kubernetes configuration.

**Version:** v0.1.0
**Last Updated:** 2025-12-29
**Total CCVEs:** 50

---

## What is a CCVE?

**CCVE** = ConfigHub Common Vulnerability/Error

A catalogued GitOps misconfiguration pattern with:
- **Unique ID** — permanent identifier (e.g., `CCVE-2025-0001`)
- **Detection logic** — programmatic identification via K8s API
- **Symptoms** — what users observe
- **Remediation** — how to fix it
- **Severity** — Critical, Warning, or Info

---

## Taxonomy

Every CCVE belongs to one category:

| Category | Description | Example CCVEs |
|----------|-------------|---------------|
| **SOURCE** | Can't fetch from Git/OCI/Helm repo | CCVE-2025-0001, 0003 |
| **RENDER** | Template/kustomize/helm rendering failed | CCVE-2025-0002 |
| **APPLY** | Cluster rejected the manifest | CCVE-2025-0004 |
| **DRIFT** | Live state diverged from desired | CCVE-2025-0005, 0011, 0012 |
| **DEPEND** | Dependency not ready, ordering issue | CCVE-2025-0015 |
| **STATE** | Stuck in pending/unknown/degraded | CCVE-2025-0006, 0007, 0013 |
| **ORPHAN** | Resource exists but owner is gone | (future) |
| **CONFIG** | Misconfiguration (not failure, but wrong) | CCVE-2025-0008, 0009, 0010 |

---

## Severity Levels

| Severity | Criteria | Action |
|----------|----------|--------|
| **Critical** | Deployment blocked, data loss risk, security exposure | Fix immediately |
| **Warning** | Degraded operation, drift, potential issues | Fix soon |
| **Info** | Suboptimal config, technical debt | Fix when convenient |

---

## Detection Confidence

| Confidence | Meaning | Example |
|------------|---------|---------|
| **High** | Deterministic — status field matches exactly | `status.conditions[?type=='Ready'].status == 'False'` |
| **Medium** | Heuristic — pattern matching, timing | Log message contains "timeout" |
| **Low** | Requires human judgment | "This looks like a misconfiguration" |

---

## CCVE Catalog

### Critical Severity

| ID | Name | Tool | Category |
|----|------|------|----------|
| [CCVE-2025-0001](CCVE-2025-0001.md) | GitRepository not ready | Flux | SOURCE |
| [CCVE-2025-0002](CCVE-2025-0002.md) | Kustomization build failed | Flux | RENDER |
| [CCVE-2025-0003](CCVE-2025-0003.md) | HelmRelease chart not ready | Flux | SOURCE |
| [CCVE-2025-0004](CCVE-2025-0004.md) | Application sync failed | Argo | APPLY |
| [CCVE-2025-0007](CCVE-2025-0007.md) | Helm release failed | Helm | STATE |
| [CCVE-2025-0012](CCVE-2025-0012.md) | Image tag drift | Any | DRIFT |
| [CCVE-2025-0015](CCVE-2025-0015.md) | ConfigHub worker disconnected | ConfigHub | DEPEND |
| [CCVE-2025-0021](CCVE-2025-0021.md) | Grafana datasource connection failed | Grafana | DEPEND |
| [CCVE-2025-0022](CCVE-2025-0022.md) | Grafana persistent volume unbound | Grafana | DEPEND |
| [CCVE-2025-0023](CCVE-2025-0023.md) | Grafana LDAP authentication failure | Grafana | CONFIG |
| [CCVE-2025-0028](CCVE-2025-0028.md) | IngressRoute service not found | Traefik | DEPEND |
| [CCVE-2025-0029](CCVE-2025-0029.md) | TLS passthrough using wrong resource type | Traefik | CONFIG |
| [CCVE-2025-0031](CCVE-2025-0031.md) | IngressRoute entryPoint not defined | Traefik | CONFIG |
| [CCVE-2025-0032](CCVE-2025-0032.md) | TLS certificate secret not found | Traefik | DEPEND |
| [CCVE-2025-0041](CCVE-2025-0041.md) | ServiceMonitor not discovered by Prometheus | Prometheus | CONFIG |
| [CCVE-2025-0042](CCVE-2025-0042.md) | Thanos object storage configuration invalid | Thanos | CONFIG |
| [CCVE-2025-0043](CCVE-2025-0043.md) | Thanos sidecar not uploading blocks | Thanos | CONFIG |
| [CCVE-2025-0044](CCVE-2025-0044.md) | Thanos Query StoreAPI endpoint unreachable | Thanos | DEPEND |
| [CCVE-2025-0048](CCVE-2025-0048.md) | Secrets Store CSI driver not registered on node | CSI | DEPEND |
| [CCVE-2025-0049](CCVE-2025-0049.md) | SecretProviderClass namespace mismatch | CSI | CONFIG |
| [CCVE-2025-0050](CCVE-2025-0050.md) | Secrets Store CSI mount failed - provider error | CSI | CONFIG |

### Warning Severity

| ID | Name | Tool | Category |
|----|------|------|----------|
| [CCVE-2025-0005](CCVE-2025-0005.md) | Application out of sync | Argo | DRIFT |
| [CCVE-2025-0006](CCVE-2025-0006.md) | Application health degraded | Argo | STATE |
| [CCVE-2025-0008](CCVE-2025-0008.md) | HelmRelease install retries exhausted | Flux | CONFIG |
| [CCVE-2025-0010](CCVE-2025-0010.md) | Resource validation skipped - CRD missing | Argo | CONFIG |
| [CCVE-2025-0011](CCVE-2025-0011.md) | Manual kubectl edit detected | Any | DRIFT |
| [CCVE-2025-0013](CCVE-2025-0013.md) | Helm release pending upgrade | Helm | STATE |
| [CCVE-2025-0014](CCVE-2025-0014.md) | ConfigHub unit pending changes | ConfigHub | DRIFT |
| [CCVE-2025-0016](CCVE-2025-0016.md) | HelmRelease dependency not ready | Flux | DEPEND |
| [CCVE-2025-0017](CCVE-2025-0017.md) | Argo Application resource too large | Argo | CONFIG |
| [CCVE-2025-0018](CCVE-2025-0018.md) | Flux webhook validation failure | Flux | CONFIG |
| [CCVE-2025-0019](CCVE-2025-0019.md) | Unmanaged resource in managed namespace | Any | ORPHAN |
| [CCVE-2025-0020](CCVE-2025-0020.md) | Replica count drift | Any | DRIFT |
| [CCVE-2025-0024](CCVE-2025-0024.md) | Grafana dashboard provisioning failed | Grafana | CONFIG |
| [CCVE-2025-0025](CCVE-2025-0025.md) | Grafana provisioned datasource not found | Grafana | CONFIG |
| [CCVE-2025-0026](CCVE-2025-0026.md) | Grafana plugin installation failed | Grafana | DEPEND |
| [CCVE-2025-0027](CCVE-2025-0027.md) | Grafana sidecar namespace whitespace error | Grafana | CONFIG |
| [CCVE-2025-0030](CCVE-2025-0030.md) | Middleware not found | Traefik | DEPEND |
| [CCVE-2025-0033](CCVE-2025-0033.md) | Cross-namespace service reference blocked | Traefik | CONFIG |
| [CCVE-2025-0045](CCVE-2025-0045.md) | Alertmanager configuration invalid | Alertmanager | CONFIG |
| [CCVE-2025-0046](CCVE-2025-0046.md) | Alert route not matching any alerts | Alertmanager | CONFIG |
| [CCVE-2025-0047](CCVE-2025-0047.md) | Inhibit rules using deprecated syntax | Alertmanager | CONFIG |

### Info Severity

| ID | Name | Tool | Category |
|----|------|------|----------|
| [CCVE-2025-0009](CCVE-2025-0009.md) | Flux reconciliation suspended | Flux | CONFIG |

---

## Quick Reference by Tool

### Flux CCVEs

- **CCVE-2025-0001** — GitRepository not ready (Critical)
- **CCVE-2025-0002** — Kustomization build failed (Critical)
- **CCVE-2025-0003** — HelmRelease chart not ready (Critical)
- **CCVE-2025-0008** — Install retries exhausted (Warning)
- **CCVE-2025-0009** — Reconciliation suspended (Info)
- **CCVE-2025-0016** — HelmRelease dependency not ready (Warning)
- **CCVE-2025-0018** — Webhook validation failure (Warning)

### Argo CD CCVEs

- **CCVE-2025-0004** — Application sync failed (Critical)
- **CCVE-2025-0005** — Application out of sync (Warning)
- **CCVE-2025-0006** — Application health degraded (Warning)
- **CCVE-2025-0010** — CRD missing validation error (Warning)
- **CCVE-2025-0017** — Application resource too large (Warning)

### Helm CCVEs

- **CCVE-2025-0007** — Release failed (Critical)
- **CCVE-2025-0013** — Release pending upgrade (Warning)

### ConfigHub CCVEs

- **CCVE-2025-0014** — Unit pending changes (Warning)
- **CCVE-2025-0015** — Worker disconnected (Critical)

### Grafana CCVEs

- **CCVE-2025-0021** — Datasource connection failed (Critical)
- **CCVE-2025-0022** — Persistent volume unbound (Critical)
- **CCVE-2025-0023** — LDAP authentication failure (Critical)
- **CCVE-2025-0024** — Dashboard provisioning failed (Warning)
- **CCVE-2025-0025** — Provisioned datasource not found (Warning)
- **CCVE-2025-0026** — Plugin installation failed (Warning)
- **CCVE-2025-0027** — Sidecar namespace whitespace error (Warning)

### Traefik CCVEs

- **CCVE-2025-0028** — IngressRoute service not found (Critical)
- **CCVE-2025-0029** — TLS passthrough using wrong resource type (Critical)
- **CCVE-2025-0030** — Middleware not found (Warning)
- **CCVE-2025-0031** — IngressRoute entryPoint not defined (Critical)
- **CCVE-2025-0032** — TLS certificate secret not found (Critical)
- **CCVE-2025-0033** — Cross-namespace service reference blocked (Warning)

### Prometheus CCVEs

- **CCVE-2025-0041** — ServiceMonitor not discovered by Prometheus (Critical)

### Thanos CCVEs

- **CCVE-2025-0042** — Object storage configuration invalid (Critical)
- **CCVE-2025-0043** — Sidecar not uploading blocks (Critical)
- **CCVE-2025-0044** — Query StoreAPI endpoint unreachable (Critical)

### Alertmanager CCVEs

- **CCVE-2025-0045** — Configuration invalid (Critical)
- **CCVE-2025-0046** — Alert route not matching any alerts (Warning)
- **CCVE-2025-0047** — Inhibit rules using deprecated syntax (Warning)

### Secrets Store CSI Driver CCVEs

- **CCVE-2025-0048** — CSI driver not registered on node (Critical)
- **CCVE-2025-0049** — SecretProviderClass namespace mismatch (Critical)
- **CCVE-2025-0050** — Mount failed - provider error (Critical)

### Cross-Tool CCVEs

- **CCVE-2025-0011** — Manual kubectl edit detected (Warning)
- **CCVE-2025-0012** — Image tag drift (Critical)
- **CCVE-2025-0019** — Unmanaged resource in managed namespace (Warning)
- **CCVE-2025-0020** — Replica count drift (Warning)

---

## Using the Scanner

The ConfigHub Agent includes a built-in CCVE scanner:

```bash
# Scan current cluster
cd ../test/atk
./scan

# JSON output for tooling
./scan --json

# Scan specific namespace
./scan --namespace production

# Filter by severity
./scan --severity critical,warning
```

### Example Output

```
CONFIG CVE SCAN: prod-east
══════════════════════════════════════════════════════════════════════════

CRITICAL (2)
──────────────────────────────────────────────────────────────────────────
[CCVE-2025-0001] GitRepository not ready
  Resource: flux-system/GitRepository/app-repo
  Message:  authentication required
  Fix:      Verify repository URL and credentials
  Docs:     https://confighub.io/cve/CCVE-2025-0001

[CCVE-2025-0004] Application sync failed
  Resource: argocd/Application/api
  Message:  ComparisonError
  Fix:      Check application manifest and target cluster connectivity
  Docs:     https://confighub.io/cve/CCVE-2025-0004

WARNING (3)
──────────────────────────────────────────────────────────────────────────
[CCVE-2025-0005] Application out of sync
  Resource: argocd/Application/frontend
  Message:  Live state differs from Git source
  Fix:      Sync application or update Git to match desired state
  Docs:     https://confighub.io/cve/CCVE-2025-0005

INFO (1)
──────────────────────────────────────────────────────────────────────────
[CCVE-2025-0009] Flux reconciliation suspended
  Resource: flux-system/Kustomization/infrastructure
  Message:  spec.suspend=true
  Fix:      Intentional if maintenance; otherwise set suspend=false
  Docs:     https://confighub.io/cve/CCVE-2025-0009

══════════════════════════════════════════════════════════════════════════
Summary: 2 critical, 3 warning, 1 info
```

---

## CCVE Schema

Each CCVE consists of two files:

### YAML Definition (`CCVE-YYYY-NNNN.yaml`)

```yaml
id: CCVE-2025-0001
aliases:
  - CCVE-FLUX-001
category: SOURCE
name: GitRepository not ready
severity: critical
confidence: high
version_added: "0.1.0"

affected_tools:
  - name: flux
    versions: ">= 0.30.0"

sources:
  - url: https://fluxcd.io/flux/cheatsheets/troubleshooting/
    type: official_docs

symptoms:
  - Kustomizations remain in "Not Ready" state
  - Changes pushed to Git not reflected in cluster

detection:
  resources: [GitRepository]
  condition: |
    status.conditions[?type=='Ready'].status == 'False'
  confidence: high

remediation:
  steps:
    - Check GitRepository authentication
    - Verify repository URL is accessible
    - Review source-controller logs
  commands:
    - flux get sources git -A
    - kubectl describe gitrepository -n flux-system <name>

references:
  - https://fluxcd.io/flux/cheatsheets/troubleshooting/

deprecated: false
superseded_by: null
```

### Markdown Documentation (`CCVE-YYYY-NNNN.md`)

Detailed explanation with:
- Overview and context
- Detailed symptoms
- Root cause analysis
- Step-by-step remediation
- Prevention guidance
- Related CCVEs
- References

---

## Contributing New CCVEs

### 1. Create YAML Definition

```bash
cp CCVE-2025-0001.yaml CCVE-2025-NNNN.yaml
# Edit with new pattern details
```

### 2. Create Markdown Documentation

```bash
cp CCVE-2025-0001.md CCVE-2025-NNNN.md
# Add detailed explanation
```

### 3. Add Test Fixture (Optional)

```bash
# Create fixture demonstrating the error
cat > fixtures/CCVE-2025-NNNN.yaml <<EOF
# Kubernetes manifests that trigger this CCVE
EOF
```

### 4. Update This README

Add the new CCVE to:
- Total count at top
- Appropriate severity table
- Tool-specific section

### 5. Test Detection

```bash
cd ../test/atk
./scan --pattern CCVE-2025-NNNN
```

---

## CCVE Lifecycle

- **Active** — Currently valid and detectable
- **Deprecated** — Pattern no longer relevant (tool version changes, etc.)
- **Superseded** — Replaced by newer CCVE with better detection

**IDs are never reused** — even deprecated CCVEs keep their ID for historical reference.

---

## Data Sources

| Priority | Source | Signal Quality | Status |
|----------|--------|----------------|--------|
| 1 | Official troubleshooting docs | Very high | ✅ Phase 1 |
| 2 | GitHub issues (closed bugs) | High | 📋 Phase 2 |
| 3 | Stack Overflow top-voted | Medium | 📋 Phase 2 |
| 4 | Maintainer relationships | Very high | 📋 Phase 3 |
| 5 | Telemetry (opt-in) | Highest | 📋 Phase 4 |

---

## OSS vs Commercial

| Capability | OSS Agent | ConfigHub |
|------------|-----------|-----------|
| CCVE database (static patterns) | ✓ | ✓ |
| Local cluster scanning | ✓ | ✓ |
| Known CCVE detection | ✓ | ✓ |
| Community contributions | ✓ | ✓ |
| Fleet-wide scanning | — | ✓ |
| Trend analysis | — | ✓ |
| Pattern discovery from telemetry | — | ✓ |
| Fix success tracking | — | ✓ |
| Custom/private CCVEs | — | ✓ |
| Auto-remediation Actions | — | ✓ |

---

## Roadmap

### Phase 1: Seed ✅ (Week 1-2)
- Scrape official docs from Flux, Argo CD, Helm, Grafana ✅
- Define taxonomy and schema ✅
- Create 26 CCVEs (Flux, Argo, Helm, ConfigHub, Grafana, Cross-tool) ✅
- Test fixtures for critical patterns ✅
- Implement scanner (in progress)

### Phase 2: Mine 📋 (Week 3-4)
- Build GitHub issue mining pipeline
- Extract patterns with LLM
- Human review queue
- Target: 50+ additional CCVEs

### Phase 3: Publish 📋 (Week 5-6)
- Launch public database website
- Create GitHub Action for CI
- Write taxonomy blog post
- Maintainer outreach

### Phase 4: Differentiate 📋 (Week 7-12)
- Telemetry opt-in
- Fleet-wide analysis
- Pattern discovery
- Auto-remediation

---

## Success Metrics

| Metric | Target (3 months) | Target (6 months) |
|--------|-------------------|-------------------|
| CCVEs in database | 50+ | 100+ |
| GitHub stars | 500+ | 1,500+ |
| Scanner usage | 1,000+ | 5,000+ |
| Community contributions | 10+ | 50+ |

---

## Related Resources

- [ConfigHub Agent](../README.md) — OSS cluster observer
- [Agent Test Kit](../test/atk/) — Testing framework
- [GitOps State Format](../docs/MAP-SCHEMA.md) — Universal output format
- [CCVE Plan](https://github.com/confighub/ccve-plan) — Full product plan

---

## License

CCVE Database: Apache 2.0 (free to use, modify, distribute)

---

## Support

- **Issues:** https://github.com/confighub/confighub-agent/issues
- **Discussions:** https://github.com/confighub/confighub-agent/discussions
- **Email:** cve@confighub.io
