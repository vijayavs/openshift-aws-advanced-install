# Advanced installation guide for OpenShift on AWS
This guide will provide step by step instructions on how to install OpenShift Container Platform on AWS using advanced installation option. A quickstart guide is available on AWS to install OpenShift. Please refer this URL https://aws.amazon.com/quickstart/architecture/openshift/

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
* A file with extension **.pem** will be downloaded on your local machine. Please save this file in a well known location

# 2. Create Security Group
This section will provide instructions on how to create Security Groups for the following purposes:
* Allow SSH access to Bastion (Ansible server) instance
* Allow SSH access to OpenShift master, etcd and nodes from Bastion instance
* Allow inbound HTTP/HTTPS traffic
* Allow TCP/UDP traffic to master, etcd and nodes in the subnet

# 3. Allow SSH access to bastion instance
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

# 4. Create bastion compute instance
* Choose **Services** -> **EC2**
* From **EC2 Dashboard**, select **Launch Instance**
* Select **Red Hat Enterprise Linux 7.5 (HVM), SSD Volume Type** as the AMI
* Choose **t2.micro** (1 vCPU, 1 GiB Memory) as the **Instance Type**
* Select **Next: Configure Instance Details**. Let the default values be as is.
* Select **Add: Storage**
* Increase **Size(GiB)** to 50
* Select **Add New Volume**. Increase **Size(GiB)** to 100. Check **Delete on Termination**
* Select **Add Tags**
* Create the following tags:
  * Key=Name Value=bastion
  * Key=cluster-id Value=
  * Key=role Value=bastion
**Note**:cluster-id key has not been provided any value because Bastion instance can be used to create more than one cluster
* Select **Next: Configure Security Group**
* Choose **Select an existing security group** option
* Choose the security group with the name **ssh-access**
* Seclect **Review and Launch**. This will create a new EC2 instance.

# 5. Allow SSH access to OpenShift cluster from bastion instance
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
  * **Source**: 172.31.0.0/16
* Select **Add Rule** again
* Provide the following information:
  * **Type**: All UDP
  * **Source**: 172.31.0.0/16
* Select **Create**
