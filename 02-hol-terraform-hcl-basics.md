# Creating AWS VPC with Terraform

Create a lab directory structure for labs in this course.  Choose somewhere convenient, like the home directory.  We will create a directory named `terraform-labs` in the home directory and open a new Visual Studio Code window in the directory.

```shell
mkdir -p terraform-labs/02-terraform-basics/02-hol-workflow
cd terraform-labs
code .
```

In the Visual Studio Code terminal, run the following code:

```shell
cd 02-terraform-basics/02-hol-workflow
terraform version
terraform -help
terraform init
```

Next, make a new file named `02-terraform-basics/02-hol-workflow/main.tf` in Visual Studio Code and paste the following code, then save it:

```terraform
resource "local_file" "my_file" {
  filename        = "hello.txt"
  content         = "Hello, Terraform!"
  file_permission = 0644
}
```

Optional, add Terraform auto-complete (only works for Unix-type operating systems):

```shell
terraform -install-autocomplete
```

Run the following commands, analyzing each:

```shell
terraform init
terraform apply
terraform plan -destroy
terraform destroy
terraform plan -out=tfplan
terraform show tfplan
terraform apply tfplan
terraform show
```

Update the `main.tf` file replacing it with the following code, then save:

```terraform
resource "local_file" "my_file"    {
  filename = "files/hello.txt"
content         =   "Hello, Terraform!"
  directory_permission = 0755
  file_permission = 0644
}
```

Run the following commands:

```shell
terraform apply -auto-approve
terraform destroy -auto-approve
terraform fmt
terraform apply -auto-approve
```

Edit your user settings in Visual Studio Code to enable auto-formatting on save.

cmd + shift + p (macOS)
ctrl + shift + p (Windows/Linux)

Type `User Settings` and click on `Preferences: User Settings (JSON)`.  Add or modify the `"[terraform]"` section being careful to mind the trailing commas, then save:

```json
    "[terraform]": {
        "editor.defaultFormatter": "hashicorp.terraform",
        "editor.formatOnSave": true,
        "editor.formatOnSaveMode": "file"
    },
```

Update the `main.tf` file replacing it with the following code, then save:

```terraform
**resource "random_string" "filename" {
  length  = 8
  special = false


resource "local_file" "my_file"    {
  filename             = "files/$random_string.filename.result}.txt"
  content              =   "Hello, Terraform!"
  directory_permission = 0755
  file_permission      = 0644
}**
```

```shell
terraform validate
terraform init
terraform validate
terraform apply -auto-approve
```

Notice the file created in the `files` directory.  It didn't yield the desired results.  Validate doesn't handle logical errors.  Fix the error in the `main.tf` file with the following content:

```terraform
resource "random_string" "filename" {
  length  = 8
  special = false
}

resource "local_file" "my_file"    {
  filename             = "files/${random_string.filename.result}.txt"
  content              =   "Hello, Terraform!"
  directory_permission = 0755
  file_permission      = 0644
}
```

Then run:

```terraform
terraform apply -auto-approve
```
