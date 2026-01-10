# Lab 10: Enterprise Management & Self-Hosted Runners

**Duration:** 70 minutes  
**Difficulty:** Advanced  
**Prerequisites:** All previous labs (1-9)

## 🎯 Learning Objectives

By the end of this lab, you will be able to:
- Understand GitHub Actions in enterprise contexts
- Set up and configure self-hosted runners
- Manage runner groups and access control
- Implement organization-level workflow templates
- Manage encrypted secrets at enterprise/organization level
- Configure runner security and best practices
- Monitor and maintain self-hosted runners
- Integrate self-hosted runners with Azure infrastructure
- Understand enterprise billing and usage limits

---

## 📖 GitHub Enterprise Models

### GitHub Plans

| Feature | Free | Team | Enterprise |
|---------|------|------|------------|
| **Public repos** | Unlimited | Unlimited | Unlimited |
| **Private repos** | Unlimited | Unlimited | Unlimited |
| **Actions minutes** | 2,000/month | 3,000/month | 50,000/month |
| **Storage** | 500 MB | 2 GB | 50 GB |
| **Self-hosted runners** | Yes | Yes | Yes |
| **Runner groups** | No | No | Yes |
| **Organization secrets** | Yes | Yes | Yes |
| **Required workflows** | No | No | Yes |
| **Policy controls** | Limited | Limited | Full |

### Enterprise Cloud vs Server

**Enterprise Cloud (GHEC):**
- ☁️ Hosted by GitHub
- 🔄 Automatic updates
- 🌐 Global availability
- 💰 Pay-per-use

**Enterprise Server (GHES):**
- 🏢 Self-hosted infrastructure
- 🔒 Complete data control
- 🔧 Custom configurations
- 📦 On-premises deployment

---

## 📖 Why Self-Hosted Runners?

### Use Cases for Self-Hosted Runners

**When to use GitHub-hosted:**
- ✅ Standard CI/CD workflows
- ✅ Quick setup, no maintenance
- ✅ Public repositories
- ✅ Standard hardware sufficient

**When to use self-hosted:**
- ✅ Specific hardware requirements (GPU, large RAM)
- ✅ Access to private networks/databases
- ✅ Compliance requirements
- ✅ Cost optimization for heavy usage
- ✅ Custom software/tools pre-installed
- ✅ Faster builds with local caching

### Cost Comparison

**GitHub-hosted runners:**
```
Linux:   $0.008/minute
Windows: $0.016/minute
macOS:   $0.08/minute

Example: 100 hours/month of Linux builds
= 6,000 minutes × $0.008 = $48/month
```

**Self-hosted runners:**
```
Azure VM (Standard_D2s_v3):
$70/month (2 vCPU, 8 GB RAM) 24/7

Savings depend on usage patterns
Break-even: ~145 hours/month
```

---

## 🏗️ Lab Architecture

```
Enterprise/Organization
    ↓
Runner Groups (by team/environment)
    ↓
Self-Hosted Runners (Azure VMs)
    ↓
Workflows → Jobs → Self-Hosted Runner → Build/Deploy
    ↓
Azure Resources (private network access)
```

---

## 📋 Prerequisites

Before starting this lab:

1. ✅ Completed all previous labs (1-9)
2. ✅ GitHub organization (create free if needed)
3. ✅ Azure subscription (for runner VMs)
4. ✅ Admin access to GitHub organization
5. ✅ Understanding of VMs and networking
6. ✅ PowerShell or Bash experience

---

## 🚀 Exercise 1: Setup Self-Hosted Runner on Azure VM

### What You'll Build
A self-hosted runner on an Azure VM that can execute workflows with access to private Azure resources.

### Step 1: Create Azure VM for Runner

```powershell
# Login to Azure
az login

# Create resource group
az group create `
  --name rg-github-runners `
  --location eastus

# Create virtual network
az network vnet create `
  --resource-group rg-github-runners `
  --name vnet-runners `
  --address-prefix 10.0.0.0/16 `
  --subnet-name subnet-runners `
  --subnet-prefix 10.0.0.0/24

# Create network security group
az network nsg create `
  --resource-group rg-github-runners `
  --name nsg-runners

# Allow outbound HTTPS (required for GitHub)
az network nsg rule create `
  --resource-group rg-github-runners `
  --nsg-name nsg-runners `
  --name AllowHTTPS `
  --priority 100 `
  --destination-port-ranges 443 `
  --direction Outbound `
  --access Allow

# Create VM
az vm create `
  --resource-group rg-github-runners `
  --name vm-runner-01 `
  --image Ubuntu2204 `
  --size Standard_D2s_v3 `
  --admin-username azureuser `
  --generate-ssh-keys `
  --vnet-name vnet-runners `
  --subnet subnet-runners `
  --nsg nsg-runners `
  --public-ip-sku Standard

# Get public IP
$PUBLIC_IP = az vm show -d `
  --resource-group rg-github-runners `
  --name vm-runner-01 `
  --query publicIps -o tsv

Write-Host "VM created with IP: $PUBLIC_IP"
```

### Step 2: Configure VM for Runner

SSH into the VM:

```powershell
ssh azureuser@$PUBLIC_IP
```

Inside the VM, run:

```bash
# Update system
sudo apt-get update
sudo apt-get upgrade -y

# Install required dependencies
sudo apt-get install -y curl git jq libicu-dev

# Install Docker (useful for container builds)
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker azureuser

# Install Node.js (for Node.js projects)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install Azure CLI
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Create runner directory
mkdir -p ~/actions-runner
cd ~/actions-runner
```

### Step 3: Register Runner with GitHub

1. **Get runner registration token**:
   - Go to your GitHub organization
   - Click **Settings** → **Actions** → **Runners**
   - Click **New self-hosted runner**
   - Select **Linux** and **x64**
   - Copy the commands

2. **On the VM, download and configure**:

```bash
# Download latest runner
curl -o actions-runner-linux-x64-2.311.0.tar.gz -L \
  https://github.com/actions/runner/releases/download/v2.311.0/actions-runner-linux-x64-2.311.0.tar.gz

# Extract
tar xzf ./actions-runner-linux-x64-2.311.0.tar.gz

# Configure runner
./config.sh --url https://github.com/YOUR_ORG_OR_USER --token YOUR_TOKEN

# When prompted:
# - Runner name: vm-runner-01
# - Runner group: Default
# - Labels: self-hosted,Linux,X64,azure
# - Work folder: _work
```

### Step 4: Install Runner as Service

```bash
# Install as service (runs on boot)
sudo ./svc.sh install

# Start service
sudo ./svc.sh start

# Check status
sudo ./svc.sh status

# View logs
journalctl -u actions.runner.* -f
```

### Step 5: Test Self-Hosted Runner

Create a test repository with `.github/workflows/test-runner.yml`:

```yaml
name: Test Self-Hosted Runner

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  test-runner:
    name: Test on Self-Hosted
    runs-on: self-hosted  # Uses your runner!
    
    steps:
      - name: Check runner info
        run: |
          echo "Runner name: $RUNNER_NAME"
          echo "Runner OS: $RUNNER_OS"
          echo "Runner arch: $RUNNER_ARCH"
          echo "Hostname: $(hostname)"
      
      - name: Check installed tools
        run: |
          echo "Node version: $(node --version)"
          echo "npm version: $(npm --version)"
          echo "Docker version: $(docker --version)"
          echo "Azure CLI version: $(az --version | head -n1)"
      
      - name: Test Azure access
        run: |
          echo "Testing Azure connectivity..."
          az account show || echo "Not logged in to Azure"
      
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Run build
        run: |
          echo "Building on self-hosted runner..."
          # Your build commands here
```

Push and verify the workflow runs on your self-hosted runner!

---

## 🚀 Exercise 2: Runner Groups and Access Control

### What You'll Build
Organize runners into groups with specific access policies for different teams.

### Step 1: Create Runner Groups

**Note:** Runner groups require GitHub Enterprise or organization plan.

1. Go to **Organization Settings** → **Actions** → **Runner groups**
2. Click **New runner group**

**Create groups:**

**Group 1: Production Runners**
- Name: `production-runners`
- Access: Only selected repositories
- Repositories: Add production repos
- Workflow access: All workflows

**Group 2: Development Runners**
- Name: `development-runners`
- Access: All repositories
- Workflow access: All workflows

**Group 3: Security Scanning**
- Name: `security-runners`
- Access: All repositories
- Workflow access: Selected workflows only

### Step 2: Assign Runners to Groups

When configuring a new runner:

```bash
./config.sh --url https://github.com/YOUR_ORG \
  --token YOUR_TOKEN \
  --runnergroup "production-runners" \
  --labels "self-hosted,Linux,X64,production,azure"
```

Or move existing runners via UI:
- Settings → Actions → Runners
- Click runner → Change group

### Step 3: Use Specific Runner Groups

```yaml
name: Production Deployment

on:
  push:
    branches: [main]

jobs:
  deploy:
    name: Deploy to Production
    runs-on: 
      group: production-runners
      labels: [self-hosted, Linux, azure]
    
    steps:
      - name: Deploy
        run: echo "Running on production runner"
```

### Step 4: Implement Runner Access Policies

Create organization-level policy file:

```yaml
# .github/workflows/runner-policy.yml
name: Runner Access Policy

on:
  workflow_dispatch:
    inputs:
      enforce:
        description: 'Enforce runner policies'
        required: true
        type: boolean

jobs:
  enforce-policy:
    runs-on: ubuntu-latest
    steps:
      - name: Check runner compliance
        run: |
          echo "Checking runner policies..."
          # Add policy checks here
          
          # Example policies:
          # - Production jobs must use production-runners group
          # - External contributors cannot use self-hosted runners
          # - Sensitive workflows require specific runner labels
```

---

## 🚀 Exercise 3: Organization-Level Secrets Management

### What You'll Build
Centralized secrets management for organization-wide access.

### Step 1: Create Organization Secrets

1. **Go to Organization Settings** → **Secrets and variables** → **Actions**
2. Click **New organization secret**

**Create these secrets:**

```
AZURE_CREDENTIALS
  - Value: Service principal JSON
  - Access: All repositories

AZURE_SUBSCRIPTION_ID
  - Value: Subscription ID
  - Access: All repositories

PROD_AZURE_CREDENTIALS
  - Value: Production service principal JSON
  - Access: Selected repositories only (production repos)

NPM_TOKEN
  - Value: npm auth token
  - Access: All repositories

DOCKER_USERNAME
DOCKER_PASSWORD
  - Access: All repositories
```

### Step 2: Use Organization Secrets

```yaml
name: Deploy with Org Secrets

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Azure Login
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}  # Organization secret
      
      - name: Deploy
        run: |
          az webapp deploy \
            --name myapp \
            --resource-group ${{ secrets.AZURE_RESOURCE_GROUP }}
```

### Step 3: Environment-Specific Secrets

```yaml
name: Multi-Environment Deploy

on:
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        options: [dev, staging, prod]

jobs:
  deploy:
    runs-on: self-hosted
    environment: ${{ github.event.inputs.environment }}
    
    steps:
      - name: Deploy to ${{ github.event.inputs.environment }}
        run: |
          echo "Deploying to ${{ github.event.inputs.environment }}"
          # Uses environment-specific secrets automatically
        env:
          AZURE_CREDS: ${{ secrets.AZURE_CREDENTIALS }}
```

### Step 4: Secrets Rotation Workflow

```yaml
name: Rotate Secrets

on:
  schedule:
    - cron: '0 0 1 * *'  # Monthly
  workflow_dispatch:

jobs:
  rotate:
    runs-on: ubuntu-latest
    permissions:
      secrets: write
    
    steps:
      - name: Check secret age
        run: |
          echo "Checking organization secrets..."
          # List secrets and check last updated date
          
          echo "⚠️ Secrets older than 90 days should be rotated"
      
      - name: Create rotation issue
        uses: actions/github-script@v7
        with:
          script: |
            await github.rest.issues.create({
              owner: context.repo.owner,
              repo: 'infrastructure',
              title: 'Monthly Secret Rotation Reminder',
              body: `## 🔐 Secret Rotation Required
              
              Please rotate the following secrets:
              - [ ] AZURE_CREDENTIALS
              - [ ] NPM_TOKEN
              - [ ] DOCKER_PASSWORD
              
              See [Secret Rotation Guide](docs/secret-rotation.md)`
            });
```

---

## 🚀 Exercise 4: Required Workflows (Enterprise)

### What You'll Build
Organization-wide required workflows that must run on all repositories.

**Note:** This feature requires GitHub Enterprise Cloud.

### Step 1: Create Required Workflow Repository

1. Create repository named `.github` in your organization
2. Create `.github/workflows/required-security-scan.yml`:

```yaml
name: Required Security Scan

on:
  pull_request:
    branches: [main, master]
  push:
    branches: [main, master]

jobs:
  security-scan:
    name: Security Scan
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'
      
      - name: Upload to Security tab
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
      
      - name: Check for critical vulnerabilities
        run: |
          CRITICAL=$(cat trivy-results.sarif | jq '[.runs[].results[] | select(.level == "error")] | length')
          if [ "$CRITICAL" -gt 0 ]; then
            echo "❌ Found $CRITICAL critical vulnerabilities"
            exit 1
          fi
          echo "✅ No critical vulnerabilities found"
```

### Step 2: Configure as Required

1. Go to **Organization Settings** → **Actions** → **General**
2. Scroll to **Required workflows**
3. Add `.github/.github/workflows/required-security-scan.yml`
4. Select repositories (All or Selected)

Now this workflow will run on every PR/push across selected repositories!

---

## 🚀 Exercise 5: Runner Monitoring and Maintenance

### What You'll Build
Monitoring, alerting, and automated maintenance for self-hosted runners.

### Step 1: Runner Health Check Workflow

```yaml
name: Runner Health Check

on:
  schedule:
    - cron: '*/30 * * * *'  # Every 30 minutes
  workflow_dispatch:

jobs:
  check-runners:
    name: Check Runner Health
    runs-on: ubuntu-latest
    
    steps:
      - name: List all runners
        id: runners
        uses: actions/github-script@v7
        with:
          script: |
            const { data: runners } = await github.rest.actions.listSelfHostedRunnersForOrg({
              org: context.repo.owner,
              per_page: 100
            });
            
            const summary = {
              total: runners.total_count,
              online: runners.runners.filter(r => r.status === 'online').length,
              offline: runners.runners.filter(r => r.status === 'offline').length,
              busy: runners.runners.filter(r => r.busy).length
            };
            
            console.log('Runner Summary:', summary);
            
            // Check for offline runners
            const offline = runners.runners.filter(r => r.status === 'offline');
            if (offline.length > 0) {
              core.warning(`${offline.length} runners are offline`);
              offline.forEach(r => {
                console.log(`Offline: ${r.name} (${r.labels.map(l => l.name).join(', ')})`);
              });
            }
            
            return summary;
      
      - name: Create alert if runners offline
        if: steps.runners.outputs.offline > 0
        uses: actions/github-script@v7
        with:
          script: |
            await github.rest.issues.create({
              owner: context.repo.owner,
              repo: 'infrastructure',
              title: '🚨 Self-Hosted Runners Offline',
              body: `## Runner Health Alert
              
              **Offline Runners:** ${{ steps.runners.outputs.offline }}
              **Total Runners:** ${{ steps.runners.outputs.total }}
              
              Please investigate and restore offline runners.
              
              Check runner status: ${context.serverUrl}/${context.repo.owner}/settings/actions/runners`,
              labels: ['infrastructure', 'urgent']
            });
```

### Step 2: Runner Update Script

Create script to update runners:

```bash
#!/bin/bash
# update-runner.sh

RUNNER_VERSION="2.311.0"
RUNNER_DIR="$HOME/actions-runner"

cd $RUNNER_DIR

# Stop runner
sudo ./svc.sh stop

# Download new version
curl -o actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz -L \
  https://github.com/actions/runner/releases/download/v${RUNNER_VERSION}/actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz

# Backup old version
tar czf backup-$(date +%Y%m%d).tar.gz bin externals

# Extract new version
tar xzf actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz

# Start runner
sudo ./svc.sh start

echo "Runner updated to version ${RUNNER_VERSION}"
```

### Step 3: Automated Runner Maintenance

```yaml
name: Runner Maintenance

on:
  schedule:
    - cron: '0 2 * * 0'  # Weekly on Sunday at 2 AM
  workflow_dispatch:

jobs:
  maintenance:
    name: Maintain Runners
    runs-on: self-hosted
    
    steps:
      - name: Clean up workspace
        run: |
          echo "Cleaning up old workflow runs..."
          find /home/azureuser/actions-runner/_work -type d -mtime +7 -exec rm -rf {} +
      
      - name: Update system packages
        run: |
          sudo apt-get update
          sudo apt-get upgrade -y
          sudo apt-get autoremove -y
          sudo apt-get autoclean
      
      - name: Check disk space
        run: |
          df -h
          DISK_USAGE=$(df -h / | awk 'NR==2 {print $5}' | sed 's/%//')
          
          if [ "$DISK_USAGE" -gt 80 ]; then
            echo "⚠️ Disk usage above 80%"
            # Trigger cleanup or alert
          fi
      
      - name: Restart runner service
        run: |
          sudo systemctl restart actions.runner.*
      
      - name: Report maintenance complete
        run: echo "✅ Maintenance completed successfully"
```

---

## 📚 Best Practices

### 1. Runner Security

```bash
# Run runner as dedicated user (not root)
sudo useradd -m -s /bin/bash github-runner
sudo su - github-runner

# Restrict runner directory permissions
chmod 700 ~/actions-runner

# Use dedicated service account for Azure access
# Don't store credentials on runner filesystem
```

### 2. Network Security

```yaml
# Restrict runner access
- Self-hosted runners should only access necessary resources
- Use Azure Private Link for Azure services
- Implement network security groups (NSGs)
- Use VPN/ExpressRoute for on-premises access
```

### 3. Runner Labels

```yaml
runs-on:
  group: production-runners
  labels:
    - self-hosted
    - linux
    - azure
    - x64
    - gpu          # Has GPU
    - high-memory  # 32GB+ RAM
    - docker       # Docker installed
```

### 4. Ephemeral Runners

For maximum security, use ephemeral runners:

```bash
# Configure runner to run only one job
./config.sh --ephemeral --url URL --token TOKEN

# After each job, runner de-registers automatically
# Spin up new runner for next job
```

### 5. Auto-Scaling Runners

Use Azure VM Scale Sets:

```powershell
# Create scale set
az vmss create `
  --resource-group rg-github-runners `
  --name vmss-runners `
  --image Ubuntu2204 `
  --instance-count 2 `
  --vm-sku Standard_D2s_v3 `
  --load-balancer "" `
  --custom-data cloud-init-runner.txt

# Auto-scale based on metrics
az monitor autoscale create `
  --resource-group rg-github-runners `
  --resource vmss-runners `
  --resource-type Microsoft.Compute/virtualMachineScaleSets `
  --name runner-autoscale `
  --min-count 1 `
  --max-count 10 `
  --count 2
```

---

## 🔍 Troubleshooting

### Issue: Runner Won't Start

```bash
# Check service status
sudo systemctl status actions.runner.*

# Check logs
journalctl -u actions.runner.* -n 100

# Check permissions
ls -la ~/actions-runner/

# Restart service
sudo systemctl restart actions.runner.*
```

### Issue: Jobs Not Picking Up Self-Hosted Runner

**Check:**
1. Runner is online in GitHub Settings → Runners
2. Runner labels match `runs-on` specification
3. Runner has access to repository (check runner groups)
4. No queued jobs blocking (check queue)

### Issue: Runner Out of Disk Space

```bash
# Check disk usage
df -h

# Clean Docker
docker system prune -a -f

# Clean old workflows
find _work -type d -mtime +7 -exec rm -rf {} +

# Clean apt cache
sudo apt-get clean
```

---

## ✅ Knowledge Check

### Question 1
When should you use self-hosted runners?

<details>
<summary>Answer</summary>

Use self-hosted runners when you need:
- Specific hardware (GPU, large RAM)
- Private network access
- Custom software pre-installed
- Cost optimization for heavy usage
- Compliance requirements
</details>

### Question 2
How do you restrict runner access to specific repositories?

<details>
<summary>Answer</summary>

Use runner groups (Enterprise feature):
1. Create runner group
2. Set access to "Selected repositories"
3. Add specific repositories
4. Assign runners to group
</details>

### Question 3
What's the recommended way to run self-hosted runners?

<details>
<summary>Answer</summary>

Best practices:
- Run as a service (not interactively)
- Use dedicated user account (not root)
- Use ephemeral runners for security
- Implement auto-scaling for demand
- Regular updates and maintenance
</details>

---

## 🎯 Challenge Exercises

### Challenge 1: GPU Runner
Set up an Azure VM with GPU for machine learning workflows.

### Challenge 2: Auto-Scaling
Implement Azure VM Scale Set with automatic scaling based on queue depth.

### Challenge 3: Multi-Cloud Runners
Set up runners in both Azure and AWS with intelligent routing.

---

## 📖 Additional Resources

- [Self-Hosted Runners](https://docs.github.com/en/actions/hosting-your-own-runners)
- [Managing Self-Hosted Runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners)
- [Enterprise Admin Guide](https://docs.github.com/en/enterprise-cloud@latest/admin)
- [Runner Security](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners#self-hosted-runner-security)

---

## 📝 Summary

In this lab, you learned:
- ✅ GitHub enterprise features and plans
- ✅ Setting up self-hosted runners on Azure VMs
- ✅ Creating and managing runner groups
- ✅ Organization-level secrets management
- ✅ Implementing required workflows
- ✅ Runner monitoring and maintenance
- ✅ Security best practices
- ✅ Auto-scaling strategies

**Congratulations!** You've completed all 10 labs covering:
1. GitHub Actions Fundamentals
2. CI/CD Workflows
3. Container Actions
4. Advanced Triggers
5. JavaScript Actions
6. Complete Pipelines
7. GitHub Script
8. GitHub Packages
9. Reusable Workflows
10. Enterprise & Self-Hosted Runners

You're now ready for GitHub Actions certification! 🎓

---

**Estimated Time to Complete:** 70 minutes  
**Difficulty:** Advanced

Excellent work completing the entire lab series! 🚀🎉
