# Lab 10: Terraform Cloud (Optional)
This is an optional lab that you can perform on your own at the end of the Terraform 101 course if you have time.  It uses the [Terraform Cloud Remote Backend](https://app.terraform.io/signup?utm_source=banner&utm_campaign=intro_tf_cloud_remote).

Duration: 10 minutes
- Task 1: Sign up for Terraform Cloud
- Task 2: Update your Terraform configuration to use the remote backend


## Task 1: Sign up for Terraform Cloud

### Step 10.1.1

Navigate to [the sign up page](https://app.terraform.io/signup?utm_source=banner&utm_campaign=intro_tf_cloud_remote) and create an account for Terraform Cloud.

Create an Organization. This can be any name you would like.

### Step 10.1.2

Terraform's CLI needs credentials before it can access Terraform Cloud. Follow these steps to allow Terraform to access your organization.

1. Open your Terraform CLI config file in a text editor; create the file if it doesn't already exist. This file is located at `%APPDATA%\terraform.rc` on Windows systems, and `~/.terraformrc` on other systems.

1. Add the following credentials block to the config file:

```shell
credentials "app.terraform.io" {
     token = "REPLACE_ME"
   }
```

1. Leave your editor open.

1. In your web browser, go to the tokens section of your user settings. Open https://app.terraform.io/app/settings/tokens, or click the user icon in the upper right corner, click "User Settings", then click "Tokens" in the left sidebar.
1. Generate a new token by entering a description and clicking the "Generate token" button. The new token will appear in a text area below the description field.
1. Copy the token to the clipboard.
1. In your text editor, paste the real token into the token argument, replacing the REPLACE_ME placeholder. Save the CLI config file and close your editor.
At this point, Terraform can use Terraform Cloud with any Terraform configuration that has enabled the remote backend.



## Task 2: Update your remote backend in the your Terraform config

### Step 10.2.1

Navigate to `main.tf` where we created our resources for this training. Add the following to the beginning of the file:


```shell
terraform {
  backend "remote" {
    organization = "<ORGANIZATION NAME>"

    workspaces {
      name = "<WORKSPACE NAME>"
    }
  }
}
```

Replace the organization with the name you chose in the previous step.

The workspace name is arbitrary, since Terraform Cloud creates workspaces on demand; if a workspace with this name doesn't yet exist, it will be automatically created the next time you run terraform init for that configuration.

### Step 10.2.2

Run `terraform init`.

```shell
> terraform init
Initializing modules...

Initializing the backend...

Successfully configured the backend "remote"! Terraform will automatically
use this backend unless the backend configuration changes.

Initializing provider plugins...
- Checking for available provider plugins...
- Downloading plugin for provider "aws" (terraform-providers/aws) 2.18.0...

The following providers do not have any version constraints in configuration,
so the latest version was installed.

To prevent automatic upgrades to new major versions that may contain breaking
changes, it is recommended to add version = "..." constraints to the
corresponding provider blocks in configuration, with the constraint strings
suggested below.

* provider.aws: version = "~> 2.18"

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

### Step 10.2.3

Create a new tag and run `terraform apply` and go to the Workspace you created. You should see state information there.

For more information on Terraform Cloud, check out the [Learn site](https://learn.hashicorp.com/terraform/?track=cloud#cloud).