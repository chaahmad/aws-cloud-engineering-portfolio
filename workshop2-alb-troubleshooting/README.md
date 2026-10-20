## **Objective**

The purpose of this project is to gain a deeper understanding of the load balancer, ALB listener, target group, and health check by troubleshooting three faults: (1) stopping the web server application service, (2) removing the web server security group’s inbound rule that allows traffic from the ALB, and (3) configuring an incorrect health check path.

## **Implementation**

I built the architecture based on the following configuration:

- VPC
- Two public and two private subnets across two availability zones
- Internet gateway (IGW)
  - IGW attachment to the VPC
- Public route table
  - Public route to the IGW
  - Public subnet associations to the public route table
- Private route table
  - Private subnet associations to the private route table
- Application load balancer and its security group
- Web server and its security group
- IAM role and instance profile for Systems Manager (SSM) access to the web server

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

The command `systemctl status nginx.service` proves that the web server is active and running. It also shows that the unit's configuration file has been parsed by **systemd** and kept in memory. The logs also show that nginx tested its configuration and started without errors. However, it doesn't prove whether it's running on port 80, whether the ALB can reach it, or whether it can reply to a health check request.

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

### Traffic flow summary from client to application

client (Linux VM) sent an HTTP request to the ALB’s DNS name -> the traffic entered the ALB, whose security group allows inbouncd TCP traffic on port 80 -> the listener accepted the HTTP request on port 80 and forwarded it to the target group -> the target group’s target is the web server, whose security group only accepts traffic from the load balancer -> the web server accepted the traffic and replied with HTTP 200 to the load balancer -> the ALB delivered the content to the client.

## Fault 1- stop the application service

- I stopped nginx and disabled it.
- From the client (Linux VM), I tested the web server with `curl -i my-alb-236131434.ca-central-1.elb.amazonaws.com` (Alb's dns name).

**Symptom**: I received 502 Bad Gateway, which means it reached the load balance, but ALB wasn’t able to get a valid response from the web server.

<img width="665" height="237" alt="image" src="https://github.com/user-attachments/assets/3f046fd0-8c7d-470b-a4b2-f6407124c11f" />

**Hypothesis**: since the request was able to reach the load balancer, the path from the client to the ALB works. Most likely the problem is in the web server, so I will check the following as part of my troubleshooting methodology :

1. Check the target health status.
2. Check the log for any unusual activity.
3. Check service status, including it’s port.

**Test 1 and finding**: the target is unhealthy. After waiting for about 60 seconds, the target changed to unhealthy. The load balancer sends a health check request every 30 seconds, and the target becomes unhealthy after failing to return a successful response two times in a row.

<img width="2085" height="297" alt="image" src="https://github.com/user-attachments/assets/aeb7451e-de8b-4f21-ba55-e3206ae0d656" />

**Test 2 and finding**: Command `ps -fC nginx` output doesn't show that the nginx is running.

<img width="465" height="38" alt="image" src="https://github.com/user-attachments/assets/edceb47d-abbe-4e98-8f18-522b2ece1abf" />

The command `systemctl status nginx.service` displays the status and logs for the web server, and it shows that the web server is inactive and disabled.

<img width="1127" height="239" alt="image" src="https://github.com/user-attachments/assets/97e8c6ef-8944-4653-9586-9d258be985e2" />

**Test 3 and finding**: The command `ss -t -n -p -a` shows all TCP sockets across all states and their processes. The output doesn’t show anything about port 80 or nginx as a process in the Process column, which means the web server is not in the listening state it was in before the fault was introduced.

<img width="1194" height="101" alt="image" src="https://github.com/user-attachments/assets/88c4bde4-44bf-4cce-b365-fae26737b729" />

This shows that nginx is down. To double-check, I ran a local test, instead from the client machine (Linux VM), and the connection to localhost on port 80 failed.

<img width="666" height="41" alt="image" src="https://github.com/user-attachments/assets/d4119274-3012-4316-8345-fa6ca97f8834" />

**Correction**: I ran `systemctl enable nginx.service` and `systemctl start nginx.service` to enable and start the web server.

**Verification**:

I ran systemctl status nginx.service to check the status of the web server, it shows as active and enabled.

<img width="1134" height="324" alt="image" src="https://github.com/user-attachments/assets/721c18d6-ddce-4ba0-b4dd-1242cf30fbe8" />

The command `ps -fC nginx` show the web server is running.

<img width="730" height="83" alt="image" src="https://github.com/user-attachments/assets/65ad5265-d550-4bdd-8cf7-7f09bba2704b" />

The command `ss -t -n -p -a` shows that the web server is in the listening state and ready to accept traffic on local address 0.0.0.0 port 80.

<img width="1243" height="273" alt="image" src="https://github.com/user-attachments/assets/9c10519a-5a0a-4281-9d36-a0be7c07951f" />

The target health status returned to healthy, which confirms the load balancer’s health checks are passing again.

<img width="1508" height="349" alt="image" src="https://github.com/user-attachments/assets/be296d19-e9eb-413a-bff0-442d20833aeb" />

The Linux commands show that the web server is up and running. For final verification, I tested connectivity locally and from the client machine (Linux VM).

Web server (local test): it was able to fetch the response headers and the HTML content from the web server.

<img width="637" height="191" alt="image" src="https://github.com/user-attachments/assets/cb3de480-1b45-4a27-8516-17a3d289d2bb" />

Client machine (Linux VM): it was also able to fetch the response headers and the HTML content from the web server through the load balancer.

<img width="642" height="206" alt="image" src="https://github.com/user-attachments/assets/ddd4a6c3-e191-46f7-b3db-544a1649cd23" />

## Fault 2 - remove the web server security group’s inbound rule for the ALB

I removed the inbound rule from the web server security group that only allows TCP traffic from the load balancer's security group to its port 80.

From the client (Linux VM), I tested the web server with `curl -i my-alb-236131434.ca-central-1.elb.amazonaws.com` (Alb's dns name).

**Symptom**: I received ‘504 Gateway Time-out’, which means it reached the load balancer, but the load balancer got no response at all from the web server, not even a negative response.

<img width="645" height="237" alt="image" src="https://github.com/user-attachments/assets/77966481-18f0-49f6-8f66-1e96c37afd94" />

Then I tested localhost (on the web server machine) and it returned a response with ‘200’, which means that the web server is running. This highlights that the web server is running, but it isn’t receiving requests from the load balancer.

<img width="397" height="194" alt="image" src="https://github.com/user-attachments/assets/5d71f8fe-14a4-441c-b3da-41e6796cff0c" />

**Hypothesis**: I will check the following:

1. The target's health status.
2. The socket status of the web server, to verify whether it’s in the listening state.
3. Whether the web server is exhausted or not.
4. The web server’s security group, to check if it’s missing any allowed rule.

**Test 1 and finding**: the target is unhealthy. This means that just because the target is unhealthy, it doesn’t automatically mean that the instance, or in this case the web server, has stopped. I noticed that the health status details said “Request timed out”, which means it failed to respond to the load balancer. I will continue to investigate why it’s not responding to the load balancer when it’s running fine.

<img width="1666" height="343" alt="image" src="https://github.com/user-attachments/assets/134396c6-f8f1-4d6e-9387-34a05d7055d8" />

**Test 2 and finding**: I ran command `ss -t -n -p -a` to view details for all TCP sockets. The red circle confirmed that nginx is in the listening state and willing to accept traffic on its local address on port 80.

<img width="1743" height="211" alt="image" src="https://github.com/user-attachments/assets/5e03e0fa-fce2-4521-aee3-784b3e8229ca" />

**Test 3 and finding**: I ran `df -h -a` to see all filesystem on disk space usage. The purpose of using this command to see if there's a heavy disk space utilization, which can explain web server is unable to respond due to disk exhaustion. However, we only used 38% of the disk, so this is not an issue.

<img width="730" height="482" alt="image" src="https://github.com/user-attachments/assets/5643a150-34f3-4652-9f92-0eda30809ed7" />

I also ran `free -h` for the same purpose as disk space utilization, except for memory utilization. This output show 573Mi of 913Mi is available, so memory is not exhausted.

<img width="658" height="65" alt="image" src="https://github.com/user-attachments/assets/a310c74a-5e97-41ac-b5d4-8cca566a826e" />

**Test 4 and finding**: I checked the web server security group, and the details show that it has one outbound rule but no inbound rules. This is the root cause of the problem. A security group only contains allow rules, so the server needs a rule that allows the desired traffic in. Without one, inbound traffic is denied.

<img width="1428" height="337" alt="image" src="https://github.com/user-attachments/assets/850134e1-5f78-4d2c-8f2a-5f30a7558b88" />

**Correction**: I added the inbound rule back to the web server security group. It only accepts HTTP traffic on port 80 from the load balancer’s security group.

<img width="1662" height="416" alt="image" src="https://github.com/user-attachments/assets/9b036f6d-80ce-46f6-8b78-170c7901f7c1" />

The following two screenshots show that inbound rule is configured;

<img width="1429" height="327" alt="image" src="https://github.com/user-attachments/assets/54237262-f24b-4e32-ad3d-0674488d0a59" />

<img width="1510" height="301" alt="image" src="https://github.com/user-attachments/assets/18c953d8-c294-44a5-bd65-b1272c78e678" />

**Verification**:

The target’s health status is showing as healthy now.

<img width="1530" height="339" alt="image" src="https://github.com/user-attachments/assets/eb14b547-5c4d-4099-bee3-ea3e0bce1103" />

From the client machine (Linux VM), I tested the web server with `curl -i my-alb-236131434.ca-central-1.elb.amazonaws.com` (the ALB’s DNS name), and it responded with a ‘200’ code, which means the web server is now responding to the load balancer’s requests successfully.

<img width="646" height="209" alt="image" src="https://github.com/user-attachments/assets/6d4e763f-27d6-42ed-a369-9f36cc28ae52" />

## Fault 3 - configure an incorrect health check path

Currently, the health check path is /, as shown in the following screenshot:

<img width="1521" height="307" alt="image" src="https://github.com/user-attachments/assets/b1a6c640-a98e-4a53-b66c-3d2e61ac7034" />

I changed it to /health.

<img width="1525" height="308" alt="image" src="https://github.com/user-attachments/assets/346dacd4-0084-4178-b98b-d2db907e82df" />

I tested the new health check path with the command `curl -i http://localhost/health`

**Symptom**: it returned a ‘404’ error, which means the page or resource I requested was not found. nginx generated the content from the /usr/share/nginx/html/404.html file. This doesn’t match the configured success criteria, which is 200.

<img width="797" height="1266" alt="image" src="https://github.com/user-attachments/assets/c6cdfc0f-6d49-499f-85da-b4f409c3d6b2" />

<img width="659" height="691" alt="image" src="https://github.com/user-attachments/assets/9c652b5f-7a44-41ec-aa52-b40425330562" />

**Hypothesis**: I will check if the following:

1. The target’s health status.
2. Error 404 could also mean that the resource file path is misconfigured or missing, so I will check that the index file exists and test it locally from the web server.
3. I will test the web server from the client machine (Linux VM) to validate the web server’s functionality.
4. Check the log file to see if there's an explanation.

**Test 1 and finding**: the target is unhealthy, and the details show the health check failure reason: 404. This doesn’t prove whether the web server can return a successful response to other requests, nor does it prove whether the index.html file exists.

<img width="2098" height="340" alt="image" src="https://github.com/user-attachments/assets/38dbe6b1-7842-4c9c-9c14-1f068fddc4c1" />

**Test 2 and finding**: the command `cd /usr/share/nginx/html/ ; ls` changed into the html folder and displayed its contents. We can see that the index.html file does exist, so the web server should be able to return the response code 200 to localhost.

<img width="552" height="37" alt="image" src="https://github.com/user-attachments/assets/fd69bea1-e50e-4b81-bb32-e655fbb2653e" />

I tested localhost, and it responded with a 200 code and the HTML content.

<img width="401" height="191" alt="image" src="https://github.com/user-attachments/assets/2d4801e6-5a7f-4ae7-8866-0030a4a1bb10" />

**Test 3 and finding**: the client machine (Linux VM) also received a response with a 200 code. This shows that just because the target is unhealthy, it doesn’t mean the web server isn’t responding correctly. This happens because the ALB fails open, meaning it continues to send requests to targets when all targets are unhealthy.

<img width="642" height="205" alt="image" src="https://github.com/user-attachments/assets/050e3d56-9bb4-4002-bae6-11072c77015f" />

**Test 4 and finding**: I ran `tail -20 /var/log/nginx/access.log` to view the last 20 lines of the web server’s access log, and it shows the root cause: the path being requested is /health. The destination the load balancer requests on the target should be the root of the website, which is the path /

<img width="859" height="336" alt="image" src="https://github.com/user-attachments/assets/8db56eef-67dc-4ec4-8ac7-04e4a8104869" />

**Correction**: I changed the health check path back to /.

<img width="853" height="407" alt="image" src="https://github.com/user-attachments/assets/1c46deaa-4206-466d-b0a8-874547a57484" />

**Verification**:

I checked the target’s health status, and it shows as healthy. However, we still need to validate that localhost receives a successful response from the web server.

<img width="1375" height="517" alt="image" src="https://github.com/user-attachments/assets/9e6552f6-b66c-427d-8364-a8b68bd5372e" />

Localhost received a 200 response from the web server. This confirmed that the web server is serving requests locally. This is the same result as before the fault, since only the health check configuration was changed.

<img width="395" height="191" alt="image" src="https://github.com/user-attachments/assets/08f970b8-5282-4d38-a447-9ebbf1dd5f02" />

## Reflection

### What I understand now

I now have a much deeper understanding of how the load balancer, listener, target group, and health check work, and how traffic flows from the client to the web server. I've also gained a clearer understanding of security groups, including the difference between the ALB security group and the web server security group. Before this project, I didn't know what the target health status values meant, except for "healthy", but now I understand what they mean. I also understand the health check settings configured for the target group and how the health check request process works. Additionally, I've gained more experience in troubleshooting, understanding various HTTP status codes, and knowing how and when to use Linux commands. For example, a stopped web server gave me a 502, a missing security group rule gave me a 504, and a wrong health check path made the target unhealthy while the client still got a 200.

## Troubleshooting reasoning

### Traffic Diagram

Source ports 59900 and 19128 are for demonstration only. Temporary ports change with each connection.

<img width="1134" height="386" alt="image" src="https://github.com/user-attachments/assets/e71b5d03-cc58-4efc-90d9-03a2649a9478" />

