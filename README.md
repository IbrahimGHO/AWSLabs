# AWSLabs
# CloudFormation Lab: Create a VPC with an EC2 Instance Hosting a Simple Website

## Objective
Create a Virtual Private Cloud (VPC) with a public subnet, an Internet Gateway, a Security Group, and an EC2 instance with an Apache web server.

## Parameters
```yaml
Parameters:
  InstanceProfileName:
    Description: Name of the IAM instance profile to attach to the EC2 instance
    Type: String
    Default: MyInstanceProfile
