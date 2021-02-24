# Lab 4: Template Provider

Duration: 20 minutes

This lab demonstrates how to define and render a text template with the `templatefile` function.

- Task 1: Create a Terraform configuration that contains a template to be rendered
- Task 2: Use `templatefile` function to render variables into the template
- Task 3: Create S3 bucket and attach policy (advanced/optional)

## Prerequisites

For this lab, we'll assume that you've installed [Terraform](https://www.terraform.io/downloads.html). This example is executed locally and doesn't require any other credentials or other authentication.

## Task 1: Create a Terraform configuration that contains a template to be rendered

You'll create an S3 bucket and then attach the necessary permissions to your user to list, get, and delete objects as well as create and destroy that bucket.

### Step 1.1: Create a Terraform configuration

Create a directory for the Terraform configuration. Create a sub-directory for templates. Create a template file.

```shell
mkdir -p /workstation/terraform/lab_4_s3-demo/templates && cd $_
```

```shell
touch iam_policy.json.tpl
```

The name of the template file is completely up to you. We like to include `tpl` to identify it as a template, but also use a suffix that corresponds to the file type of the contents (such as `json`).

### Step 4.1.2: Write contents of template with some placeholders for variables

In the file we just created, let's create a standard JSON AWS policy file that will allow objects in an S3 bucket to be modified. Our action statement defines the allowed operations and our resource statement defines where those operations can occur. We can define the resource through Terraform interpolation and define those variables outside the template and avoid hardcoding variables into our policies for added usability.

Terraform interpolation syntax is used (such as `"${bucket_name}`). No prefix is needed (`var.` or `data.` or any other).

```json
{
  "Id": "Policy1527877254663",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1527877245190",
      "Action": [
        "s3:CreateBucket",
        "s3:DeleteBucket",
        "s3:DeleteObject",
        "s3:GetObject",
        "s3:ListObjects"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::${bucket_name}"
    }
  ]
}
```

## Task 2: Use `templatefile` function to render variable values into the template

We'll write HCL code to render the template. Let's create a file named `main.tf` in the project folder and reference the template we just created.

```shell
cd ..
```

```shell
touch main.tf
```

### Step 4.2.1: Define the `templatefile` function as a `local`

The local value used here will allow `templatefile` function (created and published by HashiCorp) to be referenced as a variable later.

Pass in the `owner_id` variable.

```bash
locals {
  policy = templatefile("${path.module}/templates/iam_policy.json.tpl", {
    owner_id = var.owner_id
    bucket_name = "${var.owner_id}-${uuid()}"
  })
}
```

We also use `path.module` here for robustness in any module or submodule.

### Step 4.2.2: Pass variables into the template

Create a variable with the value you want to pass into the template. This value could be provided externally, fetched from a data source, or even hard-coded.

Templates do not have access to all variables in a configuration. They must be explicitly passed with the `vars` block.

```bash
variable "owner_id" {
  default = "anaconda"
}

locals {
  policy = templatefile("${path.module}/templates/iam_policy.json.tpl", {
    owner_id = var.owner_id
    bucket_name = "${var.owner_id}-${uuid()}"
  })
}
```

### Step 4.2.3: Render the template into an output

Rendered template content may be used anywhere: as arguments to a module or resource, or executed remotely as a script. In our case, we'll emit it as an output for easy viewing.

The `templatefile` function can be referenced as a `local` :

```bash
output "iam_policy" {
  value = local.policy
}
```

### Step 4.2.4: Run the code

Run the code with `terraform apply` and you will see the rendered template.

```shell
terraform init
```

```shell
terraform apply
```

```
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:

iam_policy = {
  "Id": "Policy1527877254663",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1527877245190",
      "Action": [
        "s3:CreateBucket",
        "s3:DeleteBucket",
        "s3:DeleteObject",
        "s3:GetObject",
        "s3:ListObjects"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::anaconda-4d5f8dd7-ddba-9509-a8ac-289e4a25b616"
    }
  ]
}
```

## Task 3: Create S3 Bucket and Attach Policy

If you have extra time and want to try a more complete, real-world example, you could create an S3 bucket, use the generated `id` in a policy template, and attach the rendered template as a policy.

### Step 4.3.1: Use a real IAM user as the owner

Update the `owner_id` variable to use a real IAM user account name. Your
workstation should have a hostname which includes the name of an animal. An IAM
user has already been created with this animal name, so configure the following,
replacing `<animal>` with the animal name from your workstation:

```bash
variable "owner_id" {
  default = "terraform-training-<animal>"
}
```

### Step 4.3.2: Create an S3 bucket

Declare the `aws` provider and create an S3 bucket with your AWS owner ID.

```bash
provider "aws" {
}

resource "aws_s3_bucket" "bucket1" {
  bucket = "${var.owner_id}-${uuid()}"
  acl = "private"
}
```

### Step 4.3.3: Render the template into a resource policy

The policy must be created using a provider. Create the `aws_iam_policy` and the `aws_iam_policy_attachment`.

```bash
resource "aws_iam_policy" "bucket1"{
  name = "${aws_s3_bucket.bucket1.id}-policy"
  policy = local.policy
}

resource "aws_iam_user_policy_attachment" "attach-policy" {
  user       = var.owner_id
  policy_arn = aws_iam_policy.bucket1.arn
}
```

### Step 4.3.4: Run the code

You should see four operations upon applying this configuration: the policy template data being read, the bucket creation, the policy creation, and the action of attaching the policy to your user.

```shell
terraform init
```

```shell
terraform apply
```

```
An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_iam_policy.bucket1 will be created
  + resource "aws_iam_policy" "bucket1" {
      + arn    = (known after apply)
      + id     = (known after apply)
      + name   = (known after apply)
      + path   = "/"
      + policy = (known after apply)
    }

  # aws_iam_user_policy_attachment.attach-policy will be created
  + resource "aws_iam_user_policy_attachment" "attach-policy" {
      + id         = (known after apply)
      + policy_arn = (known after apply)
      + user       = "tfc-training-201-anaconda"
    }

  # aws_s3_bucket.bucket1 will be created
  + resource "aws_s3_bucket" "bucket1" {
      + acceleration_status         = (known after apply)
      + acl                         = "private"
      + arn                         = (known after apply)
      + bucket                      = (known after apply)
      + bucket_domain_name          = (known after apply)
      + bucket_regional_domain_name = (known after apply)
      + force_destroy               = false
      + hosted_zone_id              = (known after apply)
      + id                          = (known after apply)
      + region                      = (known after apply)
      + request_payer               = (known after apply)
      + website_domain              = (known after apply)
      + website_endpoint            = (known after apply)

      + versioning {
          + enabled    = (known after apply)
          + mfa_delete = (known after apply)
        }
    }

Plan: 3 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_s3_bucket.bucket1: Creating...
aws_s3_bucket.bucket1: Creation complete after 4s [id=tfc-training-201-anaconda-943292c1-6089-7c72-1c41-3503e02fcaca]
aws_iam_policy.bucket1: Creating...
aws_iam_policy.bucket1: Creation complete after 1s [id=arn:aws:iam::130490850807:policy/tfc-training-201-anaconda-943292c1-6089-7c72-1c41-3503e02fcaca-policy]
aws_iam_user_policy_attachment.attach-policy: Creating...
aws_iam_user_policy_attachment.attach-policy: Creation complete after 1s [id=tfc-training-201-anaconda-20190702155915434900000001]

Apply complete! Resources: 3 added, 0 changed, 0 destroyed.

Outputs:

iam_policy = {
  "Id": "Policy1527877254663",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1527877245190",
      "Action": [
        "s3:CreateBucket",
        "s3:DeleteBucket",
        "s3:DeleteObject",
        "s3:GetObject",
        "s3:ListObjects"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::tfc-training-201-anaconda-3ae7bd64-f426-1c70-2e96-e61de9d96878"
    }
  ]
}
```

After completion of this lab, run a `terraform apply` to clean up your work.
