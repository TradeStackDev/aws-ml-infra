# ☁️ aws-ml-infra

> **Terraform + CDK modules for deploying ML training jobs on AWS ECS Fargate with auto-scaling and S3 artifact management.**

[![Python](https://img.shields.io/badge/Python-3.11+-3776ab?style=flat&logo=python&logoColor=white)](https://python.org)
[![Terraform](https://img.shields.io/badge/Terraform-1.7+-7B42BC?style=flat&logo=terraform&logoColor=white)](https://terraform.io)
[![AWS](https://img.shields.io/badge/AWS-ECS%20Fargate-FF9900?style=flat&logo=amazonaws&logoColor=white)](https://aws.amazon.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## Overview

Reusable Terraform and AWS CDK modules for running ML workloads on AWS without managing servers. Spin up a training job, let it run, store artifacts in S3, and have it scale down automatically when done.

Designed for AI agent training pipelines — specifically for running `webenv-sim` data collection jobs at scale.

---

## Features

- 🐳 ECS Fargate tasks — serverless containers, no EC2 to manage
- 💰 Spot instance support — up to 70% cost savings for fault-tolerant jobs
- ⚖️ Auto-scaling — scale task count based on SQS queue depth
- 📦 S3 artifact management — automatic upload of model checkpoints and datasets
- 🔐 IAM roles with least-privilege permissions
- 📊 CloudWatch dashboards and alarms
- 🔔 SNS alerts on job failure or completion

---

## Modules

### `ecs-training-job`
Runs a containerized ML training job as an ECS Fargate task.

### `s3-artifact-store`
S3 bucket with lifecycle policies, versioning, and IAM access control.

### `auto-scaling-worker`
ECS service that scales based on SQS queue depth — for parallel data collection.

### `vpc-networking`
VPC, subnets, security groups, and NAT gateway for private ECS tasks.

---

## Quick Start

```bash
git clone https://github.com/TradeStackDev/aws-ml-infra.git
cd aws-ml-infra/terraform

# Initialize
terraform init

# Review plan
terraform plan -var-file=environments/dev.tfvars

# Deploy
terraform apply -var-file=environments/dev.tfvars
```

---

## Example: Run a Training Job

```python
import boto3

ecs = boto3.client('ecs', region_name='us-east-1')

response = ecs.run_task(
    cluster='ml-training-cluster',
    taskDefinition='webenv-sim-collector:latest',
    launchType='FARGATE',
    networkConfiguration={
        'awsvpcConfiguration': {
            'subnets': ['subnet-xxx'],
            'securityGroups': ['sg-xxx'],
            'assignPublicIp': 'DISABLED'
        }
    },
    overrides={
        'containerOverrides': [{
            'name': 'collector',
            'environment': [
                {'name': 'TASK_SUITE', 'value': 'ecommerce-v1'},
                {'name': 'NUM_EPISODES', 'value': '500'},
                {'name': 'S3_OUTPUT_BUCKET', 'value': 'ml-artifacts-prod'},
            ]
        }]
    }
)
```

---

## Terraform Variables

```hcl
# environments/prod.tfvars
aws_region         = "us-east-1"
cluster_name       = "ml-training-cluster"
task_cpu           = 2048   # 2 vCPU
task_memory        = 4096   # 4 GB
use_spot           = true
s3_bucket_name     = "ml-artifacts-prod"
max_task_count     = 20
alert_email        = "hello@tradestackdev.com"
```

---

## Cost Optimization

| Mode | Cost | Use Case |
|------|------|----------|
| Fargate On-Demand | ~$0.04/hr per task | Time-sensitive jobs |
| Fargate Spot | ~$0.012/hr per task | Fault-tolerant batch jobs |
| Auto-scaled pool | Variable | High-throughput data collection |

---

## Requirements

- Terraform 1.7+
- AWS CLI configured with appropriate permissions
- Docker (for building training images)
- Python 3.11+ (for CDK modules)

---

## License

MIT © Adam Johnson
