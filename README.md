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
kind create cluster --name devops-test  # Or use existing cluster

# 2. Pick an example (see https://github.com/monadic/devops-examples)
cd devops-examples/drift-detector/  # or cost-optimizer/

# 3. Deploy via ConfigHub
bin/install-base                   # Create ConfigHub structure
bin/setup-worker                   # Install ConfigHub worker
bin/apply-base                     # Deploy to Kubernetes
bin/test-workflow                  # Validate everything works

# 4. See the app in action
kubectl get all -n devops-apps     # Check deployment
# Then follow the QUICKSTART.md in each example
```

## 📚 Documentation

### For Getting Started

- **[DevOps Examples Repository](https://github.com/monadic/devops-examples)** - Production-ready examples
  - [Drift Detector](https://github.com/monadic/devops-examples/tree/main/drift-detector) - Auto-correct configuration drift
  - [Cost Optimizer](https://github.com/monadic/devops-examples/tree/main/cost-optimizer) - AI-powered cost reduction
  - [Cost Impact Monitor](https://github.com/monadic/devops-examples/tree/main/cost-impact-monitor) - Pre-deployment cost analysis

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
cd devops-examples/drift-detector/
bin/install-base     # Create ConfigHub structure
bin/setup-worker     # Install worker
bin/apply-base       # Deploy to Kubernetes

# Deploy test workloads with drift
bin/deploy-test --with-drift

# Watch auto-correction
kubectl logs -n devops-apps -l app=drift-detector --follow
```

**What it shows:**
- ConfigHub Sets and Filters for targeting
- Worker-based deployment via `cub unit apply`
- Push-upgrade for multi-environment fixes
- Claude AI drift pattern analysis
- Real-time dashboard on :8080

See [QUICKSTART.md](https://github.com/monadic/devops-examples/blob/main/drift-detector/QUICKSTART.md) for full guide.

---

### 2. Cost Optimizer

AI-powered cost optimization:

```bash
cd devops-examples/cost-optimizer/
bin/install-base     # Create ConfigHub structure
bin/setup-worker     # Install worker
bin/apply-base       # Deploy to Kubernetes

# View AI recommendations
kubectl port-forward -n cost-optimizer svc/cost-optimizer 8081:8081
# Open http://localhost:8081
```

**What it shows:**
- Claude AI cost recommendations
- Worker-based deployment workflow
- OpenCost integration for accurate billing
- Real-time dashboard on :8081
- Metrics server integration

See [QUICKSTART.md](https://github.com/monadic/devops-examples/blob/main/cost-optimizer/QUICKSTART.md) for full guide.

## 📖 Learning Path

**New to this project?** Follow this order:

1. **[Quick Start](#quick-start)** (5 min) - Get running
2. **[Drift Detector QUICKSTART](https://github.com/monadic/devops-examples/blob/main/drift-detector/QUICKSTART.md)** (20 min) - Step-by-step deployment
3. **[ConfigHub Workflow](https://github.com/monadic/devops-examples/blob/main/drift-detector/WORKFLOW.md)** (15 min) - Understand the workflow
4. **[Canonical Patterns](docs/CANONICAL-PATTERNS-SUMMARY.md)** (15 min) - Understand the patterns
5. **[Cost Optimizer Example](https://github.com/monadic/devops-examples/tree/main/cost-optimizer)** (30 min) - Advanced AI integration
6. **[Master Plan](docs/DEVOPS-AS-APPS-MASTER-PLAN.md)** (20 min) - Full architecture
7. **Build your own** - Apply patterns to your use case

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

Every app follows this standardized structure:

```
app-name/
├── README.md                    # Architecture and features
├── QUICKSTART.md               # Step-by-step setup guide
├── WORKFLOW.md                 # ConfigHub workflow explanation
├── bin/
│   ├── install-base            # Create ConfigHub structure
│   ├── setup-worker            # Install ConfigHub worker
│   ├── apply-base              # Set targets and apply units
│   ├── test-workflow           # Validate deployment
│   ├── install-envs            # Set up env hierarchy (optional)
│   ├── apply-all               # Deploy to specific env (optional)
│   ├── promote                 # Push-upgrade pattern (optional)
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

**Common workflow:**
1. `bin/install-base` - Create spaces, filters, and units in ConfigHub
2. `bin/setup-worker` - Install worker that executes `cub unit apply`
3. `bin/apply-base` - Set targets and deploy to Kubernetes
4. `bin/test-workflow` - Validate everything works

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

### This Repository (Planning & Documentation)
```
devops-as-apps-project/
├── README.md                                    # This file
└── docs/
    ├── CANONICAL-PATTERNS-SUMMARY.md           # 12 must-follow patterns
    ├── CONFIGHUB-ACTUAL-FEATURES.md            # API reference
    ├── CONFIGHUB-DEPLOYMENT-PATTERN.md         # Deployment guide
    ├── DEVOPS-AS-APPS-MASTER-PLAN.md          # Architecture & vision
    └── COMPETITIVE-ADVANTAGES.md               # Why this approach wins
```

### DevOps Examples Repository (Production Code)

See [monadic/devops-examples](https://github.com/monadic/devops-examples) for production-ready implementations:

```
devops-examples/
├── drift-detector/              # Continuous drift detection
│   ├── QUICKSTART.md           # Step-by-step setup
│   ├── WORKFLOW.md             # ConfigHub workflow
│   └── bin/                    # Deployment scripts
├── cost-optimizer/             # AI-powered cost optimization
│   ├── QUICKSTART.md           # Step-by-step setup
│   ├── WORKFLOW.md             # ConfigHub workflow
│   └── bin/                    # Deployment scripts
└── cost-impact-monitor/        # Pre-deployment cost analysis
    ├── QUICKSTART.md           # Step-by-step setup
    ├── WORKFLOW.md             # ConfigHub workflow
    └── bin/                    # Deployment scripts
```

### DevOps SDK Repository (Reusable Library)

See [monadic/devops-sdk](https://github.com/monadic/devops-sdk) for the Go SDK used by all examples.

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
