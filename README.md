# OpenShift on AWS (Advanced installation guide)
This guide will provide step by step instructions on how to install OpenShift Container Platform on AWS using advanced installation option. 
These instructions help setup an OpenShift cluster with 1 ansible server, 1 master, 1 etcd and 2 app nodes. It uses default VPC. It does not create ASG (Auto Scaling Groups). Ansible server is also used an NFS storage. EBS volumes are used for storage.

A quickstart guide is available on AWS to install OpenShift. Please refer this [URL](https://aws.amazon.com/quickstart/architecture/openshift/)

**Note:** The intent of these instructions is to help someone understand OpenShift installation procedure. This guide does not help setup a Production grade OpenShift cluster. 

# Prerequisites
* AWS account
* Red Hat subscription

For the below steps, please login to AWS Management Console (https://aws.amazon.com/)

# 1. Create Key Pairs
* Choose **Services** -> **EC2**
* From the navigation menu on the left, go to **Network & Security**
* Choose **Key Pairs**
* Select **Create Key Pair**
* Provide a name for Key Pair
* Select **Create**
* A file with extension **.pem** will be downloaded on your local machine. Please save this file in a known location

# 2. Create Security Group
This section will provide instructions on how to create Security Groups for the following purposes:
* Allow SSH access to Bastion (Ansible server) instance
* Allow SSH access to OpenShift master, etcd and nodes from Bastion instance
* Allow inbound HTTP/HTTPS traffic
* Allow TCP/UDP traffic to master, etcd and nodes in the subnet

## 2.1 Security Group - Allow SSH access to bastion instance
* Choose **Services** -> **EC2**
* From the navigation menu on the left, go to **Network & Security**
* Choose **Security Groups**
* Select **Create Security Group**
* Provide the following information in **Create Security Group** dialog:
  * **Security Group Name**: ssh-access
  * **Description**: SSH Access to OpenShift Cluster
  * **VPC**: select default VPC OR create a new VPC
* Under Security group rules, **Inbound** tab, select **Add Rule**
* Provide the following information:
  * **Type**: SSH
  * **Source**: Anywhere
* Select **Create**

## 2.2 Security Group - Allow SSH access to OpenShift cluster from bastion instance
* Choose **Services** -> **EC2**
* From the navigation menu on the left, go to **Network & Security**
* Choose **Security Groups**
* Select **Create Security Group**
* Provide the following information in **Create Security Group** dialog:
  * **Security Group Name**: ssh-inside-subnet
  * **Description**: Allows ssh access to OpenShift master, etcd and app nodes from bastion instance
  * **VPC**: select default VPC OR create a new VPC
* Under Security group rules, **Inbound** tab, select **Add Rule**
* Provide the following information:
  * **Type**: All TCP
  * **Source**: Custom (172.31.0.0/16)
* Select **Add Rule** again
* Provide the following information:
  * **Type**: All UDP
  * **Source**: Custom (172.31.0.0/16)
* Select **Add Rule** again
* Provide the following information:
  * **Type**: SSH
  * **Source**: Custom (172.31.0.0/16)
* Select **Create**

**Note:** Since we are using default VPC, source (CIDR) mentioned above is taken from the default VPCs **IPv4 CIDR**

## 2.3 Security Group - Allow TCP/UDP traffic within subnet
* Choose **Services** -> **EC2**
* From the navigation menu on the left, go to **Network & Security**
* Choose **Security Groups**
* Select **Create Security Group**
* Provide the following information in **Create Security Group** dialog:
  * **Security Group Name**: all-tcp-udp-in-subnet
  * **Description**: Allows all TCP/UDP traffic in the subnet inbound and all traffic outbound
  * **VPC**: select default VPC OR create a new VPC
* Under Security group rules, **Inbound** tab, select **Add Rule**
* Provide the following information:
  * **Type**: All TCP
  * **Source**: Custom (172.31.0.0/16)
* Select **Add Rule** again
* Provide the following information:
  * **Type**: All UDP
  * **Source**: Custom (172.31.0.0/16)
* Select **Create**

**Note:** Since we are using default VPC, source (CIDR) mentioned above is taken from the default VPCs **IPv4 CIDR**

## 2.4 Security Group - Allow HTTP/HTTPS traffic to OpenShift cluster
* Choose **Services** -> **EC2**
* From the navigation menu on the left, go to **Network & Security**
* Choose **Security Groups**
* Select **Create Security Group**
* Provide the following information in **Create Security Group** dialog:
  * **Security Group Name**: tcp-http-world
  * **Description**: Allows http(s) inbound from all
  * **VPC**: select default VPC OR create a new VPC
* Under Security group rules, **Inbound** tab, select **Add Rule**
* Provide the following information:
  * **Type**: HTTP
  * **Source**: Custom (0.0.0.0/0,::/0)
* Select **Add Rule** again
* Provide the following information:
  * **Type**: HTTPS
  * **Source**: Custom (0.0.0.0/0,::/0)
* Select **Create**

# 7. Create bastion compute instance
Bastion server will be used as Ansible server and NFS storage server.

* Choose **Services** -> **EC2**
* From **EC2 Dashboard**, select **Launch Instance**
* Select **Red Hat Enterprise Linux 7.5 (HVM), SSD Volume Type** as the AMI
* Choose **t2.micro** (1 vCPU, 1 GiB Memory) as the **Instance Type**
* Select **Next: Configure Instance Details**. Keep the default values.
* Select **Add: Storage**. We will add 2 EBS volumes. One EBS root volume with 50 GiB and another EBS volume for NFS with 100 GiB
* Increase root volume **Size(GiB)** to 50
* Select **Add New Volume** for NFS. Increase NFS volume **Size(GiB)** to 100. Check **Delete on Termination**
* Select **Add Tags**
* Create the following tags:

	Key        | Value
	-----------|--------
	name       | bastion
	cluster-id |
	role       | bastion

**Note**:cluster-id key has not been provided any value because Bastion instance 
can be used to create more than one cluster

* Select **Next: Configure Security Group**
* Choose **Select an existing security group** option
* Choose the security group with the name **ssh-access**
* Seclect **Review and Launch**. This will create a new EC2 instance
* When prompted with the dialog **Select an existing key pair or create new key pair**, please choose **Choose an existing key pair** and then select the Key Pair created in Step #1

# 8. Login and update bastion instance
After creating the bastion instance, it is a good idea to login as ec2-user using the Key Pair created in Step 1

* Use the below command to login to bastion instance

  ```
  ssh -i "<key-pair-file>.pem" ec2-user@<public-dns-name>
  ```
* Above command should successfully login as ec2-user
* Switch to root user

  ```
  sudo -i
  ```  
* Update packages on bastion instance
  
  ```
  yum update -y
  ```
* Above command should complete successfully

# 9. Create master instance

* Choose **Services** -> **EC2**
* From **EC2 Dashboard**, select **Launch Instance**
* Select **Red Hat Enterprise Linux 7.5 (HVM), SSD Volume Type** as the AMI
* Choose **t2.xlarge** (4 vCPU, 16 GiB Memory) as the **Instance Type**
* Select **Next: Configure Instance Details**. Keep the default values.
* Select **Add: Storage**. We will only the root volume. 
* Increase root volume **Size(GiB)** to 50
* Select **Add Tags**
* Create the following tags:

	Key        | Value
	-----------|--------
	name       | master
	cluster-id | c1
	role       | master

* Select **Next: Configure Security Group**
* Choose **Select an existing security group** option
* Choose the security group with the name **ssh-inside-subnet** and **tcp-http-world**
* Select **Review and Launch**. This will create a new EC2 instance
* When prompted with the dialog **Select an existing key pair or create new key pair**, please choose **Choose an existing key pair** and then select the Key Pair created in Step #1

# 10. Create etcd instance

* Choose **Services** -> **EC2**
* From **EC2 Dashboard**, select **Launch Instance**
* Select **Red Hat Enterprise Linux 7.5 (HVM), SSD Volume Type** as the AMI
* Choose **t2.xlarge** (4 vCPU, 16 GiB Memory) as the **Instance Type**
* Select **Next: Configure Instance Details**. Keep the default values.
* Select **Add: Storage**. We will only the root volume. 
* Increase root volume **Size(GiB)** to 50
* Select **Add Tags**
* Create the following tags:

	Key        | Value
	-----------|--------
	name       | infra
	cluster-id | c1
	role       | infra

* Select **Next: Configure Security Group**
* Choose **Select an existing security group** option
* Choose the security group with the name **ssh-inside-subnet** and **tcp-http-world**
* Select **Review and Launch**. This will create a new EC2 instance
* When prompted with the dialog **Select an existing key pair or create new key pair**, please choose **Choose an existing key pair** and then select the Key Pair created in Step #1

# 11. Create app node instance(s)

* Choose **Services** -> **EC2**
* From **EC2 Dashboard**, select **Launch Instance**
* Select **Red Hat Enterprise Linux 7.5 (HVM), SSD Volume Type** as the AMI
* Choose **t2.xlarge** (4 vCPU, 16 GiB Memory) as the **Instance Type**
* Select **Next: Configure Instance Details**. Keep the default values.
* Select **Add: Storage**. We will only the root volume. 
* Increase root volume **Size(GiB)** to 50
* Select **Add Tags**
* Create the following tags:

	Key        | Value
	-----------|--------
	name       | node-1
	cluster-id | c1
	role       | compute

* Select **Next: Configure Security Group**
* Choose **Select an existing security group** option
* Choose the security group with the name **ssh-inside-subnet** and **tcp-http-world**
* Select **Review and Launch**. This will create a new EC2 instance
* When prompted with the dialog **Select an existing key pair or create new key pair**, please choose **Choose an existing key pair** and then select the Key Pair created in Step #1
* Please follow the above instructions to create **node-2**

