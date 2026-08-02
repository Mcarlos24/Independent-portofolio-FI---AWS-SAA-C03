# AWS SAA-C03 Portfolio - Multi-AZ Landing Zone

Cloud infrastructure project designed to demonstrate AWS Solutions Architect Associate level skills, with a secure, resilient, and extensible platform baseline.

**Revision 2 — 2 August 2026.** 45 resources, validated with `cfn-lint` (0 findings). See the CHANGELOG at the end of the template for what changed and why.

## Objective

This project deploys a production-oriented AWS landing zone, including Multi-AZ networking with NAT egress, encrypted storage, a managed PostgreSQL database whose credentials never leave AWS Secrets Manager, and an Amazon EKS foundation.

It can be used as a portfolio project to demonstrate the following topics:

- Highly available AWS architecture design across 3 Availability Zones
- Infrastructure as Code with AWS CloudFormation, including conditions and cross-stack exports
- Public and private network segmentation, with controlled outbound egress
- Encryption with AWS KMS under an explicit key policy
- Terraform state storage with S3 (Object Lock) and DynamoDB
- Multi-AZ PostgreSQL database with Amazon RDS and Secrets Manager credential management
- Kubernetes foundation with Amazon EKS, control plane logging and envelope encryption of secrets
- Cloud security and least privilege principles

## AWS Services Used

- Amazon VPC — public and private subnets across 3 Availability Zones
- Internet Gateway, NAT Gateways, per-AZ route tables
- Gateway VPC Endpoints for S3 and DynamoDB
- Amazon S3 with KMS encryption, versioning, Object Lock and a TLS-only bucket policy
- Amazon DynamoDB for Terraform locking, with point-in-time recovery
- AWS KMS for centralized encryption, with rotation and an explicit key policy
- AWS Secrets Manager for the database master credentials
- Amazon RDS PostgreSQL Multi-AZ
- Amazon EKS with a managed node group
- Amazon CloudWatch Logs
- AWS IAM, Security Groups

## Project Files

- `AWS_SAA-C03_Landing_Zone.yaml`: source CloudFormation template.
- `AWS_SAA-C03_Technical_Documentation.md`: technical project documentation.
- `README.md`: general project overview.

## Architecture

The template creates an infrastructure composed of:

- A dedicated VPC with configurable CIDR
- Three public subnets and three private subnets, one pair per Availability Zone
- A public route table with Internet access through an Internet Gateway
- **One or three NAT Gateways** (see `NatGatewayStrategy`) and **one private route table per AZ**, giving private workloads outbound egress
- **Gateway VPC Endpoints for S3 and DynamoDB**, so state traffic bypasses the NAT
- An encrypted S3 bucket for Terraform state or immutable backups, with Object Lock in GOVERNANCE mode (30 days) and lifecycle rules
- A DynamoDB table for Terraform state locking, encrypted with the platform key
- A KMS key with rotation enabled and an explicit key policy
- An encrypted Multi-AZ RDS PostgreSQL database whose master password is generated and stored in Secrets Manager
- An EKS cluster with control plane logging and envelope encryption of Kubernetes secrets
- An EKS managed node group of 2 to 4 nodes, in private subnets, reachable through SSM Session Manager

## Main Parameters

| Parameter                | Description                                                                       | Default Value           |
| ------------------------ | --------------------------------------------------------------------------------- | ----------------------- |
| `ProjectName`          | Logical project name (lowercase, used as a resource prefix)                       | `dora-cloud-platform` |
| `Environment`          | Target environment:`dev`, `test` or `prod`                                  | `dev`                 |
| `VpcCidr`              | VPC CIDR block                                                                    | `10.0.0.0/16`         |
| `NatGatewayStrategy`   | `single` (one NAT, lower cost) or `per-az` (three NATs, recommended for prod) | `single`              |
| `DBUsername`           | PostgreSQL administrator username                                                 | `postgresadmin`       |
| `DBInstanceClass`      | RDS instance class                                                                | `db.t3.micro`         |
| `EksVersion`           | EKS control plane version — check supported versions before deploying            | `1.31`                |
| `EksPublicAccessCidrs` | CIDRs allowed to reach the Kubernetes API public endpoint                         | `0.0.0.0/0`           |
| `NodeInstanceType`     | Worker node instance type                                                         | `t3.medium`           |

**There is no password parameter.** RDS generates the master password and stores it in Secrets Manager, encrypted with the platform KMS key. Retrieve it from the `RdsMasterSecretArn` output.

## Deployment

AWS CLI deployment example:

```bash
aws cloudformation deploy \
  --template-file "AWS_SAA-C03_Landing_Zone.yaml" \
  --stack-name dora-cloud-platform-dev \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameter-overrides \
    Environment=dev \
    NatGatewayStrategy=single
```

Production-style deployment, with a highly available NAT tier and a restricted Kubernetes API endpoint:

```bash
aws cloudformation deploy \
  --template-file "AWS_SAA-C03_Landing_Zone.yaml" \
  --stack-name dora-cloud-platform-prod \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameter-overrides \
    Environment=prod \
    NatGatewayStrategy=per-az \
    EksPublicAccessCidrs="203.0.113.0/24"
```

Retrieving the generated database credentials:

```bash
SECRET_ARN=$(aws cloudformation describe-stacks \
  --stack-name dora-cloud-platform-dev \
  --query "Stacks[0].Outputs[?OutputKey=='RdsMasterSecretArn'].OutputValue" --output text)

aws secretsmanager get-secret-value --secret-id "$SECRET_ARN" --query SecretString --output text
```

Linting before deployment:

```bash
pip install cfn-lint
cfn-lint AWS_SAA-C03_Landing_Zone.yaml
```

## Best Practices Demonstrated

- Data-at-rest encryption with a customer-managed KMS key, rotation enabled, under an explicit key policy
- Database credentials generated and held by Secrets Manager — no password in the template, the parameters or the stack events
- Private subnets for application workloads, with egress through NAT rather than public IPs
- PostgreSQL reachable only from the EKS security group; database egress reduced to nothing
- RDS not publicly exposed, Multi-AZ, backup retention, deletion protection, snapshot on delete
- S3 public access fully blocked, versioning, Object Lock with a default retention rule, TLS-only bucket policy
- DynamoDB point-in-time recovery
- EKS control plane logging on all five log types, envelope encryption of Kubernetes secrets, restrictable public endpoint
- SSM Session Manager on worker nodes instead of SSH and an open port 22
- `DeletionPolicy: Retain` on stateful resources, so a stack deletion cannot destroy state or backups
- Consistent `Project` / `Environment` tagging, cross-stack exports on every shared identifier

## Known Limitations

Deliberately out of scope, and worth discussing rather than hiding:

- No interface VPC Endpoints for ECR, STS or CloudWatch: they carry an hourly charge per AZ and only pay off once NAT egress is removed altogether.
- Node IAM relies on AWS managed policies rather than scoped ones; a production estate would use IRSA or EKS Pod Identity per workload.
- No WAF, GuardDuty, AWS Config or CloudTrail: these belong to an account-level baseline, not to a workload stack.
- No `taskcat` deployment test — `cfn-lint` is run, an actual deployment test is not automated.
- Multi-AZ RDS is always on, including in `dev`, to keep the architecture honest with what this documentation claims.

## Author

Charles Monteiro
Cloud Engineer / AWS Solutions Architect Associate Portfolio
