# Static Web App Deployment on AWS with Docker

This project demonstrates how to deploy a containerized static web application on AWS using Docker, Amazon ECR, and Amazon ECS. The deployment includes a secure three-tier VPC architecture, HTTPS configuration, and domain management.

## Architecture Overview

The application is deployed in a three-tier VPC architecture with public and private subnets across multiple availability zones. It uses Amazon ECS for container orchestration, Application Load Balancer for traffic distribution, and Route 53 for DNS management.

## Prerequisites

- AWS Account with appropriate permissions
- GitHub Account
- Docker installed locally
- AWS CLI installed and configured
- Domain name (for Route 53 configuration)
- Git installed locally

## Setup Instructions

### 1. GitHub Configuration
```bash
# Generate SSH key pair
ssh-keygen -t rsa -b 4096 -C "your-email@example.com"

# Add public key to GitHub
cat ~/.ssh/id_rsa.pub
# Copy the output and add it to GitHub SSH keys in settings
```

### 2. Repository Setup
```bash
# Clone the repository
git clone git@github.com:yourusername/your-repo.git
cd your-repo

# Create Dockerfile
touch Dockerfile
```

### 3. Docker Configuration
```bash
# Build container image
docker build -t your-app-name .

# Test locally
docker run -p 80:80 your-app-name

# Tag image for Docker Hub
docker tag your-app-name:latest yourusername/your-app-name:latest

# Push to Docker Hub
docker push yourusername/your-app-name:latest
```

### 4. AWS ECR Setup
```bash
# Create ECR repository
aws ecr create-repository --repository-name your-app-name

# Login to ECR
aws ecr get-login-password --region your-region | docker login --username AWS --password-stdin your-account-id.dkr.ecr.your-region.amazonaws.com

# Tag image for ECR
docker tag your-app-name:latest your-account-id.dkr.ecr.your-region.amazonaws.com/your-app-name:latest

# Push to ECR
docker push your-account-id.dkr.ecr.your-region.amazonaws.com/your-app-name:latest
```

### 5. AWS Infrastructure Setup

#### VPC and Networking
1. Create VPC with public and private subnets
2. Configure NAT Gateway for private subnet internet access
3. Set up Internet Gateway for public subnets
4. Configure route tables for both public and private subnets

#### Security Groups
1. Create ALB security group (allow HTTP/HTTPS)
2. Create ECS security group (allow inbound from ALB)
3. Configure other necessary security groups for your architecture

#### Load Balancer and SSL
1. Request SSL certificate through AWS Certificate Manager
2. Create Application Load Balancer
3. Configure HTTPS listener with SSL certificate
4. Set up target groups

#### ECS Configuration
1. Create ECS cluster
2. Define task definition with container specifications
3. Create ECS service with desired task count

#### Domain Configuration
1. Register domain in Route 53 (if not already registered)
2. Create hosted zone
3. Create A record pointing to ALB
4. Validate SSL certificate through DNS validation

## Security Considerations

- All containers run in private subnets
- Traffic flows through ALB with SSL termination
- Security groups limit access to necessary ports only
- NAT Gateway provides secure outbound internet access for private subnets

## Monitoring and Maintenance

- Use CloudWatch for container and application monitoring
- Set up alarms for critical metrics
- Regularly update container images and task definitions
- Monitor SSL certificate expiration

## Cost Considerations

- NAT Gateway incurs hourly charges
- ALB pricing based on hours and processed bytes
- ECS pricing depends on EC2 instance types or Fargate configuration
- Route 53 charges for hosted zones and queries

## Troubleshooting

1. Check ECS service events for deployment issues
2. Verify security group configurations
3. Validate SSL certificate status
4. Check container logs in CloudWatch
5. Verify target group health checks

## Contributing

Please read CONTRIBUTING.md for details on our code of conduct and the process for submitting pull requests.

## License

This project is licensed under the MIT License - see the LICENSE.md file for details