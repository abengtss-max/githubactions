# 🚀 GitHub Actions Certification - Quick Reference Guide

## 📋 Module 1: Automate Development Tasks

### Essential Commands Cheat Sheet

#### Git & GitHub
```powershell
# Clone repository
git clone https://github.com/USERNAME/REPO.git

# Create and push tags
git tag -a v1.0.0 -m "Release version 1.0.0"
git push origin v1.0.0

# Create branch
git checkout -b feature/new-feature
```

#### Azure CLI
```powershell
# Login to Azure
az login

# Create resource group
az group create --name rg-name --location eastus

# Create App Service
az webapp create --name app-name --resource-group rg-name --plan plan-name --runtime "NODE:20-lts"

# Create service principal
az ad sp create-for-rbac --name sp-name --role Contributor --scopes /subscriptions/{sub-id}/resourceGroups/{rg-name} --sdk-auth

# Delete resource group
az group delete --name rg-name --yes --no-wait
```

#### Docker
```powershell
# Build image
docker build -t action-name .

# Run container locally
docker run --rm -e INPUT_NAME="value" action-name

# Remove image
docker rmi action-name
```

#### NPM
```powershell
# Initialize project
npm init -y

# Install dependencies
npm install package-name
npm install --save-dev dev-package

# Run scripts
npm test
npm run lint
npm start
```

## 🔑 GitHub Secrets Setup

### Required Secrets for Labs

| Secret Name | Description | How to Get |
|-------------|-------------|------------|
| `AZURE_CREDENTIALS` | Service principal JSON | `az ad sp create-for-rbac --sdk-auth` |
| `AZURE_WEBAPP_NAME` | Web app name | From Azure portal or CLI |
| `AZURE_RESOURCE_GROUP` | Resource group name | From Azure portal or CLI |
| `AZURE_WEBAPP_PUBLISH_PROFILE` | Publish profile XML | `az webapp deployment list-publishing-profiles` |

### Adding Secrets to GitHub
1. Go to repository **Settings**
2. Click **Secrets and variables** → **Actions**
3. Click **"New repository secret"**
4. Enter name and value
5. Click **"Add secret"**

## 📁 Workflow File Locations

```
.github/
└── workflows/
    ├── ci.yml              # Continuous Integration
    ├── cd.yml              # Continuous Deployment
    ├── scheduled.yml       # Scheduled tasks
    └── manual.yml          # Manual workflows
```

## 🎯 Common Workflow Patterns

### Basic CI Workflow
```yaml
name: CI
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npm test
```

### Azure Deployment
```yaml
- name: Azure Login
  uses: azure/login@v1
  with:
    creds: ${{ secrets.AZURE_CREDENTIALS }}

- name: Deploy to Azure
  uses: azure/webapps-deploy@v2
  with:
    app-name: ${{ secrets.AZURE_WEBAPP_NAME }}
    package: .
```

### Manual Trigger
```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment'
        required: true
        type: choice
        options: [dev, staging, prod]
```

### Scheduled Workflow
```yaml
on:
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight
```

## 🐛 Troubleshooting Quick Fixes

### Workflow Not Running
- ✅ Check workflow is on default branch
- ✅ Verify YAML syntax (use YAML validator)
- ✅ Check trigger event configuration
- ✅ Look for workflow errors in Actions tab

### Azure Authentication Failed
- ✅ Verify `AZURE_CREDENTIALS` secret exists
- ✅ Check service principal hasn't expired
- ✅ Confirm correct subscription ID
- ✅ Verify role assignments (Contributor role)

### Container Action Failing
- ✅ Test Docker image locally first
- ✅ Check Dockerfile syntax
- ✅ Verify entrypoint.sh is executable (`chmod +x`)
- ✅ Check file line endings (LF not CRLF)

### Out of Minutes
- ✅ Use ubuntu-latest (1x multiplier)
- ✅ Implement path filtering
- ✅ Add concurrency controls
- ✅ Cache dependencies

## 📊 Lab Completion Checklist

- [ ] **Lab 1:** GitHub Actions Fundamentals (30 min)
  - [ ] Explored Actions tab and Marketplace
  - [ ] Created first workflow
  - [ ] Tested different triggers
  
- [ ] **Lab 2:** First Workflow (45 min)
  - [ ] Built Node.js application
  - [ ] Created CI workflow
  - [ ] Deployed to Azure App Service
  
- [ ] **Lab 3:** Container Actions (60 min)
  - [ ] Built Docker container action
  - [ ] Integrated Azure CLI
  - [ ] Published action with versioning
  
- [ ] **Lab 4:** Advanced Triggers (45 min)
  - [ ] Implemented scheduled workflows
  - [ ] Created manual workflows with inputs
  - [ ] Used repository_dispatch
  
- [ ] **Lab 5:** JavaScript Actions (60 min)
  - [ ] Built JavaScript action
  - [ ] Integrated Azure SDK
  - [ ] Published to GitHub
  
- [ ] **Lab 6:** Complete Pipeline (90 min)
  - [ ] Built production-grade pipeline
  - [ ] Implemented blue-green deployment
  - [ ] Added monitoring

- [ ] **Lab 7:** GitHub Script (45 min)
  - [ ] Used GitHub Script for automation
  - [ ] Automated issue management
  - [ ] Integrated with Azure

- [ ] **Lab 8:** GitHub Packages (60 min)
  - [ ] Published npm package
  - [ ] Published Docker image to GHCR
  - [ ] Published to Azure Container Registry

- [ ] **Lab 9:** Reusable Workflows (55 min)
  - [ ] Created reusable workflow
  - [ ] Implemented artifact sharing
  - [ ] Advanced caching strategies

- [ ] **Lab 10:** Enterprise & Runners (70 min)
  - [ ] Set up self-hosted runner
  - [ ] Configured runner groups
  - [ ] Implemented enterprise features

## 🎓 Key Concepts Summary

### GitHub Actions Components
- **Workflow:** Automated process defined in YAML
- **Event:** Trigger that starts a workflow
- **Job:** Set of steps executed on same runner
- **Step:** Individual task (action or command)
- **Action:** Reusable unit of code
- **Runner:** Server executing workflows

### Action Types
- **Container:** Docker-based, Linux only, consistent environment
- **JavaScript:** Node.js-based, cross-platform, faster
- **Composite:** Combines multiple steps

### Best Practices
✅ Use specific action versions (@v4)  
✅ Store secrets in GitHub Secrets  
✅ Implement path filtering  
✅ Add timeout-minutes  
✅ Use concurrency controls  
✅ Cache dependencies  
✅ Add health checks  

## 🔗 Essential Links

- [GitHub Actions Docs](https://docs.github.com/en/actions)
- [GitHub Marketplace](https://github.com/marketplace?type=actions)
- [Azure Documentation](https://docs.microsoft.com/azure/)
- [Microsoft Learn Path](https://learn.microsoft.com/training/paths/github-actions/)
- [Azure CLI Reference](https://docs.microsoft.com/cli/azure/)

## 💡 Tips & Tricks

### Speed Up Workflows
```yaml
# Cache npm dependencies
- uses: actions/setup-node@v4
  with:
    cache: 'npm'

# Cancel previous runs
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

### Debugging Workflows
```yaml
# Enable debug logging
- run: echo "::debug::Debug message"

# Set secret for runner diagnostic logs
# Add ACTIONS_RUNNER_DEBUG=true to secrets

# Save state between steps
- run: echo "key=value" >> $GITHUB_OUTPUT
  id: step1
- run: echo "${{ steps.step1.outputs.key }}"
```

### Cost Optimization
```yaml
# Skip CI for docs changes
on:
  push:
    paths-ignore:
      - '**.md'
      - 'docs/**'

# Set job timeout
jobs:
  build:
    timeout-minutes: 10
```

## 📞 Support Resources

### Lab-Specific Help
- Each lab has troubleshooting section
- Check knowledge check answers
- Review code examples

### General Help
- GitHub Actions logs (Actions tab)
- Azure Portal diagnostics
- Official documentation
- GitHub Community forum

## 🎯 Next Steps

1. ✅ Complete all 10 labs
2. ✅ Build personal projects using GitHub Actions
3. ✅ Implement learned concepts in your organization
4. ✅ Take practice certification exam
5. ✅ Schedule and pass certification exam

---

## 📖 Quick Navigation

- [Getting Started Guide](./GETTING-STARTED.md)
- [Complete Labs Overview](./LABS-OVERVIEW.md)
- [Lab 1: Fundamentals](./Lab1-GitHub-Actions-Fundamentals/README.md)
- [Lab 7: GitHub Script](./Lab7-GitHub-Script/README.md)
- [Lab 8: GitHub Packages](./Lab8-GitHub-Packages/README.md)
- [Lab 9: Reusable Workflows](./Lab9-Reusable-Workflows/README.md)
- [Lab 10: Enterprise & Runners](./Lab10-Enterprise-Runners/README.md)

---

**Print this guide for quick reference during labs!**

*Keep this handy as you work through the certification materials.*
