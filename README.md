# Container Build and Push Action

This composite GitHub Action builds and pushes a container image to GitHub Container Registry with automatic versioning support. It also optionally supports deployment to Railway.

## Features

- Automatic version incrementing (patch/minor/major)
- GitHub Container Registry integration
- Docker image metadata and labels
- Optional Railway deployment
- Configurable build context and Dockerfile path

## Usage

```yaml
name: Build and Deploy Container

on:
  push:
    branches: [ main, develop ]
  workflow_dispatch:
    inputs:
      version_increment:
        description: 'Version increment type'
        required: false
        type: choice
        options:
          - patch
          - minor
          - major

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Build and Push Container
        uses: xngen-ai/container-build-push@v1
        with:
          image_name: my-app
          image_title: My Application
          image_description: Container image for My Application
          # Optional parameters
          registry: ghcr.io
          org_name: my-org
          version_increment: ${{ github.event.inputs.version_increment }}
          dockerfile_path: ./Dockerfile
          context_path: .
          # Railway deployment (optional)
          railway_token: ${{ secrets.RAILWAY_BEARER }}
          railway_service_id: ${{ secrets.RW_SERVICE_ID }}
          railway_environment_id: ${{ secrets.RW_ENVIRONMENT_ID }}
```

## Inputs

### Required

- `image_name`: Name of the container image
- `image_title`: Title of the container image
- `image_description`: Description of the container image

### Optional

- `registry`: Container registry to push to (default: 'ghcr.io')
- `org_name`: Organization name for the container registry (default: 'xngen-ai')
- `version_increment`: Version increment type (patch/minor/major) (default: 'patch')
- `dockerfile_path`: Path to the Dockerfile (default: './Dockerfile')
- `context_path`: Build context path (default: '.')
- `railway_token`: Railway deployment token (optional)
- `railway_service_id`: Railway service ID (optional)
- `railway_environment_id`: Railway environment ID (optional)

## Secrets

The following secrets should be configured in your repository:

- `GITHUB_TOKEN`: Automatically provided by GitHub, used for container registry authentication
- `RAILWAY_BEARER`: (Optional) Railway deployment token
- `RW_SERVICE_ID`: (Optional) Railway service ID
- `RW_ENVIRONMENT_ID`: (Optional) Railway environment ID

## Complete Example

Here's a complete example of how to use this action in your repository:

### Repository Structure
```
your-app/
├── .github/
│   └── workflows/
│       └── container-build.yml
├── src/
│   └── ... (your application code)
├── Dockerfile
└── README.md
```

### Workflow File (.github/workflows/container-build.yml)
```yaml
name: Build and Deploy Container

on:
  push:
    branches: [ main, develop ]
  workflow_dispatch:
    inputs:
      version_increment:
        description: 'Version increment type'
        required: false
        type: choice
        options:
          - patch
          - minor
          - major

permissions:
  contents: read
  packages: write

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Build and Push Container
        uses: xngen-ai/container-build-push@v1
        with:
          # Required inputs
          image_name: my-backend-app
          image_title: My Backend Application
          image_description: Backend service for My Application
          
          # Optional inputs with custom values
          registry: ghcr.io
          org_name: my-organization
          version_increment: ${{ github.event.inputs.version_increment }}
          dockerfile_path: ./Dockerfile
          context_path: .
          
          # Railway deployment configuration (optional)
          railway_token: ${{ secrets.RAILWAY_BEARER }}
          railway_service_id: ${{ secrets.RW_SERVICE_ID }}
          railway_environment_id: ${{ secrets.RW_ENVIRONMENT_ID }}
```

### Repository Secrets Configuration

In your repository settings, navigate to Settings > Secrets and variables > Actions and add these secrets:

```
RAILWAY_BEARER         = your-railway-token
RW_SERVICE_ID         = your-railway-service-id
RW_ENVIRONMENT_ID     = your-railway-environment-id
```

Note: `GITHUB_TOKEN` is automatically provided by GitHub Actions.

### Dockerfile Example
```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000
CMD ["npm", "start"]
```

## Notes

- The action automatically handles GitHub Container Registry authentication using the `GITHUB_TOKEN`
- Railway deployment is optional and will only be executed if all Railway-related inputs are provided
- Version incrementing follows semantic versioning (MAJOR.MINOR.PATCH)
- The workflow can be triggered either by pushing to main/develop branches or manually through the GitHub Actions UI
- The container image will be available at: `ghcr.io/my-organization/my-backend-app:v1.0.0`
