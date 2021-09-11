---
lab:
    title: 'Lab 18: Integration between Azure DevOps and Teams'
    module: 'Module 18: Implementing System Feedback Mechanisms'
---

# Lab 18: Integration between Azure DevOps and Teams
# Student lab manual

## Lab overview

**[Microsoft Teams](https://teams.microsoft.com/start)** is a hub for teamwork in Office 365. It allows you to manage and use all your team's chats, meetings, files, and apps together in one place. It provides software development teams with a hub for teams, conversations, content and tools from across Office 365 and Azure DevOps.

In this lab, you will implement integration scenarios between Azure DevOps services and Microsoft Teams.

> **Note**: **Azure DevOps Services** integration with Microsoft Teams provides a comprehensive chat and collaborative experience across the development cycle. Teams can easily stay informed of important activities in your Azure DevOps team projects with notifications and alerts on work items, pull requests, code commits, as well as build and release events.

## Objectives

After you complete this lab, you will be able to:

- Integrate Microsoft Teams with Azure DevOps
- Integrate Azure DevOps Kanban boards and Dashboards in Teams
- Integrate Azure Pipelines with Microsoft Teams
- Install the Azure Pipelines app in Microsoft Teams
- Subscribe for Azure Pipelines notifications

## Lab duration

-   Estimated time: **60 minutes**

## Instructions

### Before you start

#### Sign in to the lab virtual machine

Ensure that you're signed in to your Windows 10 computer by using the following credentials:
    
-   Username: **Student**
-   Password: **Pa55w.rd**

#### Review applications required for this lab

Identify the applications that you'll use in this lab:
    
-   Microsoft Edge
-   Microsoft Teams. This will be installed as part of prerequisites for this lab.

#### Set up an Office 365 subscription 

Create a free trial subscription from [Microsoft Teams sign up page](https://teams.microsoft.com/start). 

#### Set up an Azure DevOps organization. 

Follow instructions available at [Create an organization or project collection](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops). When creating the Azure DevOps organization, sign in with the same user account you used to set up the Office 365 subscription.

> **Note**: Your Office 365 subscription and the Azure DevOps organization must share the same Azure Active Directory (Azure AD) tenant.

### Exercise 0: Configure the lab prerequisites

In this exercise, you will set up the prerequisites for the lab, which consist of the preconfigured **Tailwind Traders** team project based on an Azure DevOps Demo Generator template and a team created in Microsoft Teams.

#### Task 1: Configure the team project

In this task, you will use Azure DevOps Demo Generator to generate a new project based on the **Tailwind Traders** template.

1.  On your lab computer, start a web browser and navigate to [Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net). This utility site will automate the process of creating a new Azure DevOps project within your account that is prepopulated with content (work items, repos, etc.) required for the lab. 

    > **Note**: For more information on the site, see https://docs.microsoft.com/en-us/azure/devops/demo-gen.

1.  Click **Sign in** and sign in using the Microsoft account associated with your Azure DevOps subscription.
1.  If required, on the **Azure DevOps Demo Generator** page, click **Accept** to accept the permission requests for accessing your Azure DevOps subscription.
1.  On the **Create New Project** page, click **Choose template**.
1.  In the list of templates, in the toolbar, click **Private**, ensure that the **Local Drive** option is selected, click **Choose File**, in the **Open File**, navigate to the location of labfiles, in the **Allfiles\\18** folder, select **Tailwind Traders.zip**, click open, and then click **Submit**.
1.  Back on the **Create New Project** page, in the **New Project Name** textbox, type **Tailwind Traders** and, in the **Select organization** dropdown list, select your Azure DevOps organization.
1.  On the **Create New Project** page, select the **Yes, I want to fork this repository**, click **Authorize**, when prompted, authenticate using your GitHub credentials, click **Authorize azuredevops-generator-bot**, if prompted to install a missing extension, select the checkbox below the **ARM Outputs** and click **Create Project**.

    > **Note**: Wait for the process to complete. This should take about 2 minutes. In case the process fails, navigate to your DevOps organization, delete the project, and try again.

1.  On the **Create New Project** page, click **Navigate to project**.

#### Task 2: Create a team in Microsoft Teams

In this task, you will create a team in Microsoft Teams.

1.  On the lab computer, start a web browser, navigate to the [Microsoft Teams download page](https://www.microsoft.com/en-us/microsoft-365/microsoft-teams/download-app), and, from there, download and install Microsoft Teams with the default settings. 
1.  On the lab computer, launch **Microsoft Teams** by using the desktop app.

    > **Note**: Alternatively, you can use a web browser and navigate to the [Microsoft Teams launch page](https://teams.microsoft.com/dl/launcher/launcher.html?url=/_%23/l/home/0/0&type=home)

1.  When prompted to sign in, sign in with a user account that is part of the Office 365 subscription and has access to your Azure DevOps organization.
1.  In Microsoft Teams, in the toolbar on the left side of the page, click **Teams** and then, at the bottom of the teams list, click **Join or create a team**.

    >**Note**: A team is a collection of people who gather together around a common goal. 

1.  On the **Join or create team** pane, click **Create team**.
1.  On the **Create a team** panel, click *From scratch** and, on the **What kind of team will this be?** panel, click **Private**
1.  On the **Some quick details about your private team** panel, replace **Give your team a name** with **Tailwind Traders** and click on **Create**.
1.  On the **Add members to Tailwind Traders** panel, click **Skip**.

### Exercise 1: Integrate Azure Boards with Microsoft Teams

In this exercise, you will implement integration between Azure Boards and Microsoft Teams.

#### Task 1: Install and configure Azure Boards app in Microsoft Teams

In this task, you will install and configure Azure Boards app in the newly created team in Microsoft Teams.

1.  In the Microsoft Teams window, in the lower left corner, click the **Apps** icon. This will open the **Apps** pane.
1.  On the **Apps** pane, in the **Search all apps** textbox, type **Azure Boards** and, in the list of apps, click **Azure Boards**.
1.  On the **Azure Boards** panel, click **Add**.
1.  In the **Azure Boards** dropdown list at the top of the Microsoft Teams window, click **Search work items**. This will display the **Azure Boards** pane.
1.  On the **Azure Boards** panel, click the **sign in** link.
1.  When prompted to grant **Work items (full)**, **Project and team (read)**, and **Teams Integration** permissions, in the **Azure Boards Microsoft Teams Integration by Azure DevOps** dialog box, click **Accept**.
1.  Back on the **Azure Boards** panel, click the **set up** link. This will display the **Microsoft Azure DevOps Services - Profile 1** window.
1.  In the **Microsoft Azure DevOps Services - Profile 1** window, in the **Organization** dropdown list, select your Azure DevOps organization and click **Continue**.
1.  In the **Microsoft Azure DevOps Services - Profile 1** window, in the **Project** dropdown list, select **Tailwind Traders** and click **Continue**.
 
    >**Note**: This will display the list of existing work items in the **Tailwind Traders** Azure DevOps project.

1.  Scroll through the list of work items, select any of them, and click on the work item name. You will be prompted to select the app to open the corresponding work item. In the list of apps, select Microsoft Edge and click **OK**. This will automatically open a new web browser window and display the work item details in the Azure DevOps portal.
1.  Close the new browser window to return to Microsoft Teams.

    >**Note**: You also have the ability to create work items directly from the Microsoft Teams interface by selecting the **+ Create work item** option.


1.  Back on the **Azure Boards** panel, click the down facing arrowhead directly to the right of the **Open** button and, in the dropdown list, click the **Add to a team** entry.
1.  On the **Set up Azure boards for a team** panel, in the **Search** text box, type **Tailwind Traders**, in the list of results, select the **Tailwind Traders > General** entry, and click **Set up a bot**.
1.  In the list of posts in the **General** channel of the **Tailwind Traders** team, review the messages posted by the bot, including:

    ```
    Sign in to your Azure Boards account with: @Azure Boards signin
    To see what else you can do, type @Azure Boards help
    ```
1.  In the list of posts in the **General** channel of the **Tailwind Traders** team, select the post titled **Azure Boards**, press the **Enter** key, and review additional messages posted by the bot:

    ```
    link [project url] - Link to a project to create work items and receive notifications
    subscriptions - Add or remove subscriptions for this channel
    addAreapath [area path] - Add an area path from your project to this channel
    signin - Sign in to your Azure Boards account
    signout - Sign out from your Azure Boards account
    unlink - Unlink a project from this channel
    feedback - Report a problem or suggest a feature
    ```

#### Task 2: Add Azure Boards Kanban boards to Microsoft Teams

In this task, you will add Azure Boards Kanban boards to tabs in Microsoft Teams.

> **Note**: Kanban board turns your backlog into an interactive signboard, providing a visual flow of work. As work progresses from idea to completion, you update the items on the board. Each column represents a work stage, and each card represents a user story (blue cards) or a bug (red cards) at that stage of work. You can bring in your teams kanban board or favorite dashboard directly into Microsoft Teams by using Tabs. **Tabs** allow team members to access your service on a dedicated canvas, within a channel or in user's personal app space. You can leverage your existing web app to create a great tab experience within Teams.

1.  On your lab computer, switch to the web browser displaying the **Tailwind Traders** project in the Azure DevOps portal, in the vertical menu bar at the far left of the Azure DevOps portal, click **Boards**, and, in the **Boards** section, click **Boards**.
1.  On the **Boards** pane, in the **My team boards (1)** section, click **Tailwind Traders Team boards** entry. 
1.  While on the **Tailwind Traders Team** pane, in the web browser window, copy its URL into Clipboard.
1.  Switch to the Microsoft Teams window, ensure that the **General** channel of the newly created team **Tailwind Traders** is selected, and, in the upper section of the **General** pane, click the plus sign. This will display the **Add a tab** panel.
1.  On the **Add a tab** panel, click **Website**, on the **Website** panel, set **Tab name** to **Tailwind Traders Team boards**, set the **URL** to the URL you just copied into Clipboards, and then click **Save**.
1.  In the Microsoft Teams window, with the **General** channel of the **Tailwind Traders** team selected, in the list of tabs in the top menu, click the newly added **Tailwind Traders Team boards** tab and ensure that it contains the content matching the **Tailwind Traders Team** board available in the Azure DevOps portal.

> **Note**: All the work can be monitored during the daily standup's and the updates are reflected in real-time, whenever the corresponding work items states change. You also have the option to modify the Kanban board from Microsoft Teams.

### Exercise 2: Integrate Azure Pipelines with Microsoft Teams

In this exercise, you will implement integration between Azure Pipelines and Microsoft Teams.

#### Task 1: Install and configure Azure Pipelines app in Microsoft Teams

In this task, you will install and configure Azure Pipelines app in the designated team in Microsoft Teams.

> **Note**: Azure Pipelines app on Microsoft Teams enables you to monitor the events for your pipelines. You can set up and manage subscriptions for such events as releases, pending approvals, and completed builds, with notifications being posted directly into your Teams channel . You can also approve releases from within the Teams channel.

1.  In the Microsoft Teams window, in the lower left corner, click the **Apps** icon. This will open the **Apps** pane.
1.  On the **Apps** pane, in the **Search all apps** textbox, type **Azure Pipelines** and, in the list of apps, click **Azure Pipelines**.
1.  On the **Azure Pipelines** panel, click the down facing arrowhead directly to the right of the **Open** button and, in the dropdown list, click the **Add to a team** entry.
1.  On the **Set up Azure Pipelines for a team** panel, in the **Search** text box, type **Tailwind Traders**, in the list of results, select the **Tailwind Traders > General** entry, and click **Set up a bot**. You will be redirected automatically to the post view in the **General** channel of the **Tailwind Traders** team.
1.  In the list of posts in the **General** channel of the **Tailwind Traders** team, review the messages posted by the bot, including:

    ```
    Subscribe to one or more pipelines or all pipelines in a project with: @Azure Pipelines subscribe [pipeline url/ project url]
    To see what else you can do, type @Azure Pipelines help
    ```

1.  In the list of posts in the **General** channel of the **Tailwind Traders** team, select the post titled **Azure Pipelines**, press the **Enter** key, and review additional messages posted by the bot:

    ```
    subscribe [pipeline url/ project url] - Subscribe to a pipeline or all pipelines in a project to receive notifications
    subscriptions - Add or remove subscriptions for this channel
    feedback - Report a problem or suggest a feature
    signin - Sign in to your Azure Pipelines account
    signout - Sign out from your Azure Pipelines account
    ```
   
#### Task 2: Subscribe to the Azure Pipeline notifications in Microsoft Teams

In this task, you will subscribe to the Azure Pipeline notifications in Microsoft Teams

> **Note**: You can use the `@Azure Pipelines` handle to start interacting with the app.

1.  With the **Posts** tab selected, in the **General** channel of the **Tailwind Traders** team, post `@Azure Pipelines signin` to authenticate and, when prompted, click **Sign in**.
1.  In the **Azure Pipelines Sign in** pane, click **Sign in**.
1.  If prompted to grant **Service hooks (read and write)**, **Build (build an execute)**, **Release (read, write, execute, and manage)**, **Project and team (read)**, **Identity picker (read)**, and **Teams Integration** permissions, click **Accept** and then click **Close**.

    >**Note**: Now you can run the `@azure pipelines subscribe [pipeline url]` command to subscribe to an Azure DevOps pipeline.

1.  On your lab computer, switch to the web browser displaying the **Tailwind Traders** project in the Azure DevOps portal, in the vertical menu bar at the far left of the Azure DevOps portal, click **Pipelines**, on the **Pipelines** pane, click the **Website-CI** entry, and, while on the **Website-CI** pane, in the web browser window, copy its URL into Clipboard.

    >**Note**: The URL will be in the format `https://dev.azure.com/<organization_name>liveid2530565/Tailwind%20Traders/_build?definitionId=6`, where the `<organization_name>` is the placeholder representing the name of your DevOps organization.

    >**Note**: The pipeline URL can be to any page within your pipeline that has a *definitionId* or *buildId/releaseId* present in the URL.

1.  With the **Posts** tab selected, in the **General** channel of the **Tailwind Traders** team, post `@Azure Pipelines subscribe https://dev.azure.com/<organization_name>/Tailwind%20Traders/_build?definitionId=6` to subscribe to the build pipeline (make sure to replace the `<organization_name>` placeholder with the name of your DevOps organization).
1.  Wait for the confirmation that the subscription has been successfully created.
 
    >**Note**: For Build pipelines, the channel is subscribed to the **Run stage state changed** and **Run stage waiting for approval** notifications.

1.  On your lab computer, switch to the web browser displaying the **Tailwind Traders** project in the Azure DevOps portal, in the vertical menu bar at the far left of the Azure DevOps portal, click **Pipelines**, in the **Pipelines** section, click **Releases**, in the list of releases, click the **Website-CD* entry, and, with the **Website-CD** entry selected, in the web browser window, copy its URL into Clipboard.

    >**Note**: The URL will be in the format `https://dev.azure.com/<organization_name>/Tailwind%20Traders/_release?_a=releases&view=mine&definitionId=2`, where the `<organization_name>` is the placeholder representing the name of your DevOps organization.

1.  With the **Posts** tab selected, in the **General** channel of the **Tailwind Traders** team, post `@Azure Pipelines subscribe https://dev.azure.com/<organization_name>/Tailwind%20Traders/_release?_a=releases&view=mine&definitionId=2` to subscribe to the release pipeline (make sure to replace the `<organization_name>` placeholder with the name of your DevOps organization).

    >**Note**: For Release pipelines, the channel is subscribed to the **Deployment started**, **Deployment completed** and **Deployment approval pending notifications**.
 
#### Task 3: Using filters to customize subscriptions to Azure Pipelines in Microsoft Teams

In this task, you will use customize subscriptions to Azure Pipelines in Microsoft Teams.

>**Note**: When a user subscribes to any pipeline, a few subscriptions are created by default without any filters being applied. Often, users have the need to customize these subscriptions. For example, users may want to get notified only when builds fail or when deployments are pushed to a production environment. The Azure Pipelines app supports filters to customize what you see on your channel.

>**Note**: You can list and manage subscriptions by using `@Azure Pipelines subscriptions` command. This command lists all of the current subscriptions for the channel.

1.  With the **Posts** tab selected, in the **General** channel of the **Tailwind Traders** team, post `@Azure Pipelines subscriptions` command and, in the reply from the **Azure Pipelines** bot, select **Add Subscription**.
1.  In the **Azure Pipelines** **Add subscription** panel, in the **Choose event** dropdown list, ensure that **Build completed** is selected and click **Next**.
1.  In the **Azure Pipelines** **Add subscription** panel, in the **Choose pipeline** dropdown list, ensure that **Website-CI** is selected and click **Next**.
1.  In the **Azure Pipelines** **Add subscription** panel, in the **Build status** dropdown list, ensure that **[Any]** is selected and click **Submit**.
1.  In the **Azure Pipelines** **Add subscription** panel, click **OK** to acknowledge the confirmation message.
1.  In the **Azure Pipelines** **View subscriptions** panel, review the list of subscriptions and close the panel.

### Exercise 3: Review Microsoft Teams collaboration features in DevOps scenarios

In this exercise, you will review collaboration features of Microsoft Teams that provide additional value in DevOps scenarios.

#### Task 1: Review Microsoft Teams conversation capabilities

In this task, you will review some basic Microsoft Teams conversation capabilities.

## Collaboration Experience

>**Note**: Microsoft Teams posts functionality provides a straightforward way to connect and keep a history of the conversation. It includes support for emoji, stickers, and GIFs to enhance interactivity.

1.  To start a conversation, in the Microsoft Teams window, click the **Posts** tab.

    >**Note**: You can easily locate and reference work Azure DevOps items in Teams, keeping the conversation and collaboration within the Teams app. For example if you would like to discuss about any User Story you can search for it, add it to the conversation, and enter your comments.

1.  Right below the post entry, in the list of icons, click the **Boards** icon. This will automatically display the **Azure Boards** pop-up window.
1.  If needed, in the **Azure Boards** pop-up window, click the **set up** link, when prompted, in the **Choose the organization you want to link to Azure Boards** pop-up window, in the **Organization** dropdown list, select your Azure DevOps organization and click **Continue**, in the **Project** dropdown list, select **Tailwind Traders** and click **Continue**.
1.  Back in the post entry, add a reference to a work items into a post.

#### Task 2: Create channels in Microsoft Teams

In this task, you will step through the process of creating channels in Microsoft Teams

>**Note**: **Channels** are dedicated sections within a team to keep conversations organized by specific topics, projects, disciplines, based on your preferences. Team channels are places where everyone on the team can openly have conversations. Private chats are only visible to those people in the chat. Channels provide most benefits when supplemented by apps, such as tabs, connectors, and bots.

1.  In the Microsoft Teams window, locate the **Tailwind Traders** team you created earlier in this lab, click the ellipsis symbol on its right side, and, in the dropdown menu, click **Add channel**.
1.  In the **Create a channel for "Tailwind Traders" team** pop-up window, in the **Channel name** textbox, type **DevOps posts**, leave the **Description (optional)** textbox blank, in the **Privacy** dropdown list, select **Standard - Accessible to everyone on the team** and click **Next**.

#### Task 3: Share content in Microsoft Teams

In this task, you will step through the process of sharing Azure DevOps wikis in Microsoft Teams

>**Note**: As your team works together you'll undoubtedly have files that you'll want to share and collaborate on. Microsoft Teams makes it easy to share files within channels. If the files have been created by standard Microsoft Office applications, such as Word, Excel, PowerPoint, or Visio, you can view, edit, and collaborate on the files directly within Teams. 

1.  In the Microsoft Teams window, with the **DevOps posts** channel selected, click the **Files** tab and note that the toolbar includes the **Upload**, **Sync** and **Download** items. 

    >**Note**: You can also use **Drag and drop** to upload files.

    >**Note**: In addition, you have the option to share Azure DevOps resident content within the Teams as tabs. This includes Azure DevOps Wikis, which document project objectives, epics, specs, release notes and best practices.

1.  On the lab computer, switch to browser window displaying the Azure DevOps portal, in the vertical menu bar at the far left of the Azure DevOps portal, click **Overview**, and, in the **Overview** section, click **Wiki**.
1.  With the **Wiki** menu item selected, click **Publish code as wiki**.
1.  On the **Publish code as wiki** pane, ensure that **TailwindTraders-Website** appears in the dropdown menu, in the **Branch** dropdown list, select **main**, set the **Folder** value to **/Documents**, in the **Wiki name** type **TailWindTraders-Website wiki**, and click **Publish**.
1.  In the web browser window displaying the **TailWindTraders-Website wiki** page, copy its URL into Clipboard.
1.  Switch to the Microsoft Teams window, ensure that the **DevOps posts** channel of the newly created team **Tailwind Traders** is selected, and, in the upper section of the **DevOps posts** pane, click the plus sign. This will display the **Add a tab** panel.
1.  On the **Add a tab** panel, click **Website**, on the **Website** panel, set **Tab name** to **Tailwind Traders DevOps wikis**, set the **URL** to the URL you just copied into Clipboards, and then click **Save**.
1.  In the Microsoft Teams window, with the **DevOps posts** channel of the **Tailwind Traders** team selected, in the list of tabs in the top menu, click the newly added **Tailwind Traders DevOps wikis** tab and ensure that it contains the content matching the **TailWindTraders-Website wiki** wikis available in the Azure DevOps portal.

    >**Note**: Now that you have connected Microsoft Teams and Azure DevOps, consider other types of information that you can expose via Microsoft teams, for example: 

    - [Add a OneNote notebook to Teams](https://support.office.com/en-us/article/Add-a-OneNote-notebook-to-Teams-0ec78cc3-ba3b-4279-a88e-aa40af9865c2) to keep meeting notes such as **Sprint Planning Meetings** and **Retrospective Meetings**. 
    - [Connect Azure DevOps to Power BI ](https://docs.microsoft.com/en-us/azure/devops/report/powerbi/?view=azure-devops) and [add a Power BI tab ](https://support.office.com/en-us/article/add-a-powerbi-tab-to-teams-708ce6fe-0318-40fa-80f5-e9174f841918) that displays advanced reports from Azure DevOps or other data related to your project.

## Review

In this lab, you implemented integration scenarios between Azure DevOps services and Microsoft Teams.
