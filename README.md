# DevOps as Apps

**Build DevOps tools as persistent Kubernetes applications** - continuously running, maintaining state, using AI, and reacting to events in real-time.

This project demonstrates how to build modern DevOps automation using:
- **ConfigHub** for configuration management and environment promotion
- **Claude AI** for intelligent decision-making
- **Kubernetes** for persistent, event-driven applications
- **Canonical patterns** from ConfigHub's global-app reference

## Why DevOps as Apps?

Traditional DevOps tools are scripts or ephemeral workflows that run and exit. **DevOps as Apps** treats automation as first-class applications with:

- ✅ **Persistent execution** - Run continuously, not just when triggered
- ✅ **Event-driven** - React immediately using Kubernetes informers
- ✅ **AI-powered** - Claude integration for intelligent decisions
- ✅ **Stateful** - Full history and audit trail in ConfigHub
- ✅ **Multi-environment** - Dev → staging → prod with push-upgrade
- ✅ **Observable** - Built-in dashboards and metrics
- ✅ **Maintainable** - Standard K8s deployment, versioning, rollback

### vs Agentic DevOps Workflows (e.g., Cased)

| Aspect | Agentic Workflows | DevOps as Apps |
|--------|------------------|----------------|
| **Architecture** | Ephemeral workflows | Persistent applications |
| **State** | Stateless | ConfigHub-backed state |
| **Events** | Polling/triggers | Real-time informers |
| **AI** | Limited scope | Full Claude integration |
| **Environments** | Manual copying | Push-upgrade propagation |
| **Rollback** | Redeploy old version | Revision history |
| **Cost** | Per-workflow pricing | Open source + ConfigHub |

## Quick Start

```bash
# 1. Install prerequisites
brew install confighubai/tap/cub  # ConfigHub CLI
cub auth login                     # Authenticate

# 2. Pick an example
cd examples/drift-detector/        # or cost-optimizer/

# 3. Set up ConfigHub structure
bin/install-base                   # Create unique prefix, spaces, filters
bin/install-envs                   # Create environment hierarchy

# 4. Deploy and explore
bin/apply-all dev                  # Deploy via ConfigHub
# Then follow scenario tasks in the example's README.md
```

## 📚 Documentation

### For Getting Started

- **[Examples](examples/)** - Hands-on, scenario-driven guides
  - [Drift Detector](examples/drift-detector/) - Auto-correct configuration drift
  - [Cost Optimizer](examples/cost-optimizer/) - AI-powered cost reduction

### For Understanding Patterns

- **[Canonical Patterns Summary](docs/CANONICAL-PATTERNS-SUMMARY.md)** - 12 patterns you MUST follow
- **[ConfigHub Deployment Pattern](docs/CONFIGHUB-DEPLOYMENT-PATTERN.md)** - How to deploy apps via ConfigHub
- **[ConfigHub Actual Features](docs/CONFIGHUB-ACTUAL-FEATURES.md)** - API reference (what's real vs hallucinated)

### For Architecture & Design

- **[Master Plan](docs/DEVOPS-AS-APPS-MASTER-PLAN.md)** - High-level architecture and vision
- **[Competitive Advantages](docs/COMPETITIVE-ADVANTAGES.md)** - Why this beats traditional approaches

## 🎯 Live Examples

### 1. Drift Detector

Continuous drift detection with auto-correction:

```bash
cd examples/drift-detector/
bin/install-base && bin/install-envs
bin/apply-all dev

# Deploy test workloads with drift
bin/deploy-test --with-drift

# Watch auto-correction
kubectl logs -n devops-apps -l app=drift-detector --follow
```

**What it shows:**
- ConfigHub Sets and Filters for targeting
- Push-upgrade for multi-environment fixes
- Changesets for atomic bulk corrections
- Claude AI drift pattern analysis
- Real-time dashboard on :8080

---

### 2. Cost Optimizer

AI-powered cost optimization:

```bash
cd examples/cost-optimizer/
bin/install-base && bin/install-envs
bin/apply-all dev

# Deploy over-provisioned workloads
bin/deploy-test-workloads

# View AI recommendations
kubectl port-forward -n devops-apps svc/cost-optimizer 8081:8081
# Open http://localhost:8081
```

**What it shows:**
- Claude AI cost recommendations
- Store optimizations as ConfigHub units
- Promote optimizations dev → staging → prod
- Lateral promotion for region-specific opts
- OpenCost integration for accurate billing
- Real-time dashboard on :8081

## 📖 Learning Path

**New to this project?** Follow this order:

1. **[Quick Start](#quick-start)** (5 min) - Get running
2. **[Drift Detector Example](examples/drift-detector/)** (30 min) - Core patterns
3. **[Canonical Patterns](docs/CANONICAL-PATTERNS-SUMMARY.md)** (15 min) - Understand the patterns
4. **[Cost Optimizer Example](examples/cost-optimizer/)** (35 min) - Advanced AI integration
5. **[Master Plan](docs/DEVOPS-AS-APPS-MASTER-PLAN.md)** (20 min) - Full architecture
6. **Build your own** - Apply patterns to your use case

## 🔑 Core Concepts

### ConfigHub Canonical Patterns

All examples follow these patterns from the global-app reference:

1. **Unique Project Prefix** - `cub space new-prefix` prevents collisions
2. **Environment Hierarchy** - base → dev → staging → prod
3. **Filters for Targeting** - SQL-like WHERE clauses
4. **Sets for Grouping** - Bulk operations on related units
5. **Push-Upgrade** - Automatic downstream propagation
6. **Changesets** - Atomic multi-unit operations
7. **Lateral Promotion** - Conservative region-by-region rollout
8. **Revision Management** - Diff, rollback, audit trail
9. **ConfigHub Functions** - `cub run` for common operations
10. **Links** - Connect app units to infrastructure

See [CANONICAL-PATTERNS-SUMMARY.md](docs/CANONICAL-PATTERNS-SUMMARY.md) for details.

### DevOps App Structure

Every app follows this structure:

```
app-name/
├── README.md                    # Scenario-driven guide
├── bin/
│   ├── install-base            # Create ConfigHub structure
│   ├── install-envs            # Set up environment hierarchy
│   ├── apply-all               # Deploy via ConfigHub
│   ├── promote                 # Push-upgrade pattern
│   └── proj                    # Get project name
├── confighub/
│   └── base/
│       ├── namespace.yaml      # K8s namespace unit
│       ├── deployment.yaml     # App deployment unit
│       ├── service.yaml        # Service unit
│       └── rbac.yaml           # RBAC unit
├── main.go                     # Application code
└── go.mod                      # Go module
```

## 🛠️ Prerequisites

### Required

- **ConfigHub account** - Sign up at [confighub.com](https://confighub.com)
- **ConfigHub CLI (cub)** - `brew install confighubai/tap/cub`
- **Kubernetes cluster** - Kind, Minikube, or any K8s 1.19+
- **kubectl** - Configured for your cluster

### Optional

- **Go 1.21+** - For local development
- **Claude API key** - For AI features ([get one](https://console.anthropic.com))
- **OpenCost** - For accurate cloud billing (cost-optimizer example)

## 📦 What's Included

```
devops-as-apps-project/
├── README.md                                    # This file
├── examples/
│   ├── README.md                               # Examples index
│   ├── drift-detector/
│   │   └── README.md                           # Scenario-driven guide
│   └── cost-optimizer/
│       └── README.md                           # Scenario-driven guide
└── docs/
    ├── CANONICAL-PATTERNS-SUMMARY.md           # 12 must-follow patterns
    ├── CONFIGHUB-ACTUAL-FEATURES.md            # API reference
    ├── CONFIGHUB-DEPLOYMENT-PATTERN.md         # Deployment guide
    ├── DEVOPS-AS-APPS-MASTER-PLAN.md          # Architecture & vision
    └── COMPETITIVE-ADVANTAGES.md               # Why this approach wins
```

## 🎓 Key Takeaways

### For Platform Engineers

- **Stop writing scripts** - Build persistent apps instead
- **Use ConfigHub** - Single source of truth for config
- **AI-powered** - Let Claude make intelligent decisions
- **Event-driven** - React immediately with informers
- **Multi-environment** - Test in dev, promote to prod safely

### For DevOps Teams

- **Better than workflows** - Continuous, not ephemeral
- **Full audit trail** - Every change tracked in ConfigHub
- **Easy rollback** - Revision history for all configs
- **Bulk operations** - Fix drift across all environments at once
- **Cost savings** - Claude finds optimization opportunities

### For Architects

- **Modern architecture** - Apps, not scripts
- **Kubernetes-native** - Standard K8s deployment patterns
- **Observable** - Built-in dashboards and metrics
- **Scalable** - Horizontal scaling, HA, zero downtime
- **Extensible** - Full source control, easy to customize

## 🚀 Next Steps

1. **Try an example** - Start with [Drift Detector](examples/drift-detector/)
2. **Read the patterns** - [Canonical Patterns](docs/CANONICAL-PATTERNS-SUMMARY.md)
3. **Understand ConfigHub** - [Actual Features](docs/CONFIGHUB-ACTUAL-FEATURES.md)
4. **Review architecture** - [Master Plan](docs/DEVOPS-AS-APPS-MASTER-PLAN.md)
5. **Build your own** - Apply patterns to your use case

## 📞 Getting Help

- **ConfigHub**: [docs.confighub.com](https://docs.confighub.com)
- **Claude API**: [docs.anthropic.com](https://docs.anthropic.com)
- **Kubernetes**: [kubernetes.io/docs](https://kubernetes.io/docs)
- **Examples**: See each example's Troubleshooting section

## 🤝 Contributing

To add a new DevOps app example:

1. Copy structure from drift-detector or cost-optimizer
2. Follow the global-app README pattern
3. Use all 12 canonical ConfigHub patterns
4. Include scenario tasks with verification steps
5. Add to [examples/README.md](examples/README.md)

See [DEVOPS-AS-APPS-MASTER-PLAN.md](docs/DEVOPS-AS-APPS-MASTER-PLAN.md) for architecture guidance.

## 📄 License

See individual examples for licensing details.

---

**Built with ConfigHub** • **Powered by Claude AI** • **Better than Agentic DevOps Workflows**
