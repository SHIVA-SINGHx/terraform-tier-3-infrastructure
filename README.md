# 3-Tier AWS Infrastructure with Terraform

![Terraform](https://img.shields.io/badge/Terraform-IaC-7B42BC?logo=terraform&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-Multi--AZ-FF9900?logo=amazonaws&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Containerized-2496ED?logo=docker&logoColor=white)
![Go](https://img.shields.io/badge/Go-Backend-00ADD8?logo=go&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-Frontend-339933?logo=nodedotjs&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-RDS-4169E1?logo=postgresql&logoColor=white)

A highly available 3-tier web application deployed on AWS using Terraform. The frontend runs on Node.js, the backend on Go, and data is stored in PostgreSQL (Amazon RDS). The entire infrastructure is created with a single `terraform apply`.

![Architecture Diagram](AWS%20Terrafrom%20Multi-AZ%20Cloud%20Architecture%20Diagram.png)

---

## Highlights

- **Multi-AZ architecture:** VPC with 8 subnets across 2 Availability Zones (public, frontend, backend, database)
- **Load balanced:** public ALB for internet traffic, internal ALB for frontend-to-backend communication
- **Auto Scaling:** frontend scales between 2–4 instances, backend between 2–6
- **Managed database:** RDS PostgreSQL with optional Multi-AZ failover
- **Secure by design:** private subnets, bastion host for SSH, credentials stored in AWS Secrets Manager
- **Observable:** application logs shipped to CloudWatch
- **Flexible cost/availability:** toggle between high-availability and cost-optimized setups using variables

---

## Architecture

| Layer | Component |
|-------|-----------|
| Network | VPC, 8 subnets across 2 AZs, NAT Gateway (1 or 2) |
| Entry | Public Application Load Balancer |
| Frontend | Node.js containers on EC2 Auto Scaling Group (2–4) |
| Backend | Go containers on EC2 Auto Scaling Group (2–6), behind an internal ALB |
| Database | RDS PostgreSQL (Multi-AZ optional) |
| Access | Bastion host for SSH into private instances |
| Secrets | AWS Secrets Manager |
| Logging | Amazon CloudWatch |

For a deeper walkthrough, see [ARCHITECTURE.md](terraform-infra/ARCHITECTURE.md).

---

## Project Structure

```
.
├── frontend/                 # Node.js frontend + Dockerfile
├── backend/                  # Go backend + Dockerfile
├── docker-local-deployment/  # docker-compose setup for local development
└── terraform-infra/          # Terraform code (modules + environments)
    └── environments/dev/
```

---

## Prerequisites

- AWS account with credentials configured (`aws configure`)
- [Terraform](https://developer.hashicorp.com/terraform/install)
- [Docker](https://docs.docker.com/get-docker/) and a Docker Hub account
- An existing EC2 key pair in your target region

---

## Quick Start

### 1. Build and push Docker images

```bash
docker login

cd frontend
docker build -t YOUR_USERNAME/goal-tracker-frontend:latest .
docker push YOUR_USERNAME/goal-tracker-frontend:latest

cd ../backend
docker build -t YOUR_USERNAME/goal-tracker-backend:latest .
docker push YOUR_USERNAME/goal-tracker-backend:latest
```

### 2. Configure Terraform

```bash
cd terraform-infra/environments/dev
cp terraform.tfvars.example terraform.tfvars
```

Edit `terraform.tfvars`:

```hcl
region      = "us-east-1"
environment = "dev"
project     = "goal-tracker"

ssh_key_name     = "YOUR_KEY_NAME"
allowed_ssh_cidr = "YOUR_IP/32"

frontend_docker_image = "YOUR_USERNAME/goal-tracker-frontend:latest"
backend_docker_image  = "YOUR_USERNAME/goal-tracker-backend:latest"
dockerhub_username    = "YOUR_USERNAME"
dockerhub_password    = "YOUR_DOCKER_TOKEN"

# High availability options
single_nat_gateway = false  # true = cost optimized, false = HA
db_multi_az        = true   # true = HA, false = cost optimized
```

> **Note:** `terraform.tfvars` contains credentials. Never commit it to Git.

### 3. Deploy

```bash
terraform init
terraform plan
terraform apply -auto-approve
```

Deployment takes around 15–20 minutes.

### 4. Access the application

```bash
terraform output application_url
terraform output bastion_public_ip
```

---

## Local Development

Run the full stack locally without AWS:

```bash
cd docker-local-deployment
docker-compose up -d
```

Open [http://localhost:3000](http://localhost:3000).

---

## Updating the Application

After pushing new images, trigger a rolling refresh of the Auto Scaling Group:

```bash
aws autoscaling start-instance-refresh \
  --auto-scaling-group-name dev-goal-tracker-frontend-asg \
  --region us-east-1
```

---

## Troubleshooting

**View logs**
```bash
aws logs tail /aws/ec2/dev-goal-tracker/frontend --follow --region us-east-1
```

**SSH into private instances via bastion**
```bash
ssh -i your-key.pem ec2-user@BASTION_IP
ssh ec2-user@PRIVATE_INSTANCE_IP
```

**Check containers**
```bash
docker ps -a
docker logs goal-tracker-frontend
```

---

## Cost Note

NAT Gateways, Multi-AZ RDS, and multiple EC2 instances are billed while running. For testing, set `single_nat_gateway = true` and `db_multi_az = false`, and destroy resources when done.

---

## Cleanup

```bash
terraform destroy -auto-approve
```

---

## Tech Stack

Terraform · AWS (VPC, EC2, ALB, Auto Scaling, RDS, Secrets Manager, CloudWatch) · Docker · Node.js · Go · PostgreSQL

---

## Author

**Shiva Singh**
[GitHub](https://github.com/SHIVA-SINGHx)
