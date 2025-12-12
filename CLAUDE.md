# Claude Code Context for DevOps as Apps Project

## Project Overview

DevOps as Apps: persistent Kubernetes applications using ConfigHub for configuration management.

**Key repositories:**
- This repo: Planning and docs
- [devops-sdk](https://github.com/monadic/devops-sdk): Go SDK
- [devops-examples](https://github.com/monadic/devops-examples): drift-detector, cost-optimizer

**Reference implementations:**
- [global-app](https://github.com/confighubai/examples/tree/main/global-app): Canonical ConfigHub patterns
- Local: `/Users/alexis/Public/github-repos/confighub-examples/global-app/`

## Communication Standards

- Dry technical language only. No marketing, speculation, or emotional language.
- Verify all ConfigHub features against actual CLI help before documenting.
- Label uncertain content explicitly.

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

## What NOT to Do

- Don't use `cub set` commands (doesn't exist)
- Don't use `--revision=N` (use `--revision N` with space)
- Don't use `cub bulk apply` (doesn't exist, use `cub unit apply --where`)
- Don't trust old documentation - verify against `cub --help`
