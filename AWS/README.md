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
14. [RDS (Relational Database Scheme)](https://github.com/106-Sam/DevOps/tree/main/AWS#14-rds-relational-database-schema)
15. [VPC Flow Logs](https://github.com/106-Sam/DevOps/tree/main/AWS#15-vpc-flow-logs)
16. Cloud Watch
17. Auto Scaling Group (ASG)
18. Cloud Trail and Config
19. [AWS Backup (Vaults)](https://github.com/106-sam/DevOps/tree/main/AWS#19-aws-backup-vaults)
20. [VPN & connection between AZURE & AWS](https://github.com/106-sam/DevOps/tree/main/AWS#20-vpn--connection-between-azure--aws-machines)

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


In 3-Tier Architecture, we have used IaaS model (Infrastructure as a Server) for MYSQL in EC2 Instance. Now lets see how to use PaaS model (Platform as a Service)


The RDS, supports 8 Databases: 

1. Aurora (MySQL Compatible)
2. Aurora (PostgreSQL Compatible)
3. MySQL
4. PostgreSQL
5. MariaDB
6. Oracle database - AWS doesn't provide Oracle license so, we need to get our own licence rest all databases license is provided by AWS (Bring your OWN license)
7. Microsoft SQL Server
8. IBM Db2

#### Installing the DB

![RDS-Failover](https://github.com/106-Sam/DevOps/blob/main/AWS/files/RDS-failover.png "RDS AWS")


1. Create the Database
 
   Aurora & RDS > Databases > create database > db creation (Full configuration) > Templates (Dev/Test) > Availability and durability (Multi-AZ DB instance deployment 2 instance)

   Remaining proivde the default settings

   master username = sqladmin
   Managed in AWS Secrets Manager (for Github action)
   Self managed (we need to create it there)

   Burstable class (includes t classes)
   Instance type = db.t3.micro
   Storage type = ssd gp3 ( but in real time need to take io1 & io2)
   Allocated storage = 20GiB
   Select VPCs

   No public create private

2. Connection Tools

mysqldb - mysql workbench
mssql - ssms
mongodb - mongo express

3. Check failover using nslookup command


## 15. VPC Flow Logs

   - By using VPC Flow logs, we montior what kind of traffic is going out and what kind of traffic is coming in. 
   - ENI card (NIC) provide the traffic in and traffic out.
   - We connect this to Cloud Watch.
   - All the Cloud Watch data must be stored in the LOG GROUP. We create an IAM role for this.


1. Create a IAM role
   
      - Create role > Custom Trust Policy
 ```
 {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VPCFlowLogsPolicy",
            "Effect": "Allow",
            "Principal": {
                "Service": "vpc-flow-logs.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
} 

 ```   
      - Create the role with name vpcflowlogs
  
   2. Create the IAM Policy.
   
      - Create the Policy to attach it to the above role for having permission to create the logs and etc.

```

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VPCFlowLogsPermission",
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents",
                "logs:DescribeLogGroups",
                "logs:DescribeLogStreams"
            ],
            "Resource": "*"
        }
    ]
}

```
      -  VPC flow logs policy attach the policy to the IAM role.
  
3. Creation of Log Groups & setup of Flow Logs

      -  Create a machine after this create the logs groups 
      -  Cloud Watch > Log Management >  name123  rest all default.
      -  go to the respective VPC, edit it 'Flow Logs' > Create a flow Log
      -  Filter = All 
      -  Maximum aggregation interval = 1 min 
      -  Destination = Send to CloudWatch Logs 
      -  Select the destination log group.
      -  Select Service role >  and use the created role


4. Verify  Log Streams
   - Log management > name123 > Logstream
   - Double click and check the log events




## 19. AWS Backup (Vaults)

We create the backs under the AWS Vault. And we can restore the backups from the Vault. 

- We will have 30 days retention period. Basically today back will be there for 30 days and so on vice versa.

1. AWS Backup > Job dashboard > Create Backup Plan > Start with a template 
  Give the backup plan a name.

In templates, we can see 4 options:

  - Daily-35day-Retention
  - Daily-Monthly-1yr-Retention
  - Daily-Weekly-Monthly-5yr-Rentention
  - Daily-Weekly-Monthly-7yr-Rentention

2. create Backup rule keep the backup frequency as per our need like hourly, every 12 hours, daily, custom cron expression...etc.
3. Choose the backup window date, time and timezone
4. We can reduce and update the rentention days etc.
5. under backup indexes Select the where to be store like s3 or EBS 
6. (Optional) If you like to copy the back in the another region, we can choose 

  
How to Restore the backup ?

1. AWS Backup > Vaults > Vaults created by this account.
2. double-click the backup name (You can see Recovery points all the things that got backed up)
3. select the points you want to back up and click Action > Restore 
     
      - It will prompt you for network settings VPC, SUbnet, SG, Instance IAM role, Storage
      - If its an EC2 instance, you won't be attached to Public IP. Therefore, you need to create an EIP (Elastic IP address) 
      - Assocaite EIP to Instance & selec the private address in the dropdown.
      - Now you have the Public IP 


## 20. VPN & Connection between Azure & AWS Machines