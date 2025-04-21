#  VS Code Extensions and Settings

## Recommended Extensions

- HashiCorp Terraform Extension
- HashiCorp HCL Extension
- Prettier – Code Formatter
- Indent-rainbow


## Recommended settings to add into settings.json

```json
{
    "workbench.colorTheme": "Default Dark Modern",
    "workbench.sideBar.location": "right",
    "terminal.integrated.defaultProfile.windows": "Git Bash",
    "[terraform]": {
        "editor.defaultFormatter": "hashicorp.terraform",
        "editor.formatOnSave": true, 
        "editor.tabSize": 2
    },

    "terraform.languageServer.enable": true,

    "[terraform-vars]": {
        "editor.defaultFormatter": "hashicorp.terraform",
        "editor.formatOnSave": true,
        "editor.tabSize": 2
    }
}
```
