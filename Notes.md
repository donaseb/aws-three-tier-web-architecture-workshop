# aws-three-tier-web-architecture-workshop
### Setup :
- download code fron github :
-  ```git clone https://github.com/aws-samples/aws-three-tier-web-architecture-workshop.git  ```
- Create S3 bucket following the steps in the workshop, in this bucket we will upload code and that will be deployed into the servers  .
- ![image](https://github.com/user-attachments/assets/0306fe8a-ca63-43e5-887d-ccd1ba709274)
- Create IAM EC2 Instance Role : go to IAM and create an EC2 role,and select ec2 as trusted entity
- When adding permissions, include the following AWS managed policies:AmazonSSMManagedInstanceCore,AmazonS3ReadOnlyAccess
## Networking and Security:
- In this section we will be building out the VPC networking components as well as security groups that will add a layer of protection around our EC2 instances, Aurora databases, and Elastic Load Balancers.
- 
