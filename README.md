# dcos-azure-arm-template

DC/OS Azure ARM Template

An Azure ARM Tempalte for launching an Enteprise DC/OS cluster in the Azure cloud environment.

     *** NOTE: This ARM template is provided for convenience ***
     *** and is not directly supported by Mesosphere, Inc.   ***

# Step 1. Clone this git repo

$ git clone https://github.com/gregpalmr/dcos-azure-arm-template.git

$ cd dcos-azure-arm-template

# Step 2. Setup your Azure CLI 2.0

Follow these instructions for Mac and Windows:

     https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest

Login using the Azure CLI - You will be prompted to authenticate via the Azure Web page

$ az login

# Step 3. Use the Supplied ARM template to launch a new DC/OS cluster in Azure

This ARM template will launch 3 master nodes, and a number of private agent and public agent nodes. It will use the following resource types and sizes:

| Server Type           | OS Type   | VM Size                                              |
| --------------------- | --------- | ---------------------------------------------------- |
|Jump Server:           |Ubuntu     |  Standard_A1     (1 vcpu, <2gb RAM,  10gb Disk)      |
|Bootstrap Server:      |CoreOS     |  Standard_A2     (2 vcpu, 3.5gb RAM, 200gb Disk)     |
|3 Master Nodes:        |CoreOS     |  Standard_D12_v2 (4 vcpu, 28gb RAM,  200gb Disk)     |
|n Private Agent Nodes: |CoreOS     |  Standard_D12_v2 (4 vcpu, 28gb RAM,  200gb Disk)     |
|n Public Agent Nodes:  |CoreOS     |  Standard_D2_v2  (2 vcpu,  7gb RAM,  100gb Disk)     |


Create a new Azure Resource Group to contain this new DC/OS Cluster's resources 

$ az group create --location westus --name My-Proj-DCOS-Group-1

Deploy a new Enterprise DC/OS cluster using the ARM template

$ az group deployment create &#92;
    --name My-Proj-DCOS-Cluster-1 &#92;
    --resource-group My-Proj-DCOS-Group-1 &#92;
    --template-file Create_Ent_DCOS_Azure_Cluster.json &#92;
    | tee az-deployment1.out

During this process, you will be prompted for the following paramters:

SSHPublicKey:

     ssh-rsa <public ssh key contents>

customerKey: 

     XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX

dcosPassword: 

     <new dcosadmon user's password>

dcosInstallerURL: 

     https://downloads.mesosphere.io/<path to installer download>

dcosClusterName: 

     My-Projct DC/OS Cluster 1

privateAgentCount:

     [7] 9

publicAgentCount:

     [2] 1


# Step 4 - Access the Enterprise DC/OS Cluster

When the DC/OS cluster is up and running, go to your Azure Console and open the resource group you created to contain these resources and look for the master load balancer public ip address (masterLBPIP) object.

Click on the masterLBPIP object and view the details.

Copy the DNS name and paste it into your web browser's address bar to display the DC/OS Dashboard log in page.

NOTE: You may have to hit the "Refresh" button several times to get the login prompt.

![Alt text](/resources/dcos_azure_login.jpg?raw=true "DC/OS Dashboard Login Screen")

# Step 5 - SSH to the Azure jump server for this cluster

$ ssh -i ~/.ssh/my-priv-ssh-key core@<jump-server-public-ip>

Copy your SSH Private key to the jump server

$ cat > ~/.ssh/defaultkey.key
<copy and paste your private ssh key from your laptop>
[CTRL-D]

SSH to the DC/OS Bootstrap server

$ ssh -i ~/.ssh/defaultkey.key core@<bootstrap server private ip address>

OR 

SSH to one of your masters

$ ssh -i ~/.ssh/defaultkey.key core@<master node 1 private ip address>


