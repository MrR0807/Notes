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

## Implement multi-tier VPC subnets

```tf
provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "a4l_vpc" {
  cidr_block = "10.16.0.0/16"
  assign_generated_ipv6_cidr_block = true
  enable_dns_hostnames = true

  tags = {
    Name = "A4L"
  }
}

# AZ A

resource "aws_subnet" "sn-reserved-a" {
  vpc_id = aws_vpc.a4l_vpc.id
  cidr_block = cidrsubnet(aws_vpc.a4l_vpc.cidr_block, 4, 0)
  availability_zone = "us-east-1a"

  ipv6_cidr_block = cidrsubnet(aws_vpc.a4l_vpc.ipv6_cidr_block, 8, 0)
  assign_ipv6_address_on_creation = true

  tags = {
    Name = "sn-reserved-a"
  }
}

resource "aws_subnet" "sn-db-a" {
  vpc_id = aws_vpc.a4l_vpc.id
  cidr_block = cidrsubnet(aws_vpc.a4l_vpc.cidr_block, 4, 1)
  availability_zone = "us-east-1a"

  ipv6_cidr_block = cidrsubnet(aws_vpc.a4l_vpc.ipv6_cidr_block, 8, 1)
  assign_ipv6_address_on_creation = true

  tags = {
    Name = "sn-db-a"
  }
}

resource "aws_subnet" "sn-app-a" {
  vpc_id = aws_vpc.a4l_vpc.id
  cidr_block = cidrsubnet(aws_vpc.a4l_vpc.cidr_block, 4, 2)
  availability_zone = "us-east-1a"

  ipv6_cidr_block = cidrsubnet(aws_vpc.a4l_vpc.ipv6_cidr_block, 8, 2)
  assign_ipv6_address_on_creation = true

  tags = {
    Name = "sn-app-a"
  }
}

resource "aws_subnet" "sn-web-a" {
  vpc_id = aws_vpc.a4l_vpc.id
  cidr_block = cidrsubnet(aws_vpc.a4l_vpc.cidr_block, 4, 3)
  availability_zone = "us-east-1a"

  ipv6_cidr_block = cidrsubnet(aws_vpc.a4l_vpc.ipv6_cidr_block, 8, 3)
  assign_ipv6_address_on_creation = true

  tags = {
    Name = "sn-web-a"
  }
}

# AZ B

resource "aws_subnet" "sn-reserved-b" {
  vpc_id = aws_vpc.a4l_vpc.id
  cidr_block = cidrsubnet(aws_vpc.a4l_vpc.cidr_block, 4, 4)
  availability_zone = "us-east-1b"

  ipv6_cidr_block = cidrsubnet(aws_vpc.a4l_vpc.ipv6_cidr_block, 8, 4)
  assign_ipv6_address_on_creation = true

  tags = {
    Name = "sn-reserved-b"
  }
}

resource "aws_subnet" "sn-db-b" {
  vpc_id = aws_vpc.a4l_vpc.id
  cidr_block = cidrsubnet(aws_vpc.a4l_vpc.cidr_block, 4, 5)
  availability_zone = "us-east-1b"

  ipv6_cidr_block = cidrsubnet(aws_vpc.a4l_vpc.ipv6_cidr_block, 8, 5)
  assign_ipv6_address_on_creation = true

  tags = {
    Name = "sn-db-b"
  }
}

resource "aws_subnet" "sn-app-b" {
  vpc_id = aws_vpc.a4l_vpc.id
  cidr_block = cidrsubnet(aws_vpc.a4l_vpc.cidr_block, 4, 6)
  availability_zone = "us-east-1b"

  ipv6_cidr_block = cidrsubnet(aws_vpc.a4l_vpc.ipv6_cidr_block, 8, 6)
  assign_ipv6_address_on_creation = true

  tags = {
    Name = "sn-app-b"
  }
}

resource "aws_subnet" "sn-web-b" {
  vpc_id = aws_vpc.a4l_vpc.id
  cidr_block = cidrsubnet(aws_vpc.a4l_vpc.cidr_block, 4, 7)
  availability_zone = "us-east-1b"

  ipv6_cidr_block = cidrsubnet(aws_vpc.a4l_vpc.ipv6_cidr_block, 8, 7)
  assign_ipv6_address_on_creation = true

  tags = {
    Name = "sn-web-b"
  }
}

# AZ C

resource "aws_subnet" "sn-reserved-c" {
  vpc_id = aws_vpc.a4l_vpc.id
  cidr_block = cidrsubnet(aws_vpc.a4l_vpc.cidr_block, 4, 8)
  availability_zone = "us-east-1c"

  ipv6_cidr_block = cidrsubnet(aws_vpc.a4l_vpc.ipv6_cidr_block, 8, 8)
  assign_ipv6_address_on_creation = true

  tags = {
    Name = "sn-reserved-c"
  }
}

resource "aws_subnet" "sn-db-c" {
  vpc_id = aws_vpc.a4l_vpc.id
  cidr_block = cidrsubnet(aws_vpc.a4l_vpc.cidr_block, 4, 9)
  availability_zone = "us-east-1c"

  ipv6_cidr_block = cidrsubnet(aws_vpc.a4l_vpc.ipv6_cidr_block, 8, 9)
  assign_ipv6_address_on_creation = true

  tags = {
    Name = "sn-db-c"
  }
}

resource "aws_subnet" "sn-app-c" {
  vpc_id = aws_vpc.a4l_vpc.id
  cidr_block = cidrsubnet(aws_vpc.a4l_vpc.cidr_block, 4, 10)
  availability_zone = "us-east-1c"

  ipv6_cidr_block = cidrsubnet(aws_vpc.a4l_vpc.ipv6_cidr_block, 8, 10)
  assign_ipv6_address_on_creation = true

  tags = {
    Name = "sn-app-c"
  }
}

resource "aws_subnet" "sn-web-c" {
  vpc_id = aws_vpc.a4l_vpc.id
  cidr_block = cidrsubnet(aws_vpc.a4l_vpc.cidr_block, 4, 11)
  availability_zone = "us-east-1c"

  ipv6_cidr_block = cidrsubnet(aws_vpc.a4l_vpc.ipv6_cidr_block, 8, 11)
  assign_ipv6_address_on_creation = true

  tags = {
    Name = "sn-web-c"
  }
}
```

