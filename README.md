# 🚀 Hosting a 3-Tier Web Application on AWS
- This project is a practical implementation of a 3-tier architecture deployed on AWS. I followed the **AWS 3-Tier Web Application Workshop** as a learning guide and documented my own understanding and deployment steps here.
- See [AWS Three Tier Web Architecture](https://catalog.us-east-1.prod.workshops.aws/workshops/85cd2bb2-7f79-4e96-bdee-8078e469752a/en-US)
## Architecture Overview
![Architecture Diagram](https://github.com/aws-samples/aws-three-tier-web-architecture-workshop/blob/main/application-code/web-tier/src/assets/3TierArch.png)
- In this architecture, a public-facing Application Load Balancer forwards client traffic to our web tier EC2 instances. The web tier is running Nginx webservers that are configured to serve a React.js website and redirects our API calls to the application tier’s internal facing load balancer. The internal facing load balancer then forwards that traffic to the application tier, which is written in Node.js. The application tier manipulates data in an Aurora MySQL multi-AZ database and returns it to our web tier. Load balancing, health checks and autoscaling groups are created at each layer to maintain the availability of this architecture.

### Setup :
---
- download code fron github :
- To clone the project:
> ```bash
> git clone https://github.com/dona-sebastian/clothing-store-3tier-app.git
> ```
- Create S3 bucket following the steps in the workshop, in this bucket we will upload code and that will be deployed into the servers  .
- ![image](https://github.com/user-attachments/assets/0306fe8a-ca63-43e5-887d-ccd1ba709274)
- Create IAM EC2 Instance Role : go to IAM and create an EC2 role,and select ec2 as trusted entity
- When adding permissions, include the following AWS managed policies:AmazonSSMManagedInstanceCore,AmazonS3ReadOnlyAccess
## Networking and Security:
---
- In this section we will be building out the VPC networking components as well as security groups that will add a layer of protection around our EC2 instances, Aurora databases, and Elastic Load Balancers.
- Create an isolated network with the following components:VPC,Subnets,Route Tables,Internet Gateway,NAT gateway,Security Groups
- Create Vpc and subnets,after creating vpc 6 subnets are created in two availability zones ,ie 2 public subnets which are 1 in each availability zones and 4 private subnets one in each availability zones.
- ![image](https://github.com/user-attachments/assets/67a0f129-36ee-4685-a290-8402671a62c1)
- Create Internet Gateway : for the public subnet to have internet access you need to attach internet gateway
- Create Nat gateway : for our instances in the app layer private subnet to be able to access the internet they will need to go through a NAT Gateway. For high availability, you’ll deploy one NAT gateway in each of your public subnets.  so create 2 Nat gateways 1 in each availability zones in public subnet.Nat gateway is in public subnets
- Create route tables: Create a route table for the public subnets,in the edit routes add the internet gateway and also associate the publicsubnets in web tier.create 2nd Route table and in the route add the NAT Gateway in az1 as targetand associate private subnet in az1,create a 3rd one also for the private subnet in az2 and add NAT gateway in Az2.
- Create Security Groups :
  1. SG1-externalLb :this security group allows inbound traffic from outside via HTTP port 80
  2. sg2-webtier: this  security group is for the web tier instances in inbound rules allows traffic from the external loadbalancer and your ip also
  3. sg3-internal-Lb-sg : this allows traffic from webtier instances to the internal lb
  4. sg4-private-instances-sg: allows traffic from internal Lb to privateinstances via port4000
  5. s5-db-sg : allows traffic from private instances to db intsnces via port 3306
     ![image](https://github.com/user-attachments/assets/3a4a0299-1771-4d97-9602-96c7cbd1fff9)

##  Database Deployment:
- In RDs dashboard ,create DB subnet group
- ![image](https://github.com/user-attachments/assets/f6d75087-350b-4c52-bee0-70fea857f221)

- Navigate to Databases on the left hand side of the RDS dashboard and click Create database.
- Start with a Standard create for this MySQL-Compatible Amazon Aurora database.
- Let everything else as default
- in  Templates section choose Dev/Test
- Under Settings set a username and password of your choice and note them down since we'll be using password authentication to access our database
- under Availability and durability change the option to create an Aurora Replica or reader node in a different availability zone. Under Connectivity, set the VPC, choose the subnet group we created earlier, and select no for public access  .
- Set the security group we created for the database layer, make sure password authentication is selected as our authentication choice, and create the database.
- When your database is provisioned, you should see a reader and writer instance in the database subnets of each availability zone. Note down the writer endpoint for your database for later use.
- ![image](https://github.com/user-attachments/assets/c6b3263b-d050-4fa3-8e53-d54697850fda)
![image](https://github.com/user-attachments/assets/1dd0b500-43f5-4709-9c59-be42d941b2dd)
## App Tier Instance Deployment:
- The app layer consists of a Node.js application that will run on port 4000.
- Launch app tier ec2 instance.
- Select a VPC and a subnet. For Subnet, select Private-App-Subnet. For Security Group, select PrivateInstanceSG, which you created in the previous step. For Auto-assign public IP, select Disable. click Advanced details.
- In the IAM instance profile field, select the IAM Role you created in the previous step. (In our example, we created the EC2andS3Role.) Finally, click Launch instance to create the instance.
- conncet to the ec2 using the session manager in console.
- When you first connect to your instance like this, you will be logged in as ssm-user which is the default user. Switch to ec2-user by executing the following command in the browser terminal:
  ``` bash
  sudo -su ec2-user
  ```
  
-  to make sure that we are able to reach the internet via our NAT gateways. If your network is configured correctly up till this point, you should be able to ping the google DNS servers:
-  ``` bash
   ping 8.8.8.8
   ```
-  ![image](https://github.com/user-attachments/assets/c499007d-df78-4f47-9e7a-b2d581e9cfc8)
### Configure Database
- Start by downloading the MySQL CLI:
- Follow these commands to install MySQL client:👇
``` bash
sudo wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
```
 ``` bash
 sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
```

 ```  bash
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
```
- Initiate your DB connection with your Aurora RDS writer endpoint. In the following command, replace the RDS writer endpoint and the username, and then execute it in the browser terminal:
  ```  bash
  mysql -h CHANGE-TO-YOUR-RDS-ENDPOINT of write -instance -u CHANGE-TO-USER-NAME -p
  ```
- You will then be prompted to type in your password. Once you input the password and hit enter, you should now be connected to your database
- Create a database called webappdb with the following command using the MySQL CLI:
  ``` bash
  CREATE DATABASE webappdb;
  ```
- You can verify that it was created correctly with the following command:
  ``` bash
   SHOW DATABASES;
  ```
- Create a data table by first navigating to the database we just created:
 ``` bash
USE webappdb;
```
- Then, create the following transactions table by executing this create table command:

 ``` bash
   CREATE TABLE IF NOT EXISTS transactions(id INT NOT NULL AUTO_INCREMENT, amount DECIMAL(10,2), description VARCHAR(100), PRIMARY KEY(id));    
```
- Verify the table was created:
  ``` bash
      SHOW TABLES;
  ```
- insert data into table for use/testing later:
  ``` bash
   INSERT INTO transactions (amount,description) VALUES ('400','groceries');
  ```
- Verify that your data was added by executing the following command:
  ``` bash
  SELECT * FROM transactions;
  ```
- When finished, just type exit and hit enter to exit the MySQL client.
### Configure App Instance :
-The first thing we will do is update our database credentials for the app tier. To do this, open the application-code/app-tier/DbConfig.js file from the github repo in your favorite text editor on your computer. You’ll see empty strings for the hostname, user, password and database. Fill this in with the credentials you configured for your database, the writer endpoint of your database as the hostname, and webappdb for the database. Save the file.
- In S3 bucket upload the Dbconfig file which is inside the application-code in that inside the apptier folder also upload the apptier folder in the aplication code folder to the S3 bucket
- Go back to your SSM session of apptier instance. Now we need to install all of the necessary components to run our backend application.
-  Start b installing NVM (node version manager).
``` bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
source ~/.bashrc
```
- Next, install a compatible version of Node.js and make sure it's being used
``` bash
  nvm install 16
nvm use 16
```
- PM2 is a daemon process manager that will keep our node.js app running when we exit the instance or if it is rebooted. Install that as well.
  ``` bash
  npm install -g pm2
  ```
-


 


-
## License

This library is licensed under the MIT-0 License. See the LICENSE file.
