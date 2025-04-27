# Creating AWS VPC with Terraform

# 02 Terraform Basics - 05 HOL Terraform HCL Basics

For those that have gone through the Digital Cloud Training AWS Solutions Architect Associate, this lab will be creating many of the components from the VPC lab.  If you have not gone through that course, you can still complete this lab.  Ensure that you have an AWS Account (free tier is preferred) and the AWS CLI installed and authenticated within your Visual Studio Code terminal.

In the `us-east-1` region, we will be creating:

[ ] VPC:

    - Name: MyVPC
    - IPv4 CIDR Block: 10.0.0.0/16

[ ] Subnets (4):

  public_subnet_1a:
    - Name: Public-1A
    - Availability Zone: us-east-1a
    - IPv4 CIDR Block: 10.0.1.0/24

  public_subnet_1b:
    - Name: Public-1B
    - Availability Zone: us-east-1b
    - IPv4 CIDR Block: 10.0.2.0/24

  private_subnet_1a:
    - Name: Private-1A
    - Availability Zone: us-east-1a
    - IPv4 CIDR Block: 10.0.3.0/24

    private_subnet_1b:
    - Name: Private-1B
    - Availability Zone: us-east-1b
    - IPv4 CIDR Block: 10.0.4.0/24

[ ] Internet Gateway:

    - Name: MyIGW

[ ] Route Tables (2):

    public-route-table:
    - Name: Public-RT
    - Subnet associations: public-subnet-1a, public-subnet-1b
    - Destinations:
      - 0.0.0.0/0 -> MyIGW
      - 10.0.0.0/16 -> local

    private-route-table:
    - Name: Private-RT
    - Subnet associations: private-subnet-1a, private-subnet-1b
    - Destinations:
      - 10.0.0.0/16 -> local

Open the `terraform-labs` directory in Visual Studio Code and from the terminal create a new directory:

```shell
mkdir -p 02-terraform-basics/05-hol-terraform-hcl-basics
cd 02-terraform-basics/05-hol-terraform-hcl-basics
```

Create a new file names `02-terraform-basics/05-hol-terraform-hcl-basics/terraform.tf` and paste the following code:

```terraform
terraform {
  required_version = "~> 1.7"
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}
```

Create a new file names `02-terraform-basics/05-hol-terraform-hcl-basics/network.tf` and paste the following code:

```terraform
resource "aws_vpc" "vpc" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name    = "MyVPC"
    Section = "02"
    Lesson  = "05"
    Type    = "HOL"
  }
}
```

In the terminal, run the following commands:

```shell
terraform init
terraform fmt
terraform validate
terraform plan -out=tfplan
terraform apply tfplan
```

Review the new VPC in the AWS Console and save the VPC ID below for later use:

VPC_ID: <VPC_ID>

Update the `network.tf` file by adding the following code, ensuring to replace `<VPC_ID>` with the VPC ID previously saved:

```terraform
resource "aws_subnet" "subnet" {
  vpc_id                  = "<VPC_ID>"
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true
  tags = {
    Name = "Public-1A"
  }
}

resource "aws_subnet" "subnet" {
  vpc_id                  = "<VPC_ID>"
  cidr_block              = "10.0.3.0/24"
  availability_zone       = "us-east-1b"
  map_public_ip_on_launch = true
  tags = {
    Name = "Public-1B"
  }
}

resource "aws_subnet" "subnet" {
  vpc_id            = "<VPC_ID>"
  cidr_block        = "10.0.2.0/24"
  availability_zone = "us-east-1a"
  tags = {
    Name = "Private-1A"
  }
}

resource "aws_subnet" "subnet" {
  vpc_id            = "<VPC_ID>"
  cidr_block        = "10.0.4.0/24"
  availability_zone = "us-east-1b"
  tags = {
    Name = "Private-1B"
  }
}
```

Run the following commands:

```shell
terraform fmt
terraform validate
```

Validate will generate an error because we didn't give our subnets unique resource names.  Update the resource names to match the entries from earlier (public_subnet_1a, public_subnet_1b, private_subnet_1a, private_subnet_1b), then run the folowing commands:

```shell
terraform plan -out=tfplan
terraform apply tfplan
```

Review our subnets in the AWS Console.

Let's ensure that we can recreate everything from scratch.  Run the following commands:

```shell
terraform destroy
terraform apply -auto-approve
```

It is highly unlikely that you will not encounter an error.  Since the VPC ID is set as a string literal, Terraform cannot establish depenencies so it tries to deploy the VPC and all of the subnets in parallel.  Since the VPC doesn't exist yet, the subnets cannot be created.

Clear out the resources again for any that may have been created by running the following:

```shell
terraform destroy -auto-approve
```

Update the `network.tf` file and replace it with the following code:

```terraform
resource "aws_vpc" "vpc" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name    = "MyVPC"
    Section = "02"
    Lesson  = "05"
    Type    = "HOL"
  }
}

resource "aws_subnet" "public_subnet_1a" {
  vpc_id                  = aws_vpc.vpc.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true
  tags = {
    Name = "Public-1A"
  }
}

resource "aws_subnet" "public_subnet_1b" {
  vpc_id                  = aws_vpc.vpc.id
  cidr_block              = "10.0.3.0/24"
  availability_zone       = "us-east-1b"
  map_public_ip_on_launch = true
  tags = {
    Name = "Public-1B"
  }
}

resource "aws_subnet" "private_subnet_1a" {
  vpc_id            = aws_vpc.vpc.id
  cidr_block        = "10.0.2.0/24"
  availability_zone = "us-east-1a"
  tags = {
    Name = "Private-1A"
  }
}

resource "aws_subnet" "private_subnet_1b" {
  vpc_id            = aws_vpc.vpc.id
  cidr_block        = "10.0.4.0/24"
  availability_zone = "us-east-1b"
  tags = {
    Name = "Private-1B"
  }
}
```

Redeploy by running the following command:

```shell
terraform apply -auto-approve
```

Next, we'll add the Internet Gateway.  Update the `network.tf` file with the following code:

```terraform
resource "aws_internet_gateway" "igw" {
  tags = {
    Name = "MyIGW"
  }
}

resource "aws_internet_gateway_attachment" "igw" {
  internet_gateway_id = aws_internet_gateway.igw.id
  vpc_id              = aws_vpc.vpc.id
}
```

Run the following commands:

```shell
terraform plan -out=tfplan
terraform apply tfplan
```

Next, we're going to create our Route Tables and associate them.  Update the `network.tf` file with the following code:

```terraform
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

Run the following commands:

```shell
terraform plan -out=tfplan
terraform apply tfplan
```

Review the VPC in the AWS Console and ensure the all of the components have been created and properly associated.

Next, we want to remove more of the hard-coded value by using variables.  Create a new file named `variables.tf` and add the following code:

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

In the `network.tf` file, update the VPC and subnets to use the variables and a set of functions that we'll discuss in a later lesson:

```terraform
resource "aws_vpc" "vpc" {
  cidr_block = var.vpc_cidr_block

  tags = {
    Name    = var.vpc_name
    Section = "02"
    Lesson  = "05"
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
```

Run the following command:

```shell
terraform apply -auto-approve
```

Let's update the VPC CIDR block to use a different value using an environment variable.

For Unix-type systems run the following commands:

```shell
export TF_VAR_vpc_cidr_block="10.0.0.0/17"
terraform plan -out=tfplan
```

For Windows systems run the following commands:

```shell
$env:TF_VAR_vpc_cidr_block = '10.0.0.0/17'
terraform plan -out=tfplan
```

The plan says that it will force a redeployment of the VPC because of the CIDR Block change.  Let's switch to a TFVARS file.  Create a new file named `terraform.tfvars` and paste the following:

```tfvars
vpc_name       = "MyVPCTFVARS"
vpc_cidr_block = "10.0.0.0/18"
```

Run the following commands and triggering a redeployment of the VPC:

```shell
terraform plan -out=tfplan
terraform apply tfplan
```

Review the VPC in the AWS Console and ensure that the CIDR block has been updated.

Now, let's override the name with a command-line argument.  Run the following command:

```shell
terraform plan -var="vpc_name=MyVPCCLI" -out=tfplan
terraform apply tfplan
```

Now we will clean up all of the resources by running the following command:

```shell
terraform destroy -auto-approve
```
