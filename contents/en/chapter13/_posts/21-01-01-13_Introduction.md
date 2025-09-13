---
layout: post
title: 13 Infrastructure as Code (IaC)
chapter: '13'
order: 1
owner: Nguyen Le Linh
lang: en
categories:
- chapter13
---

Infrastructure as Code (IaC) is a key practice in modern cloud computing and DevOps that involves managing and provisioning computing infrastructure through machine-readable definition files, rather than physical hardware configuration or interactive configuration tools.

## Introduction to Infrastructure as Code

### What is Infrastructure as Code?
Infrastructure as Code (IaC) is the practice of managing infrastructure using code and software development techniques. Instead of manually configuring servers, networks, and other infrastructure components, IaC allows you to define your infrastructure in code files that can be version controlled, tested, and automatically deployed.

```
Traditional Infrastructure Management:
┌─────────────────────────────────────────────────────────────┐
│                Manual Configuration                          │
├─────────────────────────────────────────────────────────────┤
│ 1. Log into server console                                  │
│ 2. Manually install software                                │
│ 3. Configure settings through GUI                           │
│ 4. Repeat for each server                                   │
│ 5. Document changes manually                                │
└─────────────────────────────────────────────────────────────┘

Infrastructure as Code:
┌─────────────────────────────────────────────────────────────┐
│                 Code-Driven Infrastructure                   │
├─────────────────────────────────────────────────────────────┤
│ 1. Define infrastructure in code files                      │
│ 2. Version control infrastructure definitions               │
│ 3. Automated deployment and configuration                   │
│ 4. Consistent, repeatable deployments                       │
│ 5. Self-documenting infrastructure                          │
└─────────────────────────────────────────────────────────────┘
```

### Core Principles of IaC

#### 1. Declarative vs Imperative
```yaml
# Declarative approach (Terraform)
resource "aws_instance" "web_server" {
  ami           = "ami-0c55b159cbfafe1d0"
  instance_type = "t3.micro"
  
  tags = {
    Name = "WebServer"
    Environment = "Production"
  }
}

# What you want, not how to get it
```

```bash
# Imperative approach (Shell script)
#!/bin/bash
# How to create the infrastructure step by step

# Create security group
aws ec2 create-security-group --group-name web-sg --description "Web server security group"

# Add rules to security group
aws ec2 authorize-security-group-ingress --group-name web-sg --protocol tcp --port 80 --cidr 0.0.0.0/0

# Launch instance
aws ec2 run-instances --image-id ami-0c55b159cbfafe1d0 --count 1 --instance-type t3.micro --security-groups web-sg
```

#### 2. Idempotency
```python
# Idempotent infrastructure operations
class InfrastructureManager:
    
    def ensure_s3_bucket(self, bucket_name, region="us-east-1"):
        """Idempotent S3 bucket creation"""
        
        try:
            # Check if bucket exists
            self.s3_client.head_bucket(Bucket=bucket_name)
            print(f"Bucket {bucket_name} already exists")
            return bucket_name
            
        except ClientError as e:
            error_code = int(e.response['Error']['Code'])
            
            if error_code == 404:
                # Bucket doesn't exist, create it
                if region == 'us-east-1':
                    self.s3_client.create_bucket(Bucket=bucket_name)
                else:
                    self.s3_client.create_bucket(
                        Bucket=bucket_name,
                        CreateBucketConfiguration={'LocationConstraint': region}
                    )
                print(f"Created bucket {bucket_name}")
                return bucket_name
            else:
                raise e
    
    def ensure_ec2_instance(self, instance_config):
        """Idempotent EC2 instance management"""
        
        # Check if instance with specific tags exists
        existing_instances = self.ec2_client.describe_instances(
            Filters=[
                {'Name': 'tag:Name', 'Values': [instance_config['name']]},
                {'Name': 'instance-state-name', 'Values': ['running', 'pending']}
            ]
        )
        
        if existing_instances['Reservations']:
            instance_id = existing_instances['Reservations'][0]['Instances'][0]['InstanceId']
            print(f"Instance {instance_id} already exists")
            return instance_id
        
        # Create new instance
        response = self.ec2_client.run_instances(
            ImageId=instance_config['ami'],
            MinCount=1,
            MaxCount=1,
            InstanceType=instance_config['instance_type'],
            TagSpecifications=[{
                'ResourceType': 'instance',
                'Tags': [{'Key': 'Name', 'Value': instance_config['name']}]
            }]
        )
        
        instance_id = response['Instances'][0]['InstanceId']
        print(f"Created instance {instance_id}")
        return instance_id
```

#### 3. Version Control and Collaboration
```
IaC Version Control Workflow:
┌─────────────────────────────────────────────────────────────┐
│                    Git Repository                            │
├─────────────────────────────────────────────────────────────┤
│ infrastructure/                                             │
│ ├── environments/                                           │
│ │   ├── dev/                                               │
│ │   │   ├── main.tf                                        │
│ │   │   ├── variables.tf                                   │
│ │   │   └── terraform.tfvars                               │
│ │   ├── staging/                                           │
│ │   └── production/                                        │
│ ├── modules/                                               │
│ │   ├── vpc/                                               │
│ │   ├── ec2/                                               │
│ │   └── rds/                                               │
│ └── scripts/                                               │
│     ├── deploy.sh                                          │
│     └── destroy.sh                                         │
└─────────────────────────────────────────────────────────────┘
```

## Popular IaC Tools and Platforms

### Terraform
Terraform is a popular open-source IaC tool that supports multiple cloud providers.

```hcl
# Terraform configuration example
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "infrastructure/terraform.tfstate"
    region = "us-east-1"
  }
}

provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = {
      Environment = var.environment
      Project     = var.project_name
      ManagedBy   = "Terraform"
    }
  }
}

# Variables
variable "aws_region" {
  description = "AWS region for resources"
  type        = string
  default     = "us-east-1"
}

variable "environment" {
  description = "Environment name"
  type        = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "instance_count" {
  description = "Number of EC2 instances"
  type        = number
  default     = 2
}

# Data sources
data "aws_availability_zones" "available" {
  state = "available"
}

data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

# VPC Configuration
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name = "${var.environment}-vpc"
  }
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name = "${var.environment}-igw"
  }
}

resource "aws_subnet" "public" {
  count = 2
  
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.${count.index + 1}.0/24"
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  map_public_ip_on_launch = true
  
  tags = {
    Name = "${var.environment}-public-subnet-${count.index + 1}"
    Type = "Public"
  }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
  
  tags = {
    Name = "${var.environment}-public-rt"
  }
}

resource "aws_route_table_association" "public" {
  count = length(aws_subnet.public)
  
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

# Security Group
resource "aws_security_group" "web" {
  name_prefix = "${var.environment}-web-"
  vpc_id      = aws_vpc.main.id
  
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/16"]  # Only from VPC
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "${var.environment}-web-sg"
  }
}

# Launch Template
resource "aws_launch_template" "web" {
  name_prefix   = "${var.environment}-web-"
  image_id      = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"
  
  vpc_security_group_ids = [aws_security_group.web.id]
  
  user_data = base64encode(templatefile("${path.module}/user_data.sh", {
    environment = var.environment
  }))
  
  tag_specifications {
    resource_type = "instance"
    tags = {
      Name = "${var.environment}-web-server"
    }
  }
}

# Auto Scaling Group
resource "aws_autoscaling_group" "web" {
  name                = "${var.environment}-web-asg"
  vpc_zone_identifier = aws_subnet.public[*].id
  target_group_arns   = [aws_lb_target_group.web.arn]
  health_check_type   = "ELB"
  
  min_size         = 1
  max_size         = 5
  desired_capacity = var.instance_count
  
  launch_template {
    id      = aws_launch_template.web.id
    version = "$Latest"
  }
  
  tag {
    key                 = "Name"
    value               = "${var.environment}-web-asg"
    propagate_at_launch = false
  }
}

# Application Load Balancer
resource "aws_lb" "web" {
  name               = "${var.environment}-web-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.web.id]
  subnets            = aws_subnet.public[*].id
  
  enable_deletion_protection = var.environment == "prod"
}

resource "aws_lb_target_group" "web" {
  name     = "${var.environment}-web-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id
  
  health_check {
    enabled             = true
    healthy_threshold   = 2
    interval            = 30
    matcher             = "200"
    path                = "/"
    port                = "traffic-port"
    protocol            = "HTTP"
    timeout             = 5
    unhealthy_threshold = 2
  }
}

resource "aws_lb_listener" "web" {
  load_balancer_arn = aws_lb.web.arn
  port              = "80"
  protocol          = "HTTP"
  
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.web.arn
  }
}

# Outputs
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

output "load_balancer_dns" {
  description = "DNS name of the load balancer"
  value       = aws_lb.web.dns_name
}

output "public_subnet_ids" {
  description = "IDs of the public subnets"
  value       = aws_subnet.public[*].id
}
```

### AWS CloudFormation
CloudFormation is AWS's native IaC service using JSON or YAML templates.

```yaml
# CloudFormation template example
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Web application infrastructure'

Parameters:
  Environment:
    Type: String
    Default: dev
    AllowedValues: [dev, staging, prod]
    Description: Environment name
  
  InstanceType:
    Type: String
    Default: t3.micro
    AllowedValues: [t3.micro, t3.small, t3.medium]
    Description: EC2 instance type
  
  KeyPairName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: EC2 Key Pair for SSH access

Mappings:
  EnvironmentMap:
    dev:
      InstanceCount: 1
      DBInstanceClass: db.t3.micro
    staging:
      InstanceCount: 2
      DBInstanceClass: db.t3.small
    prod:
      InstanceCount: 3
      DBInstanceClass: db.t3.medium

Resources:
  # VPC
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true
      EnableDnsSupport: true
      Tags:
        - Key: Name
          Value: !Sub '${Environment}-vpc'
  
  # Internet Gateway
  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: !Sub '${Environment}-igw'
  
  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway
  
  # Public Subnets
  PublicSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select [0, !GetAZs '']
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Sub '${Environment}-public-subnet-1'
  
  PublicSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.2.0/24
      AvailabilityZone: !Select [1, !GetAZs '']
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Sub '${Environment}-public-subnet-2'
  
  # Route Table
  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: !Sub '${Environment}-public-rt'
  
  PublicRoute:
    Type: AWS::EC2::Route
    DependsOn: AttachGateway
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway
  
  PublicSubnetRouteTableAssociation1:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet1
      RouteTableId: !Ref PublicRouteTable
  
  PublicSubnetRouteTableAssociation2:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet2
      RouteTableId: !Ref PublicRouteTable
  
  # Security Group
  WebSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Security group for web servers
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 10.0.0.0/16
      Tags:
        - Key: Name
          Value: !Sub '${Environment}-web-sg'
  
  # Launch Template
  LaunchTemplate:
    Type: AWS::EC2::LaunchTemplate
    Properties:
      LaunchTemplateName: !Sub '${Environment}-web-template'
      LaunchTemplateData:
        ImageId: ami-0c55b159cbfafe1d0  # Amazon Linux 2
        InstanceType: !Ref InstanceType
        KeyName: !Ref KeyPairName
        SecurityGroupIds:
          - !Ref WebSecurityGroup
        UserData:
          Fn::Base64: !Sub |
            #!/bin/bash
            yum update -y
            yum install -y httpd
            systemctl start httpd
            systemctl enable httpd
            echo "<h1>Hello from ${Environment} environment!</h1>" > /var/www/html/index.html
        TagSpecifications:
          - ResourceType: instance
            Tags:
              - Key: Name
                Value: !Sub '${Environment}-web-server'
  
  # Auto Scaling Group
  AutoScalingGroup:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      AutoScalingGroupName: !Sub '${Environment}-web-asg'
      VPCZoneIdentifier:
        - !Ref PublicSubnet1
        - !Ref PublicSubnet2
      LaunchTemplate:
        LaunchTemplateId: !Ref LaunchTemplate
        Version: !GetAtt LaunchTemplate.LatestVersionNumber
      MinSize: 1
      MaxSize: 5
      DesiredCapacity: !FindInMap [EnvironmentMap, !Ref Environment, InstanceCount]
      TargetGroupARNs:
        - !Ref TargetGroup
      HealthCheckType: ELB
      HealthCheckGracePeriod: 300
      Tags:
        - Key: Name
          Value: !Sub '${Environment}-web-asg'
          PropagateAtLaunch: false
  
  # Application Load Balancer
  LoadBalancer:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Name: !Sub '${Environment}-web-alb'
      Scheme: internet-facing
      Type: application
      SecurityGroups:
        - !Ref WebSecurityGroup
      Subnets:
        - !Ref PublicSubnet1
        - !Ref PublicSubnet2
  
  TargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Properties:
      Name: !Sub '${Environment}-web-tg'
      Port: 80
      Protocol: HTTP
      VpcId: !Ref VPC
      HealthCheckPath: /
      HealthCheckProtocol: HTTP
      HealthCheckIntervalSeconds: 30
      HealthyThresholdCount: 2
      UnhealthyThresholdCount: 5
  
  Listener:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      DefaultActions:
        - Type: forward
          TargetGroupArn: !Ref TargetGroup
      LoadBalancerArn: !Ref LoadBalancer
      Port: 80
      Protocol: HTTP

Outputs:
  VPCId:
    Description: VPC ID
    Value: !Ref VPC
    Export:
      Name: !Sub '${Environment}-VPC-ID'
  
  LoadBalancerDNS:
    Description: Load Balancer DNS Name
    Value: !GetAtt LoadBalancer.DNSName
    Export:
      Name: !Sub '${Environment}-LoadBalancer-DNS'
  
  WebsiteURL:
    Description: Website URL
    Value: !Sub 'http://${LoadBalancer.DNSName}'
```

### Ansible
Ansible is an agentless automation tool that can be used for IaC.

```yaml
# Ansible playbook example
---
- name: Deploy web application infrastructure
  hosts: localhost
  connection: local
  gather_facts: false
  
  vars:
    aws_region: us-east-1
    environment: "{{ env | default('dev') }}"
    project_name: web-app
    
  tasks:
    - name: Create VPC
      amazon.aws.ec2_vpc_net:
        name: "{{ environment }}-vpc"
        cidr_block: 10.0.0.0/16
        region: "{{ aws_region }}"
        state: present
        tags:
          Environment: "{{ environment }}"
          Project: "{{ project_name }}"
      register: vpc
    
    - name: Create Internet Gateway
      amazon.aws.ec2_vpc_igw:
        vpc_id: "{{ vpc.vpc.id }}"
        region: "{{ aws_region }}"
        state: present
        tags:
          Name: "{{ environment }}-igw"
      register: igw
    
    - name: Create public subnets
      amazon.aws.ec2_vpc_subnet:
        vpc_id: "{{ vpc.vpc.id }}"
        cidr: "{{ item.cidr }}"
        az: "{{ item.az }}"
        region: "{{ aws_region }}"
        state: present
        map_public: true
        tags:
          Name: "{{ item.name }}"
          Type: Public
      register: public_subnets
      loop:
        - { cidr: "10.0.1.0/24", az: "{{ aws_region }}a", name: "{{ environment }}-public-1" }
        - { cidr: "10.0.2.0/24", az: "{{ aws_region }}b", name: "{{ environment }}-public-2" }
    
    - name: Create route table for public subnets
      amazon.aws.ec2_vpc_route_table:
        vpc_id: "{{ vpc.vpc.id }}"
        region: "{{ aws_region }}"
        routes:
          - dest: 0.0.0.0/0
            gateway_id: "{{ igw.gateway_id }}"
        subnets:
          - "{{ public_subnets.results[0].subnet.id }}"
          - "{{ public_subnets.results[1].subnet.id }}"
        tags:
          Name: "{{ environment }}-public-rt"
    
    - name: Create security group
      amazon.aws.ec2_group:
        name: "{{ environment }}-web-sg"
        description: Security group for web servers
        vpc_id: "{{ vpc.vpc.id }}"
        region: "{{ aws_region }}"
        rules:
          - proto: tcp
            ports:
              - 80
            cidr_ip: 0.0.0.0/0
            rule_desc: HTTP
          - proto: tcp
            ports:
              - 443
            cidr_ip: 0.0.0.0/0
            rule_desc: HTTPS
          - proto: tcp
            ports:
              - 22
            cidr_ip: 10.0.0.0/16
            rule_desc: SSH from VPC
        tags:
          Name: "{{ environment }}-web-sg"
      register: security_group
    
    - name: Launch EC2 instances
      amazon.aws.ec2_instance:
        name: "{{ environment }}-web-{{ item }}"
        image_id: ami-0c55b159cbfafe1d0
        instance_type: t3.micro
        region: "{{ aws_region }}"
        vpc_subnet_id: "{{ public_subnets.results[item % 2].subnet.id }}"
        security_groups:
          - "{{ security_group.group_id }}"
        user_data: |
          #!/bin/bash
          yum update -y
          yum install -y httpd
          systemctl start httpd
          systemctl enable httpd
          echo "<h1>Hello from {{ environment }} server {{ item }}!</h1>" > /var/www/html/index.html
        state: present
        wait: true
        tags:
          Environment: "{{ environment }}"
          Project: "{{ project_name }}"
          Server: "web-{{ item }}"
      loop: "{{ range(1, (instance_count | default(2)) + 1) | list }}"
      register: instances
    
    - name: Create Application Load Balancer
      amazon.aws.elb_application_lb:
        name: "{{ environment }}-web-alb"
        region: "{{ aws_region }}"
        subnets:
          - "{{ public_subnets.results[0].subnet.id }}"
          - "{{ public_subnets.results[1].subnet.id }}"
        security_groups:
          - "{{ security_group.group_id }}"
        listeners:
          - Protocol: HTTP
            Port: 80
            DefaultActions:
              - Type: forward
                TargetGroupName: "{{ environment }}-web-tg"
        tags:
          Environment: "{{ environment }}"
          Project: "{{ project_name }}"
      register: alb
    
    - name: Create target group
      amazon.aws.elb_target_group:
        name: "{{ environment }}-web-tg"
        region: "{{ aws_region }}"
        protocol: HTTP
        port: 80
        vpc_id: "{{ vpc.vpc.id }}"
        health_check_path: /
        health_check_interval: 30
        healthy_threshold_count: 2
        unhealthy_threshold_count: 5
        targets:
          - Id: "{{ item.instance_id }}"
            Port: 80
        tags:
          Environment: "{{ environment }}"
          Project: "{{ project_name }}"
      loop: "{{ instances.results }}"
      when: item.instance_id is defined
    
    - name: Display load balancer DNS
      debug:
        msg: "Application is available at: http://{{ alb.dns_name }}"
```

## Advanced IaC Patterns and Best Practices

### Modular Infrastructure Design
```hcl
# Terraform modules structure
# modules/vpc/main.tf
variable "environment" {
  description = "Environment name"
  type        = string
}

variable "cidr_block" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
}

resource "aws_vpc" "main" {
  cidr_block           = var.cidr_block
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name        = "${var.environment}-vpc"
    Environment = var.environment
  }
}

resource "aws_subnet" "public" {
  count = length(var.availability_zones)
  
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.cidr_block, 8, count.index + 1)
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true
  
  tags = {
    Name        = "${var.environment}-public-${count.index + 1}"
    Environment = var.environment
    Type        = "Public"
  }
}

resource "aws_subnet" "private" {
  count = length(var.availability_zones)
  
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(var.cidr_block, 8, count.index + 101)
  availability_zone = var.availability_zones[count.index]
  
  tags = {
    Name        = "${var.environment}-private-${count.index + 1}"
    Environment = var.environment
    Type        = "Private"
  }
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name        = "${var.environment}-igw"
    Environment = var.environment
  }
}

# NAT Gateways
resource "aws_eip" "nat" {
  count = length(var.availability_zones)
  
  domain = "vpc"
  
  tags = {
    Name        = "${var.environment}-nat-eip-${count.index + 1}"
    Environment = var.environment
  }
}

resource "aws_nat_gateway" "main" {
  count = length(var.availability_zones)
  
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id
  
  tags = {
    Name        = "${var.environment}-nat-${count.index + 1}"
    Environment = var.environment
  }
  
  depends_on = [aws_internet_gateway.main]
}

# Route Tables
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
  
  tags = {
    Name        = "${var.environment}-public-rt"
    Environment = var.environment
  }
}

resource "aws_route_table" "private" {
  count = length(var.availability_zones)
  
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main[count.index].id
  }
  
  tags = {
    Name        = "${var.environment}-private-rt-${count.index + 1}"
    Environment = var.environment
  }
}

# Route Table Associations
resource "aws_route_table_association" "public" {
  count = length(aws_subnet.public)
  
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "private" {
  count = length(aws_subnet.private)
  
  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private[count.index].id
}

# Outputs
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

output "vpc_cidr_block" {
  description = "CIDR block of the VPC"
  value       = aws_vpc.main.cidr_block
}

output "public_subnet_ids" {
  description = "IDs of the public subnets"
  value       = aws_subnet.public[*].id
}

output "private_subnet_ids" {
  description = "IDs of the private subnets"
  value       = aws_subnet.private[*].id
}

output "internet_gateway_id" {
  description = "ID of the Internet Gateway"
  value       = aws_internet_gateway.main.id
}

output "nat_gateway_ids" {
  description = "IDs of the NAT Gateways"
  value       = aws_nat_gateway.main[*].id
}
```

### Using the VPC Module
```hcl
# environments/prod/main.tf
module "vpc" {
  source = "../../modules/vpc"
  
  environment        = "prod"
  cidr_block        = "10.0.0.0/16"
  availability_zones = ["us-east-1a", "us-east-1b", "us-east-1c"]
}

module "web_application" {
  source = "../../modules/web-app"
  
  environment         = "prod"
  vpc_id             = module.vpc.vpc_id
  public_subnet_ids  = module.vpc.public_subnet_ids
  private_subnet_ids = module.vpc.private_subnet_ids
  
  instance_type      = "t3.medium"
  min_size          = 2
  max_size          = 10
  desired_capacity  = 3
}

module "database" {
  source = "../../modules/rds"
  
  environment        = "prod"
  vpc_id            = module.vpc.vpc_id
  private_subnet_ids = module.vpc.private_subnet_ids
  
  db_instance_class = "db.r5.large"
  allocated_storage = 100
  backup_retention_period = 7
  multi_az = true
}
```

### State Management and Remote Backends
```hcl
# Remote state configuration
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "environments/prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
  }
}

# Data source to reference other state files
data "terraform_remote_state" "vpc" {
  backend = "s3"
  
  config = {
    bucket = "my-terraform-state-bucket"
    key    = "shared/vpc/terraform.tfstate"
    region = "us-east-1"
  }
}

# Use outputs from other state
resource "aws_instance" "app" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"
  subnet_id     = data.terraform_remote_state.vpc.outputs.private_subnet_ids[0]
  
  tags = {
    Name = "app-server"
  }
}
```

## CI/CD Integration with IaC

### GitLab CI/CD Pipeline
```yaml
# .gitlab-ci.yml
stages:
  - validate
  - plan
  - apply
  - destroy

variables:
  TF_ROOT: ${CI_PROJECT_DIR}/infrastructure
  TF_ADDRESS: ${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/terraform/state/${CI_ENVIRONMENT_NAME}

cache:
  key: "${TF_ROOT}"
  paths:
    - ${TF_ROOT}/.terraform

before_script:
  - cd ${TF_ROOT}
  - terraform --version
  - terraform init -backend-config="address=${TF_ADDRESS}" -backend-config="lock_address=${TF_ADDRESS}/lock" -backend-config="unlock_address=${TF_ADDRESS}/lock" -backend-config="username=${CI_USERNAME}" -backend-config="password=${CI_JOB_TOKEN}" -backend-config="lock_method=POST" -backend-config="unlock_method=DELETE" -backend-config="retry_wait_min=5"

validate:
  stage: validate
  script:
    - terraform validate
    - terraform fmt -check
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    - if: '$CI_COMMIT_BRANCH && $CI_OPEN_MERGE_REQUESTS'
      when: never
    - if: '$CI_COMMIT_BRANCH'

plan:
  stage: plan
  script:
    - terraform plan -var-file="environments/${CI_ENVIRONMENT_NAME}.tfvars" -out="planfile"
  artifacts:
    name: plan
    paths:
      - ${TF_ROOT}/planfile
    reports:
      terraform: ${TF_ROOT}/planfile.json
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    - if: '$CI_COMMIT_BRANCH && $CI_OPEN_MERGE_REQUESTS'
      when: never
    - if: '$CI_COMMIT_BRANCH'

apply:
  stage: apply
  script:
    - terraform apply -input=false "planfile"
  dependencies:
    - plan
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: manual
    - if: '$CI_COMMIT_BRANCH == "develop"'
      when: manual

destroy:
  stage: destroy
  script:
    - terraform destroy -var-file="environments/${CI_ENVIRONMENT_NAME}.tfvars" -auto-approve
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: manual
      allow_failure: false
  environment:
    name: production
    action: stop
```

### GitHub Actions Workflow
```yaml
# .github/workflows/terraform.yml
name: 'Terraform'

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

permissions:
  contents: read
  pull-requests: write

jobs:
  terraform:
    name: 'Terraform'
    runs-on: ubuntu-latest
    environment: production

    defaults:
      run:
        shell: bash
        working-directory: ./infrastructure

    steps:
    - name: Checkout
      uses: actions/checkout@v3

    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v2
      with:
        terraform_version: 1.5.0

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1

    - name: Terraform Format
      id: fmt
      run: terraform fmt -check
      continue-on-error: true

    - name: Terraform Init
      id: init
      run: terraform init

    - name: Terraform Validate
      id: validate
      run: terraform validate -no-color

    - name: Terraform Plan
      id: plan
      if: github.event_name == 'pull_request'
      run: terraform plan -no-color -input=false -var-file="environments/${{ github.head_ref }}.tfvars"
      continue-on-error: true

    - name: Update Pull Request
      uses: actions/github-script@v6
      if: github.event_name == 'pull_request'
      env:
        PLAN: "terraform\n${{ steps.plan.outputs.stdout }}"
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }}
        script: |
          const output = `#### Terraform Format and Style 🖌\`${{ steps.fmt.outcome }}\`
          #### Terraform Initialization ⚙️\`${{ steps.init.outcome }}\`
          #### Terraform Validation 🤖\`${{ steps.validate.outcome }}\`
          #### Terraform Plan 📖\`${{ steps.plan.outcome }}\`

          <details><summary>Show Plan</summary>

          \`\`\`\n
          ${process.env.PLAN}
          \`\`\`

          </details>

          *Pushed by: @${{ github.actor }}, Action: \`${{ github.event_name }}\`*`;

          github.rest.issues.createComment({
            issue_number: context.issue.number,
            owner: context.repo.owner,
            repo: context.repo.repo,
            body: output
          })

    - name: Terraform Plan Status
      if: steps.plan.outcome == 'failure'
      run: exit 1

    - name: Terraform Apply
      if: github.ref == 'refs/heads/main' && github.event_name == 'push'
      run: terraform apply -auto-approve -input=false -var-file="environments/prod.tfvars"

    - name: Terraform Output
      if: github.ref == 'refs/heads/main' && github.event_name == 'push'
      run: terraform output -json > terraform-outputs.json

    - name: Upload Terraform Outputs
      if: github.ref == 'refs/heads/main' && github.event_name == 'push'
      uses: actions/upload-artifact@v3
      with:
        name: terraform-outputs
        path: ./infrastructure/terraform-outputs.json
```

## Security and Compliance in IaC

### Security Best Practices
```python
# Security scanning and validation
import json
import re
from typing import Dict, List, Any

class IaCSecurityScanner:
    
    def __init__(self):
        self.security_rules = {
            'aws_s3_bucket': self._check_s3_security,
            'aws_security_group': self._check_security_group,
            'aws_iam_role': self._check_iam_role,
            'aws_rds_instance': self._check_rds_security,
            'aws_instance': self._check_ec2_security
        }
    
    def scan_terraform_file(self, terraform_content: str) -> Dict[str, List[str]]:
        """Scan Terraform file for security issues"""
        
        issues = {
            'critical': [],
            'high': [],
            'medium': [],
            'low': []
        }
        
        try:
            # Parse HCL content (simplified parsing)
            resources = self._parse_terraform_resources(terraform_content)
            
            for resource_type, resources_list in resources.items():
                if resource_type in self.security_rules:
                    for resource in resources_list:
                        resource_issues = self.security_rules[resource_type](resource)
                        for severity, issue_list in resource_issues.items():
                            issues[severity].extend(issue_list)
            
        except Exception as e:
            issues['critical'].append(f"Failed to parse Terraform file: {str(e)}")
        
        return issues
    
    def _parse_terraform_resources(self, content: str) -> Dict[str, List[Dict]]:
        """Simplified Terraform resource parsing"""
        resources = {}
        
        # This is a simplified parser - in practice, use proper HCL parser
        resource_pattern = r'resource\s+"([^"]+)"\s+"([^"]+)"\s*\{([^}]+)\}'
        matches = re.findall(resource_pattern, content, re.DOTALL)
        
        for resource_type, resource_name, resource_body in matches:
            if resource_type not in resources:
                resources[resource_type] = []
            
            # Parse resource attributes (simplified)
            attributes = self._parse_attributes(resource_body)
            attributes['name'] = resource_name
            resources[resource_type].append(attributes)
        
        return resources
    
    def _parse_attributes(self, body: str) -> Dict[str, Any]:
        """Parse resource attributes from body"""
        attributes = {}
        
        # Simple key-value parsing
        lines = body.strip().split('\n')
        for line in lines:
            line = line.strip()
            if '=' in line:
                key, value = line.split('=', 1)
                key = key.strip()
                value = value.strip().strip('"')
                attributes[key] = value
        
        return attributes
    
    def _check_s3_security(self, resource: Dict[str, Any]) -> Dict[str, List[str]]:
        """Check S3 bucket security configuration"""
        issues = {'critical': [], 'high': [], 'medium': [], 'low': []}
        
        resource_name = resource.get('name', 'unknown')
        
        # Check for public read access
        if resource.get('acl') == 'public-read':
            issues['critical'].append(
                f"S3 bucket '{resource_name}' has public read access"
            )
        
        # Check for public read-write access
        if resource.get('acl') == 'public-read-write':
            issues['critical'].append(
                f"S3 bucket '{resource_name}' has public read-write access"
            )
        
        # Check for encryption
        if 'server_side_encryption_configuration' not in resource:
            issues['high'].append(
                f"S3 bucket '{resource_name}' does not have encryption enabled"
            )
        
        # Check for versioning
        if resource.get('versioning', {}).get('enabled') != 'true':
            issues['medium'].append(
                f"S3 bucket '{resource_name}' does not have versioning enabled"
            )
        
        # Check for logging
        if 'logging' not in resource:
            issues['low'].append(
                f"S3 bucket '{resource_name}' does not have access logging enabled"
            )
        
        return issues
    
    def _check_security_group(self, resource: Dict[str, Any]) -> Dict[str, List[str]]:
        """Check security group configuration"""
        issues = {'critical': [], 'high': [], 'medium': [], 'low': []}
        
        resource_name = resource.get('name', 'unknown')
        
        # Check for overly permissive ingress rules
        ingress_rules = resource.get('ingress', [])
        if not isinstance(ingress_rules, list):
            ingress_rules = [ingress_rules]
        
        for rule in ingress_rules:
            if isinstance(rule, dict):
                cidr_blocks = rule.get('cidr_blocks', [])
                if '0.0.0.0/0' in cidr_blocks:
                    from_port = rule.get('from_port', 0)
                    to_port = rule.get('to_port', 0)
                    
                    if from_port == 22 or to_port == 22:
                        issues['critical'].append(
                            f"Security group '{resource_name}' allows SSH (port 22) from anywhere"
                        )
                    elif from_port == 3389 or to_port == 3389:
                        issues['critical'].append(
                            f"Security group '{resource_name}' allows RDP (port 3389) from anywhere"
                        )
                    elif from_port == 0 and to_port == 65535:
                        issues['critical'].append(
                            f"Security group '{resource_name}' allows all traffic from anywhere"
                        )
        
        return issues
    
    def _check_iam_role(self, resource: Dict[str, Any]) -> Dict[str, List[str]]:
        """Check IAM role configuration"""
        issues = {'critical': [], 'high': [], 'medium': [], 'low': []}
        
        resource_name = resource.get('name', 'unknown')
        
        # Check for overly permissive policies
        assume_role_policy = resource.get('assume_role_policy', '')
        if '*' in assume_role_policy:
            issues['high'].append(
                f"IAM role '{resource_name}' has overly permissive assume role policy"
            )
        
        return issues
    
    def _check_rds_security(self, resource: Dict[str, Any]) -> Dict[str, List[str]]:
        """Check RDS instance security"""
        issues = {'critical': [], 'high': [], 'medium': [], 'low': []}
        
        resource_name = resource.get('name', 'unknown')
        
        # Check for public accessibility
        if resource.get('publicly_accessible') == 'true':
            issues['critical'].append(
                f"RDS instance '{resource_name}' is publicly accessible"
            )
        
        # Check for encryption
        if resource.get('storage_encrypted') != 'true':
            issues['high'].append(
                f"RDS instance '{resource_name}' does not have storage encryption enabled"
            )
        
        # Check for backup retention
        backup_retention = resource.get('backup_retention_period', '0')
        if int(backup_retention) < 7:
            issues['medium'].append(
                f"RDS instance '{resource_name}' has backup retention period less than 7 days"
            )
        
        return issues
    
    def _check_ec2_security(self, resource: Dict[str, Any]) -> Dict[str, List[str]]:
        """Check EC2 instance security"""
        issues = {'critical': [], 'high': [], 'medium': [], 'low': []}
        
        resource_name = resource.get('name', 'unknown')
        
        # Check for IMDSv2 enforcement
        metadata_options = resource.get('metadata_options', {})
        if metadata_options.get('http_tokens') != 'required':
            issues['medium'].append(
                f"EC2 instance '{resource_name}' does not enforce IMDSv2"
            )
        
        # Check for monitoring
        if resource.get('monitoring') != 'true':
            issues['low'].append(
                f"EC2 instance '{resource_name}' does not have detailed monitoring enabled"
            )
        
        return issues
    
    def generate_report(self, issues: Dict[str, List[str]]) -> str:
        """Generate security scan report"""
        
        report = "# Infrastructure Security Scan Report\n\n"
        
        total_issues = sum(len(issue_list) for issue_list in issues.values())
        report += f"**Total Issues Found: {total_issues}**\n\n"
        
        for severity in ['critical', 'high', 'medium', 'low']:
            if issues[severity]:
                report += f"## {severity.upper()} Issues ({len(issues[severity])})\n\n"
                for i, issue in enumerate(issues[severity], 1):
                    report += f"{i}. {issue}\n"
                report += "\n"
        
        if total_issues == 0:
            report += "✅ No security issues found!\n"
        
        return report

# Usage example
def scan_infrastructure():
    scanner = IaCSecurityScanner()
    
    # Read Terraform file
    with open('main.tf', 'r') as f:
        terraform_content = f.read()
    
    # Scan for security issues
    issues = scanner.scan_terraform_file(terraform_content)
    
    # Generate report
    report = scanner.generate_report(issues)
    
    # Save report
    with open('security_scan_report.md', 'w') as f:
        f.write(report)
    
    print("Security scan completed. Report saved to security_scan_report.md")
    
    # Return exit code based on critical issues
    return 1 if issues['critical'] else 0

if __name__ == "__main__":
    exit_code = scan_infrastructure()
    exit(exit_code)
```

### Compliance as Code
```yaml
# Open Policy Agent (OPA) policies for Terraform
# policies/terraform.rego
package terraform.security

import future.keywords.in

# Deny S3 buckets with public access
deny[msg] {
    resource := input.resource_changes[_]
    resource.type == "aws_s3_bucket"
    resource.change.after.acl in ["public-read", "public-read-write"]
    
    msg := sprintf("S3 bucket '%s' must not have public access", [resource.address])
}

# Deny security groups with SSH access from anywhere
deny[msg] {
    resource := input.resource_changes[_]
    resource.type == "aws_security_group"
    
    rule := resource.change.after.ingress[_]
    rule.from_port == 22
    "0.0.0.0/0" in rule.cidr_blocks
    
    msg := sprintf("Security group '%s' must not allow SSH access from anywhere", [resource.address])
}

# Require encryption for RDS instances
deny[msg] {
    resource := input.resource_changes[_]
    resource.type == "aws_db_instance"
    resource.change.after.storage_encrypted != true
    
    msg := sprintf("RDS instance '%s' must have storage encryption enabled", [resource.address])
}

# Require specific instance types in production
deny[msg] {
    resource := input.resource_changes[_]
    resource.type == "aws_instance"
    
    # Check if this is production environment
    tags := resource.change.after.tags
    tags.Environment == "production"
    
    # Only allow specific instance types in production
    allowed_types := ["t3.medium", "t3.large", "m5.large", "m5.xlarge"]
    not resource.change.after.instance_type in allowed_types
    
    msg := sprintf("EC2 instance '%s' in production must use approved instance types", [resource.address])
}

# Require backup retention for production RDS
deny[msg] {
    resource := input.resource_changes[_]
    resource.type == "aws_db_instance"
    
    tags := resource.change.after.tags
    tags.Environment == "production"
    
    resource.change.after.backup_retention_period < 7
    
    msg := sprintf("RDS instance '%s' in production must have backup retention >= 7 days", [resource.address])
}

# Warning for resources without proper tagging
warn[msg] {
    resource := input.resource_changes[_]
    resource.type in ["aws_instance", "aws_s3_bucket", "aws_db_instance"]
    
    required_tags := ["Environment", "Project", "Owner"]
    tags := resource.change.after.tags
    
    missing_tag := required_tags[_]
    not tags[missing_tag]
    
    msg := sprintf("Resource '%s' is missing required tag '%s'", [resource.address, missing_tag])
}
```

## Monitoring and Observability for IaC

### Infrastructure Monitoring
```python
# Infrastructure monitoring and alerting
import boto3
import json
from datetime import datetime, timedelta
from typing import Dict, List, Any

class InfrastructureMonitor:
    
    def __init__(self, aws_region='us-east-1'):
        self.cloudwatch = boto3.client('cloudwatch', region_name=aws_region)
        self.ec2 = boto3.client('ec2', region_name=aws_region)
        self.rds = boto3.client('rds', region_name=aws_region)
        self.elb = boto3.client('elbv2', region_name=aws_region)
        
    def create_infrastructure_dashboard(self, dashboard_name: str, environment: str):
        """Create CloudWatch dashboard for infrastructure monitoring"""
        
        dashboard_body = {
            "widgets": [
                {
                    "type": "metric",
                    "x": 0, "y": 0,
                    "width": 12, "height": 6,
                    "properties": {
                        "metrics": [
                            ["AWS/EC2", "CPUUtilization", "Environment", environment],
                            [".", "NetworkIn", ".", "."],
                            [".", "NetworkOut", ".", "."]
                        ],
                        "period": 300,
                        "stat": "Average",
                        "region": "us-east-1",
                        "title": f"{environment} - EC2 Metrics"
                    }
                },
                {
                    "type": "metric",
                    "x": 12, "y": 0,
                    "width": 12, "height": 6,
                    "properties": {
                        "metrics": [
                            ["AWS/ApplicationELB", "RequestCount", "Environment", environment],
                            [".", "TargetResponseTime", ".", "."],
                            [".", "HTTPCode_Target_2XX_Count", ".", "."],
                            [".", "HTTPCode_Target_4XX_Count", ".", "."],
                            [".", "HTTPCode_Target_5XX_Count", ".", "."]
                        ],
                        "period": 300,
                        "stat": "Sum",
                        "region": "us-east-1",
                        "title": f"{environment} - Load Balancer Metrics"
                    }
                },
                {
                    "type": "metric",
                    "x": 0, "y": 6,
                    "width": 12, "height": 6,
                    "properties": {
                        "metrics": [
                            ["AWS/RDS", "CPUUtilization", "Environment", environment],
                            [".", "DatabaseConnections", ".", "."],
                            [".", "ReadLatency", ".", "."],
                            [".", "WriteLatency", ".", "."]
                        ],
                        "period": 300,
                        "stat": "Average",
                        "region": "us-east-1",
                        "title": f"{environment} - RDS Metrics"
                    }
                },
                {
                    "type": "log",
                    "x": 12, "y": 6,
                    "width": 12, "height": 6,
                    "properties": {
                        "query": f"SOURCE '/aws/lambda/{environment}-*' | fields @timestamp, @message\n| filter @message like /ERROR/\n| sort @timestamp desc\n| limit 100",
                        "region": "us-east-1",
                        "title": f"{environment} - Error Logs"
                    }
                }
            ]
        }
        
        self.cloudwatch.put_dashboard(
            DashboardName=dashboard_name,
            DashboardBody=json.dumps(dashboard_body)
        )
    
    def create_infrastructure_alarms(self, environment: str):
        """Create CloudWatch alarms for infrastructure monitoring"""
        
        alarms = [
            {
                'AlarmName': f'{environment}-high-cpu-utilization',
                'ComparisonOperator': 'GreaterThanThreshold',
                'EvaluationPeriods': 2,
                'MetricName': 'CPUUtilization',
                'Namespace': 'AWS/EC2',
                'Period': 300,
                'Statistic': 'Average',
                'Threshold': 80.0,
                'ActionsEnabled': True,
                'AlarmActions': [
                    f'arn:aws:sns:us-east-1:123456789012:{environment}-alerts'
                ],
                'AlarmDescription': f'High CPU utilization in {environment}',
                'Dimensions': [
                    {
                        'Name': 'Environment',
                        'Value': environment
                    }
                ]
            },
            {
                'AlarmName': f'{environment}-high-response-time',
                'ComparisonOperator': 'GreaterThanThreshold',
                'EvaluationPeriods': 3,
                'MetricName': 'TargetResponseTime',
                'Namespace': 'AWS/ApplicationELB',
                'Period': 300,
                'Statistic': 'Average',
                'Threshold': 2.0,
                'ActionsEnabled': True,
                'AlarmActions': [
                    f'arn:aws:sns:us-east-1:123456789012:{environment}-alerts'
                ],
                'AlarmDescription': f'High response time in {environment}',
                'Dimensions': [
                    {
                        'Name': 'Environment',
                        'Value': environment
                    }
                ]
            },
            {
                'AlarmName': f'{environment}-high-error-rate',
                'ComparisonOperator': 'GreaterThanThreshold',
                'EvaluationPeriods': 2,
                'MetricName': 'HTTPCode_Target_5XX_Count',
                'Namespace': 'AWS/ApplicationELB',
                'Period': 300,
                'Statistic': 'Sum',
                'Threshold': 10.0,
                'ActionsEnabled': True,
                'AlarmActions': [
                    f'arn:aws:sns:us-east-1:123456789012:{environment}-alerts'
                ],
                'AlarmDescription': f'High error rate in {environment}',
                'Dimensions': [
                    {
                        'Name': 'Environment',
                        'Value': environment
                    }
                ]
            }
        ]
        
        for alarm in alarms:
            self.cloudwatch.put_metric_alarm(**alarm)
    
    def get_infrastructure_health(self, environment: str) -> Dict[str, Any]:
        """Get overall infrastructure health status"""
        
        health_status = {
            'environment': environment,
            'timestamp': datetime.utcnow().isoformat(),
            'overall_status': 'healthy',
            'components': {}
        }
        
        # Check EC2 instances
        ec2_health = self._check_ec2_health(environment)
        health_status['components']['ec2'] = ec2_health
        
        # Check RDS instances
        rds_health = self._check_rds_health(environment)
        health_status['components']['rds'] = rds_health
        
        # Check Load Balancers
        elb_health = self._check_elb_health(environment)
        health_status['components']['load_balancers'] = elb_health
        
        # Determine overall status
        component_statuses = [comp['status'] for comp in health_status['components'].values()]
        if 'critical' in component_statuses:
            health_status['overall_status'] = 'critical'
        elif 'warning' in component_statuses:
            health_status['overall_status'] = 'warning'
        
        return health_status
    
    def _check_ec2_health(self, environment: str) -> Dict[str, Any]:
        """Check EC2 instances health"""
        
        try:
            # Get EC2 instances for environment
            response = self.ec2.describe_instances(
                Filters=[
                    {'Name': 'tag:Environment', 'Values': [environment]},
                    {'Name': 'instance-state-name', 'Values': ['running']}
                ]
            )
            
            instances = []
            for reservation in response['Reservations']:
                instances.extend(reservation['Instances'])
            
            if not instances:
                return {
                    'status': 'warning',
                    'message': f'No running instances found in {environment}',
                    'count': 0
                }
            
            # Check instance health
            unhealthy_instances = []
            for instance in instances:
                # Get CPU utilization for last 5 minutes
                cpu_metrics = self.cloudwatch.get_metric_statistics(
                    Namespace='AWS/EC2',
                    MetricName='CPUUtilization',
                    Dimensions=[
                        {'Name': 'InstanceId', 'Value': instance['InstanceId']}
                    ],
                    StartTime=datetime.utcnow() - timedelta(minutes=10),
                    EndTime=datetime.utcnow(),
                    Period=300,
                    Statistics=['Average']
                )
                
                if cpu_metrics['Datapoints']:
                    latest_cpu = cpu_metrics['Datapoints'][-1]['Average']
                    if latest_cpu > 90:
                        unhealthy_instances.append({
                            'instance_id': instance['InstanceId'],
                            'issue': f'High CPU: {latest_cpu:.1f}%'
                        })
            
            if unhealthy_instances:
                return {
                    'status': 'warning',
                    'message': f'{len(unhealthy_instances)} instances have high CPU',
                    'count': len(instances),
                    'unhealthy': unhealthy_instances
                }
            
            return {
                'status': 'healthy',
                'message': f'All {len(instances)} instances are healthy',
                'count': len(instances)
            }
            
        except Exception as e:
            return {
                'status': 'critical',
                'message': f'Error checking EC2 health: {str(e)}',
                'count': 0
            }
    
    def _check_rds_health(self, environment: str) -> Dict[str, Any]:
        """Check RDS instances health"""
        
        try:
            # Get RDS instances for environment
            response = self.rds.describe_db_instances()
            
            environment_instances = []
            for instance in response['DBInstances']:
                tags_response = self.rds.list_tags_for_resource(
                    ResourceName=instance['DBInstanceArn']
                )
                
                tags = {tag['Key']: tag['Value'] for tag in tags_response['TagList']}
                if tags.get('Environment') == environment:
                    environment_instances.append(instance)
            
            if not environment_instances:
                return {
                    'status': 'warning',
                    'message': f'No RDS instances found in {environment}',
                    'count': 0
                }
            
            unhealthy_instances = []
            for instance in environment_instances:
                if instance['DBInstanceStatus'] != 'available':
                    unhealthy_instances.append({
                        'instance_id': instance['DBInstanceIdentifier'],
                        'issue': f'Status: {instance["DBInstanceStatus"]}'
                    })
            
            if unhealthy_instances:
                return {
                    'status': 'critical',
                    'message': f'{len(unhealthy_instances)} RDS instances are unhealthy',
                    'count': len(environment_instances),
                    'unhealthy': unhealthy_instances
                }
            
            return {
                'status': 'healthy',
                'message': f'All {len(environment_instances)} RDS instances are healthy',
                'count': len(environment_instances)
            }
            
        except Exception as e:
            return {
                'status': 'critical',
                'message': f'Error checking RDS health: {str(e)}',
                'count': 0
            }
    
    def _check_elb_health(self, environment: str) -> Dict[str, Any]:
        """Check Load Balancer health"""
        
        try:
            # Get load balancers for environment
            response = self.elb.describe_load_balancers()
            
            environment_lbs = []
            for lb in response['LoadBalancers']:
                tags_response = self.elb.describe_tags(
                    ResourceArns=[lb['LoadBalancerArn']]
                )
                
                tags = {}
                if tags_response['TagDescriptions']:
                    tags = {tag['Key']: tag['Value'] 
                           for tag in tags_response['TagDescriptions'][0]['Tags']}
                
                if tags.get('Environment') == environment:
                    environment_lbs.append(lb)
            
            if not environment_lbs:
                return {
                    'status': 'warning',
                    'message': f'No load balancers found in {environment}',
                    'count': 0
                }
            
            unhealthy_lbs = []
            for lb in environment_lbs:
                if lb['State']['Code'] != 'active':
                    unhealthy_lbs.append({
                        'lb_name': lb['LoadBalancerName'],
                        'issue': f'State: {lb["State"]["Code"]}'
                    })
            
            if unhealthy_lbs:
                return {
                    'status': 'critical',
                    'message': f'{len(unhealthy_lbs)} load balancers are unhealthy',
                    'count': len(environment_lbs),
                    'unhealthy': unhealthy_lbs
                }
            
            return {
                'status': 'healthy',
                'message': f'All {len(environment_lbs)} load balancers are healthy',
                'count': len(environment_lbs)
            }
            
        except Exception as e:
            return {
                'status': 'critical',
                'message': f'Error checking ELB health: {str(e)}',
                'count': 0
            }

# Usage example
def setup_monitoring():
    monitor = InfrastructureMonitor()
    
    environments = ['dev', 'staging', 'prod']
    
    for env in environments:
        # Create dashboard
        monitor.create_infrastructure_dashboard(f'{env}-infrastructure', env)
        
        # Create alarms
        monitor.create_infrastructure_alarms(env)
        
        # Check health
        health = monitor.get_infrastructure_health(env)
        print(f"{env} environment health: {health['overall_status']}")

if __name__ == "__main__":
    setup_monitoring()
```

## Conclusion

Infrastructure as Code (IaC) is a fundamental practice in modern cloud computing that provides:

### Key Benefits
1. **Consistency**: Reproducible infrastructure deployments
2. **Version Control**: Track changes and enable rollbacks
3. **Automation**: Reduce manual errors and deployment time
4. **Scalability**: Easily replicate environments and scale resources
5. **Documentation**: Self-documenting infrastructure
6. **Collaboration**: Enable team collaboration on infrastructure

### Best Practices
- Use declarative approaches when possible
- Implement proper state management
- Follow security and compliance guidelines
- Integrate with CI/CD pipelines
- Monitor and validate infrastructure continuously
- Use modular and reusable components

### When to Use IaC
- **Cloud Migrations**: Moving from on-premises to cloud
- **Multi-Environment Deployments**: Dev, staging, production consistency
- **Disaster Recovery**: Rapid infrastructure recreation
- **Compliance**: Auditable and repeatable deployments
- **Team Collaboration**: Shared infrastructure management

IaC is essential for organizations adopting cloud-native practices and DevOps methodologies, enabling faster, more reliable, and more secure infrastructure management at scale.
