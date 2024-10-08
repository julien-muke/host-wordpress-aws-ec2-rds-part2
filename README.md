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
* Step 2. [Add a custom domain managed by a third-party DNS provider](#domain-provider)
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


## <a name="domain-provider">➡️ Step 2 - Add a custom domain managed by a third-party DNS provider</a>

If you are not using Amazon Route 53 to manage your domain, you can add a custom domain managed by a third-party DNS provider to your application.

We are going to create a Public Hosted Zone, which is a container that holds information about how you want to route traffic on the internet for a specific domain, such as `example.com`, after you create a hosted zone, you create records that specify how you want to route traffic for the domain and subdomains.

To create a public hosted zone using the Route 53 console:

1. Sign in to the AWS Management Console and open the Route 53 console at https://console.aws.amazon.com/route53/
2. If you're new to Route 53, choose Get started under DNS management. If you're already using Route 53, choose Hosted zones in the navigation pane.
3. Choose Create hosted zone.

![route-53-1](https://github.com/user-attachments/assets/f7734f6e-4c48-4f9a-a586-1387cfd09cda)


4. In the Create Hosted Zone pane, enter the name of the domain that you want to route traffic for, in my case it's `julienmuke.cloud` which is domain that i purchased from [hostinger.com](www.hostinger.com)
5. For Type, accept the default value of Public Hosted Zone.
6. Choose Create.

![Route-53-Global](https://github.com/user-attachments/assets/a87632e9-420a-40b8-aa4c-251d1c62e57f)

Note: By default you will get two records for your domain which are SOA and NS.
NS stands for Name Server record which determine the location of your domain and help you manage mapping, we have to add this name servers to our domain provider so the provider can know where is your DNS hosted in my case it's [hostinger.com](www.hostinger.com).

Now, let's add the Name Server record to Hostinger. 

* Copy the Name Server value from Route 53.

![53-2](https://github.com/user-attachments/assets/1c7b8db7-1682-4776-a130-699667612f16)

* Paste the Name Server value to DNS Nameservers in Hostinger.

![53-3](https://github.com/user-attachments/assets/790ec3ce-742a-4eb6-8cd1-32fb57f0adee)

* Next, we will Create records that specify how you want to route traffic for the domain, so that when anyone opens our domain URL it will show the WordPress website from the load balancer.

* Click on Create record

![53-3](https://github.com/user-attachments/assets/84e95410-4145-4391-b93b-17dc967846b6)

* Keep the Record name blank 
* There are various DS record types, make sure to select **A - Routes traffic to an IPv4 address and some AWS resources**
* Enable Alias
* Select Alias Alias to Application and Classic Load Balancer.
* Select your region where you have created your load balancer i will select North Virginia
* Select our WordPress application load balancer 
* Keep everything else default and click on create records

![53-4](https://github.com/user-attachments/assets/7b30b70a-cc5e-4928-bed0-088ba752d2ad)

Next, we will create a record to migrate traffic from `www` to our domain, so if anyone adds `www` in front of your domain, it will not throw error rather it will redirect to your main domain.

* Enter `www` record name, make sure you select record type **A - Routes traffic to an IPv4 address and some AWS resources**
* Enable Alias
* Choose **Alias to another record in this hosted zone**
* Select record created `julienmuke.cloud.` previously and click create record.

![53-5](https://github.com/user-attachments/assets/87a4539b-4886-44a3-ac38-a3ee56c29beb)


Now any traffic coming to `www` record will go to load balance at DNS.


## <a name="ssl-certificate">➡️ Step 2 - Set up Free AWS SSL certificate</a>

Let's move to AWS Certificate Manager (ACM) to request a Free SSL certificate for our domain.

To request an ACM public certificate (console):

1. In the AWS Management Console and open the ACM console at https://console.aws.amazon.com/acm/home.
2. Choose Request a certificate.
3. Choose Request a public certificate, click Next.

![53-6](https://github.com/user-attachments/assets/8e6e8413-1b70-45d9-9ab0-dcd74f1834d5)

4. In the Domain names section, type your domain name, mine is `julienmuke.cloud`
5. To add another name, choose Add another name to this certificate and type the name in the text box. This is useful for protecting both a bare or apex domain (such as example.com) and its subdomains such as (*.example.com) in my case i will add `*julienmuke.cloud`
6. In the Validation method section, choose either DNS validation – recommended.
7. In the Key algorithm section, choose RSA 2048 (default) then click Request.

![Request-public-certificate-Certificate-Manager-us-east-1](https://github.com/user-attachments/assets/e90fe1f1-dd6c-4370-8ccc-1e01b5aa0b9a)

8. Click on view certificate, it will be in Pending validation status.(it will take 3-5 minuntes to validate).
9. Next, click on Create DNS records in Amazon Route 53, which will add records to our host Zone.

![53-7](https://github.com/user-attachments/assets/989128aa-4677-477d-83cf-00fb7d45e32f)

10. Select your Domains, and click Create records.

![53-8](https://github.com/user-attachments/assets/7122fbe7-4ca5-4244-9a80-c5cfb7734f81)


11. Let's add SSL certificate to our Load Balancer:

a. Go to the ECS conlose, then select Load Balancer, click on the existing one
b. Under Listeners and rules, click on Add listener

![53-10](https://github.com/user-attachments/assets/e8cce2bf-d031-49cc-9cc5-cf60000b5eef)

c. Select `HTTPS` as Protocol, and the Port will be 443.
d. Select the Target group `wp-site-TG`


![screencapture-us-east-1-console-aws-amazon-ec2-home-2024-07-18-12_26_44 copy](https://github.com/user-attachments/assets/36e137e9-3ea1-4428-ba38-68abb11ce98d)


e. Under Default SSL/TLS server certificate, select the certificate that will be applied as the default SSL/TLS server certificate for this load balancer's secure listeners.
f. Keep everything as default and click Add

![screencapture-us-east-1-console-aws-amazon-ec2-home-2024-07-18-12_26_44 copy 2](https://github.com/user-attachments/assets/240d7e93-9b7a-45e2-ad69-b280c6a5b36a)

Next, let's redirect all traffic coming from HTTP to HTTPS.

1. Go to EC2 console, then Load Balancer and select the `wp-lb`
2. Under Listeners and rules select `HTTP:80`, click on Edit Rules

![3242](https://github.com/user-attachments/assets/d4791833-3d0e-438a-8fdb-b67df4b1e713)

3. Select Default rules, go to Actions and select Edit rule

![6545](https://github.com/user-attachments/assets/b53c6f3b-e1dc-4d52-be11-3aa8b89009f2)


