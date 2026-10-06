## **Objective**
Build a VPC with public and private subnets across two Availability Zones, test which instances can communicate, then deliberately break and troubleshoot blocked traffic. The goal is to gain a deeper understanding of, and be able to explain, what makes a subnet public or private, how a private instance reaches the internet, and how routing, security groups, and NACLs come together in the big picture.
## **Implementation**
The following architecture consist of VPC, two availability zones, IGW, NACL, two public subnets, two private subnets, route table on each subnet, NAT gateway on public subnet AZ A, private server and security group on AZ A, public server and security group on AZ B:
<img width="1063" height="868" alt="image" src="https://github.com/user-attachments/assets/2cd4432d-762f-4bd8-bf14-63d62249855d" />
