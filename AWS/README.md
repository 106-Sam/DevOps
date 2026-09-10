# AWS - Amazon Web Service




### VPC Endpoints

- We use VPC Endpoint only when we want traffic to move internally (but not over the internet).
- For Internal Routing

#### How to securely manage an isolated EC2 instance in a private subnet using ***AWS Systems Manager (SSM) Session Manager*** via ***VPC Interface Endpoints (PrivateLink), completely elimination the need for an IGW, NAT Gateway, or open Inbound ports.***

1. IAM Role Configuration

- Create an IAM Role for EC2 Instances. 
- Navigate to IAM > Role > Create Role.
- Choose the AWS Service as Trusted Entity, & EC2 as use case.
- Add Permission "AmazonSSMManagedInstanceCore". 
- Give it a Role name, & Create Role. Ex:- "EC2-SSM-Instances"


2. 