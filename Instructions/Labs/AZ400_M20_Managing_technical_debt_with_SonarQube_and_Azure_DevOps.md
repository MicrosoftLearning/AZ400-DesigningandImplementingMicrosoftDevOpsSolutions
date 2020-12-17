---
lab:
    title: 'Lab: Managing technical debt with SonarQube and Azure DevOps'
    az400Module: 'Module 20: Validating Code Bases for Compliance'
---

# Lab: Managing technical debt with SonarQube and Azure DevOps
# Student lab manual

## Lab overview

In the context of Azure DevOps, the term *technical debt* represents suboptimal means of reaching tactical goals, which affect negatively the ability to reach strategic objectives in the area of software development and deployment. Technical debt affects productivity by making code hard to understand, prone to failures, time-consuming to change, and difficult to validate. Without proper oversight and management, technical debt can accumulate over time and significantly impact the overall quality of the software and the productivity of development teams in the longer term.

[SonarQube](https://www.sonarqube.org/) is an open source platform for continuous inspection of code quality that facilitates automatic reviews with static analysis of code to improve its quality by detecting bugs, code smells, and security vulnerabilities.

In this lab, you will learn how to setup SonarQube on Azure and integrate it with Azure DevOps.

## Objectives

After you complete this lab, you will be able to:

- Provision SonarQube server as an [Azure Container Instance](https://docs.microsoft.com/en-in/azure/container-instances/) from the SonarQube Docker image
- Setup a SonarQube project
- Provision an Azure DevOps Project and configure CI pipeline to integrate with SonarQube
- Analyze SonarQube reports

## Lab duration

-   Estimated time: **60 minutes**

## Instructions

### Before you start

#### Sign in to the lab virtual machine

Ensure that you're signed in to your Windows 10 computer by using the following credentials:
    
-   Username: **Admin**
-   Password: **Pa55w.rd**

#### Review the installed applications

Find the taskbar on your Windows desktop. The taskbar contains the icons for the applications that you'll use in this lab:
    
-   Microsoft Edge

#### Set up an Azure DevOps organization. 

Follow instructions available at [Create an organization or project collection](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops). When creating the Azure DevOps organization, sign in with the same the user same account you used to set up the Office 365 subscription.

#### Prepare an Azure subscription

-   Identify an existing Azure subscription or create a new one.
-   Verify that you have a Microsoft account or an Azure AD account with the Owner role in the Azure subscription and the Global Administrator role in the Azure AD tenant associated with the Azure subscription.

### Exercise 0: Configure the lab prerequisites

In this exercise, you will set up the prerequisites for the lab, which consist of the preconfigured **Tailwind Traders** team project based on an Azure DevOps Demo Generator template and a team created in Microsoft Teams.

#### Task 1: Configure the team project

In this task, you will use Azure DevOps Demo Generator to generate a new project based on the **SonarQube** template.

1.  Navigate to [Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net). This utility site will automate the process of creating a new Azure DevOps project within your account that is prepopulated with content (work items, repos, etc.) required for the lab. 

    > **Note**: For more information on the site, see https://docs.microsoft.com/en-us/azure/devops/demo-gen.

1.  Click **Sign in** and sign in using the Microsoft account associated with your Azure DevOps subscription.
1.  If required, on the **Azure DevOps Demo Generator** page, click **Accept** to accept the permission requests for accessing your Azure DevOps subscription.
1.  On the **Create New Project** page, in the **New Project Name** textbox, type **Managing technical debt with SonarQube**, in the **Select organization** dropdown list, select your Azure DevOps organization, and then click **Choose template**.
1.  In the list of templates, in the toolbar, click **DevOps Labs**, select the **SonarQube** template and click **Select Template**.
1.  Back on the **Create New Project** page, if prompted to install a missing extension, select the checkbox below the **SonarQube** and click **Create Project**.

    > **Note**: Wait for the process to complete. This should take about 2 minutes. In case the process fails, navigate to your DevOps organization, delete the project, and try again.

1.  On the **Create New Project** page, click **Navigate to project**.

#### Task 2: Create an Azure Container instance hosting the SonarQube container

In this task, you will provision an Azure Container instance hosting the SonarQube container.

1.  From the lab computer, start a web browser, navigate to the [**Azure Portal**](https://portal.azure.com), and sign in with the user account that has at least the Contributor role in the Azure subscription you are using in this lab.
1.  In the Azure portal, in the toolbar, click the **Cloud Shell** icon located directly to the right of the search text box. 
1.  If prompted to select either **Bash** or **PowerShell**, select **Bash**. 

    >**Note**: If this is the first time you are starting **Cloud Shell** and you are presented with the **You have no storage mounted** message, select the subscription you are using in this lab, and select **Create storage**. 

1.  From the Bash session in the Cloud Shell pane, run the following to create a resource group that will host the AKS deployment (replace the `<Azure_region` placeholder with the name of the Azure region where you intend to deploy resources in this lab):

    ```bash
    LOCATION=<Azure_region>
    RGNAME=az400m20l01a-RG
    az group create --name $RGNAME --location $LOCATION
    ```

1.  From the Bash session in the Cloud Shell pane, run the following to create an Azure Container instance hosting the SonarQube Docker image:
    
    ```bash
    ACINAME='az400m20aci'$RANDOM$RANDOM
    az container create --location $LOCATION --resource-group $RGNAME --name $ACINAME --image sonarqube --ports 9000 --dns-name-label $ACINAME --cpu 2 --memory 3.5
    ```

    > **Note**: The parameters designate the following settings:
         
    | Name | Description |
    | ---- | ----------- |
    | `--name` | Name of the container instance |
    | `--image` | the official [SonarQube image](https://hub.docker.com/_/sonarqube) image from Docker Hub |
    | `--ports` | The default port for SonarQube **9000** for access to SonarQube |
    | `--dns-name-label` | The DNS name for access to the SonarQube container |
    | `--cpu` | The minimum required number of CPU cores for the container |
    | `--memory` | The minimum required amount of memory (in GB) for the container |
      
    >**Note**: Wait for the deployment complete before you proceed to the next task. ACI deployment might take about 2 minutes. 

1.  From the Bash session in the Cloud Shell pane, run the following to identify the DNS name assigned Azure Container instance hosting the SonarQube Docker image:
    
    ```bash
    az container list --resource-group $RGNAME --query '[].ipAddress.fqdn' --output tsv
    ```

    >**Note**: Record the DNS name of the container. You will need it later in this lab.

### Exercise 1: Integrate SonarQube with Azure DevOps.

In this exercise, you will create a SonarQube project and configure its Quality Gate, integrate the DevOps build of our sample project with SonarQube, run the build, and analyze the resulting SonarQube report.

#### Task 1: Create a SonarQube project and configure Quality Gate

In this task, you will create a SonarQube project and configure its Quality Gate.

1.  From the lab computer, start a web browser and navigate to the SonarQube container by using the DNS name you identified in the previous task followed by **:9000** to account for the port number that the SonarQube process is listening on.

    >**Note**: The URL should resemble the string `http:\\az400m12aci<random_suffix>.<Azure_region>eastus.azurecontainer.io:9000`, where the `<Azure_region` placeholder represents the name of the Azure region where you deployed the container and the `<random_suffix>` represents the random string of characters generated by **$RANDOM** shell function.

1.  In the web browser, on the SonarQube home page, in the upper right corner, click **Log in** and, when prompted for credentials, sign in by using **admin** as **Login** and **admin** as **Password**.
1.  In the web browser, in the center of the SonarQube page, click **Create new project**.
1.  On the **Create a project** page, set the **Project key** and **Display name** to **az400m20l01sq** and click **Set up***

    >**Note**: **Name** will be displayed in the web interface of the SonarQube portal while **Key** must be unique for each project. By default, the project's visibility is set to **Public**.

    >**Note**: Now let's create a Quality Gate to enforce a policy, which will result in failure if there are bugs in the code. A Quality Gate is a PASS/FAIL check on a code quality that must be enforced before releasing software.

1.  In the web browser displaying the **Overview** tab of the **az400m20l01sq** project, in the **Generate a token** section, in the **Enter a name for your token** text box, type **az400m20l01sqToken** and click **Generate**.

    >**Note**: Make sure to record the newly generated token. You will need it later in this lab.

1.  In the web browser displaying the *Overview** tab of the **az400m20l01sq** project, in the top menu, click **Quality Gates**, in the upper left corner of the page, click **Create**, when prompted, in the **Create Quality Gate** pop-up window, in the **Name** textbox, type **az400m20l01sqQG**, and click **Save**.

    >**Note**: Let's add a condition to check for the number of bugs in the code. 

1.  In the upper right corner of the **Quality Gates** page, click **Add Condition**, in the **Add Condition** pop-up window, select the **On Overall Code** option, in the **Quality Gate fails when** dropdown list, in the **Reliability** section, select **Bugs**, with the **Operator** set to **is greater than**, in the **Value** textbox, type **0**, and click **Add Condition**.

    >**Note**: Effectively, if the number of bugs in SonarQube analysis is greater than 0, then the quality gate will fail, resulting in a failure of the corresponding Azure DevOps build. 

1.  Back on the **Quality Gates** page, in the **Projects** section, click **All**, an select the checkbox next to the **az400m20l01sq** entry to enforce this quality gate for the **az400m20l01sq** project.

#### Task 2: Integrate the DevOps build with SonarQube

In this task, you will modify the Azure Build pipeline of the project you generated earlier in this lab in order to integrate it with SonarQube, resulting in automatic analysis of its Java code.

>**Note**: Our sample project implements a Java application and uses [Maven](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/build/maven?view=azure-devops) to build the code. We will use the [SonarQube](https://marketplace.visualstudio.com/items?itemName=SonarSource.sonarqube) extension tasks to initiate code analysis by SonarQube, which will provide Quality Gate results.

1.  On your lab computer, switch to the web browser window displaying the Azure DevOps portal with the **Managing technical debt with SonarQube** project open, in the vertical menu bar at the far left of the Azure DevOps portal, click **Pipelines**.
1.  On the **Pipelines** pane, click the entry representing the **SonarQube** build pipeline and, on the **SonarQube** pane, click **Edit**.
1.  In the list of tasks of the **SonarQube** build pipeline, click **Prepare analysis on SonarQube**.
1.  On the **Prepare Analysis Configuration** pane, on the right side of the **SonarQube Server Endpoint** textbox, click **+ New** to add the SonarQube server endpoint.
1.  On the **New service connection** pane, in the **Server Url** textbox, type the same URL you used in the previous task to connect to the Azure Container instance hosting the SonarQube server, in the **Token** text box, paste the **SonarQube** token you generated and recorded in the previous task, in the **Service connection name** textbox, type **SonarQube**, and click **Save**.

    >**Note**: The URL should resemble the string `http:\\az400m12aci<random_suffix>.<Azure_region>eastus.azurecontainer.io:9000`, where the `<Azure_region` placeholder represents the name of the Azure region where you deployed the container and the `<random_suffix>` represents the random string of characters generated by **$RANDOM** shell function.

    >**Note**: The tokens are used to run analysis or invoke web services without access to the user's actual credentials.

1.  In the list of tasks of the **SonarQube** build pipeline, click **Publish Quality Gate Result** task and review its settings without making any changes. 

    >**Note**: This task will result in displaying the Quality Gate status in the build summary.

1.  In the list of tasks of the **SonarQube** build pipeline, in the top menu, click **Save & queue**, in the dropdown menu, click **Save & queue**. 
1.  In the **Run pipeline** pane, click **Save and run**. This will automatically display the pane representing the newly initiated build.
1.  On the build pane, in the **Jobs** section, click **Phase 1** and monitor the progress of the build process.
1.  Navigate back to the pane displaying the most recent run and click the **Extensions** tab. 
1.  On the **Extensions** tab, in the **SonarQube Analysis Report** section, verify that the **az400m20l01sq Quality Gate** is listed with the **Failed** status and the total of **23** bugs.

#### Task 3: Analyze SonarQube reports

In this task, you will analyze SonarQube reports.

1.  In the web browser window displaying the Azure DevOps portal with the most recent build run, on the **Extensions** tab, in the **SonarQube Analysis Report** section, click the **Detailed SonarQube report** link. This will automatically open a new web browser tab displaying the **az400m20l01sq** project page in the SonarQube.
1.  In the SonarQube portal, on the **QUALITY GATES STATUS** page of the **az400m20l01sq** status, click the **Overall Code** tab. 
1.  Review the statistics and verify that there are 23 bugs reported.

    >**Note**: In addition to the number of **Bugs**, the page displays other metrics such as **Vulnerabilities**, **Code Smells**, **Coverage**, **Duplications** and **Debt**:

    | Terms | Description |
    | ----- | ----------- |
    | **Bugs** | An issue in the code that warrants further investigation |
    | **Vulnerabilities** | A security-related issue which represents a potential backdoor for attackers |
    | **Code Smells** | A maintainability-related issue in the code. Leaving it as-is means that, at best, ongoing maintenance will incur additional overhead |
    | **Coverage** | The percentage of the code which is actually being tested by tests such as unit tests. To guard effectively against bugs, these tests should cover majority of the code |
    | **Duplications** | The percentage of the source code which is duplicated |
    | **Debt** | An estimate duration of the effort required to fix all maintainability issues |

1. Click the **Bugs** count. This will display detailed information about each bug. 

## Review

In this lab, you created a SonarQube project, configured its Quality Gate, integrated the DevOps build of our sample project with SonarQube, ran the build, and analyzed the resulting SonarQube report.