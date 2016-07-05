# USE AT YOUR OWN RISK!  All the different permutations have not be tested
# YOU ARE RESPONSIBLE FOR YOUR OWN CLUSTER

This is a modified version of the Cloudera Azure templates found at: https://github.com/Azure/azure-quickstart-templates/tree/master/cloudera-on-centos

Original pull from source: 2016.06.27
#
The goal is to create a centos 6 cluster that is configured and ready for Cloudera Manager to be downloaded an installed. The nodes on the cluster are 
configured using the original scripts to put the CentOS VM's and the supporting Azure infrastructure in a state ready for Cloudera to be installed.

The major differences are:

1. Configuration variables

  - You can opt out of deploying CDH (default = False) with the new variable InstallCDH.  
  - The default "T-Shirt Size" is now Prod (3 masters, 3+ data nodes).  You can try Eval but it hasn't been tested.
  - The other variables the same as the original template

2. The 3 master nodes have the same disk configuration as before

| Filesystem | Size | Used | Avail | Use% | Mounted on |
|:--- |:---|:---|:--- |:---|:---|
| /dev/sda1 | 512M | 98M | 388M | 21% | /boot |
| /dev/sda2 | 22G | 2.1G | 18G | 11% | / |
| /dev/sda3 | 32G | 168M | 30G | 1% | /var |
| /dev/sdb1 | 237G | 63M | 225G | 1% | /mnt/resource |
| /dev/sdc | 541G | 74M | 536G | 1% | /log |
| /dev/sdd | 541G | 74M | 536G | 1% | /var/lib/pgsql |
| /dev/sde | 541G | 74M | 536G | 1% | /data/dfs |
| /dev/sdf | 541G | 74M | 536G | 1% | /log/cloudera/zookeeper |
| tmpfs | 60G | 0 | 60G | 0% | /dev/shm |

  Note: The initialization scripts create the directory /var/lib/pgsql, but postgres is not configured.  You will have to install the metadata databases

3. The data nodes template has been chnaged to have only 3 data disks

| Filesystem | Size | Used | Avail | Use% | Mounted on |
|:--- |:---|:---|:--- |:---|:---|
| /dev/sda1 | 512M | 98M | 388M | 21% | /boot |
| /dev/sda2 | 22G | 2.1G | 18G | 11% | / |
| /dev/sda3 | 32G | 168M | 30G | 1% | /var |
| /dev/sdb1 | 237G | 63M | 225G | 1%  | mnt/resource |
| /dev/sdc | 541G | 74M | 536G | 1%  | log |
| /dev/sdd | 1.1T | 75M | 1.1T | 1% | /data1 |
| /dev/sde | 1.1T | 75M | 1.1T | 1% | /data2 |
| /dev/sdf | 1.1T | 75M | 1.1T | 1% | /data0 |
| tmpfs | 60G | 0 | 60G | 0% | /dev/shm |

  Note: The initialization scripts will create a directory /data[1-3]/impala/scratch

4. If you want to modify the templates (i.e. change number of disks)

  - Data nodes data-node-ds14.json 
  - Master nodes: master-node.json

Everything else is in the other .json setup files and the shell scripts in the scripts directory.


# Click here to begin deployment of the DANF VERSION DS14 Deployment
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fdfgit%2Fazure-centos6-cld%2Fmaster%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png" />
</a>



# Readme

The template also provisions storage accounts, virtual network, availability set, network interfaces, VMs, disks and other infrastructure and runtime resources required by the installation.

The template expects the following parameters:

| Name   | Description | Default Value |
|:--- |:---|:---|
| adminUsername  | Administrator user name used when provisioning virtual machines | testuser |
| adminPassword  | Administrator password used when provisioning virtual machines | Eur32#1e |
| cmUsername | Cloudera Manager username | cmadmin |
| cmPassword | Cloudera Manager password | cmpassword |
| storageAccountPrefix | Unique namespace for the Storage Account where the Virtual Machine's disks will be placed | defaultStorageAccountPrefix |
| numberOfDataNodes | Number of data nodes to provision in the cluster | 3 |
| dnsNamePrefix | Unique public dns name where the Virtual Machines will be exposed | defaultDnsNamePrefix |
| region | Azure data center location where resources will be provisioned |  |
| storageAccountType | The type of the Storage Account to be created | Premium_LRS |
| virtualNetworkName | The name of the virtual network provisioned for the deployment | clouderaVnet |
| subnetName | Subnet name for the virtual network where resources will be provisioned | clouderaSubnet |
| tshirtSize | T-shirt size of the Cloudera cluster (Eval, Prod) | Eval |
| vmSize | The size of the VMs deployed in the cluster (Defaults to Standard_DS14) | Standard_DS14 |
| INSTALLCDH | Install CDH (default is False) | False |


Topology
--------

The deployment topology is comprised of a predefined number (as per t-shirt sizing) Cloudera member nodes configured as a cluster, configured using a set number of manager,
name and data nodes. Typical setup for Cloudera uses 3 master nodes with as many data nodes are needed for the size that has been choosen ranging from as
few as 3 to thousands of data nodes.  The current template will scale at the highest end to 200 data nodes when using the large t-shirt size.

The following table outlines the deployment topology characteristics for each supported t-shirt size:

| T-Shirt Size | Member Node VM Size | CPU Cores | Memory | Data Disks | # of Master Node VMs | Services Placement of Master Node |
|:--- |:---|:---|:---|:---|:---|:---|:---|
| Eval | Standard_DS14 | 10 | 112 GB | (See above) | 1 (primary, secondary, cloudera manager) |
| Prod | Standard_DS14 | 10 | 112 GB | (See above) | 1 primary, 1 standby (HA), 1 cloudera manager |

##Connecting to the cluster
The machines are named according to a specific pattern.  The master node is named based on parameters and using the.

	[dnsNamePrefix]-mn0.[region].cloudapp.azure.com

If the dnsNamePrefix was clouderatest in the West US region, the machine will be located at:

	clouderatest-mn0.westus.cloudapp.azure.com

The rest of the master nodes and data nodes of the cluster use the same pattern, with -mn and -dn extensions followed by their number.  For example:

    clouderatest-mn0.westus.cloudapp.azure.com
	clouderatest-mn1.westus.cloudapp.azure.com
	clouderatest-mn2.westus.cloudapp.azure.com
	clouderatest-dn0.westus.cloudapp.azure.com
	clouderatest-dn1.westus.cloudapp.azure.com
	clouderatest-dn2.westus.cloudapp.azure.com

To connect to the master node via SSH, use the username and password used for deployment

	ssh testuser@[dnsNamePrefix]-mn0.[region].cloudapp.azure.com

Once the deployment is complete, you can navigate to the Cloudera portal to watch the operation and track it's status. Be aware that the portal dashboard will report alerts since the services are still being installed.

	http://[dnsNamePrefix]-mn0.[region].cloudapp.azure.com:7180

##Notes, Known Issues & Limitations
- All nodes in the cluster have a public IP address.
- The deployment script is not yet idempotent and cannot handle updates (although it currently works for initial provisioning only)
- SSH key is not yet implemented and the template currently takes a password for the admin user
