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
## Verification

Stack
<img width="1521" height="206" alt="image" src="https://github.com/user-attachments/assets/b803a76d-e962-4db2-9e00-cc87768e2e2b" />
VPC - General details
<img width="1533" height="924" alt="image" src="https://github.com/user-attachments/assets/eaa347ca-7011-4140-a0b1-985058bde29a" />
VPC Resource Map
<img width="1641" height="964" alt="image" src="https://github.com/user-attachments/assets/c038ba08-8238-46b7-a547-9dad58baeb8b" />
Public and private subnets
<img width="1498" height="233" alt="image" src="https://github.com/user-attachments/assets/5f0eda94-9a25-45c8-8340-4fcb95094d5f" />
Public and private route table
<img width="1300" height="170" alt="image" src="https://github.com/user-attachments/assets/84245816-f178-4746-985b-dab346af8da4" />
IGW
<img width="2688" height="1021" alt="image" src="https://github.com/user-attachments/assets/06902d2b-e730-45b0-9c30-8a3af573e8aa" />
Webserver-Security Group
<img width="2530" height="602" alt="image" src="https://github.com/user-attachments/assets/c8ff62ca-d87e-4342-9317-f83f823ef51d" />
Load Balancers-General Details
<img width="2666" height="957" alt="image" src="https://github.com/user-attachments/assets/7824d136-bd0d-4a09-aa16-4e398affbc50" />
Load Balancers-Listeners and rules
<img width="2339" height="915" alt="image" src="https://github.com/user-attachments/assets/efb0cb01-3a52-4f9f-9c5c-fe319b4429a6" />
Load Balancers-Security
<img width="2340" height="872" alt="image" src="https://github.com/user-attachments/assets/84bea716-613c-4e03-a0a8-2937f647600c" />
Target Groups-General Details
<img width="2751" height="1119" alt="image" src="https://github.com/user-attachments/assets/3e6f39ce-24fd-42fc-b486-b3c812b0ff1f" />
Target Groups-Target
<img width="2094" height="902" alt="image" src="https://github.com/user-attachments/assets/493e161f-077b-4b35-b79f-e36b902609b8" />
Target Groups-Health Check
<img width="2644" height="878" alt="image" src="https://github.com/user-attachments/assets/e146b606-8670-4162-9592-7c5e7635388c" />
The Linux command `ps -fC nginx` only shows that the nginx processes are running and nothing more, so I need to run further commands to verify that it’s serving requests.
<img width="730" height="82" alt="image" src="https://github.com/user-attachments/assets/006c1149-c44a-4f1a-80eb-9c353d58ef39" />
