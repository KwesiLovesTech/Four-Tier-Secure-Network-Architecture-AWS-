# 🛡️ Four-Tier Secure Network Architecture (AWS)

This project simulates a secure, production-grade AWS deployment using a **four-tier architecture**. I initially **built it manually using the AWS Console (UI)** to reinforce core AWS networking and security concepts, then **replicated the entire setup using Terraform** to demonstrate Infrastructure as Code (IaC) automation.

---

### 💡 Overview

The environment follows a four-tier model with strict network isolation:

- **Public Tier**: Application Load Balancer (ALB) in public subnets  
- **Web Tier**: EC2 instances in private web subnets  
- **App Tier**: Backend services (e.g., microservices ) in private app subnets  
- **Data Tier**: Amazon RDS and ElastiCache in isolated DB subnets  

Each tier is designed to have strict network segmentation and layered security.

Below is the corresponding architectural diagram:


![4 tier architecture](https://github.com/user-attachments/assets/79f51f56-d9b2-4b32-b653-f27db49b02fc)

---

## #🔐 Project Components

#### VPC Segmentation & Network ACLs
- Custom VPC with isolated subnets for each tier
- Security Groups enforcing least privilege (per-tier)
- NACLs restricting lateral movement between subnet groups

---

## 📋 Step-by-Step Instructions


## STEP 1: Create The Base Networking Infrastructure For NAT/ELB, Webservers, Appservers and Database
### A) Create The VPC Network
- Name: `Prod-VPC`
- CidirBlock: `10.0.0.0/16`

### B) Create The NAT/AL Subnet 1 and 2
1. NAT/ALB Subnet 1
- Name: `Prod-NAT-ALB-Subnet-1`
- CidirBlock: `10.0.5.0`
- Availability Zone: `us-west-1a`

2. NAT/ALB Subnet 2
- Name: `Prod-NAT-ALB-Subnet-2`
- CidirBlock: `10.0.10.0`
- Availability Zone: `us-west-1c`

### B) Create The Webserver Subnet 1 and 2
1. Webserver Subnet 1
- Name: `Prod-Webserver-Subnet-1`
- CidirBlock: `10.0.15.0`
- Availability Zone: `us-west-1a`

2. Webserver Subnet 2
- Name: `Prod-Webserver-Subnet-2`
- CidirBlock: `10.0.20.0`
- Availability Zone: `us-west-1c`

### C) Create The Appserver Subnet 1 and 2
1. Appserver Subnet 1
- Name: `Prod-Appserver-Subnet-1`
- CidirBlock: `10.0.25.0`
- Availability Zone: `us-west-1a`

2. Appserver Subnet 2
- Name: `Prod-Appserver-Subnet-2`
- CidirBlock: `10.0.30.0`
- Availability Zone: `us-west-1c`

### D) Create The Database Subnet 1 and 2
1. Database Subnet 1
- Name: `Prod-db-Subnet-1`
- CidirBlock: `10.0.35.0`
- Availability Zone: `us-west-1a`

2. Database Subnet 2
- Name: `Prod-db-Subnet-2`
- CidirBlock: `10.0.40.0`
- Availability Zone: `us-west-1c`

## STEP 2: Create 2 Public Route Rable and 6 Private Route Tables (Because of NAT Redundancy Implementation)

### A) NAT/ALB Public Subnet 1 Route Table
- Name: `Prod-NAT-ALB-Public-RT-1`
- VPC: Select the `Prod-VPC`

### B) NAT/ALB Public Subnet 2 Route Table
- Name: `Prod-NAT-ALB-Public-RT-2`
- VPC: Select the `Prod-VPC`

### C) Webserver Subnet 1 Route Table
- Name: `Prod-Webserver-RT-1`
- VPC: Select the `Prod-VPC`

### D) Webserver Subnet 2 Route Table
- Name: `Prod-Webserver-RT-2`
- VPC: Select the `Prod-VPC`

### E) Appserver Subnet 1 Table Table
- Name: `Prod-Appserver-RT-1`
- VPC: Select the `Prod-VPC`

### F) Appserver Subnet 2 Table Table
- Name: `Prod-Appserver-RT-2`
- VPC: Select the `Prod-VPC`

### G) Database Subnet 1 Route Table
- Name: `Prod-Database-RT-1`
- VPC: Select the `Prod-VPC`

### H) Database Subnet 2 Route Table
- Name: `Prod-Database-RT-2`
- VPC: Select the `Prod-VPC`

## STEP 3: Associate All Above Route Tables With Their Respective Subnets
1. Associate `Prod-NAT-ALB-Public-RT-1` with `Prod-NAT-ALB-Subnet-1` 
2. Associate `Prod-NAT-ALB-Public-RT-2` with `Prod-NAT-ALB-Subnet-2`
3. Associate `Prod-Webserver-RT-1` with `Prod-Webserver-Subnet-1` 
4. Associate `Prod-Webserver-RT-2` with `Prod-Webserver-Subnet-2`
5. Associate `Prod-Appserver-RT-1` with `Prod-Appserver-Subnet-1` 
6. Associate `Prod-Appserver-RT-2` with  `Prod-Appserver-Subnet-2`
7. Associate `Prod-Database-RT-1` with `Prod-db-Subnet-1` 
8. Associate `Prod-Database-RT-2` with `Prod-db-Subnet-2`

## STEP 4: Create and Configure IGW and NAT Gateways 
### A) Create and Configure IGW to Expose The `NAT/ALB Subnet 1` and `NAT/ALB Subnet 2`
1. Create the Internet Gatway
- Name: `Prod-VPC-IGW`
- VPC: Select the `Prod-VPC` Network

2. Configure/Edit the `Prod-NAT-ALB-Public-RT` Route Table 
- Destination: `0.0.0.0/0`
- Target: Select the `Prod-VPC-IGW`
- `SAVE`

### B) Create and Configure The NAT Gateways to point at the Web, App and Database Tiers/Subnets
1. Create the `First NAT Gateway`
- Name: `Prod-NAT-Gateway-1`
- Elastic IP: Clcik `Allocate Elastic IP`
- Click `Create NAT gateway`

2. Create the `Second NAT Gateway`
- Name: `Prod-NAT-Gateway-2`
- Elastic IP: Clcik `Allocate Elastic IP`
- Click `Create NAT gateway`

### C) Configure/Edit the Route Tables of `Webserver subnets`, `Appserver subnets` and `Database subnets` to Add the `Nat gateway` Configs
### C.1) Update the `Webserver subnet` Route tables (2) with the following configs

1. Select the `Prod-Webserver-RT-1`
- Click on Edit and `Add route`
- Destination: `0.0.0.0/0`
- Target: Select `Prod-NAT-Gateway-1`

2. Select the `Prod-Webserver-RT-2`
- Click on Edit and `Add route`
- Destination: `0.0.0.0/0`
- Target: Select `Prod-NAT-Gateway-2`

### C.2) Update the `Appserver subnet` Route tables (2) with the following configs

1. Select the `Prod-Appserver-RT-1`
- Click on Edit and `Add route`
- Destination: `0.0.0.0/0`
- Target: Select `Prod-NAT-Gateway-1`

2. Select the `Prod-Appserver-RT-2`
- Click on Edit and `Add route`
- Destination: `0.0.0.0/0`
- Target: Select `Prod-NAT-Gateway-2`

### C.3) Update the `Database subnet` Route tables (2) with the following configs

1. Select the `Prod-Database-RT-1`
- Click on Edit and `Add route`
- Destination: `0.0.0.0/0`
- Target: Select `Prod-NAT-Gateway-1`

2. Select the `Prod-Database-RT-2`
- Click on Edit and `Add route`
- Destination: `0.0.0.0/0`
- Target: Select `Prod-NAT-Gateway-2`

## STEP 5: Create Security Groups
### Create the Bastion Host Security Group
- Click on Create Security group
    - Name: `Bastion-Host-Security-Group`
    - Inbound: 
        - Ports: `22`
        - Source: Provide `Your IP or 0.0.0.0/0`

### Create the Frontend/External Load Balancer Security Group
- Navigate to `Security groups`
- Click on Create Security group
    - Name: `Frontend-LB-Security-Group`
    - Inbound: 
        - Ports: `80 and 443`
        - Source: `0.0.0.0/0`

    - Click `Create Security Group`

### Create the Webservers Security Group
- Click on Create Security group
    - Name: `Webservers-Security-Group`
    - Inbound: 
        - Ports: `80 and 443`
            - Source: Provide the `Frontend-LB-Security-Group` ID
        - Ports: `22`
            - Source: `Bastion-Host-Security-Group` ID

### Create the Backend/Internal Load Balancer Security Group
- Click on Create Security group
    - Name: `Backend-LB-Security-Group`
    - Inbound: 
        - Ports: `80 and 443`
        - Source: Provide the `Webservers-Security-Group` ID

### Create the Appservers Security Group
- Click on Create Security group
    - Name: `Appservers-Security-Group`
    - Inbound: 
        - Ports: `80 and 443`
            - Source: Provide the `Backend-LB-Security-Group` ID
        - Ports: `22`
            - Source: `Bastion-Host-Security-Group` ID

### Create the Database Security Group
- Click on Create Security group
    - Name: `Database-Security-Group`
    - Inbound: 
        - Ports: `3306`
            - Source: Provide the `Appservers-Security-Group` ID
        - Ports: `3306`
            - Source: `Bastion-Host-Security-Group` ID

## STEP 6: Create External/Frontend and Internal/Backend Load Balancers
### Create External/Frontend Load Balancer
- Navigate to `EC2/Load Balancers` and Click on `Create Load Balancer`
    - Type: Choose `Application Load Balancer`
    - Load balancer name: `Prod-Frontend-LB`
    - Scheme: `Internet-facing`
    -  IP address type: `IPv4`
    - Network mapping:
        - VPC: Select `Prod-VPC`
        - Mappings: Select the `Prod-NAT-ALB-Subnet-1` and `Prod-NAT-ALB-Subnet-2` frontend subnets
    - Security groups: Select the `Frontend-LB-Security-Group`
    - Listeners and routing: 
        - Click on `Create a target group` to create `HTTP` target group
            - target type: select `instances`
            - Target group name: `Frontend-LB-HTTP-TG`
            - Protocol and Port: `HTTP`:`80`
            - VPC: Select `Prod-VPC`
            - Protocol version: `HTTP1`
            - Health checks: `HTTP`
            - Health check path: `/`
            - Click on `Next`
            - Click on `Create target group`

    - `NOTE:` Navigate back to the page where you're creating the `LoadBalancer` and Refresh (Not The Whole Page)
    - Listeners and routing: 
        - Protocol: `HTTP`
        - Listener `HTTP:80`
        - Select the `Frontend-LB-HTTP-TG`
    
    - Click on `Create load balancer`

### Create Internal/Backend Load Balancer
- Navigate to `EC2/Load Balancers` and Click on `Create Load Balancer`
    - Type: Choose `Application Load Balancer`
    - Load balancer name: `Prod-Backend-LB`
    - Scheme: `Internal`
    -  IP address type: `IPv4`
    - Network mapping:
        - VPC: Select `Prod-VPC`
        - Mappings: Select the `Prod-Webserver-Subnet-1` and `Prod-Webserver-Subnet-2` Webserver/Provate subnets
    - Security groups: Select the `Backend-LB-Security-Group`
    - Listeners and routing: 
        - Click on `Create a target group` to create `HTTP` target group
            - target type: select `instances`
            - Target group name: `Backend-LB-HTTP-TG`
            - Protocol and Port: `HTTP`:`80`
            - VPC: Select `Prod-VPC`
            - Protocol version: `HTTP1`
            - Health checks: `HTTP`
            - Health check path: `/`
            - Click on `Next`
            - Click on `Create target group`

    - `NOTE:` Navigate back to the page where you're creating the `LoadBalancer` and Refresh (Not The Whole Page)
    - Listeners and routing: 
        - Protocol: `HTTP`
        - Listener `HTTP:80`
        - Select the `Backend-LB-HTTP-TG`
    
    - Click on `Create load balancer`

## STEP : Create an S3 Bucket Environment To Upload The Automation and Database Configs
- Navigate to `Amazon S3`
- Click on `Create Bucket`
    - Name: Use naming convention `prod-proxy-app-db-config-YOUR-LAST-NAME-and-TODAY-DATE-TIME`
    - AWS Region: Select your project region `(California) us-west-1`
    - Object Ownership: `ACLs disabled`
    - Block Public Access settings for this bucket: `Enable` `Block all public access`
    - Bucket Versioning: `Enable`
    - Default encryption: `Enable`
    - Click `CREATE BUCKET`

## STEP : Create a Bastion Host VM For Remote Access ((SSH)) To Webservers, Appservers and MySQL Database
- Navigate to Instance in EC2
- Click on `Create Instance`
    - Name: `Prod-Bastion-Host`
    - AMI: `Ubuntu 18.04`
    - Instance type: `t2.micro`
    - Key pair name: Select/Create Key pair
        - Name: `Prod-"YOUR_REGION"-Key`
    - Network:
        - VPC: `Prod-VPC`
        - Subnet: Select either `Prod-NAT-ALB-Subnet-1` or `Prod-NAT-ALB-Subnet-2`
        - Security Group: Select the `Bastion-Host-Security-Group`
    - Click `LAUNCH INSTANCE`

### Setup SSH Port Forwarding Between Your Local and Bastion Host To Point at The Web, App and DB Instance.
- 

### Create an AmazonS3ReadOnlyAccess For Your Web and App Servers
- Navigate to IAM
- Click on Roles and `Create Role`
    - Select `EC2` 
    - Click on `Next`
    - Permissions policies: Assign `AmazonS3ReadOnlyAccess`
    - Click on `Next` 
    - Name: `EC2-AmazonS3ReadOnlyAccess`
    - Click `CREATE`

## STEP 7: Create Webservers and Apservers Launch Templates
### Create Webserver Launch Template
- Naviagte to EC2/Launch Configuration
    - Click on `Create Launch Configuration`
    - Switch by Clicking on `Create launch template`
        - Name: `Prod-Webservers-LT`
        - Template version description: `Prod-Webservers-LT Version 1`
        - AMI: Select for `Ubuntu 18.04`
        - Instance type: `t2.micro`
        - Key pair: Create a new key pair `california-keypair`
        - Network Settings:
            - Subnet: Select one of the Webserver subnets
            - Security groups (Firewalls): Select the `Webservers-Security-Group`
        
        - Expand `Advance details`
            - IAM instance profile: Select `S3-AmazonS3ReadOnlyAccess` IAM Role
            
            - Click on `Create launch template`

### Create Appserver Launch Template
- Naviagte to EC2/Launch Configuration
    - Click on `Create Launch Configuration`
    - Switch by Clicking on `Create launch template`
        - Name: `Prod-Appservers-LT`
        - Template version description: `Prod-Appservers-LT Version 1`
        - AMI: Select for `Ubuntu 18.04`
        - Instance type: `t2.micro`
        - Key pair: Create a new key pair `california-keypair`
        - Network Settings:
            - Subnet: Select one of the Appserver subnets
            - Security groups (Firewalls): Select the `Appservers-Security-Group`
        
        - Expand `Advance details`
            - IAM instance profile: Select `S3-AmazonS3ReadOnlyAccess` IAM Role
            
            - `NOTE:` Update the Database Configuration file `main/appserver-database-config/wp-config.php` on GitHub before passing the User Data
            - Once changes have been made and user data passed 
            - Click on `Create launch template`

## STEP 8: Create Webserver and Appserver Auto Scaling Groups
### A). Webserver Autocsaling Group
- Navigate to `EC2/Auto Scaling`
    - Click on `Create Auto Scaling Group`
        - Auto Scaling group name: `prod-webservers-autoscaling-group`
        - Launch template: Select the `Prod-Webservers-LT`
        - Click on `NEXT`
        - VPC: Select `Prod-VPC`
        - Availability Zones and subnets: Select `Prod-Webserver-Subnet-1` and `Prod-Webserver-Subnet-2`
        - Click `NEXT`
        - Load balancing: Select "Attach to an existing load balancer", select `Frontend-LB-HTTP-TG`
            - Select `Choose from your load balancer target groups`
            - Existing load balancer target groups: Select the `Frontend-LB-HTTP-TG`

        - Health check type: Select `EC2` and `ELB`
        - Click on `NEXT`
        
        - Group size: 
            - Desired: `2`
            - Min: `2`
            - Max: `5`
        
        - Scaling policies: 
            - NOTE: Also Known as Dynamic Scaling. This defines the `Scale Out Policy/Action`
            - Select `Target tracking scaling policy`
                - Scaling policy name: `prod-asg-scale-out-policy`
                - Metric type: Select `Average VPU Utilization`
                - Target value: `80%`
            - Click on `NEXT`
            - Click on `NEXT`
            - Click on `NEXT`
            - Click on `Create Auto Scaling Group`

### B). Appserver Autocsaling Group
- Navigate to `EC2/Auto Scaling`
    - Click on `Create Auto Scaling Group`
        - Auto Scaling group name: `prod-appservers-autoscaling-group`
        - Launch template: Select the `Prod-Appserver-LT`
        - Click on `NEXT`
        - VPC: Select `Prod-VPC`
        - Availability Zones and subnets: Select `Prod-Appserver-Subnet-1` and `Prod-Appserver-Subnet-2`
        - Click `NEXT`
        - Load balancing: Select "Attach to an existing load balancer", select `Backend-LB-HTTP-TG`
            - Select `Choose from your load balancer target groups`
            - Existing load balancer target groups: Select the `Backend-LB-HTTP-TG`

        - Health check type: Select `EC2` and `ELB`
        - Click on `NEXT`
        
        - Group size: 
            - Desired: `2`
            - Min: `2`
            - Max: `5`
        
        - Scaling policies: 
            - NOTE: Also Known as Dynamic Scaling. This defines the `Scale Out Policy/Action`
            - Select `Target tracking scaling policy`
                - Scaling policy name: `prod-asg-scale-out-policy`
                - Metric type: Select `Average VPU Utilization`
                - Target value: `80%`
            - Click on `NEXT`
            - Click on `NEXT`
            - Click on `NEXT`
            - Click on `Create Auto Scaling Group`

## STEP 9: Create a Database Subnet Group and Database Instance (RDS)
### A) Create Databse Subnet Group
- Navigate to the `RDS` Service
- Click on `Subnet groups`
    - Click `Create DB Subnet Group`
    - Name: `prod-db-subnet-group`
    - VPC: Select `Prod-VPC`
    - Availability Zones: Select the two zones you used for this project. Example `us-west-1a` and `us-west-1c`
    - Subnets: Select `Prod-db-Subnet-1` and `Prod-db-Subnet-2`
    - Click on `CREATE`

### B) Create a Database Instance
- Navigate to the `RDS` Service
- Click on `Databases` and `Create Database`
    - Choose a database creation method: Select `Standard create`
    - Engine type: `MySQL`
    - Engine Version: Select the `latest`
    - Templates: `Production`
    - Deployment options: `Multi-AZ DB instance`

    - Databse Settings:
        - DB instance identifier: `prod-database`
        - Master username: `admin`
        - Master password: For example `admin2022`
        - NOTE: Password must be at least 8 characters, Can't contain / , ', " and @
        - DB instance class: Choose `Burstable classes`
            - Select `db.t3.micro`

    - Storage:
        - Storage type: Select `General Purpose SSD (gp3)`
        - Allocated storage: `30`
        - Storage autoscaling: `Enable`
        - Maximum storage threshold: Default `1000`
    
    - Connectivity:
        - Compute resource: Select `Don’t connect to an EC2 compute resource`
        - Virtual private cloud (VPC): `Prod-VPC`
        - DB Subnet group: Select your DB Subnet group `prod-db-subnet-group`
        - Public access: `NO`
            - NOTE: To remote Programatically manually, we'll have to setup a `bastion host`
        - Database authentication: Select `Password authentication`
    - Monitoring:
        - Enhanced Monitoring: `Disable`

    - Additional configuration: 
        - Initial database name: `proddatabase`
        - Backup: `Enable automated backups`
        - Encryption: `Enable encryption`
        - Maintenance: `Disable`
        - Deletion protection: `Disable`
    - Click `CREATE DATABASE`

## STEP 10: Create a Route 53 Hosted Zone and Record For The Frontend Load Balancer Endpoint
