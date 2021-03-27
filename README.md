# Demo: Deploying a Secure WordPress on AWS using CloudFormation.

## Prerequisites

- AWS account
- SSL Certificate
- SSH key
- Domain Name

## Resouces

### Network

- Virtual Private Cloud (VPC)
  - Internet Gateway 
  - Nat Gateway
  - 2 Public Subnets
  - 2 Private Subnets 
  - Public Route Table
  - Private Route Table
 
- Security Groups
  - EC2 Security Group
  - RDS Security Group
  - ALB Security Group

- Application Load Balancer

### Application

- EC2 Instance
  - WordPress (Frontend)
- RDS
  - Database (Backend)

### DNS
- Route 53

## Diagram

<img src="aws_diagram.png" alt="aws" width="800" height="500">

## Description
<p>
This template is reusable it will provision VPC with CIDR 10.0.0.0/16, with  2 Public with CIDR 10.0.1.0/24 & 10.0.2.0/24 and 2 private subnets with CIDR 10.0.11.0/24 & 10.0.11.0/24.
</p> 
<p>
After that created VPC needs to have an access to the Internet and Internet Gateway will solve that when it gets created it will attached to VPC.
</p>  
<p>For private subnets Internet comes with NAT Gateway it will be created with Elastic IP address (the reason behind it,if you want to updates your website it has to have static IP) which will attached to one of the public subnets, because in that manner private subnets won’t be open to the world it's secure. 
</p>
<p>
The next resource is Route tables (public and private) 2 public subnets will be assosiated with  Public-RT attached with Internet Gateway and 2 private subnets will be assosiated with  Private-RT which is attached to Nat Gateway. 
</p>

The next step is security groups: 
  - Load balancer security group  with HTTPS 443 and HTTP 80 ports open to 0.0.0.0/0.
  - RDS security group with MySQL port 3306 open to WordPress host's Security Group. 
  - WordPress host's Security group with port MySQL 3306 open to RDS's Security Group, and HTTP port 80 open to ELB Security Group.

## WordPress host
<p>
For this Demo, Amazon LINUX 2 machine (AMI) and t2.micro instance type were used and bash script was added in the user data section. This bash script will download php, httpd,mysql-agent and Wordpress package and unzippes it.  
</p>

### UserData
```
#!/bin/bash
sudo hostnamectl set-hostname wordpress-web
sudo amazon-linux-extras install -y php7.2
sudo yum install -y httpd 
sudo systemctl start httpd
sudo systemctl enable httpd
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo yum install php-gd -y
sudo yum install mysql -y 
sudo systemctl restart httpd
sudo cp -r wordpress/* /var/www/html
sudo chown -R apache:apache /var/www/html
sudo systemctl restart httpd   
```

## RDS database    
<p>
RDS db will be created with an engine MariaDB and version 10.4.8, database instance class, storage type , allocated storage will be chosen from the RDS database parameters. 

Enter RDS db master _```username```_ and _```password```_.

RDS Security group public access will be false for security reason. For learning purposes no backing up ,storage isn't encrypted because db.t2.micro is too small.  
</p>

## Target Group and Application Load Balancer. 

<p>
Target group created with health check enabled, since our target type is "instance" in our case it will be WordPress host, also two Listener rules HTTP and HTTPS both of them forwarded to target group. Application Load Balancer's scheme is internet facing (because we want our customers to see our website), VPC with public subnets  will be chosen otherwise it won’t work. Because only public subnets have internet, if you choose private subnet it will keep hitting your NAT gateway and eventually it will drop it. Enter your ACMcertificate to make your website secure if you have one, if not you need to create it. 
</p>

## Route 53

The last resource is Route 53, Hosted Zone Name will be available from your hosted zone. Alias target will be copied from  Hosted ZoneID and DNS name. Keep in mind that Hosted Zone Id and Alias Hosted Zone Id is different in every region. The following link has all Regions, Route 53 Hosted Zone IDs (Application Load Balancers, Classic Load Balancers) and Route 53 Hosted Zone IDs (Network Load Balancers).  
[AWS Documentation - Hosted Zone IDs](https://docs.aws.amazon.com/general/latest/gr/elb.html)
Another important thing to remember to keep in mind that, always put . after your domain name, its in AWS  documentation. As it's shown on lines 558 and 561.

## Notes 
<p>
The following  plugins are required to be installed and activated in the WordPress: 
- JSM force ssl
  - JSM's Force HTTP to HTTPS (SSL) – Simple, Safe, Reliable, and Fast!
- Simple 301 redirect 
  - Redirection

These plugins helps you to make your application secure , without redirectiong  your HTTP/80 listener to HTTPS/443.
</p>
