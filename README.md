# OCI LB TrafficManagement
This repository has practical use case for LB and Global Region Traffic Management Policies

# IMPORTANT
To complete this lab, you must have an OCI account, not all services are free tier, but you should be able to complete all the lab with an active OCI trial account, if don't have one [Click Here](https://www.oracle.com/cloud/free/)

Also, to complete the Traffic Management Policies on OCI, you need to have an active DNS registration name, if don't have one, you can use [Freenom](https://www.freenom.com/)
## Summary
In this lab you are going to configure and create on OCI:
- **Network**
- **Security Lists / Network Security Groups**
- **Create Compute Instances**
- **Install and Configure HTTPD**
- **Create and Configure Load Balancers**
- **Create Public Zones**
- **Configure Public DNS Zones and Specific IPs**
- **Create an Global Traffic Management Policy**
- **Create Health Checks for your Infrastructure**
- **Put it all together to manage the network traffic of your application**
## Required tools to create this DEMO

- [**OCI TENANT**](https://www.oracle.com/cloud/free/)
- [**Free Register of DNS name**](https://www.freenom.com/)
# START
## TASK1: Generate SSH Keys
The SSH (Secure Shell) protocol is a method for secure remote login from one computer to another. SSH enables secure system administration and file transfers over insecure networks using encryption to secure the connections between endpoints. SSH keys are an important part of securely accessing Oracle Cloud Infrastructure compute instances in the cloud.

We recommend you use the [Oracle Cloud Shell](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/cloudshellintro.htm) to interface with the OCI compute instance you will create. Oracle Cloud Shell is browser-based, does not require installation or configuration of software on your laptop, and works independently of your network setup.

IMPORTANT: If the SSH key is not created correctly, you will not be able to connect to your environment and will get errors. Please ensure you create your key properly.

To start the Oracle Cloud shell, go to your Cloud console and click the cloud shell icon at the top right of the page.
<p align="center">
  <img src="./Images/CloudShell.jpg">
</p>

Once the cloud shell has started, enter the following commands. Choose the key name you can remember. This will be the keyname you will use to connect to any compute instances you create. Press Enter twice for no passphrase

```hcl
mkdir .ssh
cd .ssh
ssh-keygen -b 2048 -t rsa -f <your SSH key name>
```

Note in the output that there are two files, a private key and a public key. Keep the private key safe and don't share its content with anyone. The public key will be needed for various activities and can be uploaded to certain systems as well as copied and pasted to facilitate secure communications in the cloud.

To list the contents of the public key, use the cat command:
```hcl
cat <your SSH key name>.pub
```
Copy the contents of the public key and save it somewhere for later. When pasting the key into the compute instance in future labs, make sure that you remove any hard returns that may have been added when copying. The .pub key should be one line.

## TASK2: Create Network - VCN
From the OCI Services menu, click Virtual Cloud Networks under Networking. Select the compartment assigned to you from drop down menu on left part of the screen under Networking and Click Start VCN Wizard.
*NOTE:* Ensure the correct Compartment is selected under COMPARTMENT list

Fill out the dialog box:

```hcl
VCN NAME: Provide a name
COMPARTMENT: Ensure your compartment is selected
VCN CIDR BLOCK: Provide a CIDR block (10.0.0.0/16)
PUBLIC SUBNET CIDR BLOCK: Provide a CIDR block (10.0.1.0/24)
PRIVATE SUBNET CIDR BLOCK: Provide a CIDR block (10.0.2.0/24)
```

Click Next
Verify all the information and Click Create.

This will create a VCN with followig components:
VCN, Public subnet, Private subnet, Internet gateway (IG), NAT gateway (NAT), Service gateway (SG)
## TASK3: Create Two Compute Instance and Install Web Server
Switch to the OCI console. From OCI services menu, Click Instances under Compute.

Click Create Instance. Fill out the dialog box:

```hcl
Name your instance: Enter a name
Choose an operating system or image source: For the image, we recommend using the Latest Oracle Linux available. It is the default selection.
Availability Domain: Select availability domain
Instance Shape: Click on change shape if you want to use a different shape from the default one
Under Configure Networking
Virtual cloud network compartment: Select your compartment
Virtual cloud network: Choose the VCN you created earlier
Subnet Compartment: Choose your compartment.
Subnet: Choose the Private Subnet
Use network security groups to control traffic : Leave un-checked
Configure Boot Volume: Leave the default choices
Add SSH Keys: Choose 'Paste SSH Keys' and paste the Public Key you created in Cloud Shell earlier. Ensure when you are pasting that you paste one line
```

NOTE: The lab instruction places the instances on a <b>private subnets</b>. If you wish to access them, you can create the Bastion Service and create an SSH Session.

If you want to manually provision and configure the Apache on each instance, you can create the instance now, proceed to TASK 4.

<b>To skip the next step, you can use the "Cloud-Init" to paste all the required instance configuration procedures to be executed automatically after the instance creation, if you do so, there's no need to SSH into the instace to configure the Web Server</b>

<p align="center">
  <img src="./Images/Cloud-Init.jpg">
</p>

Paste this script on the "Cloud-Init Script" location (sa-saopaulo-1) instances:
```hcl
#! /bin/bash
sudo setenforce 0
sudo yum clean all
sudo yum -y update
sudo yum -y install httpd
sudo systemctl start httpd
sudo systemctl enable httpd
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
sudo -s 
cat <<EOF > /var/www/html/index.html
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>OCI Load Balancer</title>
</head>
<body>
    <center> <h1>The Cloud Bootcamp</h1>
            <h2>Instance provisioned and running Apache with success &#x270C;</h2>
            <h2>Instance region: sa-saopaulo-<Instance Number ID></h2>
            <hr>
            <img src="https://objectstorage.sa-saopaulo-1.oraclecloud.com/n/grri30nzv1ul/b/public/o/bandeira-do-brasil.jpg" alt="success" width="500" height="300">
    </center>
</body>
</html>
EOF
```
Paste this script on the "Cloud-Init Script" location (us-ashburn-1) instances:
```hcl
#! /bin/bash
sudo setenforce 0
sudo yum clean all
sudo yum -y update
sudo yum -y install httpd
sudo systemctl start httpd
sudo systemctl enable httpd
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
sudo -s 
cat <<EOF > /var/www/html/index.html
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>OCI Load Balancer</title>
</head>
<body>
    <center> <h1>The Cloud Bootcamp</h1>
            <h2>Instance provisioned and running Apache with success &#x270C;</h2>
            <h2>Instance region: us-ashburn-<Instance Number ID></h2>
            <hr>
            <img src="https://objectstorage.sa-saopaulo-1.oraclecloud.com/n/grri30nzv1ul/b/public/o/Bandeira-dos-Estados-Unidos.png" alt="success" width="500" height="300">
    </center>
</body>
</html>
EOF
```

Using the same method, create 2 instances on each region, just change the Instance Number ID so when you test using your browser you can see the Load Balancer Round-Robin mechanism working.

- [**OPTIONAL**]
If you want to use [Bastion Service](https://docs.oracle.com/en-us/iaas/Content/Bastion/Concepts/bastionoverview.htm) to access your instance, on the creation process, go to "Advanced Options" -> "Oracle Cloud Agent" and select the checkbox to enable "Bastion"
<p align="center">
  <img src="./Images/Bastion.jpg">
</p>

After compleating this TASK you should have on your OCI account:
- **VCN completed on Sao Paulo region**
- **2 Instances on Running State on Sao Paulo region**
- **VCN completed on Ashburn region**
- **2 Instances on Running State on Ashburn region**

## TASK4: Install Apache on each Instance
This task let's you configure manually the Apache server on each instance. <b><i>Since we created the instance on the private subnet, you need to use the Bastion Service to SSH, look at the [getting started](https://docs.oracle.com/en-us/iaas/Content/Bastion/Concepts/bastionoverview.htm#get-started) of the service to create the Bastion and then create the session to copy the SSH command</i></b>
<p align="center">
  <img src="./Images/Bastion-Session.jpg">
</p>

After you access the instance, enter these commands to install and configure Apache:

Install Apache HTTP Server
```hcl
sudo yum -y install httpd
```
Open TCP port 80 in the local firewall
```hcl
sudo firewall-cmd --permanent  --add-port=80/tcp
```
Reload the firewall to activate the rules
```hcl
sudo firewall-cmd --reload
```
Start the web server
```hcl
sudo systemctl start httpd
sudo systemctl enable httpd
```

Create the index.html file as root (you can use VI editor). The content of the file will be displayed when the web server is accessed, use the HTML code in this LAB at the Optional part of the "Cloud-Init" setup, you can copy and paste the code in each instance depending of the region.