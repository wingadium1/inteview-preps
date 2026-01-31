# AWS Interview Preparation

## Core AWS Services

### 1. Compute Services

#### EC2 (Elastic Compute Cloud)
- **Instance Types**: General purpose, Compute optimized, Memory optimized
- **Pricing Models**: On-Demand, Reserved, Spot, Savings Plans
- **AMI (Amazon Machine Image)**: Pre-configured templates
- **Security Groups**: Virtual firewalls
- **Key Pairs**: SSH access
- **Elastic IPs**: Static IP addresses
- **Auto Scaling**: Automatically adjust capacity
- **Load Balancers**: ALB, NLB, CLB

#### Lambda
- **Serverless compute**: Run code without servers
- **Event-driven**: Triggers from various AWS services
- **Pricing**: Pay per request and compute time
- **Use Cases**: API backends, data processing, automation
- **Cold Starts**: Initial invocation latency
- **Layers**: Share code across functions
- **Environment Variables**: Configuration

### 2. Storage Services

#### S3 (Simple Storage Service)
- **Object Storage**: Store and retrieve any amount of data
- **Storage Classes**: Standard, IA, One Zone-IA, Glacier, Deep Archive
- **Bucket Policies**: Access control
- **Versioning**: Keep multiple versions of objects
- **Lifecycle Policies**: Automate transitions
- **Cross-Region Replication**: Replicate across regions
- **Pre-signed URLs**: Temporary access
- **S3 Transfer Acceleration**: Faster uploads

#### EBS (Elastic Block Store)
- **Block-level storage**: Attach to EC2 instances
- **Volume Types**: gp3, gp2, io2, io1, st1, sc1
- **Snapshots**: Point-in-time backups
- **Encryption**: At-rest and in-transit

#### EFS (Elastic File System)
- **Network file system**: Shared across multiple EC2 instances
- **Scalable**: Automatically grows and shrinks
- **Performance Modes**: General Purpose, Max I/O

### 3. Database Services

#### RDS (Relational Database Service)
- **Engines**: MySQL, PostgreSQL, Oracle, SQL Server, MariaDB
- **Multi-AZ**: High availability
- **Read Replicas**: Scale read operations
- **Automated Backups**: Point-in-time recovery
- **Parameter Groups**: Configuration
- **Storage Autoscaling**: Automatic capacity increase

#### DynamoDB
- **NoSQL Database**: Key-value and document store
- **Performance**: Single-digit millisecond latency
- **Scalability**: Automatic scaling
- **Global Tables**: Multi-region replication
- **Streams**: Capture changes
- **DAX**: In-memory cache
- **Indexes**: GSI, LSI

#### ElastiCache
- **In-memory cache**: Redis and Memcached
- **Use Cases**: Session store, caching, real-time analytics
- **Cluster Mode**: Sharding for Redis

### 4. Networking & Content Delivery

#### VPC (Virtual Private Cloud)
- **Subnets**: Public and private subnets
- **Route Tables**: Network traffic routing
- **Internet Gateway**: Connect to internet
- **NAT Gateway**: Outbound internet access for private subnets
- **Security Groups**: Instance-level security
- **NACLs**: Subnet-level security
- **VPC Peering**: Connect VPCs
- **VPN**: Connect on-premises to AWS

#### Route 53
- **DNS Service**: Domain name system
- **Routing Policies**: Simple, Weighted, Latency, Failover, Geolocation
- **Health Checks**: Monitor endpoint health

#### CloudFront
- **CDN**: Content delivery network
- **Edge Locations**: Global network of caches
- **Origins**: S3, EC2, ELB, custom origins
- **Distributions**: Web and RTMP

### 5. Security & Identity

#### IAM (Identity and Access Management)
- **Users**: Individual accounts
- **Groups**: Collection of users
- **Roles**: Temporary credentials for services
- **Policies**: JSON documents defining permissions
- **MFA**: Multi-factor authentication
- **Access Keys**: Programmatic access
- **Best Practices**: Least privilege, rotate credentials

#### KMS (Key Management Service)
- **Encryption Keys**: Create and manage keys
- **CMK**: Customer Master Keys
- **Data Keys**: Encrypt data
- **Envelope Encryption**: Encrypt keys

#### Secrets Manager
- **Store secrets**: Passwords, API keys
- **Rotation**: Automatic rotation
- **Integration**: RDS, Redshift, DocumentDB

### 6. Monitoring & Management

#### CloudWatch
- **Metrics**: Monitor resources and applications
- **Logs**: Centralized logging
- **Alarms**: Notifications and actions
- **Dashboards**: Visualize metrics
- **Events**: React to changes

#### CloudTrail
- **Audit Logs**: API call history
- **Compliance**: Governance and compliance
- **Security Analysis**: Identify security issues

#### Systems Manager
- **Parameter Store**: Configuration management
- **Session Manager**: Shell access without SSH
- **Patch Manager**: Automate patching
- **Run Command**: Execute commands remotely

### 7. Application Integration

#### SQS (Simple Queue Service)
- **Message Queue**: Decouple components
- **Standard Queue**: At-least-once delivery
- **FIFO Queue**: Exactly-once processing, ordered
- **Dead Letter Queue**: Handle failed messages
- **Visibility Timeout**: Message processing time

#### SNS (Simple Notification Service)
- **Pub/Sub**: Publish messages to topics
- **Subscribers**: Email, SMS, HTTP, Lambda, SQS
- **Fan-out**: Multiple subscribers

#### EventBridge
- **Event Bus**: Route events
- **Rules**: Match and route events
- **Targets**: Lambda, SQS, SNS, etc.

## Common Interview Questions

### Basic Questions
1. What is EC2 and what are its use cases?
2. Explain S3 storage classes
3. What is the difference between S3 and EBS?
4. What is IAM and why is it important?
5. Explain VPC and its components

### Intermediate Questions
1. How do you secure an application on AWS?
2. What is the difference between security groups and NACLs?
3. Explain RDS Multi-AZ vs Read Replicas
4. How does Auto Scaling work?
5. What are the benefits of using Lambda?

### Advanced Questions
1. Design a highly available and scalable web application on AWS
2. How do you optimize costs on AWS?
3. Explain AWS disaster recovery strategies
4. How do you implement CI/CD on AWS?
5. What are AWS best practices for security?

## Architecture Patterns

### Three-Tier Architecture
```
Users → CloudFront → ALB → EC2 (Auto Scaling) → RDS (Multi-AZ)
                       ↓
                      S3
```

### Serverless Architecture
```
Users → API Gateway → Lambda → DynamoDB
                       ↓
                      S3
```

### Microservices Architecture
```
Users → ALB → ECS/EKS → RDS/DynamoDB
              ↓
            SQS/SNS
```

## Best Practices
1. **Security**: Use IAM roles, encrypt data, enable MFA
2. **Cost Optimization**: Right-sizing, Reserved Instances, S3 lifecycle
3. **Reliability**: Multi-AZ, Auto Scaling, backups
4. **Performance**: CloudFront, ElastiCache, read replicas
5. **Operational Excellence**: CloudWatch, CloudTrail, automation
6. **Well-Architected Framework**: Follow AWS best practices

## Resources
- AWS Documentation
- AWS Well-Architected Framework
- AWS Certified Solutions Architect Study Guide
- AWS Whitepapers
- AWS re:Invent Videos
