# Laravel on ECS Demo

## Build and run with docker-compose (local)

```bash
$ docker-compose up --build
```

### Build assets

```bash
$ docker-compose run --rm nodejs bash -c "npm install && npm run dev"
```

## Build and push docker to AWS ECR (Elastic Container Registry)

### Create ECR repository
Create 2 repositories for `nginx` and `php-fpm`.
Like below.

- `laravel-ecs-demo/nginx`
- `laravel-ecs-demo/php-fpm`

### docker login (with latest `awscli`)

```bash
$ pip install awscli --upgrade
$ $(aws ecr get-login --no-include-email --region ${AWS_DEFAULT_REGION})
```

### Build nodejs and assets(js/css)

```bash
$ docker build -t nodejs -f docker/nodejs/Dockerfile .
$ docker run --rm -v $(pwd):/build -w /build nodejs npm install
$ docker run --rm -v $(pwd):/build -w /build nodejs npm run dev
```

### Build php-fpm/nginx

```bash
$ docker build -t ${REPOSITORY_URI_PHP_FPM}:latest -f docker/php-fpm/Dockerfile .
$ docker build -t ${REPOSITORY_URI_NGINX}:latest -f docker/nginx/Dockerfile .
```

### Add tags

```bash
$ docker tag ${REPOSITORY_URI_PHP_FPM}:latest ${REPOSITORY_URI_PHP_FPM}:$IMAGE_TAG
$ docker tag ${REPOSITORY_URI_NGINX}:latest ${REPOSITORY_URI_NGINX}:$IMAGE_TAG
```

### Push docker image

```bash
$ docker push ${REPOSITORY_URI_PHP_FPM}:latest
$ docker push ${REPOSITORY_URI_PHP_FPM}:$IMAGE_TAG
$ docker push ${REPOSITORY_URI_NGINX}:latest
$ docker push ${REPOSITORY_URI_NGINX}:$IMAGE_TAG
```

## Create Task definition

| name | laravel-ecs-demo |
| Launch type | EC2 |

### Add php-fpm container

| name | php-fpm |
| image (uri) | ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/php-fpm:latest | 
| working directory | /app | 
| memory | 300 | 

### Add nginx container

| name | nginx |
| image (uri) | ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/nginx:latest | 
| port mapping | tcp 80:80 | 
| memory | 300 | 
| links | php-fpm | 

## Create ECS Cluster

| name | laravel-ecs-demo-cluster |
| cluster compatibility | EC2 Linux + Networking | 
| EC2 Instance type | t2.small |
| Provisioning model | Spot |
| Number of instances | 2 |
| Maximum bid price (per instance/hour) | 10($) |

## Create ALB (Application load balancer)

| Availability Zone | (Choose from VPC of ECS Cluster) |
| Listeners | HTTP 80 |
| Target instances | (Choose from Availability Zone) |

## Create Service in ECS Cluster

| Launch type | EC2 |
| Task definition | laravel-ecs-demo |
| Cluster | laravel-ecs-demo-cluster |
| Service name | laravel-ecs-demo |
| Service type | REPLICA |
| Load balancer type | `Application Load Balancer` |
| Container to load balance | nginx:80:80 |
| Listener port | 80:HTTP |
| Target Group | (Choose of ALB) |
