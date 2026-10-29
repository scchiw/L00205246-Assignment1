CloudFormation Lab Write-Up - Assignment 1

Student Name: Seang  Chiw

Student ID: L00205246

GitHub Repo: https://github.com/scchiw/L00205246-Assignment1




1. Introduction and Adopted Approach
This report to document the CloudFormation scripts created for Assignment 1 and the testing done in my AWS 6-Month Free Tier account. There are two separate YAML files and it can be tested independently without breaking the other lab:
-task1-web.yaml: Creates the S3 bucket to store the static webpage (index.html), the IAM role for S3 permissions, the Security Group firewall rules and the public EC2 server to host the website.
-task2-threetier.yaml: Creates the 3-tier network (VPC) with a Public Web subnet, a Private App subnet and an Isolated Database subnet.


2. Task 1 - Static Website with S3 and EC2 (task1-web.yaml)

2.1 The Test System
The testing platform used is an AWS 6-Month Free Tier account in the us-east-1 region. In the first test using t2.micro, AWS reported an error that the instance type was not eligible for the Free Tier. Another error also came up when using LabInstanceProfile because that profile only works in AWS Academy lab accounts.
To fix this, I changed the instance type to t3.micro and used the Amazon Linux 2023 AMI ID (ami-08c46876c64a927b8) from my EC2 console. Secondly, I created an IAM Role (WebServerS3Role) and Instance Profile inside the YAML file so the EC2 server has permission to upload and download files from the S3 bucket.

2.2 Description of Method Used

Firstly, an S3 bucket named l00205246-static-web-163120011174 is created with WebsiteConfiguration enabled for index.html because Amazon S3 standard buckets treat uploaded files as generic data blobs by default and also let the S3 storage engine to host the container as a static web server. Specifying the IndexDocument as index.html ensures that any web traffic reaching the bucket domain is automatically directed to my default home screen asset instead of returning a directory access error.


The ObjectOwnership set to BucketOwnerEnforced because modern AWS security standards disable legacy Access Control Lists (ACLs) by default and ensures that my AWS account automatically takes full control and ownership of the index.html file as soon as the EC2 instance uploads. This prevents access permissions errors and ensures the file can be read



Secondly, the EC2 server starts up, updates the OS and installs the Apache web server (httpd). It then creates the index.html file with my header "L00205246 - Seang Chiw" in the /tmp folder.

In order for index.html to be stored in S3 bucket and copy to Apache web folder (/var/www/html/index.html) for website hosting, below command was ran.
aws s3 cp /tmp/index.html s3://${WebsiteS3Bucket}/index.html
aws s3 cp s3://${WebsiteS3Bucket}/index.html /var/www/html/index.html


2.3 Description of Results
After the stack operation completed (CREATE_COMPLETE), I checked the S3 bucket in the AWS console and confirmed that index.html (315.0 B) was stored inside the Objects tab. I then opened the WebsiteURL link from the Outputs tab and verified that "L00205246 - Seang Chiw" showed up properly as the header on the webpage.







3. Task 2 - 3-Tier Network Architecture (task2-threetier.yaml)


3.1 Description of Method Used
For Task 2 (task2-threetier.yaml), I created a VPC in us-east-1a with the IP range 10.46.0.0/16 (using 46 from my L-number L00205246). The network is divided into three subnets to separate public traffic from private backend servers:
- Tier 1 (PublicWebSubnet - 10.46.10.0/24): Connected to the PublicRouteTable which sends 0.0.0.0/0 traffic to the Internet Gateway (L00205246-VPC-IGW). MapPublicIpOnLaunch is set to true so web servers here get a public IP.
- Tier 2 (PrivateAppSubnet - 10.46.20.0/24): Connected to the PrivateRouteTable which has no route to the Internet Gateway. MapPublicIpOnLaunch is set to false so backend servers cannot be reached from the internet.
- Tier 3 (IsolatedDBSubnet - 10.46.30.0/24): Also connected to the PrivateRouteTable with MapPublicIpOnLaunch set to false to keep the database tier isolated.

3.2 Description of Results
After running the stack in CloudFormation, I checked the VPC Resource Map in the AWS console that Web Tier subnet connects to the Internet Gateway meanwhile App Tier and Database Tier subnets with Private Route Table has no internet access.

4. Best Practices and Lessons Learned
From security aspect in task1-web.yaml, Port 80 is open to 0.0.0.0/0 for viewing the website however Port 22 (SSH) only restricted to 10.46.0.0/16. Keeping SSH open to publin is a risk therefor limiting to internal network address range allows future troubleshooting without exposing the server. 

In task2-threetier.yaml, MapPublicIpOnLaunch is set to false on the private subnets to hide backend resources.
- Manageability: All resources across both files are tagged with CostCenter: ATUDevOps and Owner: Seang Chiw to keep track of resources and costs. I also used !GetAtt WebsiteS3Bucket.DomainName and !Sub in the Outputs section instead of hardcoding values.
- Cost Control: After testing both stacks and checking the results, I deleted index.html from the S3 bucket and deleted both stacks in CloudFormation so the EC2 server would not use up my 250 free hours or $120.00 Free Tier credits.



5. References and AI Declarations
AWS Documentation Used:

https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-ec2-subnet.html
AWS::EC2::Subnet
 
https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-properties-s3-bucket-ownershipcontrols.html
AWS::S3::Bucket OwnershipControls

https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-ec2-securitygroup.html
AWS::EC2::SecurityGroup
 
 

I used ChatGPT to understand and learn CloudFormation YAML syntax. It also help me to troubleshoot AWS console errors before modifying the code myself.

-Troubleshooting S3 and EC2 errors: Fixed the RequirePublicBuckets error by using OwnershipControls: BucketOwnerEnforced. Changed t2.micro to t3.micro due to my 6-month Free Tier account and replaced LabInstanceProfile with my own IAM Role (WebServerS3Role) so the EC2 server could copy index.html to S3.
-Customizing the scripts: Asked how to use !GetAtt WebsiteS3Bucket.DomainName for the output, set the VPC subnets to 10.46.x.0/24 in us-east-1a and added CostCenter: ATUDevOps tags across task1-web.yaml and task2-threetier.yaml.




