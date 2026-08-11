# cst8918-lab10

CST8918 - DevOps: Infrastructure as Code
Lab A11: Remote state storage with Azure Blob Storage

## Overview

By default, Terraform stores its state file locally. This lab configures Terraform to store state remotely in an Azure Blob Storage container instead, which is what you need for team collaboration and CI/CD pipelines.

## Backend

Remote state is configured in [terraform/terraform.tf](terraform/terraform.tf) using the `azurerm` backend:

| Setting | Value |
| --- | --- |
| Resource group | `riya0009-cst8918-tf-backend` |
| Storage account | `riya0009tfstorage` |
| Container | `tfstate` |
| State file key | `dev.terraform.tfstate` |

The backend resource group is kept separate from the managed infrastructure so it isn't accidentally destroyed along with everything else.

## Resources

[terraform/main.tf](terraform/main.tf) provisions a small sample stack, all named from the `resource_prefix` variable (default `riya0009-a11`):

- `azurerm_resource_group.rg`
- `azurerm_virtual_network.vnet`
- `azurerm_subnet.subnet`

## Usage

```bash
cd terraform

# The storage account access key is required to read/write the backend.
export ARM_ACCESS_KEY=$(az storage account keys list \
  --account-name riya0009tfstorage \
  --resource-group riya0009-cst8918-tf-backend \
  --query "[0].value" -o tsv)

terraform init
terraform fmt && terraform validate
terraform plan -out=a11.tfplan
terraform apply a11.tfplan
```

## Cleanup

```bash
terraform destroy
```

Destroying the resources does not delete the state file in the storage account — it's left behind (empty) so the backend can be reused.
