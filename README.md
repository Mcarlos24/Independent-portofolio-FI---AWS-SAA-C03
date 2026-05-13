# AWS SAA-C03 Portfolio - Multi-AZ Landing Zone

Cloud infrastructure project designed to demonstrate AWS Solutions Architect Associate level skills, with a secure, resilient, and extensible platform baseline.

## Objective

This project deploys a production-oriented AWS landing zone, including Multi-AZ networking, encrypted storage, a managed PostgreSQL database, and an Amazon EKS foundation.

It can be used as a portfolio project to demonstrate the following topics:

- Highly available AWS architecture design
- Infrastructure as Code with AWS CloudFormation
- Public and private network segmentation
- Encryption with AWS KMS
- Terraform state storage with S3 and DynamoDB
- Multi-AZ PostgreSQL database with Amazon RDS
- Kubernetes foundation with Amazon EKS
- Cloud security and least privilege principles

## AWS Services Used

- Amazon VPC
- Public and private subnets across 3 Availability Zones
- Internet Gateway and route tables
- Amazon S3 with KMS encryption, versioning, and Object Lock
- Amazon DynamoDB for Terraform locking
- AWS KMS for centralized encryption
- Amazon RDS PostgreSQL Multi-AZ
- Amazon EKS
- AWS IAM
- Security Groups

## Project Files

- `Independent Portofolio FI - Projet AWS SAA-C03.txt`: source CloudFormation template.
- `AWS_SAA_C03_Portfolio_Project.md`: technical project documentation.
- `README.md`: general project overview.

## Architecture

The template creates an infrastructure composed of:

- A dedicated VPC with configurable CIDR
- Three public subnets
- Three private subnets
- A public route table with Internet access
- An encrypted S3 bucket for Terraform state or immutable backups
- A DynamoDB table for Terraform state locking
- A KMS key with rotation enabled
- An encrypted Multi-AZ RDS PostgreSQL database
- An EKS cluster deployed in private subnets
- An EKS node group sized from 2 to 4 nodes

## Main Parameters

| Parameter | Description | Default Value |
| --- | --- | --- |
| `ProjectName` | Logical project name | `dora-cloud-platform` |
| `Environment` | Target environment | `dev` |
| `VpcCidr` | VPC CIDR block | `10.0.0.0/16` |
| `DBUsername` | PostgreSQL administrator username | `postgresadmin` |
| `DBPassword` | PostgreSQL password | Provided at deployment |

## Deployment

AWS CLI deployment example:

```bash
aws cloudformation deploy \
  --template-file "Independent Portofolio FI - Projet AWS SAA-C03.txt" \
  --stack-name dora-cloud-platform-dev \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameter-overrides \
    Environment=dev \
    DBPassword="ChangeMeWithAStrongPassword123!"
```

## Best Practices Demonstrated

- Data-at-rest encryption with KMS
- KMS key rotation enabled
- Private subnets for application workloads
- PostgreSQL access restricted to the EKS security group
- RDS not publicly exposed
- RDS backup retention configured
- Deletion protection enabled for the database
- S3 bucket public access fully blocked
- Versioning and Object Lock enabled on S3

## Possible Improvements

- Add NAT Gateways for outbound Internet access from private subnets
- Add VPC Endpoints for S3, DynamoDB, ECR, and CloudWatch
- Add CloudWatch Logs for EKS and RDS
- Parameterize the EKS version
- Add AWS Secrets Manager for the PostgreSQL password
- Add more granular IAM policies
- Add `cfn-lint` and `taskcat` tests
- Convert the source template to a `.yaml` file

## Author

C.Monteiro  
Cloud Engineer / AWS Solutions Architect Associate Portfolio

