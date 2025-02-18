---
lab:
  title: "Package Management with Azure Artifacts"
  module: "Module 07: Design and implement a dependency management strategy"
---

# Package Management with Azure Artifacts

## Lab requirements

- This lab requires **Microsoft Edge** or an [Azure DevOps supported browser.](https://docs.microsoft.com/azure/devops/server/compatibility)

- **Set up an Azure DevOps organization:** If you don't already have an Azure DevOps organization that you can use for this lab, create one by following the instructions available at [AZ-400 Lab Prerequisites](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

- **Set up the sample eShopOnWeb Project:** If you don't already have the sample eShopOnWeb Project that you can use for this lab, create one by following the instructions available at [AZ-400 Lab Prerequisites](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html).

- Visual Studio 2022 Community Edition available from [Visual Studio Downloads page](https://visualstudio.microsoft.com/downloads/). Visual Studio 2022 installation should include **ASP<nolink>.NET and web development**, **Azure development**, and **.NET Core cross-platform development** workloads.

- **.NET Core SDK:** [Download and install the .NET Core SDK (2.1.400+)](https://go.microsoft.com/fwlink/?linkid=2103972)

- **Azure Artifacts credential provider:** [Download and install the credential provider](https://go.microsoft.com/fwlink/?linkid=2099625).

## Lab overview

Azure Artifacts facilitate discovery, installation, and publishing NuGet, npm, and Maven packages in Azure DevOps. It's deeply integrated with other Azure DevOps features such as Build, making package management a seamless part of your existing workflows.

## Objectives

After you complete this lab, you will be able to:

- Create and connect to a feed.
- Create and publish a NuGet package.
- Import a NuGet package.
- Update a NuGet package.

## Estimated timing: 35 minutes

## Instructions

### Exercise 0: Configure the lab prerequisites

In this exercise, you will set up the prerequisites for the lab.

#### Task 1: (skip if done) Create and configure the team project

In this task, you will create an **eShopOnWeb** Azure DevOps project to be used by several labs.

1. On your lab computer, in a browser window open your Azure DevOps organization. Click on **New Project**. Give your project the name **eShopOnWeb** and leave the other fields with defaults. Click on **Create**.

   ![Screenshot of the create new project panel.](images/create-project.png)

#### Task 2: (skip if done) Import eShopOnWeb Git Repository

In this task you will import the eShopOnWeb Git repository that will be used by several labs.

1. On your lab computer, in a browser window open your Azure DevOps organization and the previously created **eShopOnWeb** project. Click on **Repos > Files** , **Import a Repository**. Select **Import**. On the **Import a Git Repository** window, paste the following URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> and click **Import**:

   ![Screenshot of the import repository panel.](images/import-repo.png)

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

#### Task 4: Configuring the eShopOnWeb solution in Visual Studio

In this task, you will configure Visual Studio to prepare for the lab.

1. Ensure that you are viewing the **eShopOnWeb** team project on the Azure DevOps portal.

   > **Note**: You can access the project page directly by navigating to the [https://dev.azure.com/`<your-Azure-DevOps-account-name>`/eShopOnWeb](https://dev.azure.com/`<your-Azure-DevOps-account-name>`/eShopOnWeb) URL, where the `<your-Azure-DevOps-account-name>` placeholder, represents your Azure DevOps Organization name.

1. In the vertical menu on the left side of the **eShopOnWeb** pane, click **Repos**.
1. On the **Files** pane, click **Clone**, select the drop-down arrow next to **Clone in VS Code**, and, in the dropdown menu, select **Visual Studio**.
1. If prompted whether to proceed, click **Open**.
1. If prompted, sign in with the user account you used to set up your Azure DevOps organization.
1. Within the Visual Studio interface, in the **Azure DevOps** pop-up window, accept the default local path (C:\eShopOnWeb) and click **Clone**. This will automatically import the project into Visual Studio.
1. Leave Visual Studio window open for use in your lab.

### Exercise 1: Working with Azure Artifacts

In this exercise, you will learn how to work with Azure Artifacts by using the following steps:

- Create and connect to a feed.
- Create and publish a NuGet package.
- Import a NuGet package.
- Update a NuGet package.

#### Task 1: Creating and connecting to a feed

In this task, you will create and connect to a feed.

1. In the web browser window displaying your project settings in the Azure DevOps portal, in the vertical navigational pane, select **Artifacts**.
1. With the **Artifacts** hub displayed, click **+ Create feed** at the top of the pane.

   > **Note**: This feed will be a collection of NuGet packages available to users within the organization and will sit alongside the public NuGet feed as a peer. The scenario in this lab will focus on the workflow for using Azure Artifacts, so the actual architectural and development decisions are purely illustrative. This feed will include common functionality that can be shared across projects in this organization.

1. On the **Create new feed** pane, in the **Name** textbox, type **`eShopOnWebShared`**. In the **Visibility** section, select Specific people, and in the **Scope** section, select the **Project:eShopOnWeb** option, leave other settings with their default values, and click **Create**.

   > **Note**: Any user who wants to connect to this NuGet feed must configure their environment.

1. Back on the **Artifacts** hub, click **Connect to feed**.
1. On the **Connect to feed** pane, in the **NuGet** section, select **Visual Studio** and, on the **Visual Studio** pane, copy the **Source** url. `https://pkgs.dev.azure.com/Azure-DevOps-Org-Name/_packaging/eShopOnWebShared/nuget/v3/index.json`
1. Switch back to the **Visual Studio** window.
1. In the Visual Studio window, click **Tools** menu header, in the dropdown menu, select **NuGet Package Manager** and, in the cascading menu, select **Package Manager Settings**.
1. In the **Options** dialog box, click **Package Sources** and click the plus sign to add a new package source.
1. At the bottom of the dialog box, in the **Name** textbox, replace **Package source** with **eShopOnWebShared** and, in the **Source** textbox, paste the URL you copied in the Azure DevOps portal.
1. Click **Update** and then click **OK** to finalize the addition.

   > **Note**: Visual Studio is now connected to the new feed.

#### Task 2: Creating and publishing an in-house developed NuGet package

In this task, you will create and publish an in-house developed custom NuGet package.

1. In the Visual Studio window you used to configure the new package source, in the main menu, click **File**, in the dropdown menu, click **New** and then, in the cascading menu, click **Project**.

   > **Note**: We will now create a shared assembly that will be published as a NuGet package so that other teams can integrate it and stay up to date without having to work directly with the project source.

1. On the **Recent project templates** page of the **Create a new project** pane, use the search textbox to locate the **Class Library** template, select the template for C#, and click **Next**.
1. On the **Class Library** page of the **Create a new project** pane, specify the following settings and click **Create**:

   | Setting       | Value                    |
   | ------------- | ------------------------ |
   | Project name  | **eShopOnWeb.Shared**    |
   | Location      | accept the default value |
   | Solution      | **Create new solution**  |
   | Solution name | **eShopOnWeb.Shared**    |

   Check the checkbox to **Place solution and project in the same directory**.

1. Click Next. Accept **.NET 8** as Framework option.
1. Confirm the project creation by pressing the **Create** button.
1. Within the Visual Studio interface, in the **Solution Explorer** pane, right-click **Class1.cs**, in the right-click menu, select **Delete**, and, when prompted for confirmation, click **OK**.
1. Press **Ctrl+Shift+B** or **Right-click on the EShopOnWeb.Shared Project** and select **Build** to build the project.
1. From your lab workstation, open the Start menu, and search for **Windows PowerShell**. Next, in the cascading menu, click **Open Windows PowerShell as administrator**.
1. In the **Administrator: Windows PowerShell** window, navigate to the eShopOnWeb.Shared folder, by executing the following command:

   ```powershell
   cd c:\eShopOnWeb\eShopOnWeb.Shared
   ```

   > **Note**: The **eShopOnWeb.Shared** folder is the location of the **eShopOnWeb.Shared.csproj** file. If you chose a different location or project name, navigate to that location instead.

1. Run the following to create a **.nupkg** file from the project (change the value of `XXXXXX` placeholder with a unique string).

   ```powershell
   dotnet pack .\eShopOnWeb.Shared.csproj -p:PackageId=eShopOnWeb-XXXXX.Shared
   ```

   > **Note**: The **dotnet pack** command builds the project and creates a NuGet package in the **bin\Release** folder. If you don't have a **Release** folder, you can use the **Debug** folder instead.

   > **Note**: Disregard any warnings displayed in the **Administrator: Windows PowerShell** window.

   > **Note**: dotnet pack builds a minimal package based on the information it is able to identify from the project. The argument `-p:PackageId=eShopOnWeb-XXXXXX.Shared` allows you to create a package with a specific name instead using the name contains in the project. For example, if you substitute the string `12345` to the `XXXXXX` placeholder, the name  of the package will be **eShopOnWeb-12345.Shared.1.0.0.nupkg**. The version number was retrieved from the assembly.

1. In the PowerShell window, run the following command to open the **bin\Release** folder:

   ```powershell
   cd .\bin\Release
   ```

1. Run the following to publish the package to the **eShopOnWebShared** feed. Replace the source with the URL you copied earlier from the Visual Studio **Source** url `https://pkgs.dev.azure.com/Azure-DevOps-Org-Name/_packaging/eShopOnWebShared/nuget/v3/index.json`

   ```powershell
   dotnet nuget push --source "https://pkgs.dev.azure.com/Azure-DevOps-Org-Name/_packaging/eShopOnWebShared/nuget/v3/index.json" --api-key az "eShopOnWeb.Shared.1.0.0.nupkg"
   ```

   > **Important**: You need to install the credential provider for your operating system to be able to authenticate with Azure DevOps. You can find the installation instructions at [Azure Artifacts Credential Provider](https://go.microsoft.com/fwlink/?linkid=2099625). You can install by running the following command in the PowerShell window: `iex "& { $(irm https://aka.ms/install-artifacts-credprovider.ps1) } -AddNetfx"`

   > **Note**: You need to provide an **API Key**, which can be any non-empty string. We're using **az** here. When prompted, sign in to your Azure DevOps organization.

   > **Note**: If the prompt does not appear, or you receive the warning **warn : The plugin credential provider could not acquire credentials. Authentication may require manual action. Consider re-running the command with --interactive for `dotnet`, /p:NuGetInteractive=true for MSBuild or removing the -NonInteractive switch for NuGet"**, you can add the **--interactive** parameter to the command.

1. Wait for the confirmation of the successful package push operation.
1. Switch to the web browser window displaying the Azure DevOps portal and, in the vertical navigational pane, select **Artifacts**.
1. On the **Artifacts** hub pane, click the dropdown list in the upper left corner and, in the list of feeds, select the **eShopOnWebShared** entry.

   > **Note**: The **eShopOnWebShared** feed should include the newly published NuGet package.

1. Click the NuGet package to display its details.

#### Task 3: Importing an Open-Source NuGet package to the Azure DevOps Package Feed

Besides developing your own packages, why not using the Open Source NuGet (<https://www.nuget.org>) DotNet Package library? With a few million packages available, there will always be something useful for your application.

In this task, we will use a generic "Newtonsoft.Json" sample package, but you can use the same approach for other packages in the library.

1. From the same PowerShell window used in the previous task to push the new package, navigate back to the **eShopOnWeb.Shared** folder (`cd..`), and run the following **dotnet** command to install the sample package:

   ```powershell
   dotnet add package Newtonsoft.Json
   ```

1. Check the output of the install process. It shows the different Feeds it will try to download the package:

   ```powershell
   Feeds used:
     https://api.nuget.org/v3/registration5-gz-semver2/newtonsoft.json/index.json
     https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/eShopOnWeb/_packaging/eShopOnWebShared/nuget/v3/index.json
   ```

1. Next, it will show additional output regarding the actual installation process itself.

   ```powershell
   Determining projects to restore...
   Writing C:\Users\AppData\Local\Temp\tmpxnq5ql.tmp
   info : X.509 certificate chain validation will use the default trust store selected by .NET for code signing.
   info : X.509 certificate chain validation will use the default trust store selected by .NET for timestamping.
   info : Adding PackageReference for package 'Newtonsoft.Json' into project 'c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj'.
   info :   GET https://api.nuget.org/v3/registration5-gz-semver2/newtonsoft.json/index.json
   info :   OK https://api.nuget.org/v3/registration5-gz-semver2/newtonsoft.json/index.json 124ms
   info : Restoring packages for c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj...
   info :   GET https://api.nuget.org/v3/vulnerabilities/index.json
   info :   OK https://api.nuget.org/v3/vulnerabilities/index.json 84ms
   info :   GET https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/vulnerability.base.json
   info :   GET https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/2024.02.17.11.23.35/vulnerability.update.json
   info :   OK https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/vulnerability.base.json 14ms
   info :   OK https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/2024.02.17.11.23.35/vulnerability.update.json 30ms
   info : Package 'Newtonsoft.Json' is compatible with all the specified frameworks in project 'c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj'.
   info : PackageReference for package 'Newtonsoft.Json' version '13.0.3' added to file 'c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj'.
   info : Writing assets file to disk. Path: c:\eShopOnWeb\eShopOnWeb.Shared\obj\project.assets.json
   log  : Restored c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj (in 294 ms).
   ```

1. The Newtonsoft.Json package got installed in the Packages as **Newtonsoft.Json**. From the Visual Studio **Solution Explorer**, navigate to the **eShopOnWeb.Shared** Project, expand Dependencies, and notice the **Newtonsoft.Json** under the Packages. Click on the little arrow to the left of the Packages, to open the folder and file list.

When you created the Azure DevOps Artifacts Package Feed, by design, it allows for **upstream sources**, such as nuget.org in the dotnet example that brings in the Newtonsoft.Json package from where the package is hosted. This is a common practice to avoid duplication of packages and to ensure that the latest version is always used.

1. From the Azure DevOps Portal, **refresh** the Artifacts Package Feed page. The list of packages shows both the **eShopOnWeb.Shared** custom-developed package, as well as the **Newtonsoft.Json** public sourced package.
1. From the Visual Studio **eShopOnWeb.Shared** Solution, right-click the **eShopOnWeb.Shared** Project, and select **Manage NuGet Packages** from the context menu.
1. From the NuGet Package Manager window, validate the **Package Source** is set to **eShopOnWebShared**.
1. Click **Browse**, and wait for the list of NuGet Packages to load.
1. This list will also show both the **eShopOnWeb.Shared** custom-developed package, as well as the **Newtonsoft.Json** public sourced package.

## Review

In this lab, you learned how to work with Azure Artifacts by using the following steps:

- Created and connect to a feed.
- Created and publish a NuGet package.
- Imported a custom-developed NuGet package.
