# Installing Terraform on Linux

## Binary Download

```Bash
# Download terraform binary from terraform.io
cd Downloads
unzip terraform_1.11.4_linux_amd64.zip
./terraform version
mv Downloads/terraform bin/
terraform version
```

## Package Manager

```Bash
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform
```
