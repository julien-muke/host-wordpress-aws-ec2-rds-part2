# ![aws](https://github.com/julien-muke/Search-Engine-Website-using-AWS/assets/110755734/01cd6124-8014-4baa-a5fe-bd227844d263) How to Deploy WordPress Website on AWS using EC2, RDS, ALB and more (Part2).


## <a name="introduction">🤖 Introduction</a>

This demo is the the second part of the tutorial on Deploying a WordPress website on AWS. In the first part we covered Amazon EC2 for hosting, Amazon RDS for database management.

In the second part, we will utilize Amazon Application Load Balancer (ALB) for efficient traffic distribution, Amazon Route 53 for domain name management and DNS routing and get a free SSL certificate and point our domain address to Load Balancer. This comprehensive approach not only simplifies the deployment process but also equips your WordPress site with the robust infrastructure needed to handle varying levels of traffic, ensuring optimal performance and uptime.


## <a name="design">📐 Architecture Diagram</a>


![Blank diagram-19](https://github.com/user-attachments/assets/1d4fa91d-2efd-46f8-b8c4-bbd834f4dbfc)


## Architecture Diagram Overview

* Users will request to open WordPress website, that request will be received by Route 53 which is a domain Management Service in AWS.
* We will use Route 53 to host DNS entries of the website's domain.
* Route 53 will send request to Application Load Balancer (ALB), it handles distribution of the traffic, if you have multiple instances of the same website, it will handle all the incoming requests. 
* ALB also support SSL certificate through AWS Certificate Manager, we will use it to issue a new SSL certificate for our domain name and ALB will apply that SSL certificate and send request to EC2 instance.
* EC2 instance is a virtual server where we will install all the needed packages to run WordPress  and create files of our WordPress website.
* We will make EC2 and RDS accessible to public source and enforce security via Security Group rules.
* We will create an EC2 instance which will be our Virtual Server and RDS instance which will be used for Database Hosting.


## <a name="steps">☑️ Steps</a>

The procedure for deploying this architecture on AWS consists of the following steps:

* Step 1. [Create Application Load Balancer on AWS](#alb)
* Step 2. [Domain transfer from 3rd party provider](#domain-provider)
* Step 3. [Set up Free AWS SSL certificate](#ssl-certificate)
* Step 4. [Point our domain address to Load Balancer](#domain-to-lb)


## <a name="alb">➡️ Step 1 - Create Application Load Balancer on AWS</a>

We are going to create an Application Load Balancer to point to our EC2 Instance.

To configure your load balancer:

1. Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/
2. In the navigation pane, choose Load Balancers
3. Choose Create Load Balancer.

![alb1](https://github.com/user-attachments/assets/ebaff0aa-e955-4b27-a87d-ffde95a0d8e9)


4. Under Application Load Balancer, choose Create.

![screencapture-us-east-1-console-aws-amazon-ec2-home-2024-07-17-15_08_53](https://github.com/user-attachments/assets/e15e3606-11b6-4d8b-9a14-6335137a66a9)


5. For Load balancer name, enter a name for your load balancer `wp-lb`
6. For Scheme, choose Internet-facing. An internet-facing load balancer routes requests from clients to targets over the internet.
7. For IP address type, choose IPv4, Dualstack, or Dualstack without public IPv4. Choose IPv4 if your clients use IPv4 addresses to communicate with the load balancer. Choose Dualstack if your clients use both IPv4 and IPv6 addresses to communicate with the load balancer. Choose Dualstack without public IPv4 if your clients use only IPv6 addresses to communicate with the load balancer.

![screencapture-us-east-1-console-aws-amazon-ec2-home-2024-07-17-15_40_20](https://github.com/user-attachments/assets/142dd0ac-a08b-4b9a-8c6d-f9bbbaadbdd5)

8. For VPC, select the VPC that you used for your EC2 instances. If you selected Internet-facing for Scheme, only VPCs with an internet gateway are available for selection.
9. For Mappings, enable zones for your load balancer by selecting Subnets from two or more Availability Zones.

![screencapture-us-east-1-console-aws-amazon-ec2-home-2024-07-17-15_40_20 copy](https://github.com/user-attachments/assets/4c61b139-077c-4ff4-962c-a6ea0c86c54a)


10. For Security groups, let's create a new one. 

![screencapture-us-east-1-console-aws-amazon-ec2-home-2024-07-17-15_40_20 copy 2](https://github.com/user-attachments/assets/13d0d131-c053-4587-a3c7-01faf0e1e508)


11. Enter Security group name `wp_lb-SG`
12. Make sure your default VPC is selected
13. For Inbound rules, we are going to create 2 new rules, one for `HTTP` and `HTTPS` Rules with source of `0.0.0.0/0`
14. Keep Outbound rules as default
15. Use a tag as a label that you assign to an AWS resource with Key=`Name` and Value=`LB-SG`
16. Click Create security group

![screencapture-us-east-1-console-aws-amazon-ec2-home-2024-07-17-15_15_26](https://github.com/user-attachments/assets/5367e45c-3574-4487-a120-1c8fe1c318bb)


17. Go back to Application Load Balancer and select the new `wP_lb-SG` security group.
18. Under Listeners and routing, a listener is a process that checks for connection requests using the port and protocol you configure, let's create a new target group, choose Create target group.

![Screenshot 2024-07-27 at 11 45 43](https://github.com/user-attachments/assets/14a6096f-9ef4-4659-ab55-ff7d210959bd)

19. Choose Instances as target type 
20. Enter Target group name `wp-site-TG`
21. Select your default VPC and keep everything as default, click Next

![screencapture-us-east-1-console-aws-amazon-ec2-home-2024-07-17-15_20_09](https://github.com/user-attachments/assets/5a723731-0e23-4b96-a676-e550ebc9ba9b)

22. Under Register targets, let's select the EC2 instance and click on Include as pending below.
23. Once the EC2 instance is add to the Review targets, click Create target group.

![screencapture-us-east-1-console-aws-amazon-ec2-home-2024-07-17-15_21_54](https://github.com/user-attachments/assets/7fa9d45d-4e07-44ba-99f1-d1b73b6d2c8e)


24. We have created our WordPress site Target group and currently it's not associated with any load  balancer, let's add it to our load balances. Back to Application Load Balancer select the new target group `wp-site-TG`


![screencapture-us-east-1-console-aws-amazon-ec2-home-2024-07-17-15_40_20 copy 3](https://github.com/user-attachments/assets/8964594a-07f4-47e6-b670-ded627517b25)


25. Keep the rest as default.
26. Review the load balancer configurations, we've created:
* An Internet-facing Load Balancer
* A Security groups
* A Network mapping with 1 VPC and 3 availabity zones
* 1 Target group 

![screencapture-us-east-1-console-aws-amazon-ec2-home-2024-07-17-15_40_20 copy 4](https://github.com/user-attachments/assets/4e2fba1a-55c8-4468-9d4c-79f50db7ea8c)

The Target group will be associated to the Load Balancer, and the EC2 instance will be added to the Target group and make sure the EC2 instance is healthy, lastly our load balancer will be in active State.

Let's test our Application Load Balancer, copy DNS name of the load balancer and and open it on a new browser.

![alb](https://github.com/user-attachments/assets/d9efa1aa-2456-497f-8a83-9e1fb2e6b7f0)

As you can see below we can now access our WordPress site from Load Balancer DNS.


![wp](https://github.com/user-attachments/assets/cee5c9d2-5058-4862-ab66-f0a94dd4f46b)


Next, we need to edit the security group in such a way that our application load balancer can only be accessed from HTTP and HTTPS traffic from our load  balancer Security Group and not from all IP addresses(`0.0.0.0/0`).

* Go to EC2 Instance conlose, then choose security group.
* Copy the ID of the load balancer Security Group 
* select Security Group attached to EC2 instance and go to inbound rules click on edit inbound.

![sg](https://github.com/user-attachments/assets/cc1a728a-4e59-4fb4-9f05-f15080834488)

* We have to create two new rules which only allow HTTPS and HTTPS traffic from our load balancer Security Group and not from the whole.
* Paste the security group ID from application load balancer and click Save.

![sg2](https://github.com/user-attachments/assets/570dd5b0-b7bd-4a77-a8ec-c688551b4c21)

We are now only allowing traffic  from ALB and not from any other source.


## <a name="domain-provider">➡️ Step 2 - Domain transfer from 3rd party provider</a>

