## **Objective**
Build a VPC with public and private subnets across two Availability Zones, test which instances can communicate, then deliberately break and troubleshoot blocked traffic. The goal is to gain a deeper understanding of, and be able to explain, what makes a subnet public or private, how a private instance reaches the internet, and how routing, security groups, and NACLs come together in the big picture.
## **Implementation**
The following architecture consist of VPC, two availability zones, IGW, NACL, two public subnets, two private subnets, route table on each subnet, NAT gateway on public subnet AZ A, private server and security group on AZ A, public server and security group on AZ B:
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
