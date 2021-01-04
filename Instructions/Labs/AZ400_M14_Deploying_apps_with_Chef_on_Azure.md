---
lab:
    title: 'Lab: Deploying apps with Chef on Azure'
    module: 'Module 14: Third party Infrastructure as Code Tools Available with Azure'
---

# Lab: Deploying apps with Chef on Azure
# Student lab manual

## Lab overview

In this lab, we will deploy the PartsUnlimted MRP application (PU MRP App) to an Azure VM by using Chef Server. The lab illustrates key features and capabilities of Chef in the context of Azure VM-based deployments. 

Chef supports the following deployment options:

- A supported Linux distribution configured as a Chef Server. You can use for this purpose an Azure VM based on a standard Ubuntu image.
- A preconfigured Chef Automate image. You can deploy an Azure VM based on the Chef Automate image. The image includes the Chef Server.
- Hosted Chef Server. You can sign up for a hosted Chef Server account at https://manage.chef.io and use the hosted server to configure your Chef environment.

In this lab, you will deploy an Azure VM based on the Chef Automate image and take advantage of the a free 30-day trial license. 

The lab will consist of the following high-level steps:

- Deploying a Chef Automate server in Azure.
- Configuring a workstation to connect and interact with the Chef Automate server.
- Creating a cookbook and recipe for the PU MRP application and its dependencies in order to automate its installation.
- Uploading a cookbook to the Chef Automate server.
- Creating a role to define a baseline set of cookbooks and attributes that can be applied to multiple servers.
- Creating a node to deploy the configuration to a Linux VM.
- Bootstrapping the node Linux VM to add the role, which effectively relies on Chef to deploy the PU MRP application.

## Lab environment

The lab environment consists of two Azure VMs and your lab computer. The Azure VMs include:

- a Chef Automate that houses the Chef Server
- a Chef node into which you will deploy the PU MRP application by using Chef.

You will use Azure Resource Manager templates to deploy the two Azure VMs. Your lab computer will function as a Chef Workstation that you will use to connect to and manage the Chef Server. Chef workstation can run any current version of the Windows Server or client operating system, Linux, or MacOS. 

## Objectives

After you complete this lab, you will be able to:

- Deploy a Chef Automate server and client by using an Azure Resource Manager template
- Configure a Chef workstation
- Create a Chef cookbook and recipe, and upload them to the Chef Automate server
- Create a Chef role and deploy it to the client

## Lab duration

-   Estimated time: **90 minutes**

## Instructions

### Before you start

#### Sign in to the lab virtual machine

Ensure that you're signed in to your Windows 10 computer by using the following credentials:
    
-   Username: **Student**
-   Password: **Pa55w.rd**

#### Review applications required for this lab

Identify the applications that you'll use in this lab:
    
-   Microsoft Edge

#### Prepare an Azure subscription

-   Identify an existing Azure subscription or create a new one.
-   Verify that you have a Microsoft account or an Azure AD account with the Owner role in the Azure subscription and the Global Administrator role in the Azure AD tenant associated with the Azure subscription. For details, refer to [List Azure role assignments using the Azure portal](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-list-portal) and [View and assign administrator roles in Azure Active Directory](https://docs.microsoft.com/en-us/azure/active-directory/roles/manage-roles-portal#view-my-roles).

### Exercise 1: Deploy the PartsUnlimted MRP application to an Azure VM by using Chef Server 

In this exercise, you will deploy the PartsUnlimted MRP application to an Azure VM by using Chef Server.

#### Task 1: Deploy a Chef Automate server to an Azure VM

In this task, you will deploy and configure an Azure VM by using an Azure Resource Manager template. The Azure VM will serve as a Chef Automate server.

1.  From your lab computer, start a web browser, navigate to the [**Azure Portal**](https://portal.azure.com), and sign in with the user account that has at least the Contributor role in the Azure subscription you are using in this lab.
1.  Open another browser tab and navigate to the [Azure Resource Manager template link](
https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FPartsUnlimitedMRP%2Fmaster%2FLabfiles%2FAZ-400T05-ImplemntgAppInfra%2FLabfiles%2FM04%2FDeployusingChef%2Fenv%2Fdeploychef.json). This will automatically redirect you to the **Custom deployment** blade in the Azure portal.
1.  In the Azure portal, on the **Custom deployment** blade, specify the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Subscription | the name of the Azure subscription you are using in this lab |
    | Resource group | the name of a new resource group **az400m14l02a-RG** |
    | Region | the name of an Azure region where you can deploy Azure VMs |
    | Admin Username | **azureuser** |
    | Admin Password | **Pa55w.rd1234** |
    | Authentication Type | **password** |
    | Diagnostic Storage Account Name | any unique string consisting of between 3 and 24 letters and digits, starting with a letter |
    | Diagnostic Storage Account New or Existing | **new** |
    | Location | the name of the same Azure region where you chose to deploy the Azure VMs (with no spaces) |
    | Public IP Address Name | **az400m14l02b-pip** |
    | Public IP Dns Name | any unique string consisting of between 3 and 63 letters and digits, starting with a letter |
    | Storage Account Name | the same name you chose for the diagnostic storage account |
    | Virtual Network Name | **az400m14l02b-vnet** |
    | Vm Name | **az400m14chs-vm** |

1.  On the **Custom Deployment** blade, click **Review + Create**, verify that the validation process completed, and click **Create**.

    >**Note**: Wait for the deployment to complete. This should take about 10 minutes.

1.  Once the deployment completes, in the Azure portal, use the **Search resources, services, and docs** text box at the top of the page to search for and select the **Virtual machines** resources and, on the **Virtual machines** blade, select the entry representing the newly deployed **az400m14chs-vm** virtual machine.
1.  On the **az400m14chs-vm** blade, hover with the mouse pointer over the **DNS name** entry and copy it to Clipboard. 
1.  Open a new browser tab and navigate to the DNS name you just copied into Clipboard. Use the **https** prefix. 

    >**Note**: Ignore any certificate errors. This is expected. 

1.  Verify that you can access the Chef Automate login prompt.

    >**Note**: You will need to complete the next task before signing into Chef Automate. 

#### Task 2: Configure a Chef Workstation

In this task, you will configure your lab computer as a Chef Workstation by installing the Chef Starter Kit. You will connect to, and manage, your Chef Server from the Chef Workstation.

1.  On your lab computer, start another web browser window, navigate to [the Chef Development Kit (Chef DK)](https://downloads.chef.io/chefdk) download page, select the version of the Chef Development Kit matching your lab virtual machine (Windows 10), click **Download**, in the **Download ChefDK** popup window, provide requested contact information, and click **Download** again.

    >**Note**: As of December 2020, the latest version of the Chef Development Kit is 4.12.0. Chef Development Kit 4.12.0-1-x64 for Windows 10 was tested with this lab.

1.  Once you downloaded the installer for Chef Development Kit, complete the installation with the default settings and double-click on the resulting desktop shortcut to start the **Administrator: ChefDK** PowerShell window. 
1.  In the **Administrator: ChefDK** PowerShell window, run the following to verify the installation:

    ```
    chef verify
    ```

1.  When prompted to accept product licenses, type **yes** and press the **Enter** key.
1.  If the verification fails, in the **Administrator: ChefDK** PowerShell window, run the following to configure your global Git variables with a name and email address and then re-run `chef verify` again: 

    ```
    git config --global user.name "azureuser"
    git config --global user.email "azureuser@partsunlimitedmrp.local"
    ```

    >**Note**: You will use Git to store Chef configuration details.

    >**Note**: Keep the PowerShell window open, you will use it again later in this lab.

    >**Note**: In order to connect and manage Chef Automate, you need to identify the vmId attribute of the Azure VM hosting its installation. The vmId is a unique value that is assigned to each VM in Azure. You can obtain the vmId in a number of ways, including Azure PowerShell and Azure CLI. In this lab, you will use for this purpose PuTTY.

1.  On the lab computer, in the web browser window, navigate to [the PuTTY download page](https://putty.org/), download the PuTTY installer, and run the installation with the default settings.
1.  Following the installation, in the **Start** menu, expand the **PuTTY (64-bit)** folder and click **PuTTY** icon to open the **PuTTY Configuration** window.
1.  In the **PuTTY Configuration** window, in the **Host Name (or IP address)** textbox, type the DNS name you identified at the end of the previous task and click **Open**.
1.  When prompted, in the **PuTTY Security Alert** window, click **Yes**.
1.  When prompted, in the **PuTTY** console window, sign in as **azureuser** with the **Pa55w.rd1234** password.
1.  Once signed in, in the PuTTY console window, run the following to identify the vmId:

    ```bash
    sudo chef-marketplace-ctl show-instance-id
    ```

    >**Note**: Record the value of vmId. You will need it later in this task.

1.  In the PuTTY console window, run the following to terminate the session:

    ```bash
    exit
    ```

    >**Note**: You need the Chef Starter Kit to access the Chef Automate running in the Azure VM that you installed in the previous task. To access the Chef Starter Kit, append /biscotti/setup to the DNS name you identified at the end of the previous task. The result will be a URL resembling the following format (where the `<DNS_name>` placeholder represents the DNS name you identified at the end of the previous task):

    ```
    https://<DNS_name>/biscotti/setup
    ```

1.  In the web browser window, navigate to the URL you constructed in the previous step, ignore any certificate-related error, and proceed to the **Setup Authorization** page. 
1.  On the **Setup Authorization** page, when prompted to provide **vmId**, type the value you identified withing the **PuTTY** session and click **Authorize**.
1.  When prompted to provide details of your Chef Automate Setup, specify the following settings and click **Set Up Chef Automate**:

    | Setting | Value |
    | --- | --- |
    | Enter your first name | **Azure** |
    | Enter your last name | **User** |
    | Enter your username | **azureuser** |
    | Enter your email | **azureuser@partsunlimitedmrp.local** |
    | Enter your password | **Pa55w.rd1234** |
    | Create a support account to enable the Chef Base Support included with your Chef Automate subscription | **Off** |
    | I have read and accept the EULA and Master Agreement | **On** |

    >**Note**: This will apply the **default** organization name since we do not specify an organization name as part of the Chef Automate configuration setup. You could optionally specify an organization name within the Chef Automate server. This organization value will be included in the **knife.rb** file later in this lab. 

1.  When prompted, click **Download Starter Kit**. This will trigger download of a file named starter-kit.zip, which contains pre-configured files that facilitate Chef setup. 

    >**Note**: The Chef Start Kit contains user credentials and certificate details. Different credentials and details are generated during the initial registration and are specific to the user and organizational details that you provided. Make sure not to re-use the Chef Start Kit files across multiple users or registrations.

    >**Note**: Once you have downloaded the Chef Starter Kit, the **Login to Chef Automate** button will become available. 

1.  In the web browser window, click the **Login to Chef Automate** button and sign into Chef Automate by using the username **azureuser** and password **Pa55w.rd1234** that you provided on the **Chef Automate Setup** web page. This will display the Chef Automate dashboard.
1.  On your lab computer, open File Explorer and extract the contents of the **Chef Starter Kit** zip archive from the **Downloads** folder to a folder **C:\\Labfiles\\chef**.
1.  In File Explorer, navigate to the **C:\\Labfiles\\chef\\chef-repo\\.chef** folder and open the **knife.rb** file in Notepad. 
1.  In the **knife.rb** file, verify that the fully qualified domain name included in the **chef_server_url** name matches the name Azure VM DNS name you identified in the previous task and that the organization name is set to **default** and close the Notepad window.
1.  Switch back to the **Administrator: Chef DK** PowerShell window and run the following to initiate a new repository:

    ```powershell
    Set-Location -Path 'C:\Labfiles\chef\chef-repo'
    git init
    git add -A
    git commit -m "starter kit commit"
    ```

    >**Note**: Our Chef Server has an SSL certificate that is not trusted. As a result, we have to manually trust the SSL certificate for our Chef Workstation to communicate with the Chef Server. To change this, we will import a valid SSL certificate for Chef by running the following command:

    ```chef
    knife ssl fetch
    ```

1.  In the **Administrator: Chef DK** PowerShell window, run the following to list the current content of the chef-repo:

    ```powershell
    Get-ChildItem -Path '.\' -Recurse
    ```

1.  In the **Administrator: Chef DK** PowerShell window, run the following to synchronize the content of the Chef Workstation repo with the content of your Chef Automate Azure VM-based repo. 

    ```chef
    knife download /
    ```

    >**Note**: The command downloads the entire Chef repo from your Chef Automate server.

    >**Note**: You may see errors in relation to acls, depending on your user account details, but you can ignore these errors for the purposes of this lab.

1.  In the **Administrator: Chef DK** PowerShell window, run the following again to list the current content of the chef-repo after running the `knife download /` command and note the additional files and folders that have been created in the chef-repo directory:

    ```powershell
    Get-ChildItem -Path '.\' -Recurse
    ```

1.  In the **Administrator: Chef DK** PowerShell window, run the following to commit the newly added files into the Git repository:

    ```git
    git add -A
    git commit -m "knife download commit"
    ```

#### Task 3: Create a Chef cookbook and recipe, and upload them to the Chef Automate server

In this task you will create a cookbook and recipe for the PU MRP app's dependencies to automate the installation of the PU MRP application, and upload those cookbooks and recipes to the Chef server.

1.  In the **Administrator: Chef DK** PowerShell window, run the following to generate a cookbook template in the **cookbooks** subdirectory of the **chef-repo** by using the Chef knife tool:

    ```chef
    Set-Location -Path '.\cookbooks'
    chef generate cookbook mrpapp
    ```

    >**Note**: A cookbook is a set of tasks for configuring an application or feature. It defines a scenario and everything required to support that scenario. Within a cookbook, there are a series of recipes that define a set of actions for the cookbook to perform. Cookbooks and recipes are written in the Ruby language. The chef generate cookbook command we ran created an **mrpapp** directory in the **chef-repo\\cookbooks** directory. The mrpapp directory contains all of the boilerplate code that defines a cookbook and a default recipe.

1.  In the **Administrator: Chef DK** PowerShell window, run the following to open the file **metadata.rb** for editing:

    ```powershell
    notepad .\mrpapp\metadata.rb
    ```

    >**Note**: Cookbooks and recipes can leverage other cookbooks and recipes. Our cookbook will use a pre-existing recipe for managing APT repositories. 

1.  In the Notepad window displaying content of the file **metadata.rb**, add the following line at the end of the file, then save and close the Notepad window:

    ```chef
    depends 'apt'
    ```

    >**Note**: Next we need to install three dependencies for our recipe: the **apt** cookbook, the **windows** cookbook, and the **chef-client** cookbook. You can download the three cookbooks from [the official Chef cookbook repository](https://supermarket.chef.io/cookbooks) and install them by using the `knife cookbook site` command.

1.  In the **Administrator: Chef DK** PowerShell window, run the following to download and install the cookbooks by using the `knife cookbook site` command:

    ```chef
    knife cookbook site install apt
    knife cookbook site install windows
    knife cookbook site install chef-client
    ```

    >**Note**: Next, you need to copy the full contents of the **default.rb** recipe from [the Microsoft Parts Unlimited MRP GitHub repository](https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/M04/DeployusingChef/final/default.rb).

1.  From the lab computer, start another web browser window, navigate to **https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/M04/DeployusingChef/final/default.rb** to display the **default.rb** recipe in the RAW format and copy the content of the web page into Clipboard.
1.  In the **Administrator: Chef DK** PowerShell window, run the following to open the file **C:\\Labfiles\\chef\\chef-repo\\cookbooks\\mrpapp\\recipes\\default.rb** in Notepad:

    ```powershell
    notepad .\mrpapp\recipes\default.rb
    ```

1.  In the Notepad window, review the content of the file and verify that it looks as follows:

    ```chef
    #
    # Cookbook:: mrpapp
    # Recipe:: default
    #
    # Copyright:: 2020, The Authors, All Rights Reserved.
    ```

1.  In the Notepad window, append the content of Clipboard to the end of the file, starting with a new line, save your changes, and close the Notepad window.

    >**Note**: The following explains how the recipe we copied will provision our application, explaining the meaning of the content of **default.rb** line-by-line:

    >**Note**: The recipe will run the `apt` resource. This will cause our recipe to execute `apt-get update` command prior to running. This command makes sure the versions of local packages are up-to-date:

    ```chef
     	# Runs apt-get update
     	include_recipe "apt"
    ```

    >**Note**: Next, we add an `apt_repository` resource to make sure that the **OpenJDK** repository is part of our apt repository list and it is up-to-date:

    ```chef
 	# Add the Open JDK apt repo
 	apt_repository 'openJDK' do
 		uri 'ppa:openjdk-r/ppa'
 		distribution 'trusty'
 	end
    ```

    >**Note**: Next, we use the `apt-package` recipe to ensure that **OpenJDK** and **OpenJRE** are installed. We use the headless versions as the full versions depend on a legacy package that we do not want to use:

    ```chef
 	# Install JDK and JRE
 	apt_package 'openjdk-8-jdk-headless' do
 		action :install
 	end

 	apt_package 'openjdk-8-jre-headless' do
 		action :install
     	end
    ```

    >**Note**: Next, we set the `JAVA_HOME` and `PATH` environment variables to reference OpenJDK:

    ```chef
 	# Set Java environment variables
 	ENV['JAVA_HOME'] = "/usr/lib/jvm/java-8-openjdk-amd64"
     	ENV['PATH'] = "#{ENV['PATH']}:/usr/lib/jvm/java-8-openjdk-amd64/bin"
    ```

    >**Note**: Next, we install the MongoDB database engine and Tomcat web server:

    ```chef
 	# Install MongoDB
 	apt_package 'mongodb' do
 		action :install
 	end

 	# Install Tomcat 7
 	apt_package 'tomcat7' do
 		action :install
     	end
    ```

    >**Note**: At this point, all of our dependencies are installed. We can now start configuring our application. First, we need to ensure that our MongoDB database has some baseline data in it. The `remote_file` resource downloads a file to a specified location. This task is idempotent, which, in this context, means that if the file on the server has the same checksum as the local file, no action will be taken. We also use the `notifies` command. This way, if a new version of the file is present when the resource runs, a notification is sent to the specified resource, telling it to run the new file.

    ```chef
 	# Load MongoDB data
 	remote_file 'mongodb_data' do
 		source 'https://github.com/Microsoft/PartsUnlimitedMRP/tree/master/deploy/MongoRecords.js'
 		path './MongoRecords.js'
 		action :create
 		notifies :run, "script[mongodb_import]", :immediately
     	end
    ```

    >**Note**: Next, we use a `script` resource to set the command line script that runs to load the MongoDB data we downloaded in the previous step. The `script` resource has its `action` parameter set to `nothing`, which means the script will only run when it receives a notification to run. The only time this resource will run is when it is notified by the `remote_file` resource we specified in the previous step. So every time a new version of the **MongoRecord.js** file is uploaded, the recipe will download and import it. If the **MongoRecords.js** file does not change, nothing is downloaded or imported!

    ```chef
 	script 'mongodb_import' do
 		interpreter "bash"
 		action :nothing
 		code "mongo ordering MongoRecords.js"
     	end
    ```

    >**Note**: Next, we set the port that Tomcat will run our PU MRP application on. This uses a `script` resource to invoke a regular expression to update the **/etc/tomcat7/server.xml** file. The `not_if` action is a guard statement. If the code in the `not_if` action returns `true`, the resource will not execute. This ensures that the script will only run if it needs to run.

    >**Note**: We are referencing an attribute called `#{node['tomcat']['mrp_port']}`. We have not defined this value yet, but we will in the next task. With attributes, it is possible to set variables, so we can deploy PU MRP application on one port on one server and on a different port on another server. If the port changes, we use `notifies` to invoke a service restart.

    ```chef
 	# Set tomcat port
 	script 'tomcat_port' do
 		interpreter "bash"
 		code "sed -i 's/Connector port=\".*\" protocol=\"HTTP\\/1.1\"$/Connector port=\"#{node['tomcat']['mrp_port']}\" protocol=\"HTTP\\/1.1\"/g' /etc/tomcat7/server.xml"
 		not_if "grep 'Connector port=\"#{node['tomcat']['mrp_port']}\" protocol=\"HTTP/1.1\"$' /etc/tomcat7/server.xml"
 		notifies :restart, "service[tomcat7]", :immediately
     	end
    ```

    >**Note**: Now we can download the PU MRP application and start running it in Tomcat. If we get a new version, it signals the Tomcat service to restart:

    ```chef
 	# Install the PU MRP app, restart the Tomcat service if necessary
 	remote_file 'mrp_app' do
 		source 'https://github.com/Microsoft/PartsUnlimitedMRP/tree/master/builds/mrp.war'
 		action :create
 		notifies :restart, "service[tomcat7]", :immediately
     	end
    ```

    >**Note**: We can define the Tomcat service's desired state, which in this particular case, means that the service should be running. This will cause the script to check the state of the Tomcat service, and, if necessary, start it.

    ```chef
 	# Ensure Tomcat is running
 	service 'tomcat7' do
 		action :start
 	end
    ```

    >**Note**: Finally, we can make sure the `ordering_service` is running. This uses a combination of `remote_file` and `script` resources to check if the ordering_service needs to be killed and restarted. 

    ```chef
 	remote_file 'ordering_service' do
 		source 'https://github.com/Microsoft/PartsUnlimitedMRP/tree/master/builds/ordering-service-0.1.0.jar'
 		path './ordering-service-0.1.0.jar'
 		action :create
 		notifies :run, "script[stop_ordering_service]", :immediately
 	end

 	# Kill the ordering service
 	script 'stop_ordering_service' do
 		interpreter "bash"
 	# Only run when notifed
 		action :nothing
 		code "pkill -f ordering-service"
 		only_if "pgrep -f ordering-service"
 	end

 	# Start the ordering service.
 	script 'start_ordering_service' do
 		interpreter "bash"
 		code "/usr/lib/jvm/java-8-openjdk-amd64/bin/java -jar ordering-service-0.1.0.jar &"
 		not_if "pgrep -f ordering-service"
 	end
    ```

1.  In the **Administrator: Chef DK** PowerShell window, run the following to commit the files we added to the Git repository:

    ```git
    git add .
    git commit -m "mrp cookbook commit"
    ```

    >**Note**: Now that we have created a recipe and installed the dependencies, we can upload our cookbooks and recipes to the Chef Automate server.

1.  In the **Administrator: Chef DK** PowerShell window, run the following to upload our cookbooks and recipes to the Chef Automate server by using the `knife cookbook upload` command:

    ```chef
    knife cookbook upload mrpapp --include-dependencies
    knife cookbook upload chef-client --include-dependencies
    ```

#### Task 4: Create a Chef role

In this exercise, you will use the `knife` command to create a role. The role will define a baseline set of cookbooks and attributes that can be applied to multiple servers.

>**Note**: For more information, refer to [the Chef Documentation page Knife Role](https://docs.chef.io/knife_role.html).

1.  In the **Administrator: Chef DK** PowerShell window, run the following to open the file **knife.rb** in Notepad:

    ```powershell
    notepad C:\Labfiles\chef\chef-repo\.chef\knife.rb
    ```

1.  In the Notepad window, append the following to the end of the file, starting with a new line, save your change, and close the Notepad window:

    ```chef
    knife[:editor] = "notepad"
    ```

    >**Note**: Adding this line designates Notepad as the editor to open the knife.rb when we create a role, which will take place in the next step. If you do not specify an editor in the knife.rb, you will receive an error message instructing you to set the editor environment variable.

1.  In the **Administrator: Chef DK** PowerShell window, run the following to create the role, which we will name **partsrole**.

    ```chef
    knife role create partsrole
    ```

    >**Note**: The command will open the Notepad window displaying JSON content that represents the structure of the role definition:

    ```json
    {
      "name": "partsrole",
      "description": "",
      "json_class": "Chef::Role",
      "default_attributes": {

      },
      "override_attributes": {

      },
      "chef_type": "role",
      "run_list": [

      ],
      "env_run_lists": {

      }
    }
    ```

1.  In the Notepad window, change the `default_attributes` section to:

    ```json
      "default_attributes": {
          "tomcat": {
              "mrp_port": 9080
          }
      },
    ```

1. Update the override_attributes to:

    ```json
      "override_attributes": {
          "chef_client": {
              "interval": "60",
              "splay": "1"
          }
      },
    ```

1. Update the run_list to:

    ```json
      "run_list": [
          "recipe[mrpapp]",
          "recipe[chef-client::service]"
      ],
    ```

1.  Verify that the completed file has the following content, save the changes, and close the Notepad window:

    ```json
    {
      "name": "partsrole",
      "description": "",
      "json_class": "Chef::Role",
      "default_attributes": {
          "tomcat": {
              "mrp_port": 9080
          }
      },
      "override_attributes": {
          "chef_client": {
              "interval": "60",
              "splay": "1"
          }
      },
      "chef_type": "role",
      "run_list": [
          "recipe[mrpapp]",
          "recipe[chef-client::service]"
      ],
      "env_run_lists": {

      }
    }
    ```

1.  Switch back to the **Administrator: Chef DK** PowerShell window and verify that the `knife role create partsrole` has successfully completed. 

    >**Note**: Make sure to close the Notepad window. The successful completion is indicated by the `Created role[partsrole]` message. 

#### Task 5: Bootstrap the PU MRP app server and deploy the PU MRP application

In this exercise, you will use **knife** to bootstrap the PU MRP application server and assign to it the PU MRP application role.

>**Note**: You will start by provisioning a Linux VM that will be subsequently configured as a Chef client and into which you will deploy the PU MRP application by using the role you created in the previous task.

1.  From the lab computer, start a web browser, navigate to an [Azure Resource Manager template link](
https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FPartsUnlimitedMRP%2Fmaster%2FLabfiles%2FAZ-400T05-ImplemntgAppInfra%2FLabfiles%2FM04%2FDeployusingChef%2Fenv%2Fdeploylinux.json). This will automatically redirect you to the **Custom deployment** blade in the Azure portal.
1.  In the Azure portal, on the **Custom deployment** blade, click **Edit template**
1.  On the **Edit template** blade, in the template outline pane, expand the **Variables** section and click **vmSize**.
1.  In the template details section, change `"vmSize": "Standard_A1",` to "vmSize": "Standard_D2s_v3",` and click **Save**.
1.  Back on the **Custom deployment** blade, specify the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Subscription | the name of the Azure subscription you are using in this lab |
    | Resource group | the name of a new resource group **az400m14l02b-RG** |
    | Region | the name of an Azure region where you deployed the Chef Automate Azure VM earlier in this lab |
    | Admin Username | **azureuser** |
    | Admin Password | **Pa55w.rd1234** |
    | Dns Label Prefix | any unique string consisting of between 3 and 63 letters and digits, starting with a letter |
    | Ubuntu OS Version | **16.04.0-LTS** |

1.  On the **Custom Deployment** blade, click **Review + Create**, verify that the validation process completed, and click **Create**.

    >**Note**: Wait for the deployment to complete. This should take about 2 minutes.

1.  Once the deployment completes, in the Azure portal, use the **Search resources, services, and docs** text box at the top of the page to search for and select the **Virtual machines** resources and, on the **Virtual machines** blade, select the entry representing the newly deployed virtual machine named **mrpUbuntuVM**.
1.  On the **mrpUbuntuVM** blade, hover with the mouse pointer over the **DNS name** entry and copy it to Clipboard. 
1.  Switch back to the **Administrator: Chef DK** PowerShell window, run the following to bootstrap the newly deployed Azure VM by using the **knife** tool (where the `<DNS_name>` placeholder represents the DNS name you identified at the end of the previous task):

    ```chef
    knife bootstrap <DNS_name> --ssh-user azureuser --ssh-password Pa55w.rd1234 --node-name mrp-app --run-list role[partsrole] --sudo --verbose
    ```
1.  When prompted whether to continue connecting, type **Y** and press the **Enter** key.

    >**Note**: Wait for the onboarding script to complete. This should take about 3 minutes. The script performs the following steps:

    - Installing Chef client components.
    - Assigning the `mrp` Chef role.
    - Running the `mrpapp` recipe.

1.  Once the script completes, switch to the browser window displaying the Azure portal, open another browser tab, and navigate to the URL that consist of the **http://** prefix, followed by the DNS name of the `mrpUbuntuVM` Azure VM, and has the `:9080/mrp/` suffix, resulting in the following format `http://<DNS_name>:9080/mrp/` (where the `<DNS_name>` placeholder represents the DNS name you identified at the end of the previous task).
1.  Verify that the browser tab displays the landing page of the PU MRP application.
1.  Switch to the browser window displaying the Chef Automate server dashboard, at the top of the dashboard page, click **Nodes**, and verify that it contains a single node labeled as **mrp-app**.

### Exercise 2: Remove the Azure lab resources

In this exercise, you will remove the Azure resources provisione in this lab to eliminate unexpected charges. 

>**Note**: Remember to remove any newly created Azure resources that you no longer use. Removing unused resources ensures you will not see unexpected charges.

#### Task 1: Remove the Azure lab resources

In this task, you will use Azure Cloud Shell to remove the Azure resources provisione in this lab to eliminate unnecessary charges. 

1.  In the Azure portal, open the **Bash** shell session within the **Cloud Shell** pane.
1.  List all resource groups created throughout the labs of this module by running the following command:

    ```sh
    az group list --query "[?starts_with(name,'az400m14l02')].name" --output tsv
    ```

1.  Delete all resource groups you created throughout the labs of this module by running the following command:

    ```sh
    az group list --query "[?starts_with(name,'az400m14l02')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Note**: The command executes asynchronously (as determined by the --nowait parameter), so while you will be able to run another Azure CLI command immediately afterwards within the same Bash session, it will take a few minutes before the resource groups are actually removed.

## Review

In this lab, you learned how to deploy the PartsUnlimted MRP application (PU MRP App) to an Azure VM by using Chef Server. 