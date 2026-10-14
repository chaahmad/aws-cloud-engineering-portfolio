## **Objective**
The purpose of this project is to gain a deeper understanding of the load balancer, ALB listener, target group, and health check by troubleshooting three faults: (1) stopping the web server application service, (2) removing the web server security group’s inbound rule that allows traffic from the ALB, and (3) configuring an incorrect health check path.
## **Implementation**
I've built the atchitecture based on the following configuration:
- VPC
- Two public and two private subnets across two availability zones
- Internet gateway (IGW)
  - IGW attachment to the VPC
- Public route table
  - Public route to the IGW
  - Public subnet associations to the public route table
- Private route table
  - Private subnet associations to the private route table
<img width="1223" height="999" alt="image" src="https://github.com/user-attachments/assets/1e0a4bce-9cc2-442e-81a1-43321cd4f36c" />
