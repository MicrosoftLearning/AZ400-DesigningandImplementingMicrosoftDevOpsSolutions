---
lab:
    title: 'Lab: Configuring Agent Pools and Understanding Pipeline Styles'
    az400Module: 'Module 5: Configuring Azure Pipelines'
---

# Lab: Configuring Agent Pools and Understanding Pipeline Styles
# Student lab manual

## Lab overview

YAML-based pipelines allow you to fully implement CI/CD as code, in which pipeline definitions reside in the same repository as the code that is part of your Azure DevOps project. YAML-based pipelines support a wide range of features that are part of the classic pipelines, such as pull requests, code reviews, history, branching, and templates. 

Regardless of the choice of the pipeline style, to build your code or deploy your solution by using Azure Pipelines, you need an agent. An agent hosts compute resources that runs one job at a time. Jobs can be run directly on the host machine of the agent or in a container. You have an option to run your jobs using Microsoft-hosted agents, which are managed for you, or implementing a self-hosted agent that you set up and manage on your own. 

In this lab, you will step through the process of converting a classic pipeline into a YAML-based one and running it first by using a Microsoft-hosted agent and then performing the equivalent task by using a self-hosted agent.

## Objectives

After you complete this lab, you will be able to:

- implement YAML-based pipelines
- implement self-hosted agents

## Lab duration

-   Estimated time: **90 minutes**

## Instructions

### Before you start

#### Sign in to the lab virtual machine

Ensure that you're signed in to your Windows 10 computer by using the following credentials:
    
-   Username: **Student**
-   Password: **Pa55w.rd**

#### Review the installed applications

Find the taskbar on your Windows 10 desktop. The taskbar contains the icons for the applications that you'll use in this lab:
    
-   Microsoft Edge
-   [Visual Studio Code](https://code.visualstudio.com/). This will be installed as part of prerequisites for this lab.

#### Set up an Azure DevOps organization

If you don't already have an Azure DevOps organization that you can use for this lab, create one by following the instructions available at [Create an organization or project collection](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops).

#### Prepare an Azure subscription

-   Identify an existing Azure subscription or create a new one.
-   Verify that you have a Microsoft account or an Azure AD account with the Owner role in the Azure subscription and the Global Administrator role in the Azure AD tenant associated with the Azure subscription. For details, refer to [List Azure role assignments using the Azure portal](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-list-portal) and [View and assign administrator roles in Azure Active Directory](https://docs.microsoft.com/en-us/azure/active-directory/roles/manage-roles-portal#view-my-roles).

#### Set up a GitHub account

Follow instructions available at [Signing up for a new GitHub account](https://docs.github.com/en/free-pro-team@latest/github/getting-started-with-github/signing-up-for-a-new-github-account).

### Exercise 0: Configure the lab prerequisites

In this exercise, you will set up the prerequisite for the lab, which consists of the preconfigured Parts Unlimited team project based on an Azure DevOps Demo Generator template.

#### Task 1: Configure the team project

In this task, you will use Azure DevOps Demo Generator to generate a new project based on the **PartsUnlimited** template.

1.  On your lab computer, start a web browser and navigate to [Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net). This utility site will automate the process of creating a new Azure DevOps project within your account that is prepopulated with content (work items, repos, etc.) required for the lab. 

    > **Note**: For more information on the site, see https://docs.microsoft.com/en-us/azure/devops/demo-gen.

1.  Click **Sign in** and sign in using the Microsoft account associated with your Azure DevOps subscription.
1.  If required, on the **Azure DevOps Demo Generator** page, click **Accept** to accept the permission requests for accessing your Azure DevOps subscription.
1.  On the **Create New Project** page, in the **New Project Name** textbox, type **Integrating Azure Key Vault with Azure DevOps**, in the **Select organization** dropdown list, select your Azure DevOps organization, and then click **Choose template**.
1.  On the **Choose a template** page, click the **PartsUnlimited** template, and then click **Select Template**.
1.  Back on the **Create New Project** page, select the checkbox below the **ARM Outputs** label, and click **Create Project**

    > **Note**: Wait for the process to complete. This should take about 2 minutes. In case the process fails, navigate to your DevOps organization, delete the project, and try again.

1.  On the **Create New Project** page, click **Navigate to project**.

#### Task 2: Install Visual Studio Code

In this task, you will install Visual Studio Code. If you have already implemented these prerequisites, you can proceed directly to the next task.

1.  If you don't have Visual Studio Code installed yet, from the web browser window, navigate to the [Visual Studio Code download page](https://code.visualstudio.com/), download it, and install it. 

#### Task 3: Prepare the lab environment for image deployment

In this task, you will prepare your lab environment for deployment of an image that will be used to provision a self-hosted agent.

> **Note**: The location of the self-hosted agent and its configuration is highly dependent on your requirements. In this exercise, you will use for this purpose an Azure VM that is based on the same image that is used for Microsoft-hosted agents. The images are available from the [Virtual Environments GitHub repository](https://github.com/actions/virtual-environments). You will also find there images for GitHub Actions hosted runners.

1.  On the lab computer, start a web browser, navigate to [GitHub](https://github.com) and sign into your GitHub account.
1.  In the web browser displaying your GitHub account, in the upper right corner, click the round icon representing your profile and, in the dropdown menu, click **Settings**.
1.  On the **Public profile** page, in the vertical menu on the left, click **Developer settings**.
1.  On the **GitHub Apps** page, in the vertical menu on the left, click **Personal access token**.
1.  On the **Personal access token** page, in the upper right corner, click **Generate new token**.
1.  On the **New personal access token** page, in the **Note** text box, type **az400m05l05a lab**, in the **Select scopes** section, select the **read:packages** checkbox, and click **Generate token**.
1.  On the **Personal access token** page, identify the newly generated token and record its value. You will need it later in this task.

    > **Note**: Make sure to record the personal access token at this point. You will not be able to retrieve its value once you navigate away from the current page. 

1.  On the lab computer, start a web browser, navigate to [the Packer download page](https://www.packer.io/downloads), download the current version of Windows 64-bit version of Packer, open the archive containing the **packer.exe** binary, and extract it into the **C:\Windows** directory. 
1.  On the lab computer, start Windows PowerShell ISE as administrator and, from the console pane of the **Administrator: Windows PowerShell ISE** window, run the following to install Chocolatey:

    ```powershell
    Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
    ```

1.  From the console pane of the **Administrator: Windows PowerShell ISE** window, run the following to install git:

    ```powershell
    choco install git -params '"/GitOnlyOnPath"' -y
    ```

1.  From the console pane of the **Administrator: Windows PowerShell ISE** window, run the following to install packer:

    ```powershell
    choco install packer -y
    ```

1.  From the console pane of the **Administrator: Windows PowerShell ISE** window, run the following to install Azure CLI:

    ```powershell
    Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'; rm .\AzureCLI.msi
    ```

1.  From the console pane of the **Administrator: Windows PowerShell ISE** window, run the following to install PowerShell (when prompted for confirmation to proceed, click **Yes to All**):

    ```powershell
    Install-Module -Name Az
    ```

1.  From the console pane of the **Administrator: Windows PowerShell ISE** window, run the following to sign in to sign in to your Azure subscription (when prompted for credentials, authenticate with a Microsoft account or an Azure AD account with the Owner role in the Azure subscription and the Global Administrator role in the Azure AD tenant associated with the Azure subscription):

    ```powershell
    Add-AzAccount
    ```

1.  From the console pane of the **Administrator: Windows PowerShell ISE** window, run the following to install git:

    ```powershell
    Set-Location c:\Labfiles
    git clone https://github.com/actions/virtual-environments.git virtual-environments -q
    ```

1.  On the lab computer, start File Explorer and navigate to the **C:\\Labfiles\\virtual-environments\\images\\win** directory, open the **windows2019.json** file in Notepad, and review its content.

    > **Note**: This file defines the content of the image. You can customize its configuration and, effectively, affect the image provisioning time by modifying the **provisioners** section.

1.  On the lab computer, in File Explorer, navigate to the **C:\\Labfiles\\virtual-environments\\helpers** directory and open the **GenerateResourcesAndImage.ps1** file in Notepad. 
1.  In Notepad displaying content of the **GenerateResourcesAndImage.ps1** file, in the line containing the command `New-AzStorageAccount -ResourceGroupName $ResourceGroupName -AccountName $storageAccountName -Location $AzureLocation -SkuName "Standard_LRS"``, replace `"Standard_LRS"` with `"Premium_LRS"`, save the change, and close the file.

    > **Note**: This change is intended to accelerate image provisioning.

1.  On the lab computer, in File Explorer, navigate to the **C:\\Labfiles\\virtual-environments\\images\\win** directory, open the **windows2016.json** file in Notepad, paste the following content, replace the existing content, save the changes, and close the file.

    > **Note**: These changes are intended strictly to accelerate image provisioning in this particular lab scenario. In general, you should adjust the image to match your requirements. 

    ```json
    {
        "variables": {
            "client_id": "{{env `ARM_CLIENT_ID`}}",
            "client_secret": "{{env `ARM_CLIENT_SECRET`}}",
            "subscription_id": "{{env `ARM_SUBSCRIPTION_ID`}}",
            "tenant_id": "{{env `ARM_TENANT_ID`}}",
            "object_id": "{{env `ARM_OBJECT_ID`}}",
            "resource_group": "{{env `ARM_RESOURCE_GROUP`}}",
            "storage_account": "{{env `ARM_STORAGE_ACCOUNT`}}",
            "temp_resource_group_name": "{{env `TEMP_RESOURCE_GROUP_NAME`}}",
            "location": "{{env `ARM_RESOURCE_LOCATION`}}",
            "virtual_network_name": "{{env `VNET_NAME`}}",
            "virtual_network_resource_group_name": "{{env `VNET_RESOURCE_GROUP`}}",
            "virtual_network_subnet_name": "{{env `VNET_SUBNET`}}",
            "private_virtual_network_with_public_ip": "{{env `PRIVATE_VIRTUAL_NETWORK_WITH_PUBLIC_IP`}}",
            "vm_size": "Standard_D8s_v3",
            "run_scan_antivirus": "false",
            "root_folder": "C:",
            "toolset_json_path": "{{env `TEMP`}}\\toolset.json",
            "image_folder": "C:\\image",
            "imagedata_file": "C:\\imagedata.json",
            "helper_script_folder": "C:\\Program Files\\WindowsPowerShell\\Modules\\",
            "psmodules_root_folder": "C:\\Modules",
            "install_user": "installer",
            "install_password": null,
            "capture_name_prefix": "packer",
            "image_version": "dev",
            "image_os": "win16"
        },
        "sensitive-variables": [
            "install_password",
            "client_secret"
        ],
        "builders": [
            {
                "name": "vhd",
                "type": "azure-arm",
                "client_id": "{{user `client_id`}}",
                "client_secret": "{{user `client_secret`}}",
                "subscription_id": "{{user `subscription_id`}}",
                "object_id": "{{user `object_id`}}",
                "tenant_id": "{{user `tenant_id`}}",
                "os_disk_size_gb": "256",
                "location": "{{user `location`}}",
                "vm_size": "{{user `vm_size`}}",
                "resource_group_name": "{{user `resource_group`}}",
                "storage_account": "{{user `storage_account`}}",
                "temp_resource_group_name": "{{user `temp_resource_group_name`}}",
                "capture_container_name": "images",
                "capture_name_prefix": "{{user `capture_name_prefix`}}",
                "virtual_network_name": "{{user `virtual_network_name`}}",
                "virtual_network_resource_group_name": "{{user `virtual_network_resource_group_name`}}",
                "virtual_network_subnet_name": "{{user `virtual_network_subnet_name`}}",
                "private_virtual_network_with_public_ip": "{{user `private_virtual_network_with_public_ip`}}",
                "os_type": "Windows",
                "image_publisher": "MicrosoftWindowsServer",
                "image_offer": "WindowsServer",
                "image_sku": "2016-Datacenter",
                "communicator": "winrm",
                "winrm_use_ssl": "true",
                "winrm_insecure": "true",
                "winrm_username": "packer"
            }
        ],
        "provisioners": [
            {
                "type": "powershell",
                "inline": [
                    "New-Item -Path {{user `image_folder`}} -ItemType Directory -Force"
                ]
            },
            {
                "type": "file",
                "source": "{{ template_dir }}/scripts/ImageHelpers",
                "destination": "{{user `helper_script_folder`}}"
            },
            {
                "type": "file",
                "source": "{{ template_dir }}/scripts/SoftwareReport",
                "destination": "{{user `image_folder`}}"
            },
            {
                "type": "file",
                "source": "{{ template_dir }}/post-generation",
                "destination": "C:/"
            },
            {
                "type": "file",
                "source": "{{ template_dir }}/scripts/Tests",
                "destination": "{{user `image_folder`}}"
            },
            {
                "type": "file",
                "source": "{{template_dir}}/toolsets/toolset-2016.json",
                "destination": "{{user `toolset_json_path`}}"
            },
            {
                "type": "windows-shell",
                "inline": [
                    "net user {{user `install_user`}} {{user `install_password`}} /add /passwordchg:no /passwordreq:yes /active:yes /Y",
                    "net localgroup Administrators {{user `install_user`}} /add",
                    "winrm set winrm/config/service/auth @{Basic=\"true\"}",
                    "winrm get winrm/config/service/auth"
                ]
            },
            {
                "type": "powershell",
                "inline": [
                    "if (-not ((net localgroup Administrators) -contains '{{user `install_user`}}')) { exit 1 }"
                ]
            },
            {
                "type": "powershell",
                "environment_vars": [
                    "ImageVersion={{user `image_version`}}",
                    "TOOLSET_JSON_PATH={{user `toolset_json_path`}}",
                    "PSMODULES_ROOT_FOLDER={{user `psmodules_root_folder`}}"
                ],
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-PowerShellModules.ps1",
                    "{{ template_dir }}/scripts/Installers/Initialize-VM.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-WebPlatformInstaller.ps1"
                ],
                "execution_policy": "unrestricted"
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Update-DotnetTLS.ps1"
                ]
            },
            {
                "type": "windows-restart",
                "restart_timeout": "30m"
            },
            {
                "type": "powershell",
                "inline": [
                    "setx ImageVersion {{user `image_version` }} /m",
                    "setx ImageOS {{user `image_os` }} /m"
                ]
            },
            {
                "type": "powershell",
                "environment_vars": [
                    "IMAGE_VERSION={{user `image_version`}}",
                    "IMAGEDATA_FILE={{user `imagedata_file`}}"
                ],
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Update-ImageData.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-PowershellCore.ps1"
                ]
            },
            {
                "type": "windows-restart",
                "restart_timeout": "30m"
            },
            {
                "type": "powershell",
                "valid_exit_codes": [
                    0,
                    3010
                ],
                "environment_vars": [
                    "TOOLSET_JSON_PATH={{user `toolset_json_path`}}"
                ],
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-VS.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-NET48.ps1"
                ],
                "elevated_user": "{{user `install_user`}}",
                "elevated_password": "{{user `install_password`}}"
            },
                {
                "type": "powershell",
                "environment_vars": [
                    "TOOLSET_JSON_PATH={{user `toolset_json_path`}}"
                ],
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-Nuget.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-Vsix.ps1"
                ]
            },
            {
                "type": "windows-restart",
                "restart_timeout": "30m"
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-NodeLts.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-7zip.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-Packer.ps1"
                ]
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-OpenSSL.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-Git.ps1",
                    "{{ template_dir }}/scripts/Installers/Install-GitHub-CLI.ps1"
                ]
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Enable-DeveloperMode.ps1"
                ],
                "elevated_user": "{{user `install_user`}}",
                "elevated_password": "{{user `install_password`}}"
            },
            {
                "type": "windows-shell",
                "inline": [
                    "wmic product where \"name like '%%microsoft azure powershell%%'\" call uninstall /nointeractive"
                ]
            },
            {
                "type": "powershell",
                "environment_vars": [
                    "TOOLSET_JSON_PATH={{user `toolset_json_path`}}"
                ],
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-Cmake.ps1"
                ]
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Install-GitVersion.ps1"
                ]
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Configure-DynamicPort.ps1"
                ],
                "elevated_user": "{{user `install_user`}}",
                "elevated_password": "{{user `install_password`}}"
            },
            {
                "type": "windows-restart",
                "restart_timeout": "30m"
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Finalize-VM.ps1"
                ]
            },
            {
                "type": "windows-restart",
                "restart_timeout": "30m"
            },
            {
                "type": "powershell",
                "scripts": [
                    "{{ template_dir }}/scripts/Installers/Configure-Antivirus.ps1",
                    "{{ template_dir }}/scripts/Installers/Disable-JITDebugger.ps1"
                ]
            },
            {
                "type": "powershell",
                "inline": [
                    "if( Test-Path $Env:SystemRoot\\System32\\Sysprep\\unattend.xml ){ rm $Env:SystemRoot\\System32\\Sysprep\\unattend.xml -Force}",
                    "& $env:SystemRoot\\System32\\Sysprep\\Sysprep.exe /oobe /generalize /quiet /quit",
                    "while($true) { $imageState = Get-ItemProperty HKLM:\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Setup\\State | Select ImageState; if($imageState.ImageState -ne 'IMAGE_STATE_GENERALIZE_RESEAL_TO_OOBE') { Write-Output $imageState.ImageState; Start-Sleep -s 10  } else { break } }"
                ]
            }
        ]
    }
    ```

1.  On the lab computer, in File Explorer, navigate to the **C:\\Labfiles\\virtual-environments\\images\\win\\scripts\\Installers** directory, open the **VisualStudio.Tests.ps1** file in Notepad, locate the following section:

    ```powershell
    $workLoads = @(
	"--allWorkloads --includeRecommended"
	$requiredComponents
	"--remove Component.CPython3.x64"
    )
    ```

1.  In the Notepad window displaying the content of the **VisualStudio.Tests.ps1** file, replace the section you identified in the previous step with the following content:

    ```powershell
    $workLoads = @(
	"--add Microsoft.VisualStudio.Workload.NetWeb --add Component.GitHub.VisualStudio --includeRecommended"
	"--remove Component.CPython3.x64"
    )
    ```

    > **Note**: This change is intended strictly to accelerate image provisioning in this particular lab scenario. In general, you should adjust Visual Studio components to be installed to match your requirements. 

1.  On the lab computer, in File Explorer, navigate to the **C:\\Labfiles\\virtual-environments\\images\\win\\scripts\\Tests** directory, open the **VisualStudio.Tests.ps1** file in Notepad, replace the line `$expectedComponents = Get-ToolsetContent | Select-Object -ExpandProperty visualStudio | Select-Object -ExpandProperty workloads` with `$expectedComponents = "Microsoft.VisualStudio.Workload.NetWeb"`, save the change, and close the file.

    > **Note**: This change is intended strictly to accelerate image provisioning in this particular lab scenario. In general, you should adjust the testing to match your requirements. 

1.  Switch back to the **Administrator: Windows PowerShell ISE** window and, from its console pane, run the following to import the Packer PowerShell module you will use to generate the operating system image you will use to provision the self-hosted agent:

    ```powershell
    Set-Location c:\Labfiles\virtual-environments
    Import-Module .\helpers\GenerateResourcesAndImage.ps1
    ```

1.  From the script pane of the **Administrator: Windows PowerShell ISE** window, run the following to use the Packer PowerShell module to generate the operating system image you will use to provision the self-hosted agent (replace the `<Azure_region>` placeholder with the name of the Azure region into which you want to deploy the Azure VM and the `<GitHub_PAT>` placeholder with the value of the GitHub personal access token you generated earlier in this task):

    ```powershell
    $location = '<Azure_region>'
    $githubPAT = '<GitHub_PAT>'
    $random = (-join (((48..57)+(65..90)+(97..122)) * 80 | Get-Random -Count 6 |%{[char]$_})).ToLower()
    $rgName = "az400m05$random-RG"
    $subscriptionId = (Get-AzSubscription).SubscriptionId  
    GenerateResourcesAndImage -SubscriptionId $subscriptionId -ResourceGroupName $rgName -ImageGenerationRepositoryRoot "$pwd" -ImageType 'Windows2016' -AzureLocation $location -GitHubFeedToken $githubPAT
    ```

1.  When prompted, sign in with the same user account you used earlier to authenticate to the Azure AD tenant associated with the Azure subscription.

    > **Note**: Do not wait for the script to complete but instead proceed to the next exercise. Provisioning of the image might take about 60 minutes.

### Exercise 1: Author YAML-based Azure DevOps pipelines

In this exercise, you will convert a classic Azure DevOps pipeline into a YAML-based one. 

#### Task 1: Create an Azure DevOps YAML pipeline

In this task, you will create a template-based Azure DevOps YAML pipeline.

1.  From the web browser displaying the Azure DevOps portal with the **Configuring Agent Pools and Understanding Pipeline Styles** project open, in the vertical navigational pane on the left side, click **Pipelines**. 
1.  On the **Recent** tab of the **Pipelines** pane, click **New pipeline**.
1.  On the **Where is your code?** pane, click **Azure Repos Git**. 
1.  On the **Select a repository** pane, click **PartsUnlimited**.
1.  On the **Configure your pipeline** pane, click **Starter pipeline**.
1.  On the **Review your pipeline YAML** pane, review the sample pipeline, click the down-facing caret symbol next to the **Save and run** button, click **Save** and, on the **Save** pane, click **Save**.

    > **Note**: This will result in creation of the **azure-pipelines.yml** file in the root directory of the repository hosting the project code.

    > **Note**: You will replace the content of the sample YAML pipeline with the code of pipeline generated by the classic editor and modify it to account for differences between the classic and YAML pipelines.

#### Task 2: Convert a classic pipeline into a YAML pipeline

In this task, you will convert a classic pipeline into a YAML pipeline

1.  From the web browser displaying the Azure DevOps portal with the **Configuring Agent Pools and Understanding Pipeline Styles** project open, in the vertical navigational pane on the left side, click **Pipelines**. 
1.  On the **Recent** tab of the **Pipelines** pane, hover with the mouse pointer over the right edge of the entry containing the **PartsUnlimitedE2E** entry to reveal the vertical ellipsis symbol designating the **More** menu, click the ellipsis symbol, and, in the dropdown menu, click **Edit**. This will display the build pipeline that is part of the project you generated at the beginning of the lab. 
1.  On the **Tasks** tab of the **PartsUnlimitedE2E** edit pane, click **Triggers**, on the right side of the **PartsUnlimited** pane, uncheck the **Enable continuous integration** checkbox, click the down-facing caret next to the **Save & queue** button, in the dropdown menu, click **Save**, and in the **Save build pipeline** click **Save**.

    > **Note**: This will prevent from unintended execution of automatic build due to changes to the repository during this lab.

    > **Note**: Alternatively, you could simply delete the existing pipeline once you copy its content into the new one.

1.  In the Azure DevOps portal, in the vertical navigational pane on the left side, in the **Pipelines** section, click **Pipelines**. 
1.  On the **Recent** tab of the **Pipelines** pane, click the **PartsUnlimitedE2E** entry. 
1.  On the **Runs** tab of the **PartsUnlimitedE2E** pane, in the upper right corner, click the vertical ellipsis symbol and, in the dropdown menu, click **Export to YAML**. This will automatically download the **build.yml** file to your local **Downloads** folder.

    > **Note**: The **Export to YAML** feature replaces an older **View YAML** option available from the pipeline editor pane within the Azure DevOps portal, which was limited to viewing YAML content one job at a time. The new functionality leverages existing classic and YAML pipeline infrastructure, including YAML parsing library, which results in more accurate translation between the two. It supports the following pipeline components:

    - Single and multiple jobs
    - Checkout options
    - Execution plan parallelism
    - Timeout and name metadata
    - Demands
    - Schedules and other triggers
    - Pool selection, including jobs which differ from the default
    - All tasks and all inputs, including optimizing for default inputs
    - Job and step conditions
    - Task group unrolling

    > **Note**: The only components not covered by the new functionality are variables and timezone translation. Variables defined in YAML override variables set in the user interface of the Azure DevOps portal. If the **Export to YAML** feature detects presence of Classic pipeline variables, they will be explicitly included in the comments within the newly generated YAML pipeline definition. Similarly, since cron schedules in YAML are expressed in UTC, while classic schedules rely on the organization’s timezone, their presence is also included in the comments.

    > **Note**: For more information regarding this functionality, refer to [Replacing "View YAML"](https://devblogs.microsoft.com/devops/replacing-view-yaml/)

1.  On the lab computer, start Visual Studio Code and use it to open the file **build.yml**. The file should have the following content:

    ```yaml
    name: $(date:yyyyMMdd)$(rev:.r)
    jobs:
    - job: Phase_1
      displayName: Phase 1
      cancelTimeoutInMinutes: 1
      pool:
        vmImage: vs2017-win2016
      steps:
      - checkout: self
      - task: NuGetInstaller@0
        name: NuGetInstaller_1
        displayName: NuGet restore
        inputs:
          solution: '**\*.sln'
      - task: VSBuild@1
        name: VSBuild_2
        displayName: Build solution
        inputs:
          vsVersion: 15.0
          msbuildArgs: /p:DeployOnBuild=true /p:WebPublishMethod=Package /p:PackageAsSingleFile=true /p:SkipInvalidConfigurations=true /p:PackageLocation="$(build.stagingDirectory)" /p:IncludeServerNameInBuildInfo=True /p:GenerateBuildInfoConfigFile=true /p:BuildSymbolStorePath="$(SymbolPath)" /p:ReferencePath="C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\Common7\IDE\Extensions\Microsoft\Pex"
          platform: $(BuildPlatform)
          configuration: $(BuildConfiguration)
      - task: VSTest@1
        name: VSTest_3
        displayName: Test Assemblies
        inputs:
          testAssembly: '**\$(BuildConfiguration)\*test*.dll;-:**\obj\**'
          codeCoverageEnabled: true
          vsTestVersion: latest
          platform: $(BuildPlatform)
          configuration: $(BuildConfiguration)
      - task: CopyFiles@2
        name: CopyFiles1
        displayName: Copy Files
        inputs:
          SourceFolder: $(build.sourcesdirectory)
          Contents: '**/*.json'
          TargetFolder: $(build.artifactstagingdirectory)
      - task: PublishBuildArtifacts@1
        name: PublishBuildArtifacts_5
        displayName: Publish Artifact
        inputs:
          PathtoPublish: $(build.artifactstagingdirectory)
          TargetPath: '\\my\share\$(Build.DefinitionName)\$(Build.BuildNumber)'
    ...
    ```

1.  In the Visual Studio Code window, in the top-level menu, click **Selection** and, in the dropdown menu, click **Select All**.
1.  In the Visual Studio Code window, in the top-level menu, click **Edit** and, in the dropdown menu, click **Copy**.
1.  Switch to the browser window displaying the Azure DevOps portal and, in the vertical navigational pane on the left side, in the **Pipelines** section, click **Pipelines**. 
1.  On the **Recent** tab of the **Pipelines** pane, select **PartsUnlimited** and, on the **PartsUnlimited** pane, select **Edit**.
1.  On the **PartsUnlimited** edit pane, select the existing YAML content of the pipeline, replace it with the content of Clipboard.
1.  On the **PartsUnlimited** edit pane, in the upper-right corner, click **Save**, and, on the **Save** pane, click **Save**. This will automatically trigger the build based on this pipeline. 
1.  In the Azure DevOps portal, in the vertical navigational pane on the left side, in the **Pipelines** section, click **Pipelines**.
1.  On the **Recent** tab of the **Pipelines** pane, click the **PartsUnlimited** entry, on the **Runs** tab of the **PartsUnlimited** pane, select the most recent run, on the **Summary** pane of the run, scroll down to the bottom, in the **Jobs** section, click **Phase 1** and monitor the job until its successful completion. 

### Exercise 2: Manage Azure DevOps agent pools

In this exercise, you will implement self-hosted Azure DevOps agent.

> **Note**: Before you start this exercise, make sure that the image provisioning script you started at the beginning of the lab has successfully completed.

#### Task 1: Deploy an Azure VM-based self-hosted agent

In this task, you will deploy an Azure VM that will be used to provision a self-hosted agent by using the custom image you generated at the beginning of this lab.

1.  On the lab computer, switch to the **Administrator: Windows PowerShell ISE** window, review the output of the **GenerateResourcesAndImage** script you invoked at the beginning of this lab and identify the value of the storage account name by examining the first part of the **OSDiskUri** string, representing the full path to the generated vhd file.

    > **Note**: The output should resemble the following value `OSDiskUri: https://az400m05l05xrg001.blob.core.windows.net/system/Microsoft.Compute/Images/images/packer-osDisk.40096600-7cac-47f4-8573-ffaa113c78b1.v
hd', where `az400m05l05xrg001` represents the storage account name.

1.  From the lab computer, start a web browser, navigate to the [**Azure Portal**](https://portal.azure.com), and sign in with the user account that you used to provision the Azure VM in the previous task. 
1.  In the Azure portal, search for and select the **Storage accounts** resource type and, on the **Storage accounts** blade, click the name of the storage account you identified earlier in this task. 
1.  On the storage account blade, note the name of the resource group containing the storage account. You will need it later in this task.
1.  On the storage account blade, in the vertical menu on the left side, in the **Blob service** section, click **Containers**.
1.  On the Containers blade, click **+ Container**, on the **New container** blade, in the **Name** text box, type **disks**, ensure that the **Public access level** dropbox contains the **Private (no anonymous access)** entry, and click **Create**.
1.  Back on the Containers blade, in the list of containers, click **images**, and, on the **images** blade, click the entry representing the newly generated image.
1.  On the blade displaying the image properties, next to the **URL** entry, click the **Copy to clipboard** icon and paste the copied value to Notepad. You will need it in the next step. 
1.  On the lab computer, switch to the **Administrator: Windows PowerShell ISE** window and, from its script pane, run the following to set the variables required for the Azure VM deployment by using an Azure Resource Manager template (replace the `<resource_group_name>` placeholder with the name of the resource group you identified earlier in this task, the `<storage_account_name>` placeholder with the name of the storage account you identified earlier in this task, and the `<image_url>` placeholder with the name of the image URL you identified earlier in this task):

    ```powershell
    $rgName = `<resource_group_name>'
    $storageAccountName = '<storage_account_name>'
    $imageUrl = '<image_url>'
    ```

1.  On the lab computer, switch to the File Explorer window and create a new file in the **C:\\Labfiles** folder named **az400m05-vm0.deployment.template.json** with the following content, save it, and close it. 

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
          "vmSize": {
            "type": "string",
            "defaultValue": "Standard_D2s_v3",
            "metadata": {
              "description": "Virtual machine size"
            }
          },
          "adminUsername": {
            "type": "string",
            "metadata": {
              "description": "Admin username"
            }
          },
          "adminPassword": {
            "type": "securestring",
            "metadata": {
              "description": "Admin password"
            }
          },
          "storageAccountName": {
            "type": "string",
            "metadata": {
               "description": "storage account name"
            }
          },
          "imageUrl": {
            "type": "string",
            "metadata": {
               "description": "image Url"
            }
          },
          "diskContainer": {
            "type": "string",
            "defaultValue": "disks",
            "metadata": {
              "description": "disk container"
            }
          }
        },
      "variables": {
        "vmName": "az400m05-vm0",
        "osType": "Windows",
        "osDiskVhdName": "[concat('http://',parameters('storageAccountName'),'.blob.core.windows.net/',parameters('diskContainer'),'/',variables('vmName'),'-osDisk.vhd')]",
        "nicName": "az400m05-vm0-nic0",
        "virtualNetworkName": "az400m05-vnet0",
        "publicIPAddressName": "az400m05-vm0-nic0-pip",
        "nsgName": "az400m05-vm0-nic0-nsg",
        "vnetIpPrefix": "10.0.0.0/22", 
        "subnetIpPrefix": "10.0.0.0/24", 
        "subnetName": "subnet0",
        "subnetRef": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('virtualNetworkName'), variables('subnetName'))]",
        "computeApiVersion": "2018-06-01",
        "networkApiVersion": "2018-08-01"
      },
        "resources": [
            {
                "name": "[variables('vmName')]",
                "type": "Microsoft.Compute/virtualMachines",
                "apiVersion": "[variables('computeApiVersion')]",
                "location": "[resourceGroup().location]",
                "dependsOn": [
                    "[variables('nicName')]"
                ],
                "properties": {
                    "osProfile": {
                        "computerName": "[variables('vmName')]",
                        "adminUsername": "[parameters('adminUsername')]",
                        "adminPassword": "[parameters('adminPassword')]",
                        "windowsConfiguration": {
                            "provisionVmAgent": "true"
                        }
                    },
                    "hardwareProfile": {
                        "vmSize": "[parameters('vmSize')]"
                    },
                    "storageProfile": {
                        "osDisk": {
                            "name": "[concat(variables('vmName'),'-osDisk')]",
                            "osType": "[variables('osType')]",
                            "caching": "ReadWrite",
                            "image": {
                                "uri": "[parameters('imageUrl')]"
                            },
                            "vhd": {
                                "uri": "[variables('osDiskVhdName')]"
                            },
                            "createOption": "fromImage"
                        },
                        "dataDisks": []
                    },
                    "networkProfile": {
                        "networkInterfaces": [
                            {
                                "properties": {
                                    "primary": true
                                },
                                "id": "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
                            }
                        ]
                    }
                }
            },
            {
                "type": "Microsoft.Network/virtualNetworks",
                "name": "[variables('virtualNetworkName')]",
                "apiVersion": "[variables('networkApiVersion')]",
                "location": "[resourceGroup().location]",
                "comments": "Virtual Network",
                "properties": {
                    "addressSpace": {
                        "addressPrefixes": [
                            "[variables('vnetIpPrefix')]"
                        ]
                    },
                    "subnets": [
                        {
                            "name": "[variables('subnetName')]",
                            "properties": {
                                "addressPrefix": "[variables('subnetIpPrefix')]"
                            }
                        }
                    ]
                }
            },
            {
                "name": "[variables('nicName')]",
                "type": "Microsoft.Network/networkInterfaces",
                "apiVersion": "[variables('networkApiVersion')]",
                "location": "[resourceGroup().location]",
                "comments": "Primary NIC",
                "dependsOn": [
                    "[variables('publicIpAddressName')]",
                    "[variables('nsgName')]",
                    "[variables('virtualNetworkName')]"
                ],
                "properties": {
                    "ipConfigurations": [
                        {
                            "name": "ipconfig1",
                            "properties": {
                                "subnet": {
                                    "id": "[variables('subnetRef')]"
                                },
                                "privateIPAllocationMethod": "Dynamic",
                                "publicIpAddress": {
                                    "id": "[resourceId('Microsoft.Network/publicIpAddresses', variables('publicIpAddressName'))]"
                                }
                            }
                        }
                    ],
                    "networkSecurityGroup": {
                        "id": "[resourceId('Microsoft.Network/networkSecurityGroups', variables('nsgName'))]"
                    }
                }
            },
            {
                "name": "[variables('publicIpAddressName')]",
                "type": "Microsoft.Network/publicIpAddresses",
                "apiVersion": "[variables('networkApiVersion')]",
                "location": "[resourceGroup().location]",
                "comments": "Public IP for Primary NIC",
                "properties": {
                    "publicIpAllocationMethod": "Dynamic"
                }
            },
            {
                "name": "[variables('nsgName')]",
                "type": "Microsoft.Network/networkSecurityGroups",
                "apiVersion": "[variables('networkApiVersion')]",
                "location": "[resourceGroup().location]",
                "comments": "Network Security Group (NSG) for Primary NIC",
                "properties": {
                    "securityRules": [
                        {
                            "name": "default-allow-rdp",
                            "properties": {
                                "priority": 1000,
                                "sourceAddressPrefix": "*",
                                "protocol": "Tcp",
                                "destinationPortRange": "3389",
                                "access": "Allow",
                                "direction": "Inbound",
                                "sourcePortRange": "*",
                                "destinationAddressPrefix": "*"
                            }
                        }
                    ]
                }
            }
        ],
        "outputs": {}
    }
    ```

1.  On the lab computer, in the File Explorer window and create a new file in the **C:\\Labfiles** folder named **az400m05-vm0.deployment.template.json** with the following content, save it, and close it. 

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "vmSize": {
                "value": "Standard_D2s_v3"
            },
            "adminUsername": {
                "value": "Student"
            },
            "adminPassword": {
                "value": "Pa55w.rd1234"
            }
        }
    }
    ```    

1.  From the script pane of the **Administrator: Windows PowerShell ISE** window, run the following to deploy an Azure VM that will be used to provision a self-hosted agent by using the custom image you generated at the beginning of this lab:

    ```powershell
    Set-Location -Path 'C:\Labfiles'
    New-AzResourceGroupDeployment -ResourceGroupName $rgName -Name az400m05-vm0-deployment -TemplateFile .\az400m05-vm0.deployment.template.json -TemplateParameterFile .\az400m05-vm0.deployment.template.parameters.json -storageAccountName $storageAccountName -imageUrl $imageUrl
    ```

    > **Note**: Wait for the provisioning process to complete. This should take about 5 minutes. 


#### Task 2: Configure an Azure DevOps self-hosting agent

In this task, you will configure the newly provisioned Azure VM as an Azure DevOps self-hosting agent and use it to run a build pipeline.

1.  On the lab computer, switch to the web browser window displaying the Azure Portal, in the Azure portal, search for and select the **Virtual machines** resource type and, on the **Virtual machines** blade, click **az400m05-vm0**.
1.  On the **az400m05-vm0** blade, click **Connect**, in the dropdown list, click **RDP**, on the **az400m05-vm0 \| Connect** blade, click **Download RDP File** and open the downloaded RDP file to connect to the **az400m05-vm0** Azure VM by using Remote Desktop. When prompted to sign in, provide the **Student** as the user name and **Pa55w.rd1234** as the password.
1.  Within the Remote Desktop session to **az400m05-vm0**, start a web browser, navigate to [the Azure DevOps portal](https://dev.azure.com) and sign in by using the Microsoft account associated with your Azure DevOps organization.
1.  In the Azure DevOps portal, close the **Get the agent** panel, in the upper right corner of the Azure DevOps page, click the **User settings** icon, in the dropdown menu, click **Personal access tokens**, on the **Personal Access Tokens** pane, and click **+ New Token**.
1.  On the **Create a new personal access token** pane, click the **Show all scopes** link and, specify the following settings and click **Create** (leave all others with their default values):

    | Setting | Value |
    | --- | --- |
    | Name | **Configuring Agent Pools and Understanding Pipeline Styles lab** |
    | Scope | **Agent Pools** |
    | Permissions | **Read and manage** |

1.  On the **Success** pane, copy the value of the personal access token to Clipboard.

    > **Note**: Make sure you copy the token. You will not be able to retrieve it once you close this pane. 

1.  On the **Success** pane, click **Close**.
1.  On the **Personal Access Token** pane of the Azure DevOps portal, click **Azure DevOps** symbol in the upper left corner and then click **Organization settings** label in the lower left corner.
1.  To the left side of the **Overview** pane, in the vertical menu, in the **Pipelines** section, click **Agent pools**.
1.  On the **Agent pools** pane, in the upper right corner, click **Add pool**. 
1.  On the **Add agent pool** pane, in the **Pool type** dropdown list, select **Self-hosted**, in the **Name** text box, type **az400m05l05a-pool** and then click **Create**.
1.  Back on the **Agent pools** pane, click the entry representing the newly created **az400m05l05a-pool**. 
1.  On the **Jobs** tab of the **az400m05l05a-pool** pane, click the **Agents** header.
1.  On the **Agents** tab of the **az400m05l05a-pool** pane, click the **New agent** button.
1.  On the **Get the agent** pane, ensure that the **Windows** and **x64** tabs are selected, and click **Download** to download the zip archive containing the agent binaries to download it into the local **Downloads** folder within your user profile.

    > **Note**: If you receive an error message at this point indicating that the current system settings prevent you from downloading the file, in the Internet Explorer window, in the upper right corner, click the gearwheel symbol designating the **Settings** menu header, in the dropdown menu, select **Internet Options**, in the **Internet Options** dialog box, click **Advanced**, on the **Advanced** tab, click **Reset**, in the **Reset Internet Explorer Settings** dialog box, click **Reset** again, click **Close**, and try the download again. 

1.  Within the Remote Desktop session to **az400m05-vm0**, start Windows PowerShell as administrator and, from the **Administrator: Windows PowerShell** console, run the following to create the **C:\\agent** directory and extract the content of the downloaded archive into it:

    ```powershell
    New-Item -Type Directory -Path 'C:\agent'
    Set-Location -Path 'C:\agent'
    Add-Type -AssemblyName System.IO.Compression.FileSystem
    [System.IO.Compression.ZipFile]::ExtractToDirectory("$HOME\Downloads\vsts-agent-win-x64-2.179.0.zip", "$PWD")
    ```

1.  Within the Remote Desktop session to **az400m05-vm0**, from the **Administrator: Windows PowerShell** console, run the following to configure the agent:

    ```powershell
    .\config.cmd
    ```

1.  When prompted, specify the values of the following settings:

    | Setting | Value |
    | ------- | ----- |
    | Enter server URL | the URL of your Azure DevOps organization, in the format **https://dev.azure.com/`<organization_name>`**, where `<organization_name>` represents the name of your Azure DevOps organization |
    | Enter authentication type (press enter for PAT) | **Enter** key | 
    | Enter personal access token | the access token you recorded earlier in this task |
    | Enter agent pool (press enter for default) | **az400m05l05a-pool** |
    | Enter agent name | **az400m05-vm0** |
    | Enter work folder (press enter for _work) | **C:\agent** |
    | Enter Perform an unzip for tasks for each step. (press enter for N) | **Enter** |
    | Enter run agent as service? (Y/N) (press enter for N) | **Enter** |
    | Enter configure autologon and run agent on startup (Y/N) (press enter for N) | **Enter** |

    > **Note**: You can run self-hosted agent as either a service or an interactive process. You might want to start with the interactive mode, since this simplifies verifying agent functionality. For production use, you should consider either running the agent as a service or as an interactive process with auto-logon enabled, since both persist their running state and ensure that the agent starts automatically if the operating system is restarted.

1.  Within the Remote Desktop session to **az400m05-vm0**, from the **Administrator: Windows PowerShell** console, run the following to start the agent in the interactive mode:

    ```powershell
    .\run.cmd
    ```

    > **Note**: Verify that the agent is reporting the **Listening for Jobs** status.

1.  Switch to the browser window displaying the Azure DevOps portal and close the **Get the agent** pane.
1.  Back on the **Agents** tab of the **az400m05l05a-pool** pane, note that the newly configured agent is listed with the **Online** status.
1.  In the web browser window displaying the Azure DevOps portal, in the upper left corner, click the **Azure DevOps** label.
1.  In the browser window displaying the list of projects, click the tile representing your **Configuring Agent Pools and Understanding Pipeline Styles** project. 
1.  On the **Configuring Agent Pools and Understanding Pipeline Styles** pane, in the vertical navigational pane on the left side, in the **Pipelines** section, click **Pipelines**. 
1.  On the **Recent** tab of the **Pipelines** pane, select **PartsUnlimited** and, on the **PartsUnlimited** pane, select **Edit**.
1.  On the **PartsUnlimited** edit pane, in the existing YAML-based pipeline, replace line **6** `vmImage: vs2017-win2016` designating the target agent pool the following content, designating the newly created self-hosted agent pool:

    ```yaml
    name: az400m05l05a-pool
    demands:
    - agent.name -equals az400m05-vm0
    ```

1.  On the **PartsUnlimited** edit pane, in the upper right corner of the pane, click **Save** and, on the **Save** pane, click **Save** again. This will automatically trigger the build based on this pipeline. 
1.  In the Azure DevOps portal, in the vertical navigational pane on the left side, in the **Pipelines** section, click **Pipelines**.
1.  On the **Recent** tab of the **Pipelines** pane, click the **PartsUnlimited** entry, on the **Runs** tab of the **PartsUnlimited** pane, select the most recent run, on the **Summary** pane of the run, scroll down to the bottom, in the **Jobs** section, click **Phase 1** and monitor the job until its successful completion. 

### Exercise 3: Remove the Azure lab resources

In this exercise, you will remove the Azure resources provisione in this lab to eliminate unexpected charges. 

>**Note**: Remember to remove any newly created Azure resources that you no longer use. Removing unused resources ensures you will not see unexpected charges.

#### Task 1: Remove the Azure lab resources

In this task, you will use Azure Cloud Shell to remove the Azure resources provisione in this lab to eliminate unnecessary charges. 

1.  In the Azure portal, open the **Bash** shell session within the **Cloud Shell** pane.
1.  List all resource groups created throughout the labs of this module by running the following command:

    ```sh
    az group list --query "[?starts_with(name,'az400m05')].name" --output tsv
    ```

1.  Delete all resource groups you created throughout the labs of this module by running the following command:

    ```sh
    az group list --query "[?starts_with(name,'az400m05')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Note**: The command executes asynchronously (as determined by the --nowait parameter), so while you will be able to run another Azure CLI command immediately afterwards within the same Bash session, it will take a few minutes before the resource groups are actually removed.

#### Review

In this lab, you learned how to convert classic pipelines into YAML-based ones and how to implement and use self-hosted agents.
