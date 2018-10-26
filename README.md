# Advanced installation guide for OpenShift on AWS
This guide will provide step by step instructions on how to install OpenShift Container Platform on AWS using advanced installation option. <br>
A quickstart guide is available on AWS to install OpenShift. Please refer this URL https://aws.amazon.com/quickstart/architecture/openshift/

# Prerequisites
* AWS account
* Red Hat subscription

For the below steps, please login to AWS Management Console (https://aws.amazon.com/)

# Create Key Pairs
* Choose **Services** -> **EC2**
* From the navigation menu on the left, go to **Network & Security**
* Choose **Key Pairs**
* Select **Create Key Pair**
* Provide a name for Key Pair
* Select **Create**
* A file with extension **.pem** will be downloaded on your local machine. Please save this file in a well known location

# Create Security Group
This section will provide instructions on how to create Security Groups for the following purposes:
* SSH access to Bastion (Ansible server) instance
* SSH access to OpenShift master, etcd and nodes from Bastion instance
* Inbound HTTP/HTTPS traffic
* TCP/UDP traffic to master, etcd and nodes in the subnet

# SSH Access to Bastion instance
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
  
  
