#Connect to IRC

http://webchat.freenode.net/
Channel: phpnwaws

#Creating an IAM account

1. Log into the AWS managment console
2. Select IAM from the services menu 
3. Select "Users" from the left hand nav
4. Click "Create New Users" button 
5. Enter a username in the box
6. Click "Create"
7. Download the security credentials
8. Click "Close"
9. Click on the newly created user
10. Under "Security Credentials" click manage password
11. Click auto assign password and download the newly created credentials
12. Under "Permissions" Click "Attach User Policy" click the "Select button under "Administrator Access" 

# Building The Network

1) Select 'VPC' from the services menu
2) Click 'Create VPC'
3) Enter 'tutorialvpc' in the name box and '10.0.0.0/0' in the CIDR box and click 'Yes, Create'
4) Click 'Subnets' on the left hand nav
5) Click 'Create Subnet'
6) Enter public1a in the name box and select 'eu-west-1a' from the availability zone drop down and 10.0.1.0/24 in the CIDR box.
7) Click 'Internet Gateways' on the left hand nav
8) Click 'Create Internet Gateway', enter 'publicigw' in the name box and click 'Yes, Create'
9) Right click on the new internet gateway and select 'Attach To VPC', select the previously created VPC and click 'Yes, Attach'
10) Click 'Route Tables' on the left hand nav
11) Click 'Create Route Table' and name it 'PublicRoute'
12) Click the new route table and in the bottom pane select 'Routes'
13) Click 'Edit' and add a new route with the destination "0.0.0.0/0" and the Target of the previously created gateway. Click 'Save'
14) Finally click 'Subnets' on the left hand nav. Select the previously created subnet and in the bottom pane Click 'Route Table', 'Edit' then change the route table to 'PublicRoute'.

# Launching an Instance

1) Select 'Ec2' from the services menu.
2) Click 'Launch Instance'
3) Select 'Ubuntu 14.04 HVM' from the list of images
4) Click 'Next: Configure Instance Details'. Change the following:
 - Auto Assign Public Ip: Enable
5) Click 'Next: Add Storage'
6) Click 'Next: Tag Instance'. Name the instance 'Webserver Test'
7) Click 'Next: Configure Security Group'
8) Create a new security Group with the following settings: 
 - Name: webserver
 - Description: webserver sg
 Add a rule: 
 - Type: HTTP
9) Click 'Review And Launch'
10) Click Launch
11) Select 'Create A New Key Pair' and name it phpnwtest. Save it to your desktop. 

# Logging Into The Instance

1) From the EC2 'Instances' screen copy the public ip of the newly created instance
2) Wait for the instance status to be 'running' and checks to be '2/2 Passed'
3) Open a terminal and SSH to the new instance
 - ssh -i /path/to/key ubuntu@IPADDRESS
4) SSH into the new instance and install apache 2
 - sudo apt-get update
 - sudo apt-get install apache2 php5
 - sudo nano /var/www/html/hello.php 
 - Enter some php and save the file. I suggest "<?php echo 'hello BEN'; ?>"

# Making An AMI

1) SSH into the new instance and install apache 2
 - sudo apt-get update
 - sudo apt-get install apache2
2) Input the IPADDRESS into a web browser screen and check that apache is running 
3) From the EC2 'Instances' screen right click on the newly created instance and click 'Create Image'
4) Name the image 'Webserver' and add the description 'Apache2 webserver'
5) Click on 'AMIs' on the left hand nav you should see a newly created AMI. Wait until the status reads 'available'
6) You can now launch new apache instances into your VPC. Try It!

# Cloud Formation And Cloud Former

## Create A S3 Bucket

1) Select 'S3' from the services menu
2) Click "Create Bucket"
3) Enter a bucket name, and pick a region (eu). Click 'Create'

## Setting Up A Cloud Former Instance

1) Select 'Cloud Formation' from the services menu
2) Click the "Create A Stack" button
3) Name the stack 'CloudFormer' and select "Cloud Former" from the 'source' select box
4) Click 'Next' until ariving at the review screen, acknowlege the usage charges and click "Create"
4) When the stack status reaches 'COMPLETE', click on it and in the bottom pane select 'Outputs'
5) Navigate to the URL shown, you may need to refresh occasionally until the application comes up

## Creating a Template

1) Click the 'Create Template' button
2) Tick 'Select All Resources'

Navigate through the following screens selecting: 
DNS: NA
VPC: Select the VPC
VPC Network: Select subnet, internet gateway and DHCP options
VPC Security: Unselect ACL, Select Routing Tables
Network: Select the load balancer, Unselect Cloud Front Distributions
Compute: Select the previously created instance
Storage: Unselect S3 Buckets
Application Services: NA
Config: NA
Security: Select 'webserver' security group, unselect everything else
Operational Resources: NA

3) On the sumary screen rename the resources to be inline with the elements of the architecure they represent. Use the aws managment console as a a guide to convert id numbers to names.
4) Click 'Continue'
5) Click 'Save Template' and on the next screen 'Launch Stack'


# Upgrading To Mutliple Availability Zones 

1) Select 'VPC' from the 'Services' menu
2) Click Subnets on the left hand nav
3) Click 'Create Subnet'. Name the subnet 'public1b', use the CIDR '10.0.2.0/24' and use availability zone 'eu-west-1b'
4) Click 'Subnets' on the left hand nav, Select the new subnet and change its routing table to be PublicRoute (as above)

# Adding A Load Balancer

1) Select 'EC2' from the 'Services' menu
2) Click 'Load Balancers' on the left hand nav
3) Click 'Create Load Balancer' 
4) Name it 'testloadbalancer', create it inside the test VPC and click 'Continue'
5) Change the following settings and click 'Continue':  
 - Health Check Interval: 10 
 - Healthy threshold: 2 
6) Add the two previously created subnets and click 'Continue'
7) Select the 'Webserver' security group and click 'Continue'
8) Add the previously created webserver to the the load balancer and click 'Continue'
9) Click 'Conitnue' then 'Create'
10) Navigate to the DNS Name shown in the newly created load balancers bottom pane

# Creating An Auto Scaling Group

1) Select 'EC2' from the 'Services' menu
2) Click 'Auto Scaling Groups' on the left hand nav
3) Click 'Create Auto Scaling Group'
4) Click 'Create Launch Configuration'
5) Click 'My AMIs' and select the webserver AMI previously created
6) Click 'Configure Details'
 - Name: WebserverLaunchConf
 Advanced
 - Assign Public Ip Address
7) Click 'Next Add Storage', Click 'Next: Configure Security Group'
8) Select the webserver security group and click 'Review' and then 'Create Launch Configuration'
9) Select the previously created tutorial Key
10) Use the following values:
 - Group: webservers
 - Group Size: 2
 - Network: tutorialvpc
 - Subnets: (select both)
 Advanced
 - Receive Traffic From Loadbalancers: (select testloadbalancer)
11) Click 'Next Configure Scaling Policy' and select 'Keep this group at its initial size'
12) Click 'Review', then 'Create Auto Scaling Group'
13) Monitor the new servers coming up from the instances screen

# Upgrading To A More Secure Design     

## Updating Security Groups
1) Create a new security group 'bastion'. Allow SSH access from the internet to this group. 
2) Add a new SSH access rule to the webserver group. Allow SSH traffic from the new 'Bastion' security group
3) Delete the old SSH access rule from the 'Webserver' security group

## Using Private Subnets
1) Add two additional subnets in availability zones 1a and 1b. Name them 'private1a' and 'private1b' using CIDRs '10.0.3.0/24' and '10.0.4.0/24'
2) Delete the current auto scaling group
3) Create a new launch configuration as before (using the webserver ami). However this time on the 'Configure Details' under 'Advanced' select the option 'Do not assign a public IP address to any instances.'
4) When the launch config is created select the option to launch an auto scaling group and have it deploy instances into subnets 'private1a' and 'private1b'. Start with two instances. 

## Create A Bastion Host
1) Launch an Ubuntu instance using the stock ubuntu ami as in previous examples. 
 - Ensure it has a public IP address and is in either public1a or public1b
2) SSH to this instance and use it to SSH into your web instances 

#Creating A Cloud Front Distribution Backed By S3

##Create an S3 bucket
1. Select "S3" from the service menu on the upper right hand side
2. Click "Create Bucket"
3. Give the bucket a name (this has to be unique across the region)
4. Click "Create"
5. Click "Actions" then "Upload"
6. Upload the test image in this folder

##Create a Cloud Front Distribution
1. Select "Cloud Front" from the services menu on the upper right hand side
2. Click "Create Distribution"
3. In the Origin Domain Name box select your previously created bucket
4. Click "Create Distribution"
5. Wait for the status to move from "InProgress" 

NOTE: Setting up the distribution takes a long time!

##Altering Your Bucket Permissions
1. Select "S3" from the service menu on the upper right hand side
2. Click your previously created bucket
3. Click "Properties" on the upper left hand side of the screen
4. Click "Permissions"
5. Click "Apply Bucket Policy"
6. Copy the policy from this folder into the text box, remembering to change the example bucket name to the name you created above. 

##Testing
1. Select "Cloud Front" from the services menu on the upper right hand side
2. Click the "i" button on your previously created cloud front distribution
3. Copy the "domain name" value and paste it into your browser. Add /php.png to the end and hit return
4. You should see the famous elephant

# Setting Up RDS

## Create A Database Security Group
1. Select 'VPC' from the services menu and 'Security Groups' from the left hand nav
2. Create a new security group 'database' as in previous examples
3. Allow Mysql Connections from the webserver group 
4. Allow Mysql and SSH connections from the bastion group (to enable mysql gui access via a tunnel).

## Create an RDS Subnet Group
1. Select 'RDS' from the services menu and 'Subnet Groups' from the left hand nav
2. Click 'Create DB Subnet Groups'
3. Give the group a name, description and select the previously created VPC.
4. Select Availability Zonw eu-west-1a and then add the private subnet from that zone
5. Repeat the process for eu-west-1b (private subnets are 10.0.3.0 and 10.0.4.0)

## Launch A Mysql Cluster
1. Select 'RDS' from the services menu and 'Instances' from the left hand nav
2. Click 'Launch DB Instance'
3. Select 'Mysql'
4. Select 'No'
5. On the 'Specify DB Details' page modify the following: 
 - DB Instance Class: db.t2.micro
 - Multi-AZ Deployment: yes
 - Use Provisioned IOPS: no
 - Add a database identifier, username and password
6. On the 'Configure Advanced Settings' page:
 - VPC (select the test vpc)
 - Select two subnets
 - Add the Database security group
 - Click 'Launch Instance'

 NOTE: Setting up the database instance takes a long time!
