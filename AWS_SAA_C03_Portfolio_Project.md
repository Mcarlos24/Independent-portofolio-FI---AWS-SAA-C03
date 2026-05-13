# Technical Documentation - AWS SAA-C03 Project

## Summary

This project defines an AWS infrastructure using AWS CloudFormation. The goal is to build a secure Multi-AZ cloud baseline including networking, encryption, storage, database, and container orchestration.

The template is intended as a practical portfolio demonstration for a Cloud Engineer or AWS Solutions Architect Associate profile.

## Architecture Components

### 1. KMS Encryption

The project creates a dedicated AWS KMS key for the platform:

- Key rotation enabled
- Alias based on the project name and environment
- Used by S3, RDS, and EKS secrets encryption

### 2. S3 Storage and DynamoDB Locking

The S3 bucket is designed to store Terraform state or immutable backups:

- Server-side encryption with KMS
- Versioning enabled
- Object Lock enabled
- Full public access blocking

The DynamoDB table uses a `LockID` key and `PAY_PER_REQUEST` billing mode, which matches the standard Terraform locking model.

### 3. Multi-AZ VPC Network

The template provisions a configurable VPC with:

- Default CIDR `10.0.0.0/16`
- DNS support and DNS hostnames enabled
- 3 public subnets
- 3 private subnets
- Internet Gateway
- Public route table to `0.0.0.0/0`

The public subnets use:

- `10.0.1.0/24`
- `10.0.2.0/24`
- `10.0.3.0/24`

The private subnets use:

- `10.0.11.0/24`
- `10.0.12.0/24`
- `10.0.13.0/24`

### 4. Security Groups

Two security groups are defined:

- `EksSecurityGroup` for the EKS cluster
- `DatabaseSecurityGroup` for PostgreSQL

The PostgreSQL security group only allows TCP `5432` traffic from the EKS security group.

### 5. RDS PostgreSQL

The database is configured with:

- PostgreSQL engine
- `db.t3.micro` instance
- 20 GB of storage
- KMS encryption
- Multi-AZ enabled
- Private subnet group
- Public access disabled
- 7-day backup retention
- Deletion protection enabled

### 6. IAM for EKS

The template creates two IAM roles:

- EKS cluster role with `AmazonEKSClusterPolicy`
- EKS node group role with:
  - `AmazonEKSWorkerNodePolicy`
  - `AmazonEC2ContainerRegistryReadOnly`
  - `AmazonEKS_CNI_Policy`

### 7. Amazon EKS

The EKS cluster is deployed in private subnets with:

- Kubernetes version `1.29`
- Private endpoint enabled
- Public endpoint enabled
- Kubernetes secrets encrypted with KMS
- Node group using `t3.medium` instances
- Desired capacity of 2 nodes
- Minimum capacity of 2 nodes
- Maximum capacity of 4 nodes

## CloudFormation Outputs

The stack exposes the following values:

| Output | Description |
| --- | --- |
| `VpcId` | ID of the created VPC |
| `EksClusterName` | Name of the EKS cluster |
| `RdsEndpoint` | PostgreSQL endpoint |
| `StateBucketName` | Name of the S3 bucket |
| `LockTableName` | Name of the DynamoDB table |

## Deployment Command

```bash
aws cloudformation deploy \
  --template-file "Independent Portofolio FI - Projet AWS SAA-C03.txt" \
  --stack-name dora-cloud-platform-dev \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameter-overrides \
    ProjectName=dora-cloud-platform \
    Environment=dev \
    VpcCidr=10.0.0.0/16 \
    DBUsername=postgresadmin \
    DBPassword="ChangeMeWithAStrongPassword123!"
```

## Post-Deployment Verification

```bash
aws cloudformation describe-stacks \
  --stack-name dora-cloud-platform-dev
```

```bash
aws eks update-kubeconfig \
  --name dora-cloud-platform-dev-eks
```

```bash
kubectl get nodes
```

## Security Notes

- The PostgreSQL password must be provided at deployment time and must never be committed.
- For a production version, AWS Secrets Manager or SSM Parameter Store is recommended.
- Private subnets do not have a NAT Gateway in this version.
- The EKS public endpoint is enabled; for a strictly private environment, it can be disabled.
- IAM policies can be reduced according to the exact workload requirements.

## Recommended Improvements

- Rename the source template with the `.yaml` extension
- Add `cfn-lint` to a CI pipeline
- Add NAT Gateways or VPC Endpoints depending on outbound Internet requirements
- Add CloudWatch Container Insights
- Add AWS Load Balancer Controller for EKS
- Add External Secrets Operator or Secrets Store CSI Driver
- Add a clear separation between `dev`, `test`, and `prod` environments
- Add standardized tags to all resources

