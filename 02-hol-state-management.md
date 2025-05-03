# HOL State Management

- Practice state rm, mv, import and respective block
- State migration from local to S3 backen

## Initial Config

terraform.tf

```terraform
terraform {
  required_version = "~> 1.7"
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  #  backend "s3" {
  #    bucket         = "<BUCKET_NAME>"
  #    key            = "terraform.tfstate"
  #    dynamodb_table = "dyntbl_terraform.tfstate"
  #    region         = "us-east-1"
  #  }
}

provider "aws" {
  region = "us-east-1"
}
```

network.tf

```terraform
resource "aws_vpc" "vpc" {
  cidr_block = var.vpc_cidr_block

  tags = {
    Name    = var.vpc_name
    Section = "02"
    Lesson  = "07"
    Type    = "HOL"
  }
}

resource "aws_subnet" "public_subnet_1a" {
  vpc_id                  = aws_vpc.vpc.id
  cidr_block              = cidrsubnet(var.vpc_cidr_block, 24 - split("/", var.vpc_cidr_block)[1], 1)
  availability_zone       = var.availability_zone_1
  map_public_ip_on_launch = true
  tags = {
    Name = "Public-1A"
  }
}

resource "aws_subnet" "public_subnet_1b" {
  vpc_id                  = aws_vpc.vpc.id
  cidr_block              = cidrsubnet(var.vpc_cidr_block, 24 - split("/", var.vpc_cidr_block)[1], 3)
  availability_zone       = var.availability_zone_2
  map_public_ip_on_launch = true
  tags = {
    Name = "Public-1B"
  }
}

resource "aws_subnet" "private_subnet_1a" {
  vpc_id            = aws_vpc.vpc.id
  cidr_block        = cidrsubnet(var.vpc_cidr_block, 24 - split("/", var.vpc_cidr_block)[1], 2)
  availability_zone = var.availability_zone_1
  tags = {
    Name = "Private-1A"
  }
}

resource "aws_subnet" "private_subnet_1b" {
  vpc_id            = aws_vpc.vpc.id
  cidr_block        = cidrsubnet(var.vpc_cidr_block, 24 - split("/", var.vpc_cidr_block)[1], 4)
  availability_zone = var.availability_zone_2
  tags = {
    Name = "Private-1B"
  }
}

resource "aws_internet_gateway" "igw" {
  tags = {
    Name = "MyIGW"
  }
}

resource "aws_internet_gateway_attachment" "igw" {
  internet_gateway_id = aws_internet_gateway.igw.id
  vpc_id              = aws_vpc.vpc.id
}

resource "aws_route_table" "public_route_table" {
  vpc_id = aws_vpc.vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }

  route {
    cidr_block = aws_vpc.vpc.cidr_block
    gateway_id = "local"
  }

  tags = {
    Name = "Public-RT"
  }
}

resource "aws_route_table" "private_route_table" {
  vpc_id = aws_vpc.vpc.id

  route {
    cidr_block = aws_vpc.vpc.cidr_block
    gateway_id = "local"
  }

  tags = {
    Name = "Private-RT"
  }
}

resource "aws_route_table_association" "public_subnet_1a" {
  subnet_id      = aws_subnet.public_subnet_1a.id
  route_table_id = aws_route_table.public_route_table.id
}

resource "aws_route_table_association" "public_subnet_1b" {
  subnet_id      = aws_subnet.public_subnet_1b.id
  route_table_id = aws_route_table.public_route_table.id
}

resource "aws_route_table_association" "private_subnet_1a" {
  subnet_id      = aws_subnet.private_subnet_1a.id
  route_table_id = aws_route_table.private_route_table.id
}

resource "aws_route_table_association" "private_subnet_1b" {
  subnet_id      = aws_subnet.private_subnet_1b.id
  route_table_id = aws_route_table.private_route_table.id
}
```

variables.tf

```terraform
variable "vpc_name" {
  default     = "MyVPCDefault"
  description = "The name of the VPC"
  type        = string
}

variable "vpc_cidr_block" {
  default     = "10.0.0.0/16"
  description = "The CIDR block for the VPC"
  type        = string
}

variable "availability_zone_1" {
  default     = "us-east-1a"
  description = "The first availability zone"
  type        = string
}

variable "availability_zone_2" {
  default     = "us-east-1b"
  description = "The second availability zone"
  type        = string
}
```

## Deploy configuration

```bash
terraform init
terraform apply -auto-approve
```

## Review state

Install GraphViz through one of the following methods:

1. Download and install manually from https://graphviz.org
2. For macOS with Homebrew: `brew install graphviz`
3. For Windows with Chocolatey: `choco install graphviz`
4. For Debian-based Linux: `sudo apt install graphviz`
5. For Red Hat-based Linux: `sudo yum install graphviz`

Review state and generate state graph

```shell
terraform state list
terraform graph | dot -Tpng > tfstate.png
```

## Remove and import VPC again

```shell
terraform state rm aws_vpc.vpc
terraform plan -out=tfplan
terraform import aws_vpc.vpc <VPC_ID>
terraform plan -out=tfplan
```

## Update resource names in network.tf to practice resource move operation

Change the resource name from "public_route_table" and "private_route_table" to "public-rt" and "private-rt" respectively.

CLI move

```shell
terraform state mv aws_route_table.public_route_table aws_route_table.public-rt
terraform state mv aws_route_table.private_route_table aws_route_table.private-rt
terraform plan -out=tfplan
```

## Config-driven import

```shell
terraform state rm aws_vpc.vpc
```

Make a new file called `import.tf` an paste the following content (ensuring you paste the actual VPC_ID):

```terraform
import {
  id = "<VPC_ID>"
  to = aws_vpc.vpc
}
```

Then run the following:

```shell
terraform plan -out=tfplan
terraform apply tfplan
```

## Create S3 bucket for state migration

```bash
touch s3.tf
```

```terraform
resource "random_string" "bucket_string" {
  length  = 12
  special = false
  upper   = false
}

resource "aws_s3_bucket" "backend" {
  bucket = "${random_string.bucket_string.result}-07-hol"

  tags = {
    Name    = "${random_string.bucket_string.result}-07-hol"
    Section = "02"
    Lesson  = "07"
    Type    = "HOL"
  }
}

resource "aws_s3_bucket_versioning" "backend" {
  bucket = aws_s3_bucket.backend.id

  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_dynamodb_table" "state_lock" {
  name           = "dyntbl_terraform.tfstate"
  read_capacity  = 1
  write_capacity = 1
  hash_key       = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}

output "bucket_name" {
  value = aws_s3_bucket.backend.bucket
}
```

```shell
terraform init
terraform plan -out=tfplan
terraform apply tfplan
```

Review the AWS console to verify the creation of the bucket.

Next, update the `terraform.tf` file uncommenting the `backend` block and replacing the bucket name, then run the following:

```shell
terraform init
terraform apply
rm terraform.tfstate*
terraform state list
```

To migrate the state locally, comment out the `backend` block in the `terraform.tf` file and run the following:

```shell
terraform init -migrate-state
terraform apply
```

There should be a terraform.tfstate file locally, again.

NOTE: Moving state locally is a common pattern for moving state from one remote backend type to another as many are not support to shift directly from one to another.

Cleanup your resources to avoid cost accrual. Go into your S3 bucket and empty the contents from the portal or via the AWS CLI.

In the AWS Console, empty your bucket, then run the following:

```shell
terraform destroy -auto-approve
```
