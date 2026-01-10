# Lab 7: GitHub Script & API Automation

**Duration:** 45 minutes  
**Difficulty:** Intermediate  
**Prerequisites:** Lab 1 (Fundamentals) and Lab 5 (JavaScript Actions)

## 🎯 Learning Objectives

By the end of this lab, you will be able to:
- Understand what GitHub Script is and when to use it
- Use GitHub Script to interact with GitHub APIs
- Automate GitHub workflows using Octokit
- Create, update, and manage issues and pull requests programmatically
- Implement automated issue triaging and labeling
- Use GitHub Script with Azure integration

---

## 📖 What is GitHub Script?

**GitHub Script** is a GitHub Action that provides an authenticated Octokit client and allows you to write JavaScript directly in your workflow files to interact with the GitHub API.

### Why Use GitHub Script?

Instead of creating a separate JavaScript action for simple API interactions, GitHub Script lets you write inline scripts in your workflow YAML files.

**Benefits:**
- ✅ No need to create separate action repositories
- ✅ Pre-authenticated Octokit client
- ✅ Access to workflow context (`github`, `context`)
- ✅ Simpler for one-off automations
- ✅ Fast prototyping and testing

**Use Cases:**
- Automatically label issues based on content
- Comment on pull requests
- Create issues from workflow failures
- Update project boards
- Manage releases
- Close stale issues
- Triage bugs automatically

### GitHub Script vs JavaScript Actions

| Feature | GitHub Script | JavaScript Action |
|---------|--------------|-------------------|
| **Setup** | Write inline in workflow | Separate repository |
| **Reusability** | Limited to workflow | Highly reusable |
| **Complexity** | Simple automation | Complex logic |
| **Authentication** | Automatic | Manual setup |
| **Best For** | Quick scripts | Production actions |

**Best Practice:** Use GitHub Script for simple, workflow-specific automations. Create custom JavaScript actions for complex, reusable functionality.

---

## 🏗️ Lab Architecture

This lab demonstrates:
```
GitHub Event → Workflow → GitHub Script → GitHub API → Action
     ↓                         ↓              ↓
  Issues             Octokit Client    Create/Update
  PRs                Context Access     Comment/Label
  Comments           Script Logic       Close/Assign
```

---

## 📋 Prerequisites

Before starting this lab:

1. ✅ Completed Lab 1 (GitHub Actions Fundamentals)
2. ✅ GitHub account with repository access
3. ✅ Basic JavaScript knowledge
4. ✅ Understanding of GitHub issues and pull requests
5. ✅ (Optional) Azure subscription for Azure integration exercises

---

## 🚀 Exercise 1: Your First GitHub Script

### What You'll Build
A workflow that automatically labels new issues based on keywords in the title or body.

### Step 1: Create Repository and Workflow

1. **Create a new repository** named `github-script-lab`:
   ```powershell
   # Navigate to your GitHub account
   # Click "New repository"
   # Name: github-script-lab
   # Description: Learning GitHub Script automation
   # Visibility: Public
   # Initialize with README
   # Click "Create repository"
   ```

2. **Clone the repository**:
   ```powershell
   cd C:\Users\alibengtsson\mycertifications\githubactions
   git clone https://github.com/YOUR_USERNAME/github-script-lab.git
   cd github-script-lab
   ```

3. **Create workflow directory**:
   ```powershell
   mkdir -p .github/workflows
   ```

### Step 2: Create Auto-Label Workflow

Create `.github/workflows/auto-label.yml`:

```yaml
name: Auto Label Issues

on:
  issues:
    types: [opened, edited]

jobs:
  label-issue:
    runs-on: ubuntu-latest
    permissions:
      issues: write
      contents: read
    
    steps:
      - name: Label Issue Based on Content
        uses: actions/github-script@v7
        with:
          script: |
            const issue = context.payload.issue;
            const issueBody = issue.body || '';
            const issueTitle = issue.title || '';
            const content = (issueTitle + ' ' + issueBody).toLowerCase();
            
            // Define label mapping
            const labelMap = {
              'bug': ['bug', 'error', 'crash', 'broken', 'fail'],
              'enhancement': ['feature', 'enhancement', 'improve', 'add'],
              'documentation': ['docs', 'documentation', 'readme'],
              'question': ['question', 'how to', 'help', '?'],
              'azure': ['azure', 'az', 'microsoft']
            };
            
            // Determine which labels to add
            const labelsToAdd = [];
            for (const [label, keywords] of Object.entries(labelMap)) {
              if (keywords.some(keyword => content.includes(keyword))) {
                labelsToAdd.push(label);
              }
            }
            
            // Add labels if any were found
            if (labelsToAdd.length > 0) {
              await github.rest.issues.addLabels({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: issue.number,
                labels: labelsToAdd
              });
              
              console.log(`Added labels: ${labelsToAdd.join(', ')}`);
            } else {
              console.log('No matching labels found');
            }
```

### Step 3: Create Required Labels

Before testing, create the labels in your repository:

```powershell
# You can do this via GitHub UI or using GitHub CLI
gh label create bug --description "Something isn't working" --color d73a4a
gh label create enhancement --description "New feature or request" --color a2eeef
gh label create documentation --description "Improvements to documentation" --color 0075ca
gh label create question --description "Further information is requested" --color d876e3
gh label create azure --description "Related to Azure" --color 0078d4
```

Or via GitHub UI:
1. Go to your repository
2. Click **Issues** → **Labels**
3. Click **New label** for each label above

### Step 4: Test the Workflow

1. **Commit and push the workflow**:
   ```powershell
   git add .github/workflows/auto-label.yml
   git commit -m "Add auto-label workflow"
   git push origin main
   ```

2. **Create test issues**:
   - Issue 1: Title "Bug in login feature" → Should get `bug` label
   - Issue 2: Title "Add Azure deployment docs" → Should get `documentation` and `azure` labels
   - Issue 3: Title "How to configure Azure?" → Should get `question` and `azure` labels

3. **Verify in Actions tab**:
   - Check that workflow runs successfully
   - Verify labels are applied automatically

### Understanding the Code

**Key Components:**

1. **Event Trigger:**
   ```yaml
   on:
     issues:
       types: [opened, edited]
   ```
   - Runs when issues are opened or edited

2. **Permissions:**
   ```yaml
   permissions:
     issues: write
     contents: read
   ```
   - Grants necessary access to modify issues

3. **Context Object:**
   ```javascript
   const issue = context.payload.issue;
   ```
   - `context` provides event payload data

4. **Octokit API:**
   ```javascript
   await github.rest.issues.addLabels({...});
   ```
   - `github` is pre-authenticated Octokit client

**Best Practice:** Always check if content exists before processing (`issue.body || ''`) to avoid null errors.

---

## 🚀 Exercise 2: Auto-Comment on Pull Requests

### What You'll Build
A workflow that automatically comments on new pull requests with a checklist and Azure deployment information.

### Step 1: Create PR Comment Workflow

Create `.github/workflows/pr-welcome.yml`:

```yaml
name: Welcome New PR

on:
  pull_request:
    types: [opened]

jobs:
  welcome:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
    
    steps:
      - name: Comment on PR
        uses: actions/github-script@v7
        with:
          script: |
            const pr = context.payload.pull_request;
            const author = pr.user.login;
            
            // Check if author is a first-time contributor
            const { data: contributors } = await github.rest.repos.listContributors({
              owner: context.repo.owner,
              repo: context.repo.repo
            });
            
            const isFirstTime = !contributors.some(c => c.login === author);
            
            let greeting = isFirstTime 
              ? `🎉 Welcome @${author}! Thank you for your first contribution!`
              : `👋 Welcome back @${author}! Thanks for another contribution!`;
            
            // Create checklist comment
            const comment = `${greeting}

## Pull Request Checklist

Please ensure your PR meets the following requirements:

- [ ] Code follows project style guidelines
- [ ] Tests have been added/updated
- [ ] Documentation has been updated
- [ ] All tests pass locally
- [ ] Commit messages are clear and descriptive

## Azure Deployment Preview

This PR will trigger a preview deployment to Azure when approved.

- **Preview URL:** Will be available after approval
- **Resource Group:** \`rg-${context.repo.repo}-preview-pr${pr.number}\`
- **Cleanup:** Preview environment will be deleted when PR is closed

## Review Process

1. ✅ Automated checks will run
2. 👀 A maintainer will review your code
3. 🚀 Once approved, changes will be deployed to staging
4. ✅ After testing, changes will be merged to main

---

Need help? Tag \`@maintainers\` or check our [Contributing Guide](../CONTRIBUTING.md).`;

            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: pr.number,
              body: comment
            });
            
            console.log(`Posted welcome comment on PR #${pr.number}`);
```

### Step 2: Add PR Size Labeling

Extend the workflow to add size labels based on lines changed:

```yaml
      - name: Label PR by Size
        uses: actions/github-script@v7
        with:
          script: |
            const pr = context.payload.pull_request;
            const additions = pr.additions;
            const deletions = pr.deletions;
            const totalChanges = additions + deletions;
            
            let sizeLabel;
            if (totalChanges < 10) {
              sizeLabel = 'size/XS';
            } else if (totalChanges < 50) {
              sizeLabel = 'size/S';
            } else if (totalChanges < 250) {
              sizeLabel = 'size/M';
            } else if (totalChanges < 1000) {
              sizeLabel = 'size/L';
            } else {
              sizeLabel = 'size/XL';
            }
            
            await github.rest.issues.addLabels({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: pr.number,
              labels: [sizeLabel]
            });
            
            console.log(`Added label: ${sizeLabel} (${totalChanges} lines changed)`);
```

### Step 3: Create Size Labels

```powershell
gh label create "size/XS" --description "Extra small PR" --color 00ff00
gh label create "size/S" --description "Small PR" --color 7fff00
gh label create "size/M" --description "Medium PR" --color ffff00
gh label create "size/L" --description "Large PR" --color ff7f00
gh label create "size/XL" --description "Extra large PR" --color ff0000
```

### Step 4: Test the Workflow

1. **Commit and push**:
   ```powershell
   git add .github/workflows/pr-welcome.yml
   git commit -m "Add PR welcome workflow"
   git push origin main
   ```

2. **Create a test PR**:
   ```powershell
   git checkout -b test-pr
   echo "# Test" > test.md
   git add test.md
   git commit -m "Add test file"
   git push origin test-pr
   
   # Create PR via GitHub UI or CLI
   gh pr create --title "Test PR" --body "Testing auto-comment"
   ```

3. **Verify**:
   - Check that welcome comment appears
   - Verify size label is applied
   - Test with PRs of different sizes

---

## 🚀 Exercise 3: Automated Issue Management

### What You'll Build
A comprehensive issue management system that:
- Assigns issues automatically
- Creates issues from workflow failures
- Closes stale issues

### Step 1: Auto-Assign Issues

Create `.github/workflows/auto-assign.yml`:

```yaml
name: Auto Assign Issues

on:
  issues:
    types: [opened]

jobs:
  assign:
    runs-on: ubuntu-latest
    permissions:
      issues: write
    
    steps:
      - name: Auto Assign by Label
        uses: actions/github-script@v7
        with:
          script: |
            const issue = context.payload.issue;
            const labels = issue.labels.map(l => l.name);
            
            // Define assignment rules
            const assignmentMap = {
              'bug': ['maintainer1', 'maintainer2'],
              'azure': ['azure-expert'],
              'documentation': ['doc-team'],
              'enhancement': ['product-team']
            };
            
            // Find assignees based on labels
            let assignees = [];
            for (const label of labels) {
              if (assignmentMap[label]) {
                assignees.push(...assignmentMap[label]);
              }
            }
            
            // Remove duplicates
            assignees = [...new Set(assignees)];
            
            // Assign if we found someone
            if (assignees.length > 0) {
              try {
                await github.rest.issues.addAssignees({
                  owner: context.repo.owner,
                  repo: context.repo.repo,
                  issue_number: issue.number,
                  assignees: assignees
                });
                
                console.log(`Assigned to: ${assignees.join(', ')}`);
                
                // Add comment
                await github.rest.issues.createComment({
                  owner: context.repo.owner,
                  repo: context.repo.repo,
                  issue_number: issue.number,
                  body: `🤖 Auto-assigned to ${assignees.map(a => '@' + a).join(', ')} based on labels.`
                });
              } catch (error) {
                console.log(`Could not assign: ${error.message}`);
                console.log('Note: Users must have repository access to be assigned');
              }
            }
```

### Step 2: Create Issue from Workflow Failure

Create `.github/workflows/failure-issue.yml`:

```yaml
name: Report Workflow Failures

on:
  workflow_run:
    workflows: ["CI", "CD"]
    types: [completed]

jobs:
  create-issue:
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'failure' }}
    permissions:
      issues: write
    
    steps:
      - name: Create Issue for Failure
        uses: actions/github-script@v7
        with:
          script: |
            const workflow = context.payload.workflow_run;
            const workflowName = workflow.name;
            const runNumber = workflow.run_number;
            const runUrl = workflow.html_url;
            const branch = workflow.head_branch;
            const commit = workflow.head_sha.substring(0, 7);
            
            // Check if issue already exists
            const { data: existingIssues } = await github.rest.issues.listForRepo({
              owner: context.repo.owner,
              repo: context.repo.repo,
              state: 'open',
              labels: 'workflow-failure',
              per_page: 100
            });
            
            const issueTitle = `Workflow Failure: ${workflowName} #${runNumber}`;
            const exists = existingIssues.some(issue => issue.title === issueTitle);
            
            if (!exists) {
              const issueBody = `## 🚨 Workflow Failure Detected

**Workflow:** ${workflowName}
**Run Number:** #${runNumber}
**Branch:** \`${branch}\`
**Commit:** \`${commit}\`
**Run URL:** ${runUrl}

### Details

The \`${workflowName}\` workflow failed on run #${runNumber}.

### Action Required

1. Review the workflow logs: [View Run](${runUrl})
2. Investigate the failure
3. Fix the issue
4. Re-run the workflow or push a fix

### Azure Context

If this is an Azure deployment failure, check:
- Azure credentials are valid
- Resource group exists
- Service principal has correct permissions
- Azure subscription is active

---

*This issue was automatically created by GitHub Script.*`;

              const { data: newIssue } = await github.rest.issues.create({
                owner: context.repo.owner,
                repo: context.repo.repo,
                title: issueTitle,
                body: issueBody,
                labels: ['workflow-failure', 'bug', 'automated']
              });
              
              console.log(`Created issue #${newIssue.number} for workflow failure`);
            } else {
              console.log('Issue already exists for this failure');
            }
```

### Step 3: Close Stale Issues

Create `.github/workflows/close-stale.yml`:

```yaml
name: Close Stale Issues

on:
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight
  workflow_dispatch:  # Allow manual trigger

jobs:
  stale:
    runs-on: ubuntu-latest
    permissions:
      issues: write
    
    steps:
      - name: Close Stale Issues
        uses: actions/github-script@v7
        with:
          script: |
            const daysBeforeStale = 30;
            const daysBeforeClose = 7;
            
            // Calculate date thresholds
            const now = new Date();
            const staleDate = new Date(now - daysBeforeStale * 24 * 60 * 60 * 1000);
            const closeDate = new Date(now - (daysBeforeStale + daysBeforeClose) * 24 * 60 * 60 * 1000);
            
            // Get all open issues
            const { data: issues } = await github.rest.issues.listForRepo({
              owner: context.repo.owner,
              repo: context.repo.repo,
              state: 'open',
              per_page: 100
            });
            
            for (const issue of issues) {
              // Skip pull requests
              if (issue.pull_request) continue;
              
              const updatedAt = new Date(issue.updated_at);
              const labels = issue.labels.map(l => l.name);
              
              // Skip issues with 'keep' label
              if (labels.includes('keep')) continue;
              
              // Close if already stale and past close date
              if (labels.includes('stale') && updatedAt < closeDate) {
                await github.rest.issues.update({
                  owner: context.repo.owner,
                  repo: context.repo.repo,
                  issue_number: issue.number,
                  state: 'closed',
                  state_reason: 'not_planned'
                });
                
                await github.rest.issues.createComment({
                  owner: context.repo.owner,
                  repo: context.repo.repo,
                  issue_number: issue.number,
                  body: '🤖 This issue was automatically closed due to inactivity.'
                });
                
                console.log(`Closed stale issue #${issue.number}`);
              }
              // Mark as stale if past stale date
              else if (updatedAt < staleDate && !labels.includes('stale')) {
                await github.rest.issues.addLabels({
                  owner: context.repo.owner,
                  repo: context.repo.repo,
                  issue_number: issue.number,
                  labels: ['stale']
                });
                
                await github.rest.issues.createComment({
                  owner: context.repo.owner,
                  repo: context.repo.repo,
                  issue_number: issue.number,
                  body: `⚠️ This issue has been marked as stale due to ${daysBeforeStale} days of inactivity. It will be closed in ${daysBeforeClose} days if no further activity occurs.`
                });
                
                console.log(`Marked issue #${issue.number} as stale`);
              }
            }
```

Create the labels:
```powershell
gh label create stale --description "No activity for extended period" --color cccccc
gh label create keep --description "Prevent stale marking" --color 00ff00
gh label create workflow-failure --description "Workflow execution failed" --color ff0000
gh label create automated --description "Created by automation" --color 1d76db
```

---

## 🚀 Exercise 4: GitHub Script with Azure Integration

### What You'll Build
A workflow that uses GitHub Script to manage Azure resources and post deployment information to issues/PRs.

### Step 1: Create Azure Deployment Reporter

Create `.github/workflows/azure-deployment-report.yml`:

```yaml
name: Azure Deployment Report

on:
  workflow_run:
    workflows: ["CD"]
    types: [completed]

jobs:
  report:
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    permissions:
      issues: write
      pull-requests: write
    
    steps:
      - name: Azure Login
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      
      - name: Get Azure Resource Information
        id: azure
        run: |
          RESOURCE_GROUP="${{ secrets.AZURE_RESOURCE_GROUP }}"
          WEBAPP_NAME="${{ secrets.AZURE_WEBAPP_NAME }}"
          
          # Get web app details
          WEBAPP_URL=$(az webapp show --name $WEBAPP_NAME --resource-group $RESOURCE_GROUP --query "defaultHostName" -o tsv)
          WEBAPP_STATE=$(az webapp show --name $WEBAPP_NAME --resource-group $RESOURCE_GROUP --query "state" -o tsv)
          
          # Get app insights key if available
          APPINSIGHTS=$(az monitor app-insights component show --app $WEBAPP_NAME --resource-group $RESOURCE_GROUP --query "instrumentationKey" -o tsv 2>/dev/null || echo "Not configured")
          
          echo "url=https://$WEBAPP_URL" >> $GITHUB_OUTPUT
          echo "state=$WEBAPP_STATE" >> $GITHUB_OUTPUT
          echo "insights=$APPINSIGHTS" >> $GITHUB_OUTPUT
      
      - name: Post Deployment Info to PR
        uses: actions/github-script@v7
        with:
          script: |
            const workflow = context.payload.workflow_run;
            const prNumber = workflow.pull_requests[0]?.number;
            
            if (!prNumber) {
              console.log('No PR associated with this workflow run');
              return;
            }
            
            const url = '${{ steps.azure.outputs.url }}';
            const state = '${{ steps.azure.outputs.state }}';
            const insights = '${{ steps.azure.outputs.insights }}';
            
            const comment = `## ✅ Azure Deployment Successful

### Deployment Details

| Property | Value |
|----------|-------|
| **App URL** | ${url} |
| **Status** | ${state} |
| **Application Insights** | ${insights !== 'Not configured' ? '✅ Configured' : '❌ Not configured'} |
| **Deployed At** | ${new Date().toISOString()} |
| **Workflow Run** | [View Logs](${workflow.html_url}) |

### Quick Links

- 🌐 [Open Application](${url})
- 📊 [Azure Portal](https://portal.azure.com/#@/resource/subscriptions/${{ secrets.AZURE_SUBSCRIPTION_ID }}/resourceGroups/${{ secrets.AZURE_RESOURCE_GROUP }}/providers/Microsoft.Web/sites/${{ secrets.AZURE_WEBAPP_NAME }})
${insights !== 'Not configured' ? '- 📈 [Application Insights](https://portal.azure.com/#blade/AppInsightsExtension/ResourceMenuBlade/overview/ResourceId/)' : ''}

### Health Check

Running automated health check...`;

            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: prNumber,
              body: comment
            });
            
            // Run health check
            try {
              const response = await fetch(url);
              const status = response.ok ? '✅ Healthy' : '❌ Unhealthy';
              
              await github.rest.issues.createComment({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: prNumber,
                body: `Health check result: ${status} (HTTP ${response.status})`
              });
            } catch (error) {
              await github.rest.issues.createComment({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: prNumber,
                body: `⚠️ Health check failed: ${error.message}`
              });
            }
```

### Step 2: Create Azure Cost Reporter

Create `.github/workflows/azure-cost-report.yml`:

```yaml
name: Azure Cost Report

on:
  schedule:
    - cron: '0 9 * * 1'  # Weekly on Monday at 9 AM
  workflow_dispatch:

jobs:
  cost-report:
    runs-on: ubuntu-latest
    permissions:
      issues: write
    
    steps:
      - name: Azure Login
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      
      - name: Get Cost Data
        id: cost
        run: |
          RESOURCE_GROUP="${{ secrets.AZURE_RESOURCE_GROUP }}"
          
          # Get resource list and estimated costs
          RESOURCES=$(az resource list --resource-group $RESOURCE_GROUP --query "[].{name:name, type:type}" -o json)
          
          echo "resources<<EOF" >> $GITHUB_OUTPUT
          echo "$RESOURCES" >> $GITHUB_OUTPUT
          echo "EOF" >> $GITHUB_OUTPUT
      
      - name: Create Cost Report Issue
        uses: actions/github-script@v7
        with:
          script: |
            const resources = JSON.parse(`${{ steps.cost.outputs.resources }}`);
            const resourceGroup = '${{ secrets.AZURE_RESOURCE_GROUP }}';
            
            // Build resource table
            let resourceTable = '| Resource | Type |\n|----------|------|\n';
            resources.forEach(r => {
              resourceTable += `| ${r.name} | ${r.type} |\n`;
            });
            
            const issueBody = `## 💰 Weekly Azure Cost Report

### Resource Group: \`${resourceGroup}\`

**Report Date:** ${new Date().toLocaleDateString()}
**Total Resources:** ${resources.length}

### Resources Deployed

${resourceTable}

### Action Items

- [ ] Review resource usage
- [ ] Check for unused resources
- [ ] Optimize configurations
- [ ] Review Azure Advisor recommendations

### Cost Optimization Tips

1. **App Service Plans:** Ensure correct tier for usage
2. **Storage:** Enable lifecycle management
3. **Application Insights:** Set appropriate retention
4. **Unused Resources:** Delete staging/test environments

### Links

- [Azure Cost Management](https://portal.azure.com/#blade/Microsoft_Azure_CostManagement/Menu/overview)
- [Azure Advisor](https://portal.azure.com/#blade/Microsoft_Azure_Expert/AdvisorMenuBlade/overview)

---

*This report was automatically generated by GitHub Script.*`;

            await github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: `Azure Cost Report - ${new Date().toLocaleDateString()}`,
              body: issueBody,
              labels: ['azure', 'cost-management', 'automated']
            });
            
            console.log('Created weekly cost report issue');
```

---

## 🧪 Advanced Patterns

### Pattern 1: Conditional Logic with Multiple API Calls

```javascript
const pr = context.payload.pull_request;

// Get PR files
const { data: files } = await github.rest.pulls.listFiles({
  owner: context.repo.owner,
  repo: context.repo.repo,
  pull_number: pr.number
});

// Check if docs were modified
const docsModified = files.some(f => f.filename.startsWith('docs/'));

// Get existing reviews
const { data: reviews } = await github.rest.pulls.listReviews({
  owner: context.repo.owner,
  repo: context.repo.repo,
  pull_number: pr.number
});

// Request doc review if docs changed and no doc reviewer assigned
if (docsModified && !reviews.some(r => r.user.login === 'doc-reviewer')) {
  await github.rest.pulls.requestReviewers({
    owner: context.repo.owner,
    repo: context.repo.repo,
    pull_number: pr.number,
    reviewers: ['doc-reviewer']
  });
}
```

### Pattern 2: Error Handling

```javascript
try {
  await github.rest.issues.createComment({
    owner: context.repo.owner,
    repo: context.repo.repo,
    issue_number: issueNumber,
    body: comment
  });
} catch (error) {
  core.warning(`Failed to create comment: ${error.message}`);
  
  // Try alternative action
  await github.rest.issues.addLabels({
    owner: context.repo.owner,
    repo: context.repo.repo,
    issue_number: issueNumber,
    labels: ['needs-attention']
  });
}
```

### Pattern 3: Pagination

```javascript
// Get all issues (handles pagination automatically)
const issues = await github.paginate(github.rest.issues.listForRepo, {
  owner: context.repo.owner,
  repo: context.repo.repo,
  state: 'open',
  per_page: 100
});

console.log(`Found ${issues.length} open issues`);
```

### Pattern 4: GraphQL Queries

```javascript
const query = `
  query($owner: String!, $repo: String!) {
    repository(owner: $owner, name: $repo) {
      issues(first: 10, states: OPEN) {
        nodes {
          number
          title
          labels(first: 5) {
            nodes {
              name
            }
          }
        }
      }
    }
  }
`;

const result = await github.graphql(query, {
  owner: context.repo.owner,
  repo: context.repo.repo
});

console.log(result.repository.issues.nodes);
```

---

## 📚 Best Practices

### 1. Use Permissions Wisely

```yaml
permissions:
  issues: write        # Only what you need
  pull-requests: write
  contents: read       # Default to read
```

### 2. Check for Existing Items

Always check before creating to avoid duplicates:
```javascript
const { data: existingIssues } = await github.rest.issues.listForRepo({
  owner: context.repo.owner,
  repo: context.repo.repo,
  state: 'open',
  labels: 'specific-label'
});

const exists = existingIssues.some(issue => issue.title === newTitle);
```

### 3. Provide Context in Comments

Include relevant information and links:
```javascript
const comment = `
Action taken by GitHub Script
- Triggered by: @${context.actor}
- Workflow: ${context.workflow}
- Run: ${context.runNumber}
`;
```

### 4. Handle Rate Limits

```javascript
try {
  await github.rest.issues.create({...});
} catch (error) {
  if (error.status === 403 && error.message.includes('rate limit')) {
    core.warning('Rate limit exceeded. Waiting before retry...');
    await new Promise(resolve => setTimeout(resolve, 60000));
    // Retry
  }
}
```

### 5. Use Core Functions

```javascript
const core = require('@actions/core');

// Log levels
core.info('Information message');
core.warning('Warning message');
core.error('Error message');
core.debug('Debug message');

// Set outputs
core.setOutput('issue-number', issueNumber);

// Mask secrets
core.setSecret(apiKey);
```

---

## 🔍 Troubleshooting

### Issue: Permission Denied

**Symptom:**
```
Resource not accessible by integration
```

**Solution:**
1. Add required permissions to workflow:
   ```yaml
   permissions:
     issues: write
     pull-requests: write
   ```

2. Check repository settings: Settings → Actions → General → Workflow permissions

### Issue: Script Syntax Error

**Symptom:**
```
SyntaxError: Unexpected token
```

**Solution:**
- Ensure YAML string escaping is correct
- Use `|` for multi-line scripts
- Test JavaScript locally first

### Issue: Context is Undefined

**Symptom:**
```
Cannot read property 'payload' of undefined
```

**Solution:**
```javascript
// Check before accessing
if (!context.payload.issue) {
  console.log('No issue in payload');
  return;
}
```

### Issue: API Call Fails

**Symptom:**
```
HttpError: Not Found
```

**Solution:**
- Verify owner/repo names
- Check item exists (issue number, PR number)
- Use try-catch for error handling

---

## ✅ Knowledge Check

### Question 1
What is the main advantage of GitHub Script over creating a custom JavaScript action?

<details>
<summary>Answer</summary>

GitHub Script allows you to write inline JavaScript directly in workflow files without creating separate action repositories. It provides a pre-authenticated Octokit client and is ideal for simple, workflow-specific automations.
</details>

### Question 2
What permissions are required to add labels to issues?

<details>
<summary>Answer</summary>

```yaml
permissions:
  issues: write
```

The `issues: write` permission allows adding/removing labels, creating comments, and modifying issues.
</details>

### Question 3
How do you access the issue number in a GitHub Script?

<details>
<summary>Answer</summary>

```javascript
const issueNumber = context.payload.issue.number;
```

Or for pull requests:
```javascript
const prNumber = context.payload.pull_request.number;
```
</details>

### Question 4
What's the difference between `github.rest` and `github.graphql`?

<details>
<summary>Answer</summary>

- `github.rest`: REST API calls, easier for simple operations
- `github.graphql`: GraphQL API, more efficient for complex queries with nested data

Example:
```javascript
// REST
await github.rest.issues.get({...});

// GraphQL
await github.graphql(query, variables);
```
</details>

---

## 🎯 Challenge Exercises

### Challenge 1: PR Template Checker
Create a workflow that checks if PR description includes required sections and comments if sections are missing.

**Hint:** Use `context.payload.pull_request.body` and check for headings.

### Challenge 2: Issue Dependency Tracker
Create a workflow that tracks dependencies between issues using keywords like "Depends on #123" and updates status automatically.

### Challenge 3: Azure Resource Validator
Create a workflow that validates Azure resource names before deployment and comments on PRs if naming doesn't follow conventions.

---

## 📖 Additional Resources

- [GitHub Script Documentation](https://github.com/actions/github-script)
- [Octokit REST API](https://octokit.github.io/rest.js/)
- [GitHub GraphQL API](https://docs.github.com/en/graphql)
- [Workflow Context](https://docs.github.com/en/actions/learn-github-actions/contexts)
- [GitHub API Rate Limits](https://docs.github.com/en/rest/overview/resources-in-the-rest-api#rate-limiting)

---

## 📝 Summary

In this lab, you learned:
- ✅ What GitHub Script is and when to use it
- ✅ How to interact with GitHub APIs using Octokit
- ✅ Automating issue labeling and management
- ✅ Auto-commenting on pull requests
- ✅ Creating issues from workflow failures
- ✅ Closing stale issues automatically
- ✅ Integrating GitHub Script with Azure
- ✅ Best practices for API interactions
- ✅ Error handling and troubleshooting

**Next Steps:**
- Complete Lab 8: GitHub Packages & Container Registry
- Explore more Octokit API endpoints
- Create custom automations for your projects
- Combine GitHub Script with Azure automation

---

**Estimated Time to Complete:** 45 minutes  
**Difficulty:** Intermediate  
**Prerequisites:** Labs 1, 5

Great job! You now have powerful GitHub automation skills! 🎉
