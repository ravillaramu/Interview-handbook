# Terraform AWS EC2 Example

This example configures the AWS provider, looks up the latest Amazon Linux 2023 AMI, and launches an encrypted EC2 instance in an existing subnet. The instance has no public IP and its security group allows no inbound traffic by default.

## `versions.tf`

```hcl
terraform {
  required_version = ">= 1.5.0, < 2.0.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}
```

## `variables.tf`

```hcl
variable "aws_region" {
  description = "AWS region where the instance will be created."
  type        = string
  default     = "us-east-1"
}

variable "subnet_id" {
  description = "ID of an existing subnet in the selected AWS region."
  type        = string
}

variable "vpc_id" {
  description = "ID of the VPC that contains subnet_id."
  type        = string
}

variable "instance_type" {
  description = "EC2 instance type for the example."
  type        = string
  default     = "t3.micro"
}
```

## `main.tf`

```hcl
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-2023.*-x86_64"]
  }

  filter {
    name   = "architecture"
    values = ["x86_64"]
  }

  lifecycle {
    postcondition {
      condition     = self.architecture == "x86_64"
      error_message = "The selected AMI must use the x86_64 architecture."
    }
  }
}

resource "aws_security_group" "instance" {
  name_prefix = "terraform-example-"
  description = "Security group for the example EC2 instance"
  vpc_id      = var.vpc_id

  # No inbound rules are added. Add only application-specific ingress
  # rules from trusted sources if the workload needs network access.
  egress {
    description = "Allow outbound traffic for updates and required services"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name        = "terraform-example-instance"
    Environment = "example"
    ManagedBy   = "terraform"
  }
}

resource "aws_instance" "example" {
  ami                         = data.aws_ami.amazon_linux.id
  instance_type               = var.instance_type
  subnet_id                   = var.subnet_id
  vpc_security_group_ids      = [aws_security_group.instance.id]
  associate_public_ip_address = false
  monitoring                  = true

  metadata_options {
    http_endpoint               = "enabled"
    http_tokens                 = "required"
    http_put_response_hop_limit = 1
  }

  root_block_device {
    encrypted             = true
    volume_type           = "gp3"
    delete_on_termination = true
  }

  tags = {
    Name        = "terraform-example-instance"
    Environment = "example"
    ManagedBy   = "terraform"
  }

  lifecycle {
    create_before_destroy = true
  }
}
```

The AMI data source uses a `postcondition` to validate the result it reads. `create_before_destroy` is a managed-resource lifecycle rule; it asks Terraform to create a replacement EC2 instance before destroying the old one when replacement is required. It does not by itself provide zero-downtime traffic switching, and replacement may fail if AWS limits or other constraints prevent both instances from existing at once. For production, connect instances to a load balancer and verify health before shifting traffic. `prevent_destroy` is another resource lifecycle option for critical instances, but it intentionally blocks planned replacement or deletion until that rule is removed.

## `outputs.tf`

```hcl
output "instance_id" {
  description = "ID of the EC2 instance."
  value       = aws_instance.example.id
}

output "instance_private_ip" {
  description = "Private IP address of the EC2 instance."
  value       = aws_instance.example.private_ip
}
```

## Run it

Save the snippets as the named files in one directory. Provide IDs for an existing VPC and subnet in the selected region, and use an AWS identity configured through an IAM role, AWS IAM Identity Center, or another approved credential provider.

```sh
terraform init
terraform fmt -check
terraform validate
terraform plan \
  -var='vpc_id=vpc-0123456789abcdef0' \
  -var='subnet_id=subnet-0123456789abcdef0'
terraform apply \
  -var='vpc_id=vpc-0123456789abcdef0' \
  -var='subnet_id=subnet-0123456789abcdef0'
```

Review the plan before applying. The subnet must be in the selected region and VPC and have a route appropriate to the workload. Because the instance has no public IP or inbound rules, use Systems Manager or another approved private access method if you need administration access; configure the required IAM instance profile, network path, and SSM permissions before relying on Session Manager. The sample intentionally does not configure a backend; configure a secured remote backend with access controls, encryption, versioning, and state locking for team use. Do not commit credentials, state files, or secret-bearing plan files. Pin provider versions and review upgrades in a real project.