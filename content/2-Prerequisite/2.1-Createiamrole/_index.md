---
title : "Create IAM Role"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 2.1 </b> "
---
 
#### Create IAM Role

1. Access the [AWS Management Console](https://aws.amazon.com/console/)

   - Find **IAM**
   - Select **IAM**

![Amazon CDk](/images/2.1-prerequisite/0001.png?featherlight=false&width=90pc)

2. In the **IAM** interface


   - Select **Roles**
   - Select **Create role**

![alt text](<Blank diagram - Page 3 (2).png>)

3. In the **Select trusted entity** interface

   - Select **AWS service**
   - **Use case**, select **EC2**
   - Select **Next**

![alt text](image-1.png)

4. In the **Create role** interface

   - Find the policy **AdministratorAccess**
   - Select the policy **AdministratorAccess**
   - Select **Next**

![alt text](image-2.png)

5. In the **Role details** interface

   - **Role name**, enter `CDK-Role`

![alt text](image-3.png)

6. Select **Create role**

![alt text](image-4.png)

7. Complete role creation

![alt text](image-5.png)