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
