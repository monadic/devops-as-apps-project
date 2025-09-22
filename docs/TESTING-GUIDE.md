# Testing Guide - DevOps as Apps Project

## Testing Philosophy

This project follows a **2-step testing protocol** to ensure reliability while maintaining fast development cycles:

1. **Local Tests First** - Fast feedback with mocks and simulations
2. **Real Integration Tests** - Validate against actual ConfigHub and Kubernetes

## Step 1: Local Tests First ⚡

### Prerequisites
- Go 1.21+
- No external dependencies required

### Commands
```bash
cd devops-examples/drift-detector

# Build and verify compilation
go build .

# Unit tests (fast, no external calls)
go test -v

# Demo mode (simulated end-to-end workflow)
./drift-detector demo

# Code quality
go fmt ./...
go vet ./...
```

### What This Tests
- ✅ Code compilation and syntax
- ✅ Unit logic (drift comparison, state parsing)
- ✅ Data structure validation
- ✅ ConfigHub API type correctness
- ✅ End-to-end workflow simulation
- ✅ Real vs hallucinated API verification

### Expected Output
```
=== RUN   TestCompareStates
--- PASS: TestCompareStates (0.00s)
=== RUN   TestConfigHubRealAPIUsage
    main_test.go:263: Using 13 real ConfigHub APIs
    main_test.go:264: Avoiding 6 hallucinated APIs
--- PASS: TestConfigHubRealAPIUsage (0.00s)
PASS
```

## Step 2: Real Integration Tests 🔗

### Prerequisites
- ConfigHub account and token from https://confighub.com
- Docker and Kind installed
- kubectl configured

### Environment Setup
```bash
# ConfigHub credentials
export CUB_TOKEN="your-confighub-token-here"
export CUB_API_URL="https://confighub.com/api/v1"

# Optional: Claude AI for enhanced analysis
export CLAUDE_API_KEY="your-claude-key-here"

# Local Kubernetes cluster
kind create cluster --name devops-test
kubectl cluster-info --context kind-devops-test
```

### Integration Test Commands
```bash
# Test real ConfigHub API calls
go test -tags=integration -v

# Deploy to Kind cluster and test end-to-end
kubectl apply -f k8s/  # when K8s manifests exist
./drift-detector       # Real ConfigHub + Kind integration
```

### What This Tests
- ✅ Real ConfigHub API connectivity
- ✅ Space, Set, Filter creation
- ✅ Bulk operations (BulkPatchUnits, BulkApplyUnits)
- ✅ Push-upgrade pattern
- ✅ Kubernetes informer event handling
- ✅ Live state monitoring
- ✅ Claude AI integration (if configured)

### Expected Output
```bash
# Integration tests
=== RUN   TestConfigHubAPIOperations
--- PASS: TestConfigHubAPIOperations (2.34s)

# Real deployment
[drift-detector] 2024/09/22 12:00:00 Initializing ConfigHub resources...
[drift-detector] 2024/09/22 12:00:01 Created new space: a1b2c3d4-...
[drift-detector] 2024/09/22 12:00:02 Created critical services set: e5f6g7h8-...
[drift-detector] 2024/09/22 12:00:03 Informers started, watching for changes...
```

## Test Categories

### Unit Tests
**Location**: `*_test.go` files
**Run**: `go test -v`
**Purpose**: Fast validation of individual components

### Integration Tests
**Location**: `integration_test.go`
**Run**: `go test -tags=integration -v`
**Purpose**: Real API and infrastructure testing

### Demo Mode
**Location**: `demo.go`
**Run**: `./app-name demo`
**Purpose**: End-to-end workflow simulation

### Benchmarks
**Location**: `*_test.go` with `Benchmark` prefix
**Run**: `go test -bench=.`
**Purpose**: Performance validation

## Testing New DevOps Apps

When creating new DevOps apps (cost-optimizer, security-scanner, etc.), follow this pattern:

### 1. Start with Local Tests
```bash
# Create the app
mkdir devops-examples/cost-optimizer
cd devops-examples/cost-optimizer

# Implement with tests first
go test -v  # Should pass immediately

# Add demo mode
./cost-optimizer demo
```

### 2. Add Integration Tests
```bash
# Test real ConfigHub integration
go test -tags=integration -v

# Deploy to Kind and verify
./cost-optimizer  # With real ConfigHub + Kind
```

## Troubleshooting

### Common Issues

#### "CUB_TOKEN not set"
```bash
# Get token from https://confighub.com
export CUB_TOKEN="your-actual-token"
```

#### "Kind cluster not found"
```bash
kind create cluster --name devops-test
kubectl config use-context kind-devops-test
```

#### "API error 401"
```bash
# Verify token is valid
curl -H "Authorization: Bearer $CUB_TOKEN" https://confighub.com/api/v1/spaces
```

#### "No drift detected"
```bash
# This is normal - means everything is in sync
# To test drift, manually change a deployment:
kubectl scale deployment backend-api --replicas=5
```

## Best Practices

### Test Organization
- Keep unit tests fast (< 100ms each)
- Use `t.Parallel()` for independent tests
- Mock external dependencies in unit tests
- Use real services only in integration tests

### Data Management
- Use `uuid.New()` for test IDs to avoid conflicts
- Clean up resources in integration tests
- Use dedicated test namespaces in Kubernetes

### Error Handling
- Test both success and failure paths
- Verify error messages are helpful
- Test retry and timeout behaviors

### Documentation
- Document expected test outputs
- Include setup instructions for new contributors
- Update this guide when adding new test patterns

## CI/CD Integration

### GitHub Actions (Future)
```yaml
name: Test
on: [push, pull_request]
jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - run: go test -v

  integration-tests:
    runs-on: ubuntu-latest
    env:
      CUB_TOKEN: ${{ secrets.CUB_TOKEN }}
    steps:
      - run: kind create cluster
      - run: go test -tags=integration -v
```

This testing approach ensures we catch issues early while maintaining confidence in production deployments.