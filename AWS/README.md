# AWS - Amazon Web Service

**Index of Content**

1. AWS Account Creation
2. IP Addressing
3. VPC Peering (Virtual Private Cloud)
4. Transit Gateway & NAT Gateway
5. Security Group (SG)
6. Network Load Balancer (NLB)
7. Application Load Balancer (ALB)
8. Global Accelerator & WAF Routing Policies
9. S3 Bucket
10. IAM (Identity & Access Management) 
11. IAM Extras
12. [VPC Endpoint](https://github.com/106-Sam/DevOps/tree/main/AWS#12-vpc-endpoints)
13. [EBS Volume (Elastic Block Storage)](https://github.com/106-Sam/DevOps/tree/main/AWS#13-ebs-volume-elastic-block-storage)
14. RDS (Relational Database Scheme)
15. Cloud Watch
16. Auto Scaling Group (ASG)

--- 

## 12. VPC Endpoints

- AWS VPC Endpoints allow instances in a VPC to privately access supported AWS services without requiring an IGW, NAT gateway, VPN, or AWS Direct Connect connections.
- We use VPC Endpoint only when we want traffic to move internally (but not over the internet).
- For Internal Routing only.

There are 2 types of Endpoint:

1. Interface Endpoint (AWS PrivaterLink)
   
    - Creates an Elastic Network Interface (ENI) with a private IP inside your specified subnet.
    - Supports most AWS services (SSM, EC2, CloudWatch, Secrets Manager,etc.) and Saas/marketplace services.
    - Not Free (Hourly rate + data processing fee per GB).
    - Attached via Subnets and secured using Security Groups.
  
2. Gateway Endpoint
   
   - Adds a target route entry directly inside your VPC Route Table.
   - Supports only Amazon S3 and Amazon DynamoDB.
   - Free (no hourly fee, no data transfer fee).
   - Associated with Route Tables (no SG attached)

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
  
  
4.   Connecting machines via SSM.
   
     - After searching "Session Manager", Start Session.
     - Now refresh AWS Session Manager and you will find private server & Public sever, now you can even start session.
     - And you will be able to connect the ec2 instances without keys, privately using VPC Endpoints.
     - For Internet we will be needing the NAT Gateway as usual.
     - After creating 3 endpoints to all 3 subnets, endpoints are able to the 3 Private subnets.

Note: In private servers, you won't be able to connect to Internet whereas Public server, will be connected to the Internet.

#### Interface Endpoint

  1. Create S3 buckets.
   
     - Create a bucket --> Name1 --> use all defaults and create bucket upload some files in two buckets.
     - After this try connecting to Pub-Server & Private-server EC2 Instances.
     - **Pub-server** is able download the s3 bucket files over the internet while **Pvt-server** isn't able to connect over the internet that is the reason it cannot download the files, for this we need endpoints.
     - 

What interface endpoints do? 
Ans: It will create Elastic network Interface(ENI) endpoints for each subnet.



#### Gateway Endpoint

   1. After creation of s3 bucket.
      
      - If you see in the VPC, and check the Private Route Table, you will see only one route that will be local.
      - Now, let us create the Endpoint type "gateway" and  S3 as service name(but choose gateway).
      - Select the VPC and it's Private Route table and create Endpoint.
      - Once Gateway endpoint is created, it will update Routing Table.
      - Now, we will be able to download from private server. (First do aws configure)



What Gateway Endpoint do?
Ans: It will add a route in routing table.

---

## 13. EBS Volume (Elastic Block Storage)

Difference between Instance Store Volume & EBS Volume

| Instance Store Volume | EBS Volume |
| --- | --- |
| The data is not permenant here | The data is permenant here |
| It is only for specific instance type (c5d.large) | It is availabe in all instance types (t3.micro) |
| It is of fixed size | it is not fixed siez |
| You cannot detach and attach to different servers | it can attach & detach EBS volumes |
| It is used for virtual memory [jpage files] | We can snapshot it and create new volumes from snap. |

_Note: EBS & EC2 must be in the same availability zone._


**Instance Store Volume: **

1. It's a temporary storage means data is deleted if you stop and start the server.
2. Its fixed size depending on the EC2 Instance type.
3. It is fixed type which is SSD.
4. We cannot detach and attach to other server.
5. Not available for all EC2 servers.
6. Its generally used for virtual memry

**EBS Volume: **

1. Its persistent or permanent
2. Its available for all nodes
3. We can increase it cannot decrease EBS Volume.
4. Available in gp2, gp3, io1, io2, sc1, etc.
5. We can snapshot it and create new volumes from snap.
6. We can attach and detach

While creating a volume, you will have different options

you can divide EBS Volume into two types – SSD and HDD
Disk performance based on IopS [ input output per second]
SSD 🡺 Solid state drive (Integrated circuits)
       gp2 ( normal performance) min 100 Iops max 16000 3Iops 
       gp3 ( intermediate performance)
        io ( fast performance)
       io2 ( very fast performance)
HDD 🡺 Hard disk drive ( mechanical )

Which HDD I have to take 
For example: I want to deploy high performance MSSQL DB on Ec2 Instance
When you create a Instance you will get
OS (C) : gp2/gp3
Media : Sc1/ST1
Page file:
SQL_data_MDF – IO1/IO2
SQL_data_NDF - IO1/IO2
SQL_data_TLOG - IO1/IO2
SQL_TempDB(IS) – Instance store
SQL_Backup - Sc1/ST1
You can check aws calculator s3 -> check estimate



#### Attaching EBS to Windows Machine 

1. Create a EC2 Windows Instance. (EBS-optimized Instance in advance details must be Enable)

   - Windows machine and generate the password with .pem file used while creating.
   - xfreerdp /u:<username> /p:<password> /v:<ip> /f

2. Create a EBS Volume.
      - with GPT2
      - attach it to Windows Instance.
      - select instance & device name: (Recommended device names for Windows: /dev/sda1 for root volume. xvd[f-p] for data volumes.
      - Select xvdf
)
3. Mounting EBS Volume to Windows

   - Open Disk Management & Initialize the disk.
   - Right click > New Simple Volume > Next > Specifiy Volume size > Assign Drive Letter or Path > NTFS > Finish. 
   - Now, you will be able to see the EBS volume Drive in This PC.

4. Create a new Machine detach it from here and Attach it. You will see the same data in that drive.  
      


#### Attaching EBS to Linux Machine 

1. Create a AWS Linux Instance.
   - To know the volumes in linux (lsblk)
   - Now let us add additional disk to this machine

2. Create a EBS Volume.
      - with GPT2
      - attach it to Windows Instance.
      - select instance & device name: (Recommended device names for Linux: /dev/xvda for root volume. /dev/sd[f-p] for data volumes.)
      - Select xvdf

3. Mounting EBS Volume to Linux.
   - lsblk (now you got added volume)
   - fdisk /dev/xvdf or /dev/nvme1n1 
   - choose 'm' to check option 'n' to add partition
   - rest keep it default and in the end 'w'
   - Now we have to create filesystem on this drive
 - mkfs.ext4 /dev/xvdf1 or /dev/nvme1n1p2
   - To access this we have to mount 

4. Create a Folder and mount it.

   - mkdir EBS/
   - mount /dev/nvme1n1p2 EBS/
   - lsblk 
   - Now you will see the attached volume



## 14. RDS (Relational Database Schema)