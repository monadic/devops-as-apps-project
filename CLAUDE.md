# Claude Code Context for DevOps as Apps Project

## Project Overview
This is a "DevOps as Apps" experiment that competes with agentic workflows by using persistent Kubernetes applications instead, with ConfigHub + SDK + Claude as the core platform. The goal is to build DevOps automation tools as long-running apps.

## Critical Information
This project uses ConfigHub as its configuration management backend. Many features that might seem obvious DO NOT EXIST in ConfigHub. This file serves as your reference for what's real.

## Governing Principles for Claude Sessions

### 1. Communication Standards
Use dry technical language only. Prohibited:
- Emotional language, speculation, hallucination
- Marketing terminology, jargon, hype
- Unnecessary excitement, exclamation marks
- Flattering or enthusiastic language
- Unjustified time estimates or product comparisons without explicit approval
- Editorial comments in brackets
- Phrases like "see how simple it is" at section endings

### 2. Agentic Development Methodology
Follow current best practices for agentic software development. At session start, state which methodologies are being followed and request additional guidance if needed.

### 3. Verification Protocol
Review all work at least once per task for:
- Hallucinated content
- Unchecked assumptions
- Unverified ConfigHub features

Label any uncertain content explicitly. Confirm that work is based on:
- Actual ConfigHub codebase and documentation
- Information provided by the user to this project
- Verified sources only

**Config File Validation (MANDATORY):**

Before committing any YAML or JSON configuration files, MUST validate:

```bash
# YAML syntax validation
python3 -c 'import yaml, sys; yaml.safe_load(open("file.yaml"))' || echo "Invalid YAML"

# JSON syntax validation
jq empty file.json || echo "Invalid JSON"

# Kubernetes YAML validation
- Check apiVersion and kind present
- Check metadata.name present
- Check spec section appropriate for resource type
- Verify resource limits defined
- Verify security contexts present
- Verify health probes for deployments

# ConfigHub unit validation
- Verify Labels follow project conventions
- Verify Annotations include Description
- Verify Layer label for deployment ordering
```

Never commit invalid or unvalidated configuration files.

## ConfigHub CLI Mental Model (MANDATORY)

**Before writing ANY cub command, understand this model:**

### Command Structure
```bash
cub <entity> <verb> [unit-name] [flags]
```

### Three Types of Flags

**1. Mode Flags** - Change command behavior, REQUIRE companions
- `--patch` requires one of: `--from-stdin`, `--filename`, `--restore`, `--upgrade`, `--merge-source`, `--label`, `--delete-gate`, `--destroy-gate`, or `--changeset`
- `--upgrade` requires: `--patch`
- `--from-stdin` requires: data piped via stdin

**2. Required Flags** - Must be present
- `--space <space>` - Required for most unit operations
- `--where <clause>` - Required for bulk operations with filter targeting

**3. Parameter Flags** - Optional modifiers
- `--label key=value`
- `--timeout 5m`
- `--format json`

### How to Read Help Output (MANDATORY before using any command)

```bash
CONFIGHUB_AGENT=1 cub <entity> <verb> --help
```

Help output shows:
- **Required flags** (marked as required)
- **Flag combinations** (what companions mode flags need)
- **Examples** (copy these patterns exactly)
- **WHERE clause grammar** (for --where flag)

### Common Mistakes to Avoid

❌ **Using mode flags without required companions:**
```bash
cub unit update myunit --patch '{"spec":{}}' # WRONG: --patch needs companion
cub unit update myunit --patch --data '{}' # WRONG: --data is not a valid companion
```

✅ **Correct patterns:**
```bash
# Monolithic Data update (replace entire config)
echo '{"spec":{}}' | cub unit update myunit --from-stdin --space dev
cub unit update myunit --filename newdata.yaml --space dev

# Fine-grained Data update (specific fields)
cub function do --space dev --where "Slug = 'myunit'" set-replicas 3

# Metadata update (labels, annotations)
cub unit update myunit --patch --label version=2.0 --space dev

# Push-upgrade (propagate changes from upstream)
cub unit update --patch --upgrade --space staging
```

## MANDATORY: Before Using ANY cub Command

**BLOCKING REQUIREMENT** - Do these steps EVERY TIME before writing a cub command:

### Step 1: Read Help (MANDATORY)
```bash
CONFIGHUB_AGENT=1 cub <entity> <verb> --help
```

Example for unit update:
```bash
CONFIGHUB_AGENT=1 cub unit update --help
```

### Step 2: Identify Required Flags and Companions

From help output, note:
- What flags are required? (usually `--space`)
- Is this a mode flag? (`--patch`, `--upgrade`, etc.)
- What companions does this mode flag require?
- Are there examples? (copy the pattern exactly)

### Step 3: Test in Throwaway Space First

**NEVER commit untested cub commands.** Test first:

```bash
# Create test space
TEST_SPACE="test$RANDOM"
cub space create $TEST_SPACE

# Test your command
cub unit create myunit --space $TEST_SPACE data.yaml

# If it works, adapt for real use
# If it fails, read error and fix

# Cleanup
cub space delete $TEST_SPACE
```

### Step 4: Use test-lib.sh Functions

For scripts, use standard testing functions:
```bash
source "$ROOTDIR/test/scripts/test-lib.sh"

createSpace "$SPACE"
# ... your commands ...
verifyEntityWithinSpaceExists "unit" "$UNIT" "$SPACE"
```

### 4. Comprehensive Testing Requirements

Every project must include a `test/` folder with comprehensive test coverage:

**Required Test Types:**
- Unit tests for all components
- ConfigHub API integration tests
- CLI command tests (cub)
- SDK usage tests
- Worker connection and apply verification tests
- GUI behavior tests
- ASCII table and drawing output validation
- End-to-end workflow tests
- Regression tests
- User acceptance tests for applications and flows

**Test Documentation:**
- All tests clearly documented in test folder README
- Claude agent testing strategies documented in .md files
- Test execution procedures and expected results
- Coverage reports and quality metrics

**Session Startup Protocol:**
At the beginning of each new session, Claude must:
1. Identify all test suites in the project
2. Check infrastructure requirements (see below)
3. Offer infrastructure setup or test validation options
4. Report test results before proceeding with new work
5. Flag any failing tests for resolution

**Infrastructure Requirements for Testing:**

Tests have different infrastructure dependencies:

**Unit Tests** (No infrastructure required):
- Script syntax validation
- YAML manifest validation
- Code quality checks (include code review + config review)
- Run time: < 30 seconds

**Integration Tests** (Infrastructure required):
- ConfigHub authentication (`cub auth login`)
- ConfigHub spaces and units (`bin/install-base`, `bin/install-envs`)
- Kubernetes cluster (`kind create cluster` or existing)
- ConfigHub worker (`bin/setup-worker dev`)
- Run time: 2-5 minutes

**End-to-End Tests** (Full deployment required):
- All integration test requirements
- Deployed application (`bin/ordered-apply dev`)
- Run time: 5-10 minutes

**Infrastructure Setup Protocol:**

Before running tests, Claude must:
1. Check if infrastructure exists
2. Offer setup options:
   - Option A: Set up infrastructure first, then run tests
   - Option B: Run unit tests only (no infrastructure)
   - Option C: Skip tests and proceed with work
3. If setup requested, execute in order:
   ```bash
   # ConfigHub setup
   cub auth login
   bin/install-base
   bin/install-envs

   # Kubernetes setup (if cluster available)
   bin/setup-worker dev
   bin/ordered-apply dev

   # Then run tests
   ./test/run-all-tests.sh
   ```

**ConfigHub Standard Test Infrastructure:**

All projects MUST use ConfigHub standard testing conventions:

1. **Test Library** (`test/scripts/test-lib.sh`):
   - Copy from: https://github.com/confighubai/confighub/blob/main/test/scripts/test-lib.sh
   - Provides: `createSpace`, `verifyEntityWithinSpaceExists`, `checkEntityWithinSpaceListLength`
   - Usage: `source "$ROOTDIR/test/scripts/test-lib.sh"`

2. **Test Data** (`test-data/`):
   - `metadata.json` - Default labels/annotations for units
   - `space-metadata.json` - Default space metadata
   - YAML fixtures for each service/component
   - Reference: https://github.com/confighubai/confighub/tree/main/test-data

3. **YAML Validation** (REQUIRED for all config files):
   ```bash
   # Validate syntax
   python3 -c 'import yaml, sys; yaml.safe_load(sys.stdin)' < file.yaml

   # Validate Kubernetes compliance
   - apiVersion and kind present
   - metadata.name present
   - Valid namespace references

   # Validate best practices
   - Resource limits and requests
   - Security contexts (runAsNonRoot, readOnlyRootFilesystem)
   - Health probes (liveness and readiness)
   - Required labels (app.kubernetes.io/name, app.kubernetes.io/part-of)
   ```

4. **Test Script Standards**:
   - Use `#!/bin/bash -x` and `set -e`
   - Source test-lib.sh from ROOTDIR
   - Use random space names for isolation: `SPACE="test$RANDOM"`
   - Implement cleanup with trap handlers
   - Save output to `$OUTDIR` for debugging

**Infrastructure Check Commands:**
```bash
# Check ConfigHub auth
cub auth status &>/dev/null

# Check if project initialized
[ -f ".cub-project" ]

# Check Kubernetes cluster
kubectl cluster-info &>/dev/null

# Check if worker running
kubectl get pods -n confighub -l app=confighub-worker &>/dev/null
```

**Test Folder Structure:**
```
test/
├── README.md                    # Test documentation
├── unit/                        # Unit tests
│   ├── components/
│   ├── confighub-api/
│   ├── cli/
│   ├── sdk/
│   └── workers/
├── integration/                 # Integration tests
│   ├── worker-apply/
│   └── api-integration/
├── ui/                          # UI and output tests
│   ├── gui/
│   ├── ascii-tables/
│   └── dashboards/
├── e2e/                         # End-to-end tests
├── regression/                  # Regression tests
├── acceptance/                  # User acceptance tests
└── strategies/                  # Claude agent test strategies
    ├── TESTING-STRATEGY.md
    └── COVERAGE-REQUIREMENTS.md
```

### Mini TCK (MANDATORY - Run Before Any Development)

**Location**: `/Users/alexis/Public/github-repos/devops-sdk/test-confighub-k8s`

**BLOCKING REQUIREMENT**: Run this BEFORE starting any development work.

**Usage:**
```bash
cd /Users/alexis/Public/github-repos/devops-sdk
./test-confighub-k8s
```

**Expected Results:**
- Exit code 0 (success)
- All basic operations pass (space create, unit create, apply, destroy)
- ConfigHub API connection verified
- Environment correctly configured

**If Mini TCK fails:**
1. Do NOT proceed with development
2. Fix authentication issues (`cub auth login`)
3. Check API URL configuration
4. Verify network connectivity
5. Re-run Mini TCK until it passes

**Documentation**: See `devops-sdk/TCK.md`

## 🚀 SETUP REQUIREMENTS (DO FIRST)

Before working on this project, Claude must complete these steps:

### 1. GitHub Authentication & ConfigHub Code Review
✅ **COMPLETED**: Successfully accessed and cloned latest ConfigHub repositories:

**Local cloned repositories** (always use these for latest code):
- **Main repo**: `/Users/alexis/Public/github-repos/confighub-latest/`
- **Documentation**: `/Users/alexis/Public/github-repos/confighub-docs/`
- **Examples**: `/Users/alexis/Public/github-repos/confighub-examples/`

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
- `/Users/alexis/Public/github-repos/devops-as-apps-project/`
- All `docs/` files and subdirectories
- All example implementations and patterns

### 5. Understand ConfigHub CLI Model (MANDATORY)

Before proceeding with development, understand the CLI mental model:

**Step 5.1: Read Overview Help**
```bash
CONFIGHUB_AGENT=1 cub --help-overview
```

This shows:
- All entities (space, unit, filter, set, link, changeset, target, worker)
- All verbs per entity (create, update, apply, destroy, list, get)
- WHERE clause EBNF grammar
- Common patterns and examples

**Step 5.2: Read CLI Mental Model Section**

Read the "ConfigHub CLI Mental Model" section in this document (above).

Understand:
- Command structure: `cub <entity> <verb> [unit-name] [flags]`
- Mode flags (require companions): `--patch`, `--upgrade`, `--from-stdin`
- Required flags: `--space`, `--where`
- Parameter flags: `--label`, `--timeout`, `--format`

**Step 5.3: Test Basic Commands**

Test in throwaway space:
```bash
TEST_SPACE="test$RANDOM"
cub space create $TEST_SPACE

# Test unit create
echo "test: data" | cub unit create test-unit --from-stdin --space $TEST_SPACE

# Test unit get
cub unit get test-unit --space $TEST_SPACE

# Test unit list
cub unit list --space $TEST_SPACE

# Cleanup
cub space delete $TEST_SPACE
```

**Step 5.4: Run Mini TCK**

Verify setup is correct:
```bash
cd /Users/alexis/Public/github-repos/devops-sdk
./test-confighub-k8s
```

**Do NOT proceed until all 4 sub-steps complete successfully.**

**CRITICAL**: Complete steps 1-5 before proceeding with any development work.

## 🤖 IMPORTANT: Claude AI Integration (Updated 2025-10-09)

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
echo '{"spec":{"replicas":3}}' | cub unit update backend-api-unit --from-stdin --space drift-test-demo
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
- **Global-app (Latest)**: `/Users/alexis/Public/github-repos/confighub-examples/global-app/`
  - **Backup reference**: `/Users/alexis/Public/github-repos/confighub-examples/global-app/`
  - This is the CANONICAL reference for all ConfigHub patterns
  - Study `bin/install-base`, `bin/install-envs`, `bin/new-app-env`
  - Uses `cub space new-prefix` for unique naming
  - Full environment hierarchy: base → qa → staging → prod

### Step 2: DevOps Apps Advantages
| Feature | Agent Workflows | Our Apps (ConfigHub + SDK) |
|---------|------------------|---------------------------|
| **Execution** | Ephemeral, exits | Persistent, continuous with informers |
| **Environment Cloning** | "Killer branch deploy" | Full hierarchy with `--upstream-unit` |
| **Promotion** | Manual copy | Push-upgrade with `BulkPatchUnits` |
| **Drift Correction** | None | Auto-correction with Sets/Filters |
| **State Management** | Stateless | Stateful with ConfigHub tracking |
| **Cost Analysis** | One-time script | Continuous AI optimization |
| **Bulk Operations** | Single workflow | Sets/Filters across environments |

### Step 3: Read ConfigHub Source Code
- **ConfigHub repo (Latest)**: `/Users/alexis/Public/github-repos/confighub-latest/`
- **Backup repo**: `/Users/alexis/Public/github-repos/confighub/`
- **Key files to read**:
  - `internal/models/set.go` - Understand Sets (REAL feature)
  - `internal/models/filter.go` - Understand Filters (REAL feature with WHERE clauses)
  - `internal/models/unit.go` - Unit operations and relationships
  - `public/cmd/cub/` - All 150+ CLI commands
  - `public/openapi/goclient-new/models.gen.go` - API types
- **Search patterns**: `grep -r "BulkPatch" /Users/alexis/Public/github-repos/confighub-latest/`

### Step 4: Canonical ConfigHub Commands (from global-app)
```bash
# Create unique project prefix (ALWAYS do this)
prefix=$(cub space new-prefix)  # e.g., "chubby-paws"

# Create filters for targeting
cub filter create all Unit --where-field "Space.Labels.project = '$project'"
cub filter create app Unit --where-field "Labels.type='app'"
cub filter create infra Unit --where-field "Labels.type='infra'"

# Unit creation (current CLI syntax)
cub unit create --space my-space my-unit k8s/my-unit.yaml

# Create with upstream/downstream relationships (use --upstream-space notation)
cub unit create reference-data \
  --space dev \
  --upstream-space base \
  --upstream-unit reference-data \
  --data k8s/reference-data.yaml

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
- **SDK**: `/Users/alexis/Public/github-repos/devops-sdk/`
  - `confighub.go` - Real ConfigHub client with Sets, Filters, bulk ops
  - `app.go` - Base DevOps app framework
  - `claude.go` - Claude integration
  - `kubernetes.go` - K8s utilities
- **Drift Detector**: `/Users/alexis/Public/github-repos/devops-examples/drift-detector/`
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

// NOTE: Variants exist as a CONCEPT (different customizations via upstream/downstream)
// Example: US variant (3 replicas), EU variant (5 replicas), Asia variant (2 replicas)
// All inherit from base unit but with different customizations

// ❌ WRONG - No GetVariant() API exists (variants are units with UpstreamUnitID)
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
cd /Users/alexis/Public/github-repos/devops-examples/{app-name}
bin/install-base      # Creates unique prefix, spaces, filters, base units
bin/install-envs      # Creates environment hierarchy

# Step 2: Deploy via ConfigHub
bin/apply-all dev     # Deploy to dev environment
bin/promote dev staging  # Promote to staging
bin/apply-all staging    # Apply staging

# NEVER do this:
kubectl apply -f k8s/  # Wrong! Bypasses ConfigHub
```

## MANDATORY Testing Protocol (BLOCKING)

**NO COMMITS without passing all tests**

### Step 1: Run Mini TCK (MANDATORY)
```bash
cd /Users/alexis/Public/github-repos/devops-sdk
./test-confighub-k8s
```

This validates:
- ConfigHub API connection works
- Basic operations (space create, unit create, apply, destroy)
- Your environment is correctly configured

**Exit code 0 required. If Mini TCK fails, FIX IT before proceeding.**

### Step 2: Run cub-command-analyzer (MANDATORY)

**Location**: `/Users/alexis/Public/github-repos/devops-sdk/cub-command-analyzer.sh`

```bash
# From devops-sdk directory
cd /Users/alexis/Public/github-repos/devops-sdk
./cub-command-analyzer.sh /path/to/your/project/bin/

# Or run remotely
curl -fsSL https://raw.githubusercontent.com/monadic/devops-sdk/main/cub-command-analyzer.sh | bash -s -- bin/

# Fix ALL invalid commands before proceeding
```

The analyzer validates:
- **Syntax** (correct flag combinations)
- **Grammar** (WHERE clause EBNF compliance)
- **Common errors** (--patch without companions, inline JSON, etc.)

**Exit code 0 = all valid. Exit code 1 = found invalid commands (BLOCKING).**

### Step 3: Validate Config Files (MANDATORY)
```bash
# YAML syntax validation (MANDATORY for all YAML files)
for file in confighub/**/*.yaml k8s/**/*.yaml; do
  python3 -c "import yaml, sys; yaml.safe_load(open('$file'))" || echo "INVALID: $file"
done

# JSON syntax validation (MANDATORY for all JSON files)
for file in **/*.json; do
  jq empty "$file" || echo "INVALID: $file"
done
```

**All files must pass validation. Fix invalid files before committing.**

### Step 4: Run Project-Specific Tests (MANDATORY)
```bash
# Run all tests
./test/run-all-tests.sh

# Or specific test suites
./test/unit/test-*.sh
./test/integration/test-*.sh
```

**All tests must pass. Exit code 0 required.**

### Step 5: Test Commands in Real ConfigHub Space (MANDATORY)
```bash
# Create throwaway space
TEST_SPACE="test$RANDOM"
cub space create $TEST_SPACE

# Run your script against test space
./bin/my-script.sh --space $TEST_SPACE

# Verify expected results
cub unit list --space $TEST_SPACE

# Cleanup
cub space delete $TEST_SPACE
```

**Commands must work against real ConfigHub before committing.**

### When to Run Tests (MANDATORY)

- ✅ **After setup** - Verify environment (run Mini TCK)
- ✅ **Before committing** - Verify commands valid (run analyzer + tests)
- ✅ **After making changes** - Verify nothing broke (run all tests)
- ✅ **In CI/CD** - Catch regressions (automated test runs)

### CRITICAL Testing Requirement: Validate ConfigHub-Only Commands

All tests MUST verify that apps use ConfigHub commands exclusively:
```bash
# Required test patterns in all test suites
test_api "Uses cub commands" '.corrections[0].command | contains("cub unit")' "true"
test_api "NO kubectl commands" '.corrections[0].command | contains("kubectl")' "false"
```

### Testing Environment Setup
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

## MANDATORY: Before Committing (BLOCKING)

**Run this checklist EVERY TIME before git commit**

### Pre-Commit Checklist

Execute these steps in order. **If ANY step fails, do NOT commit. Fix and re-run.**

- [ ] **Run cub-command-analyzer on changed files**
  ```bash
  cd /Users/alexis/Public/github-repos/devops-sdk
  ./cub-command-analyzer.sh /path/to/your/project/bin/
  # Exit code must be 0 (no invalid commands)
  ```

- [ ] **Fix all invalid commands**
  - Review analyzer output for each invalid command
  - Apply suggested corrections
  - Re-run analyzer until exit code 0

- [ ] **Run Mini TCK**
  ```bash
  cd /Users/alexis/Public/github-repos/devops-sdk
  ./test-confighub-k8s
  # Must pass with exit code 0
  ```

- [ ] **Run project tests**
  ```bash
  ./test/run-all-tests.sh
  # All tests must pass
  ```

- [ ] **Validate YAML/JSON syntax**
  ```bash
  # Check all YAML files
  find . -name "*.yaml" -not -path "./.git/*" -exec python3 -c "import yaml; yaml.safe_load(open('{}'))" \;

  # Check all JSON files
  find . -name "*.json" -not -path "./.git/*" -exec jq empty {} \;
  ```

- [ ] **Test commands in real ConfigHub space**
  ```bash
  TEST_SPACE="test$RANDOM"
  cub space create $TEST_SPACE
  # Run your script
  ./bin/my-script.sh --space $TEST_SPACE
  # Verify it works
  cub space delete $TEST_SPACE
  ```

- [ ] **Verify no kubectl commands in code** (ConfigHub-only requirement)
  ```bash
  # Check for prohibited kubectl usage
  grep -r "kubectl" bin/ --exclude="*.md" || echo "No kubectl found (good)"
  ```

### Failure Handling

If any checklist item fails:
1. **Do NOT commit**
2. **Fix the issue**
3. **Re-run the failed step**
4. **Re-run all subsequent steps**
5. **Only commit when ALL steps pass**

### Automated Pre-Commit Hook (Recommended)

Create `.git/hooks/pre-commit`:
```bash
#!/bin/bash

echo "Running pre-commit validation..."

# Path to SDK analyzer
ANALYZER="/Users/alexis/Public/github-repos/devops-sdk/cub-command-analyzer.sh"

# Run analyzer
if [ -f "$ANALYZER" ]; then
    "$ANALYZER" bin/
    if [ $? -ne 0 ]; then
        echo "❌ Found invalid cub commands. Commit aborted."
        exit 1
    fi
else
    echo "⚠ Analyzer not found at $ANALYZER - skipping cub validation"
fi

# Validate YAML
find . -name "*.yaml" -not -path "./.git/*" -exec python3 -c "import yaml; yaml.safe_load(open('{}'))" \; 2>/dev/null
if [ $? -ne 0 ]; then
    echo "❌ Invalid YAML found. Commit aborted."
    exit 1
fi

echo "✅ All validations passed"
exit 0
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
/Users/alexis/Public/github-repos/
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
- `/Users/alexis/Public/github-repos/confighub-examples/global-app/` - ConfigHub usage patterns
- Look for `bin/install-base` and `bin/install-envs` scripts

### Testing Commands
```bash
# Build everything
cd /Users/alexis/Public/github-repos/devops-sdk && go build ./...
cd /Users/alexis/Public/github-repos/devops-examples/drift-detector && go build .

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
- `/Users/alexis/Public/github-repos/confighub-examples/global-app/README.md` - Reference patterns

## Context from Previous Sessions
This project started as analysis of Cased.com, then evolved into building a better competitor using ConfigHub. The key insight was that persistent DevOps applications are better than ephemeral workflows. We discovered many ConfigHub features were hallucinated and had to rewrite everything to use only real APIs from the source code.

The drift-detector is now a fully working example that demonstrates all the key patterns: Sets for grouping, Filters for targeting, push-upgrade for propagation, and informers for event-driven architecture.

## Remember
If you're about to use a ConfigHub feature, VERIFY it exists in `docs/CONFIGHUB-ACTUAL-FEATURES.md` first. When in doubt, use only the confirmed operations: CreateSpace, CreateUnit, ApplyUnit, DestroyUnit, CreateSet, GetSet, CreateFilter, BulkPatchUnits, BulkApplyUnits.
