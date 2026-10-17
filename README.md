# Shopwise: Dynamic Website Hosting on AWS

## Overview

Shopwise is a dynamic web application deployment project focused on hosting a web application on Amazon Web Services (AWS). It explores cloud infrastructure design, database connectivity, application deployment, security controls, and scalability.

The deployment guide covers a multi-tier architecture using Amazon VPC, EC2, RDS, S3, an Application Load Balancer, and Auto Scaling. It also describes database migration with Flyway and DNS and SSL configuration.

**Project focus:** AWS infrastructure, cloud-based web hosting, database migration, load balancing, and scalable application deployment.

## Architecture and AWS Services

The documented architecture uses the following AWS services:

| Service | Purpose |
|---|---|
| Amazon VPC | Isolated network environment for project resources |
| Public and private subnets | Separate internet-facing and internal resources |
| Internet Gateway | Internet connectivity for public subnets |
| NAT Gateway | Outbound internet access for eligible private-subnet resources |
| Application Load Balancer | Distributes incoming application traffic |
| Amazon EC2 | Hosts the web application and migration utilities |
| Amazon RDS | Managed relational database |
| Amazon S3 | Stores application files and migration scripts |
| EC2 Instance Connect Endpoint | Provides a private connectivity option for EC2 instances |
| Auto Scaling | Adjusts application capacity according to configured policies |
| Amazon Route 53 | DNS and domain management |
| AWS Certificate Manager | SSL/TLS certificate management |
| Flyway | Applies database migration scripts |

The intended network design places internet-facing resources in public subnets and application and database resources in private subnets. Review the actual AWS configuration before treating this as a deployed or production-ready architecture.

## Prerequisites

Before beginning, prepare the following:

- An AWS account with appropriate permissions.
- AWS CLI installed and configured.
- Basic familiarity with AWS networking, EC2, security groups, and IAM.
- An application package and the required database migration scripts.
- A domain name if you plan to configure custom DNS.
- An understanding of the AWS costs associated with the resources you create.

## Deployment Guide

### 1. Configure the AWS CLI

Configure your AWS CLI credentials using an approved authentication method:

```bash
aws configure
```

Verify the active AWS identity:

```bash
aws sts get-caller-identity
```

Prefer IAM roles, IAM Identity Center, or other temporary credentials where practical. Do not commit access keys, secret keys, passwords, or other credentials to this repository.

### 2. Create the Network

Create a VPC and enable DNS support and DNS hostnames as required by your design.

Plan subnets across two Availability Zones:

- **Public subnets:** For the Application Load Balancer and NAT Gateway resources.
- **Private application subnets:** For the web application EC2 instances.
- **Private data subnets:** For RDS and any required data services, such as EFS.

Configure an Internet Gateway, route tables, and routes appropriate to each subnet. Private-subnet internet access should be limited to what the workload requires.

### 3. Configure Security Groups

Create security groups with least-privilege rules appropriate to your architecture.

- **Load balancer:** Permit the required HTTP/HTTPS traffic from intended clients.
- **Application servers:** Permit application traffic from the load balancer and administrative access through an approved management path.
- **Database:** Permit the database port only from the application security group.
- **Migration host:** Permit administrative access only through the approved management path.

Avoid exposing SSH or database ports directly to the public internet.

### 4. Prepare Amazon S3

Create an S3 bucket for application artifacts and, if needed, a separate bucket for database migration scripts.

Use appropriate IAM permissions and bucket security settings. Replace example bucket names in commands with your own values.

### 5. Create the Database

Create an RDS instance and configure its DB subnet group before launching it.

Configure database access, credentials, backups, and connectivity according to your application requirements. Store secrets securely rather than hard-coding them into scripts or source files.

### 6. Run Database Migrations

The documented workflow uses Flyway to apply SQL migration scripts to RDS.

A typical workflow is:

1. Store the migration scripts in S3.
2. Prepare a migration host in an appropriate private subnet.
3. Install and configure Flyway on the host.
4. Configure secure database connectivity.
5. Run the migrations and verify their results.

The exact Flyway commands and configuration depend on the migration files, database engine, and Flyway version.

### 7. Deploy the Application to EC2

Launch an EC2 instance or configure a launch template for your application servers.

The following is an illustrative Apache installation and application-sync example for a compatible Linux environment. Adapt the package installation commands to the selected AMI, install all required application dependencies, and configure appropriate IAM permissions before running it.

```bash
#!/bin/bash
set -e

# Install and start Apache on a compatible Linux image.
sudo yum update -y
sudo yum install -y httpd
sudo systemctl enable --now httpd

# Replace the example bucket with your application artifact bucket.
sudo aws s3 sync s3://your-bucket-name/ /var/www/html/

sudo systemctl restart httpd
```

This example does not install PHP or configure database connectivity. Add the required runtime packages, application configuration, permissions, health checks, and secrets handling for your actual application.

### 8. Configure Load Balancing and Auto Scaling

Create a target group and Application Load Balancer. Configure health checks and listener rules for the application.

Create an AMI or launch template as appropriate, then configure an Auto Scaling Group across the intended Availability Zones. Test instance replacement, application health, and scaling behavior before relying on the setup.

### 9. Configure DNS and HTTPS

If using a custom domain:

1. Configure the domain and Route 53 DNS records.
2. Request or import an appropriate SSL/TLS certificate, for example through AWS Certificate Manager.
3. Validate the certificate and configure the HTTPS listener.
4. Test the application using the configured domain.

### 10. Test the Deployment

Verify the following:

- The application is reachable through the intended load balancer or domain.
- Application health checks pass.
- The application can connect to RDS.
- Database migrations complete successfully.
- Security group rules restrict access as intended.
- Auto Scaling and instance replacement behave as expected.
- HTTPS works if configured.

## Repository Contents

This README describes the documented deployment workflow. Refer to the repository's actual files and scripts for the implementation details and configuration required for your environment.

## Cleanup and Cost Management

AWS resources may continue to incur charges even when you are not actively using the application. After testing, review the resources you created and remove those you no longer need.

A cleanup checklist may include:

1. Delete the Auto Scaling Group and associated launch templates or configurations that are no longer needed.
2. Remove the Application Load Balancer and target group.
3. Back up required data, then delete the RDS instance if it is no longer needed.
4. Remove EC2 instances, NAT Gateways, and other billable resources that are no longer required.
5. Remove unused S3 objects and buckets after verifying that no required data remains.
6. Delete project-specific networking resources after dependent resources have been removed.
7. Review Route 53 records, hosted zones, and domain registrations separately before removing them.

Resource dependencies and retained snapshots, backups, Elastic IP addresses, and domain registrations may affect charges. Verify the impact of each deletion before proceeding.

## Security Notes

- Never commit AWS credentials, database passwords, private keys, or secret configuration files.
- Use least-privilege IAM permissions and restrict network access.
- Review scripts and commands before executing them in an AWS account.
- Test configuration changes in a non-production environment first.

## License

Add a license section only if the repository contains a license file or you have selected and documented the applicable license.
