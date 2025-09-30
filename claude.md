# Claude Code Context for DevOps as Apps Project

## Project Overview
This is a "DevOps as Apps" platform that competes with Cased.com by using persistent Kubernetes applications (not ephemeral workflows) with ConfigHub + SDK + Claude as the core platform. The goal is to build DevOps automation tools as long-running apps.

## Critical Information
This project uses ConfigHub as its configuration management backend. Many features that might seem obvious DO NOT EXIST in ConfigHub. This file serves as your reference for what's real.

## 🚀 SETUP REQUIREMENTS (DO FIRST)

Before working on this project, Claude must complete these steps:

### 1. GitHub Authentication & ConfigHub Code Review
✅ **COMPLETED**: Successfully accessed and cloned latest ConfigHub repositories:

**Local cloned repositories** (always use these for latest code):
- **Main repo**: `/Users/alexisrichardson/github-repos/confighub-latest/`
- **Documentation**: `/Users/alexisrichardson/github-repos/confighub-docs/`
- **Examples**: `/Users/alexisrichardson/github-repos/confighub-examples/`

**GitHub URLs** (accessible via git, but may be private via web):
- **Main repo**: https://github.com/confighubai/confighub
- **Documentation**: https://github.com/confighubai/docs
- **Internal examples**: https://github.com/confighubai/examples-internal

**Key Findings from Latest Code Review**:
- ✅ **Sets confirmed**: `/confighub-latest/internal/models/set.go` - Real feature for grouping units
- ✅ **Filters confirmed**: `/confighub-latest/internal/models/filter.go` - Real feature with WHERE clauses, WhereData, ResourceType
- ✅ **CLI commands**: 150+ commands in `/confighub-latest/public/cmd/cub/` including all canonical patterns
- ✅ **space new-prefix**: Confirmed in `space_new_prefix.go`
- ✅ **unit push-upgrade**: Confirmed in `unit_push_upgrade.go`
- ✅ **Examples**: Updated global-app in `/confighub-examples/global-app/` (canonical patterns)

### 2. ConfigHub Authentication & Upgrade
```bash
# Authenticate with ConfigHub
cub auth login

# Upgrade to latest cub CLI version
cub upgrade
```

### 3. Claude API Key Setup
Always prompt the user for their Claude API key:
```bash
# Required for AI-powered DevOps apps
export CLAUDE_API_KEY="your-claude-api-key-here"
```

### 4. Project Documentation Review
Review ALL documentation and files in this project folder and subfolders:
- `/Users/alexisrichardson/github-repos/devops-as-apps-project/`
- All `docs/` files and subdirectories
- All example implementations and patterns

**CRITICAL**: Complete steps 1-4 before proceeding with any development work.

## 🤖 IMPORTANT: Claude AI Integration (Updated 2025-09-23)

**All DevOps examples now require Claude AI by default** for intelligent analysis:

### Setup Requirements:
1. **Get Claude API Key**: https://console.anthropic.com/settings/keys
2. **Set in environment**: Add to `.env` file or export `CLAUDE_API_KEY=sk-ant-...`
3. **Debug logging enabled by default**: Set `CLAUDE_DEBUG_LOGGING=false` to disable

### Running Examples:
```bash
# All examples now have run.sh scripts that handle Claude setup
cd any-example/
./run.sh  # Prompts for API key if not set

# To disable Claude temporarily:
ENABLE_CLAUDE=false ./run.sh
```

### What Claude Provides:
- **Cost Optimizer**: Intelligent cost recommendations with risk assessment
- **Drift Detector**: Root cause analysis and fix recommendations
- **All Future Apps**: Must include Claude integration by default

### Standard Pattern (MUST FOLLOW):
- Claude enabled by default
- Debug logging enabled by default
- Easy disable option via ENABLE_CLAUDE=false
- Fallback to basic analysis when disabled

## 🚨 CRITICAL: ConfigHub-Only Commands (NO kubectl!)

### **MANDATORY REQUIREMENT**: ALL DevOps apps MUST use ConfigHub commands EXCLUSIVELY

**ConfigHub is the SINGLE SOURCE OF TRUTH** - All configuration changes, drift corrections, and deployments MUST go through ConfigHub. Direct kubectl commands are PROHIBITED in production code.

### Drift Correction Pattern (REQUIRED)
```bash
# ✅ CORRECT - Use ConfigHub to fix drift
cub unit update backend-api-unit --patch --data '{"spec":{"replicas":3}}'
cub unit apply backend-api-unit --space drift-test-demo

# ❌ WRONG - NEVER use kubectl for corrections
kubectl scale deployment backend-api --replicas=3  # PROHIBITED!
```

### Self-Deployment Pattern (REQUIRED)
```bash
# ✅ CORRECT - ConfigHub-driven deployment
bin/install-base      # Creates units in ConfigHub
bin/install-envs      # Sets up env hierarchy
bin/apply-all dev     # Deploys via ConfigHub

# ❌ WRONG - Direct Kubernetes deployment
kubectl apply -f k8s/  # PROHIBITED!
```

### Why ConfigHub Commands Only:
1. **Single Source of Truth**: ConfigHub maintains desired state
2. **Audit Trail**: All changes tracked in ConfigHub history
3. **Environment Hierarchy**: Changes propagate via push-upgrade
4. **Drift Prevention**: ConfigHub state always matches intent
5. **GitOps Ready**: ConfigHub can trigger Git commits for Flux/Argo

### Required in All Apps:
- Drift Detector: MUST recommend `cub unit` commands only
- Cost Optimizer: MUST apply optimizations via ConfigHub
- All Future Apps: MUST interact exclusively through ConfigHub API

See `docs/CONFIGHHUB-DEPLOYMENT-PATTERN.md` for implementation details.

## How to Continue This Project

### Step 1: Read Canonical Global-App Implementation (CRITICAL)
- **Global-app (Latest)**: `/Users/alexisrichardson/github-repos/confighub-examples/global-app/`
  - **Backup reference**: `/Users/alexisrichardson/examples-internal/global-app/`
  - This is the CANONICAL reference for all ConfigHub patterns
  - Study `bin/install-base`, `bin/install-envs`, `bin/new-app-env`
  - Uses `cub space new-prefix` for unique naming
  - Full environment hierarchy: base → qa → staging → prod

### Step 2: Understand Our Competitive Advantages
| Feature | Cased (Workflows) | Our Apps (ConfigHub + SDK) |
|---------|------------------|---------------------------|
| **Execution** | Ephemeral, exits | Persistent, continuous with informers |
| **Environment Cloning** | "Killer branch deploy" | Full hierarchy with `--upstream-unit` |
| **Promotion** | Manual copy | Push-upgrade with `BulkPatchUnits` |
| **Drift Correction** | None | Auto-correction with Sets/Filters |
| **State Management** | Stateless | Stateful with ConfigHub tracking |
| **Cost Analysis** | One-time script | Continuous AI optimization |
| **Bulk Operations** | Single workflow | Sets/Filters across environments |

### Step 3: Read ConfigHub Source Code
- **ConfigHub repo (Latest)**: `/Users/alexisrichardson/github-repos/confighub-latest/`
- **Backup repo**: `/Users/alexisrichardson/github-repos/confighub/`
- **Key files to read**:
  - `internal/models/set.go` - Understand Sets (REAL feature)
  - `internal/models/filter.go` - Understand Filters (REAL feature with WHERE clauses)
  - `internal/models/unit.go` - Unit operations and relationships
  - `public/cmd/cub/` - All 150+ CLI commands
  - `public/openapi/goclient-new/models.gen.go` - API types
- **Search patterns**: `grep -r "BulkPatch" /Users/alexisrichardson/github-repos/confighub-latest/`

### Step 4: Canonical ConfigHub Commands (from global-app)
```bash
# Create unique project prefix (ALWAYS do this)
prefix=$(cub space new-prefix)  # e.g., "chubby-paws"

# Create filters for targeting
cub filter create all Unit --where-field "Space.Labels.project = '$project'"
cub filter create app Unit --where-field "Labels.type='app'"
cub filter create infra Unit --where-field "Labels.type='infra'"

# Clone with upstream relationships
cub unit create --dest-space $project-qa --space $project-base \
  --filter $project/app --label targetable=true

# Set versions (canonical promotion)
cub run set-image-reference --container-name frontend --image-reference :1.1.3 \
  --space $(bin/proj)-qa

# Promote with push-upgrade
cub unit update --patch --upgrade --space $(bin/proj)-staging

# View hierarchy
cub unit tree --node=space --filter $(bin/proj)/app --space '*'
```

### Step 5: Read Our Project Documentation
- **Master plan**: `docs/DEVOPS-AS-APPS-MASTER-PLAN.md` - Core architecture
- **Implementation plan**: `docs/DEVOPS-AS-APPS-PLAN.md` - Detailed steps
- **API reference**: `docs/CONFIGHUB-ACTUAL-FEATURES.md` - What's real vs hallucinated
- **Development guide**: `DEVELOPMENT.md` - Multi-repo setup

### Step 6: Review Current Implementation
- **SDK**: `/Users/alexisrichardson/github-repos/devops-sdk/`
  - `confighub.go` - Real ConfigHub client with Sets, Filters, bulk ops
  - `app.go` - Base DevOps app framework
  - `claude.go` - Claude integration
  - `kubernetes.go` - K8s utilities
- **Drift Detector**: `/Users/alexisrichardson/github-repos/devops-examples/drift-detector/`
  - `main.go` - Full implementation using Sets/Filters/informers
  - `main_test.go` - Comprehensive tests
  - `integration_test.go` - Real ConfigHub API tests

## REAL ConfigHub Features ✅
These are the ONLY features that actually exist:
- **Spaces** - Workspaces for configuration organization
- **Units** - Individual configuration items
- **Sets** - Groups of units for bulk operations
- **Filters** - WHERE clause queries
- **Upstream/Downstream** - Inheritance relationships via UpstreamUnitID
- **Push-upgrade** - Propagate changes with BulkPatchUnits(Upgrade: true)
- **Apply/Destroy** - Deploy and remove configurations
- **Live State** - Read-only deployment status


## Canonical Patterns (from global-app)

### Environment Hierarchy Creation
```bash
# ✅ CORRECT - Canonical pattern from global-app
bin/install-base      # Creates unique prefix with cub space new-prefix
bin/install-envs      # Creates base → qa → staging → prod hierarchy
bin/new-app-env qa base qa  # Clone from base to qa with upstream

# ❌ WRONG - Don't hardcode prefixes or skip hierarchy
kubectl apply -f deployment.yaml  # Bypasses ConfigHub entirely
```

### Multi-Environment with Upstream/Downstream
```go
// ✅ CORRECT - Use canonical upstream/downstream pattern
unit := CreateUnit(Unit{
    UpstreamUnitID: &baseUnitID,
    Space: fmt.Sprintf("%s-qa", projectPrefix),
})

// Push-upgrade to propagate changes
BulkPatchUnits(BulkPatchParams{
    Where: fmt.Sprintf("UpstreamUnitID = '%s'", baseUnitID),
    Upgrade: true,
})

// ❌ WRONG - Variants don't exist
variant := GetVariant("aws-variant")
```

### Bulk Operations
```go
// ✅ CORRECT - Use Sets and Filters
set := CreateSet("critical-services")
filter := CreateFilter(CreateFilterRequest{
    Where: fmt.Sprintf("SetID = '%s'", set.SetID),
})

// ❌ WRONG - UpgradeSet doesn't exist
UpgradeSet(setID)
```

### Event-Driven Architecture (Better than Cased workflows)
```go
// ✅ CORRECT - Use Kubernetes informers (continuous reconciliation)
app.RunWithInformers(func() error {
    // React to actual changes immediately
    detectDrift()
    return nil
})

// ❌ WRONG - Polling like Cased workflows
for {
    detectDrift()
    time.Sleep(5 * time.Minute)  // Wastes resources, slow to react
}
```

### Why Our Apps Beat Cased Workflows:
1. **Persistent**: Apps run continuously, not just when triggered
2. **Event-driven**: Informers react immediately to changes
3. **Stateful**: ConfigHub tracks all state and history
4. **Hierarchical**: Full environment promotion via push-upgrade
5. **AI-powered**: Claude integration for intelligent decisions
6. **Bulk operations**: Sets and Filters for cross-environment ops

## Project Rules
1. ALWAYS check docs/CONFIGHUB-ACTUAL-FEATURES.md before using any ConfigHub API
2. Use Go, not Python
3. Use Kubernetes informers, not polling
4. Prefer Sets and Filters for bulk operations
5. Use push-upgrade pattern for propagation
6. Dev Mode: ConfigHub → Kubernetes (direct)
7. Enterprise Mode: ConfigHub → Git → Flux/Argo → Kubernetes

## Canonical Deployment Pattern (MUST FOLLOW)

Every DevOps app MUST deploy itself through ConfigHub:

```bash
# Step 1: Create ConfigHub structure (canonical pattern)
cd /Users/alexisrichardson/github-repos/devops-examples/{app-name}
bin/install-base      # Creates unique prefix, spaces, filters, base units
bin/install-envs      # Creates environment hierarchy

# Step 2: Deploy via ConfigHub
bin/apply-all dev     # Deploy to dev environment
bin/promote dev staging  # Promote to staging
bin/apply-all staging    # Apply staging

# NEVER do this:
kubectl apply -f k8s/  # Wrong! Bypasses ConfigHub
```

## Testing Commands & Principles

### **CRITICAL Testing Requirement: Validate ConfigHub-Only Commands**
All tests MUST verify that apps use ConfigHub commands exclusively:
```bash
# Required test patterns in all test suites
test_api "Uses cub commands" '.corrections[0].command | contains("cub unit")' "true"
test_api "NO kubectl commands" '.corrections[0].command | contains("kubectl")' "false"
```

### **Standard Testing Protocol**
Always follow this 2-step testing approach:

#### **Step 1: Local Tests First**
```bash
# Build
go build ./...

# Unit tests (mock data, no external dependencies)
go test -v

# Demo mode (simulated workflow)
./drift-detector demo

# Format and lint
go fmt ./...
golangci-lint run  # if available
```

#### **Step 2: Real Integration Tests**
```bash
# Set up real ConfigHub connection
export CUB_TOKEN="your-confighub-token"
export CUB_API_URL="https://confighub.com/api/v1"

# Integration tests with real ConfigHub
go test -tags=integration -v

# Set up local Kind cluster
kind create cluster --name devops-test

# Deploy and test with real infrastructure
kubectl apply -f k8s/
./drift-detector  # runs against real ConfigHub + Kind cluster
```

### **Testing Environment Setup**
```bash
# Kind cluster for testing
kind create cluster --name devops-test
kubectl cluster-info --context kind-devops-test

# ConfigHub credentials (get from confighub.com)
export CUB_TOKEN="your-token-here"
export CUB_API_URL="https://confighub.com/api/v1"

# Optional: Claude for AI features
export CLAUDE_API_KEY="your-claude-key"
```

## Current Status (Updated with Canonical Patterns)
✅ **Completed**:
- SDK with real ConfigHub API (Sets, Filters, Push-upgrade)
- Drift detector using event-driven informers (not polling)
- Cost optimizer with AI analysis and web dashboard (:8081)
- Canonical global-app deployment pattern implemented
- Comprehensive testing (unit + integration + demo modes)
- All 3 repos created and working

### Apps Completed with Canonical Patterns:
1. **drift-detector**: Full Sets/Filters, event-driven, ConfigHub deployment
2. **cost-optimizer**: AI-powered, dashboard, Sets for grouping, auto-apply

### Competitive Advantages Demonstrated:
- **vs Cased**: Persistent apps beat ephemeral workflows
- **Environment cloning**: Better than "killer branch deploy"
- **Push-upgrade**: Automatic propagation beats manual copy
- **AI integration**: Claude provides intelligent decisions
- **Web dashboards**: Real-time visibility

🔄 **Next Steps**:
- Complete remaining 4 DevOps apps with canonical patterns
- Add Enterprise Mode (ConfigHub → Git → Flux/Argo)
- Document more competitive advantages

## Important File Locations

### Project Structure
```
/Users/alexisrichardson/github-repos/
├── confighub/                    # ConfigHub source (READ-ONLY)
├── devops-as-apps-project/       # Planning and docs
│   ├── docs/                     # All planning documents
│   ├── .claude-code/            # Claude Code configuration
│   └── CLAUDE.md                # This file
├── devops-sdk/                   # Reusable SDK
│   ├── confighub.go             # Real ConfigHub client
│   ├── app.go                   # Base app framework
│   └── go.mod                   # Module: github.com/monadic/devops-sdk
└── devops-examples/              # Example implementations
    └── drift-detector/           # Working drift detector
        ├── main.go              # Uses Sets/Filters/informers
        ├── main_test.go         # Unit tests
        └── integration_test.go  # Real API tests
```

### GitHub Repositories
- https://github.com/monadic/devops-as-apps-project
- https://github.com/monadic/devops-sdk
- https://github.com/monadic/devops-examples

### Reference Implementations
- `/Users/alexisrichardson/examples-internal/global-app/` - ConfigHub usage patterns
- Look for `bin/install-base` and `bin/install-envs` scripts

### Testing Commands
```bash
# Build everything
cd /Users/alexisrichardson/github-repos/devops-sdk && go build ./...
cd /Users/alexisrichardson/github-repos/devops-examples/drift-detector && go build .

# Run tests
go test -v                           # Unit tests
go test -tags=integration -v         # Integration tests (needs CUB_TOKEN)

# Test with real ConfigHub
export CUB_TOKEN="your-token"
export CUB_API_URL="https://api.confighub.com/v1"
go test -tags=integration -v
```

## Key Documents (ALWAYS READ THESE FIRST)
- `docs/CONFIGHUB-ACTUAL-FEATURES.md` - API reference (CRITICAL)
- `docs/DEVOPS-AS-APPS-MASTER-PLAN.md` - Master implementation plan
- `docs/DEVOPS-AS-APPS-PLAN.md` - Detailed guide
- `/Users/alexisrichardson/examples-internal/global-app/README.md` - Reference patterns

## Context from Previous Sessions
This project started as analysis of Cased.com, then evolved into building a better competitor using ConfigHub. The key insight was that persistent DevOps applications are better than ephemeral workflows. We discovered many ConfigHub features were hallucinated and had to rewrite everything to use only real APIs from the source code.

The drift-detector is now a fully working example that demonstrates all the key patterns: Sets for grouping, Filters for targeting, push-upgrade for propagation, and informers for event-driven architecture.

## Remember
If you're about to use a ConfigHub feature, VERIFY it exists in `docs/CONFIGHUB-ACTUAL-FEATURES.md` first. When in doubt, use only the confirmed operations: CreateSpace, CreateUnit, ApplyUnit, DestroyUnit, CreateSet, GetSet, CreateFilter, BulkPatchUnits, BulkApplyUnits.