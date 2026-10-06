## **Objective**
Build a VPC with public and private subnets across two Availability Zones, test which instances can communicate, then deliberately break and troubleshoot blocked traffic. The goal is to gain a deeper understanding of, and be able to explain, what makes a subnet public or private, how a private instance reaches the internet, and how routing, security groups, and NACLs come together in the big picture.
## **Implementation**
This lab builds the following network in VPC A (us-east-1). The diagram shows the layout, and the tables below list the configuration.
<img width="1063" height="868" alt="image" src="https://github.com/user-attachments/assets/2cd4432d-762f-4bd8-bf14-63d62249855d" />
| Component | Where | Details |
|---|---|---|
| VPC A | us-east-1 | 10.0.0.0/16 |
| Availability Zones | us-east-1a (AZ1), us-east-1b (AZ2) | Two AZs |
| Public subnets | AZ1 and AZ2 | 10.0.0.0/24 and 10.0.2.0/24 |
| Private subnets | AZ1 and AZ2 | 10.0.1.0/24 and 10.0.3.0/24 |
| Internet gateway (IGW) | Attached to VPC A | Gives public instances a two-way path to the internet |
| NAT gateway | Public subnet, AZ1 | Has an Elastic IP; lets private instances start outbound connections |
| Public route table | Associated with both public subnets | 10.0.0.0/16 to local; 0.0.0.0/0 to the IGW |
| Private route table | Associated with both private subnets | 10.0.0.0/16 to local; 0.0.0.0/0 to the NAT gateway |
| Network ACL (NACL) | Associated with all four subnets | Rule 100 allows all traffic, inbound and outbound |
| Private server | Private subnet, AZ1 | Own security group; reaches the internet through the NAT gateway |
| Public server | Public subnet, AZ2 | 10.0.2.100; own security group; public IPv4 address |

**Why I built it this way**

- **Two Availability Zones:** for availability, so the network spans more than one AZ.
- **Public and private subnets:** network segmentation is a security best practice. Public subnets are for resources that must be reachable from the internet, whereas private subnets are for resources that shouldn't be exposed to it. The route table is what makes the difference, not the address range.
- **NAT gateway in a public subnet:** it allows outbound traffic from the private subnets to the internet. It sits in a public subnet because it needs its own route to the internet gateway, and only a public subnet's route table has one.
- **Internet gateway (IGW):** the connecting point between the VPC and the internet. It gives resources that have a public IP a two-way path.
- **NACL and security groups:** two layers of filtering. The NACL evaluates traffic at the subnet level, whereas a security group evaluates it at the resource level.
- **Route tables:** they direct traffic according to the routes in the table. Without a route, the traffic has nowhere to go.

**Configuration Evidence**

VPC
<img width="1655" height="1028" alt="image" src="https://github.com/user-attachments/assets/56354721-8fb8-4434-85fc-8872d0b65e1c" />
Public Subnet AZ1
<img width="1442" height="596" alt="image" src="https://github.com/user-attachments/assets/21620293-7380-4388-8e99-f4a868f06f18" />
Private Subnet AZ1
<img width="1466" height="584" alt="image" src="https://github.com/user-attachments/assets/2896f778-8c53-47c1-ac6e-11bd4a34f1de" />
Public Subnet AZ2
<img width="1455" height="578" alt="image" src="https://github.com/user-attachments/assets/7a0491ee-2e4e-4536-9efe-a04a05311c63" />
Private Subnet AZ2
<img width="1455" height="585" alt="image" src="https://github.com/user-attachments/assets/535e35e3-dfeb-4ec0-b7d1-4ead6c301062" />
Public Route Table-Routes
<img width="1344" height="317" alt="image" src="https://github.com/user-attachments/assets/62507246-8840-4cfc-9f5e-87839072ed3a" />
Private Route Table-Routes
<img width="1463" height="347" alt="image" src="https://github.com/user-attachments/assets/37d8417d-8402-4eae-9e78-45a8cb8f998b" />
Private Route Table-Subnet associations
<img width="1456" height="292" alt="image" src="https://github.com/user-attachments/assets/1d36d5de-b82c-4649-ac5c-5dce2f85c0ad" />
Private Route Table-Routes
<img width="1454" height="320" alt="image" src="https://github.com/user-attachments/assets/5e16fee6-cce4-4450-874e-8a8fa04f20e5" />
Private Route Table-Subnet associations
<img width="1455" height="290" alt="image" src="https://github.com/user-attachments/assets/d325f123-b50f-490e-9fab-2633fcc7f430" />
Internet Gateway
<img width="1458" height="232" alt="image" src="https://github.com/user-attachments/assets/1c12c103-82b8-42c2-9cf4-5b371899ff8c" />
NAT Gateway
<img width="1455" height="350" alt="image" src="https://github.com/user-attachments/assets/7a5ad36c-8e60-4766-b6e8-506bd7dbc361" />
NACL-Inbound Rules
<img width="1447" height="327" alt="image" src="https://github.com/user-attachments/assets/f50ba2c0-37c0-4ae6-b8c5-01b2b68f62d8" />
NACL-Outbound Rules
<img width="1455" height="321" alt="image" src="https://github.com/user-attachments/assets/ff5f9294-57d3-4d02-b7fc-623e0793ba54" />
NACL-Subnet Associations
<img width="1454" height="365" alt="image" src="https://github.com/user-attachments/assets/9bf044ca-4b4d-4387-afaa-e642cb7028e6" />
Security Group for public and private server-Inbound Rules
<img width="1453" height="290" alt="image" src="https://github.com/user-attachments/assets/ad813f56-755a-4714-9227-34b2fd07681a" />
Security Group for public and private server-Outbound Rules
<img width="1449" height="286" alt="image" src="https://github.com/user-attachments/assets/193d39c8-8cb8-4a64-9d96-70308434246d" />
Instance in Public Server-General Details
<img width="1332" height="583" alt="image" src="https://github.com/user-attachments/assets/cb23afb9-4df6-4df6-8e53-ce38a69efa2e" />
Instance in Public Server-Security Details
<img width="1354" height="300" alt="image" src="https://github.com/user-attachments/assets/c69082c3-3693-45f1-8e49-0a10f83ef7e3" />
Instance in Private Server-General Details
<img width="1348" height="589" alt="image" src="https://github.com/user-attachments/assets/ce96c761-cfc5-4eb1-b9bf-f3e5fa77d249" />
Instance in Private Server-Security Details
<img width="1364" height="281" alt="image" src="https://github.com/user-attachments/assets/a5aef2fe-0c95-479f-8aed-ec22802a3433" />

## Verification
I ran three tests to confirm the network works as designed.

| Test | From | To | Result | What it proves |
|---|---|---|---|---|
| 1 | My computer | Public server, public IP 52.55.148.239 | Worked | The public server is reachable from the internet and replies. The IGW, route table, NACL, and security group all permit it. |
| 2 | Private server (AZ1) | Public server, private IP 10.0.2.100 | Worked | Instances in different subnets and AZs communicate over the VPC's local route. |
| 3 | Private server (AZ1) | example.com | Worked | A private instance can start a connection to the internet through the NAT gateway and IGW, and the reply returns. DNS resolution also works. |

#### Test 1: my computer to the public server's public IP
<img width="547" height="249" alt="image" src="https://github.com/user-attachments/assets/697deee9-5e3b-44fd-bd64-7dc3dd56988e" />

#### Tests 2 and 3: private server to 10.0.2.100 and to example.com
<img width="656" height="342" alt="image" src="https://github.com/user-attachments/assets/0f220e8a-deff-4e53-93ca-10acf09923a6" />

