# Azure DevOps Pipelines and Templates

Azure DevOps YAML pipelines.

[[_TOC_]]

## Prerequisites and Assumptions

Many tasks take avantage of the `$(System.AccessToken)` variable.  To gain access to this the build pipeline must enable the `Allow scripts to access the OAuth token` option in the Agent Job preferences pane.

Modifications to the working directories and URIs in the tasks may need to be customized depending on how the Terraform and wiki is structured.  

## Pipeline File Descriptions

### Build Readiness

### Terraform Plan

### Terraform Publish Artifact

### Terraform Publish Plan to Wiki

Current assumptions...
* The wiki will be cloned into `$(Agent.WorkFolder)/wiki`.
* The Terraform `tfplan` file can be found in a path stored in the `$(terraform.path)` pipeline variable.
* The wiki has a `/Builds` directory at it's root as well as a `/Builds/Template.md` file (see below).
* Terraform is already installed on the build agent.

### VSTS Agent Deploy to AKS