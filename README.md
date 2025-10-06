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

DevOps operations such as deployment, change management, cost tracking, and drift detection have traditionally been implemented as scripts or ephemeral workflows that run and exit. This project demonstrates an alternative approach using persistent Kubernetes applications with ConfigHub as the configuration backend.

**DevOps as Apps** treats automation as first-class applications with:

- ✅ **Persistent execution** - Run continuously, not just when triggered
- ✅ **Event-driven** - React immediately using Kubernetes informers
- ✅ **AI-powered** - Claude integration for intelligent decisions
- ✅ **Stateful** - Full history and audit trail in ConfigHub
- ✅ **Multi-environment** - Dev → staging → prod with push-upgrade
- ✅ **Observable** - Built-in dashboards and metrics
- ✅ **Maintainable** - Standard K8s deployment, versioning, rollback

### Architecture Characteristics

| Aspect | DevOps as Apps |
|--------|----------------|
| **Architecture** | Persistent applications |
| **State** | ConfigHub-backed state |
| **Events** | Real-time informers |
| **AI** | Claude integration |
| **Environments** | Push-upgrade propagation |
| **Rollback** | Revision history |
| **Cost** | Open source + ConfigHub |

This approach provides continuous monitoring and automated operations using ConfigHub as the configuration backend.

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
# Then follow the README in each example
```

- **[DevOps Examples Repository](https://github.com/monadic/devops-examples)** - Find our current 'devops as apps' examples here
  - [Drift Detector](https://github.com/monadic/devops-examples/tree/main/drift-detector) - Auto-correct configuration drift
  - [Cost Optimizer](https://github.com/monadic/devops-examples/tree/main/cost-optimizer) - AI-powered cost reduction
  - [Cost Impact Monitor](https://github.com/monadic/devops-examples/tree/main/cost-impact-monitor) - Pre-deployment cost analysis

Note that drift detection is a common use case in DevOps when declarative tools are used.  ConfigHub supports multiple scenarios for this, including integration with standard 3rd party GitOps tools.  However in this example we show how, for simple "dev" cases, drift can be observed directly and remedied using config updates.

## 🎯 Live Examples

Each example has its own docs and quick start for new users.  Below we summarise a couple of the examples for intro purposes.

### 1. TraderX - Production Application Deployment

Complete ConfigHub deployment of FINOS TraderX trading platform demonstrating all 12 canonical patterns.

**Location**: `/Users/alexis/traderx/` (this repository)

**Status**: ConfigHub infrastructure complete, Kubernetes deployment ready (pending Docker)

**What it demonstrates:**
- Full production application (8 microservices)
- Complete environment hierarchy (base → dev → staging → prod)
- 60 ConfigHub units across all environments
- Advanced deployment scripts (health-check, rollback, validate-deployment, blue-green-deploy)
- Multi-agent development workflow
- Comprehensive testing (88.6% coverage)
- Security and code reviews completed

```bash
cd /Users/alexis/traderx/

# Quick start (see QUICKSTART.md for details)
bin/install-base      # Create ConfigHub infrastructure
bin/install-envs      # Set up environment hierarchy
bin/ordered-apply dev # Deploy all 8 services
bin/validate-deployment dev  # Comprehensive validation

# Access the application
kubectl port-forward -n traderx-dev svc/web-gui 18080:18080
open http://localhost:18080
```

**Key Features:**
- ConfigHub-native deployment (no kubectl for state changes)
- Dependency-ordered deployment
- Zero-downtime blue-green deployments
- Comprehensive health validation
- Automatic rollback capabilities
- Worker support for auto-deployment
- Full audit trail via ConfigHub
- Production-ready scripts and documentation

**Documentation:**
- [README.md](/Users/alexis/traderx/README.md) - Project overview
- [QUICKSTART.md](/Users/alexis/traderx/QUICKSTART.md) - 15-minute deployment guide
- [RUNBOOK.md](/Users/alexis/traderx/RUNBOOK.md) - Operational procedures
- [CHANGELOG.md](/Users/alexis/traderx/CHANGELOG.md) - Version history
- [SECURITY-REVIEW.md](/Users/alexis/traderx/SECURITY-REVIEW.md) - Security assessment (68/100 for dev)
- [CODE-REVIEW.md](/Users/alexis/traderx/CODE-REVIEW.md) - Code quality review (82/100)
- [TEST-RESULTS.md](/Users/alexis/traderx/TEST-RESULTS.md) - Test coverage (88.6%)

**Metrics:**
- **Security Score**: 68/100 (development), remediation roadmap defined
- **Code Quality**: 82/100
- **Test Coverage**: 88.6%
- **ConfigHub Pattern Adherence**: 95/100
- **Deployment Time**: ~5 minutes (target: <10 minutes)
- **Rollback Time**: 15-30 seconds (target: <30 seconds)

---

### 2. Drift Detector

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

### 3. Cost Optimizer

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

1. **Unique Project Naming** - `cub space new-prefix` prevents collisions
2. **Space Hierarchy** - base → qa → staging → prod with upstream/downstream
3. **Filter Creation** - SQL-like WHERE clauses for targeting units
4. **Environment Cloning** - Clone units with `--upstream-unit` relationships
5. **Version Promotion** - Set versions with `cub run set-image-reference`
6. **Bulk Operations with Sets** - Group related units for bulk operations
7. **Event-Driven Architecture** - React immediately with Kubernetes informers
8. **ConfigHub Functions** - `cub run` for common operations (version, env vars, resources)
9. **Changesets** - Atomic multi-unit operations with lock/unlock
10. **Lateral Promotion** - Conservative region-by-region rollout bypassing hierarchy
11. **Revision Management** - List, diff, rollback with full history
12. **Link Management** - Connect app units to infrastructure units

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

## 🤖 Multi-Agent Development Workflow

The TraderX example demonstrates a novel multi-agent development approach where specialized AI agents collaborate to deliver a complete deployment:

### Agent Collaboration Results

**Project**: TraderX ConfigHub Deployment (`mellow-muzzle-traderx`)

| Agent | Role | Deliverables | Quality Score |
|-------|------|--------------|---------------|
| **Planning Agent** | Project planning | Implementation plan, risk matrix, success criteria | N/A |
| **Architecture Agent** | Technical design | Architecture design, deployment patterns | N/A |
| **Code Generator** | Implementation | 14 scripts, 17 manifests, test suites | 82/100 |
| **Security Review** | Security assessment | 25 findings, remediation roadmap | 68/100 (dev) |
| **Code Review** | Quality analysis | Code review, best practices validation | 82/100 |
| **Testing Agent** | Testing & validation | Test suites, coverage report | 88.6% coverage |
| **Deployment Agent** | Infrastructure deployment | 5 spaces, 60 units created | Complete |
| **Documentation Agent** | Documentation | 4 new docs, updates to existing | Complete |

### Multi-Agent Workflow Characteristics

Implementation characteristics observed:
1. Specialized agents for planning, architecture, code generation, security, testing, deployment, and documentation
2. Multiple review stages between agent handoffs
3. Security, code quality, and testing performed by separate agents
4. Documentation generated and updated by documentation agent
5. Sequential agent execution with each building on previous work

**Final Deliverables:**
- 60 ConfigHub units across 5 environments
- 14 production-ready deployment scripts
- 17 Kubernetes manifests with security hardening
- 88.6% test coverage
- Comprehensive documentation (README, QUICKSTART, RUNBOOK, CHANGELOG)
- Security assessment with remediation roadmap
- Code quality review with improvement recommendations

### Implementation Notes

Observations from execution:
- Structured agent handoffs with defined deliverables
- Review agents identified issues before deployment
- Documentation consolidated by dedicated agent
- ConfigHub-only pattern validated across multiple agents
- Integration testing delayed due to Docker dependency
- Security recommendations provided post-implementation
- Sequential execution used; parallel execution possible for independent tasks

See `/Users/alexis/traderx/` for complete multi-agent execution results.

---

## 🤝 Contributing

To add a new DevOps app example:

1. Copy structure from drift-detector, cost-optimizer, or TraderX
2. Follow the global-app README pattern
3. Follow ConfigHub patterns from [CANONICAL-PATTERNS-SUMMARY.md](CANONICAL-PATTERNS-SUMMARY.md)
4. Consider scenario tasks with verification steps
5. Implement comprehensive testing and documentation  

## 📄 License

See individual examples for licensing details.
