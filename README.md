# dcos-azure-arm-template

DC/OS Azure ARM Template

An Azure ARM Template for launching an Enteprise DC/OS cluster in the Azure cloud environment.

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

| Server Type           | OS Type   | VM Size          | VM Size Details                    |
| --------------------- | --------- | ---------------- | ---------------------------------- |
|Jump Server:           |Ubuntu     |  Standard_A1     | 1 vcpu, <2gb RAM,  10gb Disk       |
|Bootstrap Server:      |CoreOS     |  Standard_A2     | 2 vcpu, 3.5gb RAM, 200gb Disk      |
|3 Master Nodes:        |CoreOS     |  Standard_D12_v2 | 4 vcpu, 28gb RAM,  200gb Disk      |
|n Private Agent Nodes: |CoreOS     |  Standard_D12_v2 | 4 vcpu, 28gb RAM,  200gb Disk      |
|n Public Agent Nodes:  |CoreOS     |  Standard_D2_v2  | 2 vcpu,  7gb RAM,  100gb Disk      |


Create a new Azure Resource Group to contain this new DC/OS Cluster's resources 

$ az group create --location westus --name My-Proj-DCOS-Group-1

Deploy a new Enterprise DC/OS cluster using the ARM template

$ az group deployment create --name My-Proj-DCOS-Cluster-1 --resource-group My-Proj-DCOS-Group-1 --template-file Create_Ent_DCOS_Azure_Cluster.json | tee /tmp/az-deployment1.out

During this process, you will be prompted for the following parameters:

SSHPublicKey:

     public ssh key contents (e.g. ssh-rsa 9QTWQijZesCanLSf5nwYCTMsNGlUf ...)

customerKey (Get this from your Mesosphere sales engineer):

     XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX

securityMode:

     [1] permissive 
     
     or 
     
     [2] strict

dcosPassword: 

     new password for bootstrapuser user 

dcosInstallerURL (Get this from your Mesosphere sales engineer):

     https://downloads.mesosphere.io/<path to installer download>

dcosClusterName: 

     My-Project DC/OS Cluster 1

privateAgentCount:

     [5] 5

publicAgentCount:

     [1] 1

When your DC/OS cluster is successfully deployed, you can continue to the next step to access your cluster.

NOTE ON RUNNING SMACK STACK: 

If you want to run the SMACK Stack on this cluster (Spark, Mesos, Akka, Kafka, and Cassandra) plus Apache Hadoop HDFS, you may want to deploy the DC/OS cluster with a sub-set of private agent nodes that are configured as "storage nodes". In that case, you can use the Azure ARM template Create_Ent_DCOS_Azure_Cluster_With_3_Storage_Nodes.json to deploy an additional 3 private agent nodes that have 3 "data disks" attached to them. The new ARM template will prompt you for two new parameters:

    - privateAgentStorageNodeCount

    - storageNodeDataDiskSizeInGB

To use the SMACK Stack oriented ARM template, follow these instructions:

Create a new Azure Resource Group to contain this new DC/OS Cluster's resources

     $ az group create --location westus --name My-Proj-DCOS-Group-1

Deploy a new Enterprise DC/OS cluster using the ARM template

     $ az group deployment create --name My-Proj-DCOS-Cluster-1 --resource-group My-Proj-DCOS-Group-1 --template-file Create_Ent_DCOS_Azure_Cluster_With_3_Storage_Nodes.json | tee /tmp/az-deployment1.out

During this process, you will be prompted for the following parameters:

Please provide int value for 'privateAgentStorageNodeCount' (? for help):
 [1] 1
 [2] 2
 [3] 3
 [4] 4
 [5] 5
 [6] 6
 [7] 7
 [8] 8
 [9] 9
 [10] 10
 [11] 15
 [12] 20
 [13] 30
 [14] 0
Please enter a choice [1]: 3

Please provide int value for 'storageNodeDataDiskSizeInGB' (? for help):
 [1] 100
 [2] 200
 [3] 300
 [4] 400
 [5] 500
 [6] 700
 [7] 900
 [8] 1024
 [9] 2048
 [10] 3096
Please enter a choice [1]: 1

In the screen shot below, you can see that this private agent node has two things that are new:

- A new Mesos Attribute named STORAGE_NODE with a value of TRUE. This can be used later, when you want to start up a task and want it to run on the storage node types.

- A larger disk volume size. This new ARM template creates 3 data disks on each storage node with a minimum disk volume size of 100GB. So in this screen shot, you can see over 300GB available. The 23 GB general ROOT type volume storage, not MOUNT type volume storage available on the agent node.

![Alt text](/resources/Storage-Node.jpg?raw=true "DC/OS Storage Node Details")

# Step 4. Access the Enterprise DC/OS Cluster

When the DC/OS cluster is up and running, you can get the public ip address of your jump server by using the following AZ command:

     $ az network public-ip show -g My-Proj-DCOS-Group-1 -n masterLBPIP --query "{ fqdn:dnsSettings.fqdn, address: ipAddress }"

Or, you can go to your Azure Console and open the resource group you created to contain these resources and look for the master load balancer public ip address (masterLBPIP) object.

Click on the masterLBPIP object and view the details.

Copy the DNS name and paste it into your web browser's address bar to display the DC/OS Dashboard log in page.

NOTE: You may have to hit the "Refresh" button several times to get the login prompt.

When prompted for a user id and password, enter the user "bootstrapuser" and then the dcosPassword you provided when you launched the cluster with the ARM template.

![Alt text](/resources/dcos_azure_login.jpg?raw=true "DC/OS Dashboard Login Screen")

# Step 5. Access your DC/OS cluster nodes via SSH

When the DC/OS cluster is up and running, you can get the public ip address of your jump server by using the following AZ command:

     $ az network public-ip show -g My-Proj-DCOS-Group-1 -n jumpPIP --query "{ fqdn:dnsSettings.fqdn, address: ipAddress }"

SSH to the Azure jump server for this cluster

     $ ssh -i ~/.ssh/my-priv-ssh-key core@jump-server-public-ip-address

Copy your SSH Private key to the jump server

     $ cat > ~/.ssh/defaultkey.key

       copy and paste your private ssh key from your laptop

     [CTRL-D]

     $ chmod 400 ~/.ssh/defaultkey.key

SSH to the DC/OS Bootstrap server

     $ ssh -i ~/.ssh/defaultkey.key core@10.0.0.5

OR 

SSH to one of your masters

     $ ssh -i ~/.ssh/defaultkey.key core@master-node-private-ip-address

# Step 6. Destroy your Azure DC/OS Cluster

Destroy the Azure resource group 

     $ az group delete --name My-Proj-DCOS-Group-1

