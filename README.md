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
- Now we need to download our code from our s3 buckets onto our instance.
 ``` bash
cd ~/
aws s3 cp s3://BUCKET_NAME/app-tier/ app-tier --recursive
```
- This is the S3 source path.

- BUCKET_NAME → Replace with your actual S3 bucket name (e.g., my-app-deploy-bucket).

- /app-tier/ → The folder (prefix) in the S3 bucket containing the files you want to copy.
- Navigate to the app directory, install dependencies, and start the app with pm2
``` bash
cd ~/app-tier
npm install
pm2 start index.js
```
- To make sure the app is running correctly run the following:
  ``` bash
  pm2 list
  ```
- If you see a status of online, the app is running. If you see errored, then you need to do some troubleshooting. To look at the latest errors, use this command:
  ``` bash
  pm2 logs
  ```
 ### challenge faced :
 - the pm2 logs shows errors : How to fix it
 - Open your index.js:```nano index.js```
 - Look for the line where you import TransactionService.js.It should look like:```const TransactionService = require('./TransactionService');``` so when checked it was fine and syntactically correct
 - Run this to open the file:```nano ~/app-tier/TransactionService.js``` Look at the first few lines.Make sure that your file exports the addTransaction function properly.It should look something like this:
   ```
   // TransactionService.js
   function addTransaction(amount, desc) {
    console.log(`Adding transaction: ${amount}, ${desc}`);
    return 200;  // Example success code
   }

   module.exports = { addTransaction };
    ```
- Issue: You're requiring ./DbConfig, but it looks like that file is missing.
``` const dbcreds = require('./DbConfig'); ```
- The error you saw (MODULE_NOT_FOUND) is likely because Node.js cannot find DbConfig.js.
- Check if DbConfig.js exists:```ls -l ~/app-tier``` ![image](https://github.com/user-attachments/assets/de7536c6-8adc-4203-bd3c-9c60082b6eb5)
- so /Dbconfig not found because when i uploaded the file to s3 it was deleted as there was multiple copy and the one i uploaded was outside the /apptier folder,so i reuploaded the /Dbconfig.js file to this folder and performs the
-  ``` bash
cd ~/
aws s3 cp s3://BUCKET_NAME/app-tier/ app-tier --recursive```
- ```pm2 list```
- ![image](https://github.com/user-attachments/assets/011a44de-a1cf-46b0-bd1f-f7cde3be2c4c)
- ``` pm2 restart index```
- ``` pm2 logs ```
- ![image](https://github.com/user-attachments/assets/f04e5b5e-3f71-4f99-97c2-4b3ab1bf6dd2)
- so issue was fixed
- Right now, pm2 is just making sure our app stays running when we leave the SSM session. However, if the server is interrupted for some reason, we still want the app to start and keep running. This is also important for the AMI we will create:
  ``` pm2 startup ```
  - After running this you will see a message similar to this.
  - ![image](https://github.com/user-attachments/assets/15507d6c-f3af-47a4-a68a-544f44eca751)
  -  you should copy and past the command in the output you see in your own terminal. After you run it, save the current list of node processes with the following command:
### issue 2 :
- while copy pasting the commnad from the terminal ``` sudo env PATH=$PATH:/home/ec2-user/.nvm/versions/node/v16.20.2/bin /home/ec2-user/.nvm/versions/node/v16.20.2/lib/node_modules/pm2/bin/pm2 startup systemd -u ec2-user --hp/home/ec2-user``` the mistake was  is --hp /home/ec2-user insted hp/home/ec2-user is been done so created an error.
-  After you run it, save the current list of node processes with the following command:
  ``` pm2 save ```
## Test App Tier
- Now let's run a couple tests to see if our app is configured correctly and can retrieve data from the database.
-  To hit out health check endpoint, copy this command into your SSM terminal. This is our simple health check endpoint that tells us if the app is simply running
  ``` bash
  curl http://localhost:4000/health
```
- The response should looks like the following: ``` "This is the health check" ```
- Next, test your database connection. You can do that by hitting the following endpoint locally:
  ``` bash
   curl http://localhost:4000/transaction
  ```
### issue 3 :
the  out put of teh above command was ![image](https://github.com/user-attachments/assets/dbdec273-e5f6-4f07-ac8e-0c4193cce9f4)
- how to fix it  ``` pm2 logs ```
- ![image](https://github.com/user-attachments/assets/14f8c4c4-ffb5-486e-b997-ef486ae58e2f)
- so it was understoop that  db name was wrong ,in the Dbconfig file the dbname i gave as the databas-1 which is the name of the db created in aws but we should provide the name fo the db we created inside the mysql
-  so the database name was webappdb change the name inside the Dbconfig.js file and do ``` pm2 restartall```
-  then do the
- ``` bash
       curl http://localhost:4000/transaction
  ```
- the output ![image](https://github.com/user-attachments/assets/1a3264eb-dc3b-4582-9bd2-a095102f2f9d)
- You should see a response containing the test data we added earlier
- Your app layer is fully configured and ready to go.
## Internal Load Balancing and Auto Scaling:
- In this section of the workshop we will create an Amazon Machine Image (AMI) of the app tier instance we just created, and use that to set up autoscaling with a load balancer in order to make this tier highly available.
- Select the app tier instance we created and under Actions select Image and templates. Click Create Image. so image is created
- create Target Groups : While the AMI is being created, we can go ahead and create our target group to use with the load balancer. On the EC2 dashboard navigate to Target Groups under Load Balancing on the left hand side. Click on Create Target Group.
- The purpose of forming this target group is to use with our load blancer so it may balance traffic across our private app tier instances. Select Instances as the target type and give it a name.
-  set the protocol to HTTP and the port to 4000. Remember that this is the port our Node.ja app is running on. Select the VPC we've been using thus far, and then change the health check path to be /health. This is the health check endpoint of our app. Click Next.
-  We are NOT going to register any targets for now, so just skip that step and create the target group.
-  ![image](https://github.com/user-attachments/assets/b6a8afb7-10d2-4a20-ba98-a469280cadea)
-  create Internal Load Balancer:On the left hand side of the EC2 dashboard select Load Balancers under Load Balancing and click Create Load Balancer.
-  We'll be using an Application Load Balancer for our HTTP traffic so click the create button for that option.
-  After giving the load balancer a name, be sure to select internal since this one will not be public facing, but rather it will route traffic from our web tier to the app tier.
-  Select the correct network configuration for VPC and private subnets.
-  Select the security group we created for this internal ALB. Now, this ALB will be listening for HTTP traffic on port 80. It will be forwarding the traffic to our target group that we just created, so select it from the dropdown, and create the load balancer.
-  Launch Template : Before we configure Auto Scaling, we need to create a Launch template with the AMI we created earlier
-  Name the Launch Template, and then under Application and OS Images include the app tier AMI you created.
-  Under Instance Type select t2.micro. For Key pair and Network Settings don't include it in the template. We don't need a key pair to access our instances and we'll be setting the network information in the autoscaling group.
-  nder Advanced details use the same IAM instance profile we have been using for our EC2 instances.
-  Auto Scaling:We will now create the Auto Scaling Group for our app instances.
-  Give your Auto Scaling group a name, and then select the Launch Template we just created and click next.
-  On the Choose instance launch options page set your VPC, and the private instance subnets for the app tier
-  For this next step, attach this Auto Scaling Group to the Load Balancer we just created by selecting the existing load balancer's target group from the dropdown. Then, click next.
-  For Configure group size and scaling policies, set desired, minimum and maximum capacity to 2. Click skip to review and then Create Auto Scaling Group.
-  ![image](https://github.com/user-attachments/assets/f0d1b536-ac58-4def-a20c-3ddba0685ff5)

## Web Tier Instance Deployment 
- In this section we will deploy an EC2 instance for the web tier and make all necessary software configurations for the NGINX web server and React.js website.
- Update Config File : in the repo we have downloaded update the  application-code/nginx.conf in that replace [INTERNAL-LOADBALANCER-DNS] with your internal load balancer’s DNS entry.
- Then, upload this file and the application-code/web-tier folder to the s3 bucket.
- launch webtier instance tehn connect to it
### Configure Web Instance:
- We now need to install all of the necessary components needed to run our front-end application. Again, start by installing NVM and node
 ``` bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
source ~/.bashrc
nvm install 16
nvm use 16
```
- Now we need to download our web tier code from our s3 bucket ``` cd ~/
aws s3 cp s3://BUCKET_NAME/web-tier/ web-tier --recursive ```
- Navigate to the web-layer folder and create the build folder for the react app so we can serve our code:

  ```
  bash
  cd ~/web-tier
  npm install 16
  npm run build
  ``` 

- NGINX can be used for different use cases like load balancing, content caching etc, but we will be using it as a web server that we will configure to serve our application on port 80, as well as help direct our API calls to the internal load balance
  ```sudo amazon-linux-extras install nginx1 -y ```
### issue : command not founde error how i fixed it:
- ``` bash
  sudo yum install nginx -y
  sudo systemctl start nginx
  sudo systemctl enable nginx
  ```
- We will now have to configure NGINX. Navigate to the Nginx configuration file with the following commands and list the files in the directory:
 ```
  bash
  cd /etc/nginx
ls
  ```
- You should see an nginx.conf file. We’re going to delete this file and use the one we uploaded to s3. Replace the bucket name in the command below with the one you created for this workshop:
- also take backup of the nginx.conf file before removing
``` sudo cp nginx.conf nginx.conf_bkp  ```
- then remove the nginx .conf file 
```
bash
sudo rm nginx.conf
sudo aws s3 cp s3://BUCKET_NAME/nginx.conf .
```
- Then, restart Nginx with the following command
  ``` sudo service nginx restart ```
  ``` sudo nginx service status ```
  ![image](https://github.com/user-attachments/assets/b17ff1bb-f391-41e3-9197-996ad36d003c)

- To make sure Nginx has permission to access our files execute this command:
``` chmod -R 755 /home/ec2-user ```
- And then to make sure the service starts on boot, run this command:
  ``` sudo chkconfig nginx on ```
- Now when you plug in the public IP of your web tier instance, you should see your website, which you can find on the Instance details page on the EC2 dashboard. If you have the database connected and working correctly, then you will also see the database working.
### issue : site cant be reached : will get resolved after creating ASG and when you access it via the lb dns you could access the webapp
## External Load Balancer and Auto Scaling
- Select the web tier instance we created and under Actions select Image and templates. Click Create Image.
- create target group then.
- Create internet facing external Lb
- Create Launch Template.
- Create Auto Scaling group.
- ![image](https://github.com/user-attachments/assets/4f1bdc16-be3c-4dda-8781-c26ce28d1727)
- To test if your entire architecture is working, navigate to your external facing loadbalancer, and plug in the DNS name into your browser.
-![image](https://github.com/user-attachments/assets/015f4ad0-b853-4cb7-a829-e9844174853e)
- ![image](https://github.com/user-attachments/assets/67f265a7-a440-4ca0-98af-95cc58da6f98)
- ![image](https://github.com/user-attachments/assets/08ba324e-9e8c-4339-9bec-c4deb95a1e56)





 












 

 


-
## License

This library is licensed under the MIT-0 License. See the LICENSE file.
