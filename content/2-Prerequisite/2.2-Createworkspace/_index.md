---
title : "Create workspace"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 2.2 </b> "
---

#### Create workspace
 
1. Access the [AWS Management Console](https://aws.amazon.com/console/)

   - Find **Cloud9**
   - Select **Cloud9**

![Amazon CDk](/images/2.2-prerequisite/0001.png?featherlight=false&width=90pc)

2. In the **AWS Cloud9** interface

   - Select **Create environment**

![alt text](image.png)

3. In the **Create environment** interface

   - **Name**, enter `ASG-Cloud9-Workshop`
   - **Environment type**, select **New EC2 instance**: EC2 Instance is initialized with the Cloud9 environment. The instance is accessed via Cloud9 IDE using the SSH method.
   - **Instance type**, select **t3.small(2GiB RAM + 2vCPU)**
   - **Platform**, select **Amazon Linux 2 (recommended)**
   - **Timeout**: after 30 minutes if EC2 Instance has no processes running, Cloud9 will stop Instance.
   - Leave the **Network** option as default (default VPC)

![alt text](image-1.png)

4. Select **Create**

![alt text](image-2.png)

5. Environment interface just initialized

![alt text](image-3.png)