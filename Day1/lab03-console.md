# Lab 3: Terraform Console

Duration: 10 minutes

Terraform configurations and commands often use [expressions](https://www.terraform.io/docs/configuration/expressions.html) like `aws_instance.web.ami` to reference Terraform resources and their attributes.

Terraform includes an interactive console for evaluating expressions against the current Terraform state. This is especially useful for checking values while editing configurations.

- Task 1: Use `terraform console` to query specific instance information.

## Task 1: Use `terraform console` to query specific instance information.

### Step 3.1.1

Find the AMI ID of your instance

```shell
terraform console
```

```
> aws_instance.web.ami
ami-0f9cf087c1f27d9b1
```

Control+C exits the Terraform console

### Step 3.1.2

You can also pipe query information to the stdin of the console for evaluation

```shell
echo "aws_instance.web.ami" | terraform console
```

```
ami-0f9cf087c1f27d9b1
```
