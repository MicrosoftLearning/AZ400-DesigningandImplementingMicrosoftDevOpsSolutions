---
lab:
    title: 'Lab: Ansible with Azure'
    module: 'Module 14: Third party Infrastructure as Code Tools Available with Azure'
---

# Lab: Ansible with Azure
# Student lab manual

## Lab overview

In this lab we will deploy, configure, and manage Azure resources by using Ansible. 

Ansible is declarative configuration management software. It relies on a description of the intended configuration applicable to managed computers in the form of playbooks. Ansible automatically applies that configuration and maintains it going forward, addressing any potential discrepancies. Playbooks are formatted by using YAML.

Unlike majority of other configuration management tools, such as Puppet or Chef, Ansible is agentless, which means that it does not require the installation of any software in the managed machines. Ansible uses SSH to manage Linux servers and Powershell Remoting to manage Windows servers and clients.

In order to interact with resources other than operating systems (such as, for example, Azure resources accessible via Azure Resource Manager), Ansible supports extensions called modules. Ansible is written in Python so, effectively, the modules are implemented as Python libraries. In order to manage Azure resources, Ansible relies on [GitHub-hosted modules](https://github.com/ansible-collections/azure).

Ansible requires that the managed resources are documented in a host inventory. Ansible supports dynamic inventories for some systems, including Azure, so that the host inventory is dynamically generated at runtime.

The lab will consist of the following high-level steps:

- Deploying two Azure VMs by using Azure Cloud Shell
- Generating a dynamic inventory
- Creating an Ansible Control VM in Azure by using Azure CLI
- Installing and configuring Ansible on Azure VM
- Downloading up Ansible configuration and sample playbook files
- Creating and configuring a service principal in Azure AD
- Configuring Azure AD credentials and SSH for use with Ansible
- Deploying an Azure VM using an Ansible playbook
- Configuring an Azure VM by using an Ansible playbook
- Using Ansible playbooks for configuration and desired state management on Azure VMs
- Using Ansible with Azure Resource Manager templates to facilitate configuration management and desired state of Azure resources

## Objectives

After you complete this lab, you will be able to:

- Deploy Azure VMs by using Azure Cloud Shell
- Generate an Ansible dynamic inventory
- Create an Ansible Control VM in Azure by using Azure CLI
- Install and configure Ansible on Azure VM
- Download Ansible configuration and sample playbook files
- Create and configure a service principal in Azure AD
- Configure Azure AD credentials and SSH for use with Ansible
- Deploy an Azure VM using an Ansible playbook
- Configure an Azure VM by using an Ansible playbook
- Use Ansible playbooks for configuration and desired state management on Azure VMs
- Use Ansible with Azure Resource Manager templates to facilitate configuration management and desired state of Azure resources

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

### Exercise 1: Deploy, configure, and manage Azure VMs by using Ansible

In this exercise, you will deploy, configure, and manage Azure VMs by using Ansible.

#### Task 1: Create two Azure VMs by using Azure Cloud Shell

In this task, you will create two Azure VMs by using Azure Cloud Shell.

1.  From the lab computer, start a web browser, navigate to the [**Azure Portal**](https://portal.azure.com), and sign in with the user account that has at least the Contributor role in the Azure subscription you are using in this lab.
1.  In the Azure portal, in the toolbar, click the **Cloud Shell** icon located directly to the right of the search text box. 

    >**Note**: Alternatively, you can access Cloud Shell directly by navigating to [https://shell.azure.com](https://shell.azure.com).

1.  If prompted to select either **Bash** or **PowerShell**, select **Bash**. 

    >**Note**: If this is the first time you are starting **Cloud Shell** and you are presented with the **You have no storage mounted** message, select the subscription you are using in this lab, and select **Create storage**. 

1.  From the Bash session in the Cloud Shell pane, run the following to create a resource group that will host the Azure VMs (replace the `<Azure_region` placeholder with the name of the Azure region where you intend to deploy resources in this lab):

    ```bash
    LOCATION=<Azure_region>
    RGNAME=az400m14l03arg
    az group create --name $RGNAME --location $LOCATION
    ```

1.  From the Bash session in the Cloud Shell pane, run the following to deploy the first Azure VM running Ubuntu into the resource group you created in the previous step:

    ```bash
    VM1NAME=az400m1403vm1
    az vm create --resource-group $RGNAME --name $VM1NAME --image UbuntuLTS --generate-ssh-keys --no-wait
    ```

    >**Note**: Do not wait for the deployment to complete but proceed to the next step. 

1.  From the Bash session in the Cloud Shell pane, run the following to deploy the second Azure VM running Ubuntu into the same resource group:

    ```bash
    VM2NAME=az400m1403vm2
    az vm create --resource-group $RGNAME --name $VM2NAME --image UbuntuLTS --generate-ssh-keys
    ```

    >**Note**: Wait for the deployment to complete before you proceed to the next step. This might take about 2 minutes.

1.  From the Bash session in the Cloud Shell pane, run the following to tag the first Azure VM with the tag Nginx to identify it as an Nginx web server

    ```bash
    VM1ID=$(az vm show --resource-group $RGNAME --name $VM1NAME --query id --output tsv)
    az resource tag --tags Ansible=nginx --id $VM1ID
    ```

#### Task 2: Generate a dynamic inventory

In this task, you will generate a dynamic Ansible inventory targeting the two Azure VMs you deployed in the previous task.

>**Note**: Starting with Ansible 2.8, Ansible provides an Azure dynamic-inventory plug-in.

1.  From the Bash session in the Cloud Shell pane, run the following to create a new file named **myazure_rm.yml** and open it in the Nano text editor:

    ```bash
    nano ./myazure_rm.yml
    ```

1.  Within the Nano editor interface, paste the following content:

    ```bash
    plugin: azure_rm
    include_vm_resource_groups:
    - az400m14l03arg
    auth_source: auto

    keyed_groups:
    - prefix: tag
      key: tags
    ```

1.  Within the Nano editor interface, press **ctrl + o** key combination, press the **Enter** key, and then press **ctrl + x** key combination to save the changes you made and close the file.
1.  From the Bash session in the Cloud Shell pane, run the following to install Ansible Azure modules:

    ```bash
    curl -O https://raw.githubusercontent.com/ansible-collections/azure/dev/requirements-azure.txt
    pip install -r requirements-azure.txt
    rm requirements-azure.txt
    ansible-galaxy collection install azure.azcollection
    ```

    >**Note**: Starting with version 2.10 of Ansible, Azure modules are maintained separately from the core modules. To verify the Ansible version, run `ansible --version`.

1.  In the Bash session in the Cloud Shell pane, type `exit` and press the **Enter** key to exit the session. 

    >**Note**: This is necessary for the installation of modules to take effect. 

1.  In the Azure portal, in the toolbar, click the **Cloud Shell** icon to restart the Cloud Shell session. 
1.  From the Bash session in the Cloud Shell pane, run the following to perform a ping test and generate a dynamic inventory of all the virtual machines running in Azure in the resource group which name you included in the **myazure_rm.yml** file:

    ```bash
    ansible all -m ping -i ./myazure_rm.yml
    ```

    >**Note**: The first time you run the command you will have to acknowledge the authenticity of the target VMs, by typing **yes** and pressing the **Enter** key. You may have to do this for each target Azure VM. Once the authenticity is verified subsequent runs of the command should return successfully without the need for confirmation. If needed, run the above command a few times to generate the successful output.

    >**Note**: Disregard the deprecation warnings. 

1.  From the Bash session in the Cloud Shell pane, run the following to list hosts in the inventory:

    ```bash
    ansible all -i ./myazure_rm.yml --list-hosts
    ```

1.  From the Bash session in the Cloud Shell pane, run the following to list the tag-based groupings of the hosts in the inventory:

    ```bash
    ansible-inventory -i ./myazure_rm.yml --graph
    ```

1.  From the Bash session in the Cloud Shell pane, run the following to test connectivity to all hosts in the inventory that have a specific tag value:

    ```bash
    ansible -i ./myazure_rm.yml -m ping tag_Ansible_nginx
    ```

1.  From the Bash session in the Cloud Shell pane, run the following to list JSON-based representation of the properties of the first host (replace the `<Ansible_host_name>` with the name of the first host returned by the `ansible all -i ./myazure_rm.yml --list-hosts` command):

    ```bash
    ansible-inventory -i ./myazure_rm.yml --host <Ansible_host_name> | jq
    ```

1.  From the Bash session in the Cloud Shell pane, run the following to list JSON-based representation of the properties of all hosts in the inventory:

    ```bash
    ansible-inventory -i ./myazure_rm.yml --list | jq
    ```

1.  Close the Cloud Shell pane.

#### Task 3: Provision an Azure VM serving as the Ansible control machine

In this task, you will deploy an Azure VM by using Azure CLI and configure it to manage your Ansible environment.

>**Note**: Ansible static host inventory uses the **/etc/ansible/hosts** file. Since Azure Cloud Shell does not provide root access, limiting the storage options to the user's **$Home** directory, to implement Ansible management with non-dynamic inventory, we will deploy an Azure VM running Linux and configure it as an Ansible management system.

1.  In the Azure portal, in the toolbar, click the **Cloud Shell** icon located directly to the right of the search text box. 

    >**Note**: Alternatively, you can access Cloud Shell directly by navigating to [https://shell.azure.com](https://shell.azure.com).

1.  From the Bash session in the Cloud Shell pane, run the following to create a resource group that will host the new Azure VMs:

    ```bash
    RG1NAME=az400m14l03arg
    VM1NAME=az400m1403vm1
    LOCATION=$(az vm show --resource-group $RG1NAME --name $VM1NAME --query location --output tsv)
    RG2NAME=az400m14l03brg
    az group create --name $RG2NAME --location $LOCATION
    ```

1.  From the Bash session in the Cloud Shell pane, run the following to create a virtual network that will host the Azure VM that will serve as the Ansible management system:

    ```bash
    VNETNAME=az400m1403-vnet
    SUBNETNAME=ansible-subnet
    az network vnet create \
    --name $VNETNAME \
    --resource-group $RG2NAME \
    --location $LOCATION \
    --address-prefixes 192.168.0.0/16 \
    --subnet-name $SUBNETNAME \
    --subnet-prefix 192.168.1.0/24
    ```

1.  From the Bash session in the Cloud Shell pane, run the following to create a public IP address that you will assign to the network adapter of the Azure VM that will serve as the Ansible management system:

    ```bash
    PIPNAME=az400m1403-pip
    az network public-ip create \
    --name $PIPNAME \
    --resource-group $RG2NAME \
    --location $LOCATION
    ```
 
1.  From the Bash session in the Cloud Shell pane, run the following to create the Azure VM that will serve as the Ansible control machine:

    ```bash
    VM3NAME=az400m1403vm3
    az vm create \
    --name $VM3NAME \
    --resource-group $RG2NAME \
    --location $LOCATION \
    --image UbuntuLTS \
    --vnet-name $VNETNAME \
    --subnet $SUBNETNAME \
    --public-ip-address $PIPNAME \
    --authentication-type password \
    --admin-username azureuser \
    --admin-password Pa55w.rd1234
    ```

    >**Note**: Wait for the deployment to complete before you proceed to the next step. This might take about 2 minutes.

    >**Note**: Once the provisioning completes, in the JSON-based output, identify the value of the **"publicIpAddress"** property included in the output. 

1.  From the Bash session in the Cloud Shell pane, run the following to connect to the newly deployed Azure VM by using SSH:

    ```bash
    PIP=$(az vm show --show-details --resource-group $RG2NAME --name $VM3NAME --query publicIps --output tsv)
    ssh azureuser@$PIP
    ```

1.  When prompted for confirmation to proceed, type **yes** and press the **Enter** key and, when prompted to provide the password, type **Pa55w.rd1234**.

#### Task 4: Install and configure Ansible on an Azure VM

In this task, you will install and configure Ansible on the Azure VM you deployed in the previous task.

1.  In the Bash session in the Cloud Shell pane, within the SSH session to the newly deployed Azure VM, run the following to update the Advanced Packaging Tool (apt) package list to include the latest version and package details:

    ```bash
    sudo apt update
    ```

1.  In the Bash session in the Cloud Shell pane, within the SSH session to the newly deployed Azure VM, run the following to install the pre-requisites required for Ansible:

    ```bash
    sudo apt-get install -y libssl-dev libffi-dev python-dev python-pip
    ```

1.  In the Bash session in the Cloud Shell pane, within the SSH session to the newly deployed Azure VM, run the following to install Ansible and the required Azure modules:

    ```bash
    sudo apt-get install python3-pip
    sudo apt install ansible
    curl -O https://raw.githubusercontent.com/ansible-collections/azure/dev/requirements-azure.txt
    pip3 install -r requirements-azure.txt
    rm requirements-azure.txt
    ansible-galaxy collection install azure.azcollection
    ```

1.  In the Bash session in the Cloud Shell pane, within the SSH session to the newly deployed Azure VM, run the following to install the dnspython package to allow the Ansible playbooks to verify DNS names before deployment:

    ```bash
    sudo pip3 install dnspython 
    ```

1.  In the Bash session in the Cloud Shell pane, within the SSH session to the newly deployed Azure VM, run the following to install the **jq** JSON parsing tool:

    ```bash
    sudo apt install jq
    ```

1.  In the Bash session in the Cloud Shell pane, within the SSH session to the newly deployed Azure VM, run the following to install Azure CLI:

    ```sh
    curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
    ```

#### Task 5: Download Ansible configuration and sample playbook files

In this task, you will download from GitHub Ansible configuration repository along from the sample lab files. 

1.  In the Bash session in the Cloud Shell pane, within the SSH session to the newly deployed Azure VM, run the following to ensure that **git** is installed:

    ```bash
    sudo apt install git
    ```

1.  In the Bash session in the Cloud Shell pane, within the SSH session to the newly deployed Azure VM, run the following to clone the ansible source code from GitHub:

    ```bash
    git clone git://github.com/ansible/ansible.git --recursive
    ```

1.  In the Bash session in the Cloud Shell pane, within the SSH session to the newly deployed Azure VM, run the following to clone the PartsUnlimitedMRP repo from GitHub:

    ```bash
    git clone https://github.com/Microsoft/PartsUnlimitedMRP.git
    ```

    >**Note**: This repository contains playbooks for creating a wide range of resources, some of which we will use in the lab.

#### Task 6: Create and configure Azure Active Directory service principal

In this task, you will generate an Azure AD service principal in order to facilitate non-interactive authentication of Ansible, necessary to access Azure resources. You will also assign to the service principal the Contributor role on the resource group you created in the previous task.

1.  In the Bash session in the Cloud Shell pane, within the SSH session to the newly deployed Azure VM, run the following to sign in to the Azure AD tenant associated with your Azure subscription:

    ```bash
    az login
    ```

1.  Note the code displayed as the output of the previous command, switch to your lab computer, from your lab computer, open another tab in the browser window displaying the Azure portal, navigate to [the Microsoft Device Login page](https://microsoft.com/devicelogin) and, when prompted, enter the code and select **Next**.

1.  When prompted, sign in with credentials you are using in this lab and close the browser tab.

1.  Switch back to the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to generate a service principal:

    ```bash
    ANSIBLESP=$(az ad sp create-for-rbac --name az400m1403Ansible)
    ```

1.  In the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to display properties of the newly generated a service principal:

    ```bash
    echo $ANSIBLESP | jq
    ```

1.  Review the output and record the values of the **appId**, **password**, and **tenant** properties. The output should be in the following format:

    ```bash
    {
      "appId": "0df61458-1a6e-4b4c-b02a-d4ae43cd981e",
      "displayName": "az400m1403Ansible",
      "name": "http://az400m1403Ansible",
      "password": "JAHxIbRUypM~-DiA27Guj58E4nC.S_u~_U",
      "tenant": "44bb87a9-c9c8-4c0f-b176-88d7814533ba"
    }
    ```

1.  In the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to identify the value of your subscription by running:

    ```bash
    SUBSCRIPTIONID=$(az account show --query id --output tsv)
    echo $SUBSCRIPTIONID
    ```

1.  In the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to retrieve the value of the **ID** property of the built-in Azure Role Based Access Control Contributor role: 

    ```bash
    CONTRIBUTORID=$(az role definition list --name "Contributor" --query "[].id" --output tsv)
    echo $CONTRIBUTORID
    ```

1.  In the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to assign the Contributor role on the resource group you created earlier in this lab: 

    ```bash
    APPID=$(echo $ANSIBLESP | jq '.appId' -r)

    RG2NAME=az400m14l03brg
    az role assignment create --assignee "$APPID" \
    --role "$CONTRIBUTORID" \
    --resource-group "$RG2NAME"
    ```

#### Task 7: Configure Azure AD credentials and SSH for use with Ansible

In this task, you will configure Azure access for Ansible and SSH for use with Ansible, with the former leveraging the Azure AD service principal you created in the previous task.

>**Note**: To allow remote management from designated Ansible control system we can either create a credentials file or export the service principal details as Ansible environment variables. We will choose the first of these two options. The credentials will be stored in the **~/.azure/credentials** file. 

1.  In the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to create the credentials file: 

    ```bash
    echo "[default]" > ~/.azure/credentials
    ```

1.  In the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to append a new line containing the Azure subscription id you identified in the previous task to the Ansible credentials file: 

    ```bash
    echo subscription_id=$SUBSCRIPTIONID >> ~/.azure/credentials
    ```

1.  In the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to append a new line containing the service principal id you identified in the previous task to the Ansible credentials file: 

    ```bash
    echo client_id=$APPID >> ~/.azure/credentials
    ```
1.  In the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to append a new line containing the secret of the service principal id you identified in the previous task to the Ansible credentials file: 

    ```bash
    SECRET=$(echo $ANSIBLESP | jq '.password' -r)
    echo secret=$SECRET >> ~/.azure/credentials
    ```

1.  In the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to append a new line containing the id of the Azure AD tenant you identified in the previous task to the Ansible credentials file: 

    ```bash
    TENANT=$(echo $ANSIBLESP | jq '.tenant' -r)
    echo tenant=$TENANT >> ~/.azure/credentials
    ```

1.  In the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to verify the content of the Ansible credentials file: 

    ```bash
    cat  ~/.azure/credentials
    ```

    >**Note**: The output should resemble the following:

    ```bash
    [default]
    subscription_id=1cfb08bc-a39e-46cc-9b1e-38a182b43d60
    client_id=0df61458-1a6e-4b4c-b02a-d4ae43cd981e
    secret=JAHxIbRUypM~-DiA27Guj58E4nC.S_u~_U
    tenant=44bb87a9-c9c8-4c0f-b176-88d7814533ba
    ```

    >**Note**: Now you will create a public/private key pair for remote SSH connections and test their operations. 

1.  In the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to generate the key pair. When prompted, press the **Enter** key to accept the default values of the locations of the files and not to set the passphrase:

    ```bash
    ssh-keygen -t rsa
    ```

1.  In the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to grant read, write, and execute permissions on the **.ssh** folder hosting the private key:

    ```bash
    chmod 755 ~/.ssh
    ```

1.  In the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to create as well as set read and write permissions on the **authorized_keys** file.

    ```bash
    touch ~/.ssh/authorized_keys
    chmod 644 ~/.ssh/authorized_keys
    ```

    >**Note**: By providing keys included in this file, you are allowed access without having to provide a password.

1.  In the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to add the password to the **authorized_keys** file:

    ```bash
    ssh-copy-id azureuser@127.0.0.1
    ```

1. When prompted, type **yes** and enter the password **Pa55w.rd1234** for the **azureuser** user account you specified when deploying the third Azure VM earlier in this lab. 
1.  In the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to verify that you are not prompted for password:

    ```bash
    ssh 127.0.0.1
    ```
1.  In the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, type **exit** and press the **Enter** key to terminate the loopback connection you just established. 

>**Note**: Establishing passwordless SSH authentication is a critical step for setting up your Ansible environment. 

#### Task 8: Create a web server Azure VM by using an Ansible playbook

In this task, you will create an Azure VM hosting a web server by using an Ansible playbook. 

>**Note**: Now that we have Ansible up and running in the control Azure VM, we can deploy our first playbook in order to create and configure a managed Azure VM. The playbook will be using the localhost file, rather than a dynamic inventory. For the deployment, we will be using a sample playbook **~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_vm_web.yml** cloned from the Github repository earlier in this lab. Before deploying the sample playbook, you need to replace the public SSH key included in its content with the key you generated in the previous task. 

1.  In the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to identify the locally stored public key which you generated in the previous task:

    ```bash
    cat ~/.ssh/id_rsa.pub
    ```

1.  Record the output, including the username at the end of the output string. 
1.  In the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to open the **new_vm_web.yml** file in the Nano text editor:

    ```bash
    nano ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_vm_web.yml
    ```

1.  Within the Nano editor interface, locate the SSH string towards the end of the file, in the key_data value, and delete the key that is present in the file.:
1.  In the nano editor, locate the SSH string towards the end of the file, in the `key_data` entry, delete the existing key value and replace it with the key value that you recorded earlier in this task. 
1.  Make sure that the value of `admin_username` entry that is included in the file matches the user name you used to sign in to the Azure VM hosting the Ansible control system (**azureuser**). The same user name must be used in the `path` entry of `ssh_public_keys` section.
1.  Change the value of `vm_size` entry from `Standard_A0` to `Standard_D2s_v3`.
1.  If needed, change the name of the region in the `dnsname: '{{ vmname }}.westeurope.cloudapp.azure.com'` entry to the name of the Azure region you are targeting for deployment.
1.  Within the Nano editor interface, press **ctrl + o** key combination, press the **Enter** key, and then press **ctrl + x** key combination to save the changes you made and close the file.

    >**Note**: We will deploy an Azure VM into the resource group we created at the beginning of the lab. Use the following values for the deployment: 

    | Setting | Value |
    | --- | --- |
    | Resource group | **az400m14l03arg** |
    | Virtual network | **az400m1403vm1VNET** |
    | Subnet | **az400m1403vm1Subnet** |

    >**Note**: Note: The variables can be defined inside of playbooks or can be entered at runtime when invoking the `ansible-playbook` command by including the `--extra-vars` option. As the VM name, use only up to 15 lower case letters and numbers (no hyphens, underscore signs or upper case letters) and ensure it is globally unique, since the same name is used to generate the DNS name for the public IP address associated with the corresponding Azure VM. 

1.  In the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to deploy the sample ansible playbook that provisions an Azure VM (where the `<VM_name>` placeholder represents the unique VM name you chose):

    ```bash
    ansible-playbook ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_vm_web.yml --extra-vars "vmname=<VM_name> resgrp=az400m14l03arg vnet=az400m1403vm1VNET subnet=az400m1403vm1Subnet"
    ```

    >**Note**: You might receive the following errors if you entered an invalid VM name:

    - `fatal: [localhost]: FAILED! => {"changed": false, "failed": true, "msg": "The storage account named storageaccountname is already taken. - Reason.already_exists"}`. To resolve this, use another name for the Azure VM, since the one you used is not globally unique.
    - `fatal: [localhost]: FAILED! => {"changed": false, "failed": true, "msg": "Error creating or updating your-vm-name - Azure Error: InvalidDomainNameLabel\nMessage: The domain name label for your VM is invalid. It must conform to the following regular expression: ^[a-z][a-z0-9-]{1,61}[a-z0-9]$.”}`. To resolve this issue, use another name for the Azure VM following the required naming convention. 

    >**Note**: Wait for the deployment to complete. This might take about 3 minutes. 

    >**Note**: Once the deployment completes, you can run the dynamic inventory, to verify that the new VM is now detected by Ansible. 

1.  In the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to generate a dynamic inventory:

    ```bash
    python /usr/local/lib/python2.7/dist-packages/ansible_collections/community/general/scripts/inventory/azure_rm.py --list | jq
    ```
1.  In the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to perform a ping test, verifying that the dynamic inventory file includes the newly deployed Azure VM. As before, the first time you run the test you will have to verify the SSH host key, but successive runs will not require any interaction:

    ```bash
    sudo chmod +x /usr/local/lib/python2.7/dist-packages/ansible_collections/community/general/scripts/inventory/azure_rm.py
    ansible -i /usr/local/lib/python2.7/dist-packages/ansible_collections/community/general/scripts/inventory/azure_rm.py all -m ping
    ```

    >**Note**: While the two Azure VMs deployed earlier via the Azure Cloud Shell will appear in the inventory, they will not be accessible since they do not contain the public key you generated on the Ansible control machine. Once you receive a successful response from the newly deployed Azure VM, proceed to the next task. 

1.  In case the dynamic inventory does not complete successfully, in the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to connect to the newly deployed Azure VM by using SSH (where the `<VM_name>` placeholder represents the name you assigned to the newly provisioned Azure VM):

    ```bash
    $RG2NAME='az400m14l03arg'
    $VM4NAME='<VM_name>'
    PIP=$(az vm show --show-details --resource-group $RG2NAME --name $VM4NAME --query publicIps --output tsv)
    ssh azureuser@$PIP
    ```

1.  When prompted for confirmation to proceed, type **yes** and press the **Enter** key and, once connection is established, type **exit** and press the **Enter** key again to terminate it.

#### Task 9: Configure a newly deployed Azure VM by using an Ansible playbook

In this task, you will run another Ansible playbook, this time to configure the newly created machine. You will use a playbook that installs a software package httpd and downloads an HTML page from a Github repository. Once this is completed, you will have a fully functional Web server.

>**Note**: We will use the sample playbook **~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/httpd.yml**. We will be leverage the variable **vmname** in order to modify the hosts parameter of the playbook, that defines on which host (out of the ones returned by the dynamic inventory script) the playbook will target. 

1.  In the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to verify that the newly deployed Azure VM is currently not running any web service (where the `<IP_address>` placeholder represents the public IP address assigned to the network adapter of the newly provisioned Azure VM):

    ```bash
    curl http://<IP_address>
    ```

    >**Note**: You can identify this public IP address by running the following script (where the `<VM_name>` placeholder represents the name you assigned to the newly provisioned Azure VM):

    ```bash
    $RG2NAME='az400m14l03brg'
    $VM4NAME='<VM_name>'
    PIP=$(az vm show --show-details --resource-group $RG2NAME --name $VM4NAME --query publicIps --output tsv)
    echo $PIP
    ```

1.  Verify that the response is in the format `curl: (7) Failed to connect to 52.186.157.26 port 80: Connection refused`.
1.  From the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to install the HTTP service by using the Ansible playbook (where the `<VM_name>` placeholder represents the name you assigned to the newly provisioned Azure VM): 

    ```bash
    ansible-playbook -i /usr/local/lib/python2.7/dist-packages/ansible_collections/community/general/scripts/inventory/azure_rm.py  ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/httpd.yml --extra-vars "vmname=<VM_name>"
    ```

    >**Note**: Wait for the installation to complete. This should take less than a minute. 

1.  Once the installation completes, in the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to verify that the newly deployed Azure VM is now running a web service (where the `<IP_address>` placeholder represents the public IP address assigned to the network adapter of the newly provisioned Azure VM):

    ```bash
    curl http://<IP_address>
    ```

    >**Note**: The output should have the following content: 

    ```html
     <!DOCTYPE html>
     <html lang="en">
         <head>
             <meta charset="utf-8">
             <title>Hello World</title>
         </head>
         <body>
             <h1>Hello World</h1>
             <p>
                 <br>This is a test page
                 <br>This is a test page
                 <br>This is a test page
             </p>
         </body>
     </html>
    ```

#### Task 10: Use Ansible playbooks to implement configuration management and desired state in Azure

In this task, you will use Ansible playbooks to implement configuration management and desired state in Azure.

>**Note**: We could run the `ansible-playbook` command periodically to make sure that the configuration remains consistent with the content of the playbook corresponding playbook. To accomplish this in an automated manner, we will leverage the Linux cron functionality. In this task, we will run the command every minute, but in a production environment, you would likely choose a lower frequency.

>**Note**: We will use an Ansible playbook to set up our cron job. 

1.  From the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to open the Ansible configuration file in the Nano text editor:

    ```bash
    sudo nano /etc/ansible/ansible.cfg
    ```

1.  Within the Nano editor interface, in the `[Default]` section, remove the leading hash character `#` from the line `#host_key_checking = False`
1.  Within the Nano editor interface, press **ctrl + o** key combination, press the **Enter** key, and then press **ctrl + x** key combination to save the changes you made and close the file.
1.  From the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to open the playbook that configures a cron job in the Nano text editor:

    ```bash
    nano ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/cron.yml
    ```

1.  Within the Nano editor interface, in the `job` entry, replace the `< your-vm-name >` placeholder with the name of the Azure VM you configured in the previous task. 
1.  Within the Nano editor interface, set the `job` entry to `ansible-playbook -i /usr/local/lib/python2.7/dist-packages/ansible_collections/community/general/scripts/inventory/azure_rm.py  /home/azureuser/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/httpd.yml --extra-vars "vmname=<VM_name>"`, where the `<VM_name>` placeholder represents the name you assigned to the managed Azure VM).
1.  Within the Nano editor interface, press **ctrl + o** key combination, press the **Enter** key, and then press **ctrl + x** key combination to save the changes you made and close the file.
1.  From the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to create the cron job:

    ```bash
    ansible-playbook ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/cron.yml
    ```

1.  From the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to verify that the cron job is running:

    ```bash
    sudo tail /var/log/syslog
    ```

    >**Note**: You need root privilege, since all cron jobs for all users are logged in the same file. 

1.  From the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to identify the cron jobs from the cron editor interface:

    ```bash
    crontab -e
    ```

1.  When prompted, select **1** for nano and press the **Enter** key.
1.  Within the **crontab** editor, verify the presence of the **usr/bin/ansible-playbook** entry and then press **ctrl + x** key combination to close the editor.

    >**Note**: The 5 asterisks indicate that the job runs every minute.

    >**Note**: Now let's verify that our setup works. 

1.  From the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, run the following to connect to the managed Azure VM over SSH (where the `<IP_address>` placeholder represents the public IP address assigned to the network adapter of that Azure VM):

    ```bash
    ssh azureuser@<IP_address>
    ```

1.  From the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, within the SSH session to the Azure VM configured as a Web server, run the following to verify that the web site is still operational:

    ```bash
    curl http://127.0.0.1
    ```

1.  From the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, within the SSH session to the Azure VM configured as a Web server, run the following to delete the home page of the web site:

    ```bash
    rm /var/www/html/index.html
    ```

1.  Wait for a minute and, from the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, within the SSH session to the Azure VM configured as a Web server, run the following to verify that the home page of the web site was restored:

    ```bash
    curl http://127.0.0.1
    ```

1.  From the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, within the SSH session to the Azure VM configured as a Web server, type **exit** and press the **Enter** key to return to the SSH session to the Ansible control machine.

    >**Note**: By periodically running ansible playbooks you can correct changes that alter the state of Azure VMs from their desired configuration (as defined in the playbooks). Using this approach we can prevent configuration drift and preserve the desired state. We could also follow the same approach and run Ansible playbooks periodically targeting Azure resources. This helps you ensure that your infrastructure is deployed and configured as intended. 

#### Task 11: Use Ansible with Azure Resource Manager templates to facilitate configuration management and desired state of Azure resources

In this task, you will use Ansible in combination with Azure Resource Manager templates to facilitate configuration management and desired state of Azure resources.

>**Note**: As illustrated in the previous task, Ansible can be used to fix configuration deviations of existing resources supported by the corresponding modules. However, you can also use Ansible to deploy a playbook that will reference an Azure Resource Manager template, which provides direct access to the functionality and resources offered by Azure Resource Manager. 

>**Note**: We will deploy another Azure VM, but this time, we will use an ansible playbook that references an Azure Resource Manager template. For the sake of simplicity, we will use [a very straightforward Azure QuickStart template that provisions a single storage account](https://github.com/Azure/azure-quickstart-templates/tree/master/101-storage-account-create). You can identify the relevant playbook syntax by reviewing the **/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_ARM_deployment.yml** playbook.

1.  From the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, within the SSH session to the Azure VM configured as a Web server, run the following to open in the Nano text editor the playbook that invokes an Azure Resource Manager template:

    ```bash
    nano  ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_ARM_deployment.yml
    ```

1.  Within the Nano editor interface, in the `templateLink:` entry, replace the current URL with `https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json`.
1.  Within the Nano editor interface, press **ctrl + o** key combination, press the **Enter** key, and then press **ctrl + x** key combination to save the changes you made and close the file.
1.  From the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, within the SSH session to the Azure VM configured as a Web server, run the following to execute the playbook (replace the `<Azure_region>` placeholder with the name of the Azure region to which you deployed all resources in this lab):

    ```bash
    ansible-playbook ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_ARM_deployment.yml --extra-vars "resgrp=az400m14l03brg location=<Azure_region>"
    ```

    >**Note**: It is important to realize that ARM template deployments are idempotent, which means that deploying an ARM template once has the same effect as deploying that template multiple times. In other words, you can safely redeploy ARM templates to the same resource group without concern for duplicate resources being created. Effectively, you can schedule redeployment of ARM templates in regular intervals, for example, by relying the Linux cron utility, as demonstrated in the previous task. 

1.  From the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, within the SSH session to the Azure VM configured as a Web server, run the following to execute the same playbook (replace the `<Azure_region>` placeholder with the name of the Azure region to which you deployed all resources in this lab).

    ```bash
    ansible-playbook ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_ARM_deployment.yml --extra-vars "resgrp=az400m14l03brg location=<Azure_region>"
    ```

    >**Note**: Now we modify the storage account deployed by the template. Such changes might be difficult to detect, but could have detrimental impact on your environment. Therefore, it is useful to have a mechanism to automatically revert these changes back to their desired state. In this case, we will change the replication settings of the storage account. 

1.  From the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, within the SSH session to the Azure VM configured as a Web server, run the following to identify the current settings of the storage account you deployed earlier in this task:

    ```bash
    RGNAME=az400m14l03brg
    STORAGEACCOUNTNAME=$(az storage account list --resource-group $RGNAME --query "[].name" --output tsv)
    az storage account show \
    --name $STORAGEACCOUNTNAME \
    --resource-group $RGNAME \
    --query sku
    ```

1.  From the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, within the SSH session to the Azure VM configured as a Web server, run the following to change the SKU from `Standard_LRS` to `Standard_GRS` and verify that the change took place: 

    ```bash
    az storage account update \
    --name $STORAGEACCOUNTNAME \
    --resource-group $RGNAME \
    --sku 'Standard_GRS'

    az storage account show \
    --name $STORAGEACCOUNTNAME \
    --resource-group $RGNAME \
    --query sku
    ```

    >**Note**: Now re-run the playbook that deploys the same template. 

1.  From the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, within the SSH session to the Azure VM configured as a Web server, run the following to re-run the same playbook (replace the `<Azure_region>` placeholder with the name of the Azure region to which you deployed all resources in this lab):

    ```bash
    ansible-playbook ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_ARM_deployment.yml --extra-vars "resgrp=az400m14l03brg location=<Azure_region>"
    ```

1.  From the Bash session in the Cloud Shell pane, within the SSH session to the Azure VM configured as the Ansible control machine, within the SSH session to the Azure VM configured as a Web server, run the following to verify that the change was reverted: 

    ```bash
    az storage account show \
    --name $STORAGEACCOUNTNAME \
    --resource-group $RGNAME \
    --query sku
    ```

### Exercise 3: Remove the Azure lab resources

In this exercise, you will remove the Azure resources provisione in this lab to eliminate unexpected charges. 

>**Note**: Remember to remove any newly created Azure resources that you no longer use. Removing unused resources ensures you will not see unexpected charges.

#### Task 1: Remove the Azure lab resources

In this task, you will use Azure Cloud Shell to remove the Azure resources provisione in this lab to eliminate unnecessary charges. 

1.  In the Azure portal, open the **Bash** shell session within the **Cloud Shell** pane.
1.  List all resource groups created throughout the labs of this module by running the following command:

    ```sh
    az group list --query "[?starts_with(name,'az400m14l03')].name" --output tsv
    ```

1.  Delete all resource groups you created throughout the labs of this module by running the following command:

    ```sh
    az group list --query "[?starts_with(name,'az400m14l03')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**Note**: The command executes asynchronously (as determined by the --nowait parameter), so while you will be able to run another Azure CLI command immediately afterwards within the same Bash session, it will take a few minutes before the resource groups are actually removed.

## Review

In this lab, you learned how to deploy, configure, and manage Azure resources by using Ansible. 

