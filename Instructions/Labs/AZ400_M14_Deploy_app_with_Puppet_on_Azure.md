---
lab:
    title: 'Lab: Deploy app with Puppet on Azure'
    az400Module: 'Module 14: Third party Infrastructure as Code Tools Available with Azure'
---

# Lab: Deploy app with Puppet on Azure
# Student lab manual

## Lab overview

In this lab, we will deploy the PartsUnlimted MRP application (PU MRP App) to an Azure VM by using Puppet from Puppet Labs. 

Puppet is a configuration management system that allows you to automate the provisioning and configuring of machines, by describing the state of your Infrastructure as Code. 

The lab will consist of the following high-level steps:

- Provisioning a Puppet Master and Node in Azure VMs by using Azure Resource Manager templates
- Installing Puppet Agent on the node
- Configuring the Puppet production environment
- Testing the production environment configuration
- Creating a Puppet program to describe the prerequisites for the PU MRP app
- Running the Puppet configuration on the Node

## Lab environment

The lab environment consists of two Azure VMs and your lab computer. The Azure VMs include:

- A Puppet Node that will host the PU MRP app. The only task you will perform on the node is to install the Puppet Agent. The Puppet Agent can run on Linux or Windows. For this lab, we will use an Ubuntu Server.
- A Puppet Master that manages the configuration applied to the Node through Puppet programs. The Puppet Master must run on Linux. For this lab, we will also use an Ubuntu Server.

You will use Azure Resource Manager templates to deploy the two Azure VMs. Your lab computer will function as an administrative workstation that you will use to connect to and manage the Puppet Master. 

## Objectives

After you complete this lab, you will be able to:

- Provision a Puppet Master and Node in Azure VMs by using Azure Resource Manager templates
- Install Puppet Agent on the node
- Configure the Puppet production environment
- Test the production environment configuration
- Create a Puppet program
- Run the Puppet configuration on the Node

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

### Exercise 1: Deploy the PartsUnlimted MRP application to an Azure VM by using Puppet

In this exercise, you will deploy the PartsUnlimted MRP application to an Azure VM by using Puppet.

#### Task 1: Provision Azure VMs that will serve as the Puppet Master and its node

In this task, you will deploy and configure two Azure VMs by using Azure Resource Manager templates. The first of them will serve as the Puppet Master and the second one will become its managed node.

1.  From your lab computer, start a web browser, navigate to the [**Azure Portal**](https://portal.azure.com), and sign in with the user account that has at least the Contributor role in the Azure subscription you are using in this lab.
1.  Open another browser tab and navigate to the [Azure Resource Manager template link](
https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FPartsUnlimitedMRP%2Fmaster%2FLabfiles%2FAZ-400T05-ImplemntgAppInfra%2FLabfiles%2FM04%2FPuppet%2Fenv%2FPuppetPartsUnlimitedMRP.json). This will automatically redirect you to the **Custom deployment** blade in the Azure portal.

1.  In the Azure portal, on the **Custom deployment** blade, click **Edit template**
1.  On the **Edit template** blade, in the template outline pane, expand the **Variables** section and click **vmSize**.
1.  In the template details section, change `"pmVmSize": "Standard_D2_V2",` to "vmSize": "Standard_D2s_v3",`.
1.  In the template details section, change `"mrpVmSize": "Standard_A2",` to "vmSize": "Standard_D2s_v3",` and click **Save**.
1.  Back on the **Custom deployment** blade, specify the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Subscription | the name of the Azure subscription you are using in this lab |
    | Resource group | the name of a new resource group **az400m14l03a-RG** |
    | Region | the name of an Azure region where you can deploy Azure VMs |
    | Pm Admin Username | **azureuser** |
    | Pm Admin Password | **Pa55w.rd1234** |
    | Pm Dns Name for Public IP | any unique string consisting of between 3 and 63 letters and digits, starting with a letter |
    | Pm Console Password | **Pa55w.rd1234** |
    | Mrp Admin Username | **azureuser** |
    | Mrp Admin Password | **Pa55w.rd1234** |
    | Mrp Dns Name for Public IP | any unique string consisting of between 3 and 63 letters and digits, starting with a letter |

    >**Note**: Record the DNS names you specified during deployment. The first one (**Pm**) will be assigned to the Puppet Master. The second one (**Mrp**) will be assigned to the Puppet node. 

1.  On the **Custom Deployment** blade, click **Review + Create**, verify that the validation process completed, and click **Create**.

    >**Note**: Wait for the deployment to complete. This should take about 10 minutes.

1.  Once the deployment completes, in the Azure portal, use the **Search resources, services, and docs** text box at the top of the page to search for and select the **Virtual machines** resources and, on the **Virtual machines** blade, select the entry representing the newly deployed Azure VM that will be used as a Puppet node.
1.  On the virtual machine blade, identify and record the value of the **DNS name** property.
1.  Navigate back to the **Virtual machines** blade and select the entry representing the newly deployed Azure VM hosting the Puppet Master.
1.  On the virtual machine blade, hover with the mouse pointer over the **DNS name** entry and copy it to Clipboard. 
1.  Open a new browser tab and navigate to the DNS name you just copied into Clipboard. Use the **https** prefix. This will display the Puppet Master Console sign-in page

    >**Note**: Ignore any certificate errors. This is expected. 

1.  In the web browser window displaying the Puppet Master Console sign-in page, sign in by using **admin** as the username and **Pa55w.rd1234** as its password. This will display the Puppet Configuration Management Console.

    >**Note**: You cannot log into the Puppet Master Console with the username you specified during deployment, but you have to use the built-in admin user account instead.

#### Task 2: Install the Puppet agent on the Node Azure VM

In this task, you will add the second Azure VM as a node managed by the Puppet Master. 

1.  In the web browser displaying the Puppet Configuration Management Console, in the vertical menu on the left side, click **Nodes**, then click **Unsigned Certificates** and record the command that allows you to add nodes to be managed by with Puppet Enterprise. 

    >**Note**: You will need this command later in this task. The command will resemble the following format (where the `<Pm_Dns_Name_for_Public_IP>` placeholder designates the name you assigned to the IP address of the second Azure VM in the previous task):

    ```bash
    curl -k https://<Pm_Dns_Name_for_Public_IP>.0h03b1jj0ewetnu40a35rbm0qg.bx.internal.cloudapp.net:8140/packages/current/install.bash | sudo bash
    ``` 

1.  On the lab computer, in the web browser window, navigate to [the PuTTY download page](https://putty.org/), download the PuTTY installer, and run the installation with the default settings.
1.  Following the installation, in the **Start** menu, expand the **PuTTY (64-bit)** folder and click **PuTTY** icon to open the **PuTTY Configuration** window.
1.  In the **PuTTY Configuration** window, in the **Host Name (or IP address)** textbox, type the DNS name of the Azure VM that will host the Puppet node you identified at the end of the previous task and click **Open**.
1.  When prompted, in the **PuTTY Security Alert** window, click **Yes**.
1.  When prompted, in the **PuTTY** session to the Puppet node, sign in as **azureuser** with the **Pa55w.rd1234** password.
1.  Once signed in, in the PuTTY session to the Puppet node, run the command you recorded earlier in this task

    >**Note**: Wait for the command to install the Puppet agent and any dependencies on the node. This might take about 2 minutes. From this point forward, you will configure the node exclusively by using the Puppet Master only.

    >**Note**: You can automate the Puppet agent installation and configuration on Azure VMs by using the Puppet Agent extension from Azure Marketplace.

    >**Note**: Next, to manage the newly installed node, you need to accept the pending request from Puppet Configuration Management Console.

1.  Switch back to the web browser window displaying the Puppet Configuration Management Console, refresh the **Unsigned Certificates** page, verify that the page displays a pending unsigned certificate request, and click **Accept** to approve the request to bring the node into the inventory.

    >**Note**: This is a request to authorize the certificate that will secure communication between the Puppet Master and the node.

1.  In the web browser window displaying the Puppet Configuration Management Console, click **Nodes** and verify that you see two entries representing, respectively, the Puppet Master and its node.

    >**Note**: The Parts Unlimited MRP application (PU MRP App) is a Java application. The PU MRP App requires you to install and configure MongoDB and Apache Tomcat on the Azure VM configured as the Puppet node. However, rather than installing and configuring MongoDB and Tomcat manually, we will write a Puppet Program that will automatically configure the node. Puppet Programs are stored in a particular directory on the Puppet Master. Puppet Programs are made up of manifests that describe the desired state of the target node or nodes. The manifests can consume modules, which are pre-packaged Puppet Programs. Users can create their own modules or consume modules from a marketplace that is maintained by Puppet Labs, known as the Forge. Some modules on The Forge are supported officially, while others are open-source modules uploaded from the community. Puppet Programs are organized by environment, which allows you to manage Puppet Programs for different environments such as Dev, Test and Production.

    >**Note**: In this lab, we will use the newly onboarded node to emulate the production environment. We will also download modules from the Forge, which we will consume in order to configure the node.

#### Task 3: Configure the Puppet Production environment

In this task, you will configure our emulated Puppet Production environment. 

>**Note**: The template we used to deploy Puppet to Azure also configured a directory on the Puppet Master for managing the Production environment. The corresponding directory is **/etc/puppetlabs/code/environments/production**.

>**Note**: You will start by inspecting the Production modules.

1.  On the lab computer, in the **Start** menu, expand the **PuTTY (64-bit)** folder and click **PuTTY** icon to open the **PuTTY Configuration** window.
1.  In the **PuTTY Configuration** window, in the **Host Name (or IP address)** textbox, type the DNS name of the Azure VM that hosts the Puppet Master and click **Open**.
1.  When prompted, in the **PuTTY Security Alert** window, click **Yes**.
1.  When prompted, in the **PuTTY** session to the Puppet Master, sign in as **azureuser** with the **Pa55w.rd1234** password.
1.  Once successfully authenticated, in the PuTTY session to the Puppet Master, run the following to change the current working directory to the Production Directory **/etc/puppetlabs/code/environments/production**:

    ```bash
    cd /etc/puppetlabs/code/environments/production
    ```

1.  From the PuTTY session to the Puppet Master, run the `ls` command to list the contents of the Production Directory. 

    >**Note**: You will see directories named **manifests** and **modules**. The **manifests** directory contains descriptions of configurations that we will apply to our sample node later in this lab. The **modules** directory contains any modules that are referenced by the manifests.

    >**Note**: Next, you will install additional Puppet modules from The Forge that are needed to configure our sample node. 

1.  Within the PuTTY session to the Puppet Master, run the following commands to install required modules:

    ```bash
    sudo puppet module install puppetlabs-mongodb
    sudo puppet module install puppetlabs-tomcat
    sudo puppet module install maestrodev-wget
    sudo puppet module install puppetlabs-accounts
    sudo puppet module install puppetlabs-java
    ```

    >**Note**: The mongodb and tomcat modules from The Forge are officially supported. The wget module is a user module, and is not officially supported. The accounts module provides Puppet with classes for managing and creating users and groups on Linux. Finally, the java module provides Puppet with additional Java functionality.

    >**Note**: Next, you will create a custom module named **mrpapp*** in the **modules** directory on the Puppet Master. The custom module will configure the PU MRP app. 

1.  From the PuTTY session to the Puppet Master, run the following command to change the current directory to **/etc/puppetlabs/code/environments/production/modules**:

    ```bash
    cd /etc/puppetlabs/code/environments/production/modules
    ```

1.  From the PuTTY session to the Puppet Master, run the following command to create the **mrpapp** module:

    ```bash
    sudo puppet module generate partsunlimited-mrpapp
    ```

    >**Note**: This will start a wizard that asks a series of questions as it scaffolds the module. 

1.  To complete the creation of the **mrapp** module, press the **Enter** key for each question (accepting blank or default) until the wizard completes.

    >**Note**: You can use the `ls -la` command to verify the new module was created. The `ls -la` command lists the contents of a directory using a long list format (`-l`), including hidden files (`-a`).

    >**Note**: The mrpapp module will define our node's configuration. The configuration of nodes in the Production environment is defined in the **site.pp** file. The **site.pp** file is located in the **manifests** directory. The **.pp** filename extension is the acronym of **Puppet Program**. You will edit the **site.pp** file by adding a configuration for our node.

1.  From the PuTTY session to the Puppet Master, run the following command to open the **site.pp** in the Nano text editor:

    ```bash
    sudo nano /etc/puppetlabs/code/environments/production/manifests/site.pp
    ```

1.  Within the Nano editor interface, scroll to the bottom of the file and replace the existing `node default` section with the following one:

    ```bash
    node default {
       class { 'mrpapp': }
    }
    ```

1.  Within the Nano editor interface, press **ctrl + o** key combination, press the **Enter** key, and then press **ctrl + x** key combination to save the changes you made and close the file.

    >**Note**: The edits that you made to the **site.pp** file instruct Puppet to configure all nodes with the **mrpapp** module, by default. The **mrpapp** module (though currently empty) is in the **modules** directory of the Production environment, so Puppet will know where to find the module.

#### Task 4: Test the configuration of the Production environment 

In this task, you will test the configuration of the Production environment 

>**Note**: Before we fully describe the PU MRP app for the sample node, we will test the configuration by setting up a dummy file in the **mrpapp** module. If Puppet executes and creates the dummy file successfully, we will proceed with the full configuration of the **mrpapp** module.

>**Note**: You will start by editing the **init.pp** file, which is the entry point for the **mrpapp** module. This file resides in the **mrpapp/manifests** directory on the Puppet Master. 

1.  From the PuTTY session to the Puppet Master, run the following command to open the **init.pp** file in the Nano text editor:

    ```bash
    sudo nano /etc/puppetlabs/code/environments/production/modules/mrpapp/manifests/init.pp
    ```

1.  Within the Nano editor interface, scroll to the `class: mrpapp` declaration and modify it such that it looks as follows:

    ```puppet
    class mrpapp {
       file { '/tmp/dummy.txt':
          ensure => 'present',
          content => 'Puppet rules!'
       }
    }
    ```

    >**Note**: Make sure that you remove the comment tag (`#`) before the Class definition, as instructed.

1.  Within the Nano editor interface, press **ctrl + o** key combination, press the **Enter** key, and then press **ctrl + x** key combination to save the changes you made and close the file.

    >**Note**: Classes in Puppet Programs are not equivalent to classes in Object Oriented Programming. Classes in Puppet Programs define a resource that is configured on a node. By adding the `mrpapp` class or resource to the **init.pp** file, we have instructed Puppet to ensure that a file exists on the path /tmp/dummy.txt, and that the file has the content "Puppet rules!". We will define more advanced resources within the `mrpapp` class as we progress through this lab.

    >**Note**: Now it's time to perform the newly set up test. 

1.  Switch to the PuTTY session to the Puppet node and run the following command to test the new configuration:

    ```bash
    sudo puppet agent --test --debug
    ```

    >**Note**: By default, Puppet agents query the Puppet Master for their configuration every 30 minutes. The Puppet Agent then tests its current configuration against the configuration specified by the Puppet Master. If necessary, the Puppet Agent modifies its configuration to match the configuration specified by the Puppet Master. The command you invoked triggers immediate configuration check by the local Puppet agent against the Puppet Master. In this case, the configuration requires presence of the **/tmp/dummy.txt** file, so the node creates the file accordingly.

1.  Within the PuTTY session to the Puppet node, run the following command to verify the presence of the **/tmp/dummy.txt** file and to list the file's contents:

    ```bash
    cat /tmp/dummy.txt
    ```

1.  Verify that the "Puppet rules!" message appears as the output of the command. 

    >**Note**: Next, you will correct configuration drift. Each time the Puppet agent runs, Puppet determines if the environment is in the correct state. If it is not in the correct state, Puppet reapplies the relevant classes as necessary. This process allows Puppet to detect configuration drift and fix it. You will simulate configuration drift by deleting the dummy file **dummy.txt** from the node. 

1.  Within the PuTTY session to the Puppet node, run the following to delete the **dummy.txt** file:

    ```bash
    sudo rm /tmp/dummy.txt
    ```

1.  Within the PuTTY session to the Puppet node, run the following command to trigger the execution of the Puppet agent:

    ```bash
    sudo puppet agent --test
    ```

1.  Within the PuTTY session to the Puppet node, run the following command to verify the presence of the **/tmp/dummy.txt** file and to list the file's contents:

    ```bash
    cat /tmp/dummy.txt
    ```

#### Task 5: Create a Puppet program to describe the prerequisites of the PU MRP app

In this task, you will create a Puppet program to describe the prerequisites of the PU MRP app.

>**Note**: We have hooked the node to the Puppet Master. Now we can write the Puppet program to describe the prerequisites of the PU MRP app. In practice, the different parts of a large configuration solution are typically split into multiple manifests or modules. Splitting the configuration across multiple files is a form of modularization and promotes code reuse. For simplicity, in this lab, we will describe our entire configuration in a single Puppet program file **init.pp**, within the mrpapp module that we created earlier. In this task, we will step through the process of building our **init.pp** file.

>**Note**: You can find the completed **init.pp** file in [the Microsoft's PU MRP GitHub repository](https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/M04/Puppet/final/init.pp). You can either copy the **init.pp** file contents from GitHub or step through its edit in this task.

>**Note**: Tomcat requires the presence of a Linux user and group on the node to access and run Tomcat services. The default name that Tomcat uses for both is `Tomcat`. There are several ways to implement such configuration, but we will use for this purpose a separate class in the **init.pp** file. In addition to editing the **init.pp** file, we will make the following modifications.

- Edit the file **war.pp** to simplify deployment of the PU MRP app.
- Edit permissions on the directory **~/tomcat7/webapps** to enable extracting **.war** files at the end of this task. Editing these permissions will allow the **.war** file to be extracted automatically, when the Tomcat service is restarted as the **.pp** file runs.

>**Note**: All of the actions required to create our Puppet program will be explained as we step through the individual sections of this task. 

##### Section 5.1: Configure MongoDB

>**Note**: Once MongoDB is configured, we want Puppet to download a Mongo script that contains data for our PU MRP application's database. We will include this as part of our MongoDB setup. To implement these requirements, add a Class to configure MongoDB.

1.  Within the PuTTY session to the Puppet Master, run the following commands to open the **init.pp** file inside the **mrpapp module** in the Nano text editor:

    ```bash
    sudo nano /etc/puppetlabs/code/environments/production/modules/mrpapp/manifests/init.pp
    ```

1.  Within the Nano editor interface, scroll to the bottom of the file and append the `configuremongodb`, starting with a new line:

    ```puppet
    class configuremongodb {
      include wget
      class { 'mongodb': }->

      wget::fetch { 'mongorecords':
        source => 'https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/deploy/MongoRecords.js',
        destination => '/tmp/MongoRecords.js',
        timeout => 0,
      }->
      exec { 'insertrecords':
        command => 'mongo ordering /tmp/MongoRecords.js',
        path => '/usr/bin:/usr/sbin',
        unless => 'test -f /tmp/initcomplete'
      }->
      file { '/tmp/initcomplete':
        ensure => 'present',
      }
    }
    ```

1.  Within the Nano editor interface, press **ctrl + o** key combination and then press the **Enter** key to save the changes you made but keep the file open. 

    >**Note**: Let's examine the `configuremongodb` class.

    Line 1. We create a `class (resource)` called `configuremongodb`.
    Line 2. We include the `wget` module so that we can download files via `wget`.
    Line 3. We invoke the `mongodb resource` (from the `mongodb` module we downloaded earlier). This installs Mongo DB using defaults defined in the Puppet Mongo DB module. That is all we have to do to install Mondo DB.
    Line 5. We invoke the `fetch` resource from the `wget` module and call the resource `mongorecords`.
    Line 6. We set the source of the file we need to download from PU MRP GitHub.
    Line 7. We set the destination where the file must be downloaded to.
    Line 10. We use the built-in Puppet resource `exec` to execute a command.
    Line 11. We specify the command to execute.
    Line 12. We set the path for the command invocation.
    Line 13. We specify a condition using the keyword `unless`. We only want to execute this command once, so we create a `tmp` file once we have inserted the records (Line 15). If this file exists, we do not execute the command again.

    >**Note**: The `->` notation on Lines 3, 9 and 14 is an ordering arrow. The ordering arrow indicates to Puppet to apply the left resource before invoking the right resource. This allows us to control order of execution.

##### Section 5.2: Configure Java

1.  Within the Nano editor interface, append to the end of the **init.pp** file the following `configurejava` class definition, directly below the `configuremongodb` class definition, in order to set up Java requirements:

    ```puppet
    class configurejava {
      include apt
      $packages = ['openjdk-8-jdk', 'openjdk-8-jre']

      apt::ppa { 'ppa:openjdk-r/ppa': }->
      package { $packages:
         ensure => 'installed',
      }
    }
    ```

1.  Within the Nano editor interface, press **ctrl + o** key combination and then press the **Enter** key to save the changes you made but keep the file open. 

    >**Note**: Let's examine the `configurejava` class.

    Line 2. We include the `apt` module, which will allow us to configure new **Personal Package Archives (PPAs)**.
    Line 3. We create an array of packages that we need to install.
    Line 5. We add a **PPA**.
    Lines 6 to 8. We instruct Puppet to ensure that the packages are installed. Puppet expands the array and installs each package in the array.

    >**Note**: We cannot use the Puppet package target to install Java, because this will only install Java 7. We add the PPA and use the `apt` module to install Java 8.

##### Section 5.3: Create User and Group

1.  Within the Nano editor interface, append to the end of the **init.pp** file the following `createuserandgroup` class, to specify the Linux users and groups to be used for configuring, deploying, starting and stopping services.

    ```puppet
    class createuserandgroup {

    group { 'tomcat':
      ensure => 'present',
      gid    => '10003',
     }

    user { 'tomcat':
      ensure           => 'present',
      gid              => '10003',
      home             => '/tomcat',
      password         => '!',
      password_max_age => '99999',
      password_min_age => '0',
      uid              => '1003',
     }
    }
    ```

1.  Within the Nano editor interface, press **ctrl + o** key combination and then press the **Enter** key to save the changes you made but keep the file open. 

    >**Note**: Let's examine the `createuserandgroup` class.

    Line 1. We call the `createuserandgroup` class that we defined at the beginning of the **init.pp** file. We defined this Class to use the `user` and `group` resources.
    Lines 3 to 6. We use the `group` command to create the group `Tomcat`. We use the parameter value `ensure = > present` to verify that the `Tomcat` group exists and create it if needed. We then assign a `gid` value to the group. This assignment is not mandatory, but it helps manage groups across multiple machines.
    Lines 8 to 17. We use the `user` command to create the user `Tomcat`. We use the `ensure` command to verify that the `Tomcat` user exists and create it if needed. We assign `gid` and `uid` values to help manage users. Then we set the `Tomcat` user's home directory path and assign the user's password settings. We have left the `password` value unspecified for ease of use. In practice you should implement strict password policies.

    >**Note**: The commands `user` and `group` are made available via the `puppetlabs-accounts` module, which we installed on the Puppet Master earlier in the lab.

##### Section 5.4: Configure Tomcat

1.  Within the Nano editor interface, append to the end of the **init.pp** file the following `configuretomcat` class to configure Tomcat:

    ```puppet
    class configuretomcat {
      class { 'tomcat': }
      require createuserandgroup

     tomcat::instance { 'default':
      catalina_home => '/var/lib/tomcat7',
      install_from_source => false,
      package_name => ['tomcat7','tomcat7-admin'],
     }->

     tomcat::config::server::tomcat_users {
     'tomcat':
       catalina_base => '/var/lib/tomcat7',
       element  => 'user',
       password => 'password',
       roles => ['manager-gui','manager-jmx','manager-script','manager-status'];
     'tomcat7':
       catalina_base => '/var/lib/tomcat7',
       element  => 'user',
       password => 'password',
       roles => ['manager-gui','manager-jmx','manager-script','manager-status'];
     }->

     tomcat::config::server::connector { 'tomcat7-http':
      catalina_base => '/var/lib/tomcat7',
      port => '9080',
      protocol => 'HTTP/1.1',
      connector_ensure => 'present',
      server_config => '/etc/tomcat7/server.xml',
     }->

     tomcat::service { 'default':
      use_jsvc => false,
      use_init => true,
      service_name => 'tomcat7',
     }
    }
    ```

1.  Within the Nano editor interface, press **ctrl + o** key combination and then press the **Enter** key to save the changes you made but keep the file open. 

    >**Note**: Let's examine the `configuretomcat` class.

    Line 1. We create a `class (resource)` called `configuretomcat`.
    Line 2. We invoke the `tomcat` resource from the Tomcat module we downloaded earlier.
    Line 3. We require the `Tomcat` user and group to be available before we can configure Tomcat, so we mark the class `createuserandgroup` as required.
    Lines 6 to 9. We install the `Tomcat` package from the Tomcat module we downloaded earlier. The package name is `Tomcat7`, so we call it here. We designate the home directory where we will place our default installation. Then, we specify that it is not from source. This is required when installing from a downloaded package. We could install directly from a web source, if we wished. We also install the `tomcat7-admin` package. We will not use the `tomcat7-admin` package in this lab, but it can be used to provide the Tomcat Manager Web User Interface.
    Lines 13 to 23. We create two usernames for Tomcat to use. These are created within the Tomcat application for managing and configuring Tomcat. They are created within, and called from, the file **/var/lib/tomcat7/conf/tomcat-users.xml**. 
    Lines 26 to 31. We configure the Tomcat connector, define the protocol to use and the port number. We also ensure that the connector is present, to allow it to serve requests on the ports that we set. We also specify the connector properties for Puppet to write to the Tomcat **server.xml** file. Again, you can open the file, and view its contents, on the Puppet Master at **/var/lib/tomcat7/conf/server.xml**.
    Lines 35 to 38. We configure the Tomcat service. As there is only one installation of Tomcat, we specify the `default` as the configuration target. We ensure the service is running by setting the `use_init` value. Then, we specify the service connector name for the PU MRP app as `tomcat7`.

##### Section 5.5: Deploy a WAR File

>**Note**: The PU MRP app is compiled into a .war file that Tomcat uses to serve pages. We need to specify a resource to deploy a .war file for the PU MRP app site.

1.  Within the Nano editor interface, append to the end of the **init.pp** file the following `deploywar` class to the bottom of the file.

    ```puppet
    class deploywar {
      require configuretomcat

      tomcat::war { 'mrp.war':
        catalina_base => '/var/lib/tomcat7',
        war_source => 'https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/builds/mrp.war',
      }

     file { '/var/lib/tomcat7/webapps/':
       path => '/var/lib/tomcat7/webapps/',
       ensure => 'directory',
       recurse => 'true',
       mode => '777',
     }
    }
    ```

1.  Within the Nano editor interface, press **ctrl + o** key combination and then press the **Enter** key to save the changes you made but keep the file open. 

    >**Note**: Let's examine the deploywar Class.

    Line 1. We create a `class (resource)` called `deploywar`.
    Line 2. We instruct Puppet to ensure that `configuretomcat` has completed, before invoking the `deploywar` class.
    Line 5. We set the `catalina_base` directory so that Puppet deploys the .war to our Tomcat service.
    Line 6. We use the Tomcat module's `war` resource to deploy our .war from `war_source`.
    Line 9. We call the `file` class, defined in the Tomcat module. The `file` class allows us to configure files on our system.
    Line 10. We specify the path to the web directory for our PU MRP app.
    Line 11. We ensure a directory exists on the path we specified, before applying our configuration.
    Line 12. We specify that the configuration should be applied recursively, so it will affect all files and sub-directories within the `~/tomcat7/webapps/` directory.
    Line 13. We designate the `777` permission on all files and sub-directories within the `~/tomcat7/webapps/` directory. It is very important to set these permissions to allow read, write and execute permissions on all content within the target directory. Setting `777` permissions is acceptable in this lab. You should always set more restrictive permissions according to each user, group and access level in a production environment.

    >**Note**: Typically, the following actions are needed to deploy a .war file

    1. Copy the .war file to the Tomcat **/var/lib/tomcat7/webapps** directory
    2. Set the correct permissions on the **webapps** directory
    3. Specify values for `unpackWARS` and `autoDeploy` in the Tomcat server config file at **/var/lib/tomcat7/conf/server.xml**
    4. List a valid and available user in the Tomcat file **/var/lib/tomcat7/conf/tomcat-users.xml**

    >**Note**: Tomcat will extract and then deploy the .war when the service starts. This is why we restart the service in the **init.pp** file. When you complete this lab, you can open and view the contents of the Tomcat files **server.xml** and **tomcat-users.xml**.

##### Section 5.6: Start the Ordering Service

>**Note**: The PU MRP app calls an Ordering Service, which is a REST API that manages orders in the Mongo DB. The Ordering Service is compiled to a .jar file. We need to copy the .jar file to our node. The node then runs the Ordering Service, in the background, so that it can listen for REST API requests.

1.  Within the Nano editor interface, append to the end of the **init.pp** file the following 'orderingservice` class to ensure that the Ordering Service runs:

    ```puppet
    class orderingservice {
      package { 'openjdk-7-jre':
        ensure => 'installed',
      }

      file { '/opt/mrp':
        ensure => 'directory'
      }->
      wget::fetch { 'orderingsvc':
        source => 'https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/builds/ordering-service-0.1.0.jar',
        destination => '/opt/mrp/ordering-service.jar',
        cache_dir => '/var/cache/wget',
        timeout => 0,
      }->
  
      exec { 'stoporderingservice':
        command => "pkill -f ordering-service",
        path => '/bin:/usr/bin:/usr/sbin',
        onlyif => "pgrep -f ordering-service"
      }->

      exec { 'stoptomcat':
        command => 'service tomcat7 stop',
        path => '/bin:/usr/bin:/usr/sbin',
        onlyif => "test -f /etc/init.d/tomcat7",
      }->
      exec { 'orderservice':
        command => 'java -jar /opt/mrp/ordering-service.jar &',
        path => '/usr/bin:/usr/sbin:/usr/lib/jvm/java-8-openjdk-amd64/bin',
      }->
      exec { 'wait':
        command => 'sleep 20',
        path => '/bin',
        notify => Tomcat::Service['default']
      }
    }
    ```

1.  Within the Nano editor interface, press **ctrl + o** key combination and then press the **Enter** key to save the changes you made but keep the file open. 

    >**Note**: Let's examine the orderingservice Class.

    Line 1. We create a `class (resource)` called `orderingservice`.
    Lines 2 to 4. We install the Java Runtime Environment (JRE) required to run the application using Puppet's package resource.
    Lines 6 to 8. We verify that the directory `/opt/mrp` exists and create it if necessary.
    Lines 9 to 14. We use `wget` to retrieve the Ordering Service binary, and place it in `/opt/mrp`.
    Line 12. We specify a cache directory, to ensure that the file is only downloaded once.
    Lines 15 to 19. We stop the orderingservice if it is running.
    Lines 20 to 24. We stop the tomcat7 service if it is running.
    Lines 25 to 28. We start the Ordering Service.
    Lines 29 to 33. We wait for 20 seconds to give the Ordering Service time to start up, before notifying the tomcat service. These actions trigger a refresh on the service. Puppet will re-apply the state that we defined for the service (i.e. start it, if it is not running already).

    >**Note**: We need to wait after running the java command, since this service needs to be running before we can start Tomcat. Otherwise, Tomcat will occupy the port that the Ordering Service needs to listen on for incoming API requests.

##### Section 5.7: Complete the mrpapp Resource

1.  Within the Nano editor interface, scroll up to the top of the **init.pp** file and remove the `mrpapp` class definition by deleting the content that relates to the **dummy.txt** file which we tested earlier in the lab. 
1.  Within the Nano editor interface, add the updated `mrpapp` class definition:

    ```puppet
    class mrpapp {
      class { 'configuremongodb': }
      class { 'configurejava': }
      class { 'createuserandgroup': }
      class { 'configuretomcat': }
      class { 'deploywar': }
      class { 'orderingservice': }
    }
    ```

    >**Note**: The modifications to the `mrpapp` class designate which classes are needed in the **init.pp** file, in order to run our resources.

1.  Within the Nano editor interface, press **ctrl + o** key combination, press the **Enter** key, and then press **ctrl + x** key combination to save the changes you made and close the file.

    >**Note**: After you have made all of the necessary modifications to the **init.pp** file, it should be identical to the **init.pp** file on [the Microsoft's PU MRP GitHub repository](https://raw.githubusercontent.com/Microsoft/PartsUnlimitedMRP/master/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/M04/Puppet/final/init.pp). If you prefer, you can copy the content of the **init.pp** file from GitHub and paste it into your local **init.pp** file.

##### Section 5.8: Configure .war file extraction permissions

>**Note**: Part of the configuration we defined in the **init.pp** file involves copying the .war file for our PU MRP app to the directory **/var/lib/tomcat7/webapps**. Tomcat automatically extracts .war files from the webapps directory. However, in our lab, we will set full read, write and access permissions in the **war.pp** file to ensure that the file extracts successfully.

1.  Within the PuTTY session to the Puppet Master, run the following commands to open the **war.pp** file in the Nano editor:

    ```bash
    sudo nano /etc/puppetlabs/code/environments/production/modules/tomcat/manifests/war.pp
    ```

1.  Within the Nano editor interface, scroll to the bottom of the file and replace the existing `mode => '0640'` entry with `mode => '0777'` to modify the intended permissions. This should result in the following content:

    ```puppet
        file { "tomcat::war ${name}":
          ensure    => file,
          path      => "${_deployment_path}/${_war_name}",
          owner     => $user,
          group     => $group,
          mode      => '0777',
          subscribe => Archive["tomcat::war ${name}"],
        }
      }
    }
    ```

1.  Within the Nano editor interface, press **ctrl + o** key combination, press the **Enter** key, and then press **ctrl + x** key combination to save the changes you made and close the file.

    >**Note**: Let's examine this section of the war.pp file.

    Line 1. Specifies the file name, which comes with the Tomcat module `tomcat::war`.
    Line 2. Ensures the file is present before taking an action.
    Line 3. Defines the path to the web directory, which we previously set to `/var/lib/tomcat7/webapps`. This is where the .war file for our PU MRP app will be copied to, as specified in the **init.pp** file.
    Line 4 to 5. Set the user and group owners of the files.
    Line 6. Set the file permissions mode to `0777`, which grants read, write and execute permissions to everyone. You should not use `777` permissions in a Production environment.

#### Task 6: Run the Puppet configuration on the Puppet node

In this task, you will trigger the configuration update on the Puppet node and verify that this results in a successful deployment of the PU MRP app.

1.  Switch to the PuTTY session to the Puppet node and re-run the following command to test the new configuration:

    ```bash
    sudo puppet agent --test
    ```

    >**Note**: This first run may take a few minutes since it is necessary to download and install all required components. During subsequent runs, the Puppet agent will only verify that the existing environment is configured correctly. 

    >**Note**: Disregard error messages regarding `openjdk-7-jre`. That's expected. OpenJDK 6 and 7 are not available in Ubuntu 16.04.

1.  To verify that Tomcat is running correctly, start a web browser and navigate to the URL that consists of the `http:\\` prefix, followed by the DNS name of the Azure VM that hosts the Puppet node, followed by ':9080'. Verify that the browser displays the Tomcat landing page.
1.  To verify that the PU MRP app is running correctly, within the same browser tab, append to the URL the `/mrp` suffix. Verify that the browser displays the PU MRP app Welcome page.

### Exercise 3: Remove the Azure lab resources

In this exercise, you will remove the Azure resources provisione in this lab to eliminate unexpected charges. 

>**Note**: Remember to remove any newly created Azure resources that you no longer use. Removing unused resources ensures you will not see unexpected charges.

#### Task 1: Remove the Azure lab resources

In this task, you will use Azure Cloud Shell to remove the Azure resources provisione in this lab to eliminate unnecessary charges. 

1.  In the Azure portal, open the **Bash** shell session within the **Cloud Shell** pane.
1.  List all resource groups created throughout the labs of this module by running the following command:

    ```sh
    az group list --query "[?starts_with(name,'az400m14l03a-RG')].name" --output tsv
    ```

1.  Delete all resource groups you created throughout the labs of this module by running the following command:

    ```sh
    az group list --query "[?starts_with(name,'az400m14l03a-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Note**: The command executes asynchronously (as determined by the --nowait parameter), so while you will be able to run another Azure CLI command immediately afterwards within the same Bash session, it will take a few minutes before the resource groups are actually removed.

## Review

In this lab, you learned how to deploy the PartsUnlimted MRP application (PU MRP App) to an Azure VM by using Puppet from Puppet Labs. 
