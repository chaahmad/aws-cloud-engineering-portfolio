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
