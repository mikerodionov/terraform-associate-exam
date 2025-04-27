# Terraform Workflow HOL

## Create configuration file

```Bash
mkdir -p terraform-labs/02-terraform-basics/02-hol-workflow
code .
touch main.tf
```

## main.tf

```HCL
# resource "<TYPE>" "<NAME>" { <configuration arguments> }
# <TYPE> - The type of resource, coming from a provider
# <NAME> - This is the local name or resource label within your configuration
# <NAME> - It's how you refer to the resource elsewhere in your code
# <NAME> - Must be unique per resource type in the module

resource "random_string" "filename" {
  length  = 8
  special = false
}

resource "local_file" "my_file" {
  filename        = "example.txt"
  content         = "Hello, Terraform!"
  file_permission = "0644"
}

resource "local_file" "my_file2" {
  filename             = "files/example2.txt"
  content              = "Hello, Terraform!"
  directory_permission = 0755
  file_permission      = "0644"
}

resource "local_file" "my_file3" {
  filename             = "files/${random_string.filename.result}.txt" # string interpolation example
  content              = "Hello, Terraform!"
  directory_permission = 0755
  file_permission      = "0644"
}
# resource "<TYPE>" "<NAME>" { <configuration arguments> }
# <TYPE> - The type of resource, coming from a provider
# <NAME> - This is the local name or resource label within your configuration
# <NAME> - It's how you refer to the resource elsewhere in your code
# <NAME> - Must be unique per resource type in the module

resource "random_string" "filename" {
  length  = 8
  special = false
}

resource "local_file" "my_file" {
  filename        = "example.txt"
  content         = "Hello, Terraform!"
  file_permission = "0644"
}

resource "local_file" "my_file2" {
  filename             = "files/example2.txt"
  content              = "Hello, Terraform!"
  directory_permission = 0755
  file_permission      = "0644"
}

resource "local_file" "my_file3" {
  filename             = "files/${random_string.filename.result}.txt" # string interpolation example
  content              = "Hello, Terraform!"
  directory_permission = 0755
  file_permission      = "0644"
}
```

## Run terraform init / apply

**terraform init**
-	It can infer from resource block type “local_file” required provider (resource block goes “provider” “resource _name” and it will lookup for it in the highest priority namespace which is hashicorp (hashicorp/local) and will download it under .terraform hidden directory
-	It will also create .terraform.lock.hcl file – provider lock file, this file ensures that we download the same version of provider next time we run terraform init.
**terraform apply**
-	Outputs plan and applies it after confirmation
-	Creates terraform.tfstate file

## Other Terraform workflow commands

```Bash
terraform plan -out tfplan
terraform apply tfplan
terraform apply -destroy
terraform show # shows state info in human-readable form
terraform apply -auto-approve
terraform destroy -auto-approve
terraform fmt # apply style guidelines
terraform fmt -check # returns filename(s) of a file(s) where style needs to be corrected and non-0 exit code when such files are present in working directory
terraform validate # tries to identify syntax errors
terraform init
```
