# Deploy from Branch: Better Than Cased's Killer Feature

## The Original Cased.com "Killer Branch Deploy"

Cased's killer feature was "deploy from branch" - developers could push to a Git branch and automatically get a **temporary, ephemeral** preview environment. While this solved testing in isolation, it had major limitations:
- Environments destroyed after merge (waste resources on rebuild)
- No inheritance model between environments
- Stateless between deployments
- Manual promotion between branches

## How We Do It Better: Persistent Spaces with Hierarchy

### The Key Insight: Persistent Hierarchical Spaces vs Ephemeral Branches

```yaml
# Cased approach (ephemeral)
main branch → production (persistent)
feature/xyz branch → temporary environment (destroyed after merge)

# Our ConfigHub approach (canonical global-app pattern)
{unique-prefix}-base              # e.g., chubby-paws-base
    └── {prefix}-qa
        └── {prefix}-staging
            ├── {prefix}-prod
            └── {prefix}-feature-xyz  # Created with upstream relationship

# Key differences:
# 1. Uses 'cub space new-prefix' for unique naming
# 2. Full upstream/downstream inheritance
# 3. Environments persist and evolve (not destroyed)
# 4. Push-upgrade propagates changes automatically
```

### The Deploy-from-Branch Pattern (Enhanced with ConfigHub Features)

```go
// branch-deployer app monitors Git and creates ConfigHub spaces
type BranchDeployer struct {
    *sdk.DevOpsApp
    Mode string // "dev" or "enterprise"
}

func (b *BranchDeployer) MonitorBranches() error {
    // Use informers, not polling
    return b.RunWithInformers(func() error {
        branches := b.Git.GetBranches()

        for _, branch := range branches {
            if !b.hasSpace(branch) {
                // 1. Create space with upstream relationship (canonical pattern)
                parentSpace := b.determineParentSpace(branch)
                branchSpace := fmt.Sprintf("%s-%s", b.projectPrefix, branch.Name)

                // 2. Clone using canonical pattern from global-app
                b.Cub.RunCommand("cub unit create --dest-space %s --space %s --filter %s/app --label targetable=true",
                    branchSpace, parentSpace, b.projectPrefix)

                // 3. Reduce resources for preview (edit in place)
                b.Cub.UpdateUnit(branchSpace, UpdateUnitRequest{
                    Data: b.getPreviewResourceLimits(),
                })

                // 3. Use FILTERS to deploy only changed units
                filter := Filter{
                    Labels: map[string]string{
                        "changed": "true",
                        "deployable": "true",
                    },
                }
                changedUnits := b.Cub.ListUnits(b.spaceID, ListUnitsParams{
                    FilterID: filter.FilterID,
                })

                // 4. Apply changed units

                // 5. Deploy based on mode
                if b.Mode == "dev" {
                    // Direct deploy in dev mode
                    for _, unit := range changedUnits {
                        b.Cub.ApplyUnit(b.spaceID, unit.UnitID)
                    }
                } else {
                    // Enterprise mode: through Git/Flux
                    b.Git.CommitUnits(changedUnits, fmt.Sprintf("Preview for %s", branch.Name))
                    b.Flux.DeployPreview(branchSpace)
                }

                // 6. Create preview URL
                previewURL := b.createPreviewURL(branchSpace)
                b.Git.UpdatePRStatus(branch, previewURL)
            }
        }
        return nil
    })
}
```

## The Complete Implementation

### 1. Git Push Triggers Space Creation with Variants

```go
func (b *BranchDeployer) OnGitPush(event GitPushEvent) {
    // Developer pushes to feature/new-api branch
    if event.Branch.IsFeature() {
        spaceName := b.formatSpaceName(event.Branch)
        parentSpace := b.determineParentSpace(event.Branch)

        // 1. Use canonical pattern to create unique prefix
        projectPrefix := b.Cub.RunCommand("cub space new-prefix")
        spaceName := fmt.Sprintf("%s-%s", projectPrefix, event.Branch.Name)

        // 2. Create space with upstream relationship
        b.Cub.CreateSpace(spaceName, CreateSpaceRequest{
            Labels: map[string]string{
                "project": projectPrefix,
                "branch": event.Branch.Name,
                "type": "preview",
            },
        })

        // 3. Clone units with upstream (canonical pattern)
        b.Cub.RunCommand("cub unit create --dest-space %s --space %s --filter %s/app",
            spaceName, parentSpace, projectPrefix)
        if err != nil {
            b.handleCloneError(err)
            return
        }

        // 3. Apply filters to deploy only what's needed
        filter := Filter{
            Units: affectedUnits,
            Labels: map[string]string{"preview": "true"},
        }
        b.Cub.ApplyFilter(spaceName, filter)

        // 4. Apply branch changes to the cloned space
        b.applyBranchDiff(spaceName, event.Commits)
    }
}
```

### 2. Branch Changes Become Unit Updates

```go
func (b *BranchDeployer) applyBranchDiff(spaceName string, commits []Commit) {
    // Extract configuration changes from Git commits
    for _, commit := range commits {
        for _, file := range commit.Files {
            if b.isConfigFile(file) {
                // Convert Git changes to ConfigHub units
                unit := b.convertToUnit(file)

                // Update the unit in the branch space
                b.Cub.UpdateUnit(spaceName, unit)
            }
        }
    }

    // Claude analyzes if changes are safe
    analysis := b.Claude.Analyze(`
        Space: ${spaceName}
        Changes: ${commits}
        Parent space config: ${parentSpace}

        Are these changes safe to deploy?
        Any missing dependencies?
        Suggest deployment strategy.
    `)

    if !analysis.Safe {
        b.createWarningComment(analysis.Risks)
    }
}
```

### 3. Automatic Preview Environments

```go
func (b *BranchDeployer) createPreviewEnvironment(spaceName string) *PreviewEnv {
    // 1. Deploy the space to a preview namespace
    namespace := fmt.Sprintf("preview-%s", spaceName)

    deployment := b.K8s.Deploy(DeploymentSpec{
        Namespace: namespace,
        Space:     spaceName,
        Replicas:  1, // Minimal resources for preview
        Labels: map[string]string{
            "type": "preview",
            "branch": b.extractBranchName(spaceName),
            "ttl": "24h", // Auto-cleanup
        },
    })

    // 2. Create ingress for preview URL
    previewURL := fmt.Sprintf("https://%s.preview.example.com", spaceName)
    b.K8s.CreateIngress(namespace, previewURL)

    // 3. Return preview environment details
    return &PreviewEnv{
        URL:       previewURL,
        Namespace: namespace,
        Space:     spaceName,
        ExpiresAt: time.Now().Add(24 * time.Hour),
    }
}
```

## The Magic: Clone + Apply Pattern

### Clone: Inheritance from Parent Space

```go
// ConfigHub's clone operation provides inheritance
func (c *CubClient) Clone(source, target string) error {
    // 1. Copy all units from source space
    sourceUnits := c.GetUnits(source)

    // 2. Create target space with inherited config
    c.CreateSpace(target, SpaceConfig{
        Parent: source,
        Units:  sourceUnits,
        Labels: map[string]string{
            "cloned-from": source,
            "created-at":  time.Now().String(),
        },
    })

    return nil
}
```

### Apply: Branch-Specific Overrides

```go
// Apply makes targeted changes to cloned space
func (c *CubClient) Apply(space string, changes []Change) error {
    for _, change := range changes {
        switch change.Type {
        case "update":
            c.UpdateUnit(space, change.Unit)
        case "add":
            c.AddUnit(space, change.Unit)
        case "delete":
            c.DeleteUnit(space, change.Unit)
        }
    }

    return nil
}
```

## Advanced Patterns with Canonical ConfigHub Features

### 1. Smart Merge Using Push-Upgrade

```go
func (b *BranchDeployer) OnPullRequestMerged(pr PullRequest) {
    branchSpace := b.getSpaceForBranch(pr.Branch)
    targetSpace := b.getSpaceForBranch(pr.TargetBranch)

    // Use push-upgrade pattern (canonical from global-app)
    b.Cub.RunCommand("cub unit update --patch --upgrade --space %s", targetSpace)

    // Claude analyzes the upgrade impact
    analysis := b.Claude.Analyze(`
        Branch changes: ${pr.Changes}
        Target space: ${targetSpace}
        Upstream changes: ${b.getUpstreamChanges(targetSpace)}

        Are these changes safe to propagate?
        Any conflicts with downstream environments?
    `)

    if analysis.Safe {
        // Push-upgrade propagates to all downstream environments
        b.Cub.BulkPatchUnits(BulkPatchParams{
            Where: fmt.Sprintf("UpstreamSpace = '%s'", targetSpace),
            Upgrade: true,
        })
    }

    // Note: We DON'T delete the branch space - it can be reused!
    // This is a key advantage over Cased's ephemeral approach
}
```

### 2. Progressive Deployment from Branch

```go
func (b *BranchDeployer) ProgressiveDeployFromBranch(branch string) {
    space := b.getSpaceForBranch(branch)

    // Start with canary deployment
    b.deployCanary(space, 0.1) // 10% traffic

    // Monitor metrics
    metrics := b.K8s.GetMetrics(space)

    // Claude decides if safe to proceed
    decision := b.Claude.Analyze(`
        Canary metrics: ${metrics}
        Error rate: ${metrics.ErrorRate}
        Latency: ${metrics.Latency}

        Should we increase traffic to this branch?
    `)

    if decision.Proceed {
        b.deployCanary(space, 0.5)  // 50%
        time.Sleep(10 * time.Minute)
        b.deployCanary(space, 1.0)  // 100%
    } else {
        b.rollback(space)
    }
}
```

### 3. Multi-Service Branch Deployments

```go
func (b *BranchDeployer) DeployMicroservicesBranch(branch string) {
    // Branch might affect multiple services
    services := b.detectAffectedServices(branch)

    // Create a space for each service from the branch
    for _, service := range services {
        serviceSpace := fmt.Sprintf("%s-%s-%s",
            service.BaseSpace,
            branch.Name,
            service.Name)

        // Clone service's base config
        b.Cub.Clone(service.BaseSpace, serviceSpace)

        // Apply branch changes for this service
        b.applyServiceChanges(serviceSpace, branch, service)

        // Deploy with service mesh routing
        b.deployWithIstio(serviceSpace, branch)
    }

    // Create unified preview URL that routes to all services
    previewURL := b.createUnifiedPreview(branch, services)
    b.notifyDeveloper(previewURL)
}
```

## Comparison: Cased "Killer Branch Deploy" vs Our Approach

| Aspect | Cased Ephemeral | Our Persistent Spaces | ConfigHub Features Used |
|--------|-----------------|----------------------|------------------------|
| **Environment Lifecycle** | Destroyed after merge | Persists, can be reused | Spaces with upstream |
| **Resource Usage** | Full resources per preview | Optimized via editing | UpdateUnit for limits |
| **Inheritance** | None, copy everything | Full hierarchy | `--upstream-unit` |
| **Promotion** | Manual between branches | Automatic propagation | Push-upgrade pattern |
| **State** | Lost when destroyed | Preserved in ConfigHub | Revision history |
| **Rollback** | Redeploy old branch | Instant via revisions | Built-in versioning |
| **Bulk Operations** | Per-branch workflows | Cross-environment | Sets and Filters |
| **Monitoring** | Starts fresh each time | Continuous with history | Event-driven informers |

## The Killer Feature Realized

```bash
# Developer experience
$ git push origin feature/new-api

# Automatically happens:
Creating space: acorn-bear-feature-new-api
Cloning from: acorn-bear-staging
Applying branch changes...
Deploying preview environment...

Preview ready at: https://feature-new-api.preview.example.com
Claude analysis: Changes look safe, no conflicts detected

# When PR is merged
Merging configuration to acorn-bear-staging...
Cleaning up preview environment...
Done!
```

## Why This is Better Than Cased

### Cased Approach
- Triggers ephemeral workflows from branches
- No configuration inheritance model
- Complex YAML for deployment definitions
- Limited to their supported platforms
- Full resources for every preview (expensive!)
- No dependency awareness

### Our ConfigHub + DevOps Apps Approach (Using REAL Features)
- **Unique prefixes**: `cub space new-prefix` prevents collisions
- **Upstream/downstream**: Real inheritance model
- **Resource optimization**: Edit units in place to reduce limits
- **Filters with WHERE**: Deploy only what matches criteria
- **Sets for grouping**: Operate on related units together
- **Push-upgrade**: Automatic propagation through hierarchy
- **Live State tracking**: Know actual deployment status
- **Claude integration**: AI-powered safety analysis
- **Event-driven**: Informers react immediately to changes
- **Two modes**: Dev (direct) or Enterprise (GitOps)

### ConfigHub Features That Don't Exist (Don't Use):
- ❌ No "variants" - edit units directly instead
- ❌ No "dependency graphs" - use Sets and Filters
- ❌ No "gates" for preview creation - use Labels
- ❌ No "CloneWithVariant" - clone then edit

## Implementation Summary

The "deploy from branch" killer feature works even better in ConfigHub because:

1. **Spaces ARE branches** - Natural mapping from Git branches to ConfigHub spaces
2. **Clone provides inheritance** - No complex Git merging for config
3. **Apply enables overrides** - Branch-specific changes are clean
4. **Preview environments are automatic** - Just deploy the space
5. **Claude adds intelligence** - Analyzes safety, conflicts, merge strategies

```go
// The complete branch-deployer app
package main

type BranchDeployer struct {
    *sdk.DevOpsApp
}

func main() {
    app := &BranchDeployer{
        DevOpsApp: sdk.NewDevOpsApp(sdk.DevOpsAppConfig{
            Name: "branch-deployer",
        }),
    }

    // Watch Git for branch changes
    app.Git.OnPush(app.HandleBranchPush)
    app.Git.OnPRMerge(app.HandleMerge)

    // Continuous management of preview environments
    app.Run(func() error {
        app.cleanupExpiredPreviews()
        app.syncBranchSpaces()
        return nil
    })
}
```

This is job #13: **Intelligent Branch Deployment** - and it's actually MORE powerful than Cased's implementation!