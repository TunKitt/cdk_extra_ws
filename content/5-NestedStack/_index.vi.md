---
title : "Nested Stack"
date :  "`r Sys.Date()`" 
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

#### Nested Stack

Việc triển khai một kiến trúc lớn trong 1 file là không hợp lý. Bởi CDK hoạt động dựa trên CloudFormation, chúng ta hoàn toàn có thể vận dụng nested stack để chia nhỏ stack tổng thể ra làm nhiều stack nhỏ khác nhau, giúp thuận lợi cho việc quản lý các tài nguyên.

Trong phần cuối cùng của bài lab CDK nâng cao này, chúng ta sẽ chia nhỏ template được tạo ra ở phần trước thành các template nhỏ hơn với trách nhiệm riêng biệt. Cụ thể, chúng ta sẽ chia thành 3 template:

1. ECS và ALB
2. Lambda và S3
3. API Gateway

![CDK Advance](/images/1-cdkadvanceintro/CDKAdvanceArch.jpg)
