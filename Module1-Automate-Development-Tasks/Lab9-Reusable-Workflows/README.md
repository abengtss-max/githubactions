# Lab 9: Reusable Workflows & Artifacts

**Duration:** 55 minutes  
**Difficulty:** Intermediate to Advanced  
**Prerequisites:** Lab 2 (CI/CD) and Lab 6 (Complete Pipeline)

## 🎯 Learning Objectives

By the end of this lab, you will be able to:
- Create reusable workflows using `workflow_call`
- Share workflows across multiple repositories
- Pass inputs and secrets to reusable workflows
- Use workflow outputs effectively
- Upload and download build artifacts
- Share artifacts between jobs
- Implement advanced caching strategies
- Use matrix builds with artifact collection
- Cache dependencies efficiently with `actions/cache`

---

## 📖 What are Reusable Workflows?

**Reusable workflows** allow you to call an entire workflow from another workflow, eliminating duplication and promoting consistency across projects.

### Why Use Reusable Workflows?

**Problems with duplicated workflows:**
- 🔴 Copy-paste same workflow across repositories
- 🔴 Updates require changing multiple files
- 🔴 Inconsistent implementations
- 🔴 Maintenance nightmare

**Benefits of reusable workflows:**
- ✅ DRY (Don't Repeat Yourself) principle
- ✅ Centralized maintenance
- ✅ Consistent CI/CD across organization
- ✅ Easier to update and improve
- ✅ Better governance

### Reusable Workflow vs Composite Action

| Feature | Reusable Workflow | Composite Action |
|---------|-------------------|------------------|
| **Scope** | Entire workflow | Individual step |
| **Jobs** | Multiple jobs | Single job steps |
| **Runners** | Can specify | Uses parent runner |
| **Secrets** | Can inherit/pass | Must pass explicitly |
| **Best For** | Complete processes | Reusable step logic |

---

## 📖 What are Artifacts?

**Artifacts** are files produced during workflow execution that can be:
- Shared between jobs
- Downloaded after workflow completes
- Used for debugging
- Deployed to production

### Common Artifact Use Cases

- 📦 Build outputs (compiled code, binaries)
- 📊 Test results and coverage reports
- 📝 Logs and diagnostic information
- 🎨 Generated documentation
- 📱 Mobile app packages (APK, IPA)
- 🌐 Static website builds

---

## 🏗️ Lab Architecture

```
Reusable Workflow Repository
          ↓
    workflow_call trigger
          ↓
    Called by projects → Job 1 → Artifact → Job 2 → Deploy
                         ↓                    ↑
                       Cache ← → Cache → Job 3
```

---

## 📋 Prerequisites

Before starting this lab:

1. ✅ Completed Lab 2 (CI/CD workflows)
2. ✅ Completed Lab 6 (Complete pipeline)
3. ✅ GitHub account with multiple repositories
4. ✅ Understanding of workflow syntax
5. ✅ (Optional) Azure subscription for deployment examples

---

## 🚀 Exercise 1: Create Your First Reusable Workflow

### What You'll Build
A centralized Node.js CI workflow that can be reused across multiple projects.

### Step 1: Create Shared Workflows Repository

1. **Create repository** named `github-workflows`:
   ```powershell
   cd C:\Users\alibengtsson\mycertifications\githubactions
   mkdir github-workflows
   cd github-workflows
   git init
   ```

2. **Create reusable workflow directory**:
   ```powershell
   mkdir -p .github/workflows
   ```

### Step 2: Create Reusable Node.js CI Workflow

Create `.github/workflows/nodejs-ci.yml`:

```yaml
name: Reusable Node.js CI

on:
  workflow_call:
    inputs:
      node-version:
        description: 'Node.js version to use'
        required: false
        type: string
        default: '20'
      
      working-directory:
        description: 'Working directory for the project'
        required: false
        type: string
        default: '.'
      
      run-lint:
        description: 'Whether to run linting'
        required: false
        type: boolean
        default: true
      
      run-tests:
        description: 'Whether to run tests'
        required: false
        type: boolean
        default: true
      
      upload-coverage:
        description: 'Whether to upload coverage reports'
        required: false
        type: boolean
        default: false
    
    outputs:
      test-result:
        description: 'Test execution result'
        value: ${{ jobs.test.outputs.result }}
      
      coverage:
        description: 'Test coverage percentage'
        value: ${{ jobs.test.outputs.coverage }}
    
    secrets:
      CODECOV_TOKEN:
        description: 'Codecov token for coverage upload'
        required: false

jobs:
  install:
    name: Install Dependencies
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ${{ inputs.working-directory }}
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          cache: 'npm'
          cache-dependency-path: '${{ inputs.working-directory }}/package-lock.json'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Cache node_modules
        uses: actions/cache/save@v3
        with:
          path: ${{ inputs.working-directory }}/node_modules
          key: ${{ runner.os }}-node-${{ inputs.node-version }}-${{ hashFiles(format('{0}/package-lock.json', inputs.working-directory)) }}

  lint:
    name: Lint Code
    runs-on: ubuntu-latest
    needs: install
    if: ${{ inputs.run-lint }}
    defaults:
      run:
        working-directory: ${{ inputs.working-directory }}
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
      
      - name: Restore node_modules
        uses: actions/cache/restore@v3
        with:
          path: ${{ inputs.working-directory }}/node_modules
          key: ${{ runner.os }}-node-${{ inputs.node-version }}-${{ hashFiles(format('{0}/package-lock.json', inputs.working-directory)) }}
      
      - name: Run ESLint
        run: npm run lint --if-present

  test:
    name: Run Tests
    runs-on: ubuntu-latest
    needs: install
    if: ${{ inputs.run-tests }}
    outputs:
      result: ${{ steps.test.outcome }}
      coverage: ${{ steps.coverage.outputs.percentage }}
    defaults:
      run:
        working-directory: ${{ inputs.working-directory }}
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
      
      - name: Restore node_modules
        uses: actions/cache/restore@v3
        with:
          path: ${{ inputs.working-directory }}/node_modules
          key: ${{ runner.os }}-node-${{ inputs.node-version }}-${{ hashFiles(format('{0}/package-lock.json', inputs.working-directory)) }}
      
      - name: Run tests
        id: test
        run: npm test
      
      - name: Generate coverage report
        if: ${{ inputs.upload-coverage }}
        run: npm run test:coverage --if-present
      
      - name: Extract coverage percentage
        if: ${{ inputs.upload-coverage }}
        id: coverage
        run: |
          COVERAGE=$(cat coverage/coverage-summary.json | jq '.total.lines.pct')
          echo "percentage=$COVERAGE" >> $GITHUB_OUTPUT
      
      - name: Upload coverage to Codecov
        if: ${{ inputs.upload-coverage && secrets.CODECOV_TOKEN != '' }}
        uses: codecov/codecov-action@v3
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          directory: ${{ inputs.working-directory }}/coverage
      
      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: ${{ inputs.working-directory }}/test-results/
          retention-days: 7

  build:
    name: Build Application
    runs-on: ubuntu-latest
    needs: [lint, test]
    if: always() && needs.test.result == 'success'
    defaults:
      run:
        working-directory: ${{ inputs.working-directory }}
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
      
      - name: Restore node_modules
        uses: actions/cache/restore@v3
        with:
          path: ${{ inputs.working-directory }}/node_modules
          key: ${{ runner.os }}-node-${{ inputs.node-version }}-${{ hashFiles(format('{0}/package-lock.json', inputs.working-directory)) }}
      
      - name: Build application
        run: npm run build --if-present
      
      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: |
            ${{ inputs.working-directory }}/dist/
            ${{ inputs.working-directory }}/build/
          retention-days: 30
```

### Step 3: Create README

Create `README.md`:

```markdown
# Shared GitHub Actions Workflows

Centralized, reusable workflows for our projects.

## Available Workflows

### Node.js CI (`nodejs-ci.yml`)

Runs linting, testing, and building for Node.js projects.

#### Usage

```yaml
name: CI

on: [push, pull_request]

jobs:
  ci:
    uses: YOUR_USERNAME/github-workflows/.github/workflows/nodejs-ci.yml@main
    with:
      node-version: '20'
      run-lint: true
      run-tests: true
      upload-coverage: true
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

#### Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `node-version` | Node.js version | No | `'20'` |
| `working-directory` | Project directory | No | `'.'` |
| `run-lint` | Run linting | No | `true` |
| `run-tests` | Run tests | No | `true` |
| `upload-coverage` | Upload coverage | No | `false` |

#### Outputs

- `test-result`: Test execution result
- `coverage`: Coverage percentage

#### Secrets

- `CODECOV_TOKEN`: Optional Codecov token
```

### Step 4: Commit and Push

```powershell
git add .
git commit -m "Add reusable Node.js CI workflow"
git branch -M main

# Create repository on GitHub, then:
git remote add origin https://github.com/YOUR_USERNAME/github-workflows.git
git push -u origin main
```

### Step 5: Use the Reusable Workflow

In any Node.js project:

Create `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  ci:
    name: Run CI
    uses: YOUR_USERNAME/github-workflows/.github/workflows/nodejs-ci.yml@main
    with:
      node-version: '20'
      run-lint: true
      run-tests: true
      upload-coverage: false
  
  results:
    name: Show Results
    runs-on: ubuntu-latest
    needs: ci
    steps:
      - name: Display test results
        run: |
          echo "Test Result: ${{ needs.ci.outputs.test-result }}"
          echo "Coverage: ${{ needs.ci.outputs.coverage }}%"
```

**Replace `YOUR_USERNAME`** with your GitHub username.

---

## 🚀 Exercise 2: Advanced Artifact Management

### What You'll Build
A workflow that builds artifacts, shares them between jobs, and creates deployment packages.

### Step 1: Create Multi-Job Build with Artifacts

Create a new project `artifact-demo`:

```powershell
cd C:\Users\alibengtsson\mycertifications\githubactions
mkdir artifact-demo
cd artifact-demo
npm init -y
```

Create `.github/workflows/build-with-artifacts.yml`:

```yaml
name: Build with Artifacts

on:
  push:
    branches: [main]

jobs:
  # Job 1: Build frontend
  build-frontend:
    name: Build Frontend
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build frontend
        run: |
          mkdir -p dist/frontend
          echo '<h1>Frontend Build</h1>' > dist/frontend/index.html
          echo '/* Frontend CSS */' > dist/frontend/styles.css
          echo '// Frontend JS' > dist/frontend/app.js
      
      - name: Upload frontend artifact
        uses: actions/upload-artifact@v4
        with:
          name: frontend-build
          path: dist/frontend/
          retention-days: 7
  
  # Job 2: Build backend
  build-backend:
    name: Build Backend
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build backend
        run: |
          mkdir -p dist/backend
          echo 'console.log("Backend Server");' > dist/backend/server.js
          echo '{"version": "1.0.0"}' > dist/backend/config.json
      
      - name: Upload backend artifact
        uses: actions/upload-artifact@v4
        with:
          name: backend-build
          path: dist/backend/
          retention-days: 7
  
  # Job 3: Run tests (depends on builds)
  test:
    name: Run Tests
    runs-on: ubuntu-latest
    needs: [build-frontend, build-backend]
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Download frontend artifact
        uses: actions/download-artifact@v4
        with:
          name: frontend-build
          path: dist/frontend/
      
      - name: Download backend artifact
        uses: actions/download-artifact@v4
        with:
          name: backend-build
          path: dist/backend/
      
      - name: List downloaded artifacts
        run: |
          echo "Frontend files:"
          ls -R dist/frontend/
          echo ""
          echo "Backend files:"
          ls -R dist/backend/
      
      - name: Run integration tests
        run: |
          echo "Testing frontend..."
          test -f dist/frontend/index.html && echo "✓ Frontend OK"
          echo "Testing backend..."
          test -f dist/backend/server.js && echo "✓ Backend OK"
      
      - name: Generate test report
        run: |
          mkdir -p test-results
          echo "Test Report - $(date)" > test-results/report.txt
          echo "All tests passed!" >> test-results/report.txt
      
      - name: Upload test results
        uses: actions/upload-artifact@v4
        with:
          name: test-results
          path: test-results/
          retention-days: 30
  
  # Job 4: Create deployment package
  package:
    name: Create Deployment Package
    runs-on: ubuntu-latest
    needs: test
    
    steps:
      - name: Download all artifacts
        uses: actions/download-artifact@v4
        with:
          path: artifacts/
      
      - name: List all artifacts
        run: ls -R artifacts/
      
      - name: Create deployment package
        run: |
          mkdir -p deploy-package
          cp -r artifacts/frontend-build deploy-package/frontend
          cp -r artifacts/backend-build deploy-package/backend
          
          # Create deployment manifest
          cat > deploy-package/manifest.json << EOF
          {
            "version": "1.0.0",
            "timestamp": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
            "commit": "${{ github.sha }}",
            "components": {
              "frontend": "included",
              "backend": "included"
            }
          }
          EOF
          
          # Create archive
          tar -czf deploy-package.tar.gz -C deploy-package .
      
      - name: Upload deployment package
        uses: actions/upload-artifact@v4
        with:
          name: deployment-package
          path: deploy-package.tar.gz
          retention-days: 90
      
      - name: Generate deployment info
        run: |
          echo "Deployment package created successfully!"
          echo "Package size: $(du -h deploy-package.tar.gz | cut -f1)"
          echo "Download from: Actions → Artifacts → deployment-package"
```

### Step 2: Matrix Build with Artifact Collection

Create `.github/workflows/matrix-artifacts.yml`:

```yaml
name: Matrix Build with Artifacts

on:
  push:
    branches: [main]

jobs:
  build:
    name: Build on ${{ matrix.os }} - Node ${{ matrix.node }}
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node: ['18', '20']
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js ${{ matrix.node }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      
      - name: Build application
        run: |
          mkdir -p build
          echo "Built on ${{ matrix.os }} with Node ${{ matrix.node }}" > build/info.txt
      
      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: build-${{ matrix.os }}-node${{ matrix.node }}
          path: build/
  
  collect:
    name: Collect All Builds
    runs-on: ubuntu-latest
    needs: build
    
    steps:
      - name: Download all artifacts
        uses: actions/download-artifact@v4
        with:
          path: all-builds/
      
      - name: Create summary
        run: |
          echo "# Build Summary" > summary.md
          echo "" >> summary.md
          for dir in all-builds/*/; do
            echo "## $(basename "$dir")" >> summary.md
            cat "$dir/info.txt" >> summary.md
            echo "" >> summary.md
          done
          cat summary.md
      
      - name: Upload summary
        uses: actions/upload-artifact@v4
        with:
          name: build-summary
          path: summary.md
```

---

## 🚀 Exercise 3: Advanced Caching Strategies

### What You'll Build
Implement sophisticated caching for dependencies, build outputs, and computed results.

### Step 1: Multi-Layer Caching

Create `.github/workflows/advanced-caching.yml`:

```yaml
name: Advanced Caching

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    name: Build with Multi-Layer Cache
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      # Layer 1: npm cache (automatic via setup-node)
      - name: Setup Node with cache
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      # Layer 2: node_modules cache
      - name: Cache node_modules
        id: cache-node-modules
        uses: actions/cache@v3
        with:
          path: node_modules
          key: ${{ runner.os }}-node-modules-${{ hashFiles('package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-modules-
      
      # Layer 3: Build cache
      - name: Cache build output
        id: cache-build
        uses: actions/cache@v3
        with:
          path: |
            dist/
            .next/cache/
            .cache/
          key: ${{ runner.os }}-build-${{ hashFiles('src/**/*') }}-${{ hashFiles('package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-build-${{ hashFiles('src/**/*') }}-
            ${{ runner.os }}-build-
      
      - name: Build application
        if: steps.cache-build.outputs.cache-hit != 'true'
        run: npm run build --if-present
      
      - name: Cache hit info
        run: |
          echo "node_modules cache hit: ${{ steps.cache-node-modules.outputs.cache-hit }}"
          echo "Build cache hit: ${{ steps.cache-build.outputs.cache-hit }}"
```

### Step 2: Conditional Caching

Create `.github/workflows/conditional-cache.yml`:

```yaml
name: Conditional Caching

on:
  push:
    branches: [main]

jobs:
  build:
    name: Smart Cache Strategy
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 2  # Need previous commit for comparison
      
      - name: Check if package-lock changed
        id: package-changed
        run: |
          if git diff HEAD^ HEAD --name-only | grep -q 'package-lock.json'; then
            echo "changed=true" >> $GITHUB_OUTPUT
          else
            echo "changed=false" >> $GITHUB_OUTPUT
          fi
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Restore cache
        if: steps.package-changed.outputs.changed == 'false'
        uses: actions/cache/restore@v3
        with:
          path: node_modules
          key: ${{ runner.os }}-node-modules-${{ hashFiles('package-lock.json') }}
      
      - name: Install dependencies
        run: npm ci
      
      - name: Save cache
        if: steps.package-changed.outputs.changed == 'true'
        uses: actions/cache/save@v3
        with:
          path: node_modules
          key: ${{ runner.os }}-node-modules-${{ hashFiles('package-lock.json') }}
```

### Step 3: Cache Cleanup

Create `.github/workflows/cache-cleanup.yml`:

```yaml
name: Cache Cleanup

on:
  schedule:
    - cron: '0 2 * * 0'  # Weekly on Sunday at 2 AM
  workflow_dispatch:

jobs:
  cleanup:
    name: Clean Old Caches
    runs-on: ubuntu-latest
    permissions:
      actions: write
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Cleanup old caches
        run: |
          # This is a placeholder - GitHub auto-manages cache eviction
          # Caches are automatically removed after 7 days of no access
          # or when total cache size exceeds 10 GB
          
          echo "GitHub automatically removes:"
          echo "- Caches not accessed in 7 days"
          echo "- Oldest caches when limit exceeds 10 GB"
          echo ""
          echo "Current cache usage:"
          gh api /repos/${{ github.repository }}/actions/cache/usage --jq '.active_caches_count'
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## 🚀 Exercise 4: Reusable Workflow with Artifacts

### What You'll Build
A reusable deployment workflow that downloads artifacts and deploys to Azure.

### Step 1: Create Reusable Deploy Workflow

In `github-workflows` repository, create `.github/workflows/azure-deploy.yml`:

```yaml
name: Reusable Azure Deployment

on:
  workflow_call:
    inputs:
      environment:
        description: 'Environment to deploy to'
        required: true
        type: string
      
      artifact-name:
        description: 'Name of artifact to deploy'
        required: true
        type: string
      
      app-name:
        description: 'Azure App Service name'
        required: true
        type: string
      
      resource-group:
        description: 'Azure resource group'
        required: true
        type: string
    
    secrets:
      AZURE_CREDENTIALS:
        description: 'Azure service principal credentials'
        required: true
    
    outputs:
      deployment-url:
        description: 'Deployed application URL'
        value: ${{ jobs.deploy.outputs.url }}

jobs:
  deploy:
    name: Deploy to Azure
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    outputs:
      url: ${{ steps.deploy.outputs.webapp-url }}
    
    steps:
      - name: Download artifact
        uses: actions/download-artifact@v4
        with:
          name: ${{ inputs.artifact-name }}
          path: ./deploy
      
      - name: List deployment files
        run: ls -R ./deploy
      
      - name: Azure Login
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      
      - name: Deploy to Azure Web App
        id: deploy
        uses: azure/webapps-deploy@v2
        with:
          app-name: ${{ inputs.app-name }}
          resource-group-name: ${{ inputs.resource-group }}
          package: ./deploy
      
      - name: Get deployment URL
        run: |
          URL="https://${{ inputs.app-name }}.azurewebsites.net"
          echo "webapp-url=$URL" >> $GITHUB_OUTPUT
          echo "Deployed to: $URL"
```

### Step 2: Use Reusable Deploy Workflow

In your application repository:

```yaml
name: CI/CD with Reusable Workflows

on:
  push:
    branches: [main]

jobs:
  build:
    name: Build
    uses: YOUR_USERNAME/github-workflows/.github/workflows/nodejs-ci.yml@main
    with:
      node-version: '20'
      run-tests: true
  
  deploy-staging:
    name: Deploy to Staging
    needs: build
    uses: YOUR_USERNAME/github-workflows/.github/workflows/azure-deploy.yml@main
    with:
      environment: staging
      artifact-name: build-output
      app-name: ${{ vars.STAGING_APP_NAME }}
      resource-group: ${{ vars.AZURE_RESOURCE_GROUP }}
    secrets:
      AZURE_CREDENTIALS: ${{ secrets.AZURE_CREDENTIALS }}
  
  deploy-prod:
    name: Deploy to Production
    needs: deploy-staging
    if: github.ref == 'refs/heads/main'
    uses: YOUR_USERNAME/github-workflows/.github/workflows/azure-deploy.yml@main
    with:
      environment: production
      artifact-name: build-output
      app-name: ${{ vars.PROD_APP_NAME }}
      resource-group: ${{ vars.AZURE_RESOURCE_GROUP }}
    secrets:
      AZURE_CREDENTIALS: ${{ secrets.AZURE_CREDENTIALS }}
  
  results:
    name: Show Results
    runs-on: ubuntu-latest
    needs: [deploy-staging, deploy-prod]
    if: always()
    
    steps:
      - name: Display deployment URLs
        run: |
          echo "Staging URL: ${{ needs.deploy-staging.outputs.deployment-url }}"
          echo "Production URL: ${{ needs.deploy-prod.outputs.deployment-url }}"
```

---

## 📚 Best Practices

### 1. Version Your Reusable Workflows

```yaml
# Use specific version tag
uses: org/workflows/.github/workflows/ci.yml@v1.2.3

# Or use branch for latest
uses: org/workflows/.github/workflows/ci.yml@main
```

### 2. Artifact Naming Convention

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: build-${{ github.run_number }}-${{ github.sha }}
    path: dist/
```

### 3. Set Appropriate Retention

```yaml
retention-days: 7    # Temporary artifacts
retention-days: 30   # Test results
retention-days: 90   # Release builds
```

### 4. Cache Key Strategy

```yaml
key: ${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}-v1
restore-keys: |
  ${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}-
  ${{ runner.os }}-
```

### 5. Limit Artifact Size

```yaml
- name: Upload optimized artifact
  uses: actions/upload-artifact@v4
  with:
    name: build
    path: |
      dist/
      !dist/**/*.map
      !dist/**/*.test.js
```

---

## 🔍 Troubleshooting

### Issue: Artifact Not Found

**Symptom:** `Unable to download artifact`

**Solution:**
- Verify artifact name matches exactly
- Check artifact was uploaded successfully
- Ensure artifact hasn't expired (check retention-days)

### Issue: Cache Not Restoring

**Symptom:** Cache always misses

**Solution:**
- Verify cache key matches
- Check restore-keys for fallbacks
- Ensure path is correct
- Cache might be evicted (>7 days old or >10GB total)

### Issue: Reusable Workflow Not Found

**Symptom:** `workflow was not found`

**Solution:**
- Ensure workflow file is in `.github/workflows/`
- Check repository is accessible
- Verify branch/tag exists
- Make repository public or grant access

---

## ✅ Knowledge Check

### Question 1
What's the trigger for reusable workflows?

<details>
<summary>Answer</summary>

```yaml
on:
  workflow_call:
```

This trigger allows a workflow to be called by other workflows.
</details>

### Question 2
How do you download all artifacts in a job?

<details>
<summary>Answer</summary>

```yaml
- uses: actions/download-artifact@v4
  with:
    path: artifacts/  # Downloads all to this directory
```

Omitting the `name` parameter downloads all artifacts.
</details>

### Question 3
What's the maximum artifact retention period?

<details>
<summary>Answer</summary>

90 days is the maximum retention period for artifacts. Default is 90 days, but you can set it lower with `retention-days`.
</details>

---

## 🎯 Challenge Exercises

### Challenge 1: Multi-Environment Deploy
Create a reusable workflow that deploys to dev, staging, and prod based on input.

### Challenge 2: Artifact Diff
Create a workflow that compares artifacts from current and previous runs.

### Challenge 3: Smart Cache Invalidation
Implement cache invalidation based on file content changes, not just hash.

---

## 📖 Additional Resources

- [Reusing Workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
- [Artifacts Documentation](https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts)
- [Caching Dependencies](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows)
- [actions/cache](https://github.com/actions/cache)
- [actions/upload-artifact](https://github.com/actions/upload-artifact)

---

## 📝 Summary

In this lab, you learned:
- ✅ Creating reusable workflows with `workflow_call`
- ✅ Passing inputs, secrets, and outputs
- ✅ Uploading and downloading artifacts
- ✅ Sharing artifacts between jobs
- ✅ Advanced caching strategies
- ✅ Matrix builds with artifact collection
- ✅ Combining reusable workflows with artifacts
- ✅ Best practices for workflow reusability

**Next Steps:**
- Complete Lab 10: Enterprise & Self-Hosted Runners
- Create organization-wide workflow templates
- Implement artifact management strategies

---

**Estimated Time to Complete:** 55 minutes  
**Difficulty:** Intermediate to Advanced

Excellent work! You've mastered advanced workflow patterns! 🚀
