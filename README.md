# Shopwise: Dynamic Website Hosting on AWS

## Overview

Shopwise is a dynamic web application hosted on Amazon Web Services (AWS) that showcases best practices for cloud architecture, deployment, and management. This repository contains the complete guide and scripts for setting up a fault-tolerant, scalable, and secure web application environment on AWS.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation Guide](#installation-guide)
- [Usage](#usage)
- [Commands for Font Changes](#commands-for-font-changes)
- [Cleanup](#cleanup)
- [License](#license)

## Prerequisites

Before you begin, ensure that you have the following:

1. **AWS CLI**: Install and configure the AWS Command Line Interface.
2. **IAM User**: Create an IAM user with the necessary permissions.
3. **Access Keys**: Generate and configure Access and Secret Keys for the IAM user.

## Installation Guide

### Step-by-Step Instructions

1. **Configure AWS CLI**: Run the command:
   ```bash
   aws configure
   ```
   Test the configuration:
   ```bash
   aws iam get-user
   ```

2. **Create VPC**: Set up a Virtual Private Cloud (VPC) with DNS hostname enabled.

3. **Internet Gateway**: Create and attach an Internet Gateway to your VPC.

4. **Subnets**: Create public and private subnets across two Availability Zones for fault tolerance:
   - Public Subnets: For Application Load Balancer (ALB) and NAT Gateway.
   - Private Application Subnets: For EC2 instances.
   - Private Data Subnets: For RDS and Elastic File System (EFS).

5. **Route Tables**: 
   - Public Route Table: Route internet traffic to public subnets through Internet Gateway.
   - Private Route Table: Route traffic through NAT Gateway for private subnets.

6. **NAT Gateway**: Set up a NAT Gateway to enable secure connections from private subnets.

7. **S3 Bucket**: Create an S3 bucket to store application code.

8. **EC2 Instance Connect Endpoint**: Create to facilitate secure EC2 connections.

9. **Security Groups**: Set up security groups to control traffic:
   - **ALB Security Group**: Allow HTTP and HTTPS from the internet.
   - **Webapp Security Group**: Allow SSH access from EICE security group; allow HTTP and HTTPS access from ALB security group.
   - **Database Security Group**: Allow MySQL/Aurora access from the web application.
   - **Data Migrate Security Group**: Allow SSH access from EICE security group.

10. **RDS Instance**: Create a Relational Database Service instance (create your subnet groups first before creating the RDS).

11. **Database Migration**:
    - Create an S3 bucket to store the SQL script.
    - Create an EC2 in the private app subnet for data migration.
    - Use Flyway to migrate SQL scripts from S3 to RDS.
    - Install Flyway on EC2 instance in the private app subnet and execute migration scripts.
     
13. **Application Server (EC2)**: Launch EC2 instances to host the application, with the following setup script:
    ```bash
    #!/bin/bash
    sudo yum update -y
    sudo yum install -y httpd
    sudo systemctl enable httpd
    sudo systemctl start httpd
    # Install PHP and necessary extensions
    sudo dnf install -y php php-pdo php-openssl ...
    # Sync with S3 bucket
    sudo aws s3 sync s3://your-bucket-name /var/www/html
    sudo service httpd restart
    ```

14. **DNS and SSL Configuration**:
    - Register a domain name in Route 53.
    - Create a hosted zone and request an SSL certificate.

15. **Auto Scaling Group**: Create an AMI and configure an Auto Scaling Group for automatic scaling.

16. **Testing**: Verify the application by accessing it via the ALB DNS name.

## Commands for Font Changes

```bash
# Example commands to change fonts in your application
```

(Insert any specific commands related to font changes here)

## Cleanup

To avoid unnecessary charges, ensure you clean up all resources used in this project:

1. **Delete Auto Scaling Group and Launch Template**.
2. **Remove Application Load Balancer and Target Group**.
3. **Terminate RDS Instance** (create a final snapshot first).
4. **Delete VPC, Security Groups, and Subnets**.
5. **Delete Route 53 Record Set** (do not delete the hosted zone).

---

Feel free to customize the README further to match your project's specific requirements!
