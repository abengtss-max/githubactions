# Lab 8: GitHub Packages & Container Registry

**Duration:** 60 minutes  
**Difficulty:** Intermediate  
**Prerequisites:** Lab 2 (CI/CD) and Lab 3 (Container Actions)

## 🎯 Learning Objectives

By the end of this lab, you will be able to:
- Understand GitHub Packages and GitHub Container Registry (GHCR)
- Publish npm packages to GitHub Packages
- Build and publish Docker images to GHCR
- Use published packages in other projects
- Implement versioning and tagging strategies
- Pull and run containers from GHCR locally
- Integrate package publishing with CI/CD pipelines
- Publish packages to Azure Container Registry as well

---

## 📖 What is GitHub Packages?

**GitHub Packages** is a package hosting service integrated with GitHub that allows you to host and manage packages alongside your code.

### Supported Package Types

| Package Type | Registry | Use Case |
|--------------|----------|----------|
| npm | `npm.pkg.github.com` | Node.js libraries |
| Docker | `ghcr.io` | Container images |
| Maven | `maven.pkg.github.com` | Java libraries |
| NuGet | `nuget.pkg.github.com` | .NET packages |
| RubyGems | `rubygems.pkg.github.com` | Ruby libraries |

### GitHub Container Registry (GHCR)

**GHCR** (`ghcr.io`) is GitHub's container registry for Docker images, supporting:
- ✅ Public and private images
- ✅ Fine-grained access control
- ✅ Integration with GitHub Actions
- ✅ Anonymous pulls for public images
- ✅ OCI-compliant storage

### Benefits

- 🔗 **Integrated with GitHub:** Packages are linked to repositories
- 🔐 **Secure:** Use GitHub tokens for authentication
- 📦 **Versioned:** Semantic versioning support
- 🌐 **Public/Private:** Control access like repositories
- 🆓 **Free for public packages:** Generous free tier

---

## 🏗️ Lab Architecture

```
Source Code → Build → Package → Publish → Registry → Download → Use
     ↓          ↓        ↓         ↓          ↓          ↓       ↓
  GitHub    Workflow  npm/Docker  GHCR    ghcr.io   npm install   App
  Repo      Actions   Package     Publish           docker pull
```

---

## 📋 Prerequisites

Before starting this lab:

1. ✅ Completed Lab 2 (CI/CD workflows)
2. ✅ Completed Lab 3 (Container actions)
3. ✅ GitHub account with package access
4. ✅ Docker Desktop installed
5. ✅ Node.js 18+ installed
6. ✅ (Optional) Azure subscription for ACR integration

---

## 🚀 Exercise 1: Publish npm Package to GitHub Packages

### What You'll Build
A reusable npm library published to GitHub Packages with automated publishing on release.

### Step 1: Create npm Package Repository

1. **Create repository** named `azure-utils-lib`:
   ```powershell
   cd C:\Users\alibengtsson\mycertifications\githubactions
   mkdir azure-utils-lib
   cd azure-utils-lib
   git init
   ```

2. **Initialize npm package**:
   ```powershell
   npm init -y
   ```

3. **Update `package.json`**:
   ```json
   {
     "name": "@YOUR_USERNAME/azure-utils-lib",
     "version": "1.0.0",
     "description": "Azure utility functions for Node.js",
     "main": "index.js",
     "scripts": {
       "test": "jest"
     },
     "repository": {
       "type": "git",
       "url": "git+https://github.com/YOUR_USERNAME/azure-utils-lib.git"
     },
     "keywords": ["azure", "utilities", "nodejs"],
     "author": "Your Name",
     "license": "MIT",
     "publishConfig": {
       "registry": "https://npm.pkg.github.com"
     }
   }
   ```

   **Important:** Replace `YOUR_USERNAME` with your GitHub username.

### Step 2: Create Library Code

Create `index.js`:

```javascript
/**
 * Azure Utility Functions
 * @module azure-utils-lib
 */

/**
 * Parse Azure resource ID
 * @param {string} resourceId - Azure resource ID
 * @returns {object} Parsed components
 */
function parseResourceId(resourceId) {
  const regex = /\/subscriptions\/([^\/]+)\/resourceGroups\/([^\/]+)\/providers\/([^\/]+)\/([^\/]+)\/([^\/]+)/;
  const match = resourceId.match(regex);
  
  if (!match) {
    throw new Error('Invalid Azure resource ID format');
  }
  
  return {
    subscriptionId: match[1],
    resourceGroup: match[2],
    provider: match[3],
    resourceType: match[4],
    resourceName: match[5]
  };
}

/**
 * Generate Azure resource group name
 * @param {string} environment - Environment (dev, staging, prod)
 * @param {string} application - Application name
 * @param {string} region - Azure region code
 * @returns {string} Resource group name
 */
function generateResourceGroupName(environment, application, region) {
  const envPrefix = {
    'dev': 'd',
    'staging': 's',
    'prod': 'p'
  }[environment.toLowerCase()] || 'x';
  
  return `rg-${envPrefix}-${application}-${region}`;
}

/**
 * Validate Azure storage account name
 * @param {string} name - Storage account name to validate
 * @returns {boolean} True if valid
 */
function isValidStorageAccountName(name) {
  // Must be 3-24 chars, lowercase letters and numbers only
  const regex = /^[a-z0-9]{3,24}$/;
  return regex.test(name);
}

/**
 * Get Azure region display name
 * @param {string} regionCode - Region code (e.g., 'eastus')
 * @returns {string} Display name
 */
function getRegionDisplayName(regionCode) {
  const regions = {
    'eastus': 'East US',
    'westus': 'West US',
    'centralus': 'Central US',
    'northeurope': 'North Europe',
    'westeurope': 'West Europe',
    'southeastasia': 'Southeast Asia'
  };
  
  return regions[regionCode.toLowerCase()] || regionCode;
}

module.exports = {
  parseResourceId,
  generateResourceGroupName,
  isValidStorageAccountName,
  getRegionDisplayName
};
```

Create `index.d.ts` (TypeScript definitions):

```typescript
/**
 * Parse Azure resource ID
 */
export function parseResourceId(resourceId: string): {
  subscriptionId: string;
  resourceGroup: string;
  provider: string;
  resourceType: string;
  resourceName: string;
};

/**
 * Generate Azure resource group name
 */
export function generateResourceGroupName(
  environment: string,
  application: string,
  region: string
): string;

/**
 * Validate Azure storage account name
 */
export function isValidStorageAccountName(name: string): boolean;

/**
 * Get Azure region display name
 */
export function getRegionDisplayName(regionCode: string): string;
```

### Step 3: Add Tests

Install Jest:
```powershell
npm install --save-dev jest
```

Create `index.test.js`:

```javascript
const {
  parseResourceId,
  generateResourceGroupName,
  isValidStorageAccountName,
  getRegionDisplayName
} = require('./index');

describe('Azure Utils', () => {
  describe('parseResourceId', () => {
    test('should parse valid resource ID', () => {
      const resourceId = '/subscriptions/12345/resourceGroups/my-rg/providers/Microsoft.Web/sites/my-app';
      const result = parseResourceId(resourceId);
      
      expect(result.subscriptionId).toBe('12345');
      expect(result.resourceGroup).toBe('my-rg');
      expect(result.resourceType).toBe('sites');
      expect(result.resourceName).toBe('my-app');
    });
    
    test('should throw on invalid resource ID', () => {
      expect(() => parseResourceId('invalid')).toThrow();
    });
  });
  
  describe('generateResourceGroupName', () => {
    test('should generate correct name for prod', () => {
      const name = generateResourceGroupName('prod', 'myapp', 'eastus');
      expect(name).toBe('rg-p-myapp-eastus');
    });
    
    test('should handle dev environment', () => {
      const name = generateResourceGroupName('dev', 'testapp', 'westus');
      expect(name).toBe('rg-d-testapp-westus');
    });
  });
  
  describe('isValidStorageAccountName', () => {
    test('should validate correct names', () => {
      expect(isValidStorageAccountName('mystorageaccount')).toBe(true);
      expect(isValidStorageAccountName('storage123')).toBe(true);
    });
    
    test('should reject invalid names', () => {
      expect(isValidStorageAccountName('My-Storage')).toBe(false); // uppercase and dash
      expect(isValidStorageAccountName('ab')).toBe(false); // too short
      expect(isValidStorageAccountName('a'.repeat(25))).toBe(false); // too long
    });
  });
  
  describe('getRegionDisplayName', () => {
    test('should return display name', () => {
      expect(getRegionDisplayName('eastus')).toBe('East US');
      expect(getRegionDisplayName('westeurope')).toBe('West Europe');
    });
    
    test('should return code if unknown', () => {
      expect(getRegionDisplayName('unknown')).toBe('unknown');
    });
  });
});
```

### Step 4: Create Publishing Workflow

Create `.github/workflows/publish-npm.yml`:

```yaml
name: Publish npm Package

on:
  release:
    types: [published]

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          registry-url: 'https://npm.pkg.github.com'
          scope: '@YOUR_USERNAME'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
      
      - name: Update package version
        run: |
          VERSION=${GITHUB_REF#refs/tags/v}
          npm version $VERSION --no-git-tag-version
      
      - name: Publish to GitHub Packages
        run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Create README for package
        run: |
          cat > PACKAGE_README.md << 'EOF'
          # Azure Utils Library
          
          Published version: ${{ github.ref_name }}
          
          ## Installation
          
          ```bash
          npm install @${{ github.repository_owner }}/azure-utils-lib
          ```
          
          ## Usage
          
          ```javascript
          const { parseResourceId } = require('@${{ github.repository_owner }}/azure-utils-lib');
          
          const id = '/subscriptions/123/resourceGroups/my-rg/providers/Microsoft.Web/sites/my-app';
          const parsed = parseResourceId(id);
          console.log(parsed.resourceGroup); // 'my-rg'
          ```
          EOF
          
          cat PACKAGE_README.md
```

**Replace `YOUR_USERNAME`** in both files.

### Step 5: Create README

Create `README.md`:

```markdown
# Azure Utils Library

A collection of utility functions for working with Azure in Node.js applications.

## Installation

### From GitHub Packages

1. Create `.npmrc` in your project:
   ```
   @YOUR_USERNAME:registry=https://npm.pkg.github.com
   ```

2. Authenticate with GitHub:
   ```bash
   npm login --registry=https://npm.pkg.github.com
   ```

3. Install the package:
   ```bash
   npm install @YOUR_USERNAME/azure-utils-lib
   ```

## Usage

```javascript
const azureUtils = require('@YOUR_USERNAME/azure-utils-lib');

// Parse resource ID
const parsed = azureUtils.parseResourceId(
  '/subscriptions/12345/resourceGroups/my-rg/providers/Microsoft.Web/sites/my-app'
);
console.log(parsed.resourceGroup); // 'my-rg'

// Generate resource group name
const rgName = azureUtils.generateResourceGroupName('prod', 'myapp', 'eastus');
console.log(rgName); // 'rg-p-myapp-eastus'

// Validate storage account name
const isValid = azureUtils.isValidStorageAccountName('mystorageaccount');
console.log(isValid); // true

// Get region display name
const displayName = azureUtils.getRegionDisplayName('eastus');
console.log(displayName); // 'East US'
```

## API Reference

See [index.d.ts](./index.d.ts) for TypeScript definitions.

## License

MIT
```

### Step 6: Publish Package

1. **Create `.gitignore`**:
   ```
   node_modules/
   *.log
   .DS_Store
   ```

2. **Commit and push**:
   ```powershell
   git add .
   git commit -m "Initial commit"
   git branch -M main
   
   # Create GitHub repository first, then:
   git remote add origin https://github.com/YOUR_USERNAME/azure-utils-lib.git
   git push -u origin main
   ```

3. **Create a release**:
   ```powershell
   git tag v1.0.0
   git push origin v1.0.0
   
   # Or use GitHub UI or CLI:
   gh release create v1.0.0 --title "v1.0.0" --notes "Initial release"
   ```

4. **Verify publication**:
   - Go to repository → Packages
   - See your package listed
   - Click to view details

### Step 7: Use the Package

In another project:

1. **Create `.npmrc`**:
   ```
   @YOUR_USERNAME:registry=https://npm.pkg.github.com
   //npm.pkg.github.com/:_authToken=${GITHUB_TOKEN}
   ```

2. **Set environment variable**:
   ```powershell
   $env:GITHUB_TOKEN="your_github_token"
   ```

3. **Install and use**:
   ```powershell
   npm install @YOUR_USERNAME/azure-utils-lib
   ```

---

## 🚀 Exercise 2: Publish Docker Image to GHCR

### What You'll Build
A Docker image published to GitHub Container Registry with automated builds on push.

### Step 1: Create Docker Application

1. **Create repository** `azure-app-image`:
   ```powershell
   cd C:\Users\alibengtsson\mycertifications\githubactions
   mkdir azure-app-image
   cd azure-app-image
   git init
   ```

2. **Create simple web app** `app.js`:
   ```javascript
   const express = require('express');
   const app = express();
   const port = process.env.PORT || 3000;
   
   app.get('/', (req, res) => {
     res.json({
       message: 'Hello from Azure App Container!',
       version: process.env.APP_VERSION || '1.0.0',
       timestamp: new Date().toISOString()
     });
   });
   
   app.get('/health', (req, res) => {
     res.json({ status: 'healthy' });
   });
   
   app.listen(port, () => {
     console.log(`Server running on port ${port}`);
   });
   ```

3. **Create `package.json`**:
   ```json
   {
     "name": "azure-app-image",
     "version": "1.0.0",
     "description": "Sample Azure app for container publishing",
     "main": "app.js",
     "scripts": {
       "start": "node app.js"
     },
     "dependencies": {
       "express": "^4.18.2"
     }
   }
   ```

4. **Create `Dockerfile`**:
   ```dockerfile
   FROM node:20-alpine
   
   LABEL org.opencontainers.image.source="https://github.com/YOUR_USERNAME/azure-app-image"
   LABEL org.opencontainers.image.description="Azure demo application"
   LABEL org.opencontainers.image.licenses="MIT"
   
   WORKDIR /app
   
   COPY package*.json ./
   RUN npm ci --only=production
   
   COPY app.js ./
   
   EXPOSE 3000
   
   USER node
   
   CMD ["npm", "start"]
   ```

### Step 2: Create Build and Publish Workflow

Create `.github/workflows/publish-docker.yml`:

```yaml
name: Publish Docker Image

on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Log in to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha,prefix=sha-
      
      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          build-args: |
            APP_VERSION=${{ github.ref_name }}
      
      - name: Generate artifact attestation
        if: github.event_name != 'pull_request'
        uses: actions/attest-build-provenance@v1
        with:
          subject-name: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          subject-digest: ${{ steps.build.outputs.digest }}
          push-to-registry: true
```

### Step 3: Multi-Platform Build

For ARM and AMD64 support:

```yaml
name: Publish Multi-Platform Image

on:
  push:
    tags: ['v*']

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Log in to GHCR
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          platforms: linux/amd64,linux/arm64
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

### Step 4: Publish and Test

1. **Commit and push**:
   ```powershell
   git add .
   git commit -m "Add Docker image with GHCR publishing"
   git branch -M main
   git remote add origin https://github.com/YOUR_USERNAME/azure-app-image.git
   git push -u origin main
   ```

2. **Create release**:
   ```powershell
   git tag v1.0.0
   git push origin v1.0.0
   ```

3. **Make image public**:
   - Go to package settings
   - Change visibility to public
   - Anyone can now pull without authentication

### Step 5: Pull and Run Image Locally

```powershell
# Pull the image
docker pull ghcr.io/YOUR_USERNAME/azure-app-image:v1.0.0

# Run the container
docker run -d -p 3000:3000 --name azure-app ghcr.io/YOUR_USERNAME/azure-app-image:v1.0.0

# Test the app
curl http://localhost:3000

# Check health
curl http://localhost:3000/health

# View logs
docker logs azure-app

# Stop and remove
docker stop azure-app
docker rm azure-app
```

---

## 🚀 Exercise 3: Publish to Azure Container Registry

### What You'll Build
Publish the same image to both GHCR and Azure Container Registry (ACR).

### Step 1: Create Azure Container Registry

```powershell
# Login to Azure
az login

# Create resource group
az group create --name rg-container-registry --location eastus

# Create ACR
az acr create --resource-group rg-container-registry --name myuniqueacr123 --sku Basic

# Get ACR login server
az acr show --name myuniqueacr123 --query loginServer -o tsv
# Output: myuniqueacr123.azurecr.io
```

### Step 2: Create Service Principal

```powershell
# Get ACR resource ID
$ACR_ID = az acr show --name myuniqueacr123 --query id -o tsv

# Create service principal with push permissions
az ad sp create-for-rbac --name sp-acr-push --role acrpush --scopes $ACR_ID --sdk-auth
```

Save the output JSON as `AZURE_ACR_CREDENTIALS` secret in GitHub.

### Step 3: Create Multi-Registry Publishing Workflow

Create `.github/workflows/publish-multi-registry.yml`:

```yaml
name: Publish to Multiple Registries

on:
  push:
    tags: ['v*']

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Log in to GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Log in to Azure Container Registry
        uses: azure/docker-login@v1
        with:
          login-server: ${{ secrets.ACR_LOGIN_SERVER }}
          username: ${{ fromJson(secrets.AZURE_ACR_CREDENTIALS).clientId }}
          password: ${{ fromJson(secrets.AZURE_ACR_CREDENTIALS).clientSecret }}
      
      - name: Extract version
        id: version
        run: echo "version=${GITHUB_REF#refs/tags/v}" >> $GITHUB_OUTPUT
      
      - name: Build and push to GHCR
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ghcr.io/${{ github.repository }}:${{ steps.version.outputs.version }}
            ghcr.io/${{ github.repository }}:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max
      
      - name: Build and push to ACR
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ${{ secrets.ACR_LOGIN_SERVER }}/azure-app:${{ steps.version.outputs.version }}
            ${{ secrets.ACR_LOGIN_SERVER }}/azure-app:latest
```

Add secrets to GitHub:
- `ACR_LOGIN_SERVER`: Your ACR URL (e.g., `myuniqueacr123.azurecr.io`)
- `AZURE_ACR_CREDENTIALS`: Service principal JSON

### Step 4: Pull from ACR

```powershell
# Login to ACR
az acr login --name myuniqueacr123

# Pull image
docker pull myuniqueacr123.azurecr.io/azure-app:latest

# Run image
docker run -d -p 3000:3000 myuniqueacr123.azurecr.io/azure-app:latest
```

---

## 🚀 Exercise 4: Package Management Best Practices

### Semantic Versioning

Use semantic versioning for packages:

```yaml
name: Semantic Release

on:
  push:
    branches: [main]

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: Determine version bump
        id: version
        run: |
          # Get latest tag
          LATEST_TAG=$(git describe --tags --abbrev=0 2>/dev/null || echo "v0.0.0")
          
          # Check commit messages since last tag
          COMMITS=$(git log $LATEST_TAG..HEAD --pretty=format:"%s")
          
          if echo "$COMMITS" | grep -q "BREAKING CHANGE"; then
            BUMP="major"
          elif echo "$COMMITS" | grep -q "^feat"; then
            BUMP="minor"
          else
            BUMP="patch"
          fi
          
          echo "bump=$BUMP" >> $GITHUB_OUTPUT
```

### Package Retention Policy

Clean up old package versions:

```yaml
name: Cleanup Old Packages

on:
  schedule:
    - cron: '0 2 * * 0'  # Weekly
  workflow_dispatch:

jobs:
  cleanup:
    runs-on: ubuntu-latest
    steps:
      - name: Delete old package versions
        uses: actions/delete-package-versions@v4
        with:
          package-name: 'azure-app-image'
          package-type: 'container'
          min-versions-to-keep: 5
          delete-only-untagged-versions: true
```

### Package Documentation

Create package landing page:

```yaml
name: Update Package README

on:
  release:
    types: [published]

jobs:
  update-readme:
    runs-on: ubuntu-latest
    permissions:
      packages: write
    
    steps:
      - name: Update container description
        run: |
          echo '# Azure App Image
          
          ## Quick Start
          ```bash
          docker pull ghcr.io/${{ github.repository }}:latest
          docker run -p 3000:3000 ghcr.io/${{ github.repository }}:latest
          ```
          
          ## Tags
          - `latest`: Most recent build
          - `v1.0.0`: Specific version
          - `sha-abc123`: Commit-specific build
          ' > README.md
```

---

## 📚 Best Practices

### 1. Always Use Version Tags

```yaml
tags: |
  type=semver,pattern={{version}}
  type=semver,pattern={{major}}.{{minor}}
  type=sha
```

### 2. Set Image Labels

```dockerfile
LABEL org.opencontainers.image.source="https://github.com/user/repo"
LABEL org.opencontainers.image.description="Description"
LABEL org.opencontainers.image.licenses="MIT"
```

### 3. Use Multi-Stage Builds

```dockerfile
# Build stage
FROM node:20 AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .

# Production stage
FROM node:20-alpine
WORKDIR /app
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/app.js ./
CMD ["node", "app.js"]
```

### 4. Enable Package Security

- Enable Dependabot for packages
- Scan images for vulnerabilities
- Use minimal base images
- Run as non-root user

### 5. Cache Layers Efficiently

```yaml
- uses: docker/build-push-action@v5
  with:
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

---

## 🔍 Troubleshooting

### Issue: Authentication Failed

**For npm:**
```powershell
# Create personal access token with packages:read and packages:write
# Add to .npmrc:
//npm.pkg.github.com/:_authToken=YOUR_TOKEN
```

**For Docker:**
```powershell
echo $GITHUB_TOKEN | docker login ghcr.io -u YOUR_USERNAME --password-stdin
```

### Issue: Package Not Visible

- Check package visibility (public vs private)
- Verify permissions in repository settings
- Ensure GITHUB_TOKEN has `packages: write`

### Issue: ACR Push Failed

```powershell
# Verify service principal has acrpush role
az role assignment list --assignee SP_APP_ID --scope $ACR_ID
```

---

## ✅ Knowledge Check

### Question 1
What's the registry URL for GitHub Packages npm?

<details>
<summary>Answer</summary>

`https://npm.pkg.github.com`

Configure in package.json:
```json
"publishConfig": {
  "registry": "https://npm.pkg.github.com"
}
```
</details>

### Question 2
How do you make a GHCR image public?

<details>
<summary>Answer</summary>

1. Go to the package page
2. Click "Package settings"
3. Scroll to "Danger Zone"
4. Click "Change visibility"
5. Select "Public"
</details>

### Question 3
What permissions are needed to publish packages?

<details>
<summary>Answer</summary>

```yaml
permissions:
  contents: read    # Read repository
  packages: write   # Publish packages
```
</details>

---

## 🎯 Challenge Exercises

### Challenge 1: Version Automation
Create a workflow that automatically bumps version based on commit messages (conventional commits).

### Challenge 2: Package Mirroring
Create a workflow that mirrors your GHCR images to Docker Hub.

### Challenge 3: License Scanning
Add a step that scans package dependencies for license compliance.

---

## 📖 Additional Resources

- [GitHub Packages Documentation](https://docs.github.com/en/packages)
- [GHCR Documentation](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
- [Docker Build Push Action](https://github.com/docker/build-push-action)
- [Azure Container Registry](https://docs.microsoft.com/azure/container-registry/)

---

## 📝 Summary

In this lab, you learned:
- ✅ Publishing npm packages to GitHub Packages
- ✅ Building and publishing Docker images to GHCR
- ✅ Multi-platform container builds
- ✅ Publishing to Azure Container Registry
- ✅ Version tagging strategies
- ✅ Package management best practices
- ✅ Using published packages in projects

**Next Steps:**
- Complete Lab 9: Reusable Workflows & Artifacts
- Publish your own packages
- Integrate package publishing into CI/CD

---

**Estimated Time to Complete:** 60 minutes  
**Difficulty:** Intermediate

Congratulations! You can now publish and manage packages professionally! 📦
