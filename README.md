# Example Apps with ConfigHub

**Build, Manage and Operate modern applications with ConfigHub** 

In this repo you will find example apps for ConfigHub.  All of the apps are based on the [ConfigHub "Global App" pattern](https://github.com/confighub/examples/blob/main/global-app/README.md) which enables a set of microservice components to be deployed across stages and regions, and then kept in operation.  

You should consider everything here to be experimental -- we are seeking your feedback on the application model and deployment flow.  Our aspiration is that production grade apps are easy to build and deploy anywhere, using this model, then easily cloned, customised, combined or otherwise adapted simply by operating on configurations.

In addition we are experimenting with combinations of ConfigHub's automation model and AI agents -- towards AI Native Ops.

Our main tools are:
- **ConfigHub** to manage configurations, custom versions, and operational changes 
- **Kubernetes** for continuously running, event-driven and stateful applications
- **Claude AI** for intelligent decision-making agents
- **Canonical patterns** from ConfigHub's global-app reference and SDK

## Introducing DevOps as Apps

Although ConfigHub is useful for many kinds of applications, in these examples we want to focus on "DevOps Apps".  A lot of Ops and DevOps consists of "Jobs to be done" like deployment, change management, cost tracking, alerting on drift.  Traditionally practitioner tools run as scripts or ephemeral workflows that run and exit.  And more recently "Agentic workflows" have been proposed as way for AI-powered agents to integrate into operational workflows.  We believe the vision of "AI Native" automated operations is exciting.  In these examples we want to show you how to achieve this using *agentic devops applications* on ConfigHub, rather than using *workflows*.

**DevOps as Apps** treats automation as first-class applications with:

- ✅ **Persistent execution** - Run continuously, not just when triggered
- ✅ **Event-driven** - React immediately using Kubernetes informers
- ✅ **AI-powered** - Claude integration for intelligent decisions
- ✅ **Stateful** - Full history and audit trail in ConfigHub
- ✅ **Multi-environment** - Dev → staging → prod with push-upgrade
- ✅ **Observable** - Built-in dashboards and metrics
- ✅ **Maintainable** - Standard K8s deployment, versioning, rollback

### Comparison between vs Agentic Workflows and DevOps as Apps

| Aspect | Agentic Workflows | DevOps as Apps |
|--------|------------------|----------------|
| **Architecture** | Ephemeral workflows | Persistent applications |
| **State** | Stateless | ConfigHub-backed state |
| **Events** | Polling/triggers | Real-time informers |
| **AI** | Limited scope | Full Claude integration |
| **Environments** | Manual copying | Push-upgrade propagation |
| **Rollback** | Redeploy old version | Revision history |
| **Cost** | Per-workflow pricing | Open source + ConfigHub |

We believe these features are an excellent showcase for how SREs and DevOps teams can make use of ConfigHub for AI Native Operations.

## Get Started with Examples

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

- **[DevOps Examples Repository](https://github.com/monadic/devops-examples)** - Find our current 'devops as apps' examples here
  - [Drift Detector](https://github.com/monadic/devops-examples/tree/main/drift-detector) - Auto-correct configuration drift
  - [Cost Optimizer](https://github.com/monadic/devops-examples/tree/main/cost-optimizer) - AI-powered cost reduction
  - [Cost Impact Monitor](https://github.com/monadic/devops-examples/tree/main/cost-impact-monitor) - Pre-deployment cost analysis

Note that drift detection is a common use case in DevOps when declarative tools are used.  ConfigHub supports multiple scenarios for this, including integration with standard 3rd party GitOps tools.  However in this example we show how, for simple "dev" cases, drift can be observed directly and remedied using config updates.

## 🎯 Live Examples

Each example has its own docs and quick start for new users.  Below we summarise a couple of the examples for intro purposes.

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

See the [drift-detector README](https://github.com/monadic/devops-examples/tree/main/drift-detector) for full guide.

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

See the [cost-optimizer README](https://github.com/monadic/devops-examples/tree/main/cost-optimizer) for full guide.

## 🔑 Core Concepts

### ConfigHub Canonical Patterns

ConfigHub's ["global app" example](https://github.com/confighub/examples/blob/main/global-app/README.md) illustrates the main features of a ConfigHub app.

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

### DevOps App Structure and Deployment

Every app follows this standardized structure:

```
app-name/
├── README.md                    # Architecture and features
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

**Deployment flow:**
1. `bin/install-base` - Create spaces, filters, and units in ConfigHub
2. `bin/setup-worker` - Install worker that executes `cub unit apply`
3. `bin/apply-base` - Set targets and deploy to Kubernetes
4. `bin/test-workflow` - Validate everything works

### To learn more

- **[Canonical Patterns Summary](CANONICAL-PATTERNS-SUMMARY.md)** - A deeper explanation of the principles of ConfigHub apps 
- **[ConfigHub Deployment Pattern](CONFIGHUB-DEPLOYMENT-PATTERN.md)** - How to deploy apps via ConfigHub
- **[ConfigHub SDK extension libary](https://github.com/monadic/devops-sdk)** - some SDK addons 
- **[ConfigHub Actual Features](CONFIGHUB-ACTUAL-FEATURES.md)** - API notes (created by Claude)

This repository contains the core documentation for the ConfigHub pattern.  The examples are in a separate repo: [monadic/devops-examples](https://github.com/monadic/devops-examples).

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

### DevOps SDK Repository (Reusable Library)

See [monadic/devops-sdk](https://github.com/monadic/devops-sdk) for the Go SDK used by all examples, and [confighub/sdk](https://github.com/confighub/sdk) for the main ConfigHub SDK.

## 📞 Getting Help

- **ConfigHub**: [docs.confighub.com](https://docs.confighub.com)
- **Claude API**: [docs.anthropic.com](https://docs.anthropic.com)
- **Kubernetes**: [kubernetes.io/docs](https://kubernetes.io/docs)
- **Examples**: See each example's Troubleshooting section

## 🎓 Key Takeaways (according to Claude)

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

## 🤝 Contributing

To add a new DevOps app example:

1. Copy structure from drift-detector or cost-optimizer in [devops-examples](https://github.com/monadic/devops-examples)
2. Follow the global-app README pattern
3. Follow ConfigHub patterns from [CANONICAL-PATTERNS-SUMMARY.md](CANONICAL-PATTERNS-SUMMARY.md)
4. Consider scenario tasks with verification steps  

## 📄 License

See individual examples for licensing details.

---

**Built with ConfigHub** • **Powered by Claude AI** • **Better than Agentic DevOps Workflows**
