# Lab: For-Each

Duration: 15 minutes

So far, we've already used arguments to configure your resources. These arguments are used by the provider to specify things like the AMI to use, and the type of instance to provision. Terraform also supports a number of _Meta-Arguments_, which changes the way Terraform configures the resources. For instance, it's not uncommon to provision multiple copies of the same resource. We can do that with the _count_ argument.

The count argument does however have a few limitations in that it is entirely dependent on the count index which can be shown by performing a `terraform state list`.

A more mature approach to create multiple instances while keeping code DRY is to leverage Terraform's `for-each`.

- Task 1: Change the number of AWS instances with `count`
- Task 2: Look at the number of AWS instances with `terraform state list`
- Task 3: Decrease the Count and determine which instance will be destroyed.
- Task 4: Refactor code to use Terraform `for-each`
- Task 5: Look at the number of AWS instances with `terraform state list`
- Task 6: Update the server variables to determine which instance will be destroyed.
- Task 7: Update the output variables to pull DNS addresses.

## Task 1: Change the number of AWS instances with `count`

Create a new directory for the lab and add the following `main.tf`.  Also copy your `terraform.tfvars` to this diretory.

Notice the count is set to `2` for the number of `aws_instances`.

```hcl
variable "access_key" {}
variable "secret_key" {}
variable "region" {
  default = "us-east-1"
}
variable "ami" {}
variable "subnet_id" {}
variable "identity" {}
variable "vpc_security_group_ids" {
  type = list
}

resource "aws_instance" "web" {
  count                  = 2
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

output "public_dns" {
  value = aws_instance.web.*.public_dns
}
```

## Task 2: Look at the number of AWS instances with `terraform state list`
Run a `terraform apply` followed by a `terraform state list` to view how the servers are accounted for in Terraform's State.

```bash
terraform state list

aws_instance.web[0]
aws_instance.web[1]
```

## Task 3: Decrease the Count and determine which instance will be destroyed.
Update the count from `2` to `1`

```hcl
resource "aws_instance" "web" {
  count                  = 1
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

Run a `terraform apply` followed by a `terraform state list` to view how the servers are accounted for in Terraform's State.

```
terraform state list

aws_instance.web[0]
```

You will see that when using the `count` parameter you have very limited control as to which server Terraform will destroy.  It will always default to destroying the server with the highest index count.


## Task 4: Refactor code to use Terraform `for-each`
Refactor `main.tf` to make use of the `for-each` command rather then the count command.  Replace the existing code in your `main.tf` with the following:

```hcl
variable "identity" {
  default = "rpt"
}

variable "aws_region" {
  description = "AWS region"
  default     = "us-east-1"
}

variable "servers" {
  description = "Map of server types to configuration"
  type        = map(any)
  default = {
    server-iis = {
      ami                    = "ami-07f5c641c23596eb9"
      instance_type          = "t2.micro",
      environment            = "dev"
      subnet_id              = "subnet-031bf0c9a309fcd8d"
      vpc_security_group_ids = ["sg-01380b40dc19ad166"]
    },
    server-apache = {
      ami                    = "ami-07f5c641c23596eb9"
      instance_type          = "t2.nano",
      environment            = "test"
      subnet_id              = "subnet-031bf0c9a309fcd8d"
      vpc_security_group_ids = ["sg-01380b40dc19ad166"]
    }
  }
}

provider "aws" {
  region = var.aws_region
}

resource "aws_instance" "web" {
  for_each               = var.servers
  ami                    = each.value.ami
  instance_type          = each.value.instance_type
  subnet_id              = each.value.subnet_id
  vpc_security_group_ids = each.value.vpc_security_group_ids

  tags = {
    "Identity"    = var.identity
    "Name"        = each.key
    "Environment" = each.value.environment
  }
}
```

If you run `terraform apply` now, you'll notice that this code will destroy the previous resource and create two new servers based on the attributes defined inside the `servers` variable, which is defined as a map of our servers.


### Task 5: Look at the number of AWS instances with `terraform state list`

```bash
terraform state list

aws_instance.web["server-iis"]
aws_instance.web["server-apache"]
```

Since we used _for-each_ to the aws_instance.web resource, it now refers to multiple resources with key references from the `servers` variable.

## Task 6: Update the server variables to determine which instance will be destroyed.

Update the `servers` variable to remove the `server-iis` instance by removing the following block:

```hcl
    server-iis = {
      ami                    = "ami-07f5c641c23596eb9"
      instance_type          = "t2.micro",
      environment            = "dev"
      subnet_id              = "subnet-031bf0c9a309fcd8d"
      vpc_security_group_ids = ["sg-01380b40dc19ad166"]
    },
```

If you run `terraform apply` now, you'll notice that this code will destroy the `server-iis`, allowing us to target a specific instance that needs to be updated/removed.


### Task 7: Update the output variables to pull DNS addresses.

When using Terraform's `for-each` our output blocks need to be updated to utilize `for` to loop through the server names.  This differs from using `count` which utilized the Terraform splat operator `*`.

```
output "public_dns" {
  description = "Public DNS names of the Servers"
  value = { for p in sort(keys(var.servers)) : p => aws_instance.web[p].public_dns }
}
```

The syntax `aws_instance.web.*` refers to all of the instances, so this will output a list of all of the public IPs and public DNS records. 

