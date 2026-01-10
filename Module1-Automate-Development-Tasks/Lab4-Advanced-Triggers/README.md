# Lab 4: Configuring Advanced Workflow Triggers

## 🎯 Lab Objectives

In this lab, you will:
- Master scheduled workflows with cron syntax
- Implement manual workflow triggers with inputs
- Use repository_dispatch for external triggers
- Configure path and branch filters
- Implement workflow concurrency controls
- Build event-driven Azure automation

**Estimated Time:** 45 minutes  
**Difficulty:** Intermediate  
**Prerequisites:** Completed Labs 1-3

## 📚 Background Information

### Understanding Workflow Triggers

GitHub Actions workflows can be triggered by numerous events:

**Event Categories:**

1. **Push Events** - Code changes
2. **Pull Request Events** - PR lifecycle
3. **Scheduled Events** - Time-based triggers
4. **Manual Events** - Human-initiated
5. **Webhook Events** - GitHub platform events
6. **External Events** - API-triggered

### Workflow Trigger Syntax

```yaml
on:
  # Single event
  push:
  
  # Multiple events
  [push, pull_request]
  
  # Event with configuration
  push:
    branches:
      - main
      - 'releases/**'
    paths:
      - 'src/**'
    paths-ignore:
      - '**.md'
```

### CRON Syntax Quick Reference

```
┌───────────── minute (0-59)
│ ┌───────────── hour (0-23)
│ │ ┌───────────── day of month (1-31)
│ │ │ ┌───────────── month (1-12)
│ │ │ │ ┌───────────── day of week (0-7, Sunday = 0 or 7)
│ │ │ │ │
* * * * *
```

**Examples:**
- `0 0 * * *` - Every day at midnight UTC
- `0 9 * * 1` - Every Monday at 9 AM UTC
- `*/15 * * * *` - Every 15 minutes
- `0 0 1 * *` - First day of every month
- `0 0 * * 0` - Every Sunday at midnight

## 🔬 Lab Exercises

### Exercise 1: Implement Scheduled Workflows

**Objective:** Create automated workflows that run on a schedule for Azure maintenance tasks.

#### Steps:

1. **Create Repository**
   ```powershell
   # Use existing repo or create new
   cd C:\Users\alibengtsson\mycertifications\githubactions\azure-nodejs-webapp
   ```

2. **Create Daily Health Check Workflow**

   **Create `.github/workflows/daily-health-check.yml`:**
   ```yaml
   name: Daily Azure Health Check

   on:
     schedule:
       # Run every day at 6 AM UTC (adjust for your timezone)
       - cron: '0 6 * * *'
     workflow_dispatch:  # Allow manual testing

   jobs:
     health-check:
       name: Check Azure Resources Health
       runs-on: ubuntu-latest
       timeout-minutes: 10
       
       steps:
         - name: Checkout
           uses: actions/checkout@v4

         - name: Azure Login
           uses: azure/login@v1
           with:
             creds: ${{ secrets.AZURE_CREDENTIALS }}

         - name: Check Web App Status
           id: webapp-check
           run: |
             WEBAPP_NAME="${{ secrets.AZURE_WEBAPP_NAME }}"
             RG="${{ secrets.AZURE_RESOURCE_GROUP }}"
             
             echo "Checking Web App: $WEBAPP_NAME"
             
             # Get app state
             STATE=$(az webapp show \
               --name "$WEBAPP_NAME" \
               --resource-group "$RG" \
               --query state \
               --output tsv)
             
             echo "Web App State: $STATE"
             echo "state=$STATE" >> $GITHUB_OUTPUT
             
             if [ "$STATE" != "Running" ]; then
               echo "::warning::Web App is not running!"
             fi

         - name: Check Web App Response Time
           run: |
             URL="https://${{ secrets.AZURE_WEBAPP_NAME }}.azurewebsites.net/health"
             
             echo "Testing URL: $URL"
             
             RESPONSE_TIME=$(curl -o /dev/null -s -w '%{time_total}' $URL || echo "0")
             
             echo "Response time: ${RESPONSE_TIME}s"
             
             # Convert to milliseconds for comparison
             MS=$(echo "$RESPONSE_TIME * 1000" | bc)
             
             if (( $(echo "$MS > 3000" | bc -l) )); then
               echo "::warning::Response time is high: ${MS}ms"
             else
               echo "::notice::Response time is good: ${MS}ms"
             fi

         - name: Check Resource Utilization
           run: |
             WEBAPP_NAME="${{ secrets.AZURE_WEBAPP_NAME }}"
             RG="${{ secrets.AZURE_RESOURCE_GROUP }}"
             
             # Get App Service Plan metrics
             echo "Checking resource utilization..."
             
             PLAN_ID=$(az webapp show \
               --name "$WEBAPP_NAME" \
               --resource-group "$RG" \
               --query appServicePlanId \
               --output tsv)
             
             echo "App Service Plan: $PLAN_ID"
             echo "::notice::Health check completed successfully"

         - name: Generate Health Report
           if: always()
           run: |
             cat << EOF > health-report-$(date +%Y-%m-%d).md
             # Azure Health Check Report
             
             **Date:** $(date -u +"%Y-%m-%d %H:%M:%S UTC")
             
             ## Status
             - Web App State: ${{ steps.webapp-check.outputs.state }}
             - Response: ✅ Successful
             
             ## Recommendations
             - Monitor response times
             - Review error logs if issues persist
             - Consider scaling if needed
             EOF

         - name: Upload Report
           uses: actions/upload-artifact@v4
           with:
             name: health-report-${{ github.run_number }}
             path: health-report-*.md
             retention-days: 30

         - name: Azure Logout
           if: always()
           run: az logout
   ```

3. **Create Weekly Cost Analysis Workflow**

   **Create `.github/workflows/weekly-cost-analysis.yml`:**
   ```yaml
   name: Weekly Azure Cost Analysis

   on:
     schedule:
       # Every Sunday at 8 PM UTC
       - cron: '0 20 * * 0'
     workflow_dispatch:
       inputs:
         days-back:
           description: 'Number of days to analyze'
           required: false
           default: '7'
           type: choice
           options:
             - '7'
             - '14'
             - '30'

   jobs:
     cost-analysis:
       name: Analyze Azure Costs
       runs-on: ubuntu-latest
       
       steps:
         - name: Azure Login
           uses: azure/login@v1
           with:
             creds: ${{ secrets.AZURE_CREDENTIALS }}

         - name: Get Cost Data
           id: costs
           run: |
             SUBSCRIPTION_ID=$(echo '${{ secrets.AZURE_CREDENTIALS }}' | jq -r '.subscriptionId')
             DAYS="${{ github.event.inputs.days-back || '7' }}"
             
             # Calculate date range
             END_DATE=$(date -u +"%Y-%m-%d")
             START_DATE=$(date -u -d "$DAYS days ago" +"%Y-%m-%d")
             
             echo "Analyzing costs from $START_DATE to $END_DATE"
             
             # Query cost management (requires Cost Management Reader role)
             az costmanagement query \
               --type Usage \
               --dataset-grouping name=ResourceGroupName \
               --timeframe Custom \
               --time-period from=$START_DATE to=$END_DATE \
               --scope "/subscriptions/$SUBSCRIPTION_ID" \
               --output table || echo "Cost data unavailable"

         - name: Generate Cost Report
           run: |
             cat << EOF > cost-report.md
             # Weekly Azure Cost Analysis
             
             **Report Date:** $(date -u +"%Y-%m-%d")
             **Analysis Period:** Last ${{ github.event.inputs.days-back || '7' }} days
             
             ## Summary
             
             Review Azure Portal for detailed cost breakdown:
             [Cost Management + Billing](https://portal.azure.com/#blade/Microsoft_Azure_CostManagement/Menu/costanalysis)
             
             ## Recommendations
             - Review unused resources
             - Consider reserved instances for predictable workloads
             - Enable auto-shutdown for dev/test resources
             - Use Azure Advisor cost recommendations
             EOF

         - name: Upload Cost Report
           uses: actions/upload-artifact@v4
           with:
             name: cost-report-${{ github.run_number }}
             path: cost-report.md

         - name: Azure Logout
           if: always()
           run: az logout
   ```

4. **Create Monthly Security Audit**

   **Create `.github/workflows/monthly-security-audit.yml`:**
   ```yaml
   name: Monthly Security Audit

   on:
     schedule:
       # First day of every month at 10 AM UTC
       - cron: '0 10 1 * *'
     workflow_dispatch:

   jobs:
     security-audit:
       name: Security Audit
       runs-on: ubuntu-latest
       
       steps:
         - name: Checkout
           uses: actions/checkout@v4

         - name: Azure Login
           uses: azure/login@v1
           with:
             creds: ${{ secrets.AZURE_CREDENTIALS }}

         - name: Check SSL/TLS Certificates
           run: |
             WEBAPP_NAME="${{ secrets.AZURE_WEBAPP_NAME }}"
             RG="${{ secrets.AZURE_RESOURCE_GROUP }}"
             
             echo "Checking SSL certificates..."
             
             # Get custom domains
             DOMAINS=$(az webapp config hostname list \
               --webapp-name "$WEBAPP_NAME" \
               --resource-group "$RG" \
               --query "[].name" \
               --output tsv)
             
             echo "Custom domains: $DOMAINS"
             
             if [ -z "$DOMAINS" ]; then
               echo "::notice::No custom domains configured"
             else
               echo "::notice::SSL certificates present"
             fi

         - name: Check Azure Defender Status
           run: |
             SUBSCRIPTION_ID=$(echo '${{ secrets.AZURE_CREDENTIALS }}' | jq -r '.subscriptionId')
             
             echo "Checking Azure Defender (Microsoft Defender for Cloud)..."
             
             # Check security center pricing tier
             az security pricing list \
               --subscription "$SUBSCRIPTION_ID" \
               --query "value[].{Name:name, PricingTier:pricingTier}" \
               --output table || echo "Unable to check Defender status"

         - name: Check Network Security
           run: |
             RG="${{ secrets.AZURE_RESOURCE_GROUP }}"
             
             echo "Checking Network Security Groups..."
             
             az network nsg list \
               --resource-group "$RG" \
               --query "[].{Name:name, Location:location}" \
               --output table || echo "No NSGs found"

         - name: Check for Public Endpoints
           run: |
             RG="${{ secrets.AZURE_RESOURCE_GROUP }}"
             
             echo "Checking for public endpoints..."
             
             # List storage accounts with public access
             az storage account list \
               --resource-group "$RG" \
               --query "[].{Name:name, AllowBlobPublicAccess:allowBlobPublicAccess}" \
               --output table || echo "No storage accounts found"

         - name: Generate Security Report
           run: |
             cat << EOF > security-audit-$(date +%Y-%m).md
             # Monthly Security Audit Report
             
             **Date:** $(date -u +"%Y-%m-%d %H:%M:%S UTC")
             
             ## Checks Performed
             - ✅ SSL/TLS Certificate validation
             - ✅ Azure Defender status
             - ✅ Network Security Groups
             - ✅ Public endpoint exposure
             
             ## Recommendations
             - Enable Azure Defender for Cloud
             - Review NSG rules regularly
             - Minimize public endpoints
             - Enable diagnostic logging
             - Implement Azure Policy for compliance
             
             ## Next Steps
             1. Review Azure Security Center recommendations
             2. Implement missing security controls
             3. Schedule next audit
             EOF

         - name: Upload Security Report
           uses: actions/upload-artifact@v4
           with:
             name: security-audit-${{ github.run_number }}
             path: security-audit-*.md
             retention-days: 365  # Keep for 1 year

         - name: Azure Logout
           if: always()
           run: az logout
   ```

5. **Test Scheduled Workflows**
   
   Since scheduled workflows only run on the default branch:
   
   ```powershell
   git add .
   git commit -m "Add scheduled maintenance workflows"
   git push origin main
   ```

   **To test without waiting:**
   - Go to Actions tab
   - Select a scheduled workflow
   - Click "Run workflow"
   - Click green "Run workflow" button

### Exercise 2: Implement Manual Workflows with Inputs

**Objective:** Create workflows that can be triggered manually with custom inputs.

#### Steps:

1. **Create On-Demand Deployment Workflow**

   **Create `.github/workflows/deploy-on-demand.yml`:**
   ```yaml
   name: On-Demand Deployment

   on:
     workflow_dispatch:
       inputs:
         environment:
           description: 'Deployment environment'
           required: true
           default: 'staging'
           type: choice
           options:
             - development
             - staging
             - production
         deploy-version:
           description: 'Version to deploy (git tag or branch)'
           required: true
           default: 'main'
           type: string
         run-smoke-tests:
           description: 'Run smoke tests after deployment'
           required: false
           default: true
           type: boolean
         notification-email:
           description: 'Email for deployment notification'
           required: false
           type: string

   jobs:
     validate-inputs:
       name: Validate Deployment Inputs
       runs-on: ubuntu-latest
       outputs:
         is-production: ${{ steps.check.outputs.is-production }}
       
       steps:
         - name: Display Inputs
           run: |
             echo "🚀 Deployment Configuration"
             echo "Environment: ${{ github.event.inputs.environment }}"
             echo "Version: ${{ github.event.inputs.deploy-version }}"
             echo "Smoke Tests: ${{ github.event.inputs.run-smoke-tests }}"
             echo "Notification: ${{ github.event.inputs.notification-email }}"

         - name: Check if Production
           id: check
           run: |
             if [ "${{ github.event.inputs.environment }}" == "production" ]; then
               echo "is-production=true" >> $GITHUB_OUTPUT
               echo "::warning::Production deployment requested!"
             else
               echo "is-production=false" >> $GITHUB_OUTPUT
             fi

     deploy:
       name: Deploy to ${{ github.event.inputs.environment }}
       needs: validate-inputs
       runs-on: ubuntu-latest
       environment:
         name: ${{ github.event.inputs.environment }}
       
       steps:
         - name: Checkout Code
           uses: actions/checkout@v4
           with:
             ref: ${{ github.event.inputs.deploy-version }}

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

         - name: Set Environment Variables
           run: |
             WEBAPP_NAME="${{ secrets.AZURE_WEBAPP_NAME }}"
             ENV="${{ github.event.inputs.environment }}"
             
             # Append environment to app name for non-production
             if [ "$ENV" != "production" ]; then
               WEBAPP_NAME="${WEBAPP_NAME}-${ENV}"
             fi
             
             echo "WEBAPP_NAME=$WEBAPP_NAME" >> $GITHUB_ENV

         - name: Deploy to Azure
           uses: azure/webapps-deploy@v2
           with:
             app-name: ${{ env.WEBAPP_NAME }}
             package: .

         - name: Azure Logout
           run: az logout

     smoke-tests:
       name: Run Smoke Tests
       needs: deploy
       runs-on: ubuntu-latest
       if: github.event.inputs.run-smoke-tests == 'true'
       
       steps:
         - name: Wait for Deployment
           run: sleep 30

         - name: Run Health Check
           run: |
             ENV="${{ github.event.inputs.environment }}"
             WEBAPP_NAME="${{ secrets.AZURE_WEBAPP_NAME }}"
             
             if [ "$ENV" != "production" ]; then
               WEBAPP_NAME="${WEBAPP_NAME}-${ENV}"
             fi
             
             URL="https://${WEBAPP_NAME}.azurewebsites.net/health"
             
             echo "Testing: $URL"
             
             STATUS=$(curl -s -o /dev/null -w "%{http_code}" $URL)
             
             if [ "$STATUS" -eq 200 ]; then
               echo "✅ Smoke test passed!"
             else
               echo "❌ Smoke test failed! Status: $STATUS"
               exit 1
             fi

         - name: Test API Endpoints
           run: |
             ENV="${{ github.event.inputs.environment }}"
             WEBAPP_NAME="${{ secrets.AZURE_WEBAPP_NAME }}"
             
             if [ "$ENV" != "production" ]; then
               WEBAPP_NAME="${WEBAPP_NAME}-${ENV}"
             fi
             
             BASE_URL="https://${WEBAPP_NAME}.azurewebsites.net"
             
             # Test root endpoint
             curl -f "$BASE_URL/" || exit 1
             
             # Test info endpoint
             curl -f "$BASE_URL/api/info" || exit 1
             
             echo "✅ All API tests passed!"

     notify:
       name: Send Notification
       needs: [deploy, smoke-tests]
       runs-on: ubuntu-latest
       if: always() && github.event.inputs.notification-email != ''
       
       steps:
         - name: Determine Status
           id: status
           run: |
             if [ "${{ needs.deploy.result }}" == "success" ] && \
                [ "${{ needs.smoke-tests.result }}" == "success" || "${{ needs.smoke-tests.result }}" == "skipped" ]; then
               echo "status=✅ SUCCESS" >> $GITHUB_OUTPUT
             else
               echo "status=❌ FAILED" >> $GITHUB_OUTPUT
             fi

         - name: Display Notification
           run: |
             echo "📧 Notification"
             echo "To: ${{ github.event.inputs.notification-email }}"
             echo "Subject: Deployment ${{ steps.status.outputs.status }}"
             echo "Environment: ${{ github.event.inputs.environment }}"
             echo "Version: ${{ github.event.inputs.deploy-version }}"
             echo "Status: ${{ steps.status.outputs.status }}"
   ```

2. **Create Resource Management Workflow**

   **Create `.github/workflows/azure-resource-manager.yml`:**
   ```yaml
   name: Azure Resource Manager

   on:
     workflow_dispatch:
       inputs:
         action:
           description: 'Action to perform'
           required: true
           type: choice
           options:
             - list-resources
             - start-webapp
             - stop-webapp
             - restart-webapp
             - scale-webapp
             - view-logs
         scale-tier:
           description: 'Scale tier (if scaling)'
           required: false
           type: choice
           options:
             - B1
             - B2
             - B3
             - S1
             - S2
             - P1V2
         log-lines:
           description: 'Number of log lines to retrieve'
           required: false
           default: '100'
           type: number

   jobs:
     execute-action:
       name: Execute ${{ github.event.inputs.action }}
       runs-on: ubuntu-latest
       
       steps:
         - name: Azure Login
           uses: azure/login@v1
           with:
             creds: ${{ secrets.AZURE_CREDENTIALS }}

         - name: List Resources
           if: github.event.inputs.action == 'list-resources'
           run: |
             echo "📋 Listing all resources..."
             az resource list \
               --resource-group "${{ secrets.AZURE_RESOURCE_GROUP }}" \
               --output table

         - name: Start Web App
           if: github.event.inputs.action == 'start-webapp'
           run: |
             echo "▶️ Starting Web App..."
             az webapp start \
               --name "${{ secrets.AZURE_WEBAPP_NAME }}" \
               --resource-group "${{ secrets.AZURE_RESOURCE_GROUP }}"
             echo "✅ Web App started successfully!"

         - name: Stop Web App
           if: github.event.inputs.action == 'stop-webapp'
           run: |
             echo "⏸️ Stopping Web App..."
             az webapp stop \
               --name "${{ secrets.AZURE_WEBAPP_NAME }}" \
               --resource-group "${{ secrets.AZURE_RESOURCE_GROUP }}"
             echo "✅ Web App stopped successfully!"

         - name: Restart Web App
           if: github.event.inputs.action == 'restart-webapp'
           run: |
             echo "🔄 Restarting Web App..."
             az webapp restart \
               --name "${{ secrets.AZURE_WEBAPP_NAME }}" \
               --resource-group "${{ secrets.AZURE_RESOURCE_GROUP }}"
             echo "✅ Web App restarted successfully!"

         - name: Scale Web App
           if: github.event.inputs.action == 'scale-webapp'
           run: |
             echo "⚡ Scaling Web App to ${{ github.event.inputs.scale-tier }}..."
             
             PLAN_NAME=$(az webapp show \
               --name "${{ secrets.AZURE_WEBAPP_NAME }}" \
               --resource-group "${{ secrets.AZURE_RESOURCE_GROUP }}" \
               --query appServicePlanId \
               --output tsv | xargs basename)
             
             az appservice plan update \
               --name "$PLAN_NAME" \
               --resource-group "${{ secrets.AZURE_RESOURCE_GROUP }}" \
               --sku "${{ github.event.inputs.scale-tier }}"
             
             echo "✅ Scaled to ${{ github.event.inputs.scale-tier }}!"

         - name: View Logs
           if: github.event.inputs.action == 'view-logs'
           run: |
             echo "📜 Retrieving last ${{ github.event.inputs.log-lines }} log lines..."
             
             az webapp log tail \
               --name "${{ secrets.AZURE_WEBAPP_NAME }}" \
               --resource-group "${{ secrets.AZURE_RESOURCE_GROUP }}" \
               --lines ${{ github.event.inputs.log-lines }}

         - name: Azure Logout
           if: always()
           run: az logout
   ```

3. **Test Manual Workflows**
   - Commit and push the workflows
   - Go to Actions tab
   - Select "On-Demand Deployment"
   - Click "Run workflow"
   - Fill in inputs
   - Execute and monitor

### Exercise 3: Implement repository_dispatch Events

**Objective:** Enable external systems to trigger workflows via API.

#### Steps:

1. **Create API-Triggered Workflow**

   **Create `.github/workflows/external-trigger.yml`:**
   ```yaml
   name: External Trigger Handler

   on:
     repository_dispatch:
       types:
         - deploy-request
         - health-check
         - backup-request

   jobs:
     handle-event:
       name: Handle ${{ github.event.action }}
       runs-on: ubuntu-latest
       
       steps:
         - name: Display Event Info
           run: |
             echo "Event Type: ${{ github.event.action }}"
             echo "Sender: ${{ github.event.client_payload.sender }}"
             echo "Timestamp: ${{ github.event.client_payload.timestamp }}"
             echo "Full Payload:"
             echo '${{ toJSON(github.event.client_payload) }}'

         - name: Handle Deploy Request
           if: github.event.action == 'deploy-request'
           run: |
             echo "Processing deployment request..."
             ENV="${{ github.event.client_payload.environment }}"
             VERSION="${{ github.event.client_payload.version }}"
             echo "Deploying $VERSION to $ENV"

         - name: Handle Health Check
           if: github.event.action == 'health-check'
           run: |
             echo "Performing health check..."
             URL="${{ github.event.client_payload.url }}"
             curl -f "$URL" || echo "Health check failed!"

         - name: Handle Backup Request
           if: github.event.action == 'backup-request'
           run: |
             echo "Initiating backup..."
             RESOURCE="${{ github.event.client_payload.resource }}"
             echo "Backing up: $RESOURCE"
   ```

2. **Create PowerShell Script to Trigger Workflow**

   **Create `scripts/trigger-workflow.ps1`:**
   ```powershell
   param(
       [Parameter(Mandatory=$true)]
       [string]$GitHubToken,
       
       [Parameter(Mandatory=$true)]
       [string]$Owner,
       
       [Parameter(Mandatory=$true)]
       [string]$Repo,
       
       [Parameter(Mandatory=$true)]
       [ValidateSet("deploy-request", "health-check", "backup-request")]
       [string]$EventType,
       
       [Parameter(Mandatory=$false)]
       [hashtable]$Payload = @{}
   )

   # Build client payload
   $clientPayload = @{
       sender = $env:USERNAME
       timestamp = (Get-Date).ToUniversalTime().ToString("yyyy-MM-ddTHH:mm:ssZ")
   }

   # Merge with provided payload
   foreach ($key in $Payload.Keys) {
       $clientPayload[$key] = $Payload[$key]
   }

   # Build request body
   $body = @{
       event_type = $EventType
       client_payload = $clientPayload
   } | ConvertTo-Json -Depth 10

   # API endpoint
   $uri = "https://api.github.com/repos/$Owner/$Repo/dispatches"

   # Send request
   $headers = @{
       "Accept" = "application/vnd.github.v3+json"
       "Authorization" = "Bearer $GitHubToken"
   }

   try {
       Invoke-RestMethod -Uri $uri -Method Post -Headers $headers -Body $body
       Write-Host "✅ Workflow triggered successfully!" -ForegroundColor Green
       Write-Host "Event Type: $EventType" -ForegroundColor Cyan
       Write-Host "Check: https://github.com/$Owner/$Repo/actions" -ForegroundColor Yellow
   }
   catch {
       Write-Host "❌ Failed to trigger workflow!" -ForegroundColor Red
       Write-Host $_.Exception.Message -ForegroundColor Red
   }
   ```

3. **Usage Examples**

   ```powershell
   # Trigger deployment
   .\scripts\trigger-workflow.ps1 `
       -GitHubToken "ghp_yourtoken" `
       -Owner "yourusername" `
       -Repo "azure-nodejs-webapp" `
       -EventType "deploy-request" `
       -Payload @{
           environment = "staging"
           version = "v1.2.3"
       }

   # Trigger health check
   .\scripts\trigger-workflow.ps1 `
       -GitHubToken "ghp_yourtoken" `
       -Owner "yourusername" `
       -Repo "azure-nodejs-webapp" `
       -EventType "health-check" `
       -Payload @{
           url = "https://your-app.azurewebsites.net/health"
       }
   ```

4. **Create Personal Access Token for API**
   - GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
   - Generate new token (classic)
   - Select scopes: `repo` (full control)
   - Generate token
   - Save securely (won't see again!)

### Exercise 4: Implement Path and Branch Filters

**Objective:** Optimize workflow execution with path and branch filtering.

#### Steps:

1. **Create Path-Filtered Workflow**

   **Create `.github/workflows/smart-ci.yml`:**
   ```yaml
   name: Smart CI with Path Filters

   on:
     push:
       branches:
         - main
         - develop
         - 'feature/**'
         - 'release/**'
       paths:
         - 'src/**'
         - 'tests/**'
         - 'package.json'
         - 'package-lock.json'
       paths-ignore:
         - '**.md'
         - 'docs/**'
         - '.github/**'
         - '.vscode/**'

   jobs:
     detect-changes:
       name: Detect Changed Components
       runs-on: ubuntu-latest
       outputs:
         backend-changed: ${{ steps.filter.outputs.backend }}
         frontend-changed: ${{ steps.filter.outputs.frontend }}
         
       steps:
         - uses: actions/checkout@v4
         
         - uses: dorny/paths-filter@v2
           id: filter
           with:
             filters: |
               backend:
                 - 'src/backend/**'
                 - 'src/api/**'
               frontend:
                 - 'src/frontend/**'
                 - 'src/components/**'
               config:
                 - 'package.json'
                 - 'tsconfig.json'

     build-backend:
       name: Build Backend
       needs: detect-changes
       if: needs.detect-changes.outputs.backend-changed == 'true'
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - name: Build Backend
           run: echo "Building backend..."

     build-frontend:
       name: Build Frontend
       needs: detect-changes
       if: needs.detect-changes.outputs.frontend-changed == 'true'
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - name: Build Frontend
           run: echo "Building frontend..."
   ```

2. **Benefits of Path Filtering:**
   - ✅ Saves GitHub Actions minutes
   - ✅ Faster CI/CD feedback
   - ✅ Reduces noise in Actions tab
   - ✅ Focus on relevant changes

## 📝 Knowledge Check

1. **What's the shortest interval for scheduled workflows?**
   <details>
   <summary>Answer</summary>
   5 minutes (*/5 * * * *)
   </details>

2. **How do you trigger a workflow manually?**
   <details>
   <summary>Answer</summary>
   Add workflow_dispatch event and use "Run workflow" button in Actions tab
   </details>

3. **What's the purpose of repository_dispatch?**
   <details>
   <summary>Answer</summary>
   Allows external systems to trigger workflows via GitHub API
   </details>

4. **How can you prevent workflows from running on documentation changes?**
   <details>
   <summary>Answer</summary>
   Use paths-ignore with patterns like '**.md' and 'docs/**'
   </details>

## 🎓 Lab Summary

**You've mastered advanced workflow triggers!**

✅ Scheduled workflows with cron  
✅ Manual workflows with inputs  
✅ External API triggers  
✅ Path and branch filtering  
✅ Workflow optimization  

## 🔜 Next Steps

Proceed to **[Lab 5: Creating JavaScript Actions with Azure SDK](../Lab5-JavaScript-Actions/README.md)**

---

**Next:** Building JavaScript actions for faster execution!
