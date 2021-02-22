# Lab 6: Provisioners

Duration: 10 minutes

Your workstation has code for provisioning the webapp onto the instance we've
created. In order for this code to be installed onto the, Terraform can connect
to and provision the instance remotely with a provisioner block.

- Task 1: Create a connection block using your keypair module outputs.
- Task 2: Create a provisioner block to remotely download code to your instance.
- Task 3: Apply your configuration and watch for the remote connection.

## Task 1: Create a connection block using your keypair module outputs.

### Step 6.1.1

In `server/server.tf`, add a connection block within your web instance resource under the tags:

```hcl
resource "aws_instance" "web" {
# Leave the first part of the block unchanged, and add a connection block:
# ...

  connection {
    user        = "ubuntu"
    private_key = var.private_key
    host        = self.public_ip
  }
```

**The `private_key` variable was defined in the previous lab.**

The value of `self` refers to the resource defined by the current block. So `self.public_ip` refers to the public IP address of our `aws_instance.web`.

## Task 2: Create a provisioner block to remotely download code to your instance.

### Step 6.2.1

The file provisioner will run after the instance is created and will copy the contents of the _assets_ directory to it. Add this to the _aws_instance_ block in `server/server.tf`, right after the connection block you just added:

```hcl
  provisioner "file" {
    source      = "assets"
    destination = "/tmp/"
  }
```

### Step 6.2.2

The remote-exec provisioner runs remote commands. We can execute the script from
our assets directory. Add this after the _file_ provisioner block:

```hcl
  provisioner "remote-exec" {
    inline = [
      "sudo sh /tmp/assets/setup-web.sh",
    ]
  }
```

Make sure both provisioners are inside the _aws_instance_ resource block.

## Task 3: Apply your configuration and watch for the remote connection.

### Step 6.3.1

An important point to remember regarding _provisioners_ is that adding or removing them from an instance won't cause terraform to update or recreate the instance. You can see this now if you run `terraform apply`:

```text
...

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

...
```

In order to ensure that the provisioners run, use the `terraform taint` command. A resource that has been marked as _tainted_ will be destroyed and recreated.

```shell
terraform taint module.server.aws_instance.web
```

```
Resource instance module.server.aws_instance.web has been marked as tainted.
```

Upon running `terraform apply`, you should see new output:

```shell
terraform apply
```

```text
...
module.server.aws_instance.web: Provisioning with 'remote-exec'...
module.server.aws_instance.web (remote-exec): Connecting to remote host via SSH...
module.server.aws_instance.web (remote-exec):   Host: 18.222.77.212
module.server.aws_instance.web (remote-exec):   User: ubuntu
module.server.aws_instance.web (remote-exec):   Password: false
module.server.aws_instance.web (remote-exec):   Private key: true
module.server.aws_instance.web (remote-exec):   SSH Agent: false
module.server.aws_instance.web (remote-exec): Connected!
module.server.aws_instance.web (remote-exec): Created symlink from /etc/systemd/system/multi-user.target.wants/webapp.service to /lib/systemd/system/webapp.service.
module.server.aws_instance.web: Creation complete after 41s (ID: i-0a869f781ab03b248)

Apply complete! Resources: 1 added, 0 changed, 1 destroyed.
...
```

You can now visit your web application by pointing your browser at the public_dns output for your EC2 instance. If you want, you can also ssh to your EC2 instance with a command like `ssh -i keys/<your_private_key>.pem ubuntu@<your_dns>`.  Type yes when prompted to use the key. Type `exit` to quit the ssh session.

```shell
ssh -i keys/<your_private_key>.pem ubuntu@<your_dns>
```

```text
The authenticity of host 'ec2-54-203-220-176.us-west-2.compute.amazonaws.com (54.203.220.176)' can't be established.
ECDSA key fingerprint is SHA256:Q/IWQRIOcb1h46ic7mHR2oBCk9XOTPHCHQOy0A/pezs.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'ec2-54-203-220-176.us-west-2.compute.amazonaws.com,54.203.220.176' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-1088-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

0 packages can be updated.
0 updates are security updates.

New release '18.04.2 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


Last login: Fri Aug 16 20:58:44 2019 from 34.222.21.235
ubuntu@ip-10-1-1-96:~$ exit
logout
Connection to ec2-54-203-220-176.us-west-2.compute.amazonaws.com closed.
```
