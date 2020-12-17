---
lab:
    title: 'Lab: Implement Security and Compliance in an Azure DevOps pipeline'
    az400Module: 'Module 19: Implementing Security in DevOps Projects'
---

# Lab: Implement Security and Compliance in an Azure DevOps pipeline
# Student lab manual

## Lab overview

In this lab, we will create a new Azure DevOps project, populate the project repository with a sample application code, create a build pipeline. Next, we will install WhiteSource Bolt from the Azure DevOps Marketplace to make it available as a build task, activate it, add it to the build pipeline, use it to scan the project code for security vulnerabilities and licensing compliance issues, and finally view the resulting report.

## Objectives

After you complete this lab, you will be able to:

- Create a Build pipeline
- Install WhiteSource Bolt from the Azure DevOps marketplace and activate it
- Add WhiteSource Bolt as a build task in a build pipeline
- Run build pipeline and view WhiteSource security and compliance report

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

In this exercise, you will set up the prerequisites for the lab, which consist of a new Azure DevOps project with a repository based on the [Parts Unlimited MRP GitHub repository](https://www.github.com/microsoft/partsunlimitedmrp).

#### Task 1: Create and configure the team project

In this task, you will create an Azure DevOps project and add to it a repository based on the [Parts Unlimited MRP GitHub repository](https://www.github.com/microsoft/partsunlimitedmrp).

1.  From your lab computer, open a web browser window, navigate to **https://dev.azure.com** and sign in to your Azure DevOps organization.
1.  In the Azure DevOps portal, in the upper right corner, click **+ New project**. 
1.  On the **Create new project** pane, in the **Project name** textbox, type **securityandcomplianceproj**, ensure that the **Visibility** option is set to **Private**, and click **Create**.
1.  On the **securityandcomplianceproj** project pane, in the **Welcome to the project** section, click **Repos**.
1.  On the **securityandcomplianceproj is empty. Add some code!** pane, in the **Import a repository** section, click **Import**.
1.  On the Import a Git repository pane, ensure that the **Repository type** is set to **Git**, in the **Clone URL**, type **https://github.com/Microsoft/PartsUnlimitedMRP.git**, and click **Import**. 

    >**Note**: Once the import completes, you should now see the files populated in your Azure DevOps repo.

#### Task 2: Update references to mavenCentral() in the team project repository

In this task, you will modify content of the files in the newly created repository to update references to sources of Maven artifacts.

>**Note**: This is necessary to address [the PartsUnlimitedMRP GitHub issue #168](https://github.com/microsoft/PartsUnlimitedMRP/issues/168) and the corresponding [PartsUnlimitedMRP GitHub pull request #171](https://github.com/microsoft/PartsUnlimitedMRP/pull/171/files)

1.  On your lab computer, in the web browser window displaying the Azure DevOps portal with the **securityandcomplianceproj** project open, in the vertical menu bar at the far left of the Azure DevOps portal, click **Repos**.
1.  In the **Files** view, in the list of files, locate and select the **src\Backend\IntegrationService\build.gradle** file, on the **build.gradle** pane, click **Edit**, replace `mavenCentral()` in the lines **3** and **21** with the following line, click **Commit**, and, in the **Commit** pane, click **Commit** again:

    ```jscript
    maven { url 'https://repo1.maven.org/maven2/' }
    ```

1.  In the **Files** view, in the list of files, locate and select the **src\Backend\OrderService\build.gradle** file, on the **build.gradle** pane, click **Edit**, replace `mavenCentral()` in the lines **8** and **34** with the following line:

    ```jscript
    maven { url 'https://repo1.maven.org/maven2/' }
    ```

1.  With the file **src\Backend\OrderService\build.gradle** still in the edit mode, replace `maven { url "https://gradle.artifactoryonline.com/gradle/libs/" }` in line **6** with the following line, click **Commit**, and, in the **Commit** pane, click **Commit** again:

    ```jscript
    `maven { url "https://gradle.artifactoryonline.com/gradle/libs/" }`
    ```

1.  Repeat the previous two steps to modify **src\Backend\OrderService\build.war.gradle**.
1.  In the **Files** view, in the list of files, locate and select the **src/Backend/OrderService/gradle/wrapper/gradle-wrapper.properties** file, on the **gradle-wrapper.properties** pane, click **Edit**, replace `http\://services.gradle.org/distributions/gradle-2.1-bin.zip` in the line **6** with the following line,  click **Commit**, and, in the **Commit** pane, click **Commit** again:

    ```jscript
    https\://services.gradle.org/distributions/gradle-2.1-bin.zip
    ```

1. Repeat the previous step to modify **src/Clients/gradle/wrapper/gradle-wrapper.properties**.


### Exercise 1: Integrate WhiteSource Bolt with Azure DevOps build pipeline

In this exercise, you will create a build pipeline, install WhiteSource Bolt from the Azure DevOps Marketplace to make it available as a build task, activate it, add it to the build pipeline, use it to scan the project code for security vulnerabilities and licensing compliance issues, and finally view the resulting report.

#### Task 1: Create a Build pipeline

In this task, you will create a Build pipeline in the newly created Azure DevOps project.

1.  On your lab computer, in the web browser window displaying the Azure DevOps portal with the **securityandcomplianceproj** project open, in the vertical menu bar at the far left of the Azure DevOps portal, click **Pipelines**.
1.  With the **Pipelines** view in the **Pipelines** section selected, on the **Create your first pipeline** pane, click **Create your first pipeline**.
1.  On the **Where is your code?** pane, at the bottom of the list of options, click the link **Use the classic editor**.
1.  On the **Select a source** pane, ensure that the **Azure Repos Git** is selected, accept the default values of the remaining settings, and click **Continue**.
1.  On the **Select a template** pane, scroll down to the end of the list of available templates, click **Empty pipeline**, and click **Apply**.
1.  On the **securityandcomplianceproj-CI** pane, on the **Tasks** tab, verify that the **Agent pool** is set to **Azure Pipelines** and the **Agent Specification** is set to **vs2017-win2016**.
1.  In the list of tasks, click the plus sign to the right of the **Agent job 1** entry, on the **Add tasks** pane, select the **Build** tab, in the **Search** textbox, type **Gradle**, in the list of results, select **Gradle**, and then click **Add** three times to add three identical **Gradle** tasks to the job.

    >**Note**: Gradle will be used to build the Integration Service, Order Service, and Clients components of the MRP app.

1.  Select the first of the newly added **Gradle** tasks and, on the **Gradle** pane, specify the following settings (leave others with their default values):

    | Setting | Value |
    | ------- | ----- |
    | Display name | **IntegrationService** |
    | Gradle wrapper | **src/Backend/IntegrationService/gradlew** |
    | Working Directory | **src/Backend/IntegrationService** |
    | JUnit Test Results | clear the checkbox **Publish to Azure Pipelines**, since we will not be running automated tests in Integration Service |

1.  Select the second of the newly added **Gradle** tasks and, on the **Gradle** pane, specify the following settings (leave others with their default values):

    | Setting | Value |
    | ------- | ----- |
    | Display name | **OrderService** |
    | Gradle wrapper | **src/Backend/OrderService/gradlew** |
    | Working Directory | **src/Backend/OrderService** |
    | JUnit Test Results | ensure that the checkbox **Publish to Azure Pipelines** is enabled and the **Test Results files** textbox is set to **\*\*/TEST-\*.xml**.  |

    >**Note**: Since the Order Service does have unit tests in our sample project, we can automate running the tests as part of the build by adding a test in this Gradle task.

1.  Select the third of the newly added **Gradle** tasks and, on the **Gradle** pane, specify the following settings (leave others with their default values):

    | Setting | Value |
    | ------- | ----- |
    | Display name | **Clients** |
    | Gradle wrapper | **src/Clients/gradlew** |
    | Working Directory | **src/Clients** |
    | JUnit Test Results | clear the checkbox **Publish to Azure Pipelines**, since we will not be running automated tests in the Clients |

1.  On the **securityandcomplianceproj-CI** pane, on the **Tasks** tab, click the plus sign to the right of the **Agent job 1** entry, on the **Add tasks** pane, select the **Utility** tab, in the list of task templates, select **Copy Files**, and then click the **Add** button twice to add two **Copy Files** tasks to the job.
1.  With the **Utility** tab selected, in the **Search** textbox, type **Publish Build Artifacts**, in the list of results, select **Publish Build Artifacts**, and then click the **Add** button twice to add two **Publish Build Artifacts** tasks to the job.
1.  In the list of tasks of **Agent job 1**, select the first **Copy Files to:** task, specify the following settings (leave others with their default values):

    | Setting | Value |
    | ------- | ----- |
    | Display name | **Copy Files to $(build.artifactstagingdirectory)\drop** |
    | Source Folder | **$(Build.SourcesDirectory)\src** |
    | Contents | **\*/build/libs/!(buildSrc).?ar** |
    | Target Folder | **$(build.artifactstagingdirectory)\drop** |

    >**Note**: We will copy the files from the build and repo folders into a staging area on the agent, to be later picked up from there and published as build pipeline artifacts, which can be later used in a release pipeline. 

    >**Note**: To view more details about the variables that we are using in the tasks you can take a look at the page Predefined build variables

1.  In the list of tasks of **Agent job 1**, select the second **Copy Files to:** task, specify the following settings (leave others with their default values):

    | Setting | Value |
    | ------- | ----- |
    | Display name | **Copy Files to $(build.artifactstagingdirectory)** |
    | Source Folder | **$(Build.SourcesDirectory)** |
    | Contents | **\*\*/deploy/SSH-MRP-Artifacts.ps1**, **\*\*/deploy/deploy_mrp_app.sh** and **\*\*/deploy/MongoRecords.js** |
    | Target Folder | **$(build.artifactstagingdirectory)** |

    >**Note**: Enter each of the **Contents** entries on a separate line and verify that each of them includes the double asterisk prefix.

1.  In the list of tasks of **Agent job 1**, select the first **Publish Artifact** task, specify the following settings (leave others with their default values):

    | Setting | Value |
    | ------- | ----- |
    | Path to publish | **$(build.artifactstagingdirectory)\drop** |
    | Artifact Name | **drop** |
    | Artifact publish location | **Azure Pipelines** |

    >**Note**: In these publish tasks, we are taking the items from the agent staging area and publishing them as build pipeline artifacts for later use in a release pipeline. We will not create and deploy using our release pipeline in this lab but you have the option of retaining this build pipeline for your own tests.

1.  In the list of tasks of **Agent job 1**, select the first **Publish Artifact** task, specify the following settings (leave others with their default values):

    | Setting | Value |
    | ------- | ----- |
    | Path to publish | **$(build.artifactstagingdirectory)\deploy** |
    | Artifact Name | **deploy** |
    | Artifact publish location | **Azure Pipelines** |

1.  On the **securityandcomplianceproj-CI** pane, click **Save and Queue**, in the dropdown list, click **Save and Queue**, and, on the **Run pipeline** pane, click **Save and run**.
1.  On the build run blade, in the **Jobs** section, click **Agent job 1**, monitor the build progress, and ensure it completes successfully before you proceed to the next task.

#### Task 2: Install and activate WhiteSource Bolt in the Azure DevOps project

In this task, you will install and activate WhiteSource Bolt by using the Azure DevOps marketplace.

1.  On your lab computer, in the web browser window displaying the Azure DevOps portal with the **securityandcomplianceproj** project open, in the vertical menu bar at the far left of the Azure DevOps portal, click **Pipelines**.
1.  On the **Pipelines** pane, click **securityandcomplianceproj-CI** entry and then click **Edit**. 
1.  On the **securityandcomplianceproj-CI** pane, in the list of tasks, click the plus sign to the right of the **Agent job 1** entry, on the **Add tasks** pane, select the **Marketplace** tab, in the **Search** textbox, type **WhiteSource Bolt**, in the list of results, select **WhiteSource Bolt**, and then click **Get it free**. This will automatically open a new web browser tab, displaying the **WhiteSource Bolt** page.

    >**Note**: WhiteSource is available for build servers, while WhiteSource Bolt is intended specifically for *Azure DevOps**.

1.  On the **WhiteSource Bolt** page, click **Get it free**, when prompted, in the **Select an Azure DevOps organization** dropdown list, select your Azure DevOps organization, click **Install**, and click **Proceed to organization**. This will redirect your web browser to the Azure DevOps portal, displaying your organization's pane.
1.  In the web browser window displaying the Azure DevOps portal, click the **securityandcomplianceproj** project tile and, in the vertical menu bar at the far left of the Azure DevOps portal, click **Pipelines**.

    >**Note**: The **Pipelines** section should include an extra entry labeled **WhiteSource Bolt**.

1.  In the vertical menu bar at the far left of the Azure DevOps portal, within the **Pipelines** section, click **WhiteSource Bolt**.
1.  When prompted, provide your email address and company name, in the **Country** dropdown list, select your country, and click **Get Started** to activate the WhiteSource Bolt extension.

#### Task 3: Add WhiteSource Bolt as a build task in our build pipeline

In this task, you will add WhiteSource Bolt as a build task in our build pipeline

1.  Switch back to the web browser tab displaying the Azure DevOps portal with the **securityandcomplianceproj-CI** pane open, and, on the pane displaying the **WhiteSource Bolt** task on the right side of the **Tasks** tab, click **Refresh**. 

    >**Note**: Following the refresh, you should be able to add the **WhiteSource Bolt** task to the build pipeline.

1.  Select the **WhiteSource Bolt** task to reveal the **Add** button in its lower right corner and click **Add**.
1.  Use the mouse pointer to drag the newly added **WhiteSource Bolt** task from the bottom of the task list and drop it directly after the **Clients** **Gradle** task. 
1.  Review the properties of the **WhiteSource Bolt** task without making any changes, in the top menu, click **Save & queue** menu header, in the drop-down menu, click **Save** and, in the **Save build pipeline** pop-up window, click **Save**.

#### Task 4: Run our build pipeline and view WhiteSource Bolt security and compliance report

In this task, you will run our build pipeline and view WhiteSource Bolt security and compliance report.

1.  On the **securityandcomplianceproj-CI** pane, in the top menu, click **Queue** and, on the **Run pipeline** pane, click **Run**.
1.  On the build run blade, in the **Jobs** section, click **Agent job 1**, monitor the build progress, and ensure it completes successfully, including the newly added WhiteSource Bolt task. 
1.  Once the build completes, in the Azure DevOps portal, in the vertical menu bar at the far left of the Azure DevOps portal, within the **Pipelines** section, click **WhiteSource Bolt**.
1.  In the **WhiteSource Bolt** review the auto-generated report, including the following sections:

- Vulnerability Score
- Vulnerable Libraries
- Severity Distribution
- Aging Vulnerable Libraries
- License Risks and Compliance, with the associated Risk level
- Outdated libraries
- Inventory

    >**Note**: There are two occurrences of **Suspected Apache 2.0** with **Unknown** risk level. You have the option of exporting the report.


## Review

In this lab, you created a new Azure DevOps project, populated the project repository with a sample application code, created a build pipeline, installed WhiteSource Bolt from the Azure DevOps Marketplace to make it available as a build task, activated it, added it to the build pipeline, used it to scan the project code for security vulnerabilities and licensing compliance issues, and finally viewed the resulting report.
