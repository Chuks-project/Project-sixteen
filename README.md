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




