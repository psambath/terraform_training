# Lab 3: Template Creation using Packer

Duration: 30 minutes

### Creating a Packer Image

To build an image packer utilizes a JSON file with the following sections...

##### [Builders](https://www.packer.io/docs/builders/index.html) (required)
* responsible for creating machines and generating images from them for various platforms.
* You can have multiple builder types in one file.

Below is an example of a basic builder for AWS EBS AMI.
Create a new json file called `web-vistors.json` with the following builder.

```json
{
  "variables": {
    "aws_source_ami": "ami-039a49e70ea773ffc"
  },
  "builders": [
    {
      "type": "amazon-ebs",
      "region": "us-east-1",
      "source_ami": "{{user `aws_source_ami`}}",
      "instance_type": "t3.micro",
      "ssh_username": "ubuntu",
      "ssh_pty": "true",
      "ami_name": "tmp-{{timestamp}}",
      "tags": {
        "Created-by": "Packer",
        "OS_Version": "Ubuntu",
        "Release": "Latest"
      }
    }
  ],
  "provisioners": [
    {
      "type": "shell",
      "inline": [
        "mkdir ~/src",
        "cd ~/src",
        "git clone https://github.com/hashicorp/demo-terraform-101.git",
        "cp -R ~/src/demo-terraform-101/assets /tmp",
        "sudo sh /tmp/assets/setup-web.sh"
      ]
    }
  ]
}

```

##### [Variables](https://www.packer.io/docs/templates/user-variables.html)
* User variables allow your templates to be further configured with variables from the command-line, environment variables, Vault, or files.
    * **Note**: these can be definied within the main JSON file and also be passed from an additional variable file, we will cover how to pass those variables further below
    
    
##### [Provisioners](https://www.packer.io/docs/provisioners/index.html)
* use builtin and third-party software to install and configure the machine image after booting. Provisioners prepare the system for use, so common use cases for provisioners include:
    * installing packages 
    * patching 
    * creating users 
    * downloading application code
    
   
##### Running Packer
Once the file is ready we will need to dothe following steps...

1. **packer validate web-vistors.json** - If properly formatted the file will successfully validate
    * This command will work just fine if all the variables are within the main packer file, but if you want to pass user variables from a different file the command will have an additional flag **packer validate web-vistors.json**

Validate your configuration.

```shell
> packer validate web-vistors.json
```

```shell
> packer build web-vistors.json
```

##### Resources
* Packer [Docs](https://www.packer.io/docs/index.html)
* Packer [CLI](https://www.packer.io/docs/commands/index.html)
