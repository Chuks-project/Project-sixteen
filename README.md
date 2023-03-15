## AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM PART 1

After you have built AWS infrastructure for 2 websites manually as seen in Project 15, it is time to automate the process using Terraform. What this means is that instead of creating our resources by manually clicking on them one after the other, we are simply going to create them by defining or writing them as codes. And this is where the automation comes to play.
IAC can be done with several tools but in this project we made use of Terraform.

### The secrets of writing quality Terraform code

The secret recipe of a successful Terraform projects consists of:

- Your understanding of your goal (desired AWS infrastructure end state)
- Your knowledge of the IaC technology used (in this case – Terraform)
- Your ability to effectively use up to date Terraform documentation

As you go along completing this project, you will get familiar with Terraform-specific terminology, such as:

Attribute
Resource
Interpolations
Argument
Providers
Provisioners
Input Variables
Output Variables
Module
Data Source
Local Values
Backend


Another concept you must know is data type. This is a general programing concept, it refers to how data represented in a programming language and defines how a compiler or interpreter can use the data. Common data types are:

Integer
Float
String
Boolean, etc.

Best practices
Ensure that every resource is tagged using multiple key-value pairs. 


## VPC | SUBNETS | SECURITY GROUPS

- First of all, we will create a folder in our VSC called PBL
- Create a file in the folder, name it main.tf

### Provider and VPC resource section

- Add AWS as a provider, and a resource to create a VPC in the main.tf file.
- Provider block informs Terraform that we intend to build infrastructure within AWS.
- Resource block will create a VPC.

```
provider "aws" {
  region = "eu-central-1"
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
#### Note: You can change the configuration above to create your VPC in other region that is closer to you. The same applies to all configuration snippets that will follow.

- The next thing we need to do, is to download necessary plugins for Terraform to work. These plugins are used by providers and provisioners. At this stage, we only have provider in our main.tf file. So, Terraform will just download plugin for AWS provider.
- Lets accomplish this with terraform init command as seen in the below demonstration.

![erraform init](https://user-images.githubusercontent.com/65022146/225284036-1d79be66-6efc-4bb7-bd4d-6812a9065228.png)

Observations:
Notice that a new directory has been created: .terraform\.... This is where Terraform keeps plugins. Generally, it is safe to delete this folder. It just means that you must execute terraform init again, to download them.

#### Let us create the first 2 public subnets.

- Add below configuration to the main.tf file:

---
##### Create public subnets1
    resource "aws_subnet" "public1" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.0.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "eu-central-1a"
    }
##### Create public subnet2
    resource "aws_subnet" "public2" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.1.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "eu-central-1b"
    }
---


- We are creating 2 subnets, therefore declaring 2 resource blocks – one for each of the subnets.
- We are using the vpc_id argument to interpolate the value of the VPC id by setting it to aws_vpc.main.id. This way, Terraform knows inside which VPC to create the subnet.

##### Next thing you have to do is to run terraform plan and terraform apply
This is where terraform previews what you want to create before creating them as seen in the screenshot below:

![terraform plan 1](https://user-images.githubusercontent.com/65022146/225299491-4d7922ac-7015-4a34-b932-546255cae143.png)
![terraform plan 2](https://user-images.githubusercontent.com/65022146/225299498-037f1aa5-d803-4e19-9de3-731d676ca616.png)



- Terraform Apply

![public subnet created](https://user-images.githubusercontent.com/65022146/225300218-dfdc5857-ead4-41f0-a5db-2ea45f6a4faa.png)



##### Terraform Destroy

- To destroy whatever has been created run terraform destroy command, and type yes after evaluating the plan. This is seen in the creenshot below:

![terraform destroyed](https://user-images.githubusercontent.com/65022146/225300729-bfbbf010-dca5-418d-8d3c-a9a54cb79fd4.png)


#### Observations:

Hard coded values: Remember our best practice hint from the beginning? Both the availability_zone and cidr_block arguments are hard coded. We should always endeavour to make our work dynamic.
Multiple Resource Blocks: Notice that we have declared multiple resource blocks for each subnet in the code. This is bad coding practice. We need to create a single resource block that can dynamically create resources without specifying multiple blocks. Imagine if we wanted to create 10 subnets, our code would look very clumsy. So, we need to optimize this by introducing a count argument.

Now let us improve our code by refactoring it.

## FIXING THE PROBLEMS BY CODE REFACTORING

- Fixing Hard Coded Values: We will introduce variables, and remove hard coding.

- Starting with the provider block, declare a variable named region, give it a default value, and update the provider section by referring to the declared variable.

```
variable "region" {
        default = "eu-central-1"
    }

    provider "aws" {
        region = var.region
    }
```

- Do the same to cidr value in the vpc block, and all the other arguments.

```
 variable "region" {
        default = "eu-west-1"
    }

    variable "vpc_cidr" {
        default = "172.16.0.0/16"
    }

    variable "enable_dns_support" {
        default = "true"
    }

    variable "enable_dns_hostnames" {
        default ="true" 
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
---


- Fixing multiple resource blocks: This is where things become a little tricky. It’s not complex, we are just going to introduce some interesting concepts. Loops & Data sources

- Terraform has a functionality that allows us to pull data which exposes information to us. For example, every region has Availability Zones (AZ). Different regions have from 2 to 4 Availability Zones. With over 20 geographic regions and over 70 AZs served by AWS, it is impossible to keep up with the latest information by hard coding the names of AZs. Hence, we will explore the use of Terraform’s Data Sources to fetch information outside of Terraform. In this case, from AWS

#### Let us fetch Availability zones from AWS, and replace the hard coded value in the subnet’s availability_zone section.

```
        # Get list of availability zones
        data "aws_availability_zones" "available" {
        state = "available"
        }
```

- To make use of this new data resource, we will need to introduce a count argument in the subnet block: such as:

```
 # Create public subnet1
    resource "aws_subnet" "public" { 
        count                   = 2
        vpc_id                  = aws_vpc.main.id
        cidr_block              = "172.16.1.0/24"
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]
    }
```


##### Let us quickly understand what happened above.

  - The count tells us that we need 2 subnets. Therefore, Terraform will invoke a loop to create 2 subnets.
  - The data resource will return a list object that contains a list of AZs. Internally, Terraform will receive the data like this


`["eu-west-1a", "eu-west-1b"]`


Each of them is an index, the first one is index 0, while the other is index 1. If the data returned had more than 2 records, then the index numbers would continue to increment.

Therefore, each time Terraform goes into a loop to create a subnet, it must be created in the retrieved AZ from the list. Each loop will need the index number to determine what AZ the subnet will be created. That is why we have data.aws_availability_zones.available.names[count.index] as the value for availability_zone. When the first loop runs, the first index will be 0, therefore the AZ will be eu-central-1a. The pattern will repeat for the second loop.

But there is still a problem. If we run Terraform with this configuration, it may succeed for the first time, but by the time it goes into the second loop, it will fail because we still have cidr_block hard coded. The same cidr_block cannot be created twice within the same VPC. So, we have a little more work to do.


- In order to solve or avoid the problem of creating same cidr_block in the same VPC, we have to make cidr_block dynamic.
- To do this, We will introduce a function cidrsubnet(). It accepts 3 parameters(Its parameters are cidrsubnet(prefix, newbits, netnum).
- Let us use it first by updating the configuration, then we will explore its internals.

```
    # Create public subnet1
    resource "aws_subnet" "public" { 
        count                   = 2
        vpc_id                  = aws_vpc.main.id
        cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]
        }
```

- A closer look at cidrsubnet – this function works like an algorithm to dynamically create a subnet CIDR per AZ. Regardless of the number of subnets created, it takes care of the cidr value per subnet.

## Removing hard coded count value.

If we cannot hard code a value we want, then we will need a way to dynamically provide the value based on some input. Since the data resource returns all the AZs within a region, it makes sense to count the number of AZs returned and pass that number to the count argument.

To do this, we can introuduce length() function, which basically determines the length of a given list, map, or string.

Since data.aws_availability_zones.available.names returns a list like ["eu-central-1a", "eu-central-1b", "eu-central-1c"] we can pass it into a lenght function and get number of the AZs.

length(["eu-central-1a", "eu-central-1b", "eu-central-1c"])

= Now we can simply update the public subnet block to look like this:

---
# Create public subnet1
    resource "aws_subnet" "public" { 
        count                   = length(data.aws_availability_zones.available.names)
        vpc_id                  = aws_vpc.main.id
        cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]
}

---

