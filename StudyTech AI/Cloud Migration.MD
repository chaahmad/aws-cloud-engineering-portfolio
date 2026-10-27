## **Check 1: Cost Analysis**

There's material disprepancy between cost proposed in cloud migration document and my caluculation. The total monthly variance is approxamtely $131.74 and yearly of $1,581. The biggest drivers are VPC endpoint, CloudFront, ECS and ALB.
The following are the comparison between the proposal and my calculation for services with variances (yearly:
 - ALB: $232.32 proposal vs $305.40 recalcualtion. The reason is that proposal only included flat hourly charges without specifying any load balancer capacity units, whereas I allocated 1 GB per hour for total data processed per ALB for targets.
 - VPC Endpoint: $0 in proposal vs $867.36 recalculation. In the proposal, it stated that compute layer will interact with Secrets Manager, ECR, Amazon Bedrock and S3 via VPC endpoint. There are two types of VPC endpotings: gateway endpoint and interface endpoint. The difference between the two is that gateway is free whereas interface cost money. However, only S3 and DynamoDB are free to use whereas other services get charged. We have three servies at interface gateway across two availability zones in production environment, but only one availbility zone in preproduction environment. Thus, the total cost is $867.36
 - CloudFront: $0 in proposal vs $180 recalculation. In the proposal, it stated that CloudFront sits in the front of the application for faster delivery and caching statis/edge content, but it omitted the cost. I selected pro plan for $15 per month or $180 per year, which should be adequate for a startup.
 - Fargate in pre-production environment: $2.94 in proposal vs $88.08 in recalculation. I'm unable to calculate how it came to $2.94. The closest I get is $88.08 for ARM, 1 task per day, 2 GB per month with 0.5 vCPU that's running for 9 hours. 

The following two tables contain the costs calculated by me, along with the costs stated in the proposal and variances:

### Production

| Service | Details | Monthly | Yearly | Proposal-Yearly | Variance |
| --- | --- | ---: | ---: | ---: | ---: |
| RDS (PostgreSQL Private subnet in both Azs) | db.t4g.small for multi-az | $57.88 | $694.56 | $718.56 | ($24) |
| ECS (ASG) Fargate | 2 tasks per day, running at 24 hours (0.5 CPU) | $39.13 | $469.56 | $469.56 | $0 |
| **ALB in both subnet** | **1 GB per hour of data processed per ALB** | **$25.45** | **$305.40** | **$232.32** | **$73.08** |
| FCK-NAT (t4g.nano) | $0.0042 per hour for t4g.nano. We have 2 FCK-NAT | $6.12 | $73.44 | $52.21 | $21.23 |
| ECR | 10 GB per month | $1 | $12 | $12 | $0 |
| Secrets Manager | 2 secrets | $0.80 | $9.60 | $9.60 | $0 |
| Route 53 | 1 hosted zones | $0.50 | $6 | $6.12 | $0.12 |
| Public IP address | 1 IP address | $3.65 | $43.80 | $43.80 | $0 |
| **VPC Endpoint for Secrets Manager, ECR and Amazon Bedrock** | **3 endpoints across 2 AZ** | **$48.18** | **$578.16** | **$0** | **$578.16** |
| **CloudFront** | **1 pro plan** | **$15** | **$180** | **$0** | **$180** |

### Pre-Prod

| Service | Details | Monthly | Yearly | Proposal-Yearly | Variance |
| --- | --- | ---: | ---: | ---: | ---: |
| RDS PostgreSQL | T4g.micro, 2 for Dev and Pre-Pod, Single-AZ, 20 GB | $15.18 | $182.10 | $182.10 | $0 |
| **ECS Fargate** | **ARM, 1 tasks per day, 2 GB/month , 9 hours, 0.5 vCPU** | **$7.34** | **$88.08** | **$2.94** | **$85.14** |
| Secret Manager | 2 secrets; 1 day | $0.03 | $0.36 | $9.60 | ($9.24) |
| ECR | 5 GB storage per month | $0.50 | $6 | $6 | $0 |
| **VPC Endpoint for Secrets Manager, ECR and Amazon Bedrock** | **3 endpoints in one AZ** | **$24.10** | **$289.20** | **$0** | **$289.20** |

## **Check 2: Security**

While the proposal has certain security in place, such as a VPC endpoint, network segmentation between public and private subnets, and security groups, I believe it should also have NACLs. NACLs would serve as an additional layer of security as part of a defence-in-depth mechanism. By default, NACLs allow all inbound and outbound traffic, and we should implement NACLs for HTTPS traffic.

## **Check 3: Backup**

While the proposal contain disaster recovery plan for production and pre-prod/dev environment, it's missing a backup and rollout plan before it can begin any migration, which include pre-prod/dev environment. The proposal contain five phases for smooth migration, but there's could be unforeseeable problems within the migration that can negatively impact end user experience. I'm concern about end user experience ff the migration implementation doesn't go smoothly and it will consume a lot of time in resolving migration related issues. Thereby, I believe it's crucial to have backup and rollback plan and AWS Backup service can assist with backup configuration.
