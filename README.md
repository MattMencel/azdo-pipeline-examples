# Azure DevOps Pipelines and Templates

Azure DevOps YAML pipelines.

[[_TOC_]]

## Prerequisites and Assumptions

Many tasks take avantage of the `$(System.AccessToken)` variable.  To gain access to this the build pipeline must enable the `Allow scripts to access the OAuth token` option in the Agent Job preferences pane.

Modifications to the working directories and URIs in the tasks may need to be customized depending on how the Terraform repo and the AzDO Organization wiki folders are organized.  

## Pipeline File Descriptions

### Build Readiness

The [template-terraform-build-readiness.yml](template-terraform-build-readiness.yml) contains build readiness tasks that can be run to ensure the code is ready for build, without obvious errors, and formatted for clarity.

### Terraform Plan

The [template-terraform-plan.yml](template-terraform-plan.yml) runs terraform plan tasks.

### Terraform Publish Artifact

The [template-terraform-publish-artifact.yml](template-terraform-publish-artifact.yml) template publishes the terraform plan as an artifact so it can be used in a release to do a `terraform apply`.

One thing I’ve discovered is that the publishing of the Terraform artifact can take a really long time. When using modules the `.terraform` directory starts growing and we can eventually have thousands of files, some well over 10K files, that need to be uploaded in the artifact publish step. There is metadata that AzDO is publishing along with the files and when it has thousands of small files to process it takes a really long time. Twenty to thirty minutes in some cases.

To combat this you can compress the Terraform directory on the build agent into a single file, upload that, and then in the release step you just need to uncompress the archive before your terraform apply is run.

The compress step creates a single tar.gz file, in the default build agent directory, from the `terraform.path` and names it with the `state.key` variable.

This tar.gz file is then published as the artifact. The example above with 10K+ files in it that took over 20 minutes to publish originally, now takes no more than 15 seconds.

### Terraform Publish Plan to Wiki

The [template-terraform-publish-plan-to-wiki.yml](template-terraform-publish-plan-to-wiki.yml) template will publish the plan output to a wiki page in AzDO and include some of the build metadata in the page output.

Current assumptions...
* The wiki will be cloned into `$(Agent.WorkFolder)/wiki`.
* The Terraform `tfplan` file can be found in a path stored in the `$(terraform.path)` pipeline variable.
* The wiki has a `/Builds` directory at it's root as well as a `/Builds/Template.md` file (see below).
* Terraform is already installed on the build agent.

### Terraform Azure Pipeline

The [terraform-azure-pipeline.yml](terraform-azure-pipeline.yml) is the base pipeline from which all other template pipelines are called.

When setting up a Terraform pipeline for a new stack you can copy the `terraform-azure-pipeline.yml` file to the root of your stack, rename it to `azure-pipelines.yml` and edit it as needed.

### VSTS Agent Deploy to AKS

The [vsts-agent-deploy-to-aks.yml](vsts-agent-deploy-to-aks.yml) pipeline deploys custom VSTS Agent pods to an AKS cluster so they can be used by other build and release pipelines.