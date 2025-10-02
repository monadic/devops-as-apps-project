# Proposal: Integrating Multiple DevOps Apps

**Status**: Draft
**Author**: Claude (DevOps as Apps Project)
**Date**: 2025-01-10

## Problem Statement

We have multiple DevOps apps (drift-detector, cost-optimizer, cost-impact-monitor) that analyze the same Kubernetes resources but operate in isolation. Users want to see combined insights like "this drift costs $12/month" without complex integration work.

Traditional microservices integration requires message buses, shared databases, service meshes, and schema coordination. We need to show that ConfigHub makes this trivially simple.

## Solution: ConfigHub Native Integration

**Key Insight**: ConfigHub's cross-space queries and label conventions provide zero-config integration between DevOps apps.

### Integration Mechanism

Apps integrate through three ConfigHub primitives:

1. **Cross-Space Queries** - `cub unit list --space '*'` finds units across all spaces
2. **Label Conventions** - Namespaced labels like `drift.status` and `cost.monthly` prevent collisions
3. **Shared Resource Identity** - Units with same Slug across spaces represent same K8s resource

**No coordination required.** Apps discover each other's data automatically through ConfigHub's query API.

## Architecture

### Current State (Isolated Apps)
```
drift-detector-dev/
  └── backend-api (Labels: replicas=5, expected=3)

cost-optimizer-dev/
  └── backend-api (Labels: monthly=$45, optimizable=true)
```

### Integrated State (Same Units, Multiple Perspectives)
```bash
# Single query returns all perspectives
cub unit list --space '*' --where "Slug = 'backend-api'"

# Output shows drift + cost + deployment state
SPACE                 LABELS
drift-detector-dev    drift.detected=true, drift.cost-impact=$12/mo
cost-optimizer-dev    cost.monthly=$45, cost.savings=$12/mo
my-app-dev           version=1.2.3, replicas=5
```

### Label Convention (Standard Across All Apps)

| Namespace | Labels | Owner |
|-----------|--------|-------|
| `drift.*` | `detected`, `expected`, `actual`, `cost-impact` | drift-detector |
| `cost.*` | `monthly`, `optimizable`, `savings`, `risk` | cost-optimizer |
| `security.*` | `cve-count`, `severity`, `last-scan` | security-scanner (future) |
| `app.*` | `version`, `team`, `environment` | Application owner |

**Rule**: Each app uses its own namespace prefix. No coordination needed.

## Implementation

### Phase 1: Add Cross-Space Query Helper to SDK (1 day)

```go
// In devops-sdk/confighub.go
func (c *ConfigHubClient) GetUnitAllSpaces(slug string) ([]Unit, error) {
    return c.ListUnits("*", fmt.Sprintf("Slug = '%s'", slug))
}

func (c *ConfigHubClient) GetCombinedLabels(slug string) (map[string]string, error) {
    units := c.GetUnitAllSpaces(slug)
    // Merge all labels from all spaces
    combined := make(map[string]string)
    for _, unit := range units {
        for k, v := range unit.Labels {
            combined[k] = v
        }
    }
    return combined, nil
}
```

### Phase 2: Update Apps to Write Namespaced Labels (2 days)

**Drift Detector:**
```go
// Write findings to ConfigHub with drift.* prefix
unit.Labels["drift.detected"] = "true"
unit.Labels["drift.expected-replicas"] = "3"
unit.Labels["drift.actual-replicas"] = "5"
unit.Labels["drift.cost-impact"] = "12"
```

**Cost Optimizer:**
```go
// Write findings with cost.* prefix
unit.Labels["cost.monthly"] = "45"
unit.Labels["cost.optimizable"] = "true"
unit.Labels["cost.savings"] = "12"
```

### Phase 3: Create Combined Dashboard Script (1 day)

```bash
#!/bin/bash
# bin/combined-view

UNIT=$1
echo "=== $UNIT - Combined DevOps Insights ==="
echo ""

# Drift status
echo "📊 Drift Analysis:"
cub unit get $UNIT --space '*drift*' --format json | \
  jq -r '.Labels | to_entries | .[] | select(.key | startswith("drift")) | "  \(.key): \(.value)"'

# Cost analysis
echo ""
echo "💰 Cost Analysis:"
cub unit get $UNIT --space '*cost*' --format json | \
  jq -r '.Labels | to_entries | .[] | select(.key | startswith("cost")) | "  \(.key): \(.value)"'

# Live state
echo ""
echo "🎯 Live State:"
kubectl get deployment $UNIT -o json | jq -r '{replicas: .spec.replicas, image: .spec.template.spec.containers[0].image}'
```

### Phase 4: Document Integration Pattern (1 day)

Add to CANONICAL-PATTERNS-SUMMARY.md as **Pattern 13: Cross-App Integration**

## User Experience

### Before (Isolated)
```bash
# Check drift
cd drift-detector && ./drift-detector
# Output: backend-api has drift

# Check cost (separate terminal)
cd cost-optimizer && ./cost-optimizer
# Output: backend-api costs $45/month

# User must mentally correlate the two
```

### After (Integrated)
```bash
# Single command shows everything
bin/combined-view backend-api

# Output:
# 📊 Drift Analysis:
#   drift.detected: true
#   drift.expected-replicas: 3
#   drift.actual-replicas: 5
#   drift.cost-impact: $12/mo
#
# 💰 Cost Analysis:
#   cost.monthly: $45
#   cost.savings: $12 (if drift fixed)
#   cost.optimizable: true
#
# 🎯 Live State:
#   replicas: 5
#   image: backend-api:1.2.3
```

## Success Criteria

1. ✅ Zero configuration needed to query across apps
2. ✅ No shared code between apps (loose coupling via labels)
3. ✅ Apps work independently or together
4. ✅ New apps can integrate by following label convention
5. ✅ Combined view shows correlated insights (drift + cost)
6. ✅ Implementation < 5 days total

## Non-Goals

- Tight coupling between apps (keep them independent)
- Real-time pub/sub between apps (Kubernetes informers handle this)
- Shared state or databases (ConfigHub is the database)
- Complex orchestration (apps remain autonomous)

## Competitive Advantage

**Traditional Microservices Integration:**
- Kafka/RabbitMQ for events ($$$)
- Service mesh for discovery (complex)
- Schema registry (coordination overhead)
- Shared database (coupling)

**ConfigHub Integration:**
```bash
# That's it.
cub unit list --space '*' --where "Slug = 'backend-api'"
```

**Key Message**: "With ConfigHub, integrating DevOps apps is a single line of code. No message buses, no service meshes, no coordination."

## Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| Label name collisions | Enforce namespace prefix convention in SDK |
| Query performance across many spaces | ConfigHub handles this efficiently (it's a database) |
| Apps reading stale data | ConfigHub provides eventual consistency (fine for DevOps use cases) |

## Next Steps

1. **Week 1**: Add cross-space helpers to devops-sdk
2. **Week 2**: Update drift-detector with namespaced labels
3. **Week 3**: Update cost-optimizer with namespaced labels
4. **Week 4**: Create combined-view demo script
5. **Week 5**: Document as Pattern 13, publish blog post

**Demo target**: Show drift + cost integration at next team meeting (2 weeks)
