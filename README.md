# Publish Terraform Plans to the Wiki in Azure DevOps

An Azure DevOps (formerly VSTS) Task Group for publishing Terraform Plans to a wiki page for review.

## Prerequisites and Assumptions

The git tasks take avantage of the `$(System.AccessToken)` variable.  To gain access to this the build pipeline must enable the `Allow scripts to access the OAuth token` option in the Agent Job preferences pane.

Modifications to the working directories in the tasks may need to be customized depending on how the Terraform and wiki is structured.  

Tasks currently assume...
* The wiki will be cloned into `$(Agent.WorkFolder)/wiki`.
* The Terraform `tfplan` file can be found in a path stored in the `$(terraform.path)` pipeline variable.
* The wiki has a `/Builds` directory at it's root as well as a `/Builds/Template.md` file (see below).
* Terraform is already installed on the build agent.

## Usage

Go to the `Task Groups` section in ADO.  Import the `publish_terraform_plan.json` file.  Edit as needed.

In the builds where `terraform plan` is run, output the plan results to a plan file named `tfplan`.

Attach the new `Publish Terraform Plans` task group after the `terraform plan` tasks.

## Task Descriptions

### Task 1 - Git Clone Wiki

This task clones the main wiki to the agent.

If using custom agents there may already be an existing ./wiki directory on the agent. So this task also checks for that and removes it first if it exists.

Edit the wiki git URL in the task for your ORGANIZATION and PROJECT.

``` sh
if [ -d \"wiki\" ]; then rm -rf wiki; fi

git clone https://test:$(System.AccessToken)@ORGANIZATION.visualstudio.com/PROJECT/_git/PROJECT.wiki wiki
```

### Task 2 - Git Config

Set the git user.name and user.email.

``` sh
git config --global user.email "ado_build@EXAMPLEDOMAIN.com"
git config --global user.name "ado_build"
```


### Task 3 - Terraform Show - Output

Run terraform show and output the results to a file named `tfplan.out`.

``` sh
terraform show -no-color tfplan > tfplan.out
```

### Task 4 - Write Terraform Show to Wiki

Writes the `tfplan.out` to a new wiki page named with the build number.

* Create a directory for the build (`Builds/$(Build.DefinitionName)`) if one does not exist yet.
* Copies the `Template.md` to a new build markdown file named after the `$(Build.BuildNumber)`.
* Appends details about the build and then the contents from the `tfplan.out` file.

**Contents of Template.md**
``` markdown
<!-- DO NOT DELETE OR EDIT DIRECTLY UNLESS MODIFYING TEMPLATE -->

# Results

[[_TOC_]]
```

**Task**
``` sh
bn=$(Build.DefinitionName)
build_name=${bn// /_}
template="wiki/Builds/Template.md"
file_name="wiki/Builds/${build_name}/$(Build.BuildNumber).md"
tfout="$(System.DefaultWorkingDirectory)/$(terraform.path)/tfplan.out"

mkdir -p "wiki/Builds/${build_name}"

cat ${template} > ${file_name}

cat <<EOT >> ${file_name}

## Build Details

<table>
  <tr>
    <td><b>Build Reason</b></td>
    <td>$(Build.Reason)</td>
  </tr>
  <tr>
    <td><b>Requestor</b></td>
    <td>$(Build.RequestedFor)</td>
  </tr>
  <tr>
    <td><b>Repository</b></td>
    <td><a href=$(Build.Repository.Uri)>$(Build.Repository.Uri)</a></td>
  </tr>
  <tr>
    <td><b>Branch</b></td>
    <td>$(Build.SourceBranchName)</td>
  </tr>
  <tr>
    <td><b>Latest Commit</b></td>
    <td><a href=$(Build.Repository.Uri)/commit/$(Build.SourceVersion)>$(Build.SourceVersion)</a></td>
  </tr>
  <tr>
    <td><b>Latest Commit Message</b></td>
    <td><pre>$(Build.SourceVersionMessage)</pre></td>
  </tr>
</table>

## Terraform Show

\`\`\`
EOT

cat ${tfout} >> ${file_name}
cat <<EOT >> ${file_name}
\`\`\`
EOT
```

### Task 5 - Archive Old Builds

To keep build directories from filling up with old builds, archive all but the 10 most recent builds.

``` sh
# WORKING DIRECTORY IS $(Agent.WorkFolder)/wiki/Builds
bn=$(Build.DefinitionName)
build_name=${bn// /_}

# CD INTO THE build_name DIRECTORY
cd ${build_name}

# CREATE THE Archives DIRECTORY IF IT DOESN'T EXIST
mkdir -p "Archives"

# FIND ALL FILES IN THE CURRENT WORKING DIRECTORY | 
#   REVERSE SORT | 
#   GET ALL BUT THE LATEST 10 FILES |
#   MOVE THEM TO THE Archives FOLDER
find * -maxdepth 0 -type f | sort -nr | awk 'NR > 10' | xargs -i mv {} ./Archives/
```

### Task 6 - Git Push Wiki

Commit and push the wiki changes.

Edit the wiki git URL in the task for your ORGANIZATION and PROJECT.

``` sh
git pull
git add .
git commit -m "Build $(Build.BuildNumber)"
git push https://test:$(System.AccessToken)@ORGANIZATION.visualstudio.com/PROJECT/_git/PROJECT.wiki
```