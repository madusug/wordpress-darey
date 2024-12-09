# CAPSTONE - WORDPRESS SITE ON AWS - DIGITAL BOOST

Digital Boost wants to enhance its online presence by creating a high performing wordpress based website for their clients. The agency has noted that they need a scalable, secure and cost-effective solution that can handle increasing traffic and seamingly integrate with their existing infrastructure. I have been tasked to design and implement a wordpress solution using various AWS services, such as Networking, Compute, Objust Storage and Databases.

## VPC ARCHITECTURE

1. The first step to accomplish this would be define the IP Address range for the VPC and subnets using the Classless Inter-Domain Routing (CIDR). Using IPV4, I decided to utilize 10.x.x.x/n (10.0.0.0/16), where “10.0.0.0” represents the network address, and “/16” indicates that the first 16 bits are fixed and the remaining 16 bits are available for host assignment. In this case, the VPC can accommodate up to 65, 536 IP addresses.
This IP address will be associated with the VPC to isolate and secure the WordPress infrastructure.

2. Next, I will define the IP Addresses for the subnets. There will be 6 subnets distributed across multiple AZs for High Availability and Fault Tolerance. The subnets help divide a large network into smaller, more efficient networks. This will help reduce network congestion, improve security, improve IP address management, and easier network administration.

3. Next, I will create route tables, which are a set of rules, called routes, that determine where network traffic from a subnet or gateway is direct. I will create 3 route tables as follows:
- Public Route Table, and associate public subnets (edit subnet association option on AWS) to the route table and route traffic (edit routes option on AWS) to the internet through the Internet Gateway
- Private Route Table, and associate Availability Zone 1 private subnets (edit subnet association option on AWS) to the route table and route (edit routes option on AWS) the private subnets to the NAT Gateway in Availability Zone 1 for restricted access to the internet access
- Second Private Route Table, and associate Availability Zone 2 private subnets (edit subnet association option on AWS) to the route table and route (edit routes option on AWS) the private subnets to the NAT Gateway in Availability Zone 2 for restricted access to the internet access

4. So far, we have accounted for the following:
- IP Addresses
- VPC
- Route Tables
- Subnets

Next, I want my virtual private cloud (vpc) to have access to the internet. To do this, an internet gateway is needed. An internet gateway is a component that allows communication between a VPC and the internet. This will allow VPC resources to have access to the internet. Resources such as the public subnet can now connect to the internet.

5. After creating the Internet Gateway, the next step will be to create a NAT Gateway. The NAT Gateway will allow private resources to securely access services outside of the VPC while keeping them inaccessible to unsolicited traffic.



## STEPS:

### VPC Architecture

First, I defined the IP Addresses for my VPC, public subnets and private subnets as follows:
- VPC: 10.0.0.0/16
- Public Subnet AZ1: 10.0.0.0/24
- Public Subnet AZ2: 10.0.1.0/24
- Private App Subnet AZ1: 10.0.2.0/24
- Private App Subnet AZ2: 10.0.3.0/24
- Private Data Subnet AZ1: 10.0.4.0/24
- Private Data Subnet AZ2: 10.0.5.0/24

 VPC: I created a VPC with 1 public subnet in us-east-1a and 1 public subnet in us-east-1b, making it two availability zones.

 ![VPC](./img/1%20vpc.jpg)

Subnets: I also created two private subnets in us-east-1a and two private subnets in us-east-1b. For a total of six subnets (1 public subnet and 2 private subnets in us-east-1a, and 1 public subnet and 2 private subnets in us-east-1b)

![Subnets](./img/2%20subnets.jpg)

Internet Gateway: I created an internet gateway for the VPC to provide the VPC with access to the internet.

![Internet_Gateway](./img/7%20Internet%20Gateway.jpg)

NAT Gateway: I created two NAT Gateways in the public subnet in us-east-1a and us-east-1b respectively

![NAT_Gateway](./img/8%20Nat%20Gateways.jpg)

Route Tables and Subnet Association: I created 3 route tables, one public and two private. I associated the public route table to the two public subnets in Availability Zone 1 and Availability Zone 2, one private route table to the two private subnets in Availability Zone 1, and one private route table to the last two private subnets in Availability Zone 2.

![Route_Tables](./img/3%20route%20table2.jpg)

![RT_Subnet_Asso](./img/4%20rt%20subnet%20association.jpg)

![RT_Subnet_Asso2](./img/4%20rt%20subnet%20association2.jpg)

![RT_Subnet_Asso3](./img/4%20rt%20subnet%20association3.jpg)


Connect Route Tables to Internet Gateway and NAT Gateway: I editted the routes of each of the three route tables and connected the public route table to the internet gateway and the routes of the two private route tables to NAT gateways in AZ1 and AZ2 respectively.

![Edit_Routes](./img/5%20Editted%20Routes.jpg)
![Edit_Routes](./img/5%20Editted%20Routes1.jpg)



## RDS
Next, I started setting up a managed database service that helps organizations set up, operate and manage relational databases in AWS cloud. This is Amazon Relational Database Service (RDS).

I created the MySQL RDS database  using the Dev/Test Template and setting Availability and durability to Multi-AZ DB instance so that a primary DB instance is created and a standby DB instance is created in a different AZ. This provides high availability and data redundancy. I also connected to an Ec2 compute resource and selected the wordpress server. This in turn automatically selected the right VPC.

See image below:

![RDS](./img/9%20RDS.jpg)
![RDS2](./img/9%20RDS2.jpg)
![RDS3](./img/9%20RDS3.jpg)

## Wordpress Instances

Next, I created two servers for the wordpress based website in the private subnet in availability zone one and two each. 

![Wordpress](./img/11%20Wordpress.jpg)

To install wordpress on my instances, I had to create a bastion host in my public subnet so I can ssh into it.

![Bastion](./img/12%20Bastion.jpg)

### Networking for Bastion Host and Private Wordpress Servers.

In order for me to be able to install wordpress and perform other functions on my private web servers, I created a security group that specifically allows ssh access to the IP Adress of my Bastion host and added the newly created security group to the two private webservers.

I edited the security group (inbound rules) of my private instances and added an ssh rule (port 22) for my bastion host, using my bastion host's ip-address eg: ip-address/32. This allowed me to ssh (ssh -i key.pem ec2-user@privateIPAddress).

![Bastion1](./img/12%20Bastion1.jpg)


## Elastic File System (EFS)

I created an elastic file system and the next step was mounting it on the wordpress servers. To do this, I ssh'd into my bastion host and created a key.pem with the same name as the key.pem I used for my private servers. I then copied the content of the key.pem on my local and pasted it into the key.pem I created on my bastion host. After saving the file, I ssh'd into my private server in availability zone 1 and configured aws using aws config. Then I put in my AWS key ID and Secret Key ID. Next, I Installed amazon-efs-utils using "sudo yum install -y amazon-efs-utils". Next, I switched to my root user and ran the following commands to both mount my EFS volume and install wordpress on my servers: 

>yum update -y
mkdir -p /var/www/html
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport 10.0.4.154:/ /var/www/html

>#2. install apache 
sudo yum install -y httpd httpd-tools mod_ssl
sudo systemctl enable httpd 
sudo systemctl start httpd


>#3. install php 7.4
sudo amazon-linux-extras enable php7.4
sudo yum clean metadata
sudo yum install php php-common php-pear -y
sudo yum install php-{cgi,curl,mbstring,gd,mysqlnd,gettext,json,xml,fpm,intl,zip} -y


>#4. install mysql5.7
sudo rpm -Uvh https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
sudo yum install mysql-community-server -y
sudo systemctl enable mysqld
sudo systemctl start mysqld


>#5. set permissions
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
chown apache:apache -R /var/www/html 


>#6. download wordpress files
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
cp -r wordpress/* /var/www/html/


>#7. create the wp-config.php file
cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php


>#8. edit the wp-config.php file
nano /var/www/html/wp-config.php


>#9. restart the webserver
service httpd restart`






I repeated the same process for my private server in availability zone 2.

![EFS](./img/13%20EFS.jpg)

## Application Load Balancer

I created an instance type target group with Port protocol HTTP (port 80)

![Target](./img/14%20Target.jpg)

I also created an application load balancer in the VPC of my choice and selected two availability zones for mapping. The load balancer is not internet facing, so I chose internal. I configured my listener on protocol HTTP at Port 80 and selected the target group I created above. Thus associating them.

![LoadBalancer](./img/15%20load%20balancer.jpg)

## Auto Scaling Group

The first step here was to create a launch template. This is the first step in creating an asg. I used an AMI currently in use and used the typical configuration I used for my wordpress server. I then populated the user data with the resources I want my instances to be provisioned with.
![LT](./img/16%20LT.jpg)

I then proceeded to create the Auto Scaling group using the launch template I configured. I used the wordpress VPC and choose multiple availability zones so that my instances can be balanced across multiple zones. I also associated it to a Load balancer target 

![ASG](./img/17%20ASG.jpg)