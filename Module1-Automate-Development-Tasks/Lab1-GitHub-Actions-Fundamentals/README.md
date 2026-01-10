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

### Exercise 6: Azure Integration Overview

**Objective:** Understand how GitHub Actions integrates with Azure services.

#### Steps:

1. **Common Azure Integration Patterns**

   **Pattern 1: Azure Authentication**
   ```yaml
   - name: Login to Azure
     uses: azure/login@v1
     with:
       creds: ${{ secrets.AZURE_CREDENTIALS }}
   ```

   **Pattern 2: Deploy to Azure Web App**
   ```yaml
   - name: Deploy to Azure Web App
     uses: azure/webapps-deploy@v2
     with:
       app-name: 'my-app'
       publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
   ```

   **Pattern 3: Deploy ARM Templates**
   ```yaml
   - name: Deploy Azure Resources
     uses: azure/arm-deploy@v1
     with:
       subscriptionId: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
       resourceGroupName: my-resource-group
       template: ./azure/template.json
   ```

2. **Azure Services Commonly Automated with GitHub Actions**

   | Azure Service | Use Case | Action |
   |---------------|----------|--------|
   | App Service | Web app deployment | `azure/webapps-deploy` |
   | AKS | Kubernetes deployment | `azure/aks-set-context` |
   | Container Registry | Push Docker images | `azure/docker-login` |
   | Functions | Serverless deployment | `azure/functions-action` |
   | Static Web Apps | Static site deployment | `Azure/static-web-apps-deploy` |
   | Storage | Upload artifacts | `azure/CLI` with blob commands |

3. **Security Best Practice: Using Secrets**

   **Never hardcode credentials!** Instead:
   
   - Store sensitive data in GitHub Secrets
   - Access via `${{ secrets.SECRET_NAME }}`
   - Secrets are encrypted and never exposed in logs

   **We'll practice this in upcoming labs.**

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
