# DevOps Interview Preparation

## Core DevOps Concepts

### 1. DevOps Principles
- **Culture**: Collaboration between Dev and Ops
- **Automation**: Automate repetitive tasks
- **Continuous Integration**: Integrate code frequently
- **Continuous Delivery**: Deploy to production frequently
- **Continuous Deployment**: Automate deployment to production
- **Monitoring**: Continuous monitoring and feedback
- **Infrastructure as Code**: Manage infrastructure using code

### 2. Version Control - Git

#### Git Basics
- **Repository**: Local and remote repositories
- **Commit**: Save changes
- **Branch**: Parallel development
- **Merge**: Combine branches
- **Pull Request**: Code review and merge
- **Rebase**: Rewrite commit history
- **Cherry-pick**: Apply specific commits

#### Git Workflow
- **GitFlow**: Feature, develop, release, hotfix branches
- **Trunk-Based Development**: Short-lived feature branches
- **Forking Workflow**: Fork and pull request

#### Git Commands
```bash
git clone <repo>
git add .
git commit -m "message"
git push origin main
git pull origin main
git branch feature-branch
git checkout feature-branch
git merge feature-branch
git rebase main
git cherry-pick <commit-hash>
git stash
git log --oneline
```

### 3. Continuous Integration/Continuous Deployment (CI/CD)

#### CI/CD Tools
- **Jenkins**: Most popular CI/CD tool
- **GitLab CI**: Integrated with GitLab
- **GitHub Actions**: Integrated with GitHub
- **CircleCI**: Cloud-based CI/CD
- **Travis CI**: Cloud-based CI/CD
- **Azure DevOps**: Microsoft's CI/CD platform

#### Jenkins Pipeline
```groovy
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('Deploy') {
            steps {
                sh './deploy.sh'
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
```

#### GitHub Actions Workflow
```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Set up JDK 11
      uses: actions/setup-java@v2
      with:
        java-version: '11'
        
    - name: Build with Maven
      run: mvn clean package
      
    - name: Run tests
      run: mvn test
      
    - name: Deploy
      run: ./deploy.sh
```

### 4. Infrastructure as Code (IaC)

#### Terraform
```hcl
# Define provider
provider "aws" {
  region = "us-east-1"
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  
  tags = {
    Name = "main-vpc"
  }
}

# Create EC2 instance
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = {
    Name = "web-server"
  }
}

# Output
output "instance_ip" {
  value = aws_instance.web.public_ip
}
```

#### Ansible
```yaml
---
- name: Deploy web application
  hosts: webservers
  become: yes
  
  tasks:
    - name: Install Apache
      apt:
        name: apache2
        state: present
        
    - name: Start Apache
      service:
        name: apache2
        state: started
        enabled: yes
        
    - name: Copy application files
      copy:
        src: /local/path/
        dest: /var/www/html/
```

#### CloudFormation
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Web Server Stack'

Resources:
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: t2.micro
      SecurityGroups:
        - !Ref WebServerSecurityGroup
        
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable HTTP access
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
```

### 5. Configuration Management
- **Ansible**: Agentless, YAML-based
- **Puppet**: Declarative, agent-based
- **Chef**: Imperative, agent-based
- **SaltStack**: Python-based, fast

### 6. Monitoring and Logging

#### Monitoring Tools
- **Prometheus**: Open-source monitoring
- **Grafana**: Visualization and dashboards
- **Nagios**: Infrastructure monitoring
- **Datadog**: Cloud monitoring
- **New Relic**: Application performance monitoring

#### Logging Tools
- **ELK Stack**: Elasticsearch, Logstash, Kibana
- **Splunk**: Log analysis
- **Fluentd**: Log aggregation
- **CloudWatch Logs**: AWS logging

#### Prometheus Configuration
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'application'
    static_configs:
      - targets: ['localhost:9090']
```

### 7. Cloud Platforms
- **AWS**: Amazon Web Services
- **Azure**: Microsoft Azure
- **GCP**: Google Cloud Platform
- **DigitalOcean**: Simple cloud infrastructure

### 8. Scripting and Automation

#### Bash Scripting
```bash
#!/bin/bash

# Deployment script
APP_NAME="myapp"
DEPLOY_DIR="/opt/$APP_NAME"

echo "Deploying $APP_NAME..."

# Stop application
systemctl stop $APP_NAME

# Backup current version
cp -r $DEPLOY_DIR $DEPLOY_DIR.backup

# Deploy new version
cp -r /tmp/$APP_NAME/* $DEPLOY_DIR/

# Start application
systemctl start $APP_NAME

echo "Deployment completed!"
```

#### Python Scripting
```python
import boto3
import sys

def deploy_to_s3(bucket_name, file_path):
    """Deploy file to S3 bucket"""
    s3 = boto3.client('s3')
    
    try:
        s3.upload_file(file_path, bucket_name, file_path)
        print(f"File {file_path} uploaded to {bucket_name}")
    except Exception as e:
        print(f"Error: {e}")
        sys.exit(1)

if __name__ == "__main__":
    deploy_to_s3("my-bucket", "index.html")
```

### 9. Security in DevOps (DevSecOps)
- **Security Scanning**: SAST, DAST
- **Dependency Scanning**: Identify vulnerable dependencies
- **Container Scanning**: Scan container images
- **Secret Management**: Vault, AWS Secrets Manager
- **Compliance**: Automated compliance checks

### 10. Agile and Scrum
- **Sprints**: Time-boxed iterations
- **Stand-ups**: Daily team sync
- **Retrospectives**: Continuous improvement
- **Kanban**: Visual workflow management

## Common Interview Questions

### Basic Questions
1. What is DevOps and why is it important?
2. Explain the DevOps lifecycle
3. What is CI/CD?
4. What is the difference between continuous delivery and continuous deployment?
5. What is Infrastructure as Code?

### Intermediate Questions
1. How do you implement CI/CD pipeline?
2. Explain GitFlow workflow
3. What is blue-green deployment?
4. How do you handle secrets in CI/CD?
5. What are the benefits of Infrastructure as Code?

### Advanced Questions
1. How do you implement zero-downtime deployment?
2. Explain canary deployment strategy
3. How do you monitor microservices?
4. How do you ensure security in DevOps?
5. How do you optimize CI/CD pipeline performance?

## Deployment Strategies

### 1. Blue-Green Deployment
- Maintain two identical environments (blue and green)
- Deploy to inactive environment
- Switch traffic to new environment
- Rollback by switching back

### 2. Canary Deployment
- Deploy to small subset of users
- Monitor metrics
- Gradually increase traffic
- Rollback if issues detected

### 3. Rolling Deployment
- Deploy to one instance at a time
- Gradually replace all instances
- Minimal downtime

### 4. A/B Testing
- Deploy different versions to different users
- Compare metrics
- Choose winner

## Best Practices
1. **Automate Everything**: Build, test, deploy, monitoring
2. **Version Control**: Everything in version control
3. **Test Early and Often**: Shift-left testing
4. **Monitor Continuously**: Metrics, logs, alerts
5. **Security First**: DevSecOps approach
6. **Infrastructure as Code**: Reproducible infrastructure
7. **Immutable Infrastructure**: Replace, don't modify
8. **Microservices**: Small, independent services
9. **Documentation**: Keep documentation up-to-date
10. **Continuous Learning**: Stay updated with tools and practices

## Tools Comparison

| Category | Tools |
|----------|-------|
| CI/CD | Jenkins, GitLab CI, GitHub Actions, CircleCI |
| IaC | Terraform, CloudFormation, Ansible |
| Containers | Docker, Podman |
| Orchestration | Kubernetes, Docker Swarm, ECS |
| Monitoring | Prometheus, Grafana, Datadog, New Relic |
| Logging | ELK Stack, Splunk, Fluentd |
| Version Control | Git, GitHub, GitLab, Bitbucket |
| Cloud | AWS, Azure, GCP |

## Resources
- The Phoenix Project book
- The DevOps Handbook
- Site Reliability Engineering (SRE) book
- DevOps.com
- AWS DevOps Blog
