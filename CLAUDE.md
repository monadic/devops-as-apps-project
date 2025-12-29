# Claude Code Context for DevOps as Apps Project

## Project Overview

**THIS REPOSITORY:** Planning, documentation, and example applications for testing.

**Primary uses:**
1. DevOps-as-Apps example applications (drift-detector, cost-optimizer, etc.)
2. Test applications for confighub-agent development and validation
3. Documentation and patterns for ConfigHub usage

**NOT THIS REPOSITORY:** ConfigHub Agent implementation (see `~/Public/github-repos/confighub-agent` instead)

DevOps as Apps: persistent Kubernetes applications using ConfigHub for configuration management.

**Key repositories:**
- **This repo** (`~/devops-as-apps-project`): Planning, documentation, and examples
- **confighub-agent** (`~/Public/github-repos/confighub-agent`): Agent implementation, CCVE scanner, map tool
- [devops-sdk](https://github.com/monadic/devops-sdk): Go SDK
- [devops-examples](https://github.com/monadic/devops-examples): drift-detector, cost-optimizer

**Reference implementations:**
- [global-app](https://github.com/confighubai/examples/tree/main/global-app): Canonical ConfigHub patterns
- Local: `/Users/alexis/Public/github-repos/confighub-examples/global-app/`

## Communication Standards

- Dry technical language only. No marketing, speculation, or emotional language.
- Verify all ConfigHub features against actual CLI help before documenting.
- Label uncertain content explicitly.

## Documentation Code is Production Code (MANDATORY)

**Code examples in README.md, tutorials, and documentation files require the same validation rigor as executable scripts.**

### Why This Matters

Users copy-paste documentation examples directly. Incorrect syntax in docs causes:
- User frustration and support burden
- Loss of trust in documentation quality
- Propagation of errors as users build on broken examples

### Validation Requirements for All .md Files

**Before writing any code block in documentation:**

1. **Run Mini TCK first** - Verify your environment works before testing commands
2. **Verify CLI syntax** - Run `cub <cmd> --help` for every cub command
3. **Test the exact command** - Copy-paste and execute in a test space
4. **Check flag combinations** - Mode flags require specific companions

**Before committing documentation with code examples:**

```bash
# 1. Run Mini TCK to ensure environment is correctly configured
cd /Users/alexis/Public/github-repos/devops-sdk
./test-confighub-k8s  # Must pass before proceeding

# 2. Extract cub commands from markdown and validate with cub-command-analyzer
# This catches incorrect flag usage, non-existent commands, wrong syntax
cd /Users/alexis/Public/github-repos/devops-sdk
./cub-command-analyzer.sh /path/to/your/project/

# 3. For README files with code examples, extract and validate
grep -E '^\s*cub ' README.md | while read cmd; do
  echo "Validating: $cmd"
done
```

### Required Tools for Documentation Validation

| Tool | Location | Purpose |
|------|----------|---------|
| **Mini TCK** | `/Users/alexis/Public/github-repos/devops-sdk/test-confighub-k8s` | Verify ConfigHub API works |
| **cub-command-analyzer** | `/Users/alexis/Public/github-repos/devops-sdk/cub-command-analyzer.sh` | Validate CLI syntax |
| **cub --help** | CLI | Check individual command flags |

**Both tools MUST pass before committing documentation with cub commands.**

### Common Documentation Errors to Avoid

| Wrong | Right | Issue |
|-------|-------|-------|
| `cub link create --from X --to Y` | `cub link create slug from-unit to-unit` | Links use positional args |
| `--patch '{"spec":{}}'` | `cub run set-replicas --replicas N` | --patch is boolean, not value |
| `cub set create` | `cub filter create` | cub set doesn't exist |
| `cub changeset apply` | `cub unit apply --where "ChangeSet..."` | changeset apply doesn't exist |

### Treating Documentation Bugs as Seriously as Code Bugs

- Documentation syntax errors should block commits
- README code examples need review just like source code
- "Example code" is not exempt from validation

## CLI Command Validation (MANDATORY)

Before using ANY cub command:

```bash
# 1. Check help
CONFIGHUB_AGENT=1 cub <entity> <verb> --help

# 2. Test in throwaway space
TEST_SPACE="test$RANDOM"
cub space create $TEST_SPACE
# ... test command ...
cub space delete $TEST_SPACE
```

**Available entities:** auth, changeset, context, filter, function, helm, link, mutation, revision, run, space, tag, target, trigger, unit, worker

**NO `cub set` command exists** - Sets are API-only, not CLI.

## Verified CLI Patterns

See [docs/CLI-REFERENCE.md](docs/CLI-REFERENCE.md) for complete reference.

### Essential Commands

```bash
# Project setup
prefix=$(cub space new-prefix)
cub space create $project --label project=$project
cub filter create --space $project all Unit --where-field "Space.Labels.project = '$project'"

# Unit operations
cub unit create --space $project myunit config.yaml
cub unit create --space $project-qa myunit --upstream-unit myunit --upstream-space $project-base
cub unit update --space $project myunit --upgrade
cub unit apply myunit --space $project

# Worker setup (MUST use --include-secret)
cub worker create myworker --space $project
cub worker install myworker --space $project --namespace confighub --include-secret --export > worker.yaml
```

## Testing Requirements

### Before Any Development

```bash
# Run Mini TCK
cd /Users/alexis/Public/github-repos/devops-sdk
./test-confighub-k8s
```

### Before Committing

```bash
# Validate CLI commands
./cub-command-analyzer.sh bin/

# Validate YAML
python3 -c 'import yaml; yaml.safe_load(open("file.yaml"))'

# Run tests
./test/run-all-tests.sh
```

## File Locations

```
/Users/alexis/Public/github-repos/
├── confighub-latest/          # ConfigHub source (READ-ONLY)
├── confighub-examples/        # Official examples
│   └── global-app/            # Canonical reference
├── devops-as-apps-project/    # This repo (planning/docs)
├── devops-sdk/                # Go SDK
└── devops-examples/           # Example apps
    ├── drift-detector/
    └── cost-optimizer/
```

## App Structure

```
app-name/
├── confighub/base/            # K8s manifests
├── bin/
│   ├── install-base           # Create ConfigHub units
│   ├── setup-worker           # Install worker
│   ├── apply-base             # Deploy via ConfigHub
│   └── proj                   # Get project name
├── main.go
└── README.md
```

## Key Principles

1. **Deploy through ConfigHub** - Use `cub unit apply`, not `kubectl apply`
2. **Environment hierarchy** - base → dev → staging → prod with upstream/downstream
3. **Verify before documenting** - All CLI examples must be tested
4. **No Sets CLI** - Sets exist in API but have no CLI commands

## Critical: Unit Update Patterns

**Updating unit YAML config data from stdin:**
```bash
# CORRECT: Use "-" for YAML config data from stdin
cat config.yaml | cub unit update --space dev myunit -

# WRONG: --from-stdin is for JSON metadata, NOT config data
cat config.yaml | cub unit update myunit --from-stdin --space dev  # FAILS
```

**Updating unit metadata with --patch:**
```bash
# CORRECT: Patch mode with label changes
cub unit update --patch --space dev myunit --label version=2.0

# CORRECT: JSON metadata patch from stdin
echo '{"Labels":{"tier":"critical"}}' | cub unit update --patch --space dev myunit --from-stdin
```

**Key insight:** `--from-stdin` reads JSON metadata. The `-` argument reads YAML config data.

## What NOT to Do

- Don't use `cub set` commands (doesn't exist)
- Don't use `--revision=N` (use `--revision N` with space)
- Don't use `cub bulk apply` (doesn't exist, use `cub unit apply --where`)
- Don't confuse `--from-stdin` (JSON metadata) with `-` (YAML config data)
- Don't trust old documentation - verify against `cub --help`
