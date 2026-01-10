# Lab 2: Creating Your First GitHub Actions Workflow

## 🎯 Lab Objectives

In this lab, you will:
- Create a complete CI/CD workflow from scratch
- Build and test a Node.js application
- Implement GitHub Secrets for Azure authentication
- Deploy an application to Azure App Service
- Implement workflow best practices

**Estimated Time:** 45 minutes  
**Difficulty:** Beginner  
**Prerequisites:** Completed Lab 1

## 📚 Background Information

### What is a CI/CD Pipeline?

**Continuous Integration (CI)** is the practice of automatically building and testing code changes as they're committed to version control.

**Continuous Deployment (CD)** automates the release of validated code to production or staging environments.

**Benefits:**
- 🚀 Faster time to market
- 🐛 Earlier bug detection
- 📈 Improved code quality
- 🔄 Consistent deployments
- 👥 Better collaboration

### Workflow Structure for CI/CD

```
┌────────────────────────────────────────────────────────┐
│                    CI/CD PIPELINE                       │
├────────────────────────────────────────────────────────┤
│                                                         │
│  1. Code Commit/PR                                     │
│            ↓                                            │
│  2. Checkout Code                                      │
│            ↓                                            │
│  3. Install Dependencies                               │
│            ↓                                            │
│  4. Run Linters/Code Quality Checks                    │
│            ↓                                            │
│  5. Run Unit Tests                                     │
│            ↓                                            │
│  6. Build Application                                  │
│            ↓                                            │
│  7. Run Integration Tests                              │
│            ↓                                            │
│  8. Build Container Image (if applicable)              │
│            ↓                                            │
│  9. Deploy to Staging/Production                       │
│            ↓                                            │
│ 10. Run Smoke Tests                                    │
│            ↓                                            │
│ 11. Notify Team (Success/Failure)                      │
│                                                         │
└────────────────────────────────────────────────────────┘
```

### Azure App Service Overview

**Azure App Service** is a fully managed platform for building, deploying, and scaling web apps.

**Key Features:**
- Multiple language support (Node.js, .NET, Python, Java, PHP)
- Built-in CI/CD integration
- Auto-scaling capabilities
- Custom domain and SSL support
- Deployment slots for staging

**Why use it with GitHub Actions?**
- Seamless integration with GitHub
- Automated deployments on code changes
- Zero-downtime deployments with deployment slots
- Easy rollback capabilities

## 🔬 Lab Exercises

### Exercise 1: Create a Sample Node.js Application

**Objective:** Set up a simple Node.js web application that we'll use for CI/CD.

#### Steps:

1. **Create a New GitHub Repository**
   - Go to GitHub → New Repository
   - Name: `azure-nodejs-webapp`
   - Description: `Node.js app deployed to Azure with GitHub Actions`
   - Visibility: Public
   - Check "Add a README file"
   - Click "Create repository"

2. **Clone the Repository Locally**
   ```powershell
   # Open PowerShell
   cd C:\Users\alibengtsson\mycertifications\githubactions
   git clone https://github.com/YOUR-USERNAME/azure-nodejs-webapp.git
   cd azure-nodejs-webapp
   ```

3. **Initialize Node.js Project**
   ```powershell
   # Initialize npm project
   npm init -y
   
   # Install Express.js (web framework)
   npm install express
   
   # Install development dependencies
   npm install --save-dev jest supertest
   ```

4. **Create Application Files**

   **Create `app.js`:**
   ```javascript
   const express = require('express');
   const app = express();
   const port = process.env.PORT || 3000;

   // Middleware for JSON parsing
   app.use(express.json());

   // Root endpoint
   app.get('/', (req, res) => {
     res.json({
       message: 'Welcome to Azure Node.js App!',
       version: '1.0.0',
       timestamp: new Date().toISOString()
     });
   });

   // Health check endpoint (important for Azure)
   app.get('/health', (req, res) => {
     res.status(200).json({ status: 'healthy' });
   });

   // API endpoint
   app.get('/api/info', (req, res) => {
     res.json({
       environment: process.env.NODE_ENV || 'development',
       nodeVersion: process.version,
       platform: process.platform
     });
   });

   // 404 handler
   app.use((req, res) => {
     res.status(404).json({ error: 'Route not found' });
   });

   // Error handler
   app.use((err, req, res, next) => {
     console.error(err.stack);
     res.status(500).json({ error: 'Something went wrong!' });
   });

   // Start server
   if (require.main === module) {
     app.listen(port, () => {
       console.log(`Server running on port ${port}`);
     });
   }

   // Export for testing
   module.exports = app;
   ```

5. **Create Test File**

   **Create `app.test.js`:**
   ```javascript
   const request = require('supertest');
   const app = require('./app');

   describe('API Endpoints', () => {
     test('GET / should return welcome message', async () => {
       const response = await request(app).get('/');
       expect(response.status).toBe(200);
       expect(response.body).toHaveProperty('message');
       expect(response.body.message).toContain('Welcome');
     });

     test('GET /health should return healthy status', async () => {
       const response = await request(app).get('/health');
       expect(response.status).toBe(200);
       expect(response.body.status).toBe('healthy');
     });

     test('GET /api/info should return system info', async () => {
       const response = await request(app).get('/api/info');
       expect(response.status).toBe(200);
       expect(response.body).toHaveProperty('nodeVersion');
     });

     test('GET /invalid-route should return 404', async () => {
       const response = await request(app).get('/invalid-route');
       expect(response.status).toBe(404);
     });
   });
   ```

6. **Update package.json Scripts**

   Edit `package.json` and add/update the scripts section:
   ```json
   {
     "name": "azure-nodejs-webapp",
     "version": "1.0.0",
     "description": "Node.js app deployed to Azure with GitHub Actions",
     "main": "app.js",
     "scripts": {
       "start": "node app.js",
       "test": "jest",
       "test:coverage": "jest --coverage"
     },
     "keywords": ["azure", "nodejs", "github-actions"],
     "author": "Your Name",
     "license": "MIT",
     "dependencies": {
       "express": "^4.18.2"
     },
     "devDependencies": {
       "jest": "^29.7.0",
       "supertest": "^6.3.3"
     },
     "engines": {
       "node": ">=18.x"
     }
   }
   ```

7. **Create .gitignore File**

   **Create `.gitignore`:**
   ```
   # Dependencies
   node_modules/
   
   # Environment variables
   .env
   .env.local
   
   # Logs
   logs
   *.log
   npm-debug.log*
   
   # Testing
   coverage/
   .nyc_output/
   
   # OS files
   .DS_Store
   Thumbs.db
   
   # IDE files
   .vscode/
   .idea/
   *.swp
   *.swo
   ```

8. **Test Locally**
   ```powershell
   # Run tests
   npm test
   
   # Start the app
   npm start
   ```
   
   Open browser: `http://localhost:3000`  
   You should see the JSON response!

9. **Commit and Push**
   ```powershell
   git add .
   git commit -m "Initial Node.js application with tests"
   git push origin main
   ```

### Exercise 2: Create Basic CI Workflow

**Objective:** Create a GitHub Actions workflow that builds and tests the application on every push.

#### Steps:

1. **Create Workflow Directory**
   ```powershell
   mkdir .github\workflows
   ```

2. **Create CI Workflow File**

   **Create `.github/workflows/ci.yml`:**
   ```yaml
   name: CI - Build and Test

   # Trigger on push and pull requests to main branch
   on:
     push:
       branches: [ main ]
     pull_request:
       branches: [ main ]

   # Cancel in-progress runs for same workflow
   concurrency:
     group: ${{ github.workflow }}-${{ github.ref }}
     cancel-in-progress: true

   jobs:
     build-and-test:
       name: Build and Test Node.js App
       runs-on: ubuntu-latest
       timeout-minutes: 10

       strategy:
         matrix:
           node-version: [18.x, 20.x]

       steps:
         # Step 1: Checkout code
         - name: Checkout repository
           uses: actions/checkout@v4

         # Step 2: Setup Node.js
         - name: Setup Node.js ${{ matrix.node-version }}
           uses: actions/setup-node@v4
           with:
             node-version: ${{ matrix.node-version }}
             cache: 'npm'  # Cache npm dependencies

         # Step 3: Install dependencies
         - name: Install dependencies
           run: npm ci  # ci is faster and more reliable than install

         # Step 4: Run linter (if you add one later)
         - name: Run linter
           run: npm run lint --if-present

         # Step 5: Run tests
         - name: Run tests
           run: npm test

         # Step 6: Generate test coverage
         - name: Generate test coverage
           run: npm run test:coverage

         # Step 7: Upload coverage report
         - name: Upload coverage to Codecov
           uses: codecov/codecov-action@v3
           if: matrix.node-version == '20.x'  # Only upload once
           with:
             fail_ci_if_error: false

         # Step 8: Build application
         - name: Build application
           run: |
             echo "Building application..."
             # Add build steps if needed (e.g., TypeScript compilation)

         # Step 9: Upload build artifacts
         - name: Upload build artifacts
           uses: actions/upload-artifact@v4
           if: matrix.node-version == '20.x'
           with:
             name: node-app
             path: |
               app.js
               package.json
               package-lock.json
               node_modules/
             retention-days: 1
   ```

3. **Understanding the CI Workflow**

   **Strategy Matrix:**
   ```yaml
   strategy:
     matrix:
       node-version: [18.x, 20.x]
   ```
   - Tests on multiple Node.js versions
   - Ensures compatibility
   - Jobs run in parallel

   **Caching:**
   ```yaml
   cache: 'npm'
   ```
   - Caches node_modules
   - Speeds up subsequent runs
   - Reduces GitHub Actions minutes

   **npm ci vs npm install:**
   - `npm ci` is faster
   - Uses package-lock.json exactly
   - Better for CI/CD environments

4. **Commit and Push CI Workflow**
   ```powershell
   git add .
   git commit -m "Add CI workflow for build and test"
   git push origin main
   ```

5. **Monitor Workflow Execution**
   - Go to GitHub → Your repository → Actions tab
   - You'll see "CI - Build and Test" running
   - Click on the workflow run
   - Expand steps to see detailed logs
   - Both Node.js 18.x and 20.x should succeed

6. **Verify Artifacts**
   - After workflow completes
   - Scroll down to "Artifacts" section
   - Download `node-app` artifact
   - Verify it contains your application files

### Exercise 3: Set Up Azure App Service

**Objective:** Create an Azure App Service to host your Node.js application.

#### Steps:

1. **Login to Azure Portal**
   - Go to [https://portal.azure.com](https://portal.azure.com)
   - Sign in with your Azure account

2. **Create Resource Group**
   
   **Option A: Azure Portal**
   - Click "Resource groups" → "+ Create"
   - Subscription: Select your subscription
   - Resource group name: `rg-github-actions-lab`
   - Region: Choose closest region (e.g., East US)
   - Click "Review + create" → "Create"

   **Option B: Azure CLI (PowerShell)**
   ```powershell
   # Login to Azure
   az login

   # Create resource group
   az group create `
     --name rg-github-actions-lab `
     --location eastus
   ```

3. **Create App Service Plan**

   **What is an App Service Plan?**
   - Defines compute resources for your web app
   - Specifies region, VM size, scaling rules
   - Multiple apps can share one plan

   **Azure CLI:**
   ```powershell
   # Create App Service Plan (Free tier for lab)
   az appservice plan create `
     --name plan-github-actions-lab `
     --resource-group rg-github-actions-lab `
     --sku F1 `
     --is-linux
   ```

   **Pricing Tiers:**
   - **F1 (Free):** Good for testing, 60 min/day limit
   - **B1 (Basic):** $13/month, always on, custom domains
   - **P1V2 (Premium):** $80/month, auto-scaling, slots

4. **Create Web App**

   ```powershell
   # Create Web App with Node.js runtime
   az webapp create `
     --name webapp-nodejs-actions-$((Get-Random -Maximum 9999)) `
     --resource-group rg-github-actions-lab `
     --plan plan-github-actions-lab `
     --runtime "NODE:20-lts"
   ```

   **Note:** Replace the random number or use a unique name. Azure requires globally unique web app names.

5. **Save Your Web App Name**
   ```powershell
   # Get your webapp name
   az webapp list `
     --resource-group rg-github-actions-lab `
     --query "[].{name:name, url:defaultHostName}" `
     --output table
   ```
   
   **Save this information:**
   - Web App Name: `webapp-nodejs-actions-XXXX`
   - URL: `webapp-nodejs-actions-XXXX.azurewebsites.net`

6. **Configure Web App Settings**

   ```powershell
   # Set Node.js version
   az webapp config appsettings set `
     --name YOUR-WEBAPP-NAME `
     --resource-group rg-github-actions-lab `
     --settings NODE_ENV=production

   # Enable detailed logging
   az webapp log config `
     --name YOUR-WEBAPP-NAME `
     --resource-group rg-github-actions-lab `
     --application-logging true `
     --detailed-error-messages true `
     --level information
   ```

7. **Test Web App (Initial)**
   - Open browser to: `https://YOUR-WEBAPP-NAME.azurewebsites.net`
   - You'll see a default Azure page
   - We'll deploy our app next!

### Exercise 4: Configure GitHub Secrets for Azure

**Objective:** Securely store Azure credentials in GitHub Secrets for deployment.

#### Background: GitHub Secrets

**What are GitHub Secrets?**
- Encrypted environment variables
- Stored securely in your repository
- Never exposed in logs or workflow files
- Accessed via `${{ secrets.SECRET_NAME }}`

**Best Practices:**
- ✅ Always use secrets for credentials
- ✅ Use service principals with minimal permissions
- ✅ Rotate secrets regularly
- ✅ Use environment-specific secrets
- ❌ Never hardcode credentials
- ❌ Never commit .env files with real credentials

#### Steps:

1. **Create Azure Service Principal**

   A service principal is an identity for automated tools (like GitHub Actions) to access Azure.

   ```powershell
   # Get your subscription ID
   $subscriptionId = az account show --query id --output tsv

   # Create service principal with Contributor role
   az ad sp create-for-rbac `
     --name "github-actions-sp" `
     --role Contributor `
     --scopes /subscriptions/$subscriptionId/resourceGroups/rg-github-actions-lab `
     --sdk-auth
   ```

   **Output (SAVE THIS!):**
   ```json
   {
     "clientId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
     "clientSecret": "xxxxxxxxxxxxxxxxxxxxxxxxxxxx",
     "subscriptionId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
     "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
     "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
     "resourceManagerEndpointUrl": "https://management.azure.com/",
     "activeDirectoryGraphResourceId": "https://graph.windows.net/",
     "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
     "galleryEndpointUrl": "https://gallery.azure.com/",
     "managementEndpointUrl": "https://management.core.windows.net/"
   }
   ```

2. **Get Azure Publish Profile**

   ```powershell
   # Download publish profile
   az webapp deployment list-publishing-profiles `
     --name YOUR-WEBAPP-NAME `
     --resource-group rg-github-actions-lab `
     --xml
   ```

   **Copy the entire XML output.**

3. **Add Secrets to GitHub**

   - Go to your GitHub repository
   - Click **Settings** → **Secrets and variables** → **Actions**
   - Click **"New repository secret"**

   **Secret 1: AZURE_CREDENTIALS**
   - Name: `AZURE_CREDENTIALS`
   - Value: Paste the entire JSON from service principal creation
   - Click "Add secret"

   **Secret 2: AZURE_WEBAPP_NAME**
   - Name: `AZURE_WEBAPP_NAME`
   - Value: Your web app name (e.g., `webapp-nodejs-actions-1234`)
   - Click "Add secret"

   **Secret 3: AZURE_WEBAPP_PUBLISH_PROFILE**
   - Name: `AZURE_WEBAPP_PUBLISH_PROFILE`
   - Value: Paste the entire XML publish profile
   - Click "Add secret"

   **Secret 4: AZURE_RESOURCE_GROUP**
   - Name: `AZURE_RESOURCE_GROUP`
   - Value: `rg-github-actions-lab`
   - Click "Add secret"

4. **Verify Secrets**
   - You should see all 4 secrets listed
   - You cannot view secret values after creation (security feature)
   - You can update or delete secrets

### Exercise 5: Create CD Workflow for Azure Deployment

**Objective:** Create a deployment workflow that deploys to Azure App Service.

#### Steps:

1. **Create Deployment Workflow**

   **Create `.github/workflows/deploy.yml`:**
   ```yaml
   name: CD - Deploy to Azure

   on:
     push:
       branches: [ main ]
     workflow_dispatch:  # Allow manual trigger

   # Prevent concurrent deployments
   concurrency:
     group: production-deployment
     cancel-in-progress: false

   jobs:
     deploy:
       name: Deploy to Azure App Service
       runs-on: ubuntu-latest
       environment:
         name: production
         url: https://${{ secrets.AZURE_WEBAPP_NAME }}.azurewebsites.net
       
       steps:
         # Step 1: Checkout code
         - name: Checkout repository
           uses: actions/checkout@v4

         # Step 2: Setup Node.js
         - name: Setup Node.js
           uses: actions/setup-node@v4
           with:
             node-version: '20.x'
             cache: 'npm'

         # Step 3: Install production dependencies
         - name: Install dependencies
           run: npm ci --production

         # Step 4: Create deployment package
         - name: Create deployment package
           run: |
             zip -r deploy.zip . \
               -x "*.git*" \
               -x "*node_modules/.cache*" \
               -x "*.github*" \
               -x "*tests*" \
               -x "*.test.js"

         # Step 5: Login to Azure
         - name: Login to Azure
           uses: azure/login@v1
           with:
             creds: ${{ secrets.AZURE_CREDENTIALS }}

         # Step 6: Deploy to Azure Web App
         - name: Deploy to Azure Web App
           uses: azure/webapps-deploy@v2
           with:
             app-name: ${{ secrets.AZURE_WEBAPP_NAME }}
             package: deploy.zip

         # Step 7: Azure logout (cleanup)
         - name: Azure logout
           run: az logout

         # Step 8: Wait for deployment to stabilize
         - name: Wait for deployment
           run: sleep 30

         # Step 9: Health check
         - name: Run health check
           run: |
             url="https://${{ secrets.AZURE_WEBAPP_NAME }}.azurewebsites.net/health"
             echo "Checking health at: $url"
             
             for i in {1..5}; do
               response=$(curl -s -o /dev/null -w "%{http_code}" $url)
               if [ $response -eq 200 ]; then
                 echo "✅ Health check passed!"
                 exit 0
               fi
               echo "Attempt $i: Status $response, retrying..."
               sleep 10
             done
             
             echo "❌ Health check failed after 5 attempts"
             exit 1

         # Step 10: Notify deployment success
         - name: Deployment summary
           run: |
             echo "🚀 Deployment successful!"
             echo "Application URL: https://${{ secrets.AZURE_WEBAPP_NAME }}.azurewebsites.net"
             echo "Deployed commit: ${{ github.sha }}"
             echo "Deployed by: ${{ github.actor }}"
   ```

2. **Understanding the CD Workflow**

   **Environment Protection:**
   ```yaml
   environment:
     name: production
     url: https://...
   ```
   - Links deployment to GitHub Environment
   - Enables deployment history
   - Supports manual approvals (in paid plans)

   **Concurrency Control:**
   ```yaml
   concurrency:
     group: production-deployment
     cancel-in-progress: false
   ```
   - Prevents concurrent deployments
   - Queues deployments instead of canceling

   **Health Check:**
   - Verifies deployment success
   - Retries 5 times
   - Fails workflow if unhealthy

3. **Create GitHub Environment**

   - Go to repository **Settings** → **Environments**
   - Click **"New environment"**
   - Name: `production`
   - Click **"Configure environment"**
   - (Optional) Add required reviewers
   - Click **"Save protection rules"**

4. **Commit and Deploy**
   ```powershell
   git add .
   git commit -m "Add CD workflow for Azure deployment"
   git push origin main
   ```

5. **Monitor Deployment**
   - Go to Actions tab
   - You'll see both CI and CD workflows running
   - Click on "CD - Deploy to Azure"
   - Watch each step execute
   - Note the deployment URL in the summary

6. **Verify Deployment**
   - Open: `https://YOUR-WEBAPP-NAME.azurewebsites.net`
   - You should see your Node.js app's JSON response!
   - Test endpoints:
     - `/` - Welcome message
     - `/health` - Health status
     - `/api/info` - System information

### Exercise 6: Implement Best Practices

**Objective:** Enhance workflows with production-ready best practices.

#### Steps:

1. **Add Workflow Status Badges**

   Edit your `README.md`:
   ```markdown
   # Azure Node.js Web App

   ![CI](https://github.com/YOUR-USERNAME/azure-nodejs-webapp/workflows/CI%20-%20Build%20and%20Test/badge.svg)
   ![CD](https://github.com/YOUR-USERNAME/azure-nodejs-webapp/workflows/CD%20-%20Deploy%20to%20Azure/badge.svg)

   Node.js application deployed to Azure App Service with GitHub Actions.

   ## 🚀 Live Demo
   [https://YOUR-WEBAPP-NAME.azurewebsites.net](https://YOUR-WEBAPP-NAME.azurewebsites.net)

   ## 🏗️ Architecture
   - **Frontend:** Express.js
   - **Hosting:** Azure App Service
   - **CI/CD:** GitHub Actions
   ```

2. **Add Code Quality Checks**

   Install ESLint:
   ```powershell
   npm install --save-dev eslint
   npx eslint --init
   ```

   Follow the prompts:
   - How would you like to use ESLint? **To check syntax and find problems**
   - What type of modules? **CommonJS**
   - Which framework? **None**
   - Does your project use TypeScript? **No**
   - Where does your code run? **Node**
   - What format for config file? **JSON**

   Create `.eslintrc.json`:
   ```json
   {
     "env": {
       "node": true,
       "es2021": true,
       "jest": true
     },
     "extends": "eslint:recommended",
     "parserOptions": {
       "ecmaVersion": "latest"
     },
     "rules": {
       "no-console": "off",
       "no-unused-vars": ["warn", { "argsIgnorePattern": "^_" }]
     }
   }
   ```

   Update `package.json`:
   ```json
   "scripts": {
     "start": "node app.js",
     "test": "jest",
     "test:coverage": "jest --coverage",
     "lint": "eslint .",
     "lint:fix": "eslint . --fix"
   }
   ```

3. **Add Dependabot for Security**

   Create `.github/dependabot.yml`:
   ```yaml
   version: 2
   updates:
     # Enable version updates for npm
     - package-ecosystem: "npm"
       directory: "/"
       schedule:
         interval: "weekly"
       open-pull-requests-limit: 10
       
     # Enable version updates for GitHub Actions
     - package-ecosystem: "github-actions"
       directory: "/"
       schedule:
         interval: "weekly"
   ```

   **What Dependabot does:**
   - Automatically checks for dependency updates
   - Creates PRs for security vulnerabilities
   - Keeps GitHub Actions up to date

4. **Add Pull Request Workflow**

   Create `.github/workflows/pr-checks.yml`:
   ```yaml
   name: PR Checks

   on:
     pull_request:
       types: [opened, synchronize, reopened]

   jobs:
     validate:
       name: Validate Pull Request
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         
         - name: Setup Node.js
           uses: actions/setup-node@v4
           with:
             node-version: '20.x'
             cache: 'npm'
         
         - name: Install dependencies
           run: npm ci
         
         - name: Run linter
           run: npm run lint
         
         - name: Run tests
           run: npm test
         
         - name: Check code formatting
           run: |
             echo "✅ Code formatting check passed"
         
         - name: Security audit
           run: npm audit --audit-level=moderate
   ```

5. **Add Environment Variables**

   Update `deploy.yml` to set app settings:
   ```yaml
   # Add this step after Azure login
   - name: Configure App Settings
     uses: azure/appservice-settings@v1
     with:
       app-name: ${{ secrets.AZURE_WEBAPP_NAME }}
       app-settings-json: |
         [
           {
             "name": "NODE_ENV",
             "value": "production",
             "slotSetting": false
           },
           {
             "name": "WEBSITE_NODE_DEFAULT_VERSION",
             "value": "~20",
             "slotSetting": false
           },
           {
             "name": "SCM_DO_BUILD_DURING_DEPLOYMENT",
             "value": "false",
             "slotSetting": false
           }
         ]
   ```

6. **Commit All Improvements**
   ```powershell
   git add .
   git commit -m "Add best practices: linting, dependabot, PR checks"
   git push origin main
   ```

## 📊 Testing Your Workflow

### Test 1: Make a Code Change

1. Edit `app.js` - change the version to `1.1.0`
2. Commit and push
3. Watch CI and CD workflows run
4. Verify deployment on Azure

### Test 2: Create a Pull Request

1. Create a new branch:
   ```powershell
   git checkout -b feature/add-timestamp
   ```

2. Add a new endpoint to `app.js`:
   ```javascript
   app.get('/api/timestamp', (req, res) => {
     res.json({ timestamp: Date.now() });
   });
   ```

3. Commit and push:
   ```powershell
   git add app.js
   git commit -m "Add timestamp endpoint"
   git push origin feature/add-timestamp
   ```

4. Create pull request on GitHub
5. Watch PR checks run
6. Merge after checks pass
7. Watch deployment workflow run

### Test 3: Trigger Manual Deployment

1. Go to Actions tab
2. Select "CD - Deploy to Azure"
3. Click "Run workflow"
4. Select branch: `main`
5. Click "Run workflow"
6. Monitor execution

## 📝 Knowledge Check

1. **What's the difference between CI and CD?**
   <details>
   <summary>Answer</summary>
   CI (Continuous Integration) automatically builds and tests code. CD (Continuous Deployment) automatically releases validated code to production/staging.
   </details>

2. **Why use `npm ci` instead of `npm install` in CI/CD?**
   <details>
   <summary>Answer</summary>
   npm ci is faster, uses package-lock.json exactly, and provides cleaner installs for CI/CD environments.
   </details>

3. **What is a service principal in Azure?**
   <details>
   <summary>Answer</summary>
   A service principal is an identity for automated tools to access Azure resources with specific permissions.
   </details>

4. **Why should you never hardcode credentials in workflows?**
   <details>
   <summary>Answer</summary>
   Hardcoded credentials are exposed in version control, logs, and to anyone with repository access. Use GitHub Secrets instead.
   </details>

5. **What does the concurrency setting do?**
   <details>
   <summary>Answer</summary>
   It controls whether multiple workflow runs can execute simultaneously, preventing conflicts in deployments.
   </details>

## 🎓 Lab Summary

**Congratulations!** You've completed Lab 2 and built a complete CI/CD pipeline!

**You learned how to:**
✅ Create a Node.js application with tests  
✅ Build CI workflows for automated testing  
✅ Set up Azure App Service  
✅ Configure GitHub Secrets securely  
✅ Deploy to Azure with GitHub Actions  
✅ Implement production best practices  
✅ Add health checks and monitoring  

## 🔜 Next Steps

Proceed to **[Lab 3: Building a Container Action for Azure](../Lab3-Container-Actions/README.md)** where you'll:
- Create custom Docker container actions
- Build reusable actions for Azure operations
- Publish actions to GitHub Marketplace
- Integrate container actions in workflows

## 📚 Additional Resources

- [Azure App Service Documentation](https://docs.microsoft.com/en-us/azure/app-service/)
- [GitHub Actions - azure/webapps-deploy](https://github.com/Azure/webapps-deploy)
- [GitHub Secrets Documentation](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)

## 🧹 Cleanup (Optional)

To avoid Azure charges:
```powershell
# Delete all resources
az group delete --name rg-github-actions-lab --yes --no-wait
```

---

**Need Help?** Check Azure logs: Azure Portal → Your App Service → Log stream
