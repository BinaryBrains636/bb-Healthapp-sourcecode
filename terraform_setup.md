# BBHealthApp - AWS Infrastructure Setup with Terraform

This guide will help you create AWS infrastructure using Terraform. You will learn Infrastructure as Code (IaC) by provisioning real AWS resources.

---

## Prerequisites

Before you begin, ensure you have the following:

### 1. AWS Account
- **Sign up:** https://aws.amazon.com/free/
- **Note:** AWS Free Tier includes 750 hours/month of EC2, 1000 GB/month of EBS storage
- **Important:** You will need a credit card for verification, but you can stay within free tier limits
- **Student Benefit:** AWS Educate provides free credits for students (if available through your institution)

### 2. AWS CLI
- **Download:** https://aws.amazon.com/cli/
- **Install:** Follow installation instructions for your OS
- **Configure:**
  ```bash
  aws configure
  ```
  You'll need:
  - AWS Access Key ID (from IAM console)
  - AWS Secret Access Key (from IAM console)
  - Default region (e.g., us-west-2)
  - Default output format (json)

- **Verify:**
  ```bash
  aws --version
  aws sts get-caller-identity
  ```

### 3. Terraform
- **Download:** https://www.terraform.io/downloads
- **Install:** Extract and add to PATH
- **Verify:**
  ```bash
  terraform --version
  ```

---

## Quick Start

### Step 1: Clone/Download Project

```bash
cd windsurf-project-3/terraform
```

### Step 2: Configure Variables

Create or edit `terraform.tfvars`:

```hcl
aws_region              = "us-west-2"
environment             = "dev"
cluster_name            = "bbhealthapp-dev"
node_instance_type      = "t3.medium"
node_desired_size       = 2
node_min_size           = 1
node_max_size           = 3
enable_spot_instances   = false
enable_cluster_autoscaler = true
enable_cloudwatch_logging = true
```

### Step 3: Initialize Terraform

```bash
terraform init
```

This downloads required providers and initializes the backend.

### Step 4: Review the Plan

```bash
terraform plan \
  -var-file="terraform.tfvars"
```

Review the resources that will be created. This shows:
- VPC and networking
- EKS cluster
- Node groups
- IAM roles
- Load balancer
- Other AWS resources

### Step 5: Apply the Configuration

```bash
terraform apply \
  -var-file="terraform.tfvars"
```

Type `yes` when prompted to confirm.

**Note:** This will take 15-20 minutes to complete as AWS provisions the EKS cluster.

### Step 6: Verify Infrastructure

After Terraform apply completes, verify the created resources:

```bash
# View Terraform outputs
terraform output

# Check VPC
aws ec2 describe-vpcs --region us-west-2

# Check EKS cluster
aws eks describe-cluster --name bbhealthapp-dev --region us-west-2

# Check EC2 instances
aws ec2 describe-instances --region us-west-2

# Check ECR repositories
aws ecr describe-repositories --region us-west-2
```

### Step 7: Access AWS Console

1. Log in to AWS Console: https://console.aws.amazon.com/
2. Navigate to the region you used (e.g., us-west-2)
3. View the created resources:
   - VPC → Your VPCs
   - EC2 → Instances
   - EKS → Clusters
   - ECR → Repositories
   - IAM → Roles

---

## Understanding Terraform

### What is Terraform?

Terraform is an Infrastructure as Code (IaC) tool that allows you to define and provision infrastructure using declarative configuration files.

### Key Concepts

**Providers:**
- Plugins that interact with cloud providers
- AWS provider interacts with AWS services
- Kubernetes provider interacts with Kubernetes clusters

**Resources:**
- Infrastructure components (VPC, EC2, EKS, etc.)
- Defined in Terraform configuration files

**State:**
- Terraform maintains state of deployed infrastructure
- Stored in S3 for real AWS
- Stored locally for LocalStack

**Modules:**
- Reable Terraform configurations
- Can be shared and versioned

### Terraform Files in This Project

| File | Purpose |
|------|---------|
| `provider.tf` | AWS and Kubernetes provider configuration |
| `variables.tf` | Variable definitions |
| `main.tf` | Main Terraform configuration |
| `vpc.tf` | VPC and networking |
| `eks.tf` | EKS cluster and node groups |
| `iam.tf` | IAM roles and policies |
| `rds.tf` | Database configuration |
| `ecr.tf` | ECR repositories |
| `bastion.tf` | Bastion host configuration |
| `outputs.tf` | Output values |

---

## Cost Management

### AWS Free Tier Limits

- **EKS:** 750 hours/month of control plane
- **EC2:** 750 hours/month of t2.micro/t3.micro
- **EBS:** 1000 GB/month of storage
- **Data Transfer:** 100 GB/month

### Cost Optimization Tips

1. **Use t3.medium or smaller instances**
2. **Enable spot instances** for non-critical workloads
3. **Set up auto-scaling** to scale down when not in use
4. **Delete resources immediately after completion** to avoid ongoing costs
5. **Monitor costs** using AWS Cost Explorer

### Estimated Costs

With the default configuration:
- **EKS Cluster:** ~$0.10/hour = ~$72/month
- **EC2 Nodes (2x t3.medium):** ~$0.04/hour = ~$58/month
- **Load Balancer:** ~$0.0225/hour = ~$16/month
- **EBS Storage:** ~$0.08/GB/month
- **Total:** ~$150-200/month (outside free tier)

**With Free Tier:** ~$0-50/month depending on usage

**Important:** Delete infrastructure immediately after completing the assignment to avoid charges.

---

## Security Considerations

### IAM Best Practices

1. **Use least privilege** - Only grant necessary permissions
2. **Use IAM roles** instead of access keys when possible
3. **Rotate credentials** regularly
4. **Enable MFA** on root account and IAM users
5. **Use AWS Organizations** for multi-account setup

### Network Security

1. **Use private subnets** for databases and internal services
2. **Use security groups** to restrict traffic
3. **Enable VPC Flow Logs** for monitoring
4. **Use Network ACLs** for additional security

### Data Security

1. **Enable encryption** for EBS volumes
2. **Use AWS Secrets Manager** for sensitive data
3. **Enable encryption in transit** (TLS/SSL)
4. **Regular backups** with automated snapshots

---

## Troubleshooting

### Terraform State Lock

If Terraform state is locked:

```bash
terraform force-unlock <LOCK_ID>
```

### AWS Credentials Issues

```bash
# Verify credentials
aws sts get-caller-identity

# Reconfigure
aws configure
```

### EKS Cluster Not Ready

```bash
# Check cluster status
aws eks describe-cluster --name bbhealthapp-dev --region us-west-2
```

### Terraform State Issues

If Terraform state is locked:

```bash
terraform force-unlock <LOCK_ID>
```

### Cluster Not Ready

Check cluster status:

```bash
aws eks describe-cluster --name bbhealthapp-dev --region us-west-2
```

### Node Issues

Check node status:

```bash
aws ec2 describe-instances --region us-west-2
```

### IAM Role Issues

Verify IAM roles:

```bash
aws iam get-role --role-name AmazonEKSLoadBalancerControllerRole
aws iam get-role --role-name AmazonEKS_EBS_CSI_DriverRole
```

---

## Learning Outcomes

By completing this phase, students will learn:

1. **Infrastructure as Code (IaC)**
   - Declarative infrastructure definition
   - Version control for infrastructure
   - Reproducible deployments

2. **AWS Services**
   - VPC (Virtual Private Cloud)
   - EKS (Elastic Kubernetes Service)
   - EC2 (Elastic Compute Cloud)
   - IAM (Identity and Access Management)
   - ECR (Elastic Container Registry)
   - RDS (Relational Database Service)
   - CloudWatch (Monitoring and Logging)

3. **Terraform**
   - HCL (HashiCorp Configuration Language)
   - Terraform state management
   - Terraform modules and providers
   - Infrastructure provisioning

4. **Cloud Best Practices**
   - Cost management and optimization
   - Security best practices
   - Infrastructure automation
   - Resource cleanup and management

---

## Support

For issues or questions:

1. Check Terraform state: `terraform show`
2. Review AWS CloudWatch logs
3. Check AWS service health dashboard
4. AWS Documentation: https://docs.aws.amazon.com/
5. Terraform Documentation: https://www.terraform.io/docs/providers/aws/

---

## Cleanup

**IMPORTANT:** Delete infrastructure immediately after completing the assignment to avoid ongoing AWS charges.

### Step 1: Destroy Terraform Resources

```bash
cd terraform
terraform destroy -var-file="terraform.tfvars"
```

Type `yes` when prompted to confirm.

### Step 2: Verify Cleanup

```bash
# Check if resources are deleted
aws ec2 describe-vpcs --region us-west-2
aws eks list-clusters --region us-west-2
aws ecr describe-repositories --region us-west-2
```

### Step 3: Delete ECR Repositories (if not deleted by Terraform)

```bash
aws ecr delete-repository --repository-name bbhealthapp-master-service --force --region us-west-2
aws ecr delete-repository --repository-name bbhealthapp-register-service --force --region us-west-2
aws ecr delete-repository --repository-name bbhealthapp-document-service --force --region us-west-2
aws ecr delete-repository --repository-name bbhealthapp-frontend --force --region us-west-2
```

---

## Report Requirements

### 1. AWS Account Setup
- Screenshots of AWS console login
- AWS CLI configuration verification (`aws configure` and `aws sts get-caller-identity`)
- IAM user creation (if applicable)
- Screenshot of AWS Free Tier status

### 2. Terraform Execution
- Screenshots of `terraform init` output
- Screenshots of `terraform plan` output (showing resources to be created)
- Screenshots of `terraform apply` execution
- Time taken for infrastructure provisioning
- Any warnings or errors encountered

### 3. Infrastructure Verification
- Screenshots of AWS console showing created resources:
  - VPC (Virtual Private Cloud)
  - Subnets (public and private)
  - EKS Cluster
  - EC2 Instances (node groups)
  - ECR Repositories
  - IAM Roles
  - Security Groups
- Screenshots of Terraform outputs (`terraform output`)

### 4. Terraform Configuration Understanding
- Explanation of each Terraform file:
  - `provider.tf` - What does it configure?
  - `variables.tf` - What variables are defined?
  - `main.tf` - What is the main configuration?
  - `vpc.tf` - What network resources are created?
  - `eks.tf` - What Kubernetes resources are provisioned?
  - `iam.tf` - What IAM roles and policies are created?
  - `rds.tf` - What database configuration is included?
  - `ecr.tf` - What container registries are created?
- Explanation of `terraform.tfvars` and how you customized it

### 5. AWS Services Understanding
- Brief explanation of each AWS service used:
  - VPC (Virtual Private Cloud)
  - EKS (Elastic Kubernetes Service)
  - EC2 (Elastic Compute Cloud)
  - IAM (Identity and Access Management)
  - ECR (Elastic Container Registry)
  - RDS (Relational Database Service)
- Why each service is needed for this architecture

### 6. Cost Analysis
- Estimated costs for deployed infrastructure
- Breakdown of costs by service
- Cost optimization measures you implemented
- Comparison with AWS Free Tier limits
- Actual cost incurred (if any)

### 7. Security Implementation
- IAM roles and policies created
- Security groups configuration (what ports are open/closed)
- Network security measures (public vs private subnets)
- Any additional security best practices you implemented

### 8. Challenges & Solutions
- Any errors encountered during Terraform execution
- How you resolved each error
- Any AWS service limitations you encountered
- Workarounds or solutions implemented

### 9. Infrastructure as Code (IaC) Concepts
- What is Infrastructure as Code?
- Benefits of using Terraform over manual AWS console setup
- How Terraform state management works
- What happens when you run `terraform destroy`?

### 10. Learning Outcomes
- Top 3 things you learned from this exercise
- Understanding of cloud infrastructure provisioning
- Experience with Terraform and HCL language
- How this compares to traditional infrastructure setup

---

## Learning Outcomes

By completing this phase, students will learn:

1. **Infrastructure as Code (IaC)**
   - Declarative infrastructure definition
   - Version control for infrastructure
   - Reproducible deployments

2. **AWS Services**
   - EKS (Elastic Kubernetes Service)
   - VPC (Virtual Private Cloud)
   - EC2 (Elastic Compute Cloud)
   - IAM (Identity and Access Management)
   - ECR (Elastic Container Registry)
   - CloudWatch (Monitoring and Logging)

3. **Terraform**
   - HCL (HashiCorp Configuration Language)
   - Terraform state management
   - Terraform modules and providers
   - Infrastructure provisioning

4. **Kubernetes**
   - kubectl commands
   - Kubernetes deployments
   - Services and ingress
   - Persistent volumes

5. **DevOps Practices**
   - CI/CD integration
   - Infrastructure automation
   - Cost management
   - Security best practices

---

## Support

For issues or questions:

1. **Real AWS:**
   - Check AWS CloudWatch logs
   - Review AWS service health dashboard
   - Check Terraform state: `terraform show`
   - AWS Documentation: https://docs.aws.amazon.com/

2. **LocalStack:**
   - Check LocalStack logs: `docker logs <container>`
   - LocalStack Documentation: https://docs.localstack.cloud/
   - Verify endpoints are accessible

---

**Happy Learning! 🚀**
