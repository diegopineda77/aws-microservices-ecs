# aws-microservices-ecs
The project was developed to demonstrate architectural design, the process of implementing Infrastructure as Code (IaC) in the AWS cloud, and the containerized deployment of microservices.

<img src="./Project 1 - Containers ECS Fargate EC2.png" alt="Diagrama de Arquitectura" width="800" />

This project includes:

- Detailed Project Documentation: Project 1 - Containers ECS Fargate EC2 - en/sp.pdf

- ALB Distribution Diagram: Load balancer EC2 _ us-east-1.pdf

- Project Architecture Diagram: Project 1 - Containers ECS Fargate EC2.png

- ECS Task Definition Files: /ecs-taskdef

- Containerized Application: /docker - CatsAndDogs

- CloudFormation Templates: /cloudformation


### Step-by-Step Deployment Details

#### 1. Network Infrastructure Setup
Deploy the base networking architecture using the `vpc.yaml` CloudFormation template (VPC, Internet & NAT Gateways, Public/Private Subnets, Security Groups, and Route Tables):
```bash
aws cloudformation create-stack \
  --stack-name vpc-stack \
  --template-body file://cloudformation/vpc.yaml
```

#### 2. ECR Repositories & Image Upload
Create ECR repositories for cats, dogs, and web, then build and push the Docker images:
```
Bash
docker build -t cats ./docker-CatsAndDogs/cats
docker tag cats:latest <AWS_ACCOUNT_ID>.dkr.ecr.<REGION>[.amazonaws.com/cats:latest](https://.amazonaws.com/cats:latest)
docker push <AWS_ACCOUNT_ID>.dkr.ecr.<REGION>[.amazonaws.com/cats:latest](https://.amazonaws.com/cats:latest)
```

#### 3. ECS Cluster Creation
Create the ECS Cluster and EC2 launch infrastructure using the templates:

cloudformation/ecs-cluster-proj1.yml

cloudformation/ec2-launch-template.yml
```
Bash
aws cloudformation create-stack \
  --stack-name ecs-cluster-stack \
  --template-body file://cloudformation/ecs-cluster-proj1.yml
```

#### 4. Task Definitions Setup
Register the task definitions (webdef, catsdef, and dogdef):

catsdef & webdef: Configured to run on EC2 Launch Type.

dogdef: Configured to run on AWS Fargate using awsvpc network mode for internal communication.
```
Bash
aws ecs register-task-definition --cli-input-json file://ecs-taskdef/catsdef.json
aws ecs register-task-definition --cli-input-json file://ecs-taskdef/dogdef.json
aws ecs register-task-definition --cli-input-json file://ecs-taskdef/webdef.json
```

#### 5. Application Load Balancer (ALB)
An ALB is deployed in the public subnets to route incoming internet traffic directly to the private ECS tasks.

#### 6. ECS Services Deployment
Deploy the individual microservices using their respective CloudFormation templates:
```
Bash
# Deploy Cats Service
aws cloudformation create-stack --stack-name cats-service --template-body file://cloudformation/ecs-service-cats.yaml

# Deploy Dogs Service
aws cloudformation create-stack --stack-name dogs-service --template-body file://cloudformation/ecs-service-dogs.yaml

# Deploy Web Service
aws cloudformation create-stack --stack-name web-service --template-body file://cloudformation/ecs-service-web.yaml
```
