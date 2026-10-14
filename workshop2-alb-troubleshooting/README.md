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

Command `systemtctl status nginx.service` prove that the webserver is active and running. It also show that the unit's configuration file has been parsed by **systemd** and kept in the memory. The logs also show that nginx tested its configuration and started without errors. However, it doesn't prove whether it's running on port 80, or if ALB can reach it or whether it can reply to health check request.
<img width="1127" height="256" alt="image" src="https://github.com/user-attachments/assets/213fcdca-be41-4354-bff0-a359e3e38675" />

The command `journalctl -u nginx` shows the journal entries for the nginx service. The output is the same as what `systemctl status` showed, since that command displays the latest lines from this journal. It only covers the service starting up, so I will continue to dig for more information.
<img width="1112" height="81" alt="image" src="https://github.com/user-attachments/assets/f9bf2919-15c5-47b9-af1c-b8adfb32448b" />

The command  `ss -t -l -n -p` shows details about TCP sockets in the listening state. In this output, the red circle row shows that nginx (web server) is in the listening state and ready to accept traffic on all of the instance's IPv4 addresses on port 80. The process column also shows the nginx master and worker processes, which confirms that nginx owns the port. The peer address with a wildcard means there's no peer yet, as we are waiting for a connection. This doesn’t prove that the security group allows traffic or that the ALB can reach it. I will send traffic via the load balancer to test the connectivity to the load balancer.
<img width="1699" height="99" alt="image" src="https://github.com/user-attachments/assets/c8fb1cea-3991-4272-9a5b-620f11705e69" />

In the console, we can see the target is healthy and the health check matches what I configured in the template: HTTP protocol, 30 seconds for interval, path is /, timeout is 6 seconds, port 80, success code is 200, healthy threshold is 5 and unhealthy threshold is 2.
<img width="2094" height="902" alt="image" src="https://github.com/user-attachments/assets/35cc4d90-fdd0-447b-8fbc-ef3ee0cebb80" />
<img width="2644" height="309" alt="image" src="https://github.com/user-attachments/assets/3ab48117-018e-4630-b7f4-891d6f5450c9" />

I tested connectivity to the ALB by using the `curl` command from my client machine (the Linux VM) to the load balancer’s DNS name. As shown in the following screenshot, the DNS name is my-alb-236131434.ca-central-1.elb.amazonaws.com
<img width="1474" height="198" alt="image" src="https://github.com/user-attachments/assets/79937f84-8ec9-407e-b030-0159fd99a5c9" />

I ran `curl -i <load balancer's DNS name>` from my client VM to fetch the response headers and the web server content. It returned 200, the same code the target group’s health check expects. The Server header shows nginx, and the body is the index.html content I wrote for the web server instance.
This proves the ALB security group accepts traffic on TCP port 80, the listener accepted the HTTP request on port 80, and it forwarded the request to the target group as configured. The web server security group also allowed the ALB through, and the web server replied back through the load balancer. In summary, the load balancer, its listener, the target group, and the web server are all working before any fault is introduced.
<img width="636" height="205" alt="image" src="https://github.com/user-attachments/assets/388ea01d-422d-43ea-a229-5ef34a3ee1fb" />

However, I still need to check if the web server only accept traffic from ALB. From my client VM, I ran `curl -i -v <web server's public IP>`  against the web server’s public IP address on port 80.

<img width="1531" height="639" alt="image" src="https://github.com/user-attachments/assets/625ecd32-5de9-4756-9774-57bac11b265f" />

My client machine (Linux VM) was not able to establish connectivity with the web server, which proves that the web server’s security group is dropping traffic that doesn’t come from the load balancer.
<img width="608" height="105" alt="image" src="https://github.com/user-attachments/assets/a800d72e-23f9-4af9-ae24-b029dc9d286a" />

So far, I have verified that the ALB, its security group and listener, the target group configuration, and the web server and its security group are working as configured.
###Traffic flow summary from client to application###
client (Linux VM) sent an HTTP request to the ALB’s DNS name -> the traffic entered the ALB, which only accepts HTTP traffic on port 80 -> the listener accepted the HTTP request on port 80 and forwarded it to the target group -> the target group’s target is the web server, whose security group only accepts traffic from the load balancer -> the web server accepted the traffic and replied with HTTP 200 to the load balancer -> the ALB delivered the content to the client.
