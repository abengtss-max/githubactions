# Lab 1: Understanding GitHub Actions Fundamentals

## 🎯 Lab Objectives

In this lab, you will:
- Understand what GitHub Actions are and why they're used
- Explore the GitHub Actions ecosystem and marketplace
- Identify the key components of GitHub Actions
- Learn about runners (GitHub-hosted vs self-hosted)
- Understand workflow triggers and event types
- Explore GitHub Actions integration with Azure

**Estimated Time:** 30 minutes  
**Difficulty:** Beginner

## 📚 Background Information

### What is GitHub Actions?

**GitHub Actions** is an automation platform that allows you to build, test, and deploy your code directly from GitHub. It enables you to create custom workflows that respond to events in your repository.

**Key Benefits:**
- ✅ **Automation:** Automate repetitive tasks in your software development lifecycle
- ✅ **Integration:** Native integration with GitHub and extensive third-party integrations
- ✅ **Flexibility:** Support for multiple languages and platforms
- ✅ **Community:** Access to thousands of pre-built actions in the GitHub Marketplace
- ✅ **Cost-Effective:** Free minutes included with GitHub plans

### Why Use GitHub Actions?

Traditional development workflows require manual intervention for:
- Running unit tests after code changes
- Performing code quality checks
- Building and compiling code
- Deploying to staging/production environments
- Managing infrastructure

**GitHub Actions automates these tasks**, reducing time from idea to deployment and minimizing human error.

### Types of GitHub Actions

There are three types of actions you can create:

1. **Container Actions** 🐳
   - Package the environment with the action code
   - Run only in Linux environments
   - Support multiple programming languages
   - Consistent execution environment
   - Best for: Complex dependencies, specific OS requirements

2. **JavaScript Actions** 📜
   - Run directly on the runner machine
   - Faster execution (no container overhead)
   - Cross-platform (Linux, macOS, Windows)
   - Best for: Simple automation, API integrations

3. **Composite Actions** 🔄
   - Combine multiple workflow steps into one action
   - Reusable across workflows
   - Best for: Bundling common sequences of steps

### GitHub Actions Core Components

```
┌─────────────────────────────────────────────────────────────┐
│                         WORKFLOW                            │
│  (Automated process defined in YAML file)                   │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │                    JOB 1                              │  │
│  │  (Runs on a specific runner)                          │  │
│  │                                                        │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌────────────┐ │  │
│  │  │   STEP 1     │→ │   STEP 2     │→ │  STEP 3    │ │  │
│  │  │ (Checkout)   │  │ (Build)      │  │  (Test)    │ │  │
│  │  └──────────────┘  └──────────────┘  └────────────┘ │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │                    JOB 2                              │  │
│  │  (Can run in parallel or sequentially)                │  │
│  │                                                        │  │
│  │  ┌──────────────┐  ┌──────────────┐                  │  │
│  │  │   STEP 1     │→ │   STEP 2     │                  │  │
│  │  │ (Deploy)     │  │ (Notify)     │                  │  │
│  │  └──────────────┘  └──────────────┘                  │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

**Component Definitions:**

- **Workflow:** Automated process defined in a YAML file in `.github/workflows/`
- **Event:** Specific activity that triggers a workflow (push, pull request, schedule, etc.)
- **Job:** Set of steps that execute on the same runner
- **Step:** Individual task that runs commands or actions
- **Action:** Reusable unit of code that performs a specific task
- **Runner:** Server that executes your workflows

### GitHub-Hosted vs Self-Hosted Runners

| Feature | GitHub-Hosted | Self-Hosted |
|---------|---------------|-------------|
| **Setup** | No setup required | Manual installation required |
| **Maintenance** | Managed by GitHub | You manage updates |
| **Environment** | Fresh VM per job | Persistent environment |
| **Customization** | Limited | Full control |
| **Cost** | Free minutes included | Free (your infrastructure) |
| **Use Case** | Standard builds/tests | Custom hardware, private networks |
| **OS Support** | Linux, Windows, macOS | Any OS you configure |

**Best Practice:** Start with GitHub-hosted runners for simplicity. Move to self-hosted runners only when you need specific hardware, private network access, or want to reduce costs for high-volume workflows.

## 🔬 Lab Exercises

### Exercise 1: Explore the GitHub Actions Interface

**Objective:** Familiarize yourself with the GitHub Actions UI and marketplace.

#### Steps:

1. **Navigate to a Repository**
   - Go to [GitHub](https://github.com) and sign in
   - Create a new repository:
     - Click the "+" icon in the top right → "New repository"
     - Name: `github-actions-lab1`
     - Description: `Learning GitHub Actions fundamentals`
     - Visibility: Public (to avoid consuming private repo minutes)
     - Check "Add a README file"
     - Click "Create repository"

2. **Explore the Actions Tab**
   - In your new repository, click the **"Actions"** tab
   - You'll see suggested workflows based on your repository content
   - **Observation:** GitHub automatically suggests workflows based on detected languages and frameworks

3. **Browse Workflow Templates**
   - Notice the categories:
     - **Deployment** - Deploy to Azure, AWS, etc.
     - **Continuous Integration** - Build and test code
     - **Automation** - Issue management, notifications
     - **Security** - Code scanning, dependency checks
   
   - **For Azure developers**, note these relevant templates:
     - "Deploy to Azure Web App"
     - "Deploy to Azure Kubernetes Service"
     - "Azure Static Web Apps CI/CD"
     - "Azure Container Instances"

4. **Explore GitHub Marketplace**
   - Open a new tab and go to [GitHub Marketplace](https://github.com/marketplace?type=actions)
   - Search for "Azure" - you'll find hundreds of Azure-related actions
   - Click on **"Azure Login"** action
   - Review:
     - Description
     - Usage statistics
     - Documentation
     - Example code snippets
   
   **Popular Azure Actions:**
   - `azure/login` - Authenticate with Azure
   - `azure/webapps-deploy` - Deploy to Azure Web Apps
   - `azure/aks-set-context` - Connect to AKS clusters
   - `azure/arm-deploy` - Deploy ARM templates

5. **Verify Action Publisher**
   - Look for the **blue checkmark** ✓ next to action names
   - This indicates **GitHub-verified publishers**
   - **Security Best Practice:** Always prefer verified actions and review their source code before use

### Exercise 2: Understand Workflow Anatomy

**Objective:** Learn how to read and understand a GitHub Actions workflow file.

#### Steps:

1. **Create a Sample Workflow**
   - In your `github-actions-lab1` repository
   - Click **"Actions"** tab → **"New workflow"**
   - Click **"set up a workflow yourself"**

2. **Examine the Default Template**
   
   GitHub provides a starter template. Let's understand each component:

   ```yaml
   name: CI

   # Triggers - Define when this workflow runs
   on:
     push:
       branches: [ "main" ]
     pull_request:
       branches: [ "main" ]

   # Jobs - One or more jobs to execute
   jobs:
     build:
       # Runner - Where the job executes
       runs-on: ubuntu-latest

       # Steps - Sequential tasks in the job
       steps:
       - uses: actions/checkout@v4
       - name: Run a one-line script
         run: echo Hello, world!
       - name: Run a multi-line script
         run: |
           echo Add other actions to build,
           echo test, and deploy your project.
   ```

3. **Component Breakdown**

   **a) Workflow Name:**
   ```yaml
   name: CI
   ```
   - **Purpose:** Display name in GitHub Actions UI
   - **Best Practice:** Use descriptive names like "CI Pipeline", "Deploy to Azure", etc.

   **b) Trigger Events:**
   ```yaml
   on:
     push:
       branches: [ "main" ]
     pull_request:
       branches: [ "main" ]
   ```
   - **Purpose:** Defines what events trigger this workflow
   - **This example:** Runs on pushes and PRs to the main branch
   - **Azure Use Case:** Trigger deployment workflows only for production branches

   **c) Jobs:**
   ```yaml
   jobs:
     build:
       runs-on: ubuntu-latest
   ```
   - **Purpose:** Groups steps that run together
   - **jobs.build:** Job identifier (can be any name)
   - **runs-on:** Specifies the runner environment

   **d) Steps:**
   ```yaml
   steps:
   - uses: actions/checkout@v4
   ```
   - **uses:** Runs a pre-built action (here: checking out code)
   - **@v4:** Version specification (security best practice)

   ```yaml
   - name: Run a one-line script
     run: echo Hello, world!
   ```
   - **name:** Descriptive name for the step
   - **run:** Executes shell commands

4. **Rename and Commit**
   - Change the filename to `lab1-basics.yml`
   - Commit directly to the main branch
   - **Observation:** The workflow will run automatically!

5. **View Workflow Execution**
   - Go to the **Actions** tab
   - Click on the workflow run (you'll see "CI" with a yellow dot or green check)
   - Click on the **"build"** job
   - Expand each step to see detailed logs
   
   **What's happening:**
   - GitHub provisions a fresh Ubuntu VM
   - Installs necessary software
   - Checks out your code
   - Executes your commands
   - Cleans up the environment

### Exercise 3: Explore Runner Environments

**Objective:** Understand different runner environments and their specifications.

#### Steps:

1. **Review GitHub-Hosted Runner Specifications**
   
   - Open [GitHub Documentation on Runners](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners)
   
   **Available Runners:**
   
   | Label | OS | Processor | RAM | Storage |
   |-------|----|-----------|----|---------|
   | `ubuntu-latest` | Ubuntu 22.04 | 4-core | 16 GB | 14 GB SSD |
   | `windows-latest` | Windows Server 2022 | 4-core | 16 GB | 14 GB SSD |
   | `macos-latest` | macOS 13 | 3-core | 14 GB | 14 GB SSD |

2. **Create a Runner Comparison Workflow**
   
   - In your repository, create `.github/workflows/runner-info.yml`:

   ```yaml
   name: Runner Information
   
   on:
     workflow_dispatch:  # Manual trigger
   
   jobs:
     ubuntu-runner:
       runs-on: ubuntu-latest
       steps:
         - name: Display Ubuntu Runner Info
           run: |
             echo "Runner OS: $(uname -s)"
             echo "Runner Architecture: $(uname -m)"
             echo "CPU Cores: $(nproc)"
             echo "Total Memory: $(free -h | awk '/^Mem:/ {print $2}')"
             echo "Disk Space: $(df -h / | awk 'NR==2 {print $2}')"
     
     windows-runner:
       runs-on: windows-latest
       steps:
         - name: Display Windows Runner Info
           run: |
             echo "Runner OS: $env:OS"
             systeminfo | findstr /C:"Processor"
             systeminfo | findstr /C:"Total Physical Memory"
   ```

3. **Run the Workflow Manually**
   - Go to **Actions** tab
   - Select **"Runner Information"** workflow
   - Click **"Run workflow"** button → **"Run workflow"**
   - Wait for completion and review the output

   **What You're Learning:**
   - Different runner environments have different capabilities
   - Manual workflow triggers using `workflow_dispatch`
   - Accessing system information in workflows

### Exercise 4: Understanding Workflow Triggers

**Objective:** Learn the various ways to trigger workflows and when to use each.

#### Steps:

1. **Study Common Trigger Events**

   | Event | Use Case | Example |
   |-------|----------|---------|
   | `push` | Run on code pushes | CI builds, tests |
   | `pull_request` | Run on PR creation/update | Code review checks |
   | `schedule` | Run at specific times | Nightly builds, reports |
   | `workflow_dispatch` | Manual trigger | On-demand deployments |
   | `release` | Run when release is published | Production deployments |
   | `issues` | Run on issue events | Auto-labeling, triage |
   | `repository_dispatch` | External API trigger | Cross-repo automation |

2. **Create a Multi-Trigger Workflow**
   
   Create `.github/workflows/multi-trigger.yml`:

   ```yaml
   name: Multi-Trigger Demo
   
   on:
     # Trigger on push to main or develop branches
     push:
       branches:
         - main
         - develop
       paths-ignore:
         - '**.md'  # Ignore markdown files
     
     # Trigger on pull requests targeting main
     pull_request:
       branches:
         - main
       types:
         - opened
         - synchronize
     
     # Trigger manually
     workflow_dispatch:
       inputs:
         environment:
           description: 'Environment to deploy'
           required: true
           default: 'staging'
           type: choice
           options:
             - staging
             - production
     
     # Trigger on schedule (every day at midnight UTC)
     schedule:
       - cron: '0 0 * * *'
   
   jobs:
     identify-trigger:
       runs-on: ubuntu-latest
       steps:
         - name: Checkout code
           uses: actions/checkout@v4
         
         - name: Identify Trigger Event
           run: |
             echo "Workflow triggered by: ${{ github.event_name }}"
             echo "Repository: ${{ github.repository }}"
             echo "Branch: ${{ github.ref }}"
             echo "Commit SHA: ${{ github.sha }}"
             
         - name: Show Manual Input (if applicable)
           if: github.event_name == 'workflow_dispatch'
           run: |
             echo "Manual trigger detected!"
             echo "Selected environment: ${{ github.event.inputs.environment }}"
   ```

3. **Test Different Triggers**
   - **Commit this file** to trigger the `push` event
   - **Run manually** via Actions tab → Run workflow
   - **Create a pull request** to see PR trigger
   - **Wait for midnight UTC** to see scheduled run (or modify cron for testing)

4. **Analyze the Output**
   - Review logs for each trigger type
   - Note how `github.event_name` differs
   - Understand context variables available in workflows

**Best Practice - Trigger Selection for Azure Deployments:**
- **Development:** `push` to feature branches → Deploy to dev Azure environment
- **Staging:** `pull_request` to main → Deploy to staging Azure environment
- **Production:** `release` published → Deploy to production Azure environment
- **Manual:** `workflow_dispatch` for hotfixes or troubleshooting

### Exercise 5: GitHub Actions Usage Limits and Cost Management

**Objective:** Understand GitHub Actions usage limits and how to optimize costs.

#### Steps:

1. **Review Your Usage**
   - Go to your GitHub profile → **Settings**
   - Select **Billing and plans** → **Plans and usage**
   - Click **Actions & Packages**
   - Review:
     - Minutes used
     - Storage used
     - Included minutes for your plan

2. **Understand Free Tier Limits**

   | Plan | Included Minutes/Month | Minute Multipliers |
   |------|------------------------|-------------------|
   | Free | 2,000 | Linux: 1x, Windows: 2x, macOS: 10x |
   | Pro | 3,000 | Same multipliers |
   | Team | 3,000 | Same multipliers |
   | Enterprise | 50,000 | Same multipliers |

   **Example Calculation:**
   - 100 minutes on Linux = 100 minutes used
   - 100 minutes on Windows = 200 minutes used
   - 100 minutes on macOS = 1,000 minutes used

3. **Cost Optimization Best Practices**

   **✅ DO:**
   - Use `ubuntu-latest` (cheapest) when possible
   - Set appropriate `timeout-minutes` to prevent runaway workflows
   - Use `paths` and `paths-ignore` to skip unnecessary runs
   - Cache dependencies to speed up workflows
   - Use concurrency controls to cancel outdated runs
   
   **❌ DON'T:**
   - Run workflows for documentation-only changes
   - Use macOS runners unless specifically needed
   - Run expensive workflows on every commit
   - Forget to set timeouts

4. **Create an Optimized Workflow**

   Create `.github/workflows/cost-optimized.yml`:

   ```yaml
   name: Cost-Optimized Workflow
   
   on:
     push:
       branches: [ main, develop ]
       paths:
         - 'src/**'
         - 'tests/**'
         - '.github/workflows/**'
       paths-ignore:
         - '**.md'
         - 'docs/**'
   
   # Cancel in-progress runs when new commits are pushed
   concurrency:
     group: ${{ github.workflow }}-${{ github.ref }}
     cancel-in-progress: true
   
   jobs:
     build:
       runs-on: ubuntu-latest  # Cheapest option
       timeout-minutes: 15     # Prevent runaway workflows
       
       steps:
         - uses: actions/checkout@v4
         
         - name: Example build step
           run: echo "Building..."
   ```

### Exercise 6: Hands-On Azure Web App Deployment

**Objective:** Deploy a real Node.js web application to Azure App Service using GitHub Actions.

**Duration:** 45-60 minutes

**What You'll Learn:**
- Create an Azure Web App using Azure Portal or CLI
- Configure GitHub repository with Azure credentials
- Build and deploy a Node.js application automatically
- Test the deployed application in Azure

---

#### Part 1: Create a Simple Node.js Application

1. **Create a new repository in your GitHub account**
   
   Navigate to GitHub and create a new repository:
   - Repository name: `github-actions-azure-demo`
   - Description: "Demo app for GitHub Actions + Azure deployment"
   - Set to **Public** or **Private**
   - ✅ Check "Add a README file"
   - Click **Create repository**

2. **Clone the repository to your local machine**

   ```bash
   git clone https://github.com/YOUR-USERNAME/github-actions-azure-demo.git
   cd github-actions-azure-demo
   ```

3. **Create a simple Node.js web application**

   Create `package.json`:
   ```json
   {
     "name": "azure-demo-app",
     "version": "1.0.0",
     "description": "Demo app for GitHub Actions and Azure",
     "main": "server.js",
     "scripts": {
       "start": "node server.js",
       "test": "echo \"No tests yet\" && exit 0"
     },
     "engines": {
       "node": ">=18.0.0"
     },
     "dependencies": {
       "express": "^4.18.2"
     }
   }
   ```

4. **Create `server.js`**

   ```javascript
   const express = require('express');
   const app = express();
   const PORT = process.env.PORT || 3000;

   app.get('/', (req, res) => {
     res.send(`
       <html>
         <head>
           <title>GitHub Actions + Azure Demo</title>
           <style>
             body {
               font-family: Arial, sans-serif;
               background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
               color: white;
               display: flex;
               justify-content: center;
               align-items: center;
               height: 100vh;
               margin: 0;
             }
             .container {
               text-align: center;
               background: rgba(255,255,255,0.1);
               padding: 50px;
               border-radius: 20px;
               box-shadow: 0 8px 32px rgba(0,0,0,0.3);
             }
             h1 { font-size: 3em; margin: 0; }
             p { font-size: 1.2em; }
             .badge { 
               background: #28a745; 
               padding: 10px 20px; 
               border-radius: 5px;
               display: inline-block;
               margin-top: 20px;
             }
           </style>
         </head>
         <body>
           <div class="container">
             <h1>🚀 Deployment Successful!</h1>
             <p>This app was deployed using <strong>GitHub Actions</strong></p>
             <p>Running on <strong>Azure App Service</strong></p>
             <div class="badge">✅ CI/CD Pipeline Active</div>
             <p style="margin-top: 30px; font-size: 0.9em;">
               Deployed at: ${new Date().toLocaleString()}
             </p>
           </div>
         </body>
       </html>
     `);
   });

   app.get('/health', (req, res) => {
     res.json({ status: 'healthy', timestamp: new Date().toISOString() });
   });

   app.listen(PORT, () => {
     console.log(`Server running on port ${PORT}`);
   });
   ```

5. **Create `.gitignore`**

   ```
   node_modules/
   npm-debug.log
   .env
   .DS_Store
   ```

6. **Test the application locally** (optional)

   ```bash
   npm install
   npm start
   ```
   
   Visit `http://localhost:3000` to verify it works.

7. **Commit and push to GitHub**

   ```bash
   git add .
   git commit -m "Add Node.js application"
   git push origin main
   ```

---

#### Part 2: Create Azure Web App

**Option A: Using Azure Portal (Recommended for Beginners)**

1. **Sign in to Azure Portal**
   - Go to [https://portal.azure.com](https://portal.azure.com)
   - Sign in with your Azure account

2. **Create a new Web App**
   - Click **"Create a resource"**
   - Search for **"Web App"**
   - Click **Create**

3. **Configure the Web App**
   
   **Basics tab:**
   - **Subscription:** Select your subscription
   - **Resource Group:** Create new → Name it `rg-github-actions-demo`
   - **Name:** `webapp-github-demo-[YOUR-NAME]` (must be globally unique)
   - **Publish:** Code
   - **Runtime stack:** Node 18 LTS
   - **Operating System:** Linux
   - **Region:** Choose closest to you (e.g., East US)
   - **Pricing Plan:** Click "Explore pricing plans"
     - Select **F1 (Free)** for testing
     - Click **Select**

4. **Review + Create**
   - Click **"Review + create"**
   - Verify settings
   - Click **Create**
   - Wait 1-2 minutes for deployment

5. **Note your Web App details** (you'll need these):
   - Once deployed, click **"Go to resource"**
   - **Copy the Web App name** (e.g., `webapp-github-demo-abengtss-max`)
   - **Copy the Resource Group name** (e.g., `rg-github-actions-demo`)

**Option B: Using Azure CLI (Advanced)**

```bash
# Login to Azure
az login

# Create resource group
az group create --name rg-github-actions-demo --location eastus

# Create App Service plan (Free tier)
az appservice plan create \
  --name plan-github-demo \
  --resource-group rg-github-actions-demo \
  --sku F1 \
  --is-linux

# Create Web App
az webapp create \
  --name webapp-github-demo-YOUR-NAME \
  --resource-group rg-github-actions-demo \
  --plan plan-github-demo \
  --runtime "NODE:18-lts"
```

---

#### Part 3: Create Azure Service Principal for GitHub

**Important:** Azure has disabled basic authentication (publish profiles) for security. We'll use the modern **Service Principal** method instead.

**Option A: Using Azure Cloud Shell (Easiest)**

1. **Open Azure Cloud Shell**
   - In Azure Portal, click the **Cloud Shell** icon (>_) in the top toolbar
   - Select **Bash** or **PowerShell** (Bash recommended)
   - Wait for it to initialize

2. **Get your Subscription ID**
   ```bash
   az account show --query id -o tsv
   ```
   
   **Copy this ID** - you'll need it in the next step.

3. **Create Service Principal**
   
   Replace `YOUR-SUBSCRIPTION-ID` and `webapp-github-demo-YOUR-NAME` with your actual values:

   ```bash
   az ad sp create-for-rbac \
     --name "github-actions-demo-sp" \
     --role contributor \
     --scopes /subscriptions/YOUR-SUBSCRIPTION-ID/resourceGroups/rg-github-actions-demo \
     --sdk-auth
   ```

   **Example:**
   ```bash
   az ad sp create-for-rbac \
     --name "github-actions-demo-sp" \
     --role contributor \
     --scopes /subscriptions/c001a7f0-e371-42c4-99b9-1a23848a72a0/resourceGroups/rg-github-actions-demo \
     --sdk-auth
   ```

4. **Copy the entire JSON output**
   
   You'll get output like this:
   ```json
   {
     "clientId": "12345678-1234-1234-1234-123456789abc",
     "clientSecret": "your-secret-here",
     "subscriptionId": "c001a7f0-e371-42c4-99b9-1a23848a72a0",
     "tenantId": "87654321-4321-4321-4321-abcdef123456",
     "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
     "resourceManagerEndpointUrl": "https://management.azure.com/",
     "activeDirectoryGraphResourceId": "https://graph.windows.net/",
     "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
     "galleryEndpointUrl": "https://gallery.azure.com/",
     "managementEndpointUrl": "https://management.core.windows.net/"
   }
   ```

   **⚠️ IMPORTANT:** Copy this ENTIRE JSON output immediately - you won't be able to see the secret again!

**Option B: Using Local Azure CLI**

If you have Azure CLI installed locally:

```bash
# Login to Azure
az login

# Get subscription ID
az account show --query id -o tsv

# Create service principal (replace YOUR-SUBSCRIPTION-ID)
az ad sp create-for-rbac \
  --name "github-actions-demo-sp" \
  --role contributor \
  --scopes /subscriptions/YOUR-SUBSCRIPTION-ID/resourceGroups/rg-github-actions-demo \
  --sdk-auth
```

Copy the entire JSON output.

---

#### Part 4: Configure GitHub Secrets

1. **Add Azure Credentials to GitHub**
   - Go to your GitHub repository: `https://github.com/YOUR-USERNAME/github-actions-azure-demo`
   - Click **Settings** → **Secrets and variables** → **Actions**
   - Click **"New repository secret"**
   
   **First Secret:**
   - **Name:** `AZURE_CREDENTIALS`
   - **Value:** Paste the entire JSON output from the service principal creation
   - Click **Add secret**

2. **Add Web App Name**
   - Click **"New repository secret"** again
   - **Name:** `AZURE_WEBAPP_NAME`
   - **Value:** Your web app name (e.g., `webapp-github-demo-abengtss-max`)
   - Click **Add secret**

3. **Verify your secrets**
   - You should now see both secrets listed:
     - `AZURE_CREDENTIALS` (shows `***`)
     - `AZURE_WEBAPP_NAME` (shows `***`)

---

#### Part 5: Create GitHub Actions Workflow

1. **Create workflow directory**

   In your local repository:
   ```bash
   mkdir -p .github/workflows
   ```

2. **Create deployment workflow**

   Create `.github/workflows/azure-deploy.yml`:

   ```yaml
   name: Deploy to Azure Web App

   on:
     push:
       branches: [ main ]
     workflow_dispatch:  # Allow manual trigger

   jobs:
     build-and-deploy:
       runs-on: ubuntu-latest
       
       steps:
       # Step 1: Checkout code
       - name: 📥 Checkout code
         uses: actions/checkout@v4

       # Step 2: Setup Node.js
       - name: 🔧 Setup Node.js
         uses: actions/setup-node@v4
         with:
           node-version: '18'
           cache: 'npm'

       # Step 3: Install dependencies
       - name: 📦 Install dependencies
         run: npm ci

       # Step 4: Run tests (if any)
       - name: 🧪 Run tests
         run: npm test

       # Step 5: Login to Azure using Service Principal
       - name: 🔐 Azure Login
         uses: azure/login@v1
         with:
           creds: ${{ secrets.AZURE_CREDENTIALS }}

       # Step 6: Deploy to Azure Web App
       - name: 🚀 Deploy to Azure Web App
         uses: azure/webapps-deploy@v2
         with:
           app-name: ${{ secrets.AZURE_WEBAPP_NAME }}
           package: .

       # Step 7: Azure Logout (security best practice)
       - name: 🔓 Azure Logout
         run: az logout
         if: always()

       # Step 8: Verify deployment
       - name: ✅ Verify deployment
         run: |
           echo "🎉 Deployment complete!"
           echo "🌐 Visit: https://${{ secrets.AZURE_WEBAPP_NAME }}.azurewebsites.net"
   ```

3. **Understanding the workflow changes:**
   - ✅ **No hardcoded app name** - uses `${{ secrets.AZURE_WEBAPP_NAME }}`
   - ✅ **Service Principal authentication** - more secure than publish profiles
   - ✅ **Azure logout** - cleans up session after deployment
   - ✅ **Uses secrets for all sensitive data**

4. **Commit and push the workflow**

   ```bash
   git add .github/workflows/azure-deploy.yml
   git commit -m "Add Azure deployment workflow"
   git push origin main
   ```

---

#### Part 6: Monitor and Test the Deployment

1. **Watch the workflow run**
   - Go to your GitHub repository
   - Click the **Actions** tab
   - You should see "Deploy to Azure Web App" workflow running
   - Click on it to watch real-time logs

2. **Verify each step completes:**
   - ✅ Checkout code
   - ✅ Setup Node.js
   - ✅ Install dependencies
   - ✅ Run tests
   - ✅ Azure Login (new with Service Principal)
   - ✅ Deploy to Azure Web App
   - ✅ Azure Logout
   - ✅ Verify deployment

3. **Check for errors**
   - If any step fails (red ❌), click on it to see error details
   - Common issues:
     - `AZURE_CREDENTIALS` secret not added correctly
     - `AZURE_WEBAPP_NAME` secret has wrong value
     - Service Principal doesn't have correct permissions
     - Azure Web App not created properly

4. **Test the deployed application**
   
   Open your browser and visit (replace with YOUR app name):
   ```
   https://webapp-github-demo-abengtss-max.azurewebsites.net
   ```
   
   You should see your colorful deployment success page!

5. **Test the health endpoint**
   ```
   https://webapp-github-demo-abengtss-max.azurewebsites.net/health
   ```
   
   Should return JSON:
   ```json
   {
     "status": "healthy",
     "timestamp": "2026-01-12T..."
   }
   ```

6. **Verify in Azure Portal**
   - Go to Azure Portal
   - Navigate to your Web App
   - Click **Browse** to open the site
   - Check **Deployment Center** to see deployment history
   - You'll see "External Git" as deployment source

---

#### Part 7: Test Continuous Deployment

1. **Make a change to the app**

   Edit `server.js` and update the title:
   ```javascript
   <h1>🎯 Updated via GitHub Actions!</h1>
   <p>This deployment was triggered automatically</p>
   ```

2. **Commit and push**
   ```bash
   git add server.js
   git commit -m "Update welcome message"
   git push origin main
   ```

3. **Watch automatic deployment**
   - Go to Actions tab in GitHub
   - New workflow run should start automatically
   - Wait for completion (~2-3 minutes)
   - Refresh your browser at the Azure URL
   - You should see the updated message!

---

#### Verification Checklist

Make sure you can answer YES to all:

- [ ] ✅ Created Node.js application with package.json and server.js
- [ ] ✅ Pushed code to GitHub repository
- [ ] ✅ Created Azure Web App (Free tier is fine)
- [ ] ✅ Created Azure Service Principal with contributor role
- [ ] ✅ Added `AZURE_CREDENTIALS` as GitHub Secret
- [ ] ✅ Added `AZURE_WEBAPP_NAME` as GitHub Secret
- [ ] ✅ Created GitHub Actions workflow file
- [ ] ✅ Workflow runs successfully (all green ✅)
- [ ] ✅ Can access application at azurewebsites.net URL
- [ ] ✅ Application shows correct content
- [ ] ✅ Made a code change and saw automatic redeployment

---

#### Understanding What Happened

**The CI/CD Pipeline:**
1. **Trigger:** You pushed code to the `main` branch
2. **Build:** GitHub Actions checked out code, installed dependencies, ran tests
3. **Authenticate:** Workflow logs into Azure using Service Principal credentials
4. **Deploy:** Workflow deploys application to App Service
5. **Cleanup:** Logs out of Azure (security best practice)
6. **Verify:** Application is live and accessible via HTTPS

**Key Azure Concepts:**
- **App Service:** Platform-as-a-Service (PaaS) for hosting web apps
- **Service Principal:** Identity for applications to access Azure resources (like a service account)
- **Resource Group:** Logical container for Azure resources
- **App Service Plan:** Defines compute resources (CPU, memory)
- **RBAC (Role-Based Access Control):** Service Principal has "Contributor" role on Resource Group

**Key GitHub Actions Concepts:**
- **Secrets:** Secure storage for sensitive data (credentials, passwords, tokens)
- **Workflow:** Automated process defined in YAML
- **Actions:** Reusable units (`azure/login@v1`, `azure/webapps-deploy@v2`)
- **Service Principal Authentication:** More secure than basic auth (publish profiles)

**Why Service Principal is Better:**
- ✅ More secure - no basic authentication
- ✅ Fine-grained permissions (only access specific resource group)
- ✅ Can be rotated/revoked easily
- ✅ Follows Azure best practices
- ✅ Works with all Azure services

---

#### Troubleshooting Common Issues

**Issue 1: "Azure login failed"**
- ✅ Verify `AZURE_CREDENTIALS` secret contains the entire JSON output
- Check no extra spaces or line breaks were added
- Ensure the JSON is valid (use a JSON validator)
- Verify the Service Principal was created successfully

**Issue 2: "Error: AADSTS700016: Application not found"**
- ✅ Service Principal was deleted or not created properly
- Recreate the Service Principal using the `az ad sp create-for-rbac` command
- Make sure you used the `--sdk-auth` flag

**Issue 3: "Error: The subscription is not registered to use Microsoft.Web"**
- ✅ Your subscription needs to register the resource provider:
  ```bash
  az provider register --namespace Microsoft.Web
  ```
- Wait 5-10 minutes for registration to complete

**Issue 4: "Deployment successful but site shows error"**
- ✅ Check if `package.json` has correct start script
- Verify `PORT` environment variable is used: `process.env.PORT`
- Check Azure Web App logs in Portal → Log stream

**Issue 5: "Workflow doesn't trigger"**
- ✅ Ensure workflow file is in `.github/workflows/` directory
- Verify file has `.yml` or `.yaml` extension
- Check you pushed to the `main` branch

**Issue 6: "npm install fails"**
- ✅ Verify `package.json` is valid JSON
- Check dependencies are available on npm registry

**Issue 7: "Forbidden: The client with object id does not have authorization"**
- ✅ Service Principal doesn't have correct permissions
- Verify the scope includes your resource group in the `az ad sp create-for-rbac` command
- Try giving broader scope: `/subscriptions/YOUR-SUBSCRIPTION-ID`

---

#### Challenge: Add Staging Environment

**Advanced Exercise:** Create a staging slot and deploy to it first!

1. **Upgrade App Service Plan** (Staging requires Standard tier or higher)
   ```bash
   az appservice plan update \
     --name plan-github-demo \
     --resource-group rg-github-actions-demo \
     --sku S1
   ```
   **Note:** This will incur charges (~$70/month). Use only for learning, then delete.

2. **Create deployment slot in Azure:**
   ```bash
   az webapp deployment slot create \
     --name webapp-github-demo-abengtss-max \
     --resource-group rg-github-actions-demo \
     --slot staging
   ```

3. **Update your workflow to deploy to staging first:**
   ```yaml
   jobs:
     deploy-to-staging:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         
         - name: Setup Node.js
           uses: actions/setup-node@v4
           with:
             node-version: '18'
             
         - name: Install dependencies
           run: npm ci
           
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
             
     deploy-to-production:
       needs: deploy-to-staging
       runs-on: ubuntu-latest
       environment: production  # Requires manual approval
       steps:
         - name: Azure Login
           uses: azure/login@v1
           with:
             creds: ${{ secrets.AZURE_CREDENTIALS }}
             
         - name: Swap Staging to Production
           run: |
             az webapp deployment slot swap \
               --name ${{ secrets.AZURE_WEBAPP_NAME }} \
               --resource-group rg-github-actions-demo \
               --slot staging \
               --target-slot production
   ```

4. **Test staging:** `https://webapp-github-demo-abengtss-max-staging.azurewebsites.net`

5. **Set up GitHub Environment protection:**
   - Go to Repository Settings → Environments
   - Create "production" environment
   - Add required reviewers for approval before production deployment

---

#### Clean Up Resources (When Done)

**To avoid Azure charges:**

```bash
# Delete the entire resource group (removes everything)
az group delete --name rg-github-actions-demo --yes --no-wait
```

Or in Azure Portal:
1. Go to Resource Groups
2. Select `rg-github-actions-demo`
3. Click **Delete resource group**
4. Type the name to confirm
5. Click **Delete**

---

#### What's Next?

This exercise gave you hands-on experience with:
- ✅ Creating a deployable application
- ✅ Setting up Azure infrastructure
- ✅ Configuring CI/CD pipeline with GitHub Actions
- ✅ Managing secrets securely
- ✅ Automated testing and deployment

**You're now ready for Lab 2** where you'll build more complex workflows!

## 📝 Knowledge Check

Answer these questions to test your understanding:

1. **What are the three types of GitHub Actions?**
   <details>
   <summary>Click to reveal answer</summary>
   Container Actions, JavaScript Actions, and Composite Actions
   </details>

2. **What is the purpose of a runner in GitHub Actions?**
   <details>
   <summary>Click to reveal answer</summary>
   A runner is a server that executes your workflows. It can be GitHub-hosted or self-hosted.
   </details>

3. **Which runner is most cost-effective for standard builds?**
   <details>
   <summary>Click to reveal answer</summary>
   ubuntu-latest (Linux runners use 1x minute multiplier)
   </details>

4. **How do you manually trigger a workflow?**
   <details>
   <summary>Click to reveal answer</summary>
   Use the workflow_dispatch event trigger and run it from the Actions tab
   </details>

5. **What's the best practice for versioning actions in workflows?**
   <details>
   <summary>Click to reveal answer</summary>
   Use specific versions (commit SHA or release tags) like @v4 or @sha-hash for security and stability
   </details>

## 🎓 Lab Summary

**Congratulations!** You've completed Lab 1. You should now understand:

✅ What GitHub Actions are and their benefits  
✅ The three types of actions (Container, JavaScript, Composite)  
✅ Workflow anatomy (events, jobs, steps, actions)  
✅ GitHub-hosted vs self-hosted runners  
✅ Various trigger events and when to use them  
✅ Usage limits and cost optimization strategies  
✅ Azure integration overview  

## 🔜 Next Steps

Proceed to **[Lab 2: Creating Your First GitHub Actions Workflow](../Lab2-First-Workflow/README.md)** where you'll:
- Create a complete CI workflow
- Integrate with Azure services
- Implement secret management
- Deploy to Azure Web App

## 📚 Additional Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [GitHub Actions Marketplace](https://github.com/marketplace?type=actions)
- [Azure Actions on GitHub](https://github.com/Azure/actions)
- [Workflow Syntax Reference](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions)

---

**Need Help?** Review the official documentation or reach out to the GitHub community.
