# Lab 6: End-to-End CI/CD Pipeline to Azure

## 🎯 Lab Objectives

In this lab, you will:
- Build a production-grade multi-stage pipeline
- Implement comprehensive testing strategy
- Configure Azure deployment slots
- Implement blue-green deployments
- Add security scanning and compliance checks
- Set up monitoring and observability
- Apply all best practices learned

**Estimated Time:** 90 minutes  
**Difficulty:** Advanced  
**Prerequisites:** Completed Labs 1-5

## 📚 Background Information

### Production Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    PRODUCTION CI/CD PIPELINE                 │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  STAGE 1: CODE QUALITY                                      │
│  ├── Checkout code                                          │
│  ├── Lint & format check                                    │
│  ├── Security scan (SAST)                                   │
│  └── Dependency audit                                       │
│                                                              │
│  STAGE 2: BUILD & TEST                                      │
│  ├── Install dependencies                                   │
│  ├── Run unit tests                                         │
│  ├── Generate coverage report                               │
│  ├── Build application                                      │
│  └── Create artifacts                                       │
│                                                              │
│  STAGE 3: INTEGRATION TEST                                  │
│  ├── Deploy to test environment                             │
│  ├── Run integration tests                                  │
│  ├── Run E2E tests                                          │
│  └── Performance testing                                    │
│                                                              │
│  STAGE 4: SECURITY & COMPLIANCE                             │
│  ├── Container image scan                                   │
│  ├── Infrastructure compliance                              │
│  ├── License compliance                                     │
│  └── Secrets detection                                      │
│                                                              │
│  STAGE 5: STAGING DEPLOYMENT                                │
│  ├── Deploy to staging slot                                 │
│  ├── Smoke tests                                            │
│  ├── Manual approval gate                                   │
│  └── Swap to production                                     │
│                                                              │
│  STAGE 6: PRODUCTION DEPLOYMENT                             │
│  ├── Blue-green deployment                                  │
│  ├── Health monitoring                                      │
│  ├── Automated rollback                                     │
│  └── Notification & reporting                               │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## 🔬 Lab Exercises

### Exercise 1: Create Production-Grade Application

**Objective:** Build a comprehensive Node.js application with all testing layers.

#### Steps:

1. **Create Repository Structure**

   ```powershell
   cd C:\Users\alibengtsson\mycertifications\githubactions
   mkdir azure-production-app
   cd azure-production-app
   npm init -y
   ```

2. **Install Comprehensive Dependencies**

   ```powershell
   # Production dependencies
   npm install express dotenv applicationinsights

   # Development dependencies
   npm install --save-dev `
     jest supertest `
     eslint eslint-config-airbnb-base eslint-plugin-import `
     @types/jest @types/express `
     prettier `
     husky lint-staged
   ```

3. **Create Application Structure**

   Create folders:
   ```powershell
   mkdir src, tests, tests\unit, tests\integration, .github, .github\workflows
   ```

4. **Build Production Application**

   **Create `src/app.js`:**
   ```javascript
   const express = require('express');
   const appInsights = require('applicationinsights');

   // Initialize Application Insights if key provided
   if (process.env.APPINSIGHTS_INSTRUMENTATIONKEY) {
     appInsights.setup()
       .setAutoDependencyCorrelation(true)
       .setAutoCollectRequests(true)
       .setAutoCollectPerformance(true)
       .setAutoCollectExceptions(true)
       .setAutoCollectDependencies(true)
       .start();
   }

   const app = express();
   const port = process.env.PORT || 3000;

   app.use(express.json());

   // Health check
   app.get('/health', (req, res) => {
     res.status(200).json({
       status: 'healthy',
       timestamp: new Date().toISOString(),
       uptime: process.uptime(),
       environment: process.env.NODE_ENV || 'development'
     });
   });

   // Root endpoint
   app.get('/', (req, res) => {
     res.json({
       message: 'Azure Production App',
       version: process.env.APP_VERSION || '1.0.0',
       environment: process.env.NODE_ENV || 'development'
     });
   });

   // API endpoints
   app.get('/api/data', (req, res) => {
     res.json({
       data: [1, 2, 3, 4, 5],
       count: 5
     });
   });

   app.post('/api/data', (req, res) => {
     const { value } = req.body;
     res.status(201).json({
       message: 'Data created',
       value
     });
   });

   // Error handling
   app.use((err, req, res, next) => {
     console.error(err.stack);
     res.status(500).json({
       error: 'Something went wrong!',
       message: err.message
     });
   });

   if (require.main === module) {
     app.listen(port, () => {
       console.log(`Server running on port ${port}`);
     });
   }

   module.exports = app;
   ```

5. **Create Comprehensive Tests**

   **Create `tests/unit/app.test.js`:**
   ```javascript
   const request = require('supertest');
   const app = require('../../src/app');

   describe('Unit Tests', () => {
     describe('GET /', () => {
       it('should return application info', async () => {
         const response = await request(app).get('/');
         expect(response.status).toBe(200);
         expect(response.body).toHaveProperty('message');
         expect(response.body).toHaveProperty('version');
       });
     });

     describe('GET /health', () => {
       it('should return healthy status', async () => {
         const response = await request(app).get('/health');
         expect(response.status).toBe(200);
         expect(response.body.status).toBe('healthy');
       });
     });

     describe('GET /api/data', () => {
       it('should return data array', async () => {
         const response = await request(app).get('/api/data');
         expect(response.status).toBe(200);
         expect(response.body.data).toBeInstanceOf(Array);
       });
     });

     describe('POST /api/data', () => {
       it('should create data', async () => {
         const response = await request(app)
           .post('/api/data')
           .send({ value: 'test' });
         expect(response.status).toBe(201);
         expect(response.body.value).toBe('test');
       });
     });
   });
   ```

6. **Configure ESLint and Prettier**

   **Create `.eslintrc.json`:**
   ```json
   {
     "env": {
       "node": true,
       "es2021": true,
       "jest": true
     },
     "extends": "airbnb-base",
     "parserOptions": {
       "ecmaVersion": "latest"
     },
     "rules": {
       "no-console": "off",
       "comma-dangle": ["error", "never"]
     }
   }
   ```

   **Create `.prettierrc`:**
   ```json
   {
     "singleQuote": true,
     "trailingComma": "none",
     "printWidth": 100,
     "tabWidth": 2
   }
   ```

7. **Update package.json Scripts**

   ```json
   {
     "scripts": {
       "start": "node src/app.js",
       "test": "jest",
       "test:unit": "jest tests/unit",
       "test:integration": "jest tests/integration",
       "test:coverage": "jest --coverage",
       "lint": "eslint src tests",
       "lint:fix": "eslint src tests --fix",
       "format": "prettier --write \"src/**/*.js\" \"tests/**/*.js\"",
       "format:check": "prettier --check \"src/**/*.js\" \"tests/**/*.js\""
     }
   }
   ```

### Exercise 2: Build Multi-Stage CI Pipeline

**Objective:** Create a comprehensive CI pipeline with multiple quality gates.

#### Steps:

**Create `.github/workflows/ci-pipeline.yml`:**
```yaml
name: CI Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  NODE_VERSION: '20.x'

jobs:
  code-quality:
    name: Code Quality Checks
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install Dependencies
        run: npm ci

      - name: Lint Check
        run: npm run lint

      - name: Format Check
        run: npm run format:check

      - name: Dependency Audit
        run: npm audit --audit-level=moderate

  security-scan:
    name: Security Scanning
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Run CodeQL Analysis
        uses: github/codeql-action/init@v2
        with:
          languages: javascript

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v2

      - name: Check for Secrets
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: ${{ github.event.repository.default_branch }}
          head: HEAD

  build-and-test:
    name: Build and Test
    needs: [code-quality, security-scan]
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18.x, 20.x]
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'

      - name: Install Dependencies
        run: npm ci

      - name: Run Unit Tests
        run: npm run test:unit

      - name: Run Integration Tests
        run: npm run test:integration

      - name: Generate Coverage
        run: npm run test:coverage

      - name: Upload Coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
          flags: unittests
          name: codecov-${{ matrix.node-version }}

      - name: Build Application
        run: |
          echo "Building application..."
          # Add build steps if needed

      - name: Create Artifact
        if: matrix.node-version == '20.x'
        run: |
          mkdir -p artifact
          cp -r src package*.json artifact/
          tar -czf app-${{ github.sha }}.tar.gz artifact

      - name: Upload Build Artifact
        if: matrix.node-version == '20.x'
        uses: actions/upload-artifact@v4
        with:
          name: application-build
          path: app-${{ github.sha }}.tar.gz
          retention-days: 7
```

### Exercise 3: Implement Azure Deployment with Slots

**Objective:** Deploy to Azure using deployment slots for blue-green deployments.

#### Steps:

1. **Create Azure Resources with Slots**

   ```powershell
   # Variables
   $rgName = "rg-production-app"
   $location = "eastus"
   $planName = "plan-production"
   $appName = "webapp-production-$((Get-Random -Maximum 9999))"

   # Create resource group
   az group create --name $rgName --location $location

   # Create App Service Plan (Standard tier for slots)
   az appservice plan create `
     --name $planName `
     --resource-group $rgName `
     --sku S1 `
     --is-linux

   # Create Web App
   az webapp create `
     --name $appName `
     --resource-group $rgName `
     --plan $planName `
     --runtime "NODE:20-lts"

   # Create staging slot
   az webapp deployment slot create `
     --name $appName `
     --resource-group $rgName `
     --slot staging

   # Configure always on
   az webapp config set `
     --name $appName `
     --resource-group $rgName `
     --always-on true

   az webapp config set `
     --name $appName `
     --resource-group $rgName `
     --slot staging `
     --always-on true
   ```

2. **Create CD Pipeline with Deployment Slots**

**Create `.github/workflows/cd-pipeline.yml`:**
```yaml
name: CD Pipeline

on:
  workflow_run:
    workflows: ["CI Pipeline"]
    types: [completed]
    branches: [main]
  workflow_dispatch:

concurrency:
  group: production-deployment
  cancel-in-progress: false

jobs:
  deploy-staging:
    name: Deploy to Staging Slot
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'success' || github.event_name == 'workflow_dispatch' }}
    environment:
      name: staging
      url: https://${{ secrets.AZURE_WEBAPP_NAME }}-staging.azurewebsites.net
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20.x'

      - name: Install Dependencies
        run: npm ci --production

      - name: Azure Login
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Deploy to Staging Slot
        uses: azure/webapps-deploy@v2
        with:
          app-name: ${{ secrets.AZURE_WEBAPP_NAME }}
          slot-name: staging
          package: .

      - name: Configure App Settings
        uses: azure/appservice-settings@v1
        with:
          app-name: ${{ secrets.AZURE_WEBAPP_NAME }}
          slot-name: staging
          app-settings-json: |
            [
              {
                "name": "NODE_ENV",
                "value": "production"
              },
              {
                "name": "APP_VERSION",
                "value": "${{ github.sha }}"
              }
            ]

  smoke-test-staging:
    name: Smoke Test Staging
    needs: deploy-staging
    runs-on: ubuntu-latest
    steps:
      - name: Wait for Deployment
        run: sleep 30

      - name: Health Check
        run: |
          URL="https://${{ secrets.AZURE_WEBAPP_NAME }}-staging.azurewebsites.net/health"
          echo "Testing: $URL"
          
          for i in {1..10}; do
            STATUS=$(curl -s -o /dev/null -w "%{http_code}" $URL)
            if [ $STATUS -eq 200 ]; then
              echo "✅ Health check passed!"
              exit 0
            fi
            echo "Attempt $i: Status $STATUS, retrying..."
            sleep 10
          done
          
          echo "❌ Health check failed!"
          exit 1

      - name: API Tests
        run: |
          BASE_URL="https://${{ secrets.AZURE_WEBAPP_NAME }}-staging.azurewebsites.net"
          
          # Test root endpoint
          curl -f "$BASE_URL/" || exit 1
          
          # Test API endpoint
          curl -f "$BASE_URL/api/data" || exit 1
          
          echo "✅ All smoke tests passed!"

  swap-to-production:
    name: Swap to Production
    needs: smoke-test-staging
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://${{ secrets.AZURE_WEBAPP_NAME }}.azurewebsites.net
    steps:
      - name: Azure Login
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Swap Slots
        run: |
          echo "🔄 Swapping staging to production..."
          
          az webapp deployment slot swap \
            --name "${{ secrets.AZURE_WEBAPP_NAME }}" \
            --resource-group "${{ secrets.AZURE_RESOURCE_GROUP }}" \
            --slot staging \
            --target-slot production
          
          echo "✅ Swap completed!"

      - name: Azure Logout
        if: always()
        run: az logout

  verify-production:
    name: Verify Production
    needs: swap-to-production
    runs-on: ubuntu-latest
    steps:
      - name: Wait for Swap
        run: sleep 30

      - name: Production Health Check
        run: |
          URL="https://${{ secrets.AZURE_WEBAPP_NAME }}.azurewebsites.net/health"
          
          STATUS=$(curl -s -o /dev/null -w "%{http_code}" $URL)
          if [ $STATUS -eq 200 ]; then
            echo "✅ Production is healthy!"
          else
            echo "❌ Production health check failed!"
            exit 1
          fi

      - name: Verify Version
        run: |
          VERSION=$(curl -s "https://${{ secrets.AZURE_WEBAPP_NAME }}.azurewebsites.net/" | jq -r '.version')
          echo "Deployed version: $VERSION"
          echo "Expected SHA: ${{ github.sha }}"

  rollback:
    name: Automated Rollback
    needs: verify-production
    if: failure()
    runs-on: ubuntu-latest
    steps:
      - name: Azure Login
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Rollback Deployment
        run: |
          echo "❌ Deployment verification failed, rolling back..."
          
          az webapp deployment slot swap \
            --name "${{ secrets.AZURE_WEBAPP_NAME }}" \
            --resource-group "${{ secrets.AZURE_RESOURCE_GROUP }}" \
            --slot staging \
            --target-slot production
          
          echo "✅ Rollback completed!"

      - name: Send Alert
        run: |
          echo "🚨 ALERT: Production deployment failed and was rolled back!"
```

### Exercise 4: Add Monitoring and Observability

**Objective:** Implement Application Insights and monitoring.

#### Steps:

1. **Create Application Insights**

   ```powershell
   # Create Application Insights
   az monitor app-insights component create `
     --app production-app-insights `
     --location eastus `
     --resource-group rg-production-app `
     --application-type web

   # Get instrumentation key
   $instrumentationKey = az monitor app-insights component show `
     --app production-app-insights `
     --resource-group rg-production-app `
     --query instrumentationKey `
     --output tsv

   # Configure Web App
   az webapp config appsettings set `
     --name $appName `
     --resource-group rg-production-app `
     --settings "APPINSIGHTS_INSTRUMENTATIONKEY=$instrumentationKey"
   ```

2. **Create Monitoring Workflow**

**Create `.github/workflows/monitoring.yml`:**
```yaml
name: Production Monitoring

on:
  schedule:
    - cron: '*/15 * * * *'  # Every 15 minutes
  workflow_dispatch:

jobs:
  health-monitoring:
    name: Monitor Production Health
    runs-on: ubuntu-latest
    steps:
      - name: Check Production Health
        id: health
        run: |
          URL="https://${{ secrets.AZURE_WEBAPP_NAME }}.azurewebsites.net/health"
          RESPONSE=$(curl -s -w "\n%{http_code}" $URL)
          STATUS=$(echo "$RESPONSE" | tail -n1)
          BODY=$(echo "$RESPONSE" | sed '$d')
          
          echo "status=$STATUS" >> $GITHUB_OUTPUT
          
          if [ "$STATUS" -ne 200 ]; then
            echo "::error::Health check failed with status $STATUS"
            exit 1
          fi

      - name: Azure Login
        if: failure()
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Collect Diagnostics
        if: failure()
        run: |
          echo "Collecting diagnostics..."
          
          az webapp log tail \
            --name "${{ secrets.AZURE_WEBAPP_NAME }}" \
            --resource-group "${{ secrets.AZURE_RESOURCE_GROUP }}" \
            --lines 100

      - name: Send Alert
        if: failure()
        run: |
          echo "🚨 ALERT: Production health check failed!"
          echo "Status: ${{ steps.health.outputs.status }}"
```

## 📝 Complete Workflow Best Practices Summary

### ✅ Security Best Practices
- Always use GitHub Secrets for credentials
- Never commit sensitive data
- Use service principals with minimal permissions
- Rotate secrets regularly
- Enable Dependabot for security updates
- Run security scans (CodeQL, Trivy, etc.)

### ✅ Performance Optimization
- Use caching (npm, dependencies)
- Parallelize independent jobs
- Use path/branch filtering
- Set appropriate timeouts
- Use ubuntu-latest for cost efficiency

### ✅ Reliability
- Implement health checks
- Use deployment slots for zero-downtime
- Automated rollback on failure
- Retry logic for flaky operations
- Comprehensive error handling

### ✅ Observability
- Structured logging
- Application Insights integration
- Workflow status badges
- Detailed artifact retention
- Monitoring and alerting

## 🎓 Lab Summary

**Congratulations! You've built a production-grade CI/CD pipeline!**

✅ Multi-stage CI pipeline  
✅ Blue-green deployments  
✅ Automated testing  
✅ Security scanning  
✅ Monitoring & observability  
✅ All best practices  

## 🏆 Module 1 Complete!

You've now completed all labs for **"Automate Development Tasks by using GitHub Actions"**!

**Skills Mastered:**
- GitHub Actions fundamentals
- Creating workflows
- Building custom actions (Container & JavaScript)
- Advanced triggering
- Azure integration
- Production-ready pipelines

## 🔜 Next Module

Ready for **Module 2: Build continuous integration workflows**!

---

**Congratulations on completing Module 1! 🎉**
