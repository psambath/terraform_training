# Lab 4: Variables

Duration: 15 minutes

We don't want to hard code all of our values in the main.tf file. We can create a variable file for easier use.

- Task 1: Create variables in a configuration block
- Task 2: Interpolate those variables
- Task 3: Create a terraform.tfvars file

## Task 1: Create a new configuration block for variables

### Step 4.1.1

Add three variables at the top of your configuration file:

```hcl
variable "access_key" {}
variable "secret_key" {}
variable "region" {
  default = "<REGION>"
}
```

## Task 2: Interpolate those variables into your existing code

### Step 4.2.1

Update the _provider_ block to replace the hard-coded values with the new
variables.

```hcl
provider "aws" {
  access_key = var.access_key
  secret_key = var.secret_key
  region     = var.region
}
```

### Step 4.2.2

Rerun `terraform plan` for Terraform to pick up the new variables. Notice that you will be prompted for input now.

```shell
terraform plan
```

```text
var.access_key
  Enter a value:
```

Use Ctrl+C to exit out of this plan.

## Task 3: Edit your terraform.tfvars file

Variables only defined in the configuration file will be prompted for input. We
can avoid this by creating a variables file. You'll find a file called
`terraform.tfvars` in your Terraform directory with several commented lines.

### Step 4.3.1

Edit the `terraform.tfvars` file by uncommenting the first two key-value pairs:

```
access_key = "<ACCESSKEY>"
secret_key = "<SECRETKEY>"

...
```

Rerun `terraform plan` and notice that this no longer prompts you for input.

### Step 4.3.2

You'll see that there are several other values in `terraform.tfvars`. Uncomment them and add matching variables to your `main.tf` file:

```hcl
variable "access_key" {}
variable "secret_key" {}
variable "region" {
  default = "<REGION>"
}
variable "ami" {}
variable "subnet_id" {}
variable "identity" {}
variable "vpc_security_group_ids" {
  type = list
}
```

### Step 4.3.3

Edit your resource block with these changes as well.

```hcl
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

After making these changes, rerun `terraform plan`. You should see that there
are no changes to apply, which is correct, since the variables contain the same
values we had previously hard-coded:

```text
...

No changes. Infrastructure is up-to-date.

This means that Terraform did not detect any differences between your
configuration and real physical resources that exist. As a result, no
actions need to be performed.
```
