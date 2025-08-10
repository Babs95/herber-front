# CI/CD Pipeline Setup Guide

This document provides a comprehensive guide to set up and configure the CI/CD pipeline for the **herber-front** Angular application using GitHub Actions and Docker Hub.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [GitHub Actions Workflow](#github-actions-workflow)
- [Docker Configuration](#docker-configuration)
- [Setup Instructions](#setup-instructions)
- [Pipeline Triggers](#pipeline-triggers)
- [Monitoring and Troubleshooting](#monitoring-and-troubleshooting)
- [Security Best Practices](#security-best-practices)
- [Customization Options](#customization-options)

## Overview

This CI/CD pipeline automates the following processes:

1. **Continuous Integration (CI)**:
   - Code quality checks (linting)
   - Unit testing
   - Build verification
   - Multi-platform Docker image building

2. **Continuous Deployment (CD)**:
   - Automated Docker image publishing to Docker Hub
   - Multi-architecture support (AMD64, ARM64)
   - Automated tagging strategy

## Prerequisites

Before setting up this CI/CD pipeline, ensure you have:

### Required Accounts
- [GitHub](https://github.com) account with repository access
- [Docker Hub](https://hub.docker.com) account

### Required Tools (for local testing)
- Node.js 18 or higher
- npm or yarn package manager
- Docker Desktop
- Angular CLI (`npm install -g @angular/cli`)

### Repository Requirements
- Angular project with standard structure
- `package.json` with required scripts:
  - `npm run build`
  - `npm run test`
  - `npm run lint`

## Project Structure

The CI/CD setup includes the following files:

```
herber-front/
├── .github/
│   └── workflows/
│       └── ci-cd.yml           # GitHub Actions workflow
├── .dockerignore               # Docker ignore file
├── Dockerfile                  # Multi-stage Docker build
├── package.json               # Node.js dependencies
├── angular.json              # Angular CLI configuration
└── CI-CD-SETUP.md           # This documentation
```

## GitHub Actions Workflow

### Workflow File Location
`.github/workflows/ci-cd.yml`

### Jobs Overview

#### 1. Test Job (`test`)
- **Triggers**: All pushes and pull requests
- **Purpose**: Quality assurance and build verification
- **Steps**:
  - Checkout code
  - Setup Node.js environment
  - Install dependencies
  - Run linting checks
  - Execute unit tests
  - Build application

#### 2. Build and Push Job (`build-and-push`)
- **Triggers**: Only on pushes to `main` branch
- **Purpose**: Build and deploy Docker images
- **Steps**:
  - Checkout code
  - Setup Docker Buildx
  - Login to Docker Hub
  - Extract metadata for tagging
  - Build and push multi-platform images

### Environment Variables

The workflow uses these environment variables:

```yaml
env:
  DOCKER_IMAGE_NAME: herber-front  # Docker image name
  NODE_VERSION: '18'               # Node.js version
```

## Docker Configuration

### Dockerfile Explanation

The `Dockerfile` uses a multi-stage build approach:

#### Stage 1: Build (node:18-alpine)
- Installs Node.js dependencies
- Builds the Angular application for production
- Creates optimized, minified assets

#### Stage 2: Production (nginx:alpine)
- Uses lightweight Nginx server
- Copies built application from build stage
- Exposes port 80
- Serves static files efficiently

### .dockerignore File
Excludes unnecessary files from the Docker build context:
- `node_modules/`
- Development files
- Git repository
- Log files
- IDE configurations

## Setup Instructions

### Step 1: Repository Setup

1. Fork or clone this repository
2. Ensure all required files are present:
   - `.github/workflows/ci-cd.yml`
   - `Dockerfile`
   - `.dockerignore`

### Step 2: Docker Hub Configuration

1. **Create Docker Hub Repository**:
   - Log in to [Docker Hub](https://hub.docker.com)
   - Create a new repository named `herber-front` (or your preferred name)
   - Set visibility (public/private) as needed

2. **Generate Access Token**:
   - Go to Docker Hub → Account Settings → Security
   - Click "New Access Token"
   - Name: `github-actions-herber-front`
   - Permissions: Read, Write, Delete
   - Click "Generate"
   - **Copy the token immediately** (you won't be able to see it again)
   - Keep this tab open until you've added it to GitHub

### Step 3: GitHub Secrets Configuration

Add the following secrets to your GitHub repository:

1. **Navigate to Repository Settings**:
   - Go to your GitHub repository
   - Click "Settings" tab (at the top of the repository page)
   - In the left sidebar, click "Secrets and variables"
   - Click "Actions"

2. **Add DOCKER_USERNAME Secret**:
   - Click "New repository secret"
   - In the "Name" field, enter: `DOCKER_USERNAME`
   - In the "Secret" field, enter your Docker Hub username
   - Click "Add secret"

3. **Add DOCKER_PASSWORD Secret**:
   - Click "New repository secret" again
   - In the "Name" field, enter: `DOCKER_PASSWORD`
   - In the "Secret" field, **paste the Docker Hub access token you copied**
   - Click "Add secret"

4. **Visual Reference**:
   ```
   GitHub Repository → Settings → Secrets and variables → Actions
   
   ┌─────────────────────────────────────┐
   │ Name: DOCKER_PASSWORD               │
   ├─────────────────────────────────────┤
   │ Secret: [PASTE YOUR TOKEN HERE]     │
   │                                     │
   │ dckr_pat_abc123xyz789...           │
   └─────────────────────────────────────┘
           [Add secret] button
   ```

5. **Verification**:
   - After adding both secrets, you should see them listed under "Repository secrets"
   - The values will be hidden for security
   - Both `DOCKER_USERNAME` and `DOCKER_PASSWORD` should appear in the list

   | Secret Name | Description | Example Value |
   |-------------|-------------|---------------|
   | `DOCKER_USERNAME` | Your Docker Hub username | `yourusername` |
   | `DOCKER_PASSWORD` | Docker Hub access token | `dckr_pat_abc123...` |

> **Important Notes:**
> - Don't close the Docker Hub page until you've successfully pasted the token
> - The token will look like: `dckr_pat_xxxxxxxxxxxxxxxxx`
> - Once you click "Add secret", the token will be hidden and you won't be able to see it again
> - If you lose the token, you'll need to generate a new one from Docker Hub

### Step 4: Verify Package.json Scripts

Ensure your `package.json` contains these scripts:

```json
{
  "scripts": {
    "build": "ng build",
    "test": "ng test",
    "lint": "ng lint"
  }
}
```

### Step 5: Test the Pipeline

1. **Commit and Push**:
   ```bash
   git add .
   git commit -m "Add CI/CD pipeline configuration"
   git push origin main
   ```

2. **Monitor Execution**:
   - Go to GitHub repository → "Actions" tab
   - Watch the workflow execution
   - Check for any errors

## Pipeline Triggers

### Automatic Triggers

| Event | Branches | Jobs Executed |
|-------|----------|---------------|
| Push | `main`, `develop` | `test` + `build-and-push` (main only) |
| Pull Request | `main` | `test` only |

### Manual Triggers
- Navigate to Actions tab
- Select workflow
- Click "Run workflow"

## Monitoring and Troubleshooting

### Viewing Workflow Results

1. **GitHub Actions Tab**:
   - Repository → Actions
   - Click on workflow run
   - Expand job steps to see detailed logs

2. **Docker Hub Verification**:
   - Check Docker Hub repository for new images
   - Verify tags and timestamps

### Common Issues and Solutions

#### 1. Build Failures

**Symptom**: Test job fails during build step
**Solutions**:
- Check Angular version compatibility
- Verify `angular.json` configuration
- Ensure all dependencies are in `package.json`

#### 2. Docker Hub Authentication Errors

**Symptom**: "Login failed" or "Access denied"
**Solutions**:
- Verify `DOCKER_USERNAME` matches exactly
- Regenerate Docker Hub access token
- Check token permissions (Read, Write, Delete)
- Ensure secrets are added to correct repository

#### 3. Test Failures

**Symptom**: Tests fail in CI environment
**Solutions**:
- Run tests locally: `npm run test -- --no-watch --browsers=ChromeHeadless`
- Check for environment-specific dependencies
- Verify test configuration in `angular.json`

#### 4. Linting Errors

**Symptom**: Lint job fails
**Solutions**:
- Run locally: `npm run lint`
- Fix linting issues or update ESLint configuration
- Ensure consistent code formatting

## Security Best Practices

### 1. Secrets Management
- Never commit secrets to repository
- Use GitHub repository secrets only
- Regularly rotate Docker Hub tokens
- Use least-privilege access tokens

### 2. Docker Security
- Use official, minimal base images
- Regularly update base images
- Scan images for vulnerabilities
- Use multi-stage builds to reduce attack surface

### 3. Workflow Security
- Pin action versions (e.g., `@v4` instead of `@latest`)
- Review third-party actions before use
- Limit workflow permissions
- Use branch protection rules

## Customization Options

### 1. Modify Docker Image Name

Update the environment variable in `.github/workflows/ci-cd.yml`:

```yaml
env:
  DOCKER_IMAGE_NAME: your-custom-name
```

### 2. Add Additional Environments

Create separate workflows for staging/development:

```yaml
# .github/workflows/staging.yml
on:
  push:
    branches: [ develop ]
```

### 3. Custom Build Arguments

Add build arguments to Dockerfile:

```dockerfile
ARG BUILD_CONFIGURATION=production
RUN npm run build -- --configuration=$BUILD_CONFIGURATION
```

### 4. Additional Testing

Add end-to-end tests:

```yaml
- name: Run e2e tests
  run: npm run e2e -- --headless
```

### 5. Slack/Email Notifications

Add notification step:

```yaml
- name: Notify Slack
  if: failure()
  uses: 8398a7/action-slack@v3
  with:
    status: failure
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

### 6. Code Coverage

Add coverage reporting:

```yaml
- name: Generate coverage report
  run: npm run test -- --code-coverage --no-watch --browsers=ChromeHeadless

- name: Upload coverage to Codecov
  uses: codecov/codecov-action@v3
```

### 7. Multi-Environment Docker Tags

Customize tagging strategy in workflow:

```yaml
tags: |
  type=ref,event=branch
  type=semver,pattern={{version}}
  type=semver,pattern={{major}}.{{minor}}
  type=raw,value=latest,enable={{is_default_branch}}
```

## Advanced Configuration

### Environment-Specific Builds

For different environments, create separate Docker images:

```yaml
matrix:
  environment: [staging, production]
steps:
  - name: Build for ${{ matrix.environment }}
    run: npm run build -- --configuration=${{ matrix.environment }}
```

### Parallel Testing

Speed up CI with parallel test execution:

```yaml
strategy:
  matrix:
    node-version: [16, 18, 20]
```

### Conditional Deployments

Deploy only on specific conditions:

```yaml
if: github.ref == 'refs/heads/main' && !contains(github.event.head_commit.message, '[skip-deploy]')
```

## Support and Maintenance

### Regular Maintenance Tasks

1. **Monthly**:
   - Update base Docker images
   - Review and update GitHub Actions versions
   - Check for security vulnerabilities

2. **Quarterly**:
   - Review and update Node.js version
   - Audit dependencies for security issues
   - Review workflow performance metrics

### Getting Help

- **GitHub Actions Issues**: Check [GitHub Actions documentation](https://docs.github.com/en/actions)
- **Docker Issues**: Refer to [Docker documentation](https://docs.docker.com/)
- **Angular Issues**: Visit [Angular documentation](https://angular.dev/)

---

## Conclusion

This CI/CD pipeline provides a robust, automated solution for building and deploying Angular applications. It ensures code quality through automated testing and linting, while providing reliable deployment to Docker Hub with multi-platform support.

For questions or issues, please refer to the troubleshooting section or create an issue in the repository.

---

*Last updated: August 2025*