---
lab:
    title: 'Lab: Version Controlling with Git in Azure Repos'
    az400Module: 'Module 4: Scaling Git for Enterprise DevOps'
---

# Lab: Version Controlling with Git in Azure Repos
# Student lab manual

## Lab overview

Azure DevOps supports two types of version control, Git and Team Foundation Version Control (TFVC). Here is a quick overview of the two version control systems:

- **Team Foundation Version Control (TFVC)**: TFVC is a centralized version control system. Typically, team members have only one version of each file on their dev machines. Historical data is maintained only on the server. Branches are path-based and created on the server.

- **Git**: Git is a distributed version control system. Git repositories can live locally (such as on a developer's machine). Each developer has a copy of the source repository on their dev machine. Developers can commit each set of changes on their dev machine and perform version control operations such as history and compare without a network connection.

Git is the default version control provider for new projects. You should use Git for version control in your projects unless you have a specific need for centralized version control features in TFVC.

In this lab, you will learn how to work with branches and repositories in Azure DevOps.

## Objectives

After you complete this lab, you will be able to:

-   Work with branches in Azure Repos
-   Work with repositories in Azure Repos

## Lab duration

-   Estimated time: **30 minutes**

## Instructions

### Before you start

#### Sign in to the lab virtual machine

Ensure that you're signed in to your Windows 10 virtual machine by using the following credentials:
    
-   Username: **Admin**
-   Password: **Pa55w.rd**

#### Review the installed applications

Find the taskbar on your Windows 10 desktop. The taskbar contains the icons for the applications that you'll use in this lab:
    
-   Microsoft Edge
-   File Explorer
-   [Visual Studio Code](https://code.visualstudio.com/) with the [C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) extension installed.
-   [Git for Windows](https://gitforwindows.org/) 2.29.2 or later.

#### Set up an Azure DevOps organization 

Follow instructions available at [Create an organization or project collection](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops).

### Exercise 0: Configure the lab prerequisites

In this exercise, you will set up the prerequisites for the lab, which include the preconfigured Parts Unlimited team project based on an Azure DevOps Demo Generator template and a Visual Studio Code configuration.

#### Task 1: Configure the team project

In this task, you will use Azure DevOps Demo Generator to generate a new project based on the **Parts Unlimited** template.

1.  Navigate to [Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net). This utility site will automate the process of creating a new Azure DevOps project within your account that is prepopulated with content (work items, repos, etc.) required for the lab. 

    > **Note**: For more information on the site, see https://docs.microsoft.com/en-us/azure/devops/demo-gen.

1.  Click **Sign in** and sign in using the Microsoft account associated with your Azure DevOps subscription.
1.  If required, on the **Azure DevOps Demo Generator** page, click **Accept** to accept the permission requests for accessing your Azure DevOps subscription.
1.  On the **Create New Project** page, in the **New Project Name** textbox, type **Version Controlling with Git in Azure Repos**, in the **Select organization** dropdown list, select your Azure DevOps organization, and then click **Choose template**.
1.  In the list of templates, locate the **PartsUnlimited** template and click **Select Template**.
1.  Back on the **Create New Project** page, click **Create Project**

    > **Note**: Wait for the process to complete. This should take about 2 minutes. In case the process fails, navigate to your DevOps organization, delete the project, and try again.

1.  On the **Create New Project** page, click **Navigate to project**.

#### Task 2: Configuring Visual Studio Code

In this task, you will configure a Git credential helper to securely store the Git credentials used to communicate with Azure DevOps. If you have already configured a credential helper and Git identity, you can proceed directly to the next task.

1.  On the lab computer, open **Visual Studio Code**. 
1.  In the Visual Studio Code interface, from the main menu, select **Terminal \| New Terminal** to open the **TERMINAL** pane.
1.  In the **TERMINAL** pane, run the following command below to configure the credential helper.

    ```git
    git config --global credential.helper wincred
    ```
1.  In the **TERMINAL** pane, run the following commands to configure a user name and email for Git commits (replace the placeholders in braces with your preferred user name and email):

    ```git
    git config --global user.name "<John Doe>"
    git config --global user.email <johndoe@example.com>
    ```

### Exercise 1: Managing branches from Azure DevOps

In this exercise, you will work with branches by using Azure DevOps.
You can manage your repo branches directly from the Azure DevOps portal, in addition to the functionality available in Visual Studio Code.

#### Task 1: Creating a new branch

In this task, you will create a branch by using the Azure DevOps portal and use fetch it by using Visual Studio Code.

1.  If needed, start a web browser, navigate to your Azure DevOps organization, and open the **Version Controlling with Git in Azure Repos** project you generated in the previous exercise. 

    > **Note**: Alternatively, you can access the project page directly by navigating to the [https://dev.azure.com/`<your-Azure-DevOps-account-name>`/Version%20Controlling%20with%20Git%20in%20Azure%20Repos) URL, where the `<your-Azure-DevOps-account-name>` placeholder, represents your account name. 

1.  In the web browser window, navigate to the **Commits** pane of the project and select **Branches**.
1.  On the **Branches** pane, click **New branch**.
1.  In the **Create a branch** panel, in the **Name** textbox, type **release**, ensure that **master** appears in the **Based on** dropdown list, in the **Work items to link** drop-down list, select one or more available work items, and click **Create**.
1.  Switch to the **Visual Studio Code** window. 
1.  Press **Ctrl+Shift+P** to open the **Command Palette**.
1.  At the **Command Palette** prompt, start typing **Git: Fetch** and select **Git: Fetch** when it becomes visible. This command will update the origin branches in the local snapshot.
1.  In the lower left corner of the Visual Studio Code window, click the **master** entry again.
1.  In the list of branches, select **origin/release**. This will create a new local branch called **release** and check it out.

#### Task 2: Deleting and restoring a branch

In this task, you will use the Azure DevOps portal to delete and restore the branch you created in the previous task.

1.  Switch to the web browser displaying the **Mine** tab of the **Branches** pane in the Azure DevOps portal. 
1.  On the **Mine** tab of the **Branches** pane, hover the mouse pointer over the **release** branch entry to reveal the ellipsis symbol on the right side.
1.  Click the ellipsis, in the pop-up menu, select **Delete branch**, and, when prompted for confirmation, click **Delete**.
1.  On the **Mine** tab of the **Branches** pane, select the **All** tab. 
1.  On the **All** tab of the **Branches** pane, in the **Search branch name** text box, type **release**. 
1.  Review the **Deleted branches** section containing the entry representing the newly deleted branch. 
1.  In the **Deleted branches** section, hover the mouse pointer over the **release** branch entry to reveal the ellipsis symbol on the right side.
1.  Click the ellipsis, in the pop-up menu and select **Restore**.

    > **Note**: You can use this functionality to restore a deleted branch as long as you know its exact name.

#### Task 3: Locking and unlocking a branch

In this task, you will use the Azure DevOps portal to lock and unlock the master branch.

Locking is ideal for preventing new changes that might conflict with an important merge or to place a branch into a read-only state. Alternatively, you can use branch policies and pull requests instead of locking if you just want to ensure that changes in a branch are reviewed before they are merged.

Locking does not prevent cloning of a repo or fetching updates made in the branch into your local repo. If you lock a branch, share with your team the reason for locking it and make sure they know what to do to work with the branch after it is unlocked.

1.  Switch to the web browser displaying the **Mine** tab of the **Branches** pane in the Azure DevOps portal. 
1.  On the **Mine** tab of the **Branches** pane, hover the mouse pointer over the **master** branch entry to reveal the ellipsis symbol on the right side.
1.  Click the ellipsis and, in the pop-up menu, select **Lock**.
1.  On the **Mine** tab of the **Branches** pane, hover the mouse pointer over the **master** branch entry to reveal the ellipsis symbol on the right side.
1.  Click the ellipsis and, in the pop-up menu, select **Unlock**.

#### Task 4: Tagging a release

In this task, you will use the Azure DevOps portal to tag a release in the Azure DevOps Repos.

The product team has decided that the current version of the site should be released as v1.1.0-beta. 

1.  In the vertical navigational pane of the of the Azure DevOps portal, in the **Repos** section, select **Tags**.
1.  In the **Tags** pane, click **New tag**.
1.  In the **Create a tag** panel, in the **Name** text box, type **v1.1.0-beta**, in the **Based on** drop-down list leave the **master** entry selected, in the **Description** text box, type **Beta release v1.1.0** and click **Create**.

    > **Note**: You have now tagged the project at this release. You could tag commits for a variety of reasons and Azure DevOps offers the flexibility to edit and delete them, as well as manage their permissions.

### Exercise 2: Managing repositories

In this exercise, you will use the Azure DevOps portal to create and delete a Git repository in Azure DevOps Repos.

You can create Git repos in team projects to manage your project's source code. Each Git repo has its own set of permissions and branches to isolate itself from other work in your project.

#### Task 1: Creating a new repo from Azure DevOps

In this task, you will use the Azure DevOps portal to create a Git repository in Azure DevOps Repos.

1.  In the web browser displaying the Azure DevOps portal, in the vertical navigational pane, click the plus sign in the upper left corner, directly to the right of the project name and, in the cascading menu, click **New repository**.
1.  In the **Create a repository** pane, in the **Repository type**, leave the default **Git** entry, in the **Repository name** text box, type **New Repo**, leave other settings with their default values, and click **Create**. 

    > **Note**: You have the option to create a file named **README.md**. This would be the default markdown file that is rendered when someone navigates to the repo root with a web browser. Additionally, you can preconfigure the repo with a **.gitignore** file. This file specifies which files, based on naming pattern and/or path, to ignore from source control. There are multiple templates available that include the common patterns and paths to ignore based on the project type you are creating. 

    > **Note**: At this point, your repo is available. You can now clone it with Visual Studio Code or any other git-compatible tool.

#### Task 2: Deleting and renaming Git repos

In this task, you will use the Azure DevOps portal to delete a Git repository in Azure DevOps Repos.

Sometimes you'll have a need to rename or delete a repo, which is just as easy. 

1.  In the web browser displaying the Azure DevOps portal, at the bottom of the vertical navigational pane, click **Project settings**.
1.  In the **Project Settings** vertical navigational pane, scroll down to the **Repos** section and click **Repositories**.
1.  On the **Repositories** tab of the **All Repositories** pane, hover the mouse pointer over the **New Repo** branch entry to reveal the ellipsis symbol on the right side.
1.  Click the ellipsis, in the pop-up menu, select **Delete**, in the **Delete New Repo repository** panel, type **New Repo**, and click **Delete**.

#### Review

In this lab, you used the Azure DevOps portal to manage branches and repositories. 