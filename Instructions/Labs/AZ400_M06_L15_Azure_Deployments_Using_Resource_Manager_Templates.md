---
lab:
  title: "Deployments using Azure Bicep templates"
  module: "Module 06: Manage infrastructure as code using Azure and DSC"
---

# Deployments using Azure Bicep templates

# Student lab manual

## Lab requirements

- This lab requires **Microsoft Edge** or an [Azure DevOps supported browser.](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers)

- **Set up an Azure DevOps organization:** If you don't already have an Azure DevOps organization that you can use for this lab, create one by following the instructions available at [Create an organization or project collection](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops).

- Identify an existing Azure subscription or create a new one.

- Verify that you have a Microsoft account or an Azure AD account with the Owner role in the Azure subscription and the Global Administrator role in the Azure AD tenant associated with the Azure subscription.

- [Visual Studio Code](https://code.visualstudio.com/). This will be installed as part of prerequisites for this lab.

## Lab overview

In this lab, you'll create an Azure Bicep template and modularize it using the Azure Bicep Modules concept. You'll then modify the main deployment template to use the module and finally deploy the all the resources to Azure.

## Objectives

After you complete this lab, you will be able to:

- Understand and create a Azure Bicep Templates.
- Create a reusable Bicep module for storage resources.
- Upload Linked Template to Azure Blob Storage and generate SAS token.
- Modify the main template to use the module.
- Modify the main template to update dependencies.
- Deploy all the resources to Azure using Azure Bicep Templates.

## Estimated timing: 60 minutes

## Instructions

### Exercise 0: Configure the lab prerequisites

In this exercise, you will set up the prerequisites for the lab, which include Visual Studio Code.

#### Task 1: Install and configure Git and Visual Studio Code

In this task, you will install Visual Studio Code. If you have already implemented this prerequisite, you can proceed directly to the next task.

1. If you don't have Visual Studio Code installed yet, from your lab computer, start a web browser, navigate to the [Visual Studio Code download page](https://code.visualstudio.com/), download it, and install it.

### Exercise 1: Author and deploy Azure Bicep templates

In this lab, you will create an Azure Bicep template and a template module. You will then modify the main deployment template to use the template module and update the dependencies, and finally deploy the templates to Azure.

#### Task 1: Create Azure Bicep template

In this task, you will use Visual Studio Code to create an Azure Bicep template

1. From your lab computer, start Visual Studio Code, in Visual Studio Code, click the **File** top level menu, in the dropdown menu, select **Preferences**, in the cascading menu, select **Extensions**, in the **Search Extensions** textbox, type **Bicep**, select the one published by Microsoft, and click **Install** to install the Azure Bicep language support.
1. In a web browser, connect to **<https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.compute/vm-simple-windows/main.bicep>**. Click on **Raw** option for the file. Copy the contents of the code window and paste it into Visual Studio Code editor.

   > **Note**: Rather than creating a template from scratch we will use one of the [Azure Quickstart Templates](https://azure.microsoft.com/en-us/resources/templates/) named **Deploy a simple Windows template VM**. The templates are downloadable the templates from GitHub - [vm-simple-windows](https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.compute/vm-simple-windows).

1. On your lab computer, open File Explorer and create the following local folder that will serve to store templates:

   - **C:\\templates**

1. Switch back to Visual Studio Code window with our main.bicep template, click the **File** top level menu, in the dropdown menu, click **Save as**, and save the template as **main.bicep** in the newly created local folder **C:\\templates**.
1. Review the template to get a better understanding of its structure. There are five resource types included in the template:

   - Microsoft.Storage/storageAccounts
   - Microsoft.Network/publicIPAddresses
   - Microsoft.Network/virtualNetworks
   - Microsoft.Network/networkInterfaces
   - Microsoft.Compute/virtualMachines

1. In Visual Studio Code, save the file again, but this time choose **C:\\templates** as the destination and **storage.bicep** as the file name.

   > **Note**: We now have two identical JSON files: **C:\\templates\\main.bicep** and **C:\\templates\\storage.bicep**.

#### Task 2: Create a template module for storage resources

In this task, you will modify the templates you saved in the previous task such that the storage template module **storage.bicep** will create a storage account only, it will be imported by the first template. The storage template module needs to pass a value back to the main template, **main.bicep**, and this value will be defined in the outputs element of the storage template module.

1. In the **storage.bicep** file displayed in the Visual Studio Code window, under the **resources section**, remove all the resource elements except the **storageAccounts** resource. It should result in a resource section looking as follows:

   ```bicep
   resource storageAccount 'Microsoft.Storage/storageAccounts@2022-05-01' = {
     name: storageAccountName
     location: location
     sku: {
       name: 'Standard_LRS'
     }
     kind: 'Storage'
   }
   ```

1. Next, remove all the variables definitions:

   ```bicep
   var storageAccountName = 'bootdiags${uniqueString(resourceGroup().id)}'
   var nicName = 'myVMNic'
   var addressPrefix = '10.0.0.0/16'
   var subnetName = 'Subnet'
   var subnetPrefix = '10.0.0.0/24'
   var virtualNetworkName = 'MyVNET'
   var networkSecurityGroupName = 'default-NSG'
   var securityProfileJson = {
     uefiSettings: {
       secureBootEnabled: true
       vTpmEnabled: true
     }
     securityType: securityType
   }
   var extensionName = 'GuestAttestation'
   var extensionPublisher = 'Microsoft.Azure.Security.WindowsAttestation'
   var extensionVersion = '1.0'
   var maaTenantName = 'GuestAttestation'
   var maaEndpoint = substring('emptyString', 0, 0)
   ```

1. Next, remove all parameter values except location and add the following parameter code, resulting in the following outcome:

   ```bicep
   @description('Location for all resources.')
   param location string = resourceGroup().location

   @description('Name for the storage account.')
   param storageAccountName string
   ```

1. Next, at the end of the file, remove the current output and add a new one called storageURI output value. Modify the output so it looks like the below.

   ```bicep
   output storageURI string = storageAccount.properties.primaryEndpoints.blob
   ```

1. Save the storage.bicep template module. The storage template should now look as follows:

   ```bicep
   @description('Location for all resources.')
   param location string = resourceGroup().location

   @description('Name for the storage account.')
   param storageAccountName string

   resource storageAccount 'Microsoft.Storage/storageAccounts@2022-05-01' = {
     name: storageAccountName
     location: location
     sku: {
       name: 'Standard_LRS'
     }
     kind: 'Storage'
   }

   output storageURI string = storageAccount.properties.primaryEndpoints.blob
   ```

#### Task 3: Modify the main template to use the template module

In this task, you will modify the main template to reference the template module you created in the previous task.

1. In Visual Studio Code, click the **File** top level menu, in the dropdown menu, select **Open File**, in the Open File dialog box, navigate to **C:\\templates\\main.bicep**, select it, and click **Open**.
1. In the **main.bicep** file, in the resource section remove the storage resource element

   ```bicep
   resource storageAccount 'Microsoft.Storage/storageAccounts@2022-05-01' = {
     name: storageAccountName
     location: location
     sku: {
       name: 'Standard_LRS'
     }
     kind: 'Storage'
   }
   ```

1. Next, add the following code directly in the same location where the newly deleted storage resource element was:

   ```bicep
   module storageModule './storage.bicep' = {
     name: 'linkedTemplate'
     params: {
       location: location
       storageAccountName: storageAccountName
     }
   }
   ```

1. We also need to modify the reference to the storage account blob URI in our virtual machine resource to use the output of the module instead. Find the virtual machine resource and replace the diagnosticsProfile section with the following:

   ```bicep
   diagnosticsProfile: {
     bootDiagnostics: {
       enabled: true
       storageUri: storageModule.outputs.storageURI
     }
   }
   ```

1. Review the following details in the main template:

   - A module in the main template is used to link to another template.
   - The module has a symbolic name called storageModule. This name is used for configuring any dependencies.
   - You can only use Incremental deployment mode when using template modules.
   - A relative path is used for your template module.
   - Use parameters to pass values from the main template to the template modules.

> **Note**: With Azure ARM Templates, you would have used a storage account to upload the linked template to make it easier for others to use them. With Azure Bicep modules, you have the option to upload them to Azure Bicep Module registry which has both public and private registry options. More information can be found on the [Azure Bicep documentation](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/modules#file-in-registry).

1. Save the template.

#### Task 5: Deploy resources to Azure by using template modules

> **Note**: You can deploy templates in several ways, such as using Azure CLI installed locally or from the Azure Cloud Shell or from a CI/CD pipeline. In this lab, you will use Azure CLI from the Azure Cloud Shell.

> **Note**: In contrast to ARM templates you cannot use Azure portal to directly deploy Bicep templates.

> **Note**: To use Azure Cloud Shell, you will upload the both the main.bicep and storage.bicep files into your Cloud Shell's home directory.

> **Note**: Currently, Azure CLI does not support deploying remote Bicep files. You can build the bicep files to get the ARM Template JSON and then upload them to an storage account, then deploy them remotely.

1. On the lab computer, in the web browser displaying the Azure Portal, click the **Cloud Shell** icon to open Cloud Shell.
   > **Note**: If you have the PowerShell session from earlier in this exercise still active, switch to Bash (next step).
1. In the Cloud Shell pane, click **PowerShell**, in the dropdown menu, click **Bash** and, when prompted, click **Confirm**.
1. In the Cloud Shell pane, click the **Upload/download files** icon and, in the dropdown menu, click **Upload**.
1. In the **Open** dialog box, navigate to and select **C:\\templates\\main.bicep** and click **Open**.
1. Follow the same steps to upload the **C:\\templates\\storage.bicep** file too.
1. From a **Bash** session in the Cloud Shell pane, run the following to perform a deployment by using a newly uploaded template:

   ```bash
   az deployment group what-if --name az400m06l15deployment --resource-group az400m06l15-RG --template-file main.bicep
   ```

1. When prompted to provide the value for 'adminUsername', type **Student** and press the **Enter** key.
1. When prompted to provide the value for 'adminPassword', type **Pa55w.rd1234** and press the **Enter** key. (Password typing will not be shown)
1. Review the result of this command which validates your deployment and let's you know if there is any errors in your templates. This is very valuable especially when deploying templates with many resources and in business critical cloud environments.

1. From a **Bash** session in the Cloud Shell pane, run the following to perform a deployment by using a newly uploaded template:

   ```bash
   LOCATION='<region>'
   ```
   > **Note**: replace the name of the region with a region close to your location. If you do not know what locations are available, run the `az account list-locations -o table` command.
  
   ```bash
   az group create --name az400m06l15deployment --location $LOCATION
   ```

   ```bash   
   az deployment group create --name az400m06l15deployment --resource-group az400m06l15-RG --template-file main.bicep
   ```

1. When prompted to provide the value for 'adminUsername', type **Student** and press the **Enter** key.
1. When prompted to provide the value for 'adminPassword', type **Pa55w.rd1234** and press the **Enter** key. (Password typing will not be shown)

1. If you receive errors when running the above command to deploy the template, try the following:

   - If you have multiple Azure subscriptions ensure you have set the subscription context to the correct one where the resource group is deployed.
   - Ensure that the linked template is accessible via the URI you specified.

> **Note**: As a next step, you could now modularize the remaining resource definitions in the main deployment template, such as the network and virtual machine resource definitions.

> **Note**: If you are not planning on using the deployed resources, you should delete them to avoid associated charges. You can do so simply by deleting the resource group **az400m06l15-RG**.

### Exercise 2: Remove the Azure lab resources

In this exercise, you will remove the Azure resources provisioned in this lab to eliminate unexpected charges.

> **Note**: Remember to remove any newly created Azure resources that you no longer use. Removing unused resources ensures you will not see unexpected charges.

#### Task 1: Remove the Azure lab resources

In this task, you will use Azure Cloud Shell to remove the Azure resources provisioned in this lab to eliminate unnecessary charges.

1. In the Azure portal, open the **Bash** shell session within the **Cloud Shell** pane.
1. List all resource groups created throughout the labs of this module by running the following command:

   ```bash
   az group list --query "[?starts_with(name,'az400m06l15-RG')].name" --output tsv
   ```

1. Delete all resource groups you created throughout the labs of this module by running the following command:

   ```bash
   az group list --query "[?starts_with(name,'az400m06l15-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

   > **Note**: The command executes asynchronously (as determined by the --nowait parameter), so while you will be able to run another Azure CLI command immediately afterwards within the same Bash session, it will take a few minutes before the resource groups are actually removed.

## Review

In this lab, you learned how to create an Azure Resource manager template, modularize it by using a linked template, modify the main deployment template to call the linked template and updated dependencies, and finally deploy the templates to Azure.
