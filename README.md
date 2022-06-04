# hello-nginx 

docker build -t yourgithubuser/hello-nginx:v1 .
docker push yourgithubuser/hello-nginx:v1

## Fargate Deployments

Note for the Fargate deploy you need a cluster and service to exist first.  I first created an ecs task execution role as defined in [AWS Documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_execution_IAM_role.html).  Then I did something like this: 

```
aws ecs register-task-definition --region us-east-1 --cli-input-json file://task-def.json --profile main

aws ecs create-cluster --region us-east-1 --cluster-name hello-nginx --profile main

aws ecs create-service --region us-east-1 --cluster hello-nginx --service-name hello-nginx --task-definition hello-nginx:1 --desired-count 1 --launch-type "FARGATE" --network-configuration "awsvpcConfiguration={subnets=[subnet-0c8babcd3b3f91d60, subnet-04472e65327d2d3fe],securityGroups=[sg-0566cffd1bd7b9f75],assignPublicIp=ENABLED}" --profile main
```

Note you may want to use a load balancer and put these containers in a private subnet.  I didn't for this example.

