# DevOps as Apps - Examples

This directory contains practical examples of DevOps tools built as persistent Kubernetes applications using ConfigHub for configuration management and Claude AI for intelligent decision-making.

## Available Examples

### 1. [Drift Detector](drift-detector/)

**What it demonstrates:**
- Continuous drift detection using Kubernetes informers
- ConfigHub Sets and Filters for grouping and targeting
- Push-upgrade pattern for multi-environment propagation
- Changesets for atomic bulk corrections
- Claude AI for drift pattern analysis

**Key ConfigHub features:**
- Event-driven architecture (not polling)
- Revision history and rollback
- Lateral promotion for conservative rollout
- Live state monitoring

**Scenario tasks:**
- Detect and auto-fix drift
- Promote fixes across environments
- Use changesets for bulk corrections
- Monitor drift patterns with Sets

**Time to complete:** ~30 minutes

---

### 2. [Cost Optimizer](cost-optimizer/)

**What it demonstrates:**
- Continuous cost optimization with Claude AI
- Real-time cost analysis and recommendations
- ConfigHub-driven optimization deployment
- OpenCost integration for accurate billing
- Multi-region cost management

**Key ConfigHub features:**
- Store recommendations as versioned units
- Push-upgrade for testing optimizations
- Lateral promotion for region-specific opts
- Rollback on failed optimizations

**Scenario tasks:**
- Analyze costs and view recommendations
- Apply and promote optimizations
- Roll back problematic changes
- Use changesets for bulk optimization

**Time to complete:** ~35 minutes

---

## Example Structure

All examples follow the same canonical pattern derived from ConfigHub's global-app reference:

```
example-name/
├── README.md                    # Scenario-driven guide
├── bin/
│   ├── install-base            # Create ConfigHub structure
│   ├── install-envs            # Set up environment hierarchy
│   ├── apply-all               # Deploy via ConfigHub
│   ├── promote                 # Push-upgrade pattern
│   ├── proj                    # Get project name
│   └── ...                     # Example-specific scripts
├── confighub/
│   └── base/
│       ├── namespace.yaml      # K8s namespace unit
│       ├── deployment.yaml     # App deployment unit
│       ├── service.yaml        # Service unit
│       └── rbac.yaml           # RBAC unit
├── main.go                     # Application code
├── go.mod                      # Go module
└── run.sh                      # Quick start script
```

## Common Prerequisites

All examples require:

- **ConfigHub account**: Sign up at [confighub.com](https://confighub.com)
- **ConfigHub CLI**: Install with `brew install confighubai/tap/cub` (or download from releases)
- **Kubernetes cluster**: Kind, Minikube, or any K8s 1.19+
- **kubectl**: Configured for your cluster
- **Go 1.21+**: For local development (optional)

Set up authentication:

```bash
# Authenticate with ConfigHub
cub auth login

# Verify connection
cub space list
```

## Quick Start Any Example

```bash
# 1. Navigate to example
cd drift-detector/  # or cost-optimizer/

# 2. Set up ConfigHub structure
bin/install-base
bin/install-envs

# 3. Deploy to Kubernetes
bin/apply-all dev

# 4. Follow scenario tasks in README.md
```

## Comparison: DevOps as Apps vs Agentic Workflows

| Aspect | Agentic Workflows (e.g., Cased) | DevOps as Apps (Our Examples) |
|--------|----------------------------------|-------------------------------|
| **Execution** | Ephemeral, exits after run | Persistent, continuous |
| **State** | Stateless between runs | Stateful with ConfigHub history |
| **Events** | Polling or cron triggers | Real-time Kubernetes informers |
| **AI** | Limited to workflow scope | Full Claude API integration |
| **Multi-env** | Manually copy workflows | Push-upgrade propagation |
| **Rollback** | Redeploy old workflow | ConfigHub revision history |
| **Bulk Ops** | Loop through resources | Sets/Filters with WHERE clauses |
| **Audit** | Workflow logs | Complete ConfigHub versioning |

## ConfigHub Canonical Patterns Used

All examples demonstrate these patterns from the global-app reference:

1. **Unique Project Prefix**: `cub space new-prefix` for collision-free naming
2. **Environment Hierarchy**: base → dev → staging → prod
3. **Filters for Targeting**: WHERE clauses for selective operations
4. **Sets for Grouping**: Bulk operations on related units
5. **Push-Upgrade**: Automatic downstream propagation
6. **Changesets**: Atomic multi-unit operations
7. **Lateral Promotion**: Conservative region-by-region rollout
8. **Revision Management**: Diff, rollback, audit trail
9. **ConfigHub Functions**: `cub run` for common operations
10. **Links**: Connect app units to infrastructure

See [../docs/CANONICAL-PATTERNS-SUMMARY.md](../docs/CANONICAL-PATTERNS-SUMMARY.md) for details.

## Learning Path

**Recommended order:**

1. **Start here**: [Drift Detector](drift-detector/) - Simplest example, core patterns
2. **Then**: [Cost Optimizer](cost-optimizer/) - Advanced AI integration, OpenCost
3. **Next**: Read [canonical patterns](../docs/CANONICAL-PATTERNS-SUMMARY.md)
4. **Finally**: Build your own DevOps app using the patterns

## Example-Specific Features

### Drift Detector Highlights

- ✅ Event-driven with Kubernetes informers
- ✅ Real-time drift detection (< 1 second)
- ✅ Claude AI drift pattern analysis
- ✅ Auto-correction with audit trail
- ✅ Dashboard on port 8080

### Cost Optimizer Highlights

- ✅ Continuous cost monitoring
- ✅ Claude AI recommendations
- ✅ OpenCost integration for accurate billing
- ✅ Region-specific optimizations
- ✅ Dashboard on port 8081
- ✅ Historical cost trending

## Future Examples (Coming Soon)

3. **Security Scanner** - CVE detection with auto-patching
4. **Compliance Auditor** - Continuous policy enforcement
5. **Upgrade Manager** - Intelligent version rollout
6. **Branch Deployer** - Better than "killer branch deploy"

## Getting Help

**ConfigHub Questions:**
- Docs: [docs.confighub.com](https://docs.confighub.com)
- Slack: [confighub.com/slack](https://confighub.com/slack)
- Examples: [github.com/confighubai/examples](https://github.com/confighubai/examples)

**Claude API Questions:**
- Docs: [docs.anthropic.com](https://docs.anthropic.com)
- Get API key: [console.anthropic.com](https://console.anthropic.com)

**Example-Specific Issues:**
- Check example's README.md Troubleshooting section
- Review [canonical patterns](../docs/CANONICAL-PATTERNS-SUMMARY.md)
- Verify ConfigHub features in [actual features doc](../docs/CONFIGHUB-ACTUAL-FEATURES.md)

## Contributing

To add a new example:

1. Copy the structure from drift-detector or cost-optimizer
2. Follow the global-app README pattern
3. Include all canonical ConfigHub patterns
4. Add scenario tasks with verification steps
5. Test on fresh Kind cluster
6. Document all prerequisites clearly
7. Add to this index

See [../docs/DEVOPS-AS-APPS-MASTER-PLAN.md](../docs/DEVOPS-AS-APPS-MASTER-PLAN.md) for architecture guidance.
