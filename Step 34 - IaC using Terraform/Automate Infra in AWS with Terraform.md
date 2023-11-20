# Automate Infrastructure with IaC Using Terraform Part 1

In this project we will start building the AWS infrastructure with the power of Infrastructure as Code (IaC) using Terraform. 

STEP 1: Create the Provider and VPC resource section
```yml
provider "aws" {
  region = "us-east-1"
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = "172.16.0.0/16"
  enable_dns_support             = "true"
  enable_dns_hostnames           = "true"
  enable_classiclink             = "false"
  enable_classiclink_dns_support = "false"
}
```
- Execute the command `terraform init` to download necessary plugins for Terraform to work. These plugins are used by `providers` and `provisioners`. At this stage, we only have `provider` in our `main.tf` file. So, Terraform will just download plugin for AWS provider.
  
    ![Alt text](<Terraform Code/Screenshot/terraform_init.jpg>)



  - A new directory has been created: `.terraform\....` This is where Terraform keeps plugins. Generally, it is safe to delete this folder. It just means that you must execute     `terraform init` again, to download them.
- Execute the `terraform plan` to see what terraform intends to create before executing `terraform apply` command to create it.
  ![Alt text](<Terraform Code/Screenshot/terraform_plan.jpg>)

  ![Alt text](<Terraform Code/Screenshot/terraform_plan_1.jpg>)

  - A new file is created `terraform.tfstate`. This is how Terraform keeps itself up to date with the exact state of the infrastructure. It reads this file to know what already exists, what should be added, or destroyed based on the entire terraform code that is being developed.
  - Another file gets created during planning and apply but this file gets deleted immediately. `terraform.tfstate.lock.info` is what Terraform uses to track, who is running its code against the infrastructure at any point in time. This is very important for teams working on the same Terraform repository at the same time. The lock prevents a user from executing Terraform configuration against the same infrastructure when another user is doing the same – it allows to avoid duplicates and conflicts.


```yml
# Create public subnets1
    resource "aws_subnet" "public1" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.0.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "us-east-1a"

}
# Create public subnet2
    resource "aws_subnet" "public2" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.1.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "us-east-1b"
}
```
![Alt text](<Terraform Code/Screenshot/VPC.jpg>)
![Alt text](<Terraform Code/Screenshot/Subnets.jpg>)


## Fixing the Problems by Code Reafactoring
So far we have used hard coded values and multiple resource blocks to create the resources. Now, let's fix this problem by refactoring.

- We will use variable and fix the hard coded values.
    ```yml
    variable "region" {
    default = "eu-central-1"
    }

    provider "aws" {
    region = var.region
    }
    variable "region" {
    default = "eu-central-1"
    }

    variable "vpc_cidr" {
    default = "172.16.0.0/16"
    }

    variable "enable_dns_support" {
    default = "true"
    }

    variable "enable_dns_hostnames" {
    default = "true"
    }

    variable "enable_classiclink" {
    default = "false"
    }

    variable "enable_classiclink_dns_support" {
    default = "false"
    }

    provider "aws" {
    region = var.region
    }

    # Create VPC
    resource "aws_vpc" "main" {
    cidr_block                     = var.vpc_cidr
    enable_dns_support             = var.enable_dns_support
    enable_dns_hostnames           = var.enable_dns_support
    enable_classiclink             = var.enable_classiclink
    enable_classiclink_dns_support = var.enable_classiclink

    }
    ```
    - Here, we started with the provider block, declare a variable named `region` and `vpc_cidr`, give it a default value, and update the provider section by referring to the declared variable.
- We will use Loops & Data sources concepts to fix the multiple resource blocks.

    ```yml
    # Get list of availability zones
    data "aws_availability_zones" "available" {
    state = "available"
    }
    # Create public subnet1
    resource "aws_subnet" "public" {
    count                   = 2
    vpc_id                  = aws_vpc.main.id
    cidr_block              = "172.16.1.0/24"
    map_public_ip_on_launch = true
    availability_zone       = data.aws_availability_zones.available.names[count.index]
    }
    ```
- Let's make the `cidr_block` dynamic using `cidrsubnet()` function.

    ```yml
    # Create public subnet1
    resource "aws_subnet" "public" {
    count                   = 2
    vpc_id                  = aws_vpc.main.id
    cidr_block              = cidrsubnet(var.vpc_cidr, 4, count.index)
    map_public_ip_on_launch = true
    availability_zone       = data.aws_availability_zones.available.names[count.index]
    }
    ```
    Its parameters are `cidrsubnet(prefix, newbits, netnum)`
    - The `prefix` parameter must be given in CIDR notation, same as for VPC.
    - The `newbits` parameter is the number of additional bits with which to extend the prefix.
    - The `netnum` parameter is a whole number that can be represented as a binary integer with no more than newbits binary digits, which will be used to populate the additional bits added to the prefix

- The final problem to solve is removing hard coded count value.
    ```yml
    # Create public subnet1
    resource "aws_subnet" "public" {
    count                   = length(data.aws_availability_zones.available.names)
    vpc_id                  = aws_vpc.main.id
    cidr_block              = cidrsubnet(var.vpc_cidr, 4, count.index)
    map_public_ip_on_launch = true
    availability_zone       = data.aws_availability_zones.available.names[count.index]
    }
    ```
    - What we have now, is sufficient to create the subnet resource required. But if you observe, it is not satisfying our business requirement of just 2 subnets. The length function will return number 3 to the count argument, but what we actually need is 2.
    - Declare a variable to store the desired number of public subnets, and set the default value
    ```yml
    variable "preferred_number_of_public_subnets" {
    default = 2
    }
    # Create public subnets
    resource "aws_subnet" "public" {
    count                   = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets
    vpc_id                  = aws_vpc.main.id
    cidr_block              = cidrsubnet(var.vpc_cidr, 4, count.index)
    map_public_ip_on_launch = true
    availability_zone       = data.aws_availability_zones.available.names[count.index]

    }
    ```
    Let's break it down:
	- The first part `var.preferred_number_of_public_subnets == null` checks if the value of the variable is set to null or has some value defined.
	- The second part `?` and `length(data.aws_availability_zones.available.names)` means, if the first part is true, then use this. In other words, if preferred number of public subnets is null (Or not known) then set the value to the data returned by length function.
    - The third part `:` and `var.preferred_number_of_public_subnets` means, if the first condition is false, i.e preferred number of public subnets is not null then set the value to whatever is definied in `var.preferred_number_of_public_subnets`

## Introducing `variables.tf` & `terraform.tfvars`

Instead of having a long list of variables in `main.tf` file, we can actually make our code a lot more readable and better structured by moving out some parts of the configuration content to other files.
- We will put all variable declarations in a separate file and provide non default values to each of them
    - Create a new file and name it `variables.tf`
      - Copy all the variable declarations into the new file.
    - Create another file, name it `terraform.tfvars`
      - Set values for each of the variables.

### main.tf 
![Alt text](<Terraform Code/Screenshot//main.jpg>)

### variables.tf 
![Alt text](<Terraform Code/Screenshot/variables.jpg>)

### terraform.tfvars
![Alt text](<Terraform Code/Screenshot/tfvars.jpg>)




