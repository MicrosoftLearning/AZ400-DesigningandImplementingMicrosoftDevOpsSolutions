---
lab:
    title: "Configure GitHub Actions as Code with YAML"
    module: "Module 02: Implement CI with Azure Pipelines and GitHub Actions"
---

# Configure GitHub Actions as Code with YAML

## Lab requirements

- This lab requires **Microsoft Edge** or a [GitHub supported browser](https://docs.github.com/en/get-started/using-github/supported-browsers).

- **GitHub Account:** If you don't already have a GitHub account that you can use for this lab, follow instructions available at [Signing up for a new GitHub account](https://github.com/join) to create one.

- Identify an existing Azure subscription or create a new one.

- Verify that you have a Microsoft account or a Microsoft Entra account with the Contributor or the Owner role in the Azure subscription. For details, refer to [List Azure role assignments using the Azure portal](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) and [View and assign administrator roles in Azure Active Directory](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Lab overview

Many teams prefer to define their CI/CD pipelines using YAML workflows that can be version-controlled alongside their code. GitHub Actions provides a powerful platform for implementing pipelines as code, allowing teams to define complex multi-stage workflows with approvals, environments, and sophisticated deployment strategies. This approach enables better collaboration, version control of pipeline definitions, and the ability to apply software engineering practices to infrastructure and deployment processes.

This lab focuses on advanced GitHub Actions concepts and serves as a GitHub Actions equivalent to the Azure DevOps "Configure Pipelines as Code with YAML" lab. It builds upon the basic GitHub Actions concepts covered in the "Implement GitHub Actions for CI/CD" lab (M02_L05) by introducing more sophisticated workflow patterns, reusable components, and enterprise-grade features.

## Objectives

After you complete this lab, you will be able to:

- Configure advanced CI/CD workflows as code with GitHub Actions
- Implement multi-stage workflows with environments and approvals
- Use workflow templates and reusable components
- Apply security best practices for GitHub Actions workflows

## Estimated timing: 50 minutes

## Instructions

### Exercise 0: Configure the lab prerequisites

In this exercise, you will set up the prerequisites for the lab.

#### Task 1: Create and configure the GitHub repository

In this task, you will create a **eShopOnWeb-Actions** GitHub repository to be used for this lab.

1. Open a web browser, navigate to the [GitHub website](https://github.com/), and sign in using your account.
1. Click on **New** to create a new repository.
1. On the **Create a new repository** page, perform the following actions:
   - Repository name: **eShopOnWeb-Actions**
   - Description: **Advanced GitHub Actions workflows lab**
   - Visibility: **Public** (or Private if you prefer)
   - Initialize with: Check **Add a README file**
   - Leave other options as default
1. Click **Create repository**.

#### Task 2: Import eShopOnWeb source code

In this task, you will import the eShopOnWeb source code to your repository.

1. In your newly created repository, click on the **Code** tab if not already selected.
1. Click on **uploading an existing file** link or go to **Add file > Upload files**.
1. Open a new browser tab and navigate to the [eShopOnWeb repository](https://github.com/MicrosoftLearning/eShopOnWeb).
1. Click on **Code** and then **Download ZIP**.
1. Extract the downloaded ZIP file to your local machine.
1. Go back to your repository tab and upload the extracted files by:
   - Dragging and dropping the contents of the extracted folder (excluding the main folder itself)
   - Or clicking **choose your files** and selecting the files
1. Add a commit message: **Initial import of eShopOnWeb source code**
1. Click **Commit changes**.

### Exercise 1: Configure Advanced GitHub Actions Workflows as Code

In this exercise, you will create sophisticated GitHub Actions workflows that demonstrate pipelines as code principles.

#### Task 1: Create a multi-stage workflow template

In this task, you will create a comprehensive workflow that includes build, test, and deployment stages.

1. In your **eShopOnWeb-Actions** repository, navigate to the **Actions** tab.
1. Click **New workflow**.
1. Click **set up a workflow yourself** to create a custom workflow.
1. Replace the default content with the following advanced workflow definition:

```yaml
name: eShopOnWeb Advanced CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        default: 'development'
        type: choice
        options:
        - development
        - staging
        - production
      deploy_infrastructure:
        description: 'Deploy infrastructure'
        required: false
        default: false
        type: boolean

env:
  AZURE_WEBAPP_NAME: 'eshop-actions-${{ github.run_number }}'
  AZURE_WEBAPP_PACKAGE_PATH: '.'
  DOTNET_VERSION: '8.x'
  RESOURCE_GROUP: 'rg-eshop-actions'

jobs:
  validate:
    name: Validate and Analyze
    runs-on: ubuntu-latest
    outputs:
      should-deploy: ${{ steps.changes.outputs.should-deploy }}
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      with:
        fetch-depth: 0
    
    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: ${{ env.DOTNET_VERSION }}
    
    - name: Cache dependencies
      uses: actions/cache@v3
      with:
        path: ~/.nuget/packages
        key: ${{ runner.os }}-nuget-${{ hashFiles('**/*.csproj') }}
        restore-keys: |
          ${{ runner.os }}-nuget-
    
    - name: Restore dependencies
      run: dotnet restore
    
    - name: Lint and format check
      run: |
        dotnet format --verify-no-changes --verbosity diagnostic
    
    - name: Security scan
      run: |
        dotnet list package --vulnerable
    
    - name: Check for deployment changes
      id: changes
      run: |
        if git diff --name-only HEAD~1 | grep -E "(src/|infra/|\.github/workflows/)" > /dev/null; then
          echo "should-deploy=true" >> $GITHUB_OUTPUT
        else
          echo "should-deploy=false" >> $GITHUB_OUTPUT
        fi

  build:
    name: Build and Test
    runs-on: ubuntu-latest
    needs: validate
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: ${{ env.DOTNET_VERSION }}
    
    - name: Cache dependencies
      uses: actions/cache@v3
      with:
        path: ~/.nuget/packages
        key: ${{ runner.os }}-nuget-${{ hashFiles('**/*.csproj') }}
        restore-keys: |
          ${{ runner.os }}-nuget-
    
    - name: Restore dependencies
      run: dotnet restore
    
    - name: Build application
      run: dotnet build --configuration Release --no-restore
    
    - name: Run unit tests
      run: |
        dotnet test tests/UnitTests/ \
          --configuration Release \
          --no-build \
          --logger trx \
          --results-directory TestResults/ \
          --collect:"XPlat Code Coverage"
    
    - name: Publish test results
      uses: dorny/test-reporter@v1
      if: always()
      with:
        name: Unit Test Results
        path: TestResults/*.trx
        reporter: dotnet-trx
    
    - name: Upload coverage reports
      uses: codecov/codecov-action@v3
      with:
        directory: ./TestResults/
        flags: unittests
        name: codecov-umbrella
    
    - name: Publish application
      run: |
        dotnet publish src/Web/Web.csproj \
          --configuration Release \
          --output ./publish \
          --no-build
    
    - name: Upload build artifacts
      uses: actions/upload-artifact@v4
      with:
        name: webapp-build-${{ github.run_number }}
        path: ./publish/
        retention-days: 30
    
    - name: Upload infrastructure templates
      uses: actions/upload-artifact@v4
      with:
        name: infrastructure-templates
        path: infra/
        retention-days: 30

  security-scan:
    name: Security Analysis
    runs-on: ubuntu-latest
    needs: build
    permissions:
      security-events: write
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Initialize CodeQL
      uses: github/codeql-action/init@v2
      with:
        languages: csharp
    
    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: ${{ env.DOTNET_VERSION }}
    
    - name: Build for analysis
      run: dotnet build --configuration Release
    
    - name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v2
    
    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        scan-type: 'fs'
        scan-ref: '.'
        format: 'sarif'
        output: 'trivy-results.sarif'
    
    - name: Upload Trivy scan results
      uses: github/codeql-action/upload-sarif@v2
      if: always()
      with:
        sarif_file: 'trivy-results.sarif'

  deploy-development:
    name: Deploy to Development
    runs-on: ubuntu-latest
    needs: [validate, build, security-scan]
    if: needs.validate.outputs.should-deploy == 'true' && (github.ref == 'refs/heads/develop' || github.event_name == 'workflow_dispatch')
    environment: 
      name: development
      url: https://${{ env.AZURE_WEBAPP_NAME }}-dev.azurewebsites.net
    
    steps:
    - name: Download build artifacts
      uses: actions/download-artifact@v4
      with:
        name: webapp-build-${{ github.run_number }}
        path: ./publish/
    
    - name: Login to Azure
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    
    - name: Deploy infrastructure
      if: github.event.inputs.deploy_infrastructure == 'true'
      uses: azure/arm-deploy@v1
      with:
        subscriptionId: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
        resourceGroupName: ${{ env.RESOURCE_GROUP }}
        template: infra/webapp.bicep
        parameters: webAppName=${{ env.AZURE_WEBAPP_NAME }}-dev
    
    - name: Deploy to Azure Web App
      uses: azure/webapps-deploy@v2
      with:
        app-name: ${{ env.AZURE_WEBAPP_NAME }}-dev
        package: ./publish/

  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: [validate, build, security-scan, deploy-development]
    if: needs.validate.outputs.should-deploy == 'true' && github.ref == 'refs/heads/main'
    environment: 
      name: staging
      url: https://${{ env.AZURE_WEBAPP_NAME }}-staging.azurewebsites.net
    
    steps:
    - name: Download build artifacts
      uses: actions/download-artifact@v4
      with:
        name: webapp-build-${{ github.run_number }}
        path: ./publish/
    
    - name: Login to Azure
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    
    - name: Deploy to Azure Web App
      uses: azure/webapps-deploy@v2
      with:
        app-name: ${{ env.AZURE_WEBAPP_NAME }}-staging
        package: ./publish/
    
    - name: Run integration tests
      run: |
        # Add integration test commands here
        echo "Running integration tests against staging environment..."
        # curl -f https://${{ env.AZURE_WEBAPP_NAME }}-staging.azurewebsites.net/health

  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: [validate, build, security-scan, deploy-staging]
    if: needs.validate.outputs.should-deploy == 'true' && github.ref == 'refs/heads/main'
    environment: 
      name: production
      url: https://${{ env.AZURE_WEBAPP_NAME }}.azurewebsites.net
    
    steps:
    - name: Download build artifacts
      uses: actions/download-artifact@v4
      with:
        name: webapp-build-${{ github.run_number }}
        path: ./publish/
    
    - name: Login to Azure
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    
    - name: Deploy to Azure Web App
      uses: azure/webapps-deploy@v2
      with:
        app-name: ${{ env.AZURE_WEBAPP_NAME }}
        package: ./publish/
    
    - name: Post-deployment verification
      run: |
        echo "Verifying production deployment..."
        # Add verification steps here
```

5. Name the file `advanced-pipeline.yml`
6. Click **Commit changes...** and then **Commit changes** to save the workflow.

#### Task 2: Create reusable workflow components

In this task, you will create reusable workflow components to promote code reuse and consistency across pipelines.

1. In your repository, navigate to **.github/workflows/** directory.
1. Click **Add file > Create new file**.
1. Name the file `build-template.yml` and add the following reusable workflow:

```yaml
name: Reusable Build Template

on:
  workflow_call:
    inputs:
      dotnet-version:
        required: false
        type: string
        default: '8.x'
      configuration:
        required: false
        type: string
        default: 'Release'
      run-tests:
        required: false
        type: boolean
        default: true
    outputs:
      artifact-name:
        description: "Name of the build artifact"
        value: ${{ jobs.build.outputs.artifact-name }}
      test-results:
        description: "Test execution results"
        value: ${{ jobs.build.outputs.test-results }}

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      artifact-name: ${{ steps.artifact.outputs.name }}
      test-results: ${{ steps.test.outputs.results }}
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
    
    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: ${{ inputs.dotnet-version }}
    
    - name: Cache dependencies
      uses: actions/cache@v3
      with:
        path: ~/.nuget/packages
        key: ${{ runner.os }}-nuget-${{ hashFiles('**/*.csproj') }}
        restore-keys: |
          ${{ runner.os }}-nuget-
    
    - name: Restore dependencies
      run: dotnet restore
    
    - name: Build application
      run: dotnet build --configuration ${{ inputs.configuration }} --no-restore
    
    - name: Run tests
      if: inputs.run-tests
      id: test
      run: |
        dotnet test --configuration ${{ inputs.configuration }} --no-build --logger trx --results-directory TestResults/
        echo "results=success" >> $GITHUB_OUTPUT
    
    - name: Publish application
      run: |
        dotnet publish src/Web/Web.csproj \
          --configuration ${{ inputs.configuration }} \
          --output ./publish \
          --no-build
    
    - name: Create artifact
      id: artifact
      run: |
        artifact_name="webapp-build-${{ github.run_number }}"
        echo "name=$artifact_name" >> $GITHUB_OUTPUT
    
    - name: Upload build artifacts
      uses: actions/upload-artifact@v4
      with:
        name: ${{ steps.artifact.outputs.name }}
        path: ./publish/
        retention-days: 30
```

4. Click **Commit changes...** and then **Commit changes**.

#### Task 3: Create a deployment template

1. Create another file named `deploy-template.yml`:

```yaml
name: Reusable Deployment Template

on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
      app-name:
        required: true
        type: string
      artifact-name:
        required: true
        type: string
      deploy-infrastructure:
        required: false
        type: boolean
        default: false
    secrets:
      AZURE_CREDENTIALS:
        required: true
      AZURE_SUBSCRIPTION_ID:
        required: false

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: 
      name: ${{ inputs.environment }}
      url: https://${{ inputs.app-name }}.azurewebsites.net
    
    steps:
    - name: Download build artifacts
      uses: actions/download-artifact@v4
      with:
        name: ${{ inputs.artifact-name }}
        path: ./publish/
    
    - name: Login to Azure
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    
    - name: Deploy infrastructure
      if: inputs.deploy-infrastructure
      uses: azure/arm-deploy@v1
      with:
        subscriptionId: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
        resourceGroupName: rg-eshop-actions
        template: infra/webapp.bicep
        parameters: webAppName=${{ inputs.app-name }}
    
    - name: Deploy to Azure Web App
      uses: azure/webapps-deploy@v2
      with:
        app-name: ${{ inputs.app-name }}
        package: ./publish/
    
    - name: Deployment verification
      run: |
        echo "Verifying deployment to ${{ inputs.environment }}..."
        # Add verification logic here
```

2. Click **Commit changes...** and then **Commit changes**.

### Exercise 2: Configure GitHub Environments and Protection Rules

In this exercise, you will set up GitHub environments with approval processes and protection rules.

#### Task 1: Create GitHub Environments

1. In your repository, go to **Settings**.
1. In the left sidebar, click **Environments**.
1. Click **New environment**.
1. Enter **development** as the environment name and click **Configure environment**.
1. Configure the development environment:
   - Leave **Required reviewers** unchecked (automatic deployment)
   - Set **Deployment branches** to **Selected branches** and add `develop` branch
   - Click **Save protection rules**

1. Create another environment named **staging**:
   - Check **Required reviewers** and add your GitHub username
   - Set **Wait timer** to 5 minutes
   - Set **Deployment branches** to **Selected branches** and add `main` branch
   - Click **Save protection rules**

1. Create a third environment named **production**:
   - Check **Required reviewers** and add your GitHub username
   - Set **Wait timer** to 10 minutes
   - Set **Deployment branches** to **Selected branches** and add `main` branch
   - Under **Environment secrets**, you can add production-specific secrets
   - Click **Save protection rules**

#### Task 2: Configure Branch Protection Rules

1. Still in **Settings**, click **Branches** in the left sidebar.
1. Click **Add rule** to create a branch protection rule.
1. Configure the rule for the **main** branch:
   - Branch name pattern: `main`
   - Check **Require a pull request before merging**
   - Check **Require status checks to pass before merging**
   - Check **Require branches to be up to date before merging**
   - In the search box, add status checks: `Validate and Analyze`, `Build and Test`, `Security Analysis`
   - Check **Restrict pushes that create files larger than 100MB**
   - Click **Create**

#### Task 3: Add Repository Secrets

1. In **Settings**, click **Secrets and variables** > **Actions**.
1. Click **New repository secret** and add the following secrets:
   - Name: `AZURE_CREDENTIALS`
   - Value: (Your Azure service principal JSON - you'll need to create this in Azure)
   - Click **Add secret**

1. Add another secret:
   - Name: `AZURE_SUBSCRIPTION_ID`
   - Value: (Your Azure subscription ID)
   - Click **Add secret**

### Exercise 3: Test Advanced Workflow Features

In this exercise, you will test the advanced features of your GitHub Actions pipelines.

#### Task 1: Test the complete workflow

1. Create a new branch for testing:
   - Go to **Code** tab
   - Click the branch dropdown (should show "main")
   - Type `develop` and click **Create branch: develop from main**

1. Make a small change to trigger the workflow:
   - Navigate to any file in the `src/` directory
   - Click the pencil icon to edit
   - Make a minor change (add a comment or change some text)
   - Commit directly to the `develop` branch

1. Go to the **Actions** tab to see the workflow execution:
   - The workflow should trigger automatically
   - Watch the different jobs execute (validate, build, security-scan, deploy-development)
   - Notice that only development deployment occurs for the develop branch

#### Task 2: Test manual workflow dispatch

1. In the **Actions** tab, click on your **eShopOnWeb Advanced CI/CD Pipeline** workflow.
1. Click **Run workflow**.
1. Configure the manual run:
   - Use workflow from: `main`
   - Target environment: `staging`
   - Deploy infrastructure: `true`
1. Click **Run workflow** and observe the execution.

#### Task 3: Test approval process

1. Create a pull request from `develop` to `main`:
   - Go to **Pull requests** tab
   - Click **New pull request**
   - Set base: `main` and compare: `develop`
   - Add title: "Test approval process"
   - Click **Create pull request**

1. Merge the pull request:
   - Click **Merge pull request**
   - Click **Confirm merge**

1. Go to **Actions** to see the workflow execution:
   - The workflow should now include staging and production deployments
   - Notice the approval requirements for staging and production environments
   - Approve the deployments when prompted

#### Task 4: Review workflow insights

1. In the **Actions** tab, explore the workflow runs:
   - Click on different workflow runs to see details
   - Review the job dependencies and execution times
   - Check the artifacts produced by each run

1. Go to **Insights** > **Dependency graph** to see workflow dependencies.
1. Review **Security** > **Code scanning** for any security issues found.

## Review

In this lab, you configured advanced GitHub Actions workflows as code with YAML. You learned how to:

- Create sophisticated multi-stage workflows with conditional logic
- Implement reusable workflow components for better maintainability
- Configure GitHub environments with approval processes and protection rules
- Set up branch protection rules and required status checks
- Integrate security scanning and code analysis into your pipelines
- Use manual workflow dispatch with input parameters
- Test and validate complex workflow scenarios

These advanced techniques enable teams to implement robust CI/CD pipelines using GitHub Actions as code, providing the same level of sophistication and control as traditional pipeline platforms while maintaining the benefits of version-controlled, collaborative pipeline development.