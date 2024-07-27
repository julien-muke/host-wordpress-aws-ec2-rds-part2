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
* Step 2. [Add EC2 security group to only allow traffic from Load Balancer](#ec2-alb)
* Step 3. [Domain transfer from 3rd party provider](#domain-transfer)
* Step 4. [Set up Free AWS SSL certificate](#ssl-certificate)
* Step 5. [Point our domain address to Load Balancer](#domain-to-lb)


## <a name="create-ec2-for-wordpress">➡️ Step 1 - Create Application Load Balancer on AWS</a>

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


10. 
