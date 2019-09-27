# Azure Devops YAML Pipelines

Example Azure Devops YAML Pipelines

## Prerequisites and Assumptions

Many tasks take advantage of the `$(System.AccessToken)` variable.  To gain access to this the build pipeline must enable the `Allow scripts to access the OAuth token` option in the Agent Job preferences pane.

Modifications to the working directories and URIs in the tasks may need to be customized depending on how the Terraform repo and the AzDO Organization wiki folders are organized.  

## Terraform

Pipeline and templates for use in Terraform plan and apply. The [azure-pipelines.yml](Terraform/azure-pipelines.yml) is the base pipeline from which all other template pipelines are called.

When setting up a Terraform pipeline for a new stack you can copy the `azure-pipelines.yml` file to the root of your stack and edit it as needed.
