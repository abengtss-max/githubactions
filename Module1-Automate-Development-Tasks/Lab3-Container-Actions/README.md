# Lab 3: Building a Container Action for Azure

## 🎯 Lab Objectives

In this lab, you will:
- Understand Docker container basics
- Create a custom container action
- Build Azure CLI automation tools as container actions
- Package and version container actions
- Use container actions in workflows
- Publish actions to GitHub Marketplace (optional)

**Estimated Time:** 60 minutes  
**Difficulty:** Intermediate  
**Prerequisites:** Completed Lab 1 and Lab 2, Docker Desktop installed

## 📚 Background Information

### What are Container Actions?

**Container actions** package code and dependencies in a Docker container, providing a consistent execution environment regardless of the host runner.

**Architecture:**
```
┌─────────────────────────────────────────────────────┐
│              Container Action                        │
├─────────────────────────────────────────────────────┤
│                                                      │
│  ┌────────────────────────────────────────────┐    │
│  │          Docker Container                  │    │
│  │                                             │    │
│  │  ┌──────────────────────────────────────┐  │    │
│  │  │  Operating System (Ubuntu/Alpine)    │  │    │
│  │  └──────────────────────────────────────┘  │    │
│  │                                             │    │
│  │  ┌──────────────────────────────────────┐  │    │
│  │  │  Runtime (Python, Node.js, etc.)     │  │    │
│  │  └──────────────────────────────────────┘  │    │
│  │                                             │    │
│  │  ┌──────────────────────────────────────┐  │    │
│  │  │  Dependencies (Libraries, Tools)     │  │    │
│  │  └──────────────────────────────────────┘  │    │
│  │                                             │    │
│  │  ┌──────────────────────────────────────┐  │    │
│  │  │  Your Code (Action Logic)            │  │    │
│  │  └──────────────────────────────────────┘  │    │
│  │                                             │    │
│  └─────────────────────────────────────────────┘    │
│                                                      │
└─────────────────────────────────────────────────────┘
```

### Why Use Container Actions?

**Advantages:**
- ✅ **Consistent Environment:** Same execution environment every time
- ✅ **Complex Dependencies:** Package system libraries, tools, SDKs
- ✅ **Multiple Languages:** Use any programming language
- ✅ **Isolation:** No conflicts with runner environment
- ✅ **Reproducibility:** Container image ensures repeatability

**Disadvantages:**
- ❌ **Linux Only:** Can only run on Linux runners
- ❌ **Slower:** Container pull and startup overhead
- ❌ **Larger:** Container images consume more storage
- ❌ **Complex:** More components to maintain

**Best Use Cases for Container Actions:**
- Azure CLI automation
- Custom tool integration (Terraform, kubectl, etc.)
- Multi-language workflows
- Specific OS/library requirements
- Reusable organizational tools

### Container Action Metadata

Every container action needs an `action.yml` file:

```yaml
name: 'Action Name'
description: 'What the action does'
author: 'Your Name'

inputs:
  input-name:
    description: 'Input description'
    required: true
    default: 'default value'

outputs:
  output-name:
    description: 'Output description'

runs:
  using: 'docker'
  image: 'Dockerfile'
  
branding:
  icon: 'cloud'
  color: 'blue'
```

## 🔬 Lab Exercises

### Exercise 1: Create a Simple Container Action

**Objective:** Build a basic "Hello World" container action to understand the fundamentals.

#### Steps:

1. **Create New Repository**
   - Go to GitHub → New Repository
   - Name: `azure-greetings-action`
   - Description: `Custom container action for Azure workflows`
   - Visibility: Public
   - Add README
   - Click "Create repository"

2. **Clone Repository Locally**
   ```powershell
   cd C:\Users\alibengtsson\mycertifications\githubactions
   git clone https://github.com/YOUR-USERNAME/azure-greetings-action.git
   cd azure-greetings-action
   ```

3. **Create Action Metadata File**

   **Create `action.yml`:**
   ```yaml
   name: 'Azure Greetings'
   description: 'A container action that greets Azure users'
   author: 'Your Name'

   inputs:
     azure-region:
       description: 'Azure region to display'
       required: true
       default: 'eastus'
     user-name:
       description: 'Name of the user to greet'
       required: false
       default: 'Azure Developer'

   outputs:
     greeting-message:
       description: 'The generated greeting message'
     timestamp:
       description: 'Time when greeting was generated'

   runs:
     using: 'docker'
     image: 'Dockerfile'

   branding:
     icon: 'cloud'
     color: 'blue'
   ```

4. **Create Dockerfile**

   **Create `Dockerfile`:**
   ```dockerfile
   # Use official Python slim image
   FROM python:3.11-slim

   # Set working directory
   WORKDIR /action

   # Copy action script
   COPY entrypoint.sh /action/entrypoint.sh

   # Make script executable
   RUN chmod +x /action/entrypoint.sh

   # Set entrypoint
   ENTRYPOINT ["/action/entrypoint.sh"]
   ```

   **Understanding the Dockerfile:**
   - `FROM`: Base image (Python in this case)
   - `WORKDIR`: Sets working directory inside container
   - `COPY`: Copies files from host to container
   - `RUN`: Executes commands during image build
   - `ENTRYPOINT`: Command to run when container starts

5. **Create Entrypoint Script**

   **Create `entrypoint.sh`:**
   ```bash
   #!/bin/sh
   set -e

   # Read inputs from environment variables
   # GitHub Actions passes inputs as INPUT_<NAME> in uppercase
   AZURE_REGION="${INPUT_AZURE_REGION}"
   USER_NAME="${INPUT_USER_NAME}"

   # Generate greeting
   TIMESTAMP=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
   GREETING="👋 Hello ${USER_NAME}! Welcome to Azure ${AZURE_REGION}"

   # Print to console (visible in workflow logs)
   echo "::notice::${GREETING}"
   echo "Generated at: ${TIMESTAMP}"

   # Set outputs for use in subsequent workflow steps
   # Syntax: echo "::set-output name=<output-name>::<value>"
   echo "::set-output name=greeting-message::${GREETING}"
   echo "::set-output name=timestamp::${TIMESTAMP}"

   # Exit with success
   exit 0
   ```

   **Key Concepts:**
   - **Inputs:** Accessed via `INPUT_<NAME>` environment variables
   - **Outputs:** Set using `::set-output name=<name>::<value>`
   - **Logging:** Use `::notice::`, `::warning::`, `::error::`
   - **Exit Codes:** 0 = success, non-zero = failure

6. **Create README**

   **Create `README.md`:**
   ```markdown
   # Azure Greetings Action

   A simple container action that greets Azure users.

   ## Usage

   ```yaml
   - name: Greet User
     uses: YOUR-USERNAME/azure-greetings-action@v1
     with:
       azure-region: 'eastus'
       user-name: 'John Doe'
   ```

   ## Inputs

   - `azure-region` - Azure region (default: eastus)
   - `user-name` - User name (default: Azure Developer)

   ## Outputs

   - `greeting-message` - The generated greeting
   - `timestamp` - Generation timestamp
   ```

7. **Test Locally with Docker**

   ```powershell
   # Build the Docker image
   docker build -t azure-greetings-action .

   # Test run the container
   docker run --rm `
     -e INPUT_AZURE_REGION="westus" `
     -e INPUT_USER_NAME="Test User" `
     azure-greetings-action
   ```

   **Expected output:**
   ```
   👋 Hello Test User! Welcome to Azure westus
   Generated at: 2026-01-10T12:00:00Z
   ```

8. **Commit and Push**
   ```powershell
   git add .
   git commit -m "Initial container action implementation"
   git push origin main
   ```

9. **Create Version Tag**

   Container actions should be versioned:
   ```powershell
   # Create and push tag
   git tag -a v1 -m "Version 1.0.0"
   git push origin v1
   ```

   **Versioning Best Practices:**
   - Use semantic versioning (v1.0.0, v1.1.0, v2.0.0)
   - Create major version tags (v1, v2) that point to latest minor
   - Never force-push tags (breaking change for users)

10. **Test the Action in a Workflow**

    Create a test repository or use an existing one:

    **Create `.github/workflows/test-greetings.yml`:**
    ```yaml
    name: Test Greetings Action

    on:
      workflow_dispatch:

    jobs:
      test:
        runs-on: ubuntu-latest
        steps:
          - name: Checkout
            uses: actions/checkout@v4

          - name: Run Greetings Action
            id: greet
            uses: YOUR-USERNAME/azure-greetings-action@v1
            with:
              azure-region: 'westeurope'
              user-name: 'GitHub Actions User'

          - name: Display Outputs
            run: |
              echo "Greeting: ${{ steps.greet.outputs.greeting-message }}"
              echo "Timestamp: ${{ steps.greet.outputs.timestamp }}"
    ```

    Run manually and verify!

### Exercise 2: Build Azure Resource Lister Container Action

**Objective:** Create a production-ready container action that lists Azure resources.

#### Steps:

1. **Create New Repository**
   - Name: `azure-resource-lister-action`
   - Description: `List Azure resources using GitHub Actions`

2. **Clone and Setup**
   ```powershell
   git clone https://github.com/YOUR-USERNAME/azure-resource-lister-action.git
   cd azure-resource-lister-action
   ```

3. **Create Enhanced Action Metadata**

   **Create `action.yml`:**
   ```yaml
   name: 'Azure Resource Lister'
   description: 'Lists Azure resources in a subscription or resource group'
   author: 'Your Name'

   inputs:
     azure-credentials:
       description: 'Azure service principal credentials (JSON)'
       required: true
     resource-group:
       description: 'Resource group name (optional, leave empty for all resources)'
       required: false
       default: ''
     resource-type:
       description: 'Filter by resource type (e.g., Microsoft.Web/sites)'
       required: false
       default: ''
     output-format:
       description: 'Output format: table, json, yaml'
       required: false
       default: 'table'

   outputs:
     resource-count:
       description: 'Number of resources found'
     resources-json:
       description: 'Resources in JSON format'

   runs:
     using: 'docker'
     image: 'Dockerfile'

   branding:
     icon: 'list'
     color: 'blue'
   ```

4. **Create Advanced Dockerfile**

   **Create `Dockerfile`:**
   ```dockerfile
   # Use Azure CLI base image
   FROM mcr.microsoft.com/azure-cli:latest

   # Install additional tools
   RUN apk add --no-cache jq curl

   # Set working directory
   WORKDIR /action

   # Copy scripts
   COPY entrypoint.sh /action/entrypoint.sh
   COPY list-resources.sh /action/list-resources.sh

   # Make scripts executable
   RUN chmod +x /action/*.sh

   # Set entrypoint
   ENTRYPOINT ["/action/entrypoint.sh"]
   ```

   **Why This Dockerfile:**
   - Uses official Azure CLI image (includes az CLI)
   - Installs `jq` for JSON processing
   - Modular design with separate scripts

5. **Create Entrypoint Script**

   **Create `entrypoint.sh`:**
   ```bash
   #!/bin/sh
   set -e

   echo "::group::Azure Resource Lister Action"
   echo "Starting Azure resource listing..."

   # Validate inputs
   if [ -z "${INPUT_AZURE_CREDENTIALS}" ]; then
     echo "::error::azure-credentials input is required"
     exit 1
   fi

   # Parse Azure credentials
   echo "Parsing Azure credentials..."
   CLIENT_ID=$(echo "${INPUT_AZURE_CREDENTIALS}" | jq -r '.clientId')
   CLIENT_SECRET=$(echo "${INPUT_AZURE_CREDENTIALS}" | jq -r '.clientSecret')
   TENANT_ID=$(echo "${INPUT_AZURE_CREDENTIALS}" | jq -r '.tenantId')
   SUBSCRIPTION_ID=$(echo "${INPUT_AZURE_CREDENTIALS}" | jq -r '.subscriptionId')

   # Login to Azure
   echo "::notice::Logging into Azure..."
   az login --service-principal \
     --username "${CLIENT_ID}" \
     --password "${CLIENT_SECRET}" \
     --tenant "${TENANT_ID}" > /dev/null 2>&1

   # Set subscription
   az account set --subscription "${SUBSCRIPTION_ID}"

   # Verify login
   ACCOUNT_INFO=$(az account show --query "{name:name, id:id}" -o table)
   echo "::notice::Logged in to subscription:"
   echo "${ACCOUNT_INFO}"

   echo "::endgroup::"

   # Call resource listing script
   /action/list-resources.sh

   # Logout
   echo "::group::Cleanup"
   echo "Logging out from Azure..."
   az logout
   echo "::endgroup::"

   exit 0
   ```

6. **Create Resource Listing Script**

   **Create `list-resources.sh`:**
   ```bash
   #!/bin/sh
   set -e

   echo "::group::Listing Azure Resources"

   RESOURCE_GROUP="${INPUT_RESOURCE_GROUP}"
   RESOURCE_TYPE="${INPUT_RESOURCE_TYPE}"
   OUTPUT_FORMAT="${INPUT_OUTPUT_FORMAT}"

   # Build query command
   if [ -n "${RESOURCE_GROUP}" ]; then
     echo "::notice::Listing resources in resource group: ${RESOURCE_GROUP}"
     QUERY_CMD="az resource list --resource-group ${RESOURCE_GROUP}"
   else
     echo "::notice::Listing all resources in subscription"
     QUERY_CMD="az resource list"
   fi

   # Add type filter if specified
   if [ -n "${RESOURCE_TYPE}" ]; then
     echo "::notice::Filtering by type: ${RESOURCE_TYPE}"
     QUERY_CMD="${QUERY_CMD} --resource-type ${RESOURCE_TYPE}"
   fi

   # Execute query
   RESOURCES_JSON=$(eval "${QUERY_CMD}" -o json)
   RESOURCE_COUNT=$(echo "${RESOURCES_JSON}" | jq '. | length')

   echo "::notice::Found ${RESOURCE_COUNT} resources"

   # Output in requested format
   case "${OUTPUT_FORMAT}" in
     json)
       echo "${RESOURCES_JSON}" | jq '.'
       ;;
     yaml)
       echo "${RESOURCES_JSON}" | jq -r 'to_entries | .[] | "- name: \(.value.name)\n  type: \(.value.type)\n  location: \(.value.location)"'
       ;;
     table|*)
       echo "${RESOURCES_JSON}" | jq -r '.[] | [.name, .type, .location, .resourceGroup] | @tsv' | \
         awk 'BEGIN {printf "%-40s %-50s %-15s %-30s\n", "NAME", "TYPE", "LOCATION", "RESOURCE GROUP"; printf "%.0s-" {1..140}; printf "\n"} {printf "%-40s %-50s %-15s %-30s\n", $1, $2, $3, $4}'
       ;;
   esac

   # Set outputs
   echo "::set-output name=resource-count::${RESOURCE_COUNT}"
   
   # Save JSON output (truncate if too large)
   if [ "${RESOURCE_COUNT}" -lt 100 ]; then
     echo "::set-output name=resources-json::${RESOURCES_JSON}"
   else
     echo "::warning::Too many resources to output as JSON. Use separate Azure CLI query."
   fi

   echo "::endgroup::"
   ```

7. **Create Comprehensive README**

   **Create `README.md`:**
   ```markdown
   # Azure Resource Lister Action

   GitHub Action to list Azure resources using Azure CLI in a container.

   ## Features

   - ✅ List all resources in subscription
   - ✅ Filter by resource group
   - ✅ Filter by resource type
   - ✅ Multiple output formats (table, JSON, YAML)
   - ✅ Secure authentication with service principal

   ## Usage

   ### List All Resources

   ```yaml
   - name: List Azure Resources
     uses: YOUR-USERNAME/azure-resource-lister-action@v1
     with:
       azure-credentials: ${{ secrets.AZURE_CREDENTIALS }}
       output-format: 'table'
   ```

   ### List Resources in Resource Group

   ```yaml
   - name: List Resources in RG
     uses: YOUR-USERNAME/azure-resource-lister-action@v1
     with:
       azure-credentials: ${{ secrets.AZURE_CREDENTIALS }}
       resource-group: 'my-resource-group'
   ```

   ### Filter by Type

   ```yaml
   - name: List Web Apps
     uses: YOUR-USERNAME/azure-resource-lister-action@v1
     with:
       azure-credentials: ${{ secrets.AZURE_CREDENTIALS }}
       resource-type: 'Microsoft.Web/sites'
       output-format: 'json'
   ```

   ## Inputs

   | Input | Description | Required | Default |
   |-------|-------------|----------|---------|
   | `azure-credentials` | Azure SP credentials JSON | Yes | - |
   | `resource-group` | Resource group name | No | (all) |
   | `resource-type` | Resource type filter | No | (all) |
   | `output-format` | Output format (table/json/yaml) | No | table |

   ## Outputs

   | Output | Description |
   |--------|-------------|
   | `resource-count` | Number of resources found |
   | `resources-json` | Resources in JSON format |

   ## Setup

   ### 1. Create Azure Service Principal

   ```bash
   az ad sp create-for-rbac \
     --name "github-actions-reader" \
     --role "Reader" \
     --scopes /subscriptions/{subscription-id} \
     --sdk-auth
   ```

   ### 2. Add to GitHub Secrets

   Add the output JSON as `AZURE_CREDENTIALS` secret.

   ## Examples

   ### Complete Workflow

   ```yaml
   name: Azure Resource Inventory

   on:
     schedule:
       - cron: '0 0 * * 0'  # Weekly
     workflow_dispatch:

   jobs:
     inventory:
       runs-on: ubuntu-latest
       steps:
         - name: List All Resources
           id: list
           uses: YOUR-USERNAME/azure-resource-lister-action@v1
           with:
             azure-credentials: ${{ secrets.AZURE_CREDENTIALS }}
             output-format: 'table'

         - name: Display Count
           run: |
             echo "Total resources: ${{ steps.list.outputs.resource-count }}"
   ```

   ## License

   MIT
   ```

8. **Build and Test Locally**

   ```powershell
   # Build image
   docker build -t azure-resource-lister-action .

   # Test (requires Azure credentials)
   docker run --rm `
     -e INPUT_AZURE_CREDENTIALS='{"clientId":"...","clientSecret":"...","tenantId":"...","subscriptionId":"..."}' `
     -e INPUT_OUTPUT_FORMAT="table" `
     azure-resource-lister-action
   ```

9. **Commit, Tag, and Push**

   ```powershell
   git add .
   git commit -m "Add Azure resource lister container action"
   git tag -a v1.0.0 -m "First release"
   git tag -a v1 -m "Major version 1"
   git push origin main --tags
   ```

### Exercise 3: Use Container Action in Real Workflow

**Objective:** Integrate the container action into a practical Azure automation workflow.

#### Steps:

1. **Create Automation Repository**
   
   Use your existing `azure-nodejs-webapp` repository or create new one.

2. **Create Resource Audit Workflow**

   **Create `.github/workflows/azure-audit.yml`:**
   ```yaml
   name: Azure Resource Audit

   on:
     schedule:
       - cron: '0 9 * * 1'  # Every Monday at 9 AM UTC
     workflow_dispatch:

   jobs:
     audit-resources:
       name: Audit Azure Resources
       runs-on: ubuntu-latest
       
       steps:
         - name: Checkout
           uses: actions/checkout@v4

         - name: List All Resources
           id: all-resources
           uses: YOUR-USERNAME/azure-resource-lister-action@v1
           with:
             azure-credentials: ${{ secrets.AZURE_CREDENTIALS }}
             output-format: 'table'

         - name: List Web Apps Only
           id: webapps
           uses: YOUR-USERNAME/azure-resource-lister-action@v1
           with:
             azure-credentials: ${{ secrets.AZURE_CREDENTIALS }}
             resource-type: 'Microsoft.Web/sites'
             output-format: 'json'

         - name: Generate Report
           run: |
             echo "# Azure Resource Audit Report" > audit-report.md
             echo "" >> audit-report.md
             echo "**Date:** $(date -u +"%Y-%m-%d %H:%M:%S UTC")" >> audit-report.md
             echo "" >> audit-report.md
             echo "## Summary" >> audit-report.md
             echo "- Total Resources: ${{ steps.all-resources.outputs.resource-count }}" >> audit-report.md
             echo "- Web Apps: ${{ steps.webapps.outputs.resource-count }}" >> audit-report.md
             echo "" >> audit-report.md

         - name: Upload Report
           uses: actions/upload-artifact@v4
           with:
             name: azure-audit-report
             path: audit-report.md
             retention-days: 30

         - name: Send Notification (Optional)
           run: |
             echo "✅ Audit completed successfully!"
             echo "Total resources: ${{ steps.all-resources.outputs.resource-count }}"
   ```

3. **Test the Workflow**
   - Run manually via Actions tab
   - Verify container action executes
   - Check artifact upload
   - Review logs for any issues

### Exercise 4: Advanced Container Action Patterns

**Objective:** Learn advanced techniques for production container actions.

#### Pattern 1: Multi-Stage Docker Build

**Optimized Dockerfile:**
```dockerfile
# Build stage
FROM mcr.microsoft.com/azure-cli:latest AS builder

# Install build dependencies
RUN apk add --no-cache python3 py3-pip

# Install Python packages
COPY requirements.txt /tmp/
RUN pip3 install --no-cache-dir -r /tmp/requirements.txt

# Runtime stage
FROM mcr.microsoft.com/azure-cli:latest

# Copy only necessary files from builder
COPY --from=builder /usr/lib/python3.11/site-packages /usr/lib/python3.11/site-packages

# Install runtime dependencies only
RUN apk add --no-cache jq curl

# Copy action scripts
WORKDIR /action
COPY entrypoint.sh /action/

RUN chmod +x /action/entrypoint.sh

ENTRYPOINT ["/action/entrypoint.sh"]
```

**Benefits:**
- Smaller final image
- Faster pull times
- Reduced attack surface

#### Pattern 2: Error Handling and Logging

**Robust entrypoint.sh:**
```bash
#!/bin/sh
set -e  # Exit on error

# Error handler
handle_error() {
  echo "::error::Action failed at line $1"
  az logout 2>/dev/null || true
  exit 1
}

trap 'handle_error $LINENO' ERR

# Logging function
log_info() {
  echo "::notice::$1"
}

log_error() {
  echo "::error::$1"
}

# Validate inputs
validate_inputs() {
  if [ -z "${INPUT_AZURE_CREDENTIALS}" ]; then
    log_error "azure-credentials is required"
    exit 1
  fi
}

# Main logic
main() {
  log_info "Starting action..."
  validate_inputs
  
  # Your action logic here
  
  log_info "Action completed successfully"
}

main
```

#### Pattern 3: Action Versioning Strategy

**Create version management script:**

**Create `scripts/release.sh`:**
```bash
#!/bin/bash
set -e

VERSION=$1

if [ -z "$VERSION" ]; then
  echo "Usage: ./scripts/release.sh v1.2.3"
  exit 1
fi

# Extract major version
MAJOR_VERSION=$(echo $VERSION | cut -d. -f1)

# Create and push tags
git tag -a $VERSION -m "Release $VERSION"
git tag -fa $MAJOR_VERSION -m "Update $MAJOR_VERSION to $VERSION"

git push origin $VERSION
git push origin $MAJOR_VERSION --force

echo "Released $VERSION (major: $MAJOR_VERSION)"
```

Usage:
```powershell
chmod +x scripts/release.sh
./scripts/release.sh v1.2.3
```

## 📝 Knowledge Check

1. **What's the main advantage of container actions over JavaScript actions?**
   <details>
   <summary>Answer</summary>
   Container actions provide a consistent execution environment with all dependencies packaged, while JavaScript actions rely on the runner's environment.
   </details>

2. **Why can container actions only run on Linux runners?**
   <details>
   <summary>Answer</summary>
   GitHub Actions uses Docker to run container actions, and Docker containers on GitHub runners are Linux-based.
   </details>

3. **How are inputs passed to container actions?**
   <details>
   <summary>Answer</summary>
   Inputs are passed as environment variables with the prefix INPUT_ in uppercase (e.g., INPUT_AZURE_REGION).
   </details>

4. **What's the purpose of the branding section in action.yml?**
   <details>
   <summary>Answer</summary>
   Branding defines the icon and color displayed in the GitHub Actions Marketplace and workflow visualizations.
   </details>

5. **Why use multi-stage Docker builds?**
   <details>
   <summary>Answer</summary>
   Multi-stage builds reduce final image size by separating build-time dependencies from runtime dependencies.
   </details>

## 🎓 Lab Summary

**Congratulations!** You've mastered container actions!

**You learned how to:**
✅ Create custom container actions from scratch  
✅ Build Dockerfiles for GitHub Actions  
✅ Handle inputs and outputs in containers  
✅ Integrate Azure CLI in container actions  
✅ Version and release actions properly  
✅ Use container actions in workflows  
✅ Implement production-ready patterns  

## 🔜 Next Steps

Proceed to **[Lab 4: Configuring Advanced Workflow Triggers](../Lab4-Advanced-Triggers/README.md)** where you'll:
- Master scheduled workflows with cron
- Implement repository_dispatch events
- Create workflow_dispatch with inputs
- Build event-driven Azure automation
- Implement workflow reusability

## 📚 Additional Resources

- [Docker Documentation](https://docs.docker.com/)
- [GitHub Actions - Creating Container Actions](https://docs.github.com/en/actions/creating-actions/creating-a-docker-container-action)
- [Azure CLI Docker Image](https://hub.docker.com/_/microsoft-azure-cli)
- [Action Metadata Syntax](https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions)

## 🧹 Cleanup

Remove Docker images:
```powershell
docker rmi azure-greetings-action
docker rmi azure-resource-lister-action
```

---

**Need Help?** Test containers locally before pushing to GitHub!
