---
lab:
    title: 'Lab: Modernizing your Existing ASP.NET Apps with Azure'
    az400Module: 'Module 15: Managing Containers using Docker'
---

# Lab: Modernizing your Existing ASP.NET Apps with Azure
# Student lab manual

## Lab overview

If you decide to modernize your web applications as part of migration to Azure, you don't necessarily have to entirely re-architect them. Re-architecting an application by using a more advanced approach, such as micro-services, isn't always an option, because of cost and time restraints. However, you might be able to modernize your apps by leveraging containerization and Azure PaaS services. For example, migrating the data tier of your apps to a managed service offering such as Azure's SQL Database could be limited to an update to connection strings, without the need to change the underlying code.

In this lab, you will use the Nerd Dinner Application to illustrate this approach. Nerd Dinner is an open source ASP.NET MVC project. You can see the site running live at [http://www.nerddinner.com](http://www.nerddinner.com). You will move the application DB to Azure SQL instance and add the Docker support to the application to run the application in Azure Container Instances.

## Objectives

After you complete this lab, you will be able to:

- Migrate LocalDB to SQL Server in Azure
- Use Docker tools in Visual Studio and add Docker support for applications
- Publish Docker images to Azure Container Registry (ACR)
- Push Docker images from ACR to Azure Container Instances (ACI)

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
-   Visual Studio 2019 Community Edition available from [Visual Studio Downloads page](https://visualstudio.microsoft.com/downloads/). Visual Studio 2019 installation should include **ASP.NET and web development**, **Azure development**, and **.NET Core cross-platform development** workloads. This is already preinstalled on your lab computer.
-   **Docker for windows** available from [the Docker documentation site](https://docs.docker.com/docker-for-windows/install/#download-docker-for-windows). This will be installed as part of prerequisites for this lab. 

#### Prepare an Azure subscription

-   Identify an existing Azure subscription or create a new one.
-   Verify that you have a Microsoft account or an Azure AD account with the Owner role in the Azure subscription and the Global Administrator role in the Azure AD tenant associated with the Azure subscription. For details, refer to [List Azure role assignments using the Azure portal](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-list-portal) and [View and assign administrator roles in Azure Active Directory](https://docs.microsoft.com/en-us/azure/active-directory/roles/manage-roles-portal#view-my-roles).

### Exercise 0: Configure the lab prerequisites

In this exercise, you will set up the prerequisites for the lab, which include cloning the Nerd Dinner application and configuring Docker Desktop.

#### Task 1: Clone the Nerd Dinner application

In this task, you will create a clone of the Nerd Dinner application and open it in Visual Studio.

1.  From your lab computer, start a web browser and navigate to [https://github.com/spboyer/nerddinner-mvc4](https://github.com/spboyer/nerddinner-mvc4). 
1.  On the [spboyer/nerddinner-mvc4](https://github.com/spboyer/nerddinner-mvc4) page, click **Code** and, in the dropdown list, click the Clipboard icon next to the HTTPS URL of the code repository.
1.  From the lab computer, start Visual Studio and, on the launching page, click **Clone a repository**.
1.  On the **Clone a repository** page, in the **Repository location** textbox, paste the content of Clipboard, and click **Clone**.
1.  If prompted to install additional components, click **Install**.
1.  Within the Visual Studio window, in the top menu, click **Build** and, in the dropdown menu, click **Rebuild Solution**.
1.  Within the Visual Studio window, in the top menu, click **IIS Express**. This will automatically open a web browser displaying the web application.
1.  Verify that the application runs locally and close the web browser window displaying the running application.

#### Task 2: Configure Docker Desktop 

1.  From the lab computer, start a web browser, navigate to [the Docker documentation site](https://docs.docker.com/docker-for-windows/install/#download-docker-for-windows), download and install Docker Desktop for Windows with the default settings.
1.  On the lab computer, expand the taskbar, right-click the **Docker** icon, and, in the right-click menu, select the **Switch to Windows containers** option.
1.  When prompted for confirmation, click **Switch**.

### Exercise 1: Modernize the Nerd Dinner application

In this exercise, you will modernize the Nerd Dinner application by using Docker-based containerization and Azure SQL Database.

#### Task 1: Create an Azure SQL database

In this task, you will create an Azure SQL database.

1.  From the lab computer, start a web browser, navigate to the [**Azure Portal**](https://portal.azure.com), and sign in with the user account that has at least the Contributor role in the Azure subscription you are using in this lab.
1.  In the Azure portal, search for and select the **SQL databases** resource type and, on the **SQL databases** blade, click **+ Add**.
1.  On the **Basics** tab of the **Create SQL Database** blade, specify the following settings:

    | Setting | Value | 
    | --- | --- |
    | Subscription | the name of your Azure subscription |
    | Resource group | the name of a new resource group **az400m15l01a-RG** |
    | Database name | **nerddinnerlab** | 

1.  On the **Basics** tab of the **Create SQL Database** blade, directly below the **Select a server** dropdown list, click **Create new**. 
1.  On the **New server** blade, specify the following settings and click **OK**:

    | Setting | Value | 
    | --- | --- |
    | Server name | any valid, globally unique server name | 
    | Server admin login | **sqluser** |
    | Password | **Pa55w.rd1234** |
    | Location | any Azure region close to your lab location where you can provision Azure SQL databases |

    > **Note**: Record the name you used for the server. You will need it later in this lab.

1.  Back on the **Basics** tab of the **Create SQL Database** blade, next to the **Compute + storage** label, click **Configure database**.
1.  On the **Configure** blade, click **Looking for basic, standard, premium?** and click **Apply**.
1.  Back on the **Basics** tab of the **Create SQL Database** blade, click **Next: Networking >**:
1.  On the **Networking** tab of the **Create SQL Database** blade, specify the following settings (leave others with their default values) and click **Additional settings**:

    | Setting | Value | 
    | --- | --- |
    | Connectivity method | **Public endpoint** |
    | Allow Azure services and resources to access this server | **Yes** |
    | Add current client IP address | **Yes** |

1.  On the **Additional settings** tab of the **Create SQL Database** blade, specify the following settings (leave others with their default values) and click **Review + create**:

    | Setting | Value | 
    | --- | --- |
    | Use existing data | **None** |
    | Azure Defender or SQL | **Not now** |

1.  On the **Review + create** of the **Create SQL Database** blade, click **Create**. 

    > **Note**: Wait for the deployment to complete. This should take about 5 minutes.

#### Task 2: Migrate the LocalDB to SQL Server in Azure

In this task, you will migrate the application's LocalDB to the Azure SQL database you created in the previous task.

1.  In Visual Studio, in the top menu, click **View** and, in the dropdown menu, click **SQL Server Object Explorer**.
1.  In the **SQL Server Object Explorer** pane, click the **Add Server** icon, in the **Connect** dialog box, specify the following settings and click **Connect** (replace **<server-name>** with the name of the logical server you created in the previous task):

    | Setting | Value | 
    | --- | --- |
    | Server Name | **<server-name>.database.windows.net** |
    | Authentication | **SQL Server Authentication** |
    | User Name | **sqluser** |
    | Password | **Pa55w.rd1234** |
    | Database Name | **nerddinnerlab** | 

    > **Note**: First, you will copy the schema from the LocalDB instance to the newly provisioned Azure SQL database.

1.  In the **SQL Server Object Explorer** pane, expand the **localdb** node representing the localDB instance and then, expand its **Databases** folder.  
1.  In the list of databases, right-click the database that is part of the project you created based on the Nerd Dinner Application repository and, in the right-click menu, click **Schema Compare**. This will open the **SqlSchemaCompare1** tab in the main pane of the Visual Studio window.
1.  At the top of the **SqlSchemaCompare1** tab, in the **Select target** dropdown list, click **Select Target**. 
1.  In the **Select Target Schema** dialog box, ensure that the **Database** option is selected, click **Select Connection**, in the **Connect** dialog box, select **nerddinnerlab** Azure SQL database, click **Connect**, and back in the **Select Target Schema** dialog box, click **OK**.
1.  On the **SqlSchemaCompare1** tab in the main pane of the Visual Studio window, click **Compare**. Wait for the comparison to complete.
1.  Once the comparison completes, on the **SqlSchemaCompare1** tab, click **Update** and, when prompted for the confirmation, click **Yes**.
1.  Close the **SqlSchemaCompare1** tab.

    > **Note**: Now you will copy data from the LocalDB instance to the newly provisioned Azure SQL database.

1.  In the **SQL Server Object Explorer** pane, right-click on the LocalDB Instance and, in the right-click menu, click **Data Comparison**. This will launch the **New Data Comparison** wizard and open the **SqlDataCompare1** tab.
1.  On the **Choose Source and Target Databases** page of the **New Data Comparison** wizard, in the **Target Database** section, click **Select Connection**, in the **Connect** dialog box, select **nerddinnerlab** Azure SQL database, click **Connect**, and back on the **Choose Source and Target Databases** page, click **Next**.
1.  On the **Select which tables, fields, and views to compare**, select the checkboxes next to the **Tables** and **Views** entries and click **Finish**.
1.  At the top menu of the **SqlDataCompare1** tab, click **Update Target** and, when prompted for confirmation, click **Yes**.
1.  Close the **SqlDataCompare1** tab.

    > **Note**: In order to avoid code changes, you will use **web.config** transforms. In this specific case, you will add a new **web.release.config** entry. 

1.  In Visual Studio, in the **Solution Explorer** pane, expand the **Web.config** entry and select **Web.Release.config**. This will open the **Web.Release.config** file in the central pane of the Visual Studio window.
1.  On the **Web.Release.config** pane, add the following entry (replace `<server_name>` placeholder with the name of the server you created in the first task of this exercise) directly below the `<configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">` line and save the change:

    ```csharp
    <connectionStrings>
      <add name="DefaultConnection" connectionString="Data Source=<server_name>.database.windows.net;Initial Catalog=nerddinnerlab;Integrated Security=False;User ID=sqluser;Password=Pa55w.rd1234;Connect Timeout=30;Encrypt=True;TrustServerCertificate=False;ApplicationIntent=ReadWrite;MultiSubnetFailover=False" providerName="System.Data.SqlClient" xdt:Transform="SetAttributes" xdt:Locator="Match(name)"/>
    </connectionStrings>
    ```

    > **Note**: Now you have successfully migrated the application's LocalDB to the Azure SQL database and updated the connection string to reflect the change of the database location.

#### Task 3: Add Docker support and debug the application locally within the Docker container by using Visual Studio

In this task, you will add Docker support to Visual Studio and use it to debug the application locally within the Docker container.

1.  In order to use Visual Studio to containerize the application with Docker, in the Visual Studio, in the **Solution Explorer** window, right-click the **NerdDinner** project, in the right-click menu, select **Add** and, in the cascading menu, click **Container Orchestrator Support**.
1.  In the **Add Container Orchestration Support** dialog box, in the **Container orchestrator** dropdown list, select **Docker Compose** and click **OK**. This will automatically open **Dockerfile** tab in the main pane of the Visual Studio window.
1.  If prompted to start Docker Desktop, click **Yes**. 

    > **Note**: Visual Studio automatically adds required files to the solution, including the **docker-compose** project and **Dockerfile**. It also inspects the project to determine the proper base image to use for your project. In the case of the Nerd Dinner solution, it chose to use **microsoft/aspnet:4.8-windowsservercore-ltsc2019 base image:

    ```csharp
    FROM mcr.microsoft.com/dotnet/framework/aspnet:4.8-windowsservercore-ltsc2019
    ARG source
    WORKDIR /inetpub/wwwroot
    COPY ${source:-obj/Docker/publish} .
    ```
    > **Note**: To run the application locally and debug within the Docker container using Visual Studio and to test the connectivity to the Azure SQL database, set the **docker-compose** as the startup project and launch it.

1.  In the Visual Studio, in the **Solution Explorer** window, right-click the **docker-compose** project, and, in the right-click menu, select **Set as Startup Project**.
1.  In the Visual Studio top level menu, click **Docker Compose**.

    > **Note**: Visual Studio will downloads the base images and subsequently build the image for deployment. Once the build is completed, you will see the application launching in the local browser.

    > **Note**: The download might take a significant amount of time depending on your available bandwidth.

1.  On the lab computer, start **Command Prompt** as Administrator and, from the **Administrator: C:\\windows\\system32\cmd.exe**, run the following to list local docker images and verify that the list includes the image referenced in the **Dockerfile**:

    ```cmd
    docker images
    ```

#### Task 4: Publish a Docker image to Azure Container Registry

In this task, you will use Visual Studio to publish the Docker image you built in the previous step to Azure Container Registry.

1.  In the Visual Studio, in the **Solution Explorer** window, right-click on the **NerdDinner** project and, in the right-click menu, click **Publish**. This will initiate the **Publish** wizard.
1.  On the **Where are you publishing today?** page of the **Publish** wizard, ensure that the **Azure** option is selected, and click **Next**.
1.  On the **Which Azure service would you like to use to host your application?** page of the **Publish** wizard, select the **Azure Container Registry** option, and click **Next**.
1.  On the **Select existing or create a new Azure Container Registry** page of the **Publish** wizard, if needed, click **Sign in**, when prompted, sign in with the user account that has at least the Contributor role in the Azure subscription you are using in this lab.
1.  On the **Select existing or create a new Azure Container Registry** page of the **Publish** wizard, click the plus sign.
1.  In the **Create new** dialog box, specify the following settings and click **Create**:

    | Setting | Value | 
    | --- | --- |
    | DNS Prefix | accept the default value |
    | Subscription | the name of your Azure subscription |
    | Resource group | **az400m15l01a-RG** |
    | SKU | **Standard** |
    | Registry Location | the same Azure region you chose for deployment of Azure SQL database |

1.  Back on the **Select existing or create a new Azure Container Registry** page of the **Publish** wizard, click **Finish**.
1.  In the Visual Studio interface, on the **NerdDinner** tab, click **Publish**.

    > **Note**: Wait until the publish operation completes. As the result of the publish action, the Docker image will be created and pushed to the Azure Container Registry.

1.  Once the publish is successful, switch to the web browser window displaying the Azure portal, in the Azure portal, search for and select the **Container Registries**, and, on the **Azure Container Registry**, click the entry representing the newly created Azure Container registry, on its blade, in the vertical menu on the left hand side, in the **Services** section, click **Repositories**, and verify that it includes the **nerddiner** entry.

#### Task 5: Push the new Docker images from ACR to Azure Container Instances (ACI)

In this task, you will create an Azure Container instance (ACI) and push to it the newly uploaded Docker image from ACR.

> **Note**: Alternatively, you could deploy the container to Azure Kubernetes Service (AKS) or Service Fabric.

> **Note**: You will use Azure CLI to create an Azure Container instance.

1.  In the Azure portal, in the toolbar, click the **Cloud Shell** icon located directly to the right of the search text box. 
1.  If prompted to select either **Bash** or **PowerShell**, select **Bash**. 

    >**Note**: If this is the first time you are starting **Cloud Shell** and you are presented with the **You have no storage mounted** message, select the subscription you are using in this lab, and select **Create storage**. 

1.  From the Bash session in the Cloud Shell pane, run the following to create a new resource group for ACI:

    ```bash
    RESOURCEGROUPNAME1='az400m15l01a-RG'
    LOCATION=$(az group show --name $RESOURCEGROUPNAME1 --query location --output tsv)
    RESOURCEGROUPNAME2='az400m15l02a-RG'
    az group create --name $RESOURCEGROUPNAME2 --location $LOCATION
    ```

1.  From the Bash session in the Cloud Shell pane, run the following command to create an ACI and push the NerdDinner image into it from the ACR you created in the previous task:

    ```bash
    ACINAME='nerddinner'$RANDOM$RANDOM
    ACRNAME=$(az acr list --resource-group $RESOURCEGROUPNAME1 --query '[].name' --output tsv)
    ```

1.  From the Bash session in the Cloud Shell pane, run the following command to create a service principal with the **Pull** permissions to the ACR you created in the previous task and store its password in a shell variable:

    ```bash
    SPPASSWORD=$(az ad sp create-for-rbac \
    --name http://$ACRNAME-pull \
    --scopes $(az acr show --name $ACRNAME --query id --output tsv) \
    --role acrpull \
    --query password \
    --output tsv)
    ```

1.  From the Bash session in the Cloud Shell pane, run the following command to retrieve the name of the service principal you created in the previous step and store its password in a shell variable:

    ```bash  
    SPUSERNAME=$(az ad sp show --id http://$ACRNAME-pull --query appId --output tsv)
    ```

1.  From the Bash session in the Cloud Shell pane, run the following command to retrieve the name of the Azure Container Registry containing the image and store it in a shell variable:

    ```bash  
    ACRLOGINSERVER=$(az acr show --name $ACRNAME --resource-group $RESOURCEGROUPNAME1 --query "loginServer" --output tsv)
    ```

1.  From the Bash session in the Cloud Shell pane, run the following command to create an Azure Container instance based on the image stored in the Azure Container registry, which you access by using the service principal with pull permissions that you created in this task:

    ```bash
    az container create \
    --name $ACINAME \
    --resource-group $RESOURCEGROUPNAME2 \
    --image $ACRLOGINSERVER/nerddinner:latest \
    --registry-login-server $ACRLOGINSERVER \
    --registry-username $SPUSERNAME \
    --registry-password $SPPASSWORD \
    --dns-name-label $ACINAME \
    --os-type windows \
    --query ipAddress.fqdn
    ```

    > **Note**: Wait until the deployment completes. This might take about 5 minutes. The output will include the fully qualified DNS domain name assigned to the Azure Container instance.

1.  From the lab computer, open another web browser tab, navigate to the IP address you identified based on the output of the command that provisioned the Azure Container instance and verify that the NerdDinner application is running.

## Exercise 2: Remove the Azure lab resources

In this exercise, you will remove the Azure resources provisione in this lab to eliminate unexpected charges. 

>**Note**: Remember to remove any newly created Azure resources that you no longer use. Removing unused resources ensures you will not see unexpected charges.

#### Task 1: Remove the Azure lab resources

In this task, you will use Azure Cloud Shell to remove the Azure resources provisione in this lab to eliminate unnecessary charges. 

1.  In the Azure portal, open the **Bash** shell session within the **Cloud Shell** pane.
1.  List all resource groups created throughout the labs of this module by running the following command:

    ```sh
    az group list --query "[?starts_with(name,'az400m15l0')].name" --output tsv
    ```

1.  Delete all resource groups you created throughout the labs of this module by running the following command:

    ```sh
    az group list --query "[?starts_with(name,'az400m15l0')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Note**: The command executes asynchronously (as determined by the --nowait parameter), so while you will be able to run another Azure CLI command immediately afterwards within the same Bash session, it will take a few minutes before the resource groups are actually removed.

## Review

In this lab, you learned how to modernize existing .NET applications with Azure cloud and Windows Containers with minimal code and configuration changes.