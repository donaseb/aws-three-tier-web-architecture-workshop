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
  2. sg2-webtier: 


-
## License

This library is licensed under the MIT-0 License. See the LICENSE file.
