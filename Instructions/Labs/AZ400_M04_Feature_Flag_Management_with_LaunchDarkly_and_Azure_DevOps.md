---
lab:
    title: 'Lab 10: Feature Flag Management with LaunchDarkly and Azure DevOps'
    module: 'Module 04: Design and implement a release strategy'
---

# Lab 10: Feature Flag Management with LaunchDarkly and Azure DevOps
# Student lab manual

## Lab overview

[**LaunchDarkly**](https://launchdarkly.com/) is a continuous delivery platform that provides feature flags as a service. LaunchDarkly gives you the power to separate feature rollout from code deployment and manage feature flags at scale. Integration of LaunchDarkly with Azure DevOps minimizes potential risks associated with frequent releases. To further integrate releases with your development process, you can link feature flag roll-outs to Azure DevOps work items. 

In this lab, you will learn how to optimize management of feature flags in Azure DevOps by leveraging LaunchDarkly. 

## Objectives

After you complete this lab, you will be able to:

- Create feature flags in LaunchDarkly
- Integrate LaunchDarkly with Web applications
- Automatically roll-out LaunchDarkly feature flags as part of Azure DevOps release pipelines

## Lab duration

-   Estimated time: **60 minutes**

## Instructions

### Before you start

#### Sign in to the lab virtual machine

Ensure that you're signed in to your Windows 10 virtual machine by using the following credentials:
    
-   Username: **Student**
-   Password: **Pa55w.rd**

#### Review applications required for this lab

Identify the applications that you'll use in this lab:
    
-   Microsoft Edge
-   Visual Studio 2019 Community Edition available from [Visual Studio Downloads page](https://visualstudio.microsoft.com/vs). Visual Studio 2019 installation should include **ASP.NET and web development**, **Azure development**, and **.NET Core cross-platform development** workloads. This is already preinstalled on your lab computer.

#### Set up a LaunchDarkly trial account

Use a web browser to navigate to the [LaunchDarkly web site](https://launchdarkly.com/) and create a trial account. 

#### Set up an Azure DevOps organization.

Follow instructions available at [Create an organization or project collection](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops).

#### Prepare an Azure subscription

-   Identify an existing Azure subscription or create a new one.
-   Verify that you have a Microsoft account or an Azure AD account with the Owner role in the Azure subscription and the Global Administrator role in the Azure AD tenant associated with the Azure subscription. For details, refer to [List Azure role assignments using the Azure portal](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-list-portal) and [View and assign administrator roles in Azure Active Directory](https://docs.microsoft.com/en-us/azure/active-directory/roles/manage-roles-portal#view-my-roles).

### Exercise 0: Configure the lab prerequisites

In this exercise, you will set up the prerequisites for the lab, which consist of the preconfigured Parts Unlimited team project based on an Azure DevOps Demo Generator template. 

#### Task 1: Configure the team project

In this task, you will use Azure DevOps Demo Generator to generate a new project based on the **Parts Unlimited** template.

1.  On your lab computer, start a web browser and navigate to [Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net). This utility site will automate the process of creating a new Azure DevOps project within your account that is prepopulated with content (work items, repos, etc.) required for the lab. 

    > **Note**: For more information on the site, see https://docs.microsoft.com/en-us/azure/devops/demo-gen.

1.  Click **Sign in** and sign in using the Microsoft account associated with your Azure DevOps subscription.
1.  If required, on the **Azure DevOps Demo Generator** page, click **Accept** to accept the permission requests for accessing your Azure DevOps subscription.
1.  On the **Create New Project** page, in the **New Project Name** textbox, type **LaunchDarkly**, in the **Select organization** dropdown list, select your Azure DevOps organization, and then click **Choose template**.
1.  In the list of templates, in the toolbar, click **DevOps Labs**, select the **LaunchDarkly** template and click **Select Template**.
1.  Back on the **Create New Project** page, if prompted to install a missing extension, select the checkbox below the **LaunchDarkly Integration V2** label and click **Create Project**

    > **Note**: Wait for the process to complete. This should take about 2 minutes. In case the process fails, navigate to your DevOps organization, delete the project, and try again.

1.  On the **Create New Project** page, click **Navigate to project**.

### Exercise 1: Configure feature flags in Azure DevOps by using LaunchDarkly

In this exercise, you will configure feature flags in Azure DevOps by using LaunchDarkly. 

#### Task 1: Create a feature flag in LaunchDarkly

In this task, you will create a feature flag in LaunchDarkly

1.  From your lab computer, start a web browser, navigate to the [LaunchDarkly web site](https://app.launchdarkly.com/), and create a user account. Your browser session will be redirected to the **Default Project** portal where you can create a feature flag. 
1.  In the LaunchDarkly portal, in the vertical menu bar on the left, click **Feature flags**.
1. On the **Feature flags** pane, click **Create FLAG**.
1.  On the **Create a feature flag** pane, in the **Name** text box, type **Member portal** and click the **SAVE FLAG** button.

    > **Note**: You've created a flag named **Member Portal**. Let's assume that you want to use this flag to determine the visibility of the **Member Portal** feature in your ASP.NET MVC web app. 

    > **Note**: To integrate LaunchDarkly into your application, you need an SDK key. 

1.  In the LaunchDarkly portal, in the vertical menu bar on the left, click **Account settings** and then click the **Projects** tab.

    > **Note**: On the **Projects** tab of the **Account settings** pane, you will find two predefined environments: **Production** and **Test**.  You can use the production environment SDK key for this project. 

1.  Record the SDK key for the production environment. You will need it later in this lab.

#### Task 2: Integrate LaunchDarkly in your Web application

In this task, you will integrate LaunchDarkly in your Web application.

1.  On your lab computer, switch to the web browser window displaying the Azure DevOps portal with the **LaunchDarkly** project open and, in the lower left corner of the project pane, click **Project settings**.
1.  In the vertical menu bar titled **Project Settings**, in the **Repos** section, select **Repositories**.
1.  On the **All Repositories** pane, click **LaunchDarkly**, on the **Repository Settings** pane, verify that the **Commit mention linking** and **Commit mention work item resolution** settings are set to **On**.
1.  In the vertical menu bar at the far left of the Azure DevOps portal, click **Repos** and verify that you are viewing the **Files** pane.

    > **Note**: In case the repo is empty, in the **Import a repository** section, select **Import**, on the **Import a Git repository** pane, in the **Clone URL** text box, enter `https://github.com/hsachinraj/PartsUnlimited.git`, and click **Import**.

1.  On the **Files** pane, click **Clone**.
1. On the **Clone Repository** pane, in the **IDE** drop-down list, select **Visual Studio**. If prompted, select **Open Microsoft Visual Studio Web Protocol Handler Selector**. This will automatically launch Visual Studio. Sign in using the Microsoft account associated with your Azure DevOps subscription. 
1.  Within the Visual Studio window, in the **Azure DevOps** dialog box, click **Clone** and, if prompted, sign in using the Microsoft account associated with your Azure DevOps subscription. 
1.  Within the Visual Studio window, click the top level **Git** menu, click **Local repositories**, click **Folder**, in the **Select Folder**, navigate to the local folder into which you cloned the LaunchDarkly repository, and click **Select Folder**.
1.  Within the Visual Studio window, click the top level **View** menu and, in the dropdown menu, click **Git Changes**. 
1. Within the Visual Studio window, at the top of the **Git Changes** pane, in the **master** dropdown list, click the down-pointing arrow head, in the drop-down dialog box, click **Remotes**, and, in the list of remote branches, select **origin/launch-darkly**. Select **Check out**. 

    > **Note**: This will automatically check out the **launch-darkly** branch. 

1.  Within the Visual Studio window, switch to the **Solution Explorer** window, and double-click **PartsUnlimited.sln** file to open the solution. 

    > **Note**: To integrate **LaunchDarkly** with .NET applications you need to install the NuGet package with the **LaunchDarkly client**. In the current project, that package has already been added for the ease of use.

1.  In the **Solution Explorer** pane, expand the **PartsUnlimitedWebsite** node, right-click the **Dependencies** subnode, and, in the right-click menu, select the **Manage NuGet Packages** entry.
1.  In the **NuGet: PartsUnlimitedWebsite** pane, note that **LaunchDarkly.Client** is already installed.
1.  In the Visual Studio interface, in the top menu, click **IIS Express** to launch the application locally. 
1.  Verify that the application launches successfully in the local browser session and that the **Member Portal** section is present in the upper right corner of the web interface.

    > **Note**: Let's assume that this **Member Portal** module is a new feature and you would like to control it by using **LaunchDarkly feature flags**. This way, when you turn on the flag in LaunchDarkly, the feature should be visible to users.

1.  Close the web browser window displaying the web application interface.
1.  In the Visual Studio window, in the **Solution Explorer**, navigate to and open **PartsUnlimitedWebsite\\Controllers\\HomeController.cs**, replace its content with the code available at [the updated HomeController.cs code snippet](https://raw.githubusercontent.com/Microsoft/azuredevopslabs/master/labs/vstsextend/launchdarkly/codesnippet/HomeController.cs), and save the change. 
1.  In the Visual Studio window, in the **Solution Explorer**, navigate to and open **PartsUnlimitedWebsite\\Controllers\\AccountController.cs**, replace its content with the code available at [the updated AccountController.cs code snippet](https://raw.githubusercontent.com/Microsoft/azuredevopslabs/master/labs/vstsextend/launchdarkly/codesnippet/AccountController.cs), and save the change. 
1.  In the Visual Studio window, in the **Solution Explorer**, navigate to and open **PartsUnlimitedWebsite\\Views\Shared\\_Layout.chtml** and replace line 55 `@await Html.PartialAsync("_Login")` with the below code.

    ```cshtml
    @if (ViewBag.togglevalue == true)
    {
      @await Html.PartialAsync("_Login")
    }
    ```

1.  Switch to the tab displaying the content of the **Home Controller.cs** file and replace the placeholder `__YourLaunchDarklySDKKey__` in the line `static LdClient client = new LdClient("__YourLaunchDarklySDKKey__");` with the LaunchDarkly account SDK key which you copied in previous task of this exercise. 

    > **Note**: This will create an LdClient object with your environment-specific SDK key.

1.  Switch to the tab displaying the content of the **AccountController.cs** file and replace the placeholder `__YourLaunchDarklySDKKey__` in the line `static LdClient client = new LdClient("__YourLaunchDarklySDKKey__");` with the LaunchDarkly account SDK key which you copied in previous task of this exercise. 

    > **Note**: The above changes will result in the **HomeController** starting by initializing a static LaunchDarkly client. The methods to view **MemberPortal** are modified to check if the Feature flag toggle in LaunchDarkly is On or Off. The **_Layout.cshtml** page checks the toggle value and renders the MemberPortal link if the flag is turned on. 

1.  In the Visual Studio window, switch back to the **HomeController.cs** tab and review its content, focusing on the code between lines 57 and 74:

    ```
        //LaunchDarkly start
        User user = LaunchDarkly.Client.User.WithKey("administrator@test.com");
        bool value = client.BoolVariation("member-portal", user, false);
        if (value)
        {
          ViewBag.Message = "Your application description page.";
          ViewData["togglevalue"] = value;
          return View(viewModel);
        }
        else
        {
          return View(viewModel);
        }
        // return View(viewModel);

    }
    //LaunchDarkly End
    ```

    > **Note**: When you request a feature flag, you need to pass in a user object. So we are initializing user object in the beginning. This will be used to check whether a user with specified key exists in LaunchDarkly or not. In this sample, we have hardcoded the user value. In real-life scenarios, you could retrieve this value by identifying the logged in user or performing a database lookup.

    > **Note**: Then we are calling the **BoolVariation** method to the check feature flag value in LaunchDarkly. If the flag is true it will set **[ViewData["togglevalue"]** to true which is used in **_Layout.chtml** to view Member Portal module. If it is false it won't show Member Portal module.

    > **Note**: Similarly in **AccountController.cs** we have added LaunchDarkly code to **Login()** method which is responsible to show Login page once you click on **Member portal** icon. If the flag in LaunchDarkly is false, an HttpNotFound error will be returned for the Login page.

1. In the Visual Studio window, click **Save All** and click **IIS Express** to launch the application locally. If the **IIS Express** option isn't available, select the **Browser Link** option from the toolbar and select **Browser Link Dashboard**. From the dashboard select **View in browser**. 
1.  Verify that the **Member portal** link no longer appears in the web browser displaying the Parts Unlimited website, since the **MemberPortal** flag is, at this point, turned off .

    > **Note**: This implements the feature flag control using LaunchDarkly. You could now manually enable the toggle from the LaunchDarkly portal manually. However, in this lab, we will configure it via Azure DevOps Release pipeline using the LaunchDarkly extension. To include feature flags as part of the release process, we will associate this change with an Azure DevOps work item. 

1.  Close the web browser window displaying the web application interface.
1.  In the Visual Studio window, if needed, click the top level **View** menu and, in the dropdown menu, click **Team Explorer**.
1. In the **Team Explorer** pane, on the **Home** page, click **Work Items**. Select the link to view new hub if prompted.
1.  In the **Team Explorer** pane, on the **Work Items** page, note the single work item associated with this branch, including the work item ID. 
1.  Double-click the entry representing the work item. This will automatically open another web browser tab, displaying the work item in the Azure DevOps portal. 
1.  On the work item pane, verify that work item is assigned to you. 
1. Switch back to the Visual Studio window, navigate to the **Git Changes** pane, in the **Enter a message** textbox, type **Integated LaunchDarkly #<work\_item\_ID>**, where **<work\_item\_ID>** represents the ID you noted earlier, click the down-pointing arrow head next to the **Commit All** button, and, in the dropdown list, click **Commit All and Push**. If the **Commit All** is greyed out you can select the up arrow to perform the push. 

#### Task 3: Automatically rollout LaunchDarkly feature flags during release

In this task, you will configure automatic roll out of the LaunchDarkly feature flags during an Azure DevOps release

> **Note**: We need a LaunchDarkly access token to integrate with Azure DevOps services. 

1.  To retrieve a LaunchDarkly access token, switch back to the browser window displaying the LaunchDarkly portal, in the vertical menu on the left side, click **Account settings**, on the **Account settings** pane, click the **Authorization** tab, and, in the **Access tokens** section, click **CREATE TOKEN**.
1.  On the **Create an access token** pane, in the **Name** textbox, type **Member portal**, in the **Role** dropdown list, select **Writer** and click **SAVE TOKEN**.
1.  Back on the **Authorization** tab of the **Account settings** pane, copy the access token and paste it into Notepad. 

    > **Note**: Make sure you copy the token. You will not be able to retrieve it once you leave the current page.

1.  Switch back to the browser window displaying the Azure DevOps portal and, on the **LaunchDarkly** project pane, click **Project settings**. 
1. On the **Project details** pane, in the **Pipelines** section, click **Service connections** and then click **Create service connection**.
1.  On the **New service connection** pane, click **LaunchDarkly** and then click **Next**.
1.  On the **New LaunchDarkly service connection** pane, in the **Access token** text box, paste the access token you retrieved earlier in this task, in the **Service connection name** text box, type **az-400 m12l01 LaunchDarkly**, and click **Save**.
1.  In the Azure DevOps portal, in the vertical menu on the left side, click **Boards** and, on the **Work items** pane, click the work item you assigned to yourself in the previous task.
1.  On the **Implement FeatureFlag Management using LaunchDarkly** work item pane, select the **Launch Darkly** tab in the upper right corner. 
1.  On the **LauchDarkly** tab, in the **Select a feature flag** section, select the **Member portal** feature flag.
1.  In the Azure DevOps portal, in the vertical menu bar on the left side, click **Pipelines** and, in the **Pipelines** section, select **Releases**. 
1.  On the **Pipelines / Releases** pane, select the **LaunchDarkly_CD** pipeline and click **Edit**.
1.  On the **All pipelines / LaunchDarkly_CD** pane, click the **1 job, 3 tasks** link of the **Stage 1** stage to view the tasks in the pipeline.
1.  Verify that the stage contains the following tasks:

    - **Azure Resource Group Deployment**: This task deploys an Azure app service for your PartsUnlimited website by using an ARM template.
    - **LaunchDarkly Rollout**: This task turns on the feature flag in your LaunchDarkly subscription.
    - **Azure App Service Deploy**: This task deploys the PartsUnlimited web app to the Azure app service created in the first task.

1.  In the list of tasks, select the **Azure Resource Group Deployment** task, in the **Azure subscription** dropdown list, click the down-facing caret, in the list of entries, select the Azure subscription you want to use in this lab, and click **Authorize** to configure Azure service connection. If prompted to sign in using the account with the Owner role in the Azure subscription and the Global Administrator role in the Azure AD tenant associated with the Azure subscription.
1.  In the list of tasks, select the **LaunchDarkly Rollout** task and, in the **Account** dropdown list, select the entry representing the LaunchDarkly service connection you created earlier in this task.

    > **Note**: Verify that the **Flag state** is set to **On**. This setting controls the state of the flag during release.

1.  In the list of tasks, select the **Azure App Service Deploy** task, in the **Azure subscription** dropdown list, select the Azure subscription service connection you created previously and click **Save**.
1.  In the Azure DevOps portal, in the upper right corner of the Azure DevOps page, click the **User settings** icon, in the dropdown menu, click **Personal access tokens**, on the **Personal Access Tokens** pane, and click **+ New Token**.
1.  On the **Create a new personal access token** pane, specify the following settings and click **Create** (leave all others with their default values):

    | Setting | Value |
    | --- | --- |
    | Name | **LaunchDarkly and Azure DevOps lab** |
    | Scopes | **Full access** |

1.  On the **Success** pane, copy the value of the personal access token to Clipboard.

    > **Note**: Make sure you copy the token. You will not be able to retrieve it once you close this pane. 

1.  On the **Success** pane, click **Close**.
1.  Navigate back to the **All pipelines / LaunchDarkly_CD** pane, click **Edit**, and click the **Variables** tab. 
1.  In the list of variables, set the value of **launchdarkly-pat** variable to the newly generated personal access token.
1.  In the list of variables, set the value of **launchdarkly-project-name** variable to **LaunchDarkly** and click **Save**.

    > **Note**: This completes configuration of the release pipeline. 

1.  In the Azure DevOps portal, in the vertical menu bar on the left side, in the **Pipelines** section, select **Pipelines**.
1. On the **Pipelines** pane, click the entry representing the **LaunchDarkly-CI** build pipeline, on the **LaunchDarkly-CI** pane, click **Run pipeline**, and, on the **Run pipeline** pane, click **Run**.

    > **Note**: This CI pipeline compiles .Net Core project. Once the build completes successfully a release would be triggered to deploy app and rollout feature flag in LaunchDarkly.

1.  On the build pipeline run pane, in the **Jobs** section, click the **Agent job 1** entry and monitor the progress of the build process.
1.  Once the build completes, in the Azure DevOps portal, in the vertical menu bar on the left side, in the **Pipelines** section, select **Releases**.
1.  On the **Pipelines / Releases** pane, on the **LaunchDarkly_CD** pane, click the **Release-1** entry and monitor the progress of the deployment process.

    > **Note**: Once the deployment completes successfully you should be able to see **MemberPortal** feature flag is turned on in LaunchDarkly dashboard.

1.  From the lab computer, start a web browser, navigate to the [**Azure Portal**](https://portal.azure.com), and sign in with the user account that has the Owner role in the Azure subscription you are using in this lab.
1.  In the Azure portal, search for and select the **App Services** resources and, from the **App Services** blade, navigate to the web app into which you deployed the application via the Azure DevOps release pipeline.
1.  On the web app blade, click **Browse**. This will open another web browser tab, displaying the newly deployed web application.
1.  On the Parts Unlimited web page, verify that the **Member Portal** feature is enabled.

> **Note**: LaunchDarkly offers a number of other features, including:
    
- **User Targeting**: LaunchDarkly targeting lets you turn features on or off for individual users or groups of users. You can use it to roll features out for internal testing, private betas, or usability tests before performing a broader rollout. 
- **Custom targeting rules**: In addition to targeting individual users, LaunchDarkly allows you to target segments of users by constructing custom rules. In other words, you can create custom rules to target users based on any attributes you specify. 
- **Projects and environments to manage your development process**: [Projects](https://docs.launchdarkly.com/docs/projects) allow you to manage multiple different software projects under one LaunchDarkly account. [Environments](https://docs.launchdarkly.com/docs/environments) allow you to manage your feature flags throughout your entire development lifecycle — from local development to QA, staging, and production. 

### Exercise 2: Remove the Azure lab resources

In this exercise, you will remove the Azure resources provisione in this lab to eliminate unexpected charges. 

>**Note**: Remember to remove any newly created Azure resources that you no longer use. Removing unused resources ensures you will not see unexpected charges.

#### Task 1: Remove the Azure lab resources

In this task, you will use the Azure portal to remove the Azure resources provisioned in this lab to eliminate unnecessary charges. 

1.  In the Azure portal, navigate to the App Service instance you deployed earlier in this lab. 
1.  On the App Service instance blade, click the link representing the name of the resource group containing the App Service instance.
1.  On the blade of the resource group containing the App Service instance, click **Delete resource group**, when prompted, provide the name of the resource group, and click **Delete**.

## Review

In this lab, you learned how to optimize management of feature flags in Azure DevOps by leveraging LaunchDarkly. 
