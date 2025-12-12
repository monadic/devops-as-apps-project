# ConfigHub CLI Reference

Verified against `cub --help` output. Run `CONFIGHUB_AGENT=1 cub <command> --help` for details.

## Available Commands

```
auth, changeset, context, filter, function, helm, info, invocation, k8s,
link, mutation, organization, revision, run, space, tag, target, trigger,
unit, unit-action, unit-event, upgrade, user, version, view, worker
```

**Note**: There is NO `cub set` command. Sets exist in the API but not CLI.

## Core Patterns

### Project Setup
```bash
# Generate unique prefix
prefix=$(cub space new-prefix)
project="${prefix}-myapp"

# Create spaces
cub space create $project --label project=$project

# Create filter
cub filter create --space $project all Unit --where-field "Space.Labels.project = '$project'"
```

### Unit Operations
```bash
# Create unit from file
cub unit create --space $project myunit config.yaml

# Create with upstream (for cloning)
cub unit create --space $project-qa myunit \
  --upstream-unit myunit --upstream-space $project-base

# Bulk clone to destination space
cub unit create --dest-space $project-qa --space $project-base \
  --where "Labels.type = 'app'"

# Update unit
cub unit update --space $project myunit config.yaml

# Upgrade from upstream
cub unit update --space $project myunit --upgrade

# Bulk upgrade
cub unit update --patch --upgrade --space $project --where "UpstreamUnitID != ''"

# Apply
cub unit apply myunit --space $project
cub unit apply --space $project --where "Labels.tier = 'backend'"

# List
cub unit list --space $project
```

### Worker Setup
```bash
# Create worker
cub worker create myworker --space $project

# Install with credentials (MUST use --include-secret)
cub worker install myworker --space $project --namespace confighub --include-secret --export > worker.yaml
kubectl apply -f worker.yaml

# Verify
cub worker list --space $project
cub target list --space $project
```

### Functions (cub run)
```bash
# Set image version
cub run set-image-reference --container-name app --image-reference :1.0.0 --space $project --unit myunit

# Set environment variable
cub run set-env-var --container-name app --env-var KEY --env-value value --space $project --unit myunit

# Set resources
cub run set-container-resources --container-name app --memory 1Gi --cpu 500m --space $project --unit myunit
```

### Links
```bash
# Single link
cub link create --space $project mylink from-unit to-unit

# Bulk links
cub link create --where-space "Slug = '$project'" --where-from "Labels.type = 'app'" --where-to "Slug = 'namespace'"
```

### Changesets
```bash
# Create changeset
cub changeset create --space $project mychange --description "My changes"

# Associate units
cub unit update --patch --space $project --changeset $project/mychange --where "Slug = 'myunit'"

# Apply with changeset
cub unit apply --space $project --revision "ChangeSet:$project/mychange"
```

### Revisions
```bash
# List revisions
cub revision list myunit --space $project

# Diff
cub unit diff -u myunit --space $project --from=5

# Rollback
cub unit apply myunit --space $project --revision 5
```

## Key Flags

| Flag | Purpose |
|------|---------|
| `--space` | Target space (required for most ops) |
| `--where` | Filter expression for bulk ops |
| `--upstream-unit` | Create with upstream relationship |
| `--upgrade` | Pull changes from upstream |
| `--include-secret` | Include credentials in worker manifest |
| `--from-stdin` | Read data from stdin |
| `--patch` | Metadata-only update |

## WHERE Clause Syntax

```sql
Slug = 'myunit'
Labels.tier = 'backend'
Space.Labels.project = 'myproject'
Slug LIKE 'app-%'
Labels.env IN ('dev', 'staging')
condition1 AND condition2
```
