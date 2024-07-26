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

* Step 1. [Domain transfer from 3rd party provider](#domain-transfer)
* Step 2. [Free AWS SSL certificate](#ssl-certificate)
* Step 3. [Point our domain address to Load Balancer](#domain-to-lb)


## <a name="create-ec2-for-wordpress">➡️ Step 1 - Transer Domain Name to AWS Route 53</a>

