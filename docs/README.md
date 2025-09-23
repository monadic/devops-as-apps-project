# DevOps as Apps Documentation Guide

## 📚 Documentation Structure

This guide helps you navigate our comprehensive documentation for the **Config as Data + DevOps as Apps** platform. Read documents in the suggested order for best understanding.

### The Core Insight:
**Configuration is data, not code. Data needs applications, not scripts.**

## 🎯 Quick Start Path

If you're new, read these in order:

1. **[DEVOPS-AS-APPS-MASTER-PLAN.md](DEVOPS-AS-APPS-MASTER-PLAN.md)** - Start here! Core vision and architecture
2. **[CANONICAL-PATTERNS-SUMMARY.md](CANONICAL-PATTERNS-SUMMARY.md)** - Essential patterns you MUST follow
3. **[CONFIGHUB-ACTUAL-FEATURES.md](CONFIGHUB-ACTUAL-FEATURES.md)** - What's real vs hallucinated in ConfigHub

## 📖 Complete Documentation Index

### 1. Foundation Documents (Read First)

#### **[CONFIG-AS-DATA-AND-DEVOPS-AS-APPS.md](CONFIG-AS-DATA-AND-DEVOPS-AS-APPS.md)**
**Purpose**: Understanding why Config as Data and DevOps as Apps are inseparable
**Key Insight**: They're the same breakthrough - data needs applications
**Must Read First**: This explains the unified concept

#### **[DEVOPS-AS-APPS-MASTER-PLAN.md](DEVOPS-AS-APPS-MASTER-PLAN.md)**
**Purpose**: Core architectural vision and implementation strategy
**Key Concepts**:
- Config as Data + Apps working together
- Persistent apps managing configuration data
- ConfigHub + SDK + Claude integration
- Dev Mode vs Enterprise Mode

**Must Understand**:
- Configuration is data that needs applications
- Why persistent apps beat ephemeral workflows
- How ConfigHub provides the data layer

#### **[CANONICAL-PATTERNS-SUMMARY.md](CANONICAL-PATTERNS-SUMMARY.md)**
**Purpose**: The EXACT patterns every app must follow
**Key Concepts**:
- `cub space new-prefix` for unique naming
- Environment hierarchy (base → qa → staging → prod)
- Push-upgrade pattern for propagation
- Event-driven architecture with informers

**Must Understand**:
- These patterns come from `/Users/alexisrichardson/examples-internal/global-app/`
- This is the CANONICAL reference - deviation causes problems

#### **[CONFIGHUB-ACTUAL-FEATURES.md](CONFIGHUB-ACTUAL-FEATURES.md)**
**Purpose**: What ConfigHub features actually exist vs hallucinations
**Key Concepts**:
- REAL: Spaces, Units, Sets, Filters, Push-upgrade
- FAKE: Variants API, Gates, Dependency graphs
- How to create variants (clone + edit)

**Must Understand**:
- Always verify features exist before using
- ConfigHub source: `/Users/alexisrichardson/github-repos/confighub/`

### 2. Implementation Guides

#### **[DEVOPS-AS-APPS-PLAN.md](DEVOPS-AS-APPS-PLAN.md)**
**Purpose**: Detailed implementation roadmap
**Key Concepts**:
- Phase 1: Build example apps
- Phase 2: Extract SDK
- Phase 3: Optional CRD
- Market positioning

**References**:
- Global-app: Example of ConfigHub patterns in production
- Helm platform: Shows Kubernetes integration patterns

#### **[CONFIGHUB-DEPLOYMENT-PATTERN.md](CONFIGHUB-DEPLOYMENT-PATTERN.md)**
**Purpose**: How to deploy apps through ConfigHub (not kubectl)
**Key Concepts**:
- `bin/install-base` creates ConfigHub structure
- `bin/install-envs` sets up hierarchy
- `bin/apply-all` deploys via ConfigHub

**Must Understand**:
- NEVER use `kubectl apply -f` directly
- Always deploy through ConfigHub units

#### **[TESTING-GUIDE.md](TESTING-GUIDE.md)**
**Purpose**: How to test DevOps apps
**Key Concepts**:
- Demo mode for local testing
- Integration tests with real ConfigHub
- Kind cluster for Kubernetes testing

### 3. Competitive & Market Documents

#### **[COMPETITIVE-ADVANTAGES.md](COMPETITIVE-ADVANTAGES.md)**
**Purpose**: How we beat competitors
**Key Comparisons**:
- vs Cased.com: Persistent > Ephemeral
- vs Flux/Argo: Complete reconciliation > Just Git→K8s
- ConfigHub features that enable superiority

#### **[GITOPS-AS-APPS-COMPARISON.md](GITOPS-AS-APPS-COMPARISON.md)**
**Purpose**: How we enhance/complete GitOps
**Key Concepts**:
- GitOps is ONE reconciliation loop, we generalize it
- Security/Cost/Compliance as reconciliation
- Reverse GitOps (Live → ConfigHub → Git)
- Selling to Flux/Argo users

**Must Understand**:
- We don't replace GitOps, we complete it
- Everything is a reconciliation loop

#### **[DEVOPS-AS-APPS-MARKET-STRATEGY.md](DEVOPS-AS-APPS-MARKET-STRATEGY.md)**
**Purpose**: Go-to-market strategy
**Key Markets**:
- Primary: GitOps users (Flux/Argo)
- Secondary: Platform engineers, Security teams
- Positioning: "Complete your GitOps"

### 4. Use Cases and Customer Examples

#### **[RBC-USE-CASE.md](RBC-USE-CASE.md)**
**Purpose**: Enterprise validation from Royal Bank of Canada
**Key Story**: 30 Grafana instances, 6 environments, 3-day investigation
**Solution**: Config as Data + Worker Applications
**Results**: Eliminated drift, reduced incidents, freed platform team

### 5. Specific Pattern Examples

#### **[DEPLOY-FROM-BRANCH-PATTERN.md](DEPLOY-FROM-BRANCH-PATTERN.md)**
**Purpose**: How we beat Cased's "killer branch deploy"
**Key Concepts**:
- Persistent spaces vs ephemeral environments
- Clone + edit for variants
- Canonical patterns in action

#### **[UPGRADE-MANAGER-USE-CASE.md](UPGRADE-MANAGER-USE-CASE.md)**
**Purpose**: Example of continuous upgrade management
**Key Concepts**:
- Better than Chkk's one-time agent
- Uses Sets and Filters
- Push-upgrade for propagation

## 🎯 The Unified Story

**Config as Data**: Configuration stored as structured, queryable data
**DevOps as Apps**: Modern applications managing that data
**Together**: The complete solution

RBC built this internally. We're productizing it for everyone.

## 🔗 Key External References

### ConfigHub Source Code
**Location**: `/Users/alexisrichardson/github-repos/confighub/`
**Why Important**: This is the source of truth for what features exist
**Key Files**:
- `internal/models/set.go` - Sets implementation
- `internal/models/filter.go` - Filters implementation
- `public/openapi/goclient-new/models.gen.go` - API types

### Global-App Reference Implementation
**Location**: `/Users/alexisrichardson/examples-internal/global-app/`
**Why Important**: This is the CANONICAL pattern we follow
**Key Files**:
- `bin/install-base` - How to create ConfigHub structure
- `bin/install-envs` - How to set up environment hierarchy
- `README.md` - Complete documentation of patterns

### Helm Platform Components
**Location**: `/Users/alexisrichardson/examples-internal/helm-platform-components/`
**Why Important**: Shows Kubernetes integration patterns
**Usage**: Reference for Kubernetes manifests and Helm charts

### Our SDK
**Location**: `/Users/alexisrichardson/github-repos/devops-sdk/`
**Why Important**: Reusable components for all DevOps apps
**Key Files**:
- `app.go` - Base app framework with informers
- `confighub.go` - ConfigHub client implementation
- `claude.go` - AI integration
- `kubernetes.go` - K8s utilities

### Example Apps
**Location**: `/Users/alexisrichardson/github-repos/devops-examples/`
**Current Apps**:
- `drift-detector/` - Fully implemented with Sets/Filters
- `cost-optimizer/` - AI-powered with dashboard

## 🚀 Getting Started Checklist

- [ ] Read MASTER-PLAN to understand vision
- [ ] Study CANONICAL-PATTERNS to learn required patterns
- [ ] Review ACTUAL-FEATURES to avoid hallucinations
- [ ] Check global-app source for pattern examples
- [ ] Look at drift-detector for implementation reference
- [ ] Understand ConfigHub's role (configuration management)
- [ ] Understand SDK's role (standardized app framework)
- [ ] Learn the reconciliation loop concept from GITOPS-AS-APPS

## 💡 Key Concepts to Master

1. **Persistent vs Ephemeral**: Our apps run continuously, not just when triggered
2. **Environment Hierarchy**: base → qa → staging → prod with inheritance
3. **Push-Upgrade**: Changes propagate automatically through environments
4. **Reconciliation Loops**: Every operational concern follows Desired → Actual → Reconcile
5. **Reverse GitOps**: Capturing live changes back to ConfigHub
6. **Sets and Filters**: Powerful grouping and targeting of resources
7. **Event-Driven**: Informers react immediately, no polling

## ❓ Common Questions

**Q: What's ConfigHub's role?**
A: ConfigHub is our configuration management system providing spaces, units, sets, filters, and push-upgrade capabilities. It's the source of truth for all configuration.

**Q: What's the SDK's role?**
A: The SDK provides a standardized framework for building DevOps apps with ConfigHub integration, Kubernetes informers, and Claude AI capabilities.

**Q: Why follow global-app patterns?**
A: Global-app is the proven, production-tested pattern for ConfigHub usage. It shows the canonical way to structure spaces, manage environments, and deploy applications.

**Q: How do we beat Cased.com?**
A: Persistent apps with continuous reconciliation beat ephemeral workflows. We maintain state, learn from history, and react immediately to changes.

**Q: How do we enhance GitOps?**
A: We generalize the GitOps reconciliation pattern to ALL operational concerns (security, cost, compliance), not just deployments. Plus bidirectional sync.

## 📝 Documentation Standards

When adding new documentation:
1. Follow the established patterns
2. Reference global-app for canonical examples
3. Verify ConfigHub features actually exist
4. Include practical examples with code
5. Show competitive advantages clearly
6. Explain both Dev and Enterprise modes

## 🎯 Next Steps

After reading the documentation:
1. Clone the example apps (drift-detector, cost-optimizer)
2. Run demo mode to see them in action
3. Deploy using canonical patterns
4. Build your own DevOps app following the patterns

---

**Remember**: When in doubt, follow the global-app patterns exactly. They are canonical for a reason.