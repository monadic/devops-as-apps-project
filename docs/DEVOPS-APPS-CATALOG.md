# DevOps Apps Catalog: Jobs to Be Done

## Overview
This catalog details specific DevOps applications that demonstrate the **Config as Data + DevOps as Apps** pattern. Each app shows how treating configuration as data and managing it with modern applications beats traditional approaches.

## The Jobs-to-be-Done Framework

Each DevOps job follows the same pattern:
1. **Configuration is Data** - Stored in ConfigHub with structure and relationships
2. **Apps Manage the Data** - Persistent applications query, analyze, and update
3. **Intelligence via AI** - Claude provides smart decisions
4. **Continuous Operation** - Apps run 24/7, not just when triggered

## Core Apps (Implemented)

### 1. Drift Detector
**Job**: Keep actual state aligned with desired state
**Traditional Approach**: Periodic scripts that compare and alert
**Our Approach**: Continuous app with auto-remediation

```go
func (d *DriftDetector) Run() {
    app.RunWithInformers(func() error {
        // ConfigHub provides desired state (data)
        desired := d.Cub.ListUnits(d.SpaceID)

        // App queries actual state
        actual := d.K8s.GetCurrentState()

        // Intelligence determines fixes
        drift := d.ComputeDrift(desired, actual)
        fix := d.Claude.Analyze("Safe fix?", drift)

        // App takes action
        d.Cub.UpdateUnit(fix)
        d.K8s.Apply(fix)
    })
}
```

**ConfigHub Features Used**:
- **Sets**: Group critical services for monitoring
- **Filters**: Target specific resources (`WHERE tier='critical'`)
- **Live State**: Track actual deployment status
- **Push-upgrade**: Propagate fixes across environments

**Why Better**: Continuous monitoring vs periodic checks, auto-fix vs manual intervention

### 2. Cost Optimizer
**Job**: Minimize cloud spend while maintaining performance
**Traditional Approach**: Monthly reports and manual rightsizing
**Our Approach**: Continuous AI-powered optimization

```go
func (c *CostOptimizer) Run() {
    app.RunWithInformers(func() error {
        // Query high-cost resources (config as data)
        expensive := c.Cub.ListUnits(ListParams{
            Filter: "cost > 100 AND utilization < 50"
        })

        // AI analyzes optimization opportunities
        optimizations := c.Claude.Analyze(`
            Resources: ${expensive}
            SLAs: ${c.getSLAs()}
            Find safe optimizations
        `)

        // App applies optimizations
        for _, opt := range optimizations {
            if opt.Risk == "low" {
                c.Cub.UpdateUnit(opt.UnitID, opt.NewConfig)
                c.Cub.ApplyUnit(opt.UnitID)
            }
        }

        // Dashboard shows real-time savings
        c.Dashboard.Update(optimizations)
    })
}
```

**ConfigHub Features Used**:
- **Sets**: Group resources by cost tier
- **Filters**: Find optimization candidates
- **Upstream/downstream**: Test in QA before prod
- **Revision history**: Easy rollback if needed

**Why Better**: Continuous vs periodic, AI-driven vs rule-based, automatic application vs manual

## Planned Apps (Next Phase)

### 3. Security Scanner
**Job**: Find and fix vulnerabilities continuously
**Traditional Approach**: CI/CD scanning, manual patching
**Our Approach**: Continuous scanning with auto-remediation

```go
func (s *SecurityScanner) Run() {
    app.RunWithInformers(func() error {
        // Query deployed configs
        units := s.Cub.ListUnits(s.SpaceID)

        // Continuous vulnerability scanning
        for _, unit := range units {
            vulns := s.ScanForCVEs(unit)

            if vulns.Critical() {
                // AI determines safe fix
                fix := s.Claude.Analyze(`
                    CVEs: ${vulns}
                    Config: ${unit}
                    Generate safe patch
                `)

                // Apply fix immediately
                s.Cub.UpdateUnit(unit.UnitID, fix)
                s.Cub.ApplyUnit(unit.UnitID)

                // Push to all environments
                s.Cub.BulkPatchUnits(BulkPatchParams{
                    Where: fmt.Sprintf("UpstreamUnitID = '%s'", unit.UnitID),
                    Upgrade: true,
                })
            }
        }
    })
}
```

**ConfigHub Features Used**:
- **Bulk operations**: Patch all vulnerable instances
- **Push-upgrade**: Propagate security fixes
- **Sets**: Group by security tier
- **Audit trail**: Track all patches

**Why Better**: Immediate patching vs waiting for next deployment

### 4. Compliance Auditor
**Job**: Maintain continuous compliance (SOC2, PCI-DSS, etc.)
**Traditional Approach**: Periodic audits, manual remediation
**Our Approach**: Continuous compliance with auto-fix

```go
func (c *ComplianceAuditor) Run() {
    app.RunWithInformers(func() error {
        // Query all production configs
        prodConfigs := c.Cub.ListUnits(ListParams{
            Filter: "environment = 'prod'"
        })

        // Check compliance rules
        violations := c.CheckCompliance(prodConfigs, c.Rules)

        // AI determines fixes
        for _, violation := range violations {
            fix := c.Claude.Analyze(`
                Violation: ${violation}
                Generate compliant config
            `)

            // Apply fix
            c.Cub.UpdateUnit(violation.UnitID, fix)

            // Generate audit report
            c.GenerateAuditReport(violation, fix)
        }
    })
}
```

**ConfigHub Features Used**:
- **Filters**: Target compliance scope
- **Revision history**: Complete audit trail
- **Sets**: Group by compliance domain

**Why Better**: Continuous vs periodic, auto-remediation vs manual fixes

### 5. Capacity Planner
**Job**: Predict and prevent capacity issues
**Traditional Approach**: React to alerts, manual scaling
**Our Approach**: Predictive scaling with AI

```go
func (p *CapacityPlanner) Run() {
    app.RunWithInformers(func() error {
        // Query resource configs
        resources := p.Cub.ListUnits(ListParams{
            Filter: "type = 'deployment'"
        })

        // Gather metrics
        metrics := p.K8s.GetMetrics()

        // AI predicts needs
        forecast := p.Claude.Analyze(`
            Current: ${resources}
            Metrics: ${metrics}
            Trends: ${p.getTrends()}
            Predict next 24h needs
        `)

        // Proactive scaling
        if forecast.NeedsScaling {
            p.Cub.UpdateUnit(forecast.ScalingConfig)
            p.Cub.ApplyUnit(forecast.UnitID)
        }
    })
}
```

**ConfigHub Features Used**:
- **Sets**: Group scalable resources
- **Upstream/downstream**: Test scaling in staging
- **Push-upgrade**: Apply proven configs to prod

**Why Better**: Predictive vs reactive, AI-driven vs threshold-based

### 6. Upgrade Manager
**Job**: Safely upgrade all dependencies
**Traditional Approach**: Manual upgrades, hope nothing breaks
**Our Approach**: Intelligent upgrade orchestration

(See UPGRADE-MANAGER-USE-CASE.md for detailed implementation)

### 7. Branch Deployer
**Job**: Deploy feature branches for testing
**Traditional Approach**: Complex CI/CD pipelines
**Our Approach**: Persistent app managing branch environments

(See DEPLOY-FROM-BRANCH-PATTERN.md for detailed implementation)

## Common Patterns Across All Apps

### 1. Configuration as Queryable Data
```go
// Every app queries config as data
configs := app.Cub.ListUnits(ListParams{
    Filter: "complex SQL-like queries",
})
```

### 2. AI-Powered Decisions
```go
// Every app uses Claude for intelligence
decision := app.Claude.Analyze(context, data)
```

### 3. Continuous Operation
```go
// Every app runs continuously
app.RunWithInformers(func() error {
    // React to changes immediately
})
```

### 4. Bulk Operations via Sets
```go
// Every app can operate on groups
set := app.Cub.GetSet("critical-services")
app.Cub.BulkApplyUnits(BulkApplyParams{
    Where: fmt.Sprintf("SetID = '%s'", set.SetID),
})
```

### 5. Environment Propagation
```go
// Every app uses push-upgrade
app.Cub.BulkPatchUnits(BulkPatchParams{
    Where: "UpstreamUnitID = baseID",
    Upgrade: true,
})
```

## Why This Architecture Wins

### Traditional DevOps Tools:
- **Triggered**: Run when scheduled or manually invoked
- **Stateless**: No memory between runs
- **Isolated**: Each tool independent
- **Rule-based**: Static thresholds and conditions
- **File-based**: Configuration in scattered files

### DevOps as Apps with Config as Data:
- **Continuous**: Always running, always monitoring
- **Stateful**: Remember history and patterns
- **Integrated**: Share data through ConfigHub
- **Intelligent**: AI-powered decisions
- **Data-based**: Configuration as queryable database

## Implementation Priority

1. **Security Scanner** - Immediate value, clear ROI
2. **Compliance Auditor** - Required for enterprise
3. **Capacity Planner** - Prevents outages
4. **Upgrade Manager** - Reduces maintenance burden
5. **Branch Deployer** - Developer productivity

## Summary

Each DevOps job becomes a modern application that:
1. Treats configuration as data (stored in ConfigHub)
2. Runs continuously (not triggered)
3. Uses AI for decisions (Claude integration)
4. Operates in bulk (Sets and Filters)
5. Propagates changes (push-upgrade)

This is the future: **DevOps tools as real applications managing configuration as data**.