Experiment-8

AWS Two-Tier Architecture



	Afreen Ali
                                                                                                                  500096949
Step 1) First Create a VPC.
Give it an Ipv4 range here I gave 10.0.0.0/16.
 













Step 2) After creating a VPC, create a subnet and select the VPC created.
 
Step 3) Now create subnets for the Web server (that will be for front-end EC2)
Give name and Ipv4 subnet CIDR block like this-
•	Web-sub1 – 10.0.5.0/24
•	Web-sub2 – 10.0.7.0/24
  








Step 4) Now create subnets for the database (that will be for database)
Give name and Ipv4 subnet CIDR block like this-
•	nat-sub1 – 10.0.3.0/24
•	nat-sub2 – 10.0.4.0/24
   





Step 5) Now create subnets for the alb (that will be for load balancer)
Give name and Ipv4 subnet CIDR block like this-
•	nat-sub1 – 10.0.1.0/24
•	nat-sub2 – 10.0.2.0/24

  
 
Step 6) Create an internet Gateway and attach it to VPC  







Step 7) Create a NAT Gateway 
Select the web server subnet and allocate Elastic IP.
 



Step 8) Now go to Routes and create a route for public, select your vpc.  



Step 9) Similarly create a private route.
 
Step 10) In public Route, click on edit route , add 0.0.0.0/0, and select target as Internet Gateway and add it.
 
Step 11)Select Public subnet and under Subnet association associate the following subnets-
•	Alb subnets (2)
•	Web server subnets(2)
A total of 4 subnets are to be associated with it.  
Step 12) Similarly In the private Route, click on edit route, add 0.0.0.0/0, and select target as NAT Gateway and add it.
 





Step 13) Select private subnet and under subnet association add the two subnets created for the database (nat-sub) .
  
Step 14) Now on to the Security Group we will create the security groups
1)	Load Balancer Security Group
2)	Webserver Security Group
3)	Database Security Group
For First Security Group (lb-sg), select your vpc
 
For the inbound Rules of lb security groups add the following-
•	Type - All traffic, Source – 0.0.0.0/0 (anywhere on internet)
•	Type - HTTP, Source – 0.0.0.0/0 (anywhere on internet)  

Similarly, create another Security group for the webserver 

For the inbound Rules of web security groups add the following-
•	Type - SHH, Source – 0.0.0.0/0 (to connect to instance if required)
•	Type - HTTP, Source – lb-Sg (lb security group that you created before)
 
Similarly, create a Security Group for the Database 
 
For the inbound Rules of lb security groups add the following-
•	Type - PostgreSQL, Source – Web-server sg(Web Server Security Group)
 
Resource Map of VPC
 


Step 15) Go to RDS - > Parameters Groups, Create and modify parameter group.
 
Edit these two parameters –
•	Password_encryption = md5
•	Rds.force_ssl =0
  








Step 16) Go to RDS and create a Postgres Database.	 
 
In templates select Free tier
 
Step 17) Here give –
•	DB instance identifier
•	Master Username
•	Master Password
 
Step 16) In the Connectivity option Select
•	Don’t connect to EC2
•	In VPC select the VPC that you Created
 
•	Select Public Access -No
•	Security Group – Select the one you Created for Database
  

Under Additional Configuration, give it a name ( preferably same as DB instance identifier)
 
And Select the Parameter group that you created now and create now.
 








Step 17) Go toEC2-> load balancer and create target group for instane.
 
Step 18) Give it a name , let the protocol be HTTP, IP address type Ipv4
And select the VPC you created and create the target group.
  
Step 19) Now go to Load balancer.
 
Select VPC you created and in subnet mappings add the subnets created for load balancer.
 
Under Security Groups select the Load balancer security group created previously,
And in the Listeners select the target group created and create your Load balancer.
 
Step 20) Create EC2 -> launch template
•	Give it a name
•	AMI - Ubuntu
 Under Network Settings select the security groups created for Webserver.
 
Under Advanced details
Select DNS-> enable resource-based Ipv4 DNS requests
 
And paste the bash script in the user data.
Script
#!/bin/bash
exec > >(tee /var/log/user-data.log|logger -t user-data -s 2>/dev/console) 2>&1

echo "Updating package lists..."
sudo apt update && echo "Package lists updated."

echo "Installing PostgreSQL..."
sudo apt install postgresql -y && echo "PostgreSQL installed."

echo "Installing Node.js..."
sudo apt install nodejs -y && echo "Node.js installed."

echo "Installing npm..."
sudo apt install npm -y && echo "npm installed."

echo "Cloning the To-do app repository..."
git clone https://github.com/Hexton09/To-do-app /home/ubuntu/To-do-app && echo "Repository cloned."

echo "Changing directory to the To-do app..."
cd /home/ubuntu/To-do-app || exit
echo "Current directory: $(pwd)"

echo "Installing npm dependencies..."
npm install && echo "Dependencies installed."

echo "Installing dotenv..."
npm install dotenv && echo "dotenv installed."

echo "Configuring port redirection from 80 to 3000..."
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 3000 && echo "Port redirection configured."

# Pre-seed the debconf database with our answers
echo iptables-persistent iptables-persistent/autosave_v4 boolean true | sudo debconf-set-selections
echo iptables-persistent iptables-persistent/autosave_v6 boolean true | sudo debconf-set-selections

echo "Installing iptables-persistent for port redirection..."
sudo apt-get install -y iptables-persistent && echo "iptables-persistent installed."

echo "Saving iptables rules automatically..."
sudo netfilter-persistent save && echo "iptables rules saved."

echo "Starting the application..."
npm start && echo "Application started."

 
 

app link - https://github.com/Hexton09/To-do-app.git
Step-21 : clone the repo into your github and add the following configuration in the .env file :

DB_HOST=’ database1.cfoe604g66rf.us-east-1.rds.amazonaws.com’
DB_PORT=5432
DB_DATABASE=' database1'
DB_USER='postgres'
DB_PASSWORD='password'
  


Step 22: Go to instances and create an Ubuntu instance
•	Select t2 micro as the instance type 
•	Create or select your key pair
 ’
Under network settings 
•	Select your VPC created 
•	Auto-assign public Ip -enable (if you want to SSH connect to check for issues)
•	Subnet – web server security group
 
Under advanced select enable resource-based Ipv4 DNS requests
And under user data paste the script and create the instance
  
Similarly, create 2 instances
Step 23) Now go to Target groups and click on register target and add both instances.
 
Check if the instances are healthy,if not then troubleshoot
Step 24) With this it should be healthy, and now open the DNS of Load balancer
 
 







