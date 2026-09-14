# AWS - Amazon Web Service




### VPC Endpoints

- We use VPC Endpoint only when we want traffic to move internally (but not over the internet).
- For Internal Routing

#### How to securely manage an isolated EC2 instance in a private subnet using ***AWS Systems Manager (SSM) Session Manager*** via ***VPC Interface Endpoints, completely elimination the need for an IGW, NAT Gateway, or open Inbound ports.***

1. IAM Role Configuration

   - Create an IAM Role for EC2 Instances. 
   - Navigate to IAM > Role > Create Role.
   - Choose the AWS Service as Trusted Entity, & EC2 as use case.
   - Add Permission "AmazonSSMManagedInstanceCore". 
   - Give it a Role name, & Create Role. Ex:- "EC2-SSM-Instances"


2. Create an EC2 Instances 

   - Deploy the Public & Private Servers [I already have 3 Public & 3 Private Subnets(RT - subnet association in Private Subnets)].
   - While creating the EC2 instances - Advance details > IAM Instance Profile  select the Role (which we created in Step1).
   - We must not attach a key pair not needed for SSM. 
   - Security Group must allow HTTP & SSH 
   - RT table Pub-RT needs IGW (Internet Gateway) & PVT-RT just the SG default.
  

3.  Create VPC Endpoints
  
    - Create 3 VPC Endpoints one *ec2messages, ssmmessages & ssm.*
    - For *ec2messages, ssm* enable all the Private subnets.
    - For *ssmmessages*, enable 1 Private subnets.
    - Select IPv4, then SG- private ones.
    - Note: we can modify the subnets and all configurations.
  

4.  