# Development Setup

## Repository Structure

This project uses three separate repositories:

1. **devops-as-apps-project** (this repo)
   - Planning and documentation
   - Architecture decisions
   - Claude Code configuration

2. **[devops-sdk](https://github.com/monadic/devops-sdk)**
   - Reusable Go SDK
   - ConfigHub client (real APIs only)
   - Kubernetes utilities
   - Base app framework

3. **[devops-examples](https://github.com/monadic/devops-examples)**
   - Drift Detector
   - Cost Optimizer
   - Upgrade Manager
   - Other DevOps apps

## Local Development

Clone all three repos side by side:
```bash
cd ~/github-repos
git clone https://github.com/monadic/devops-as-apps-project
git clone https://github.com/monadic/devops-sdk
git clone https://github.com/monadic/devops-examples
```

## Working with the SDK

In devops-examples apps:
```go
import (
    "github.com/monadic/devops-sdk/app"
    "github.com/monadic/devops-sdk/confighub"
)
```

## Go Module Setup

Each example app should have:
```go
module github.com/monadic/devops-examples/drift-detector

require github.com/monadic/devops-sdk v1.0.0
```

## Testing Changes Locally

To test SDK changes before pushing:
```bash
# In devops-examples/drift-detector
go mod edit -replace github.com/monadic/devops-sdk=../../devops-sdk
go mod tidy
```

Remember to remove the replace directive before committing!