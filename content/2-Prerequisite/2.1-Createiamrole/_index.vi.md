---
title : "Tạo IAM Role"
date :  "`r Sys.Date()`" 
weight : 1
chapter : false
pre : " <b> 2.1 </b> "
---
 
#### Tạo IAM Role

1. Truy cập giao diện [AWS Management Console](https://aws.amazon.com/console/)

   - Tìm **IAM**
   - Chọn **IAM**

![Amazon CDk](/images/2.1-prerequisite/0001.png?featherlight=false&width=90pc)

2. Trong giao diện **IAM**

   - Chọn **Roles**
   - Chọn **Create role**

![alt text](<Blank diagram - Page 3 (2).png>)

3. Trong giao diện **Select trusted entity**

   - Chọn **AWS service**
   - **Use case**, chọn **EC2**
   - Chọn **Next**

![alt text](image-1.png)

4. Trong giao diện **Create role**

   - Tìm policy **AdministratorAccess**
   - Chọn policy **AdministratorAccess**
   - Chọn **Next**

![alt text](image-2.png)

5. Trong giao diện **Role details**

   - **Role name**, nhập `CDK-Role`

![alt text](image-3.png)

6. Chọn **Create role**

![alt text](image-4.png)

7. Hoàn thành tạo role

![alt text](image-5.png)

