# Lab 1: Lab Setup

Duration: 20 minutes

- Task 1: Connect to the Student Workstation
- Task 2: Verify Terraform installation
- Task 3: Generate your first Terraform Configuration
- Task 4: Use the Terraform CLI to Get Help
- Task 5: Apply and Update your Configuration

## Task 1: Connect to the Student Workstation

In the previous lab, you learned how to connect to your workstation with either VSCode, SSH, or the web-based client.

One you've connected, make sure you've navigated to the `/workstation/terraform` directory. This is where we'll do all of our work for this training.

## Task 2: Verify Terraform installation

### Step 1.2.1

Run the following command to check the Terraform version:

```shell
terraform -version
```

You should see:

```text
Terraform v0.12.6
```

## Task 3: Generate your first Terraform Configuration

### Step 1.3.1

In the /workstation/terraform directory, edit the file titled `main.tf` to create an AWS instance with the following properties available in the comments at the top of the file:

- ami
- subnet_id
- vpc_security_group_ids
- tags.Identity

Your final `main.tf` file should look similar to this with different values:

```hcl
provider "aws" {
  access_key = "<YOUR_ACCESSKEY>"
  secret_key = "<YOUR_SECRETKEY>"
  region     = "<REGION>"
}

resource "aws_instance" "web" {
  ami           = "<AMI>"
  instance_type = "t2.micro"

  subnet_id              = "<SUBNET>"
  vpc_security_group_ids = ["<SECURITY_GROUP>"]

  tags = {
    "Identity" = "<IDENTITY>"
  }
}
```

Don't forget to save the file before moving on!

## Task 4: Use the Terraform CLI to Get Help

### Step 1.4.1

Execute the following command to display available commands:

```shell
terraform -help
```

```text
Usage: terraform [-version] [-help] <command> [args]

The available commands for execution are listed below.
The most common, useful commands are shown first, followed by
less common or more advanced commands. If you're just getting
started with Terraform, stick with the common commands. For the
other commands, please read the help and docs before usage.

Common commands:
    apply              Builds or changes infrastructure
    console            Interactive console for Terraform interpolations
    destroy            Destroy Terraform-managed infrastructure
    env                Workspace management
    fmt                Rewrites config files to canonical format
    get                Download and install modules for the configuration
    graph              Create a visual graph of Terraform resources
    import             Import existing infrastructure into Terraform
    init               Initialize a Terraform working directory
    output             Read an output from a state file
    plan               Generate and show an execution plan

    ...
```
* (full output truncated for sake of brevity in this guide)


Or, you can use short-hand:

```shell
terraform -h
```

### Step 1.4.2

Navigate to the Terraform directory and initialize Terraform
```shell
cd /workstation/terraform
```

```shell
terraform init
```

```text
Initializing provider plugins...
...

Terraform has been successfully initialized!
```

### Step 1.4.3

Get help on the `plan` command and then run it:

```shell
terraform -h plan
```

```shell
terraform plan
```

## Task 5: Apply and Update your Configuration

### Step 1.5.1

Run the `terraform apply` command to generate real resources in AWS

```shell
terraform apply
```

You will be prompted to confirm the changes before they're applied. Respond with
`yes`.

### Step 1.5.2

Use the `terraform show` command to view the resources created and find the IP address for your instance.

Ping that address to ensure the instance is running.

### Step 1.5.3

Terraform can perform in-place updates on your instances after changes are made to the `main.tf` configuration file.

Add two tags to the AWS instance:

- Name
- Environment

```hcl
  tags = {
    "Identity"    = "..."
    "Name"        = "Student"
    "Environment" = "Training"
  }
```

### Step 1.5.4

Plan and apply the changes you just made and note the output differences for additions, deletions, and in-place changes.

```shell
terraform apply
```

You should see output indicating that _aws_instance.web_ will be modified:

```text
...

# aws_instance.web will be updated in-place
~ resource "aws_instance" "web" {

...
```

When prompted to apply the changes, respond with `yes`.
