# Lab 2: Outputs

Duration: 10 minutes

Outputs allow us to query for specific values rather than parse metadata in `terraform show`.

- Task 1: Create output variables in your configuration file
- Task 2: Use the output command to find specific variables

## Task 1: Create output variables in your configuration file

### Step 2.1.1

Create two new output variables named "public_dns" and "public_ip" to output the instance's public_dns and public_ip attributes

```hcl
output "public_ip" {
  value = aws_instance.web.public_ip
}

output "public_dns" {
  value = aws_instance.web.public_dns
}
```

### Step 2.1.2

Run the refresh command to pick up the new output

```shell
terraform refresh
```

## Task 2: Use the output command to find specific variables

### Step 2.2.1 Try the terraform output command with no specifications

```shell
terraform output
```

### Step 2.2.2 Query specifically for the public_dns attributes

```shell
terraform output public_dns
```

### Step 2.2.3 Wrap an output query to ping the DNS record

```shell
ping $(terraform output public_dns)
```
