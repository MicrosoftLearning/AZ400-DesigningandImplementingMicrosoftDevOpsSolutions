---
lab:
    title: 'Deploying Docker containers to Azure App Service web apps'
    module: 'Module 03: Implement CI with Azure Pipelines and GitHub Actions'
---

# Deploying Docker containers to Azure App Service web apps

## Student lab manual

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

## Estimated timing: 30 minutes

## Instructions

### Exercise 0: Configure the lab prerequisites

In this exercise, you will set up the prerequisites for the lab, which consist of a new Azure DevOps project with a repository based on the [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

#### Task 1: (skip if done) Create and configure the team project

In this task, you will create an **eShopOnWeb** Azure DevOps project to be used by several labs.

1. On your lab computer, in a browser window open your Azure DevOps organization. Click on **New Project**. Give your project the name **eShopOnWeb** and choose **Scrum** on the **Work Item process** dropdown. Click on **Create**.

#### Task 2: (skip if done) Import eShopOnWeb Git Repository

In this task you will import the eShopOnWeb Git repository that will be used by several labs.

1. On your lab computer, in a browser window open your Azure DevOps organization and the previously created **eShopOnWeb** project. Click on **Repos>Files** , **Import**. On the **Import a Git Repository** window, paste the following URL https://github.com/MicrosoftLearning/eShopOnWeb.git  and click on **Import**:

2. The repository is organized the following way:
    - **.ado** folder contains Azure DevOps YAML pipelines.
    - **.devcontainer** folder container setup to develop using containers (either locally in VS Code or GitHub Codespaces).
    - **.azure** folder contains Bicep&ARM infrastructure as code templates used in some lab scenarios.
    - **.github** folder container YAML GitHub workflow definitions.
    - **src** folder contains the .NET 6 website used on the lab scenarios.

#### Task 3: (skip if done) Set main branch as default branch

1. Go to **Repos>Branches**.
2. Hover on the **main** branch then click the ellipsis on the right of the column.
3. Click on **Set as default branch**.

### Exercise 1: Manage the service connection

In this exercise, you will configure the service connection with your Azure Subscription then import and run the CI pipeline.

#### Task 1: (skip if done) Manage the service connection

You can create a connection from Azure Pipelines to external and remote services for executing tasks in a job.

In this task, you will create a service principal by using the Azure CLI, which will allow Azure DevOps to:

- Deploy resources on your azure subscription.
- Push the Docker image to Azure Container Registry.
- Add a role assignment to allow Azure App Service pull the Docker image from Azure Container Registry.

> **Note**: If you do already have a service principal, you can proceed directly to the next task.

You will need a service principal to deploy  Azure resources from Azure Pipelines.

A service principal is automatically created by Azure Pipeline when you connect to an Azure subscription from inside a pipeline definition or when you create a new service connection from the project settings page (automatic option). You can also manually create the service principal from the portal or using Azure CLI and re-use it across projects.

1. From the lab computer, start a web browser, navigate to the [**Azure Portal**](https://portal.azure.com), and sign in with the user account that has the Owner role in the Azure subscription you will be using in this lab and has the role of the Global Administrator in the Microsoft Entra tenant associated with this subscription.
2. In the Azure portal, click on the **Cloud Shell** icon, located directly to the right of the search textbox at the top of the page.
3. If prompted to select either **Bash** or **PowerShell**, select **Bash**.

   >**Note**: If this is the first time you are starting **Cloud Shell** and you are presented with the **You have no storage mounted** message, select the subscription you are using in this lab, and select **Create storage**.

4. From the **Bash** prompt, in the **Cloud Shell** pane, run the following commands to retrieve the values of the Azure subscription ID attribute:

    ```bash
    subscriptionName=$(az account show --query name --output tsv)
    subscriptionId=$(az account show --query id --output tsv)
    echo $subscriptionName
    echo $subscriptionId
    ```

    > **Note**: Copy both values to a text file. You will need them later in this lab.

5. From the **Bash** prompt, in the **Cloud Shell** pane, run the following command to create a service principal:

    ```bash
    az ad sp create-for-rbac --name sp-az400-azdo --role contributor --scopes /subscriptions/$subscriptionId
    ```

    > **Note**: The command will generate a JSON output. Copy the output to text file. You will need it later in this lab.

6. Next, from the lab computer, start a web browser, navigate to the Azure DevOps **eShopOnWeb** project. Click on **Project Settings>Service Connections (under Pipelines)** and **New Service Connection**.
7. On the **New service connection** blade, select **Azure Resource Manager** and **Next** (may need to scroll down).
8. The choose **Service principal (manual)** and click on **Next**.
9. Fill in the empty fields using the information gathered during previous steps:
    - Subscription Id and Name.
    - Service Principal Id (appId), Service principal key (password) and Tenant ID (tenant).
    - In **Service connection name** type **azure-connection**. This name will be referenced in YAML pipelines when needing an Azure DevOps Service Connection to communicate with your Azure subscription.

10. Click on **Verify and Save**.

### Exercise 2: Import and run the CI pipeline

In this exercise, you will import and run the CI pipeline.

#### Task 1: Import and run the CI pipeline

1. Go to **Pipelines>Pipelines**
2. Click on **New pipeline** button
3. Select **Azure Repos Git (YAML)**
4. Select the **eShopOnWeb** repository
5. Select **Existing Azure Pipelines YAML file**
6. Select the **/.ado/eshoponweb-ci-docker.yml** file then click on **Continue**
7. In the YAML pipeline definition, customize:
   - **YOUR-SUBSCRIPTION-ID** with your Azure subscription ID.
   - **rg-az400-container-NAME** with the resource group name that will be created by the pipeline (it can be an existing resource group too).

8. Click on **Save and Run** and wait for the pipeline to execute successfully.

    > **Note**: The deployment may take a few minutes to complete.

    The CI definition consists of the following tasks:
    - **Resources**: It downloads the repository files that will be used in the following tasks.
    - **AzureResourceManagerTemplateDeployment**: Deploys the Azure Container Registry using bicep template.
    - **PowerShell**: Retrieve the **ACR Login Server** value from the previous task's output and create a new parameter **acrLoginServer**
    - [**Docker**](https://learn.microsoft.com/azure/devops/pipelines/tasks/reference/docker-v0?view=azure-pipelines) **- Build**: Build the Docker image and create two tags (Latest and current BuildID)
    - **Docker - Push**: Push the images to Azure Container Registry

9. Your pipeline will take a name based on the project name. Let's **rename** it for identifying the pipeline better. Go to **Pipelines>Pipelines** and click on the recently created pipeline. Click on the ellipsis and **Rename/move** option. Name it **eshoponweb-ci-docker** and click on **Save**.

10. Navigate to the [**Azure Portal**](https://portal.azure.com), search for the Azure Container Registry in the recently created Resource Group (it should be named **rg-az400-container-NAME**). On the left-hand side click **Repositories** under **Services** and make sure that the repository **eshoponweb/web** was created. When you click the repository link, you should see two tags (one of them is **latest**), these are the pushed images. If you don't see this, check the status of your pipeline.

### Exercise 3: Import and run the CD pipeline

In this exercise, you will configure the service connection with your Azure Subscription then import and run the CD pipeline.

#### Task 1: Add a new role assignment

In this task, you will add a new role assignment to allow Azure App Service pull the Docker image from Azure Container Registry.

1. Navigate to the [**Azure Portal**](https://portal.azure.com).
2. In the Azure portal, click on the **Cloud Shell** icon, located directly to the right of the search textbox at the top of the page.
3. If prompted to select either **Bash** or **PowerShell**, select **Bash**.
4. From the **Bash** prompt, in the **Cloud Shell** pane, run the following commands to retrieve the values of the Azure subscription ID attribute:

    ```sh
    spId=$(az ad sp list --display-name sp-az400-azdo --query "[].id" --output tsv)
    echo $spId
    roleName=$(az role definition list --name "User Access Administrator" --query "[0].name" --output tsv)
    echo $roleName
    ```

5. After getting the service principal ID and the role name, let's create the role assignment by running this command (replace **rg-az400-container-NAME** with your resource group name)

    ```sh
    az role assignment create --assignee $spId --role $roleName --resource-group "rg-az400-container-NAME"
    ```

You should now see the JSON output which confirms the success of the command run.

#### Task 2: Import and run the CD pipeline

In this task, you will import and run the CD pipeline.

1. Go to **Pipelines>Pipelines**
2. Click on **New pipeline** button
3. Select **Azure Repos Git (YAML)**
4. Select the **eShopOnWeb** repository
5. Select **Existing Azure Pipelines YAML File**
6. Select the **/.ado/eshoponweb-cd-webapp-docker.yml** file then click on **Continue**
7. In the YAML pipeline definition, customize:
   - **YOUR-SUBSCRIPTION-ID** with your Azure subscription ID.
   - **rg-az400-container-NAME** with the resource group name defined before in the lab.

8. Click on **Save and Run** and wait for the pipeline to execute successfully.

    > **Note**: The deployment may take a few minutes to complete.
    
    > **Important**: If you receive the error message "TF402455: Pushes to this branch are not permitted; you must use a pull request to update this branch.", you need to uncheck the "Require a minimum number of reviewers" Branch protection rule enabled in the previous labs.

    The CD definition consists of the following tasks:
    - **Resources**: It downloads the repository files that will be used in the following tasks.
    - **AzureResourceManagerTemplateDeployment**: Deploys the Azure App Service using bicep template.
    - **AzureResourceManagerTemplateDeployment**: Add role assignment using Bicep

9. Your pipeline will take a name based on the project name. Let's **rename** it for identifying the pipeline better. Go to **Pipelines>Pipelines** and hover on the recently created pipeline. Click on the ellipsis and **Rename/move** option. Name it **eshoponweb-cd-webapp-docker** and click on **Save**.

    > **Note 1**: The use of the **/.azure/bicep/webapp-docker.bicep** template creates an app service plan, a web app with system assigned managed identity enabled, and references the Docker image pushed previously: **${acr.properties.loginServer}/eshoponweb/web:latest**.

    > **Note 2**: The use of the **/.azure/bicep/webapp-to-acr-roleassignment.bicep** template creates a new role assignment for the web app with AcrPull role to be able to retrieve the Docker image. This could be done in the first template, but since the role assignment can take some time to propagate, it's a good idea to do both tasks separately.

#### Task 3: Test the solution

1. In the Azure Portal, navigate to the recently created Resource Group, you should now see three resources (App Service, App Service Plan and Container Registry).

1. Navigate to the App Service, then click **Browse**, this will take you to the website.

Congratulations! In this exercise, you deployed a website using a custom Docker image.

### Exercise 4: Remove the Azure lab resources

In this exercise, you will remove the Azure resources provisioned in this lab to eliminate unexpected charges.

>**Note**: Remember to remove any newly created Azure resources that you no longer use. Removing unused resources ensures you will not see unexpected charges.

#### Task 1: Remove the Azure lab resources

In this task, you will use Azure Cloud Shell to remove the Azure resources provisioned in this lab to eliminate unnecessary charges.

1. In the Azure portal, open the **Bash** shell session within the **Cloud Shell** pane.
2. List all resource groups created throughout the labs of this module by running the following command:

    ```sh
    az group list --query "[?starts_with(name,'rg-az400-container-')].name" --output tsv
    ```

3. Delete all resource groups you created throughout the labs of this module by running the following command:

    ```sh
    az group list --query "[?starts_with(name,'rg-az400-container-')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Note**: The command executes asynchronously (as determined by the --nowait parameter), so while you will be able to run another Azure CLI command immediately afterwards within the same Bash session, it will take a few minutes before the resource groups are actually removed.

## Review

In this lab, you learned how to use an Azure DevOps CI/CD pipeline to build a custom Docker image, push it to Azure Container Registry, and deploy it as a container to Azure App Service.
