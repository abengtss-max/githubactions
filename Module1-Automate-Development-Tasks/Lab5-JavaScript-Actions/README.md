# Lab 5: Creating JavaScript Actions with Azure SDK

## 🎯 Lab Objectives

In this lab, you will:
- Understand JavaScript action architecture
- Build JavaScript actions using GitHub Actions Toolkit
- Integrate Azure SDK for JavaScript
- Create cross-platform actions
- Package and publish JavaScript actions
- Implement error handling and logging

**Estimated Time:** 60 minutes  
**Difficulty:** Intermediate  
**Prerequisites:** Completed Labs 1-4, Node.js 18+ installed

## 📚 Background Information

### Why JavaScript Actions?

**JavaScript Actions** run directly on the runner machine without container overhead, making them:

**Advantages:**
- ⚡ **Faster execution** - No container pull/build time
- 🌐 **Cross-platform** - Run on Linux, Windows, macOS
- 📦 **Simpler packaging** - Just JavaScript files
- 🔄 **Easier development** - Faster iteration cycles
- 💰 **Cost-effective** - Less execution time

**When to Use:**
- API integrations
- File operations
- Simple automation tasks
- Cross-platform requirements
- Performance-critical workflows

### JavaScript Action Structure

```
my-action/
├── action.yml          # Action metadata
├── index.js            # Main entry point
├── package.json        # NPM dependencies
├── package-lock.json   # Locked dependencies
├── node_modules/       # Dependencies (committed!)
├── dist/              # Compiled code (if TypeScript)
└── README.md          # Documentation
```

**Important:** Unlike typical Node.js projects, JavaScript actions **must commit node_modules** because GitHub runners don't run npm install.

### GitHub Actions Toolkit

GitHub provides `@actions/core` and `@actions/toolkit` packages:

```javascript
const core = require('@actions/core');
const github = require('@actions/github');

// Get inputs
const myInput = core.getInput('my-input');

// Set outputs
core.setOutput('result', 'value');

// Logging
core.info('Information message');
core.warning('Warning message');
core.error('Error message');

// Fail action
core.setFailed('Action failed!');
```

## 🔬 Lab Exercises

### Exercise 1: Create a Simple JavaScript Action

**Objective:** Build a basic JavaScript action to understand the fundamentals.

#### Steps:

1. **Create Repository**
   ```powershell
   cd C:\Users\alibengtsson\mycertifications\githubactions
   mkdir azure-greeting-js-action
   cd azure-greeting-js-action
   git init
   ```

2. **Initialize NPM Project**
   ```powershell
   npm init -y
   ```

3. **Install GitHub Actions Core**
   ```powershell
   npm install @actions/core @actions/github
   ```

4. **Create Action Metadata**

   **Create `action.yml`:**
   ```yaml
   name: 'Azure JavaScript Greeting'
   description: 'Greets Azure users using JavaScript'
   author: 'Your Name'
   
   inputs:
     name:
       description: 'Name to greet'
       required: true
       default: 'Azure Developer'
     region:
       description: 'Azure region'
       required: false
       default: 'eastus'
   
   outputs:
     greeting:
       description: 'The greeting message'
     time:
       description: 'Time of greeting'
   
   runs:
     using: 'node20'
     main: 'index.js'
   ```

5. **Create Main JavaScript File**

   **Create `index.js`:**
   ```javascript
   const core = require('@actions/core');
   const github = require('@actions/github');

   async function run() {
     try {
       // Get inputs
       const name = core.getInput('name', { required: true });
       const region = core.getInput('region');

       // Log information
       core.info(`Starting greeting action...`);
       core.info(`Name: ${name}`);
       core.info(`Region: ${region}`);

       // Generate greeting
       const greeting = `👋 Hello ${name}! Welcome to Azure ${region}!`;
       const time = new Date().toISOString();

       // Set outputs
       core.setOutput('greeting', greeting);
       core.setOutput('time', time);

       // Log success
       core.notice(greeting);
       
       // Get workflow context
       core.info(`Workflow: ${github.context.workflow}`);
       core.info(`Actor: ${github.context.actor}`);
       
     } catch (error) {
       core.setFailed(`Action failed: ${error.message}`);
     }
   }

   run();
   ```

6. **Create README**

   **Create `README.md`:**
   ```markdown
   # Azure JavaScript Greeting Action

   A simple JavaScript action that greets Azure users.

   ## Usage

   ```yaml
   - name: Greet User
     uses: YOUR-USERNAME/azure-greeting-js-action@v1
     with:
       name: 'John Doe'
       region: 'westus'
   ```

   ## Inputs

   - `name` - Name to greet (required)
   - `region` - Azure region (default: eastus)

   ## Outputs

   - `greeting` - The greeting message
   - `time` - Timestamp of greeting
   ```

7. **Test Locally**

   Create `test-local.js`:
   ```javascript
   // Mock the action inputs
   process.env.INPUT_NAME = 'Test User';
   process.env.INPUT_REGION = 'westeurope';

   // Run the action
   require('./index');
   ```

   Run: `node test-local.js`

8. **Commit and Push**
   ```powershell
   # IMPORTANT: Commit node_modules!
   git add .
   git commit -m "Initial JavaScript action"
   
   # Create GitHub repo and push
   gh repo create azure-greeting-js-action --public --source=. --push
   
   # Tag version
   git tag -a v1 -m "Version 1"
   git push origin v1
   ```

### Exercise 2: Build Azure Resource Manager JavaScript Action

**Objective:** Create a production-ready action using Azure SDK.

#### Steps:

1. **Create New Repository**
   ```powershell
   mkdir azure-resource-manager-js
   cd azure-resource-manager-js
   npm init -y
   ```

2. **Install Dependencies**
   ```powershell
   npm install @actions/core @actions/github
   npm install @azure/identity @azure/arm-resources @azure/arm-subscriptions
   ```

3. **Create Enhanced Action Metadata**

   **Create `action.yml`:**
   ```yaml
   name: 'Azure Resource Manager JS'
   description: 'Manage Azure resources using JavaScript and Azure SDK'
   author: 'Your Name'
   
   inputs:
     azure-credentials:
       description: 'Azure service principal credentials (JSON)'
       required: true
     action-type:
       description: 'Action type: list, get, delete, check'
       required: true
       default: 'list'
     resource-group:
       description: 'Resource group name'
       required: false
     resource-name:
       description: 'Specific resource name'
       required: false
   
   outputs:
     resource-count:
       description: 'Number of resources found'
     result:
       description: 'Action result details'
   
   runs:
     using: 'node20'
     main: 'index.js'
   
   branding:
     icon: 'database'
     color: 'blue'
   ```

4. **Create Main Action Logic**

   **Create `index.js`:**
   ```javascript
   const core = require('@actions/core');
   const { ClientSecretCredential } = require('@azure/identity');
   const { ResourceManagementClient } = require('@azure/arm-resources');

   async function run() {
     try {
       core.info('🚀 Starting Azure Resource Manager Action');

       // Get inputs
       const azureCredsJson = core.getInput('azure-credentials', { required: true });
       const actionType = core.getInput('action-type', { required: true });
       const resourceGroup = core.getInput('resource-group');
       const resourceName = core.getInput('resource-name');

       // Parse Azure credentials
       const azureCreds = JSON.parse(azureCredsJson);
       core.info('✅ Parsed Azure credentials');

       // Create credential
       const credential = new ClientSecretCredential(
         azureCreds.tenantId,
         azureCreds.clientId,
         azureCreds.clientSecret
       );

       // Create resource management client
       const client = new ResourceManagementClient(
         credential,
         azureCreds.subscriptionId
       );
       core.info('✅ Connected to Azure');

       // Execute action
       let result;
       switch (actionType) {
         case 'list':
           result = await listResources(client, resourceGroup);
           break;
         case 'get':
           result = await getResource(client, resourceGroup, resourceName);
           break;
         case 'check':
           result = await checkResourceExists(client, resourceGroup, resourceName);
           break;
         default:
           throw new Error(`Unknown action type: ${actionType}`);
       }

       // Set outputs
       core.setOutput('result', JSON.stringify(result));
       core.info('✅ Action completed successfully');

     } catch (error) {
       core.setFailed(`❌ Action failed: ${error.message}`);
       throw error;
     }
   }

   async function listResources(client, resourceGroup) {
     core.info('📋 Listing resources...');
     
     let resources = [];
     let iterator;

     if (resourceGroup) {
       core.info(`Listing resources in resource group: ${resourceGroup}`);
       iterator = client.resources.listByResourceGroup(resourceGroup);
     } else {
       core.info('Listing all resources in subscription');
       iterator = client.resources.list();
     }

     for await (const resource of iterator) {
       resources.push({
         name: resource.name,
         type: resource.type,
         location: resource.location,
         id: resource.id
       });
     }

     core.info(`Found ${resources.length} resources`);
     core.setOutput('resource-count', resources.length);

     // Log summary table
     core.startGroup('📊 Resources Summary');
     resources.forEach(r => {
       core.info(`  • ${r.name} (${r.type}) - ${r.location}`);
     });
     core.endGroup();

     return resources;
   }

   async function getResource(client, resourceGroup, resourceName) {
     if (!resourceGroup || !resourceName) {
       throw new Error('resource-group and resource-name required for get action');
     }

     core.info(`Getting resource: ${resourceName} in ${resourceGroup}`);

     // List resources and find matching one
     const iterator = client.resources.listByResourceGroup(resourceGroup);
     
     for await (const resource of iterator) {
       if (resource.name === resourceName) {
         core.info(`✅ Found resource: ${resource.name}`);
         return {
           name: resource.name,
           type: resource.type,
           location: resource.location,
           id: resource.id,
           properties: resource.properties
         };
       }
     }

     throw new Error(`Resource ${resourceName} not found in ${resourceGroup}`);
   }

   async function checkResourceExists(client, resourceGroup, resourceName) {
     core.info(`Checking if resource exists: ${resourceName}`);

     try {
       const resource = await getResource(client, resourceGroup, resourceName);
       core.info('✅ Resource exists');
       return { exists: true, resource };
     } catch (error) {
       core.info('❌ Resource does not exist');
       return { exists: false };
     }
   }

   run();
   ```

5. **Create Package Configuration**

   Update `package.json`:
   ```json
   {
     "name": "azure-resource-manager-js",
     "version": "1.0.0",
     "description": "JavaScript action for Azure Resource Management",
     "main": "index.js",
     "scripts": {
       "test": "node test-local.js"
     },
     "keywords": ["azure", "github-actions", "resource-management"],
     "author": "Your Name",
     "license": "MIT",
     "dependencies": {
       "@actions/core": "^1.10.1",
       "@actions/github": "^6.0.0",
       "@azure/arm-resources": "^5.2.0",
       "@azure/identity": "^4.0.0",
       "@azure/arm-subscriptions": "^5.1.0"
     }
   }
   ```

6. **Create Comprehensive README**

   **Create `README.md`:**
   ```markdown
   # Azure Resource Manager JavaScript Action

   Manage Azure resources using JavaScript and Azure SDK.

   ## Features

   - ✅ List all resources or by resource group
   - ✅ Get specific resource details
   - ✅ Check resource existence
   - ✅ Fast execution (no container overhead)
   - ✅ Cross-platform support

   ## Usage Examples

   ### List All Resources

   ```yaml
   - name: List Resources
     uses: YOUR-USERNAME/azure-resource-manager-js@v1
     with:
       azure-credentials: ${{ secrets.AZURE_CREDENTIALS }}
       action-type: 'list'
   ```

   ### List Resources in Resource Group

   ```yaml
   - name: List RG Resources
     uses: YOUR-USERNAME/azure-resource-manager-js@v1
     with:
       azure-credentials: ${{ secrets.AZURE_CREDENTIALS }}
       action-type: 'list'
       resource-group: 'my-resource-group'
   ```

   ### Get Specific Resource

   ```yaml
   - name: Get Resource
     id: get-resource
     uses: YOUR-USERNAME/azure-resource-manager-js@v1
     with:
       azure-credentials: ${{ secrets.AZURE_CREDENTIALS }}
       action-type: 'get'
       resource-group: 'my-rg'
       resource-name: 'my-webapp'

   - name: Display Result
     run: echo "${{ steps.get-resource.outputs.result }}"
   ```

   ### Check Resource Exists

   ```yaml
   - name: Check Resource
     id: check
     uses: YOUR-USERNAME/azure-resource-manager-js@v1
     with:
       azure-credentials: ${{ secrets.AZURE_CREDENTIALS }}
       action-type: 'check'
       resource-group: 'my-rg'
       resource-name: 'my-resource'

   - name: Conditional Step
     if: steps.check.outputs.result.exists == 'true'
     run: echo "Resource exists!"
   ```

   ## Inputs

   | Input | Description | Required | Default |
   |-------|-------------|----------|---------|
   | `azure-credentials` | Azure SP credentials JSON | Yes | - |
   | `action-type` | Action: list, get, check | Yes | list |
   | `resource-group` | Resource group name | No | - |
   | `resource-name` | Resource name | No | - |

   ## Outputs

   | Output | Description |
   |--------|-------------|
   | `resource-count` | Number of resources (list action) |
   | `result` | Action result (JSON) |

   ## Development

   ```bash
   npm install
   npm test
   ```

   ## License

   MIT
   ```

7. **Commit and Publish**

   ```powershell
   # Install dependencies
   npm install

   # Commit everything including node_modules
   git add .
   git commit -m "Azure Resource Manager JS Action"
   
   # Create repo and push
   gh repo create azure-resource-manager-js --public --source=. --push
   
   # Tag version
   git tag -a v1.0.0 -m "First release"
   git tag -a v1 -m "Major version 1"
   git push origin --tags
   ```

### Exercise 3: Advanced Patterns for JavaScript Actions

**Objective:** Learn production-ready patterns for robust JavaScript actions.

#### Pattern 1: Error Handling

**Create `utils/errors.js`:**
```javascript
const core = require('@actions/core');

class ActionError extends Error {
  constructor(message, details = {}) {
    super(message);
    this.name = 'ActionError';
    this.details = details;
  }
}

function handleError(error) {
  if (error instanceof ActionError) {
    core.error(`${error.message}`);
    if (error.details) {
      core.error(`Details: ${JSON.stringify(error.details)}`);
    }
  } else {
    core.error(`Unexpected error: ${error.message}`);
    core.error(error.stack);
  }
  
  core.setFailed(error.message);
}

module.exports = { ActionError, handleError };
```

#### Pattern 2: Input Validation

**Create `utils/validation.js`:**
```javascript
const core = require('@actions/core');
const { ActionError } = require('./errors');

function validateInputs(inputs) {
  const errors = [];

  // Check required inputs
  if (!inputs.azureCredentials) {
    errors.push('azure-credentials is required');
  }

  // Validate JSON format
  if (inputs.azureCredentials) {
    try {
      JSON.parse(inputs.azureCredentials);
    } catch (e) {
      errors.push('azure-credentials must be valid JSON');
    }
  }

  // Validate action type
  const validActions = ['list', 'get', 'check', 'delete'];
  if (!validActions.includes(inputs.actionType)) {
    errors.push(`action-type must be one of: ${validActions.join(', ')}`);
  }

  if (errors.length > 0) {
    throw new ActionError('Input validation failed', { errors });
  }

  core.info('✅ Input validation passed');
}

module.exports = { validateInputs };
```

#### Pattern 3: Logging Utilities

**Create `utils/logging.js`:**
```javascript
const core = require('@actions/core');

function logResourceSummary(resources) {
  core.startGroup('📊 Resource Summary');
  
  const typeCount = {};
  const locationCount = {};

  resources.forEach(resource => {
    // Count by type
    typeCount[resource.type] = (typeCount[resource.type] || 0) + 1;
    // Count by location
    locationCount[resource.location] = (locationCount[resource.location] || 0) + 1;
  });

  core.info('\nBy Type:');
  Object.entries(typeCount).forEach(([type, count]) => {
    core.info(`  ${type}: ${count}`);
  });

  core.info('\nBy Location:');
  Object.entries(locationCount).forEach(([location, count]) => {
    core.info(`  ${location}: ${count}`);
  });

  core.endGroup();
}

function logProgress(current, total, message) {
  const percent = Math.round((current / total) * 100);
  core.info(`[${percent}%] ${message}`);
}

module.exports = { logResourceSummary, logProgress };
```

## 📝 Knowledge Check

1. **What's the main advantage of JavaScript actions over container actions?**
   <details>
   <summary>Answer</summary>
   Faster execution (no container overhead) and cross-platform support (Linux, Windows, macOS).
   </details>

2. **Why must JavaScript actions commit node_modules?**
   <details>
   <summary>Answer</summary>
   GitHub runners don't run npm install, so all dependencies must be included in the repository.
   </details>

3. **What package provides action utilities like getInput and setOutput?**
   <details>
   <summary>Answer</summary>
   @actions/core package from GitHub Actions Toolkit.
   </details>

## 🎓 Lab Summary

**Congratulations! You've mastered JavaScript actions!**

✅ Created JavaScript actions from scratch  
✅ Integrated Azure SDK for JavaScript  
✅ Implemented error handling and validation  
✅ Built production-ready patterns  
✅ Published versioned actions  

## 🔜 Next Steps

Proceed to **[Lab 6: End-to-End CI/CD Pipeline to Azure](../Lab6-Complete-Pipeline/README.md)** for the ultimate challenge!

---

**Next:** Build a complete production pipeline with all best practices!
