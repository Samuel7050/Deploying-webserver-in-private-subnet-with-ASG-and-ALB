# Deploying-webserver-in-private-subnet-with-ASG-and-ALB

Introduction 
In this project, we will be creating a Virtual Private Cloud (VPC) on Amazon Web Services (AWS) with two public and Private subnets and an autoscaling group to ensure high availability and scalability of our web application. We’ll also be configuring an Application Load Balancer to distribute traffic to the autoscaling group, and implementing security groups to ensure the safety and security of our infrastructure. So, let’s get started and build an infrastructure that can handle any amount of traffic with ease.
Create a custom VPC and all its dependencies. 
We are going to be creating a custom VPC with all associated dependencies using the VPC and More option. This will automatically create your VPC, subnets, Internet gateway, Nat Gateways and route tables, 
This option of creating VPC and its associated dependencies will help eliminate errors that can be encountered while creating each dependency individually.

1.	Open the Amazon VPC console at https://console.aws.amazon.com/vpc/.
2.	In the VPC Dashboard, choose Create VPC.
3.	Under VPC settings, choose VPC and more.
4.	Complete these fields as follows:
•	Keep Auto-generated selected under Name tag auto-generation. Change project to webserver
•	The IPv4 CIDR block should be 10.0.0.0/16.
•	Keep No IPv6 CIDR block option selected.
•	The Tenancy should remain Default.
•	Select 2 for the Number of Availability Zones (AZs).
•	Select 2 for the Number of public subnet and Private subnet
•	Choose Customize subnet CIDR blocks to configure the public and pri-vate subnet Ips address range. The public subnet CIDR block should be as shown in the screenshot below 
•	For Nat Gateway, choose 1per AZ
•	Choose None for S3 endpoint
5.	Choose Create VPC. It takes several minutes for the VPC to be create



 
 
 

Below shows the resource map of the VPC and the associated dependencies that will be automatically created.

 






VPC and Associated dependencies created successfully. See below 
 










Security Groups. A security group acts as a firewall that controls the traffic allowed to and from the resources in your virtual private cloud (VPC). You can choose the ports and protocols to allow for inbound traffic and for outbound traffic.
In this project we will be creating 2 security groups. 1. Webserver security group and 2. ALB security group
1. Webserver-sg
1.	Open the Amazon VPC console at https://console.aws.amazon.com/vpc/.
2.	In the navigation pane, choose Security groups.
3.	Choose Create security group.
4.	Enter a name (webserver-sg) and description (Allow SSH into webserver) for the security group. You cannot change the name and description of a security group after it is created.
5.	From VPC, choose the VPC and select the webserver-vpc created above

 
6.	Add security rules as shown below 

 
 
7.	You can add tags now, or you can add them later.
8.	Choose Create security group.

2. ALB-sg
1.	Repeat steps 1-3 above 
2.	Enter a name (alb-sg) and description (Allow inbound traffic)
3.	From VPC, choose the VPC and select the webserver-vpc created above

 
 
4.	You can add tags now, or you can add them later.
5.	Choose Create security group.

   Both security groups created successfully
 


 Create S3 bucket: Amazon Simple Storage Service (Amazon S3) is an object storage service that offers industry-leading scalability, data availability, security, and performance. We will be creating an S3 bucket to store our website files.  Remember, we need to create a Role for the EC2 instance hosting our website to be able to access the S3 bucket and pull the website files. This role will be attached to our EC2 instance 

1.	Sign in to the AWS Management Console and open the Amazon S3 console at  https://console.aws.amazon.com/s3/.
2.	In the left navigation pane, choose Buckets.
3.	Choose Create bucket.
a.	The Create bucket page opens.
4.	For Bucket name, enter a name for your bucket. (The bucket name must be unique because S3 is a global service) 
5.	For Region, choose the AWS Region where you want the bucket to reside.
6.	You can leave other settings has they are or adjust based on your needs 
7.	Click on create bucket button to create your bucket 

8.	 

 Upload your website files to S3 bucket created 
1.	Navigate to the bucket created in step 1 above and click on it 
2.	Choose Upload.
3.	In the Upload window, do one of the following:
a.	Drag and drop the website files (index.html) to the Upload window.
b.	Choose Add file, choose the files to upload, and choose Open.
4.	To upload the website file, navigate to the bottom of the page, choose Upload.
5.	Amazon S3 uploads your website folders. When the upload is finished, you see a success message on the Upload: status page.
 
 

Create EC2 Role with full Access to S3 Bucket 
1.	Open the IAM console at https://console.aws.amazon.com/iam/.
2.	In the navigation pane, choose Roles then choose Create role.
3.	On the Select trusted entity page, choose AWS service, and then select the EC2 use case. Choose Next.
4.	On the Add permissions page, search and select AmazonS3FullAccess. Choose Next.
5.	On the Name, review, and create page, enter a name and description for the role. Optionally, add tags to the role. Choose Create role.
 	 


Create a setup server
An Amazon EC2 instance is a virtual server in Amazon's Elastic Compute Cloud (EC2) for running applications on the Amazon Web Services (AWS) infrastructure.
A set up server is an EC2 instance server that will be launched in the public subnet of our VPC. On this setup server we will load our website site files and run our websites to ensure that everything is working fine.
We will then create an AMI from the setup server which will be carrying all website files pre-installed. This AMI will be used to launch our webservers in the private sub-nets.

Note that the webserver will be terminated as soon as we have created our AMI.

1.	Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.
2.	From the EC2 console dashboard, in the Launch instance box, choose Launch instance, and then choose Launch instance from the options that appear.
3.	Under Name and tags, for Name, enter Setupserver.
4.	Under Application and OS Images (Amazon Machine Image), do the follow-ing:
a.	Choose Quick Start, and then choose Amazon Linux. This is the operat-ing system (OS) for your instance.
b.	From Amazon Machine Image (AMI), select Amazon Linux 2(Free tier eligible).
5.	Under Instance type, from the Instance type list, Choose the t2.micro instance type, (eligible for the free tier).  
6.	Under Key pair (login), you can either create a new keypair or you choose from existing keypair. I will be using an existing Keypair. Use this link to learn more about Keypair creation.  https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html
7.	Next to Network settings, choose Edit. 
a.	For VPC – click the drop-down arrow and select your VPC (webserver-vpc)
b.	For Subnet info – click the drop-down arrow and select the (webserver- subnet-public1 -us-east-1a)
c.	For Security group - Choose Select existing security group and choose webserver-sg
8.	Keep the default selections for the other configuration settings for your instance.
9.	Review a summary of your instance configuration in the Summary panel, and when you're ready, choose Launch instance.
 
 

 




Our setup server is up and running.

 

Let’s attach our EC2 role created above. This will enable our setupserver to have full access to the website files stored in our S3 bucket.
•	Go to Ec2 dashboard and select the webserver 
•	Click on action, then security and then click on Modify IAM role
•	Click on the drop-down arrow to select the IAM role created earlier and attach.

 


Connecting to our Setupserver via SSH
•	Copy the ssh command as shown in the snippet below
 
•	Open window power shell on your local machine and paste the command
  

•	We are logged in into our setup server as seen below 

 

•	We will run the following commands to update and install Apache on the setup server. 
Please ensure you run all commands as a super user 
sudo su
yum update -y
yum install -y httpd
             systemctl enable httpd
             systemctl start httpd

 

 


 

•	Sync the setup server with our S3bucket using below command
 aws s3 sync s3://webservergos700 /var/www/html 
This command will download the website folder into the HTML directory as shown be-low

 
•	copy the content of the website folder into the html directory with the command 
   mv website/* /var/www/html

 
 We can remove the website folder since we have copied out the content. Use this command
rm -r website/ /var/www/html
 
With this done, we can now access our website through the setupserver
Copy the public IP address of our Setupserver and paste it on a web browser

 

Our website is deployed successfully as shown below

 


Create AMI from the setupserver
•	Select the setupserver, click on action, then image and templates and then click on create image 
•	Fill details for your AMI and then click create image
 
Our AMI is created and available 
 
•	Now its time to stop and terminate our setupserver
 
 

Create Target Group 
Target groups route requests to one or more registered targets, such as EC2 instances, using the protocol and port number that you specify.

1.	Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.
2.	On the navigation pane, under LOAD BALANCING, choose Target Groups.
3.	Choose Create target group.
4.	For Choose a target type, select Instances to register targets by instance ID
5.	For Target group name, type a name for the target group. This name must be unique per region per account
6.	For Protocol choose (HTTP) and Port Choose (80).
7.	For VPC, select our custom vpc (webserver-vpc)
8.	 For Protocol version, modify the default value as needed.
9.	 In the Health checks leave default settings
10.	Add one or more tags if you require 
11.	Choose Next.
12.	Choose Create target group.
 
 
 
 
 

Our Target group is successfully created. Note that no instance is associated with the target group. This association will be done when we create our launch template and autoscaling group
 

Create  Load balancer
Elastic Load Balancing automatically distributes your incoming traffic across multiple targets, such as EC2 instances, containers, and IP addresses, in one or more Availability Zones. It moni-tors the health of its registered targets, and routes traffic only to the healthy targets. Elastic Load Balancing scales your load balancer as your incoming traffic changes over time. It can automati-cally scale to the vast majority of workloads.

1.	Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.
2.	In the navigation pane, choose Load Balancers.
3.	Choose Create Load Balancer.
4.	Under Application Load Balancer, choose Create.
5.	Basic configuration
a.	For Load balancer name, enter webserver-alb
b.	For Scheme, choose Internet-facing or Internal. 
c.	For IP address type, choose IPv4 
6.	Network mapping
a.	For VPC, select webserver-vpc
b.	For Mappings, select the public subnets of both AZs 
7.	For Security groups, select an existing security group and choose the alb-sg created earlier
8.	For Listeners and routing, the default listener accepts HTTP traffic on port 80 and choose the target group created above. keep the default protocol and port. Keep other settings as default
9.	Tag as required or leave as default
10.	Create, review your configuration, and choose Create load balancer. 


 


 


 

 
 

Load Balancer created successfully


 


Create a launch template 

1.	Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.
2.	In the navigation pane, choose Launch Templates, and then choose Create launch template.
3.	For Launch template name, enter a descriptive name for the launch template.
4.	For Template version description, provide a brief description of this version of the launch template.
 
5.	For application and OS Images (AMI) Choose Owned by me and select the webserver-AMI created earlier
6.	For instance type, select t2. micro (free tier eligible)
7.	For Key Pair – Select and existing key pair (same as used earlier)
8.	For Security Group select the alb-sg and the webserver-sg created earlier 
9.	Leave other settings as default
10.	Review and create launch template

 
 

 

Launch Template created successfully
 








Create Autoscaling Group

Amazon EC2 Auto Scaling helps you ensure that you have the correct number of Amazon EC2 instances available to handle the load for your application. You create collections of EC2 instances, called Auto Scaling groups. You can specify the minimum number of in-stances in each Auto Scaling group, and Amazon EC2 Auto Scaling ensures that your group never goes below this size. You can specify the maximum number of instances in each Auto Scaling group, and Amazon EC2 Auto Scaling ensures that your group never goes above this size. If you specify the desired capacity, either when you create the group or at any time thereafter, Amazon EC2 Auto Scaling ensures that your group has this many instances. If you specify scaling policies, then Amazon EC2 Auto Scaling can launch or terminate instances as demand on your application increases or decreases.


1.	Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/, and choose Auto Scaling Groups from the navigation pane.
2.	Choose Create an Auto Scaling group.
3.	On the Choose launch template or configuration page, do the following:
a.	For Auto Scaling group name, enter a name for your Auto Scaling group.
b.	For Launch template, choose an existing launch template we created earlier 
c.	For Launch template version, choose default
d.	Verify that your launch template and choose Next.
4.	On the Choose instance launch options page, under Network, for VPC, choose the custom vpc created earlier (webserver-vpc)
5.	For Availability Zones and subnets, choose the 2 private availability Zones 
6.	Instance type- continue to the next step to create an Auto Scaling group that us-es the instance type in the launch template.
7.	Choose Next to continue to the next step..
8.	On the Configure advanced options page, configure the following options, and then choose Next:
a.	To register your Amazon EC2 instances with a load balancer, choose an existing load balancer created above 
b.	 For Health checks, Additional health check types, select Turn on Elastic Load Balancing health checks.
c.	Leave default settings for other options
 
9.	 On the Configure group size and scaling policies page, configure the follow-ing options, and then choose Next:
a.	For Desired capacity, enter the initial number of instances to launch. Also enter the minimum and maximum number of instances to launch
b.	Leave other options as default  
10.	(Optional) To receive notifications, for Add notification, configure the notifica-tion, and then choose Next.  
11.	(Optional) To add tags, choose Add tag, provide a tag key and value for each tag, and then choose Next.  
12.	On the Review page, choose Create Auto Scaling group.


 


 

 
 


 

 
Review and click on create autoscalling

 

Let’s confirm if our ASG is working by checking our webserver instances 

 

To confirm we can access our webserver through our load balancer, we will copy the DNS name from the load balancer as shown below

 

Paste on the web browser. Here we have our website 

 

Congratulations!!! Our website is successfully deployed on webserver running in private subnets with autoscaling and traffic is been routed via an application load balancer 

    Notes: 
•	Ensure proper clean-up is done on your AWS account so as not to incur unnecessary bills on your account.


