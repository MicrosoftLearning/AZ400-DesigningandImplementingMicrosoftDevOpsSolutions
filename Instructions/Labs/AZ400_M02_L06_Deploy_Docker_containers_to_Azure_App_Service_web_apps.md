---
lab:
    title: 'Deploy Docker containers to Azure App Service web apps'
    module: 'Module 02: Implement CI with Azure Pipelines and GitHub Actions'
---

# Deploy Docker containers to Azure App Service web apps

## Lab requirements

- This lab requires **Microsoft Edge** or an [Azure DevOps supported browser.](https://learn.microsoft.com/azure/devops/server/compatibility)

- **Set up an Azure DevOps organization:** If you don't already have an Azure DevOps organization that you can use for this lab, create one by following the instructions available at [Create an organization or project collection](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization).

- Identify an existing Azure subscription or create a new one.

- Verify that you have a Microsoft account or a Microsoft Entra account with the Contributor or the Owner role in the Azure subscription. For details, refer to [List Azure role assignments using the Azure portal](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) and [View and assign administrator roles in Azure Active Directory](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Lab overview

In this lab, you will learn how to use an Azure DevOps CI/CD pipeline to build a custom Docker image, push it to Azure Container Registry, and deploy it as a container to Azure App Service.

## Objectives

After you complete this lab, you will be able to:

- Build a custom Docker image by using a Microsoft hosted Linux agent.
- Push an image to Azure Container Registry.
- Deploy a Docker image as a container to Azure App Service by using Azure DevOps.

## Estimated timing: 20 minutes

## Instructions

### Exercise 0: (skip if done) Configure the lab prerequisites

In this exercise, you will set up the prerequisites for the lab, which consist of a new Azure DevOps project with a repository based on the [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Task 1: (skip if done) Create and configure the team project

In this task, you will create an **eShopOnWeb** Azure DevOps project to be used by several labs.

1. On your lab computer, in a browser window open your Azure DevOps organization. Click on **New Project**. Give your project the name **eShopOnWeb** and choose **Scrum** on the **Work Item process** dropdown. Click on **Create**.

#### Task 2: (skip if done) Import eShopOnWeb Git Repository

In this task you will import the eShopOnWeb Git repository that will be used by several labs.

1. On your lab computer, in a browser window open your Azure DevOps organization and the previously created **eShopOnWeb** project. Click on **Repos > Files** , **Import**. On the **Import a Git Repository** window, paste the following URL <https://github.com/MicrosoftLearning/eShopOnWeb.git>  and click on **Import**:

1. The repository is organized the following way:
    - **.ado** folder contains Azure DevOps YAML pipelines.
    - **.devcontainer** folder container setup to develop using containers (either locally in VS Code or GitHub Codespaces).
    - **infra** folder contains Bicep&ARM infrastructure as code templates used in some lab scenarios.
    - **.github** folder container YAML GitHub workflow definitions.
    - **src** folder contains the .NET 8 website used on the lab scenarios.

#### Task 3: (skip if done) Set main branch as default branch

1. Go to **Repos > Branches**.
1. Hover on the **main** branch then click the ellipsis on the right of the column.
1. Click on **Set as default branch**.

### Exercise 1: Import and run the CI pipeline

In this exercise, you will configure the service connection with your Azure Subscription then import and run the CI pipeline.

#### Task 1: Import and run the CI pipeline

1. Go to **Pipelines > Pipelines**
1. Click on **New pipeline** button (or **Create Pipeline** if you don't have other pipelines previously created)
1. Select **Azure Repos Git (YAML)**
1. Select the **eShopOnWeb** repository
1. Select **Existing Azure Pipelines YAML file**
1. Select the **main** branch and the **/.ado/eshoponweb-ci-docker.yml** file, then click on **Continue**
1. In the YAML pipeline definition, customize:
   - **YOUR-SUBSCRIPTION-ID** with your Azure subscription ID.
   - Replace the **resourceGroup** with the resource group name used during the service connection creation, for example, **AZ400-RG1**.

1. Review the pipeline definition. The CI definition consists of the following tasks:
    - **Resources**: It downloads the repository files that will be used in the following tasks.
    - **AzureResourceManagerTemplateDeployment**: Deploys the Azure Container Registry using bicep template.
    - **PowerShell**: Retrieve the **ACR Login Server** value from the previous task's output and create a new parameter **acrLoginServer**
    - [**Docker**](https://learn.microsoft.com/azure/devops/pipelines/tasks/reference/docker-v0?view=azure-pipelines) **- Build**: Build the Docker image and create two tags (Latest and current BuildID)
    - **Docker - Push**: Push the images to Azure Container Registry

1. Click on **Save and Run**.

1. Open the pipeline execution. If you see a warning message "This pipeline needs permission to access a resource before this run can continue to Build", click on **View** and then **Permit** and **Permit** again. This will allow the pipeline to access the Azure subscription.

    > **Note**: The deployment may take a few minutes to complete.

1. Your pipeline will take a name based on the project name. Let's **rename** it for identifying the pipeline better. Go to **Pipelines > Pipelines** and click on the recently created pipeline. Click on the ellipsis and **Rename/move** option. Name it **eshoponweb-ci-docker** and click on **Save**.

1. Navigate to the [**Azure Portal**](https://portal.azure.com), search for the Azure Container Registry in the recently created Resource Group (it should be named **AZ400-RG1**). On the left-hand side click **Repositories** under **Services** and make sure that the repository **eshoponweb/web** was created. When you click the repository link, you should see two tags (one of them is **latest**), these are the pushed images. If you don't see this, check the status of your pipeline.

### Exercise 2: Import and run the CD pipeline

In this exercise, you will configure the service connection with your Azure Subscription then import and run the CD pipeline.

#### Task 1: Import and run the CD pipeline

In this task, you will import and run the CD pipeline.

1. Go to **Pipelines > Pipelines**
1. Click on **New pipeline** button
1. Select **Azure Repos Git (YAML)**
1. Select the **eShopOnWeb** repository
1. Select **Existing Azure Pipelines YAML File**
1. Select the **main** branch and the **/.ado/eshoponweb-cd-webapp-docker.yml** file, then click on **Continue**
1. In the YAML pipeline definition, customize:
   - **YOUR-SUBSCRIPTION-ID** with your Azure subscription ID.
   - Replace the **resourceGroup** with the resource group name used during the service connection creation, for example, **AZ400-RG1**.
   - Replace **location** with the Azure region where the resources will be deployed.

1. Review the pipeline definition. The CD definition consists of the following tasks:
    - **Resources**: It downloads the repository files that will be used in the following tasks.
    - **AzureResourceManagerTemplateDeployment**: Deploys the Azure App Service using bicep template.
    - **AzureResourceManagerTemplateDeployment**: Add role assignment using Bicep

1. Click on **Save and Run**.

1. Open the pipeline execution. If you see a warning message "This pipeline needs permission to access a resource before this run can continue to Deploy", click on **View** and then **Permit** and **Permit** again. This will allow the pipeline to access the Azure subscription.

    > **Note**: The deployment may take a few minutes to complete.

    > [!IMPORTANT]
    > If you receive the error message "TF402455: Pushes to this branch are not permitted; you must use a pull request to update this branch.", you need to uncheck the "Require a minimum number of reviewers" Branch protection rule enabled in the previous labs.

1. Your pipeline will take a name based on the project name. Let's **rename** it for identifying the pipeline better. Go to **Pipelines > Pipelines** and hover on the recently created pipeline. Click on the ellipsis and **Rename/move** option. Name it **eshoponweb-cd-webapp-docker** and click on **Save**.

    > **Note 1**: The use of the **/infra/webapp-docker.bicep** template creates an app service plan, a web app with system assigned managed identity enabled, and references the Docker image pushed previously: **${acr.properties.loginServer}/eshoponweb/web:latest**.

    > **Note 2**: The use of the **/infra/webapp-to-acr-roleassignment.bicep** template creates a new role assignment for the web app with AcrPull role to be able to retrieve the Docker image. This could be done in the first template, but since the role assignment can take some time to propagate, it's a good idea to do both tasks separately.

#### Task 2: Test the solution

1. In the Azure Portal, navigate to the recently created Resource Group, you should now see three resources (App Service, App Service Plan and Container Registry).

1. Navigate to the App Service, then click **Browse**, this will take you to the website.

> [!IMPORTANT]
> Remember to delete the resources created in the Azure portal to avoid unnecessary charges.

## Review

In this lab, you have learned how to use an Azure DevOps CI/CD pipeline to build a custom Docker image, push it to Azure Container Registry, and deploy it as a container to Azure App Service.
