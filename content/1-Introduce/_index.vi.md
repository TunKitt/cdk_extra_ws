---
title : "Giới thiệu"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
pre : " <b> 1. </b> "
---

#### Tổng quan 

Trước khi làm workshop về CDK này, bạn nên đi qua workshop về [CDK cơ bản](https://000038.awsstudygroup.com/) để hiểu các khái niệm cơ bản của CDK. Bạn cũng nên có một vài kiến thức cơ bản về API Gateway, Elastic Load Balancer, Elastic Container Service và Lambda.

Chúng ta sẽ tái sử dụng workspace đã tạo trong phần CDK cơ bản để thực hành bài lab này. `CDK 2.27.0` sẽ được sử dụng. Sau khi hoàn thành bài lab này, bạn sẽ biết cách

- Sử dụng CDK để khởi tạo một kiến trúc ứng dụng cơ bản sử dụng API Gateway, Elastic Load Balancer, Elastic Container Service, và Lambda.
- Tạo nested stack sử dụng CDK

Kiến trúc tổng thể của hệ thống mà chúng ta sẽ xây dựng như sau:


![CDK Advance](/images/1-cdkadvanceintro/CDKAdvanceArch.jpg)