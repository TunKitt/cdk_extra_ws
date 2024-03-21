---
title : "Tạo nested stack với CDK"
date :  "`r Sys.Date()`" 
weight : 1
chapter : false
pre : " <b> 5.1 </b> "
---
 
#### Tạo nested stack với CDK

1. Trước hết, hãy tạo file `cdk_workshop_02/main_stack.py` để chứa root stack
```bash
touch cdk_workshop_02/main_stack.py
```

2. Khởi tạo nested stack cho ECS Fargate + Load balancer với file `cdk_workshop_02/lb_fargate_stack.py`
```python
from aws_cdk import (
    NestedStack,
    aws_ecs as ecs,
    aws_ecs_patterns as ecsp,
)
from constructs import Construct

class LBFargateStack(NestedStack):

    def __init__(self, scope: Construct, **kwargs) -> None:
        super().__init__(scope, "LB Fargate Stack", **kwargs)

        # Declare a Load Balancer Fargate 
        lb_fargate_service = ecsp.ApplicationLoadBalancedFargateService(
			self, 
		    "MyWebServer",
            task_image_options=ecsp.ApplicationLoadBalancedTaskImageOptions(
                image=ecs.ContainerImage.from_registry("nginxdemos/hello")),
            public_load_balancer=True,
            desired_count=3
        )
        
        self.lb = lb_fargate_service.load_balancer
```

3. Khởi tạo nested stack cho Lambda với file `cdk_workshop_02/lambda_stack.py`
```python
from aws_cdk import (
    NestedStack,
    aws_s3 as s3,
    aws_lambda as lambda_
)
from constructs import Construct

class LambdaStack(NestedStack):

    def __init__(self, scope: Construct, **kwargs) -> None:
        super().__init__(scope, "Lambda Stack", **kwargs)

        # Add S3 bucket
        bucket = s3.Bucket(self, "WidgetStore")
        
        # Add Lambda function
        handler = lambda_.Function(self, "WidgetHandler",
                    runtime=lambda_.Runtime.NODEJS_20_X,
                    code=lambda_.Code.from_asset("resources"),
                    handler="widget.main",
                    environment=dict(
                    BUCKET=bucket.bucket_name)
                    )
        
        # Grant bucket permission to lambda function
        bucket.grant_read_write(handler)
        
        self.handler = handler
```

4. Khởi tạo nested stack cho API Gateway với file `cdk_workshop_02/api_gateway_stack.py`
```python
from aws_cdk import (
    NestedStack,
    aws_apigateway as apigateway,
    aws_ecs_patterns as ecsp,
    aws_lambda as lambda_
)
from constructs import Construct

class APIGatewayStack(NestedStack):

    def __init__(self, scope: Construct, lb_dns: str, lambda_handler: lambda_.Function,  **kwargs) -> None:
        super().__init__(scope, "API Gateway Stack", **kwargs)

        # Define API Gateway
        api = apigateway.RestApi(self, "ProxyToLBECS")

        # Add resource and method for ECS proxy request
        ecs_proxy = api.root.add_resource("ecs")
        ecs_proxy.add_method("GET", apigateway.HttpIntegration(f"http://{lb_dns}"))

        # Create Lambda intergration
        get_widgets_integration = apigateway.LambdaIntegration(lambda_handler,
                request_templates={"application/json": '{ "statusCode": "200" }'})
        
        # Add resource and method for Lambda proxy request
        lambda_proxy = api.root.add_resource("lambda")
        lambda_proxy.add_method("GET", get_widgets_integration)
        
        self.url = api.url
```

5. Lắp ráp lại vào file template gốc để tạo root stack với file `cdk_workshop_02/main_stack.py`
```python
from aws_cdk import (
    Stack,
    aws_apigateway as apigateway,
    aws_s3 as s3,
    aws_lambda as lambda_,
    CfnOutput
)
from constructs import Construct
from .lambda_stack import LambdaStack
from .lb_fargate_stack import LBFargateStack
from .api_gateway_stack import APIGatewayStack

class MainStack(Stack):

    def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)
        
        lb_fargate_stack = LBFargateStack(self)
        
        lambda_stack = LambdaStack(self)
        
        # Get the DNS value of the Application Load Balancer 
        lb_dns = lb_fargate_stack.lb.load_balancer_dns_name
        
        api_gateway_stack = APIGatewayStack(self, lb_dns, lambda_stack.handler)
        
        # Define stack output
        CfnOutput(self, "API Gateway URL", value=api_gateway_stack.url)
```

6. Thay đổi code trong file `app.py` để sử dụng template chúng ta mới tạo
```python
#!/usr/bin/env python3
import os

import aws_cdk as cdk

from cdk_workshop_02.main_stack import MainStack


app = cdk.App()
# CdkWorkshop02Stack(app, "CdkWorkshop02Stack")
MainStack(app, "MainStack")

app.synth()
```

7. Triển khai
```bash
cdk bootstrap
cdk deploy
```

![alt text](<Blank diagram - Page 2 (4).png>)

Sau khi triển khai xong, bạn có thể thấy các stack đã được tạo trong AWS CloudFormation Console

![CDK Advance](/images/5-nestedstack/cf-console.png)

Bạn có thể thử API gateway URL cho Lambda và ECS như ở phần trước.

![CDK Advance](/images/5-nestedstack/lb-result.png)
![CDK Advance](/images/5-nestedstack/lambda-result.png)

**Lưu ý**: Việc thay đổi code trong file `app.py` tương ứng với việc triển khai một CloudFormation template mới. Trong bài lab nâng cao này, mỗi CloudFormation stack cần 2 elastic IP để hoạt động. Khi triển khai một stack, nếu số lượng elastic IP vượt quá giới hạn của account (mặc định là 5), CloudFormation stack có thể không khởi tạo được và bị kẹt ở trạng thái `IN_PROGRESS`. Hãy đảm bảo bạn release hết các elastic IP không cần thiết khi làm bài lab này.
