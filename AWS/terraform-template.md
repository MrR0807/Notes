# VPC

## Custom VPC

```tf
provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "a4l_vpc" {
  cidr_block = "10.16.0.0/16"
  assign_generated_ipv6_cidr_block = true
  enable_dns_hostnames = true
}
```
