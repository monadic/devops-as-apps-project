# ConfigHub ACTUAL Features (Based on Source Code Review)

## ✅ CONFIRMED ConfigHub Features (These Actually Exist)

Based on reviewing the actual ConfigHub source code at `github.com/confighubai/confighub`:

### Core Entities
- **Spaces**: Namespaces for configuration (confirmed in `models/space.go`)
- **Units**: Core configuration units containing YAML/HCL/etc (confirmed in `models/unit.go`)
- **Sets**: Groups of related units (confirmed in `models/set.go`)
- **Targets**: Where configurations are deployed
- **Filters**: Query and filter units (extensive filter API in client)
- **ChangeSets**: Groups of changes to apply together

### Actual API Operations (from `public/openapi/goclient-new/client.gen.go`)

#### Space Operations
```go
CreateSpace()
GetSpace()
UpdateSpace()
DeleteSpace()
ListSpaces()
```

#### Unit Operations
```go
CreateUnit()
GetUnit()
UpdateUnit()
DeleteUnit()
ListUnits()
ApplyUnit()        // Apply configuration to target
DestroyUnit()      // Remove configuration from target
BulkApplyUnits()   // Apply multiple units
BulkDestroyUnits() // Destroy multiple units
```

#### Set Operations (CONFIRMED - Sets DO exist!)
```go
CreateSet()
GetSet()
UpdateSet()
DeleteSet()
ListSets()
```

#### Filter Operations (CONFIRMED - Filters DO exist!)
```go
CreateFilter()
GetFilter()
UpdateFilter()
DeleteFilter()
ListFilters()
BulkCreateFilters()
BulkDeleteFilters()
```

#### Upgrade Operations (from `unit_push_upgrade.go`)
```go
// Push-upgrade propagates changes from upstream to downstream units
// Units have UpstreamUnitID and UpstreamRevisionNum fields
BulkPatchUnits(params) with Upgrade: true
```

### CLI Commands (from `public/cmd/cub/`)
```bash
cub unit create
cub unit apply         # Deploy to Kubernetes/cloud
cub unit destroy        # Remove from Kubernetes/cloud
cub unit push-upgrade   # Upgrade downstream units
cub unit diff
cub unit tree           # Show dependency tree
cub unit livestate      # Get live deployment status
```

### Toolchain Types Supported
- Kubernetes/YAML
- OpenTofu/HCL
- AppConfig/Properties
- ConfigHub/YAML

## ❌ Features We ASSUMED But Are NOT Confirmed

### 1. Variants ❌
- No evidence of "variants" (aws-variant, gcp-variant) in the code
- This was our hallucination!

### 2. CloneWithVariant ❌
- No such operation found
- Units can have upstream relationships but not "variant cloning"

### 3. Gates ❌
- No "gates" for controlling promotion
- There are "ApplyGates" mentioned but different from what we assumed

### 4. Dependency Graph ❌
- No GetDependencyGraph() operation
- Units have upstream/downstream relationships but not full dependency management

### 5. Live Status Updates ❌
- No UpdateStatus() operation we can call
- Units have livestate but it's read-only

## ✅ What ConfigHub ACTUALLY Does

### Hierarchy and Inheritance
- Units can have **upstream units** (UpstreamUnitID field)
- Push-upgrade propagates changes downstream
- This provides inheritance but NOT through "cloning with variants"

### Sets for Grouping
- Sets ARE real and group related units
- Units have optional SetID field
- Can operate on sets as a group

### Filters for Querying
- Rich filter API for selecting units
- WHERE clauses for bulk operations
- This enables targeted operations

### Apply/Destroy Pattern
- Units are applied to targets (Kubernetes clusters, cloud providers)
- Can bulk apply/destroy
- Tracks live state

## 🔧 How to Use ConfigHub Correctly

### Dev Mode (Direct)
```go
// Direct ConfigHub → Kubernetes
client := confighub.NewClient()
units := client.ListUnits(spaceID)
client.ApplyUnit(unitID)  // Direct to K8s
```

### Enterprise Mode (With Git)
```go
// ConfigHub → Git → Flux/Argo → Kubernetes
units := client.ListUnits(spaceID)
git.Commit(units.Data)
// Let Flux/Argo handle deployment
```

### Working with Sets
```go
// Create a set of related units
set := client.CreateSet(spaceID, "frontend-services")
// Add units to set
client.UpdateUnit(unitID, SetID: set.ID)
// Operate on the set
client.BulkApplyUnits(SetID: set.ID)
```

### Using Filters
```go
// Filter units by criteria
filter := client.CreateFilter(spaceID, WHERE: "Slug LIKE 'prod-%'")
units := client.ListUnits(FilterID: filter.ID)
```

### Push-Upgrade Pattern
```go
// Upgrade downstream units from upstream
client.BulkPatchUnits(
    Where: "UpstreamUnitID = X AND UpstreamRevisionNum < current",
    Upgrade: true,
)
```

## ⚠️ What We Need to Fix in Our Documentation

1. **Remove all references to "variants"** - they don't exist
2. **Remove "CloneWithVariant"** - not a real operation
3. **Remove "gates" for promotion control** - not how it works
4. **Remove "GetDependencyGraph"** - doesn't exist
5. **Fix upgrade pattern** - use push-upgrade, not "UpgradeSet"
6. **Clarify Sets** - they exist but work differently than assumed
7. **Clarify Filters** - they exist and are powerful
8. **Fix hierarchy** - based on upstream/downstream, not "space cloning"

## ✅ Safe Subset to Build With

```go
type RealConfigHubApp struct {
    // CONFIRMED operations only
    client.CreateSpace()
    client.CreateUnit()
    client.ApplyUnit()
    client.DestroyUnit()
    client.CreateSet()
    client.CreateFilter()
    client.ListUnits(FilterID)
    client.BulkApplyUnits(SetID)
    client.PushUpgrade(upstreamUnitID)
}
```

## Next Steps

1. **Update all .md files** to remove hallucinated features
2. **Rewrite examples** using only confirmed operations
3. **Fix the SDK design** to match actual ConfigHub capabilities
4. **Test with real ConfigHub** before making claims