---
lab:
    title: 'Package Management with Azure Artifacts'
    module: 'Module 08: Design and implement a dependency management strategy'
---

# Package Management with Azure Artifacts

## Student lab manual

## Lab requirements

- This lab requires **Microsoft Edge** or an [Azure DevOps supported browser.](https://docs.microsoft.com/azure/devops/server/compatibility)

- **Set up an Azure DevOps organization:** If you don't already have an Azure DevOps organization that you can use for this lab, create one by following the instructions available at [AZ-400 Lab Prerequisites](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

- **Set up the sample EShopOnWeb Project:** If you don't already have the sample EShopOnWeb Project that you can use for this lab, create one by following the instructions available at [AZ-400 Lab Prerequisites](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

- Visual Studio 2022 Community Edition available from [Visual Studio Downloads page](https://visualstudio.microsoft.com/downloads/). Visual Studio 2022 installation should include **ASP<nolink>.NET and web development**, **Azure development**, and **.NET Core cross-platform development** workloads.

## Lab overview

Azure Artifacts facilitate discovery, installation, and publishing NuGet, npm, and Maven packages in Azure DevOps. It's deeply integrated with other Azure DevOps features such as Build, making package management a seamless part of your existing workflows.

## Objectives

After you complete this lab, you will be able to:

- Create and connect to a feed.
- Create and publish a NuGet package.
- Import a NuGet package.
- Update a NuGet package.

## Estimated timing: 40 minutes

## Instructions

### Exercise 0: Configure the lab prerequisites

In this exercise, we want to remind you about validating the lab prerequisites, having both an Azure DevOps Organization ready, as well as having created the EShopOnWeb project. See the instructions above for more details.

#### Task 1: Configuring the EShopOnWeb solution in Visual Studio

In this task, you will configure Visual Studio to prepare for the lab.

1. Ensure that you are viewing the **EShopOnWeb** team project on the Azure DevOps portal.

    > **Note**: You can access the project page directly by navigating to the [https://dev.azure.com/`<your-Azure-DevOps-account-name>`/EShopOnWeb](https://dev.azure.com/`<your-Azure-DevOps-account-name>`/EShopOnWeb) URL, where the `<your-Azure-DevOps-account-name>` placeholder, represents your Azure DevOps Organization name.

2. In the vertical menu on the left side of the **EShopOnWeb** pane, click **Repos**.
3. On the **Files** pane, click **Clone**, select the drop-down arrow next to **Clone in VS Code**, and, in the dropdown menu, select **Visual Studio**.
4. If prompted whether to proceed, click **Open**.
5. If prompted, sign in with the user account you used to set up your Azure DevOps organization.
6. Within the Visual Studio interface, in the **Azure DevOps** pop-up window, accept the default local path (C:\EShopOnWeb) and click **Clone**. This will automatically import the project into Visual Studio.
7. Leave Visual Studio window open for use in your lab.

### Exercise 1: Working with Azure Artifacts

In this exercise, you will learn how to work with Azure Artifacts by using the following steps:

- Create and connect to a feed.
- Create and publish a NuGet package.
- Import a NuGet package.
- Update a NuGet package.

#### Task 1: Creating and connecting to a feed

In this task, you will create and connect to a feed.

1. In the web browser window displaying your project settings in the Azure DevOps portal, in the vertical navigational pane, select **Artifacts**.
2. With the **Artifacts** hub displayed, click **+ Create feed** at the top of the pane.

    > **Note**: This feed will be a collection of NuGet packages available to users within the organization and will sit alongside the public NuGet feed as a peer. The scenario in this lab will focus on the workflow for using Azure Artifacts, so the actual architectural and development decisions are purely illustrative.  This feed will include common functionality that can be shared across projects in this organization.

3. On the **Create new feed** pane, in the **Name** textbox, type **EShopOnWebShared**, in the **Scope** section, select the **Organization** option, leave other settings with their default values, and click **Create**.

    > **Note**: Any user who wants to connect to this NuGet feed must configure their environment.

4. Back on the **Artifacts** hub, click **Connect to feed**.
5. On the **Connect to feed** pane, in the **NuGet** section, select **Visual Studio** and, on the **Visual Studio** pane, copy the **Source** url. (https://pkgs.dev.azure.com/<Azure-DevOps-Org-Name>_packaging/EShopOnWebShared/nuget/v3/index.json)
6. Switch back to the **Visual Studio** window.
7. In the Visual Studio window, click **Tools** menu header, in the dropdown menu, select **NuGet Package Manager** and, in the cascading menu, select **Package Manager Settings**.
8. In the **Options** dialog box, click **Package Sources** and click the plus sign to add a new package source.
9. At the bottom of the dialog box, in the **Name** textbox, replace **Package source** with **EShopOnWebShared** and, in the **Source** textbox, paste the URL you copied in the Azure DevOps portal.
10. Click **Update** and then click **OK** to finalize the addition.

    > **Note**: Visual Studio is now connected to the new feed.

#### Task 2: Creating and publishing an in-house developed NuGet package

In this task, you will create and publish an in-house developed custom NuGet package.

1. In the Visual Studio window you used to configure the new package source, in the main menu, click **File**, in the dropdown menu, click **New** and then, in the cascading menu, click **Project**.

    > **Note**: We will now create a shared assembly that will be published as a NuGet package so that other teams can integrate it and stay up to date without having to work directly with the project source.

2. On the **Recent project templates** page of the **Create a new project** pane, use the search textbox to locate the **Class Library** template, select the template for C#, and click **Next**.
3. On the **Class Library** page of the **Create a new project** pane, specify the following settings and click **Create**:

    | Setting | Value |
    | --- | --- |
    | Project name | **EShopOnWeb.Shared** |
    | Location | accept the default value |
    | Solution | **Create new solution** |
    | Solution name | **EShopOnWeb.Shared** |

    Leave the setting **Place solution and project in the same directory** enabled.

4. Click Next. Accept **.NET 6.0 (Long Term Support)** as Framework option.
5. Confirm the project creation by pressing the **Create** button.
6. Within the Visual Studio interface, in the **Solution Explorer** pane, right-click **Class1.cs**, in the right-click menu, select **Delete**, and, when prompted for confirmation, click **OK**.
7. Press **Ctrl+Shift+B** or **Right-click on the EShopOnWeb.Shared Project** and select **Build** to build the project.

    > **Note**: In the next task we'll use **NuGet.exe** to generate a NuGet package directly from the built project, but it requires the project to be built first.

8. Switch to the web browser displaying the Azure DevOps portal.
9. Navigate to the **Connect to feed** pane, in the **NuGet** section and select **NuGet.exe**. This will display the **NuGet.exe** pane.
10. On the **NuGet.exe** pane, click **Get the tools**.
11. On the **Get the tools** pane, click the **Download the latest NuGet** link. This will automatically open another browser tab displaying the **Available NuGet Distribution Versions** page.
12. On the **Available NuGet Distribution Versions** page, select **nuget.exe - recommended latest v6.x** and download the executable to the local **EShopOnWeb.Shared Project** folder (If you kept the default folder locations, this should be C:\EShopOnWeb\EShopOnWeb.Shared).
13. Select the **nuget.exe** file, and open its properties by right-clicking on the file, and selecting **Properties** from the context menu.
14. In the Properties context window, from the **General** tab, select **Unblock** under the Security section. Confirm by pressing **Apply** and **OK**.
15. From your lab workstation, open the Start menu, and search for **Windows PowerShell**. Next, in the cascading menu, click **Open Windows PowerShell as administrator**.
16. In the **Administrator: Windows PowerShell** window, navigate to the EShopOnWeb.Shared folder, by executing the following command:

    ```text
    cd c:\EShopOnWeb\EShopOnWeb.Shared
    ```

    Run the following to create a **.nupkg** file from the project.

    > **Note**: This is a shortcut to package the NuGet bits for deployment. NuGet is highly customizable. To learn more, refer to the [NuGet package creation page](https://docs.microsoft.com/nuget/create-packages/overview-and-workflow).

    ```text
    .\nuget.exe pack ./EShopOnWeb.Shared.csproj
    ```

    > **Note**: Disregard any warnings displayed in the **Administrator: Windows PowerShell** window.

    > **Note**: NuGet builds a minimal package based on the information it is able to identify from the project. For example, note that the name is **EShopOnWeb.Shared.1.0.0.nupkg**. That version number was retrieved from the assembly.

17. After the successful creation of the package, run the following to publish the package to the **EShopOnWebShared** feed:

    > **Note**: You need to provide an **API Key**, which can be any non-empty string. We're using **AzDO** here. When prompted, sign in to your Azure DevOps organization.

    ```text
    .\nuget.exe push -source "EShopOnWebShared" -ApiKey AzDO EShopOnWeb.Shared.1.0.0.nupkg
    ```

18. Wait for the confirmation of the successful package push operation.
19. Switch to the web browser window displaying the Azure DevOps portal and, in the vertical navigational pane, select **Artifacts**.
20. On the **Artifacts** hub pane, click the dropdown list in the upper left corner and, in the list of feeds, select the **EShopOnWebShared** entry.

    > **Note**: The **EShopOnWebShared** feed should include the newly published NuGet package.

21. Click the NuGet package to display its details.

#### Task 3: Importing an Open-Source NuGet package to the Azure DevOps Package Feed

Besides developing your own packages, why not using the Open Source NuGet (https://www.nuget.org) DotNet Package library? With a few million packages available, there will always be something useful for your application.

In this task, we will use a generic "Hello World" sample package, but you can use the same approach for other packages in the library.

1. From the same PowerShell window, run the following **nuget** command to install the sample package:

    ```text
    .\nuget install HelloWorld -ExcludeVersion
    ```

2. Check the output of the install process. At the first line, it shows the different Feeds it will try to download the package:

    ```text
    Feeds used:
      https://api.nuget.org/v3/index.json
      https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/eShopOnWeb/_packaging/EShopOnWebPFeed/nuget/v3/index.json
    ```

3. Next, it will show additional output regarding the actual installation process itself.

    ```text
    Installing package 'Helloworld' to 'C:\eShopOnWeb\EShopOnWeb.Shared'.
      GET https://api.nuget.org/v3/registration5-gz-semver2/helloworld/index.json
      OK https://api.nuget.org/v3/registration5-gz-semver2/helloworld/index.json 114ms
    MSBuild auto-detection: using msbuild version '17.5.0.10706' from 'C:\Program Files\Microsoft Visual Studio\2022\Professional\MSBuild\Current\bin'.
      GET https://pkgs.dev.azure.com/pdtdemoworld/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v3/registrations2-semver2/helloworld/index.json
      OK https://pkgs.dev.azure.com/pdtdemoworld/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v3/registrations2-semver2/helloworld/index.json 698ms
    
    Attempting to gather dependency information for package 'Helloworld.1.3.0.17' with respect to project 'C:\eShopOnWeb\EShopOnWeb.Shared', targeting 'Any,Version=v0.0'
    Gathering dependency information took 21 ms
    Attempting to resolve dependencies for package 'Helloworld.1.3.0.17' with DependencyBehavior 'Lowest'
    Resolving dependency information took 0 ms
    Resolving actions to install package 'Helloworld.1.3.0.17'
    Resolved actions to install package 'Helloworld.1.3.0.17'
    Retrieving package 'HelloWorld 1.3.0.17' from 'nuget.org'.
      GET https://api.nuget.org/v3-flatcontainer/helloworld/1.3.0.17/helloworld.1.3.0.17.nupkg
      OK https://api.nuget.org/v3-flatcontainer/helloworld/1.3.0.17/helloworld.1.3.0.17.nupkg 133ms
    Installed HelloWorld 1.3.0.17 from https://api.nuget.org/v3/index.json with content hash 1Pbk5sGihV5JCE5hPLC0DirUypeW8hwSzfhD0x0InqpLRSvTEas7sPCVSylJ/KBzoxbGt2Iapg72WPbEYxLX9g==.
    Adding package 'HelloWorld.1.3.0.17' to folder 'C:\eShopOnWeb\EShopOnWeb.Shared'
    Added package 'HelloWorld.1.3.0.17' to folder 'C:\eShopOnWeb\EShopOnWeb.Shared'
    Successfully installed 'HelloWorld 1.3.0.17' to C:\eShopOnWeb\EShopOnWeb.Shared
    Executing nuget actions took 686 ms
    ```

4. The HelloWorld package got installed in a subfolder **HelloWorld**, under the EShopOnWeb.Shared folder. From the Visual Studio **Solution Explorer**, navigate to the **EShopOnWeb.Shared** Project, and notice the **HelloWorld** subfolder. Click on the little arrow to the left of the subfolder, to open the folder and file list.
5. Notice the **lib** subfolder, having a **signature.p7s** signature file, which proofs the origin of the package. Next, notice the **HelloWorld.nupkg** package file itself.

#### Task 4: Upload the Open-Source NuGet package to Azure Artifacts

Let's consider this package an "approved" package for our DevOps team to reuse, by uploading it to the Azure Artifacts Package feed created earlier.

1. From the PowerShell window, execute the following command:

    ```text
    .\nuget.exe push -source "EShopOnWebShared" -ApiKey AzDO c:\EShopOnWeb\EShopOnWeb.Shared\HelloWorld\HelloWorld.nupkg
    ```

    > **Note**:  This results in an error message:

    ```text
    Response status code does not indicate success: 409 (Conflict - 'HelloWorld 1.3.0.17' cannot be published to the feed because it exists in at least one of the feed's upstream sources. Publishing this copy would prevent you from using 'HelloWorld 1.3.0.17' from 'NuGet Gallery'. For more information, see https://go.microsoft.com/fwlink/?linkid=864880 (DevOps Activity ID: AE08BE89-C2FA-4FF7-89B7-90805C88972C)).
    ```

When you created the Azure DevOps Artifacts Package Feed, by design, it allows for **upstream sources**, such as nuget.org in the dotnet example. However, nothing blocks your DevOps team to create an **"internal-only"** Package Feed.

1. Navigate to the Azure DevOps Portal, browse to **Artifacts**, and select the **EShopOnWebShared** Feed.
2. Click **Search Upstream Sources**
3. In the **Go to an Upstream Package** window, select **NuGet** as Package Type, and enter **HelloWorld** in the search field.
4. Confirm by pressing the **Search** button.
5. This results in a list of all HelloWorld packages with the different versions available.
6. Click the **left arrow key** to return to the **EShopOnWebShared** Feed.
7. Click the cogwheel to open **Feed Settings**. Within the Feed Settings page, select **Upstream Sources**.
8. Notice the different Upstream Package Managers for different development languages. Select **NuGet Gallery** from the list. Press the **Delete** button, followed by pressing the **Save** button.

9. With these saved changes, it will be possible to upload the **HelloWorld** package using the NuGet.exe from the PowerShell Window, by relaunching the following command:

    ```text
     .\nuget.exe push -source "EShopOnWebShared" -ApiKey AzDO c:\EShopOnWeb\EShopOnWeb.Shared\HelloWorld\HelloWorld.nupkg
    ```

    > **Note**: This should now result in a successful upload 

    ```text
    Pushing HelloWorld.nupkg to 'https://pkgs.dev.azure.com/pdtdemoworld/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v2/'...
      PUT https://pkgs.dev.azure.com/<AZUREDEVOPSORGANIZATION>/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v2/
    MSBuild auto-detection: using msbuild version '17.5.0.10706' from 'C:\Program Files\Microsoft Visual Studio\2022\Professional\MSBuild\Current\bin'.
      Accepted https://pkgs.dev.azure.com/pdtdemoworld<AZUREDEVOPSORGANIZATION>/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v2/ 1645ms
    Your package was pushed.
    PS C:\eShopOnWeb\EShopOnWeb.Shared>
    ```

10. From the Azure DevOps Portal, **refresh** the Artifacts Package Feed page. The list of packages shows both the **EShopOnWeb.Shared** custom-developed package, as well as the **HelloWorld** public sourced package.
11. From the Visual Studio **EShopOnWeb.Shared** Solution, right-click the **EShopOnWeb.Shared** Project, and select **Manage NuGet Packages** from the context menu.
12. From the NuGet Package Manager window, validate the **Package Source** is set to **EShopOnWebShared**.
13. Click **Browse**, and wait for the list of NuGet Packages to load.
14. This list will also show both the **EShopOnWeb.Shared** custom-developed package, as well as the **HelloWorld** public sourced package.

## Review

In this lab, you learned how to work with Azure Artifacts by using the following steps:

- Created and connect to a feed.
- Created and publish a NuGet package.
- Imported a custom-developed NuGet package.
- Imported a public-sourced NuGet package.
