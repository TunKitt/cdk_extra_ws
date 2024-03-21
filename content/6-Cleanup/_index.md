---
title : "Clean up resources"
date : "`r Sys.Date()`"
weight : 6
chapter : false
pre : " <b> 6. </b> "
---

#### Clean up resources

Similar to the CDK Basic Workshop, you can clean up all the resources in order to avoid unwanted charges. To delete all the resources in the stack, run the command
```bash
cdk destroy
```

![CDK Advance](/images/6-cleanup/destroy-stack.png?featherlight=false&width=90pc)

If you have uploaded files into the bucket, it cannot be destroyed by the `cdk destroy` command. You will need to manually delete all the files inside the bucket, and then delete the bucket.

Congratulations on finishing the CDK Advance Workshop!