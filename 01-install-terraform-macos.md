# Install Terraform on MacOS

## Binary download 

```Bash
# Download architecture specific binary (AMD64/ARM64)
cd Downloads
./terrafom version # to execute downloaded Terraform binary
mkdir bin
mv Downloads/terraform bin/
PATH=$PATH:~/bin
terraform version
# Modify bash profile
vi .bash_profile
# add  new line: export PATH=$PATH:~/bin
```

## Using Homebrew package manager

```Bash
# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
# Install Terraform package
brew install terraform # Worked w/o extra steps before Terraform 1.6 (swith to BSL license)
# hence you need to add private HashiCorp registry of brew packages first
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
terraform version
```
