# Lab 1: Null Resource

Duration: 15 minutes

This lab demonstrates the use of the `null_resource`. Instances of `null_resource` are treated like normal resources, but they don't do anything. Like with any other resource, you can configure provisioners and connection details on a null_resource. You can also use its triggers argument and any meta-arguments to control exactly where in the dependency graph its provisioners will run.

- Task 1: Create a AWS Instance using Terraform
- Task 2: Use `null_resource` with a VM to take action with `triggers`.

We'll demonstrate how `null_resource` can be used to take action on a set of existing resources that are specified within the `triggers` argument


## Task 1: Create a AW Instances using Terraform
### Step 16.1.1: Create Server instances

Build the web servers using the AWS Server Modules (previous labs)

You can see this now if you run `terraform apply`:

```text
...

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

...
```


## Task 2: Use `null_resource` with an EC2 instance to take action with `triggers`
### Step 16.2.1: Use `null_resource`

Add `null_resource` stanza to the `server.tf`.  Notice that the trigger for this resource is set to 

```hcl
resource "null_resource" "web_cluster" {
  # Changes to any instance of the cluster requires re-provisioning
  triggers = {
    web_cluster_instance_ids = "${join(",", aws_instance.web.*.id)}"
  }

  # Bootstrap script can run on any instance of the cluster
  # So we just choose the first in this case
  connection {
    host = "${element(aws_instance.web.*.public_ip, 0)}"
  }

  provisioner "local-exec" {
    # Bootstrap script called with private_ip of each node in the clutser
    command = "echo ${join("Nodes of the Cluster : ", aws_instance.web.*.private_ip)}"
  }
}
```
Initialize the configuration with a `terraform init` followed by a `plan` and `apply`.

### Step 16.2.2: Re-run `plan` and `apply` to trigger `null_resource`
After the infrastructure has completed its buildout, re-run a plan and apply and notice if the null resource is triggered.  If you modify the `count` vaule of your `aws_instance` the null resource will be triggered.

```shell
terraform apply
```

Run `apply` a few times to see the `null_resource`.

### Step 16.2.3: Destroy
Finally, run `destroy`.

```shell
terraform destroy
```
