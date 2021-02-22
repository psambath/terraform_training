# Lab 10: Destroy

Duration: 5 minutes

You've now learned about all of the key features of Terraform, and how to use it
to manage your infrastructure. The last command we'll use is `terraform
destroy`. As you might guess from the name, this will destroy all of the
infrastructure managed by this configuration.

- Task 1: Destroy your infrastructure

## Task 1: Destroy your infrastructure

### Step 10.1.1

Run the command `terraform destroy`:

```shell
terraform destroy
```

```text
# ...
module.keypair.tls_private_key.generated: Refreshing state... [id=6dd9f3893ed2be3206ffc993bf7e1173f5d7db76]
module.keypair.local_file.private_key_pem[0]: Refreshing state... [id=a10696fac0211b55ab3076a06e09d0820a57b9fa]
module.keypair.local_file.public_key_openssh[0]: Refreshing state... [id=9b60deb2581eaf3e417a7a7001a460f088d0b917]
module.keypair.null_resource.chmod[0]: Refreshing state... [id=6079893051072852963]
module.keypair.aws_key_pair.generated: Refreshing state... [id=robin-training-env-ant-key]
module.server.aws_instance.web[0]: Refreshing state... [id=i-05d32b2b6f6783ec7]
module.server.aws_instance.web[1]: Refreshing state... [id=i-0f8ce7aec8e54199d]

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
 - destroy

Terraform will perform the following actions:

 # module.keypair.aws_key_pair.generated will be destroyed

# ...

Plan: 0 to add, 0 to change, 7 to destroy.

Do you really want to destroy all resources?
 Terraform will destroy all your managed infrastructure, as shown above.
 There is no undo. Only 'yes' will be accepted to confirm.

 Enter a value: yes

module.keypair.null_resource.chmod[0]: Destroying... [id=6079893051072852963]
module.keypair.null_resource.chmod[0]: Destruction complete after 0s
module.keypair.local_file.public_key_openssh[0]: Destroying... [id=9b60deb2581eaf3e417a7a7001a460f088d0b917]

# ...

module.keypair.tls_private_key.generated: Destruction complete after 0s

Destroy complete! Resources: 7 destroyed.
```

You'll need to confirm the action by responding with `yes`. You could achieve
the same effect by removing all of your configuration and running `terraform
apply`, but you often will want to keep the configuration, but not the
infrastructure created by the configuration.
