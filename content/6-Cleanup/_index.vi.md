---
title : "Dọn dẹp tài nguyên"
date :  "`r Sys.Date()`" 
weight : 6
chapter : false
pre : " <b> 6. </b> "
---

#### Dọn dẹp tài nguyên

Tương tự như phần CDK cơ bản, bạn có thể dọn dẹp hết các tài nguyên đã tạo để tránh bị charge phí không đáng có. Để xoá các tài nguyên trong stack, chạy lệnh
```bash
cdk destroy
```

![CDK Advance](/images/6-cleanup/destroy-stack.png?featherlight=false&width=90pc)

Ngoài ra, với dịch vụ **S3 bucket**, nếu bạn đã upload file vào bucket, nó sẽ không thể bị xoá bởi lệnh `cdk destroy`, bạn sẽ cần xoá hết toàn bộ các file có trong bucket, và sau đó xoá bucket.

Chúc mừng bạn đã hoàn thành bài lab CDK nâng cao!