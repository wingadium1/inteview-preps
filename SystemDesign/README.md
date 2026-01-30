# System Design Interview Preparation

## Core System Design Concepts

### 1. Scalability

#### Vertical Scaling (Scale Up)
- Add more power to existing machine
- Increase CPU, RAM, storage
- Limitations: Hardware limits, single point of failure
- Use case: Quick fix, legacy systems

#### Horizontal Scaling (Scale Out)
- Add more machines
- Distribute load across multiple servers
- Benefits: No hardware limits, fault tolerance
- Challenges: Data consistency, session management

### 2. Load Balancing

#### Load Balancer Types
- **Layer 4 (Transport)**: TCP/UDP level
- **Layer 7 (Application)**: HTTP/HTTPS level

#### Load Balancing Algorithms
- **Round Robin**: Distribute requests evenly
- **Least Connections**: Route to server with fewest connections
- **IP Hash**: Route based on client IP
- **Weighted Round Robin**: Based on server capacity
- **Least Response Time**: Route to fastest server

#### Popular Load Balancers
- Nginx
- HAProxy
- AWS ELB/ALB/NLB
- Google Cloud Load Balancer

### 3. Caching

#### Cache Strategies
- **Cache-Aside (Lazy Loading)**: Application manages cache
- **Write-Through**: Write to cache and database simultaneously
- **Write-Back**: Write to cache first, database later
- **Refresh-Ahead**: Refresh cache before expiration

#### Cache Eviction Policies
- **LRU (Least Recently Used)**: Remove least recently used
- **LFU (Least Frequently Used)**: Remove least frequently used
- **FIFO**: First in, first out
- **TTL**: Time-to-live expiration

#### Caching Layers
- Client-side cache (Browser)
- CDN cache
- Application cache (Redis, Memcached)
- Database cache

### 4. Database Design

#### SQL vs NoSQL

**SQL (Relational)**
- Structured data
- ACID transactions
- Complex queries
- Examples: MySQL, PostgreSQL, Oracle

**NoSQL**
- Unstructured/semi-structured data
- Horizontal scaling
- Types:
  - Key-Value: Redis, DynamoDB
  - Document: MongoDB, Couchbase
  - Column-Family: Cassandra, HBase
  - Graph: Neo4j, Amazon Neptune

#### Database Scaling

**Read Replicas**
- Replicate data to read-only databases
- Distribute read load
- Master handles writes, replicas handle reads

**Sharding (Partitioning)**
- Split data across multiple databases
- Horizontal partitioning
- Strategies:
  - Range-based: By ID ranges
  - Hash-based: By hash function
  - Geographic: By location

**Database Indexing**
- Speed up queries
- Trade-off: Faster reads, slower writes
- Types: B-tree, Hash, Bitmap

### 5. Consistency and Availability

#### CAP Theorem
Can only achieve 2 out of 3:
- **Consistency**: All nodes see same data
- **Availability**: System always responds
- **Partition Tolerance**: System works despite network failures

#### Consistency Models
- **Strong Consistency**: Immediate consistency
- **Eventual Consistency**: Eventually consistent
- **Read-after-write Consistency**: See own writes immediately

#### ACID vs BASE

**ACID (SQL)**
- Atomicity
- Consistency
- Isolation
- Durability

**BASE (NoSQL)**
- Basically Available
- Soft state
- Eventual consistency

### 6. Message Queues

#### Benefits
- Asynchronous processing
- Decouple services
- Load leveling
- Reliability

#### Popular Message Queues
- RabbitMQ
- Apache Kafka
- Amazon SQS
- Redis Pub/Sub

#### Patterns
- **Point-to-Point**: One producer, one consumer
- **Publish-Subscribe**: One producer, multiple consumers
- **Request-Reply**: Synchronous communication

### 7. API Design

#### REST API
- Resource-based
- HTTP methods: GET, POST, PUT, DELETE, PATCH
- Stateless
- JSON format

#### GraphQL
- Query language for APIs
- Request exactly what you need
- Single endpoint

#### gRPC
- Protocol Buffers
- HTTP/2
- Bi-directional streaming

#### API Gateway
- Single entry point
- Authentication/Authorization
- Rate limiting
- Request routing
- Load balancing

### 8. Microservices Architecture

#### Characteristics
- Small, independent services
- Loosely coupled
- Independently deployable
- Technology agnostic

#### Communication
- Synchronous: REST, gRPC
- Asynchronous: Message queues, events

#### Challenges
- Distributed system complexity
- Data consistency
- Service discovery
- Monitoring and debugging

#### Patterns
- API Gateway
- Service Discovery (Consul, Eureka)
- Circuit Breaker (Hystrix)
- Event Sourcing
- CQRS (Command Query Responsibility Segregation)

### 9. Security

#### Authentication
- Username/Password
- OAuth 2.0
- JWT (JSON Web Tokens)
- Multi-factor Authentication

#### Authorization
- Role-Based Access Control (RBAC)
- Attribute-Based Access Control (ABAC)

#### Security Practices
- HTTPS/TLS
- Input validation
- SQL injection prevention
- XSS prevention
- CSRF protection
- Rate limiting
- DDoS protection

### 10. Monitoring and Observability

#### Metrics
- System metrics: CPU, memory, disk
- Application metrics: Request rate, error rate, latency
- Business metrics: Active users, conversions

#### Logging
- Centralized logging (ELK, Splunk)
- Structured logging
- Log levels: DEBUG, INFO, WARN, ERROR

#### Tracing
- Distributed tracing (Jaeger, Zipkin)
- Track requests across services

#### Alerting
- Define thresholds
- On-call rotation
- Incident management

## Common System Design Interview Questions

### Design Questions

#### 1. Design URL Shortener (like bit.ly)
**Requirements:**
- Shorten long URLs
- Redirect to original URL
- Analytics

**Components:**
- Load balancer
- Application servers
- Database (URL mappings)
- Cache (Redis)
- Counter service

**Key Points:**
- Base62 encoding for short URLs
- Database sharding by hash
- Cache frequently accessed URLs

#### 2. Design Twitter/Social Media Feed
**Requirements:**
- Post tweets
- Follow users
- View feed

**Components:**
- User service
- Tweet service
- Timeline service
- Notification service
- Cache layer
- Message queue

**Key Points:**
- Fan-out on write vs fan-out on read
- Timeline generation
- Push vs pull model

#### 3. Design WhatsApp/Chat Application
**Requirements:**
- One-on-one messaging
- Group chat
- Online status
- Message delivery

**Components:**
- WebSocket server
- Message service
- User service
- Notification service
- Database (messages, users)
- Cache

**Key Points:**
- WebSocket for real-time
- Message queue for reliability
- Read receipts
- Last seen status

#### 4. Design Uber/Ride-Sharing Service
**Requirements:**
- Match riders with drivers
- Real-time location
- Pricing
- Payment

**Components:**
- Location service
- Matching service
- Payment service
- Notification service
- Database
- Cache

**Key Points:**
- Geospatial indexing (QuadTree)
- Real-time updates
- Surge pricing
- Driver location updates

#### 5. Design Netflix/Video Streaming
**Requirements:**
- Upload videos
- Stream videos
- Recommendations

**Components:**
- Upload service
- Encoding service
- CDN
- Recommendation engine
- Database
- Object storage (S3)

**Key Points:**
- Video encoding (different formats)
- Adaptive bitrate streaming
- CDN for content delivery
- Machine learning for recommendations

#### 6. Design Instagram
**Requirements:**
- Upload photos
- Follow users
- View feed
- Like/comment

**Components:**
- Image service
- Feed service
- User service
- Social graph service
- Object storage
- CDN

**Key Points:**
- Image processing
- Feed generation
- Timeline caching
- Notification system

## System Design Process

### 1. Requirements Clarification (5 min)
- Functional requirements
- Non-functional requirements
- Scale (users, requests, data)
- Constraints

### 2. Back-of-the-Envelope Estimation (5 min)
- Storage requirements
- Bandwidth requirements
- Number of servers needed

### 3. System Interface Definition (5 min)
- API design
- Define endpoints

### 4. Data Model (5 min)
- Database schema
- Tables and relationships

### 5. High-Level Design (10 min)
- Draw block diagram
- Identify major components
- Data flow

### 6. Detailed Design (15 min)
- Deep dive into components
- Discuss trade-offs
- Address bottlenecks

### 7. Bottlenecks and Scaling (5 min)
- Single points of failure
- Performance bottlenecks
- Scaling strategies

## Key Principles

### 1. Trade-offs
Every design decision has trade-offs. Discuss:
- Consistency vs Availability
- Latency vs Throughput
- Read-heavy vs Write-heavy
- SQL vs NoSQL

### 2. Numbers to Remember
- L1 cache: 0.5 ns
- L2 cache: 7 ns
- RAM: 100 ns
- SSD: 150,000 ns
- Network (same datacenter): 500,000 ns
- Network (cross-continent): 150,000,000 ns

### 3. Availability
- 99% = 3.65 days downtime/year
- 99.9% = 8.76 hours downtime/year
- 99.99% = 52.56 minutes downtime/year
- 99.999% = 5.26 minutes downtime/year

## Best Practices
1. Start with high-level design
2. Identify bottlenecks early
3. Consider failure scenarios
4. Think about monitoring
5. Discuss trade-offs
6. Ask clarifying questions
7. Use real-world numbers
8. Draw diagrams
9. Be specific about technologies
10. Consider cost implications

## Resources
- Designing Data-Intensive Applications by Martin Kleppmann
- System Design Interview by Alex Xu
- Grokking the System Design Interview
- High Scalability blog
- Architecture of Open Source Applications
