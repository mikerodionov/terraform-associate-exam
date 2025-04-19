# Terraform Installation on Windows

## Direct download of executable from terraform.io

```PoSH
# Download latest version of terraform from terraform.io
# Extract archive
mkdir C:\Terraform
Expand-Archive C:\Downloads\terraform_1.11.4_windows_amd64.zip -DestinationPath C:\Terraform
# Add terraform executable location to Path variable
$env:Path += ';C:\Terraform'
terravorm -version
```

## Usingh Chocolatey package manager

```Posh
# Install Chocolatey from elevated PoSh window
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
# Install Terraform using Chocolatey
choco install terraform -y
terraform -version
choco upgrade terraform
```
