## Architecture
<img width="1024" height="559" alt="image" src="https://github.com/user-attachments/assets/8a18adff-c194-4db1-8cf6-e6a4160e5425" />

## Trade-off decision made:
- Fargate for REST API instead of as 29 second timeout incompatible with inference workloads.
- No VM management overhead compared to if the api endpoint were deployed on Ec2.

## What works

- VPC, subnets(each with required route table configs), Internet & Nat Gateways, security groups, instances and the container deployed via CloudFormation in ap-south-1.
- Private subnet isolation - workers unreachable from the internet.
- Proper security group chaining done to maintain RPC wiring.
- Fargate task in public subnet with port 80 exposed.

## Instructions to deploy:
- Use the cli command below to deploy the IaC template on aws:

  aws cloudformation deploy \
  --template-file ml_infra.yaml \
  --stack-name alchemyst-inference-stack \
  --region ap-south-1 \
  --capabilities CAPABILITY_IAM
- Use this to tear the infrastructure down:

  aws cloudformation delete-stack \
  --stack-name alchemyst-inference-stack \
  --region ap-south-1
  
## Production hardening I would add

- HTTPS on the REST API endpoint with ACM certificate
- IAM roles for EC2 instances with least privilege and no hardcoded credentials.

## If the model were 100x larger
- I'd maintain distributed request, container, and worker state in Amazon DynamoDB while scaling containers and worker EC2s in production as in a scaled container environment, load balancers may route different requests to different container instances while worker registrations and RPC sessions remain attached to specific tasks which can create mismatches where the runtime believes a worker session exists on one container while traffic or lifecycle events shift elsewhere.
- EC2 instances would need GPU so i'd go with the p3 or g4dn family.
- RPC layer would need an sqs queue to handle burst without dropping requests.

## What's incomplete and why.

- Worker application code not fully deployed on EC2 instances because of negligible cicd hands on experience.
- Spent most time on understanding the code and architectural reasoning.
  
  
     
