---
title : "Lambda và S3"
date :  "`r Sys.Date()`" 
weight : 4
chapter : false
pre : " <b> 4. </b> "
---
 
#### Lambda và S3
Ở phần này, chúng ta sẽ cùng nhau cấu hình một hàm Lambda và kết hợp với API Gateway. Ta sẽ tái sử dụng template ở phần trước (API Gateway và ECS)

1. Tạo thư mục `resources` ở thư mục gốc của project
```bash
mkdir resources
```

2. Tạo file `widget.js` trong thư mục `resources`
```javascript
/* 
This code uses callbacks to handle asynchronous function responses.
It currently demonstrates using an async-await pattern. 
AWS supports both the async-await and promises patterns.
For more information, see the following: 
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises
https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/calling-services-asynchronously.html
https://docs.aws.amazon.com/lambda/latest/dg/nodejs-prog-model-handler.html 
*/
const AWS = require('aws-sdk');
const S3 = new AWS.S3();

const bucketName = process.env.BUCKET;

exports.main = async function(event, context) {
  try {
    var method = event.httpMethod;
    
    const data = await S3.listObjectsV2({ Bucket: bucketName }).promise();
        var body = {
          widgets: data.Contents.map(function(e) { return e.Key })
        };
        return {
          statusCode: 200,
          headers: {},
          body: JSON.stringify(body)
        };
  } catch(error) {
    var body = error.stack || JSON.stringify(error, null, 2);
    return {
      statusCode: 400,
        headers: {},
        body: JSON.stringify(body)
    }
  }
}
```

Hàm lambda này sẽ trả về danh sách các object có trong S3 bucket.


3. Trong `cdk_workshop_02/cdk_workshop_02_stack.py` khởi tạo S3 bucket
```python
# Add S3 bucket
bucket = s3.Bucket(self, "WidgetStore")
```

4. Tạo lambda function và gán quyền tương ứng với S3 bucket vừa tạo
```python
# Add Lambda function
handler = lambda_.Function(self, "WidgetHandler",
		runtime=lambda_.Runtime.NODEJS_14_X,
    code=lambda_.Code.from_asset("resources"),
    handler="widget.main",
    environment=dict(BUCKET=bucket.bucket_name)
)
        
# Grant bucket permission to lambda function
bucket.grant_read_write(handler)
```

5. Tạo intergration cho API Gateway 
```python
# Create intergration
get_widgets_integration = apigateway.LambdaIntegration(
	handler,
	request_templates={"application/json": '{ "statusCode": "200" }'}
)
```

6. Thêm resource và method tương ứng cho API Gateway
```python
# Add resource and method for proxy request
lambda_proxy = api.root.add_resource("lambda")
lambda_proxy.add_method("GET", get_widgets_integration)
```

7. Đừng quên import thêm các thư viện cần thiết để sử dụng được Lambda, API Gateway và S3. 

Sau khi import xong, hãy kiểm tra lại nội dung file `cdk_workshop_02/cdk_workshop_02_stack.py` của bạn

```python
from aws_cdk import (
    Stack,
    aws_ecs as ecs,
    aws_ecs_patterns as ecsp,
    aws_apigateway as apigateway,
    aws_s3 as s3,
    aws_lambda as lambda_
)
from constructs import Construct

class CdkWorkshop02Stack(Stack):

    def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)
        
        # Declare a Load Balancer Fargate 
        lb_fargate_service = ecsp.ApplicationLoadBalancedFargateService(
			self, 
		    "Workshop02-MyWebServer",
            task_image_options=ecsp.ApplicationLoadBalancedTaskImageOptions(
                image=ecs.ContainerImage.from_registry("nginxdemos/hello")),
            public_load_balancer=True,
            desired_count=3
        )
        
        # Define API Gateway
        api = apigateway.RestApi(self, "ProxyToLBECS")
        
        # Get the DNS value of the Application Load Balancer 
        lb = lb_fargate_service.load_balancer
        lb_dns = lb_fargate_service.load_balancer.load_balancer_dns_name
        
        # Add resource and method for proxy request
        proxy = api.root.add_resource("ecs")
        proxy.add_method("GET", apigateway.HttpIntegration(f"http://{lb_dns}"))
        
        # Add S3 bucket
        bucket = s3.Bucket(self, "WidgetStore")
        
        # Add Lambda function
        handler = lambda_.Function(self, "WidgetHandler",
        		runtime=lambda_.Runtime.NODEJS_20_X,
            code=lambda_.Code.from_asset("resources"),
            handler="widget.main",
            environment=dict(BUCKET=bucket.bucket_name)
        )
                
        # Grant bucket permission to lambda function
        bucket.grant_read_write(handler)
        
        # Create intergration
        get_widgets_integration = apigateway.LambdaIntegration(
        	handler,
        	request_templates={"application/json": '{ "statusCode": "200" }'}
        )
        
        # Add resource and method for proxy request
        lambda_proxy = api.root.add_resource("lambda")
        lambda_proxy.add_method("GET", get_widgets_integration)
```

8. Triển khai stack
```bash
cdk deploy
```
![alt text](<Blank diagram - Page 2 (3).png>)

9. Hoàn thành

Truy cập vào đường link của API gateway, thêm path `/lambda`. Bạn sẽ thấy kết quả trả về từ hàm lambda.

Để kiểm tra, hãy truy cập vào S3 Console. Truy cập vào bucket được tạo và upload một file bất kỳ

![CDK Advance](/images/4-lambdaands3/s3-1.png?featherlight=false&width=90pc)
![CDK Advance](/images/4-lambdaands3/s3-2.png?featherlight=false&width=90pc)


Truy cập lại vào đường dẫn đến hàm lambda `xxxxxxx.execute-api.[REGION].amazonaws.com/prod/lambda` của API Gateway. Bạn sẽ thấy tên file mới được hiển thị.
![CDK Advance](/images/4-lambdaands3/lambda-result.png?featherlight=false&width=90pc)

Nếu bạn đã chạy được đến phần này, chúc mừng bạn đã hoàn thành phần thứ hai của bài lab CDK nâng cao. Ở phần tiếp theo, chúng ta sẽ refractor lại project bằng cách tách nhỏ các thành phần trong file `cdk_workshop_02/cdk_workshop_02_stack.py` thành các file khác nhau, và sử dụng *nested stack* để triển khai kiến trúc hệ thống của chúng ta.