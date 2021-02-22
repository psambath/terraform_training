# Lab 3: Terraform Cloud Basics

Duration: 45 minutes

This lab demonstrates how to connect Terraform Cloud to a source code management system (GitHub) and create a workspace that can apply the Terraform configuration when changes are committed.

- Task 1: Connect GitHub to Terraform Cloud and Fork a GitHub Repo
- Task 2: Create public_key and private_key Variables
- Task 3: Queue a Plan (Warning: it will fail)
- Task 4: Edit Code on GitHub to Use Variables instead of file
- Task 5: Confirm and Apply the Plan

## Prerequisites

For this lab, we'll assume that you've installed [Terraform](https://www.terraform.io/downloads.html) and that you have [signed up](https://app.terraform.io/signup/account) for a trial Terraform Cloud account. You'll also need a [GitHub](https://github.com/) account.

## Task 1: Connect GitHub to Terraform Cloud and Fork a GitHub Repo

Using a GitHub repository will allow us to use source control best practices on our infrastructure configs.

Populating variables to Terraform Cloud will give Terraform Cloud our AWS credentials so it can run Terraform on our behalf.

Connecting Terraform Cloud to GitHub will give us a continuous integration style of workflow for managing infrastructure.

### Step 3.1.1: Fork the repo

Visit this GitHub repository and fork it so you have a copy in your own GitHub account:

    https://github.com/hashicorp/demo-terraform-101

Optionally, clone the repository to your local machine (if you prefer to edit code locally instead of in the browser).

```shell
cd /workstation/terraform
git clone https://github.com/$USER/demo-terraform-101.git
```

We will work with the `after-tfc` branch. If you choose to work locally, check out this branch:

```shell
git checkout -t origin/after-tfc
```

### Step 3.1.2: Connect GitHub to Terraform Cloud and create a workspace

Now go to [app.terraform.io](https://app.terraform.io)

We are going to connect our GitHub repository to Terraform Cloud. If you don’t already have an organization in Terraform Cloud, create one. If you were invited to an existing Terraform Cloud organization, you can access that as well.

You’ll see an empty page where your workplaces will be. Click the "New Workspace" button in the top right.

On the "New Workspace page", select GitHub -> GitHub.com to continue. A new window should open asking you to authorize Terraform Cloud to your GitHub account. If you have not logged into GitHub recently, you may need to login first.

![TFC](images/tfe-basics/01.png "TFC")

Click the green Authorize button to connect Terraform Cloud to your GitHub account.

![TFC](images/tfe-basics/02.png "TFC")

Next, you will see a list of your GitHub repositories. Choose the repository you forked in the first step.

![TFC](images/tfe-basics/03.png "TFC")

On the final step, leave the workspace name and "Advanced options" unchanged, and click the purple Create workspace button to create the workspace.

It will take a few minutes for Terraform Cloud to connect to your GitHub repository. Once that's complete, you should see a notification that your configuration was uploaded successfully.

![TFC](images/tfe-basics/04.png "TFC")

### Step 3.1.3: Configure variables

Go to the "Variables" tab.  On the variables page, you'll see there are two kinds of variables:

- Terraform variables: these are fed into Terraform, similar to `terraform.tfvars`
- Environment variables: these are populated in the runtime environment where Terraform executes

In the "Terraform Variables" section, click "Add Variable" and add keys and values for all the variables in the project's `variables.tf` file. You'll need your `identity` which is your unique animal name as well as `ami` which should be `ami-0735ea082a1534cac`.

## Task 2: Create `public_key` and `private_key` Variables in Terraform Cloud

### Step 3.2.1: Configure SSH keys

Terraform Cloud runs in an environment with limited privileges. It does not have access to your instance's SSH keys that were read from local storage when this code was originally written.

Start by copying the contents of the instance's public and private SSH keys to Terraform Cloud as Terraform variables.

**NOTE**: These keys are temporary and are used only for this class. However, you should be careful about ever sending private keys to any website, API, individual, or machine. You could avoid this scenario by building the instance in Packer or by other means.

Here is one way to display them in your terminal in a way that can be copied to your local clipboard.

Use `cat` with each file in the terminal, copy them to the clipboard, and create variables on Terraform Cloud named `public_key` and `private_key`. Be sure to click "Save Variable" after entering each key and value.

```shell
cat ~/.ssh/id_rsa.pub
```

```
ssh-rsa AAAA...
```

You can mark values as sensitive, which makes them "write-only" (they will not be displayed in the UI after they are saved). Be sure to do that at least with your private key.

```shell
cat ~/.ssh/id_rsa
```

```
-----BEGIN RSA PRIVATE KEY-----
MII...
```

You MUST hit "Save Variable". Variables will not be saved otherwise.

### Step 3.2.2: Enter AWS Credentials

There is also a section for environment variables. We'll use these to store AWS credentials.

Add variables for your AWS credentials. You can retrieve those in use by your instance with `echo`:

```shell
echo $AWS_ACCESS_KEY_ID
```

```
AAAA
```

```shell
echo $AWS_SECRET_ACCESS_KEY
```

```
AAAA
```

```shell
echo $AWS_DEFAULT_REGION
```

```
us-east-1
```

Back in Terraform Cloud, create a new Environment Variable for each of the three AWS values; `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and `AWS_DEFAULT_REGION`. Make sure to mark `AWS_SECRET_ACCESS_KEY` as "Sensitive".

Click the "Save Variable" button for each.

## Task 3: Queue a Plan

For this task, you'll queue a `terraform plan`.

### Step 3.3.1: Queue a plan and read the output

Click the "Queue Plan" button at the top right. You'll be asked for a reason, which you can set as anything you like. This will take you to the Run Details page.

Scroll down to where it shows the plan. You’ll see the same kind of output that you are used to seeing on the command line.

We'll make another change from GitHub before running this plan, so click "Discard Run".

## Task 4: Edit Code on GitHub to Upgrade the AWS Provider Version

Edit code on GitHub to upgrade the AWS provider version to `>= 2.27.0`.

You'll make a pull request with these changes and observe the status of the pull request on GitHub.

### Step 3.4.1

On GitHub, find the "Branch" pulldown and switch to the `after-tfc` branch.

Navigate to `main.tf`. Find the pencil icon. Click to edit this file directly in the browser.

![TFC](images/tfe-basics/11.png "TFC")

Edit the code to match the lines below.

```bash
provider "aws" {
  # MODIFY this line to look for 2.27.0 or greater
  version = ">= 2.27.0"
}
```

Scroll to the bottom and select the option to "Create a new branch and start a pull request."

![TFC](images/tfe-basics/12.png "TFC")

You’ll be taken to a screen to create a pull request. Click the green "Propose file change" button. The page will be pre-populated with your commit message. Click "Create pull request."

After a few seconds, you'll see that Terraform Cloud checked the plan and that it passed.

![TFC](images/tfe-basics/14.png "TFC")

To see what was checked, click "Show all checks" and click "Details" next to the line that says "Terraform plan has changes."

Merge the pull request to the `after-tfc` branch with the big green "Merge pull request" button. Click the "Confirm merge" button.

## Task 5: Confirm and Apply the Plan

### Step 3.5.1: Confirm and `apply`

Back at Terraform Cloud, find the "Current Run" tab. Click it and you’ll see the merge commit has triggered a plan and it needs confirmation.

![TFC](images/tfe-basics/15.png "TFC")

After you confirm the `plan`, go to the bottom of the page and click "Confirm & Apply". You’ll be given the chance to leave a comment (optional). Finally, click "Confirm Plan".

![TFC](images/tfe-basics/16.png "TFC")

This will queue a `terraform apply`.

Examine the output of `apply` and find the IP address of the new instance. The output looks like what you’ve previously seen in the terminal. Copy the `public_ip` address and paste it into your browser. You'll see the running web application.

![TFC](images/tfe-basics/17.png "TFC")

## Task 6: Destroy

To clean up, destroy the infrastructure you've just created.

### Step 3.6.1: Configure CONFIRM_DESTROY variable

Go to the "Settings" tab for the workspace in Terraform Cloud and scroll to the bottom. Note the instructions under "Workspace Delete." We want to destroy the infrastructure but not necessarily the workspace.

You'll need to create an environment variable (not a Terraform variable) named `CONFIRM_DESTROY` and set it to `1`.

Go to the "Variables" tab and do that.

![Confirm Destroy](images/confirm-destroy.png "Confirm Destroy variable")

Click "Save Variable".

### Step 3.6.2: Queue destroy plan

Now go back to the "Settings" tab. Scroll to the bottom and click "Queue Destroy Plan." Note the messages under "Plan" that indicate that it will destroy several resources.

Click "Confirm and Apply." After a few seconds, your infrastructure will be destroyed as requested.
