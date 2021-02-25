# Lab: Data Resources and Conditions

Duration: 15 minutes

So far, we've already used arguments to configure your resources. These arguments are used by the provider to specify things like the AMI to use, and the type of instance to provision.

Terraform also supports the ability to look up information through the use of `data` resources.

For instance, it's not uncommon to need to query the value of an AMI type within a regions for an EC2 instance.  AMIs are specific to each region therefore it is often more vaulable to query for this value rather then specifying it in a variable.  This is where the use of a `data` resource block within HCL is convienent.

To expand on this process we will also introduce the Conditional Expression, allowing us to determine if we want to pull a Linux or Windows image.


- Task 1: Query the AMI Image for Ubuntu and Windows using a Terraform `data` resource.
- Task 2: Look at the returned value of the data resources using a `terraform state list` and `terraform show` command.
- Task 3: Leverage Terraform's interpolation to specify the AMI for our `aws_instance`
- Task 4: Use Terraform conditional operators to determine the image/operating system of our `aws_instance`

## Task 1: Query the AMI Image for Ubuntu and Windows using a Terraform `data` resource.

Create a new directory for the lab and add a `data.tf` file with the following items.  Also copy your `terraform.tfvars` to this directory.

```hcl
data "aws_ami" "windows" {
  most_recent = true
  filter {
    name   = "name"
    values = ["Windows_Server-2019-English-Full-Base-*"]
  }
  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
  owners = ["801119661308"]
}

data "aws_ami" "ubuntu" {
  most_recent = true

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-bionic-18.04-amd64-server-*"]
  }

  owners = ["099720109477"]
}
```

### Task 2: Look at the returned value of the data resources using a `terraform state list` and `terraform show` command.

Run a `terraform init` followed by a `terraform apply` to query the data items specified and view those items by issuing a `terraform state list`.

```bash
terraform state list

data.aws_ami.windows
data.aws_ami.ubuntu
```


```bash
terraform state show data.aws_ami.ubuntu
```

```bash
# data.aws_ami.ubuntu:
data "aws_ami" "ubuntu" {
    architecture          = "x86_64"
    arn                   = "arn:aws:ec2:us-east-1::image/ami-02fe94dee086c0c37"
    block_device_mappings = [
        {
            device_name  = "/dev/sda1"
            ebs          = {
                "delete_on_termination" = "true"
                "encrypted"             = "false"
                "iops"                  = "0"
                "snapshot_id"           = "snap-074c9b6e7aeb0e066"
                "throughput"            = "0"
                "volume_size"           = "8"
                "volume_type"           = "gp2"
            }
            no_device    = ""
            virtual_name = ""
        },
#...
```

### Task 3: Leverage Terraform's interpolation to specify the AMI for our `aws_instance`
Add a `main.tf` in the same directory as the `data.tf` and the following code to that file.

```hcl
resource "aws_instance" "web" {
  ami                    = data.aws_ami.ubuntu.image_id
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

Create a `variables.tf` in the same directory and add the following.

```hcl
variable subnet_id {}
variable vpc_security_group_ids {
  type = list
}
variable identity {}
```

Run a `terraform apply` to build out the EC2 instance using the AMI that was looked up from the data lookup.

Run a `terraform state show aws_instance.web` to see that the server was build using the AMI from the data interpolation.

```bash
terraform state show aws_instance.web

# aws_instance.web:
resource "aws_instance" "web" {
    ami                          = "ami-02fe94dee086c0c37"
```

### Task 4: Use Terraform conditional operators to determine the image operating system for our `aws_instance`

A conditional expression uses the value of a bool expression to select one of two values.  The syntax of a conditional expression is as follows:

```hcl
condition ? true_val : false_val
```

We create and utilize a variable to determine which AMI will be used via a conditional expression.  Update your `variables.tf` to add a new variable for `server_os`

```hcl
variable server_os {
    type = string
    description = "Server Operating System"
    default = "ubuntu"
}
```

Update your `main.tf` to include a conditional operator for selecting an AMI based on the value of the `server_os` variable.

```
resource "aws_instance" "web" {
  ami                    = (var.server_os == "ubuntu") ? data.aws_ami.ubuntu.image_id : data.aws_ami.windows.image_id
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

Change the value of your `server_os` variable to now specify `windows` and the conditional operator will query for the Windows AMI using the `data` interpolation.
