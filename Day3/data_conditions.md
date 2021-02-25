# Lab: Data Resources and Conditions

Duration: 15 minutes

So far, we've already used arguments to configure your resources. These arguments are used by the provider to specify things like the AMI to use, and the type of instance to provision.

Terraform also supports the ability to look up information through the use of `data` resources.

For instance, it's not uncommon to need to query the value of an AMI type within a regions for an EC2 instance.  AMIs are specific to each region therefore it is often more vaulable to query for this value rather then specifying it in a variable.  This is where the use of a `data` resource block within HCL is convienent.

To expand on this process we will also introduce the Conditional Expression, allowing us to determine if we want to pull a Linux or Windows image.


- Task 1: Query the AMI Image for Ubuntu and Windows using a Terraform `data` resource.
- Task 2: Look at the returned value of the data resources using a `terraform state list` and `terraform show` command.
- Task 3: Leverage Terraform's interpolation to specify the AMI for our `aws_instance`
- Task 4: Use Terraform conditional operators to determine the image type of our `aws_instance`

## Task 1: Query the AMI Image for Ubuntu and Windows using a Terraform `data` resource.

Create a new directory for the lab and add a `data.tf` file with the following items:

```hcl
data "aws_ami" "ubuntu_16_04" {
  most_recent = true

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-xenial-16.04-amd64-server-*"]
  }

  owners = ["099720109477"]
}

data "aws_ami" "ubuntu_18_04" {
  most_recent = true

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-bionic-18.04-amd64-server-*"]
  }

  owners = ["099720109477"]
}
```

### Task 2: Look at the returned value of the data resources using a `terraform state list` and `terraform show` command.

Run a `terraform apply` to query the data items specified and view those items by issuing a `terraform state list`.


### Task 3: Leverage Terraform's interpolation to specify the AMI for our `aws_instance`

Add a `main.tf` in the same directory as the `data.tf` and the following code to that file. Also copy your `terraform.tfvars` to this directory.

```hcl
resource "aws_instance" "web" {
  ami = (var.ubuntu_version == "18") ? data.aws_ami.ubuntu_18_04.image_id : data.aws_ami.ubuntu_16_04.image_id
  instance_type          = "t2.micro"
  subnet_id              = var.subnet_id
  vpc_security_group_ids = var.vpc_security_group_ids
  key_name               = var.key_name

  connection {
    user        = "ubuntu"
    private_key = var.private_key
    host        = self.public_ip
  }

  provisioner "file" {
    source      = "assets"
    destination = "/tmp/"
  }

  provisioner "remote-exec" {
    inline = [
      "sudo sh /tmp/assets/setup-web.sh",
    ]
  }
  
  tags = {
    "Identity"    = var.identity
    "Name"        = "Student"
    "Environment" = "Training"
  }
}
```

Create a `variables.tf` in the same directory and add the following.

```hcl
variable subnet_id {}
variable vpc_security_group_ids {
  type = list
}
variable identity {}
variable key_name {}
variable private_key {}

variable ubuntu_version {
    type = string
    description = "Ubuntu Version.  Example: 16, 18"
    default = 18
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

When using Terraform's `for-each` our output blocks need to be updated to utilize `for` to loop through the server names.  This differs from using `count` which utilized the Terraform splat operator `*`.  Add the following output block to your `main.tf`.

```hcl
output "public_dns" {
  description = "Public DNS names of the Servers"
  value = { for p in sort(keys(var.servers)) : p => aws_instance.web[p].public_dns }
}
```
