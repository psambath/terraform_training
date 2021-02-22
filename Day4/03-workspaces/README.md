# Terraform Enterprise - Workspaces

## Expected Outcome

In this challenge, you will create a repository, a TFE workspace, and link them together.

## How to

### [GitHub] Create a new github repository

Login to github and create a new repository by navigating to <https://github.com/new>.

Use the following settings:
- `name` = "ptfe-workspace"
- Public repo
- Check "Initialize this repository with a README"
- Add `.gitignore` of type "Terraform"

![](img/github-new.png)

Once created, clone the repository to your local machine.

### [TFE] Create a Workspace

Login to TFE and click the "+ New Workspace" button.

Select the VCS Connection to github.

![](img/tfe-find-repo.png)

In the repository section:

* Click link "Advanced Options" to show all
* Enter the name of the repository you created above
* Set the working directory to "/first-workspace"

![](img/tfe-new-workspace.png)

### [Local] Update your repository

In your repository, create a folder called "first-workspace/" and then create a `main.tf` file with the following contents:

```hcl
variable "name_count" {
  description = "The number of names to produce."
  default     = 3
}

variable "name_prefix" {
  description = "The prefix string to start each name with."
  default     = null
}

resource "random_pet" "names" {
  count  = var.name_count
  prefix = var.name_prefix
}

output "names" {
  value = random_pet.names.*.id
}
```

Commit the changes through the GitHub UI.

*Optional to do this through a CLI.*
```sh
git add *
git commit -m "My First Workspace"
git push origin master
```

### [TFE] Queue a Plan

Back in your TFE Workspace, navigate to your workspace.

![](img/queue-plan.png)

Verify the plan succeeded, it should look something like this:

![](img/tfe-first-run-plan.png)

Discard the the Run.

### [TFE] Workspace Variables

Go to the "Variables" tab.

Enter the following variables in the Terraform Variables section:

- "name_count" = <a number between 1 and 10>
- "name_prefix" = "<insert your username>"

![](img/workspace-var.png)

### [TFE] Apply

Back in your TFE Workspace, queue another plan.

Once the plan succeeds you should see a different output than the first plan.

Confirm the plan, by clicking "Confirm & Apply".

Verify the outputs are as expected.

![](img/tfe-first-run-output.png)

### Extra Credit

1. Find the workspace id (hint: it will start with "ws-").
2. Commit a change to source control, what happens?
3. Queue two plans, can you apply them out of order?
4. Check out the state file and explore version differences.

## How To (Phase 2)

### [Local] Update your Repository

In your repository, create a folder called `app-web` and then create a `main.tf` file with the following contents:

```hcl
provider "aws" {
}

resource "aws_instance" "web" {
  ami                    = var.ami
  instance_type          = "t2.micro"
  subnet_id              = var.subnet_id
  vpc_security_group_ids = var.vpc_security_group_ids

  tags = {
    "Identity"    = var.identity
    "Name"        = "Student"
    "Environment" = "Training"
  }
}
```
Commit the changes in GitHub.

In your `app-web` folder create a `variables.tf` file with the following contents:

```hcl
variable "ami" {}
variable "web_count" {}
variable "subnet_id" {}
variable "identity" {}
variable "vpc_security_group_ids" {
  type = list
}
```

Commit the changes in GitHub.

### [TFE] Create a Workspace

Login to TFE and click the "+ New Workspace" button.

Create another workspace, similar to above, with the following changes:

* Name the workspace `app-web`
* Use the same repository (ptfe-workspace)
* Point the workspace to the repository working directory of `/app-web`

### [TFE] Add Variables
Enter the following into the Variables section.  Your values will differ, but use those values that were in your `terraform.tfvars` file from previous labs.

```sh
ami                    = ami-03e33c1cefd1d3d74
subnet_id              = subnet-0a5e93f323f7f9138
identity               = terraform-training-ant
vpc_security_group_ids = ["sg-02713b4780094ac55"]
web_count              = 2
```

![](img/tfe-wrkspace-vars.png)


### [TFE] Add Environment Variables

Enter the following into the Environment Variables section:

```sh
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=
```

> For the AWS_SECRET_ACCESS_KEY environment variable, be sure to check the box "sensitive".

![](img/tfe-env-vars.png)

### [TFE] Plan and Apply

Queue another plan, if all looks good, Apply!

### [TFE] Destroy

In your 'app-web' Workspace, navigate to the Variables page.

Add a new Environment Variable: "CONFIRM_DESTROY = 1"

Click the Workspace "Setting" menu -> "Destruction and Deletion"

![](img/tfe-destroy-menu.png)

Click the "Queue destroy plan" to queue a destructive plan.

![](img/tfe-destroy-queue.png)

If the destroy plan looks good, apply it.


## Resources

[Random Terraform Provider](https://registry.terraform.io/providers/hashicorp/random)
