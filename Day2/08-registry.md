# Lab 8: Private Module Registry

Duration: 20 minutes

This lab demonstrates how to publish modules to the private module registry.

- Task 1: Fork a repository on GitHub and add it to your private module repository
- Task 2: Configure the module with the web UI

## Prerequisites

For this lab, we'll assume that you've installed [Terraform](https://www.terraform.io/downloads.html) and that you have a [GitHub](https://github.com) account.

## Task 1: Fork a repository on GitHub and add it to your private module repository

For this task, you'll fork an existing GitHub repository that is pre-built to work as a module. You'll import it to your private module repository.

### Step 8.1.1: Fork on GitHub

Go to GitHub and fork the `terraform-demo-animal` repository to your own account.

```
https://github.com/hashicorp/terraform-demo-animal/
```

### Step 8.1.2: Import to Terraform Cloud

Go to Terraform Cloud and click the "Modules" button in the top menu.

Click the "+ Add Module" button on the right. You'll see a form where you can choose your VCS provider (use GitHub) and a field to find your fork of the `terraform-demo-animal` repository.

Click "Publish Module" to add it to your Terraform Cloud organization.

You can now view the details of the module.

## Task 2: Configure the module with the web UI

The Terraform Cloud module configuration designer supports a producer/consumer pattern where some teams create modules and other teams use them to create infrastructure. You'll use the configuration designer to generate code that can be copy-and-pasted into a new Terraform project.

### Step 8.2.1: Launch the configuration designer

Start by either clicking the "Open in Configuration Designer" button under the right hand code snippet, or go back to the organization dashboard and click the "+ Design Configuration" button.

![Module Design Configuration](images/module-design-configuration.png "Module Design Configuration")

In either case, you'll see a screen with a list of modules. Click the "Add Module" button on the `animal` module that you imported.

### Step 8.2.2: Configure variables

Click the green "Next" button to proceed to the configuration screen. You'll see a list of variables, a description of each, and an input field where you can type a value for the variable.

![Module Variables](images/module-variables.png "Module Variables")

Type any name into the name field, such as "web".

Click the large green "Next" button on the top right.

You'll be taken to a screen where you can preview the generated code, or download it as a file (it will be named `main.tf`).

In a production scenario, you would save this file to a new or existing Terraform project, add it to a repository in your source code control system, and connect the repository to Terraform Cloud for provisioning.
