# Understanding ConfigHub and SDK Roles

## The Three-Layer Architecture

```
┌─────────────────────────────────────────┐
│         DevOps Applications             │  Layer 3: Your Apps
│  (drift-detector, cost-optimizer, etc)  │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│           DevOps SDK                    │  Layer 2: Framework
│  (app.go, claude.go, kubernetes.go)     │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│            ConfigHub                    │  Layer 1: Foundation
│  (Spaces, Units, Sets, Filters)         │
└─────────────────────────────────────────┘
```

## ConfigHub: The Configuration Management Foundation

### What ConfigHub IS:
ConfigHub is a **configuration management system** that provides:

1. **Hierarchical Configuration Storage**
   ```yaml
   base-config
       └── qa-config (inherits from base)
           └── staging-config (inherits from qa)
               └── prod-config (inherits from staging)
   ```

2. **Core Primitives**:
   - **Spaces**: Isolated workspaces for configuration
   - **Units**: Individual configuration items (like K8s manifests)
   - **Sets**: Groups of units for bulk operations
   - **Filters**: SQL-like WHERE clauses for targeting

3. **Key Operations**:
   ```bash
   # Create unique namespace
   cub space new-prefix  # Returns "chubby-paws"

   # Create hierarchy
   cub unit create --upstream-unit base-unit

   # Bulk operations
   cub unit apply --filter "Labels.tier='critical'"

   # Push changes downstream
   cub unit update --patch --upgrade
   ```

### What ConfigHub is NOT:
- ❌ Not a deployment tool (that's Kubernetes)
- ❌ Not a CI/CD pipeline (that's GitHub Actions/Jenkins)
- ❌ Not a monitoring system (that's Prometheus)
- ❌ Not an AI platform (that's Claude)

### ConfigHub's Role in Our Architecture:
```go
// ConfigHub stores and manages all configuration
type ConfigHub struct {
    // Stores configuration with hierarchy
    StoreConfig(space string, unit Unit) error

    // Retrieves with inheritance
    GetConfig(space string) ([]Unit, error)

    // Propagates changes downstream
    PushUpgrade(upstreamID string) error

    // Groups for bulk operations
    CreateSet(name string, units []Unit) error

    // Powerful querying
    Filter(where string) ([]Unit, error)
}
```

## SDK: The DevOps Application Framework

### What the SDK IS:
The SDK is a **Go framework** that standardizes how DevOps apps are built:

1. **Base Application Structure**:
   ```go
   // Every app starts with this
   type DevOpsApp struct {
       Name    string
       Cub     *ConfigHubClient  // ConfigHub integration
       K8s     *KubernetesClient // K8s integration
       Claude  *ClaudeClient     // AI integration
       Logger  *Logger
   }
   ```

2. **Event-Driven Architecture**:
   ```go
   // SDK provides informers for reactive apps
   app.RunWithInformers(func() error {
       // React to changes immediately
       return processChanges()
   })
   ```

3. **Standard Integrations**:
   - ConfigHub client (confighub.go)
   - Kubernetes utilities (kubernetes.go)
   - Claude AI helpers (claude.go)
   - Logging and metrics (app.go)

### What the SDK is NOT:
- ❌ Not a standalone product (it's a library)
- ❌ Not ConfigHub itself (it uses ConfigHub)
- ❌ Not the apps (it helps build apps)

### SDK's Role in Our Architecture:
```go
// SDK provides the framework for building apps
import "github.com/monadic/devops-sdk"

func main() {
    // SDK gives you everything you need
    app := sdk.NewDevOpsApp(sdk.DevOpsAppConfig{
        Name: "my-devops-app",
    })

    // Built-in ConfigHub integration
    units := app.Cub.GetUnits(spaceID)

    // Built-in K8s integration
    app.K8s.Apply(units)

    // Built-in AI integration
    analysis := app.Claude.Analyze("Should I deploy?", context)

    // Event-driven by default
    app.RunWithInformers(reconcileFunc)
}
```

## How They Work Together

### Example: Drift Detector

```go
// 1. APP LAYER: Your drift detector logic
type DriftDetector struct {
    *sdk.DevOpsApp  // 2. SDK LAYER: Inherits framework
}

func (d *DriftDetector) DetectDrift() {
    // 3. CONFIGHUB LAYER: Get desired state
    desired := d.Cub.GetUnits(d.SpaceID)

    // 2. SDK LAYER: Use K8s utilities
    actual := d.K8s.GetCurrentState()

    if drift := d.ComputeDrift(desired, actual); drift != nil {
        // 2. SDK LAYER: Use AI integration
        fix := d.Claude.Analyze("How to fix?", drift)

        // 3. CONFIGHUB LAYER: Store fix
        d.Cub.UpdateUnit(unitID, fix)

        // 2. SDK LAYER: Apply fix
        d.K8s.Apply(fix)
    }
}

func main() {
    // 2. SDK LAYER: Framework initialization
    app := &DriftDetector{
        DevOpsApp: sdk.NewDevOpsApp(config),
    }

    // 2. SDK LAYER: Event-driven execution
    app.RunWithInformers(app.DetectDrift)
}
```

## The Power of Separation

### ConfigHub Handles:
- Configuration storage and versioning
- Environment hierarchy and inheritance
- Bulk operations via Sets and Filters
- Change propagation via push-upgrade

### SDK Handles:
- Application structure and lifecycle
- Kubernetes integration and informers
- Claude AI integration
- Standard logging and metrics

### Your App Handles:
- Business logic (drift detection, cost optimization, etc.)
- Decision making (when to act)
- Domain-specific knowledge

## Real-World Analogy

Think of it like building a house:

1. **ConfigHub** = The foundation and utilities (plumbing, electrical)
   - Provides essential services
   - You don't see it but everything depends on it

2. **SDK** = The framework and standard components (walls, doors, windows)
   - Gives structure to your building
   - Standardizes how things fit together

3. **Your App** = The custom design and interior (layout, decoration)
   - Makes it unique for your needs
   - Uses the framework and utilities

## Why This Architecture Wins

### Without Our Platform:
```yaml
Every team builds:
- Custom configuration management
- Custom K8s integration
- Custom monitoring
- Custom AI integration
- Custom deployment patterns

Result: 10-15 different tools, nothing works together
```

### With Our Platform:
```yaml
ConfigHub provides:
- Universal configuration management

SDK provides:
- Standard app framework

You focus on:
- Your actual DevOps logic

Result: Consistent, powerful apps that work together
```

## Common Misconceptions Clarified

### ❌ "ConfigHub deploys to Kubernetes"
✅ **Reality**: ConfigHub stores configuration. The SDK helps your app deploy to Kubernetes.

### ❌ "The SDK is another DevOps tool"
✅ **Reality**: The SDK is a library for building DevOps tools.

### ❌ "ConfigHub competes with Git"
✅ **Reality**: ConfigHub manages configuration with features Git doesn't have (Sets, Filters, push-upgrade).

### ❌ "You need to learn ConfigHub APIs directly"
✅ **Reality**: The SDK wraps ConfigHub APIs in convenient methods.

## Summary

- **ConfigHub**: Where your configuration lives (the database)
- **SDK**: How you build apps (the framework)
- **Your Apps**: What solves problems (the business logic)

Together, they create a platform where DevOps applications are:
- **Consistent**: All follow the same patterns
- **Powerful**: Leverage shared capabilities
- **Simple**: Focus on logic, not plumbing

This is why we can build in days what traditionally takes months.