# Lab 8: Meta-Arguments

Duration: 10 minutes

So far, we've already used arguments to configure your resources. These arguments are used by the provider to specify things like the AMI to use, and the type of instance to provision. Terraform also supports a number of _Meta-Arguments_, which changes the way Terraform configures the resources. For instance, it's not uncommon to provision multiple copies of the same resource. We can do that with the _count_ argument.

- Task 1: Change the number of AWS instances with `count`
- Task 2: Modify the rest of the configuration to support multiple instances
- Task 3: Add variable interpolation to the Name tag to count the new instance

## Task 1: Change the number of AWS instances with `count`

### Step 8.1.1

Add a count argument to the AWS instance in `server/server.tf` with a value of 2:

```hcl
# ...
resource "aws_instance" "web" {
  count                  = 2
  ami                    = var.ami
  instance_type          = "t2.micro"
# ... leave the rest of the resource block unchanged...
}
```

## Task 2: Modify the rest of the configuration to support multiple instances

### Step 8.2.1

If you run `terraform apply` now, you'll get an error. Since we added _count_ to the aws_instance.web resource, it now refers to multiple resources. Because of this, values like `aws_instance.web.public_ip` no longer refer to the public_ip of a single resource. We need to tell terraform which resource we're referring to.

To do so, modify the output blocks in `server/server.tf` as follows:

```
output "public_ip" {
  value = aws_instance.web.*.public_ip
}

output "public_dns" {
  value = aws_instance.web.*.public_dns
}
```

The syntax `aws_instance.web.*` refers to all of the instances, so this will output a list of all of the public IPs and public DNS records. 

### Step 8.2.2

Run `terraform apply` to add the new instance. You should see two IP addresses and two DNS addresses in the outputs.

## Task 3: Add variable interpolation to the Name tag to count the new instances

### Step 8.3.1

Interpolate the count variable by changing the Name tag to include the current
count over the total count. Update `server/server.tf` to add a new variable
definition, and use it:

```hcl
# ...
variable private_key {}
variable num_webs {
  default = "2"
}

resource "aws_instance" "web" {
  count                  = var.num_webs
  ami                    = var.ami
  instance_type          = "t2.micro"
  subnet_id              = var.subnet_id
  vpc_security_group_ids = var.vpc_security_group_ids
  key_name               = var.key_name

  tags = {
    "Identity"    = var.identity
    "Name"        = "Student ${count.index + 1}/${var.num_webs}"
    "Environment" = "Training"
  }

# ...
```

The solution builds on our previous discussion of variables. We must create a
variable to hold our count so that we can reference that count twice in our
resource. Next, we replace the value of the count parameter with the variable
interpolation. Finally, we interpolate the current count (+ 1 because it's
zero-indexed) and the variable itself.

### Step 8.3.2

Run `terraform apply` in the terraform directory. You should see the revised tags that count the instances in the apply log.

```shell
terraform apply
```
