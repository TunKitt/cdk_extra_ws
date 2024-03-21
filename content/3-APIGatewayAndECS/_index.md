---
title : "ECS, ALB and API Gateway"
date :  "`r Sys.Date()`" 
weight : 3
chapter : false
pre : " <b> 3. </b> "
---
 
#### Create ECS Cluster and Application Load Balancer

1. Access the Cloud9 Workspace
![alt text](image-1.png)

2. Create a new directory named **```cdk-workshop-02```**
![alt text](image-2.png)

3. Create CDK project
```bash
cd ~/environment/cdk-workshop-02
cdk init --language python
```

4. Import libraries in file `cdk-workshop-02/cdk_workshop_02_stack.py`
```python
from aws_cdk import (
    Stack,
    aws_ecs as ecs,
    aws_ecs_patterns as ecsp,
    aws_apigateway as apigateway
)
from constructs import Construct
```

5. In `cdk-workshop-02/cdk_workshop_02_stack.py`, declare a ECS cluster together with Application Load Balancer in the `__init__()` function
```python
      # Declare a Load Balancer Fargate 
      lb_fargate_service = ecsp.ApplicationLoadBalancedFargateService(
				self, 
				"Workshop02-MyWebServer",
            task_image_options=ecsp.ApplicationLoadBalancedTaskImageOptions(
               image=ecs.ContainerImage.from_registry("nginxdemos/hello")
            ),
            public_load_balancer=True,
            desired_count=3
      )
```

`ecsp.ApplicationLoadBalancedFargateService` will create the following resources:

- A new VPC with default configuration (subnet, Internet Gateway, RouteTable, EIP, Nat Gateway, …)
- A ECS cluster in the newly created VPC, running on Fargate
- An ECS Task defined by the `nginxdemos/hello` image from Dockerhub
- A ECS container group initiated from the ECS Task, with the number of instances is 3
- A Application Load Balancer fronting the ECS cluster
- Needed roles and security groups

#### Configure API Gateway

6. Declare an API Gateway
```python
# Define API Gateway
api = apigateway.RestApi(self, "ProxyToLBECS")
```

7. Configure API Gateway resource to point to the DNS of the Application Load Balancer
```python
# Get the DNS value of the Application Load Balancer 
lb = lb_fargate_service.load_balancer
lb_dns = lb_fargate_service.load_balancer.load_balancer_dns_name

# Add resource and method for proxy request
proxy = api.root.add_resource("ecs")
proxy.add_method("GET", apigateway.HttpIntegration(f"http://{lb_dns}"))
```

You can read more about the methods and properties of ApplicationLoadBalancedFargateService at [the official CDK documentation of AWS (Python)](https://docs.aws.amazon.com/cdk/api/v2/python/aws_cdk.aws_ecs_patterns/ApplicationLoadBalancedFargateService.html)

8. Check the code in `cdk-workshop-02/cdk_workshop_02_stack.py`
```python
from aws_cdk import (
    Stack,
    aws_ecs as ecs,
    aws_ecs_patterns as ecsp,
    aws_apigateway as apigateway
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
```

#### Deploy

9. Verify the resources that would be created by running
```bash
pip install aws-cdk.aws-lambda-python-alpha
cdk diff
```

10. Deploy the stack
```
cdk bootstrap
cdk deploy
```

![CDK Advance](/images/3-apigatewayandecs/deploy-stack.png?featherlight=false&width=90pc)

Choose `y` to continue

11. Finish
![alt text](<Blank diagram - Page 2 (2)-1.png>)

Access the link to the API Gateway (`ProxyToLBECSEndpoint`), add the path `/ecs`. You will be able to access the webserver deployed in ECS.

To verify that the ALB works properly, hit refresh a couple of times, and you will see the target of the website change. In the previous section, we set the parameter `desired_count=3`, so there will be 3 containers being deployed in parallel behind the ALB. You will notice the change in the field `Server address` between the refreshes.

![CDK Advance](/images/3-apigatewayandecs/nginx-result.png?featherlight=false&width=90pc)

If you have reached this state, congratulations on finishing the first part of the CDK Advance Workshop. In the next section, we will use CDK to deploy a Lambda function behind an API Gateway resource.