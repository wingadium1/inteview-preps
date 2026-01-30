# Microservices Interview Preparation

## Core Microservices Concepts

### 1. What are Microservices?

#### Definition
- **Architectural style**: Application as suite of small services
- **Independently deployable**: Each service can be deployed separately
- **Loosely coupled**: Services communicate via well-defined APIs
- **Technology agnostic**: Different services can use different tech stacks
- **Business capability focused**: Each service represents a business capability

#### Monolith vs Microservices

**Monolithic Architecture:**
- Single codebase
- Tight coupling
- Single deployment unit
- Difficult to scale specific components
- Technology lock-in

**Microservices Architecture:**
- Multiple codebases
- Loose coupling
- Independent deployments
- Scale specific services
- Technology flexibility

### 2. Microservices Characteristics

#### Key Characteristics
1. **Componentization via Services**: Services are independently deployable
2. **Organized around Business Capabilities**: Service boundaries aligned with business
3. **Products not Projects**: Teams own services end-to-end
4. **Smart endpoints and dumb pipes**: Simple communication mechanisms
5. **Decentralized Governance**: Teams choose their own technology
6. **Decentralized Data Management**: Each service owns its data
7. **Infrastructure Automation**: CI/CD, automated testing
8. **Design for Failure**: Handle failures gracefully
9. **Evolutionary Design**: Services evolve independently

### 3. Service Communication

#### Synchronous Communication

**REST API**
```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    
    @Autowired
    private RestTemplate restTemplate;
    
    @PostMapping
    public Order createOrder(@RequestBody Order order) {
        // Call user service
        User user = restTemplate.getForObject(
            "http://user-service/api/users/" + order.getUserId(),
            User.class
        );
        
        // Call inventory service
        Boolean available = restTemplate.postForObject(
            "http://inventory-service/api/inventory/check",
            order.getItems(),
            Boolean.class
        );
        
        if (available) {
            return orderRepository.save(order);
        }
        throw new OutOfStockException();
    }
}
```

**gRPC**
```protobuf
syntax = "proto3";

service OrderService {
  rpc CreateOrder (OrderRequest) returns (OrderResponse);
  rpc GetOrder (OrderId) returns (Order);
}

message OrderRequest {
  string user_id = 1;
  repeated Item items = 2;
}

message OrderResponse {
  string order_id = 1;
  string status = 2;
}
```

#### Asynchronous Communication

**Message Queue (RabbitMQ)**
```java
@Service
public class OrderService {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    public void createOrder(Order order) {
        // Save order
        orderRepository.save(order);
        
        // Publish event
        OrderCreatedEvent event = new OrderCreatedEvent(order.getId());
        rabbitTemplate.convertAndSend(
            "order.exchange",
            "order.created",
            event
        );
    }
}

@Component
public class OrderEventListener {
    
    @RabbitListener(queues = "inventory.queue")
    public void handleOrderCreated(OrderCreatedEvent event) {
        // Process order
        inventoryService.reserveItems(event.getOrderId());
    }
}
```

**Event Streaming (Kafka)**
```java
@Service
public class OrderProducer {
    
    @Autowired
    private KafkaTemplate<String, OrderEvent> kafkaTemplate;
    
    public void publishOrder(Order order) {
        OrderEvent event = new OrderEvent(order);
        kafkaTemplate.send("orders", order.getId(), event);
    }
}

@Service
public class OrderConsumer {
    
    @KafkaListener(topics = "orders", groupId = "inventory-service")
    public void consume(OrderEvent event) {
        inventoryService.processOrder(event);
    }
}
```

### 4. Service Discovery

#### Client-Side Discovery
```java
@Service
public class OrderService {
    
    @Autowired
    private DiscoveryClient discoveryClient;
    
    public User getUser(String userId) {
        List<ServiceInstance> instances = 
            discoveryClient.getInstances("user-service");
        
        if (instances.isEmpty()) {
            throw new ServiceUnavailableException();
        }
        
        ServiceInstance instance = instances.get(0);
        String url = instance.getUri() + "/api/users/" + userId;
        
        return restTemplate.getForObject(url, User.class);
    }
}
```

#### Server-Side Discovery (Load Balancer)
```java
@FeignClient(name = "user-service")
public interface UserClient {
    
    @GetMapping("/api/users/{id}")
    User getUserById(@PathVariable String id);
}

@Service
public class OrderService {
    
    @Autowired
    private UserClient userClient;
    
    public Order createOrder(Order order) {
        User user = userClient.getUserById(order.getUserId());
        // Process order
    }
}
```

#### Service Registry
- **Consul**: Service discovery and configuration
- **Eureka**: Netflix service registry
- **Zookeeper**: Distributed coordination
- **etcd**: Distributed key-value store

### 5. API Gateway

#### Responsibilities
- **Routing**: Route requests to appropriate services
- **Authentication**: Verify user identity
- **Authorization**: Check permissions
- **Rate Limiting**: Prevent abuse
- **Load Balancing**: Distribute requests
- **Caching**: Cache responses
- **Request/Response Transformation**: Modify requests/responses
- **Monitoring**: Log and monitor requests

#### Example (Spring Cloud Gateway)
```java
@Configuration
public class GatewayConfig {
    
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            .route("user_route", r -> r
                .path("/api/users/**")
                .filters(f -> f
                    .addRequestHeader("X-Request-Source", "gateway")
                    .circuitBreaker(config -> config
                        .setName("userServiceCircuitBreaker")
                        .setFallbackUri("forward:/fallback/users")
                    )
                )
                .uri("lb://user-service")
            )
            .route("order_route", r -> r
                .path("/api/orders/**")
                .uri("lb://order-service")
            )
            .build();
    }
}
```

### 6. Circuit Breaker Pattern

#### Purpose
- Prevent cascading failures
- Fail fast when service is down
- Automatic recovery

#### Implementation (Resilience4j)
```java
@Service
public class UserService {
    
    @Autowired
    private RestTemplate restTemplate;
    
    @CircuitBreaker(name = "userService", fallbackMethod = "getUserFallback")
    @Retry(name = "userService")
    @RateLimiter(name = "userService")
    public User getUser(String userId) {
        return restTemplate.getForObject(
            "http://user-service/api/users/" + userId,
            User.class
        );
    }
    
    public User getUserFallback(String userId, Exception e) {
        // Return cached or default user
        return new User(userId, "Unknown");
    }
}
```

#### Configuration
```yaml
resilience4j:
  circuitbreaker:
    instances:
      userService:
        slidingWindowSize: 10
        failureRateThreshold: 50
        waitDurationInOpenState: 10000
        permittedNumberOfCallsInHalfOpenState: 3
  retry:
    instances:
      userService:
        maxAttempts: 3
        waitDuration: 1000
  ratelimiter:
    instances:
      userService:
        limitForPeriod: 10
        limitRefreshPeriod: 1000
```

### 7. Data Management

#### Database per Service
- Each service owns its database
- No shared databases
- Loose coupling

#### Saga Pattern
Manage distributed transactions across services

**Choreography-based Saga**
```java
// Order Service
public void createOrder(Order order) {
    order.setStatus("PENDING");
    orderRepository.save(order);
    
    // Publish event
    eventPublisher.publish(new OrderCreatedEvent(order));
}

// Inventory Service
@EventListener
public void onOrderCreated(OrderCreatedEvent event) {
    if (inventoryAvailable(event.getItems())) {
        reserveInventory(event.getItems());
        eventPublisher.publish(new InventoryReservedEvent(event.getOrderId()));
    } else {
        eventPublisher.publish(new InventoryReservationFailedEvent(event.getOrderId()));
    }
}

// Payment Service
@EventListener
public void onInventoryReserved(InventoryReservedEvent event) {
    if (processPayment(event.getOrderId())) {
        eventPublisher.publish(new PaymentSuccessEvent(event.getOrderId()));
    } else {
        eventPublisher.publish(new PaymentFailedEvent(event.getOrderId()));
    }
}
```

**Orchestration-based Saga**
```java
@Service
public class OrderSagaOrchestrator {
    
    public void createOrder(Order order) {
        try {
            // Step 1: Reserve inventory
            inventoryService.reserveInventory(order.getItems());
            
            // Step 2: Process payment
            paymentService.processPayment(order);
            
            // Step 3: Confirm order
            order.setStatus("CONFIRMED");
            orderRepository.save(order);
            
        } catch (Exception e) {
            // Compensating transactions
            inventoryService.releaseInventory(order.getItems());
            paymentService.refund(order);
            
            order.setStatus("FAILED");
            orderRepository.save(order);
        }
    }
}
```

#### CQRS (Command Query Responsibility Segregation)
- Separate models for reads and writes
- Optimize each model independently
- Event sourcing for writes

### 8. Microservices Patterns

#### 1. API Gateway Pattern
Single entry point for clients

#### 2. Service Discovery Pattern
Services register and discover each other

#### 3. Circuit Breaker Pattern
Prevent cascading failures

#### 4. Bulkhead Pattern
Isolate resources for different services

#### 5. Saga Pattern
Manage distributed transactions

#### 6. Event Sourcing Pattern
Store state changes as events

#### 7. CQRS Pattern
Separate read and write models

#### 8. Strangler Pattern
Gradually migrate from monolith to microservices

#### 9. Sidecar Pattern
Auxiliary services alongside main service (e.g., logging, monitoring)

#### 10. Backend for Frontend (BFF) Pattern
Separate backend for each frontend

### 9. Security

#### Authentication & Authorization

**JWT Authentication**
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            // Disable CSRF for stateless JWT authentication
            // Note: For session-based authentication, keep CSRF enabled
            .csrf().disable()
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/public/**").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer()
            .jwt();
        
        return http.build();
    }
}
```

#### Service-to-Service Authentication
- Mutual TLS (mTLS)
- API Keys
- Service Mesh (Istio)

### 10. Monitoring and Observability

#### Distributed Tracing
```java
@Configuration
public class TracingConfig {
    
    @Bean
    public Sampler defaultSampler() {
        return Sampler.ALWAYS_SAMPLE;
    }
}
```

#### Centralized Logging
```yaml
logging:
  level:
    root: INFO
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"
```

#### Metrics
```java
@RestController
public class OrderController {
    
    @Autowired
    private MeterRegistry meterRegistry;
    
    @PostMapping("/orders")
    public Order createOrder(@RequestBody Order order) {
        Timer.Sample sample = Timer.start(meterRegistry);
        
        try {
            Order created = orderService.createOrder(order);
            meterRegistry.counter("orders.created", "status", "success").increment();
            return created;
        } catch (Exception e) {
            meterRegistry.counter("orders.created", "status", "failed").increment();
            throw e;
        } finally {
            sample.stop(meterRegistry.timer("orders.creation.time"));
        }
    }
}
```

## Common Interview Questions

### Basic Questions
1. What are microservices and how are they different from monoliths?
2. What are the advantages and disadvantages of microservices?
3. How do microservices communicate with each other?
4. What is service discovery?
5. What is an API Gateway?

### Intermediate Questions
1. How do you handle distributed transactions in microservices?
2. What is the circuit breaker pattern?
3. How do you implement authentication in microservices?
4. What is the difference between choreography and orchestration in Saga pattern?
5. How do you handle data consistency in microservices?

### Advanced Questions
1. How do you migrate from monolith to microservices?
2. How do you implement distributed tracing?
3. What are the challenges of microservices and how do you address them?
4. How do you design microservices boundaries?
5. How do you handle versioning in microservices?

## Microservices Challenges

### 1. Increased Complexity
- Multiple services to manage
- Distributed system challenges
- **Solution**: Automation, good tooling, monitoring

### 2. Data Consistency
- No ACID transactions across services
- Eventual consistency
- **Solution**: Saga pattern, event sourcing

### 3. Network Latency
- Multiple network calls
- Service dependencies
- **Solution**: Caching, async communication, optimize service boundaries

### 4. Testing Complexity
- Integration testing is complex
- End-to-end testing challenges
- **Solution**: Contract testing, service virtualization

### 5. Deployment Complexity
- Multiple deployments
- Version management
- **Solution**: CI/CD, containerization, orchestration

## Best Practices

1. **Design for Failure**: Implement circuit breakers, retries, timeouts
2. **Decentralize Data**: Database per service
3. **API Gateway**: Single entry point
4. **Service Discovery**: Dynamic service location
5. **Containerization**: Docker for consistency
6. **Orchestration**: Kubernetes for management
7. **CI/CD**: Automate build and deployment
8. **Monitoring**: Centralized logging and monitoring
9. **Security**: Implement proper authentication and authorization
10. **Documentation**: API documentation (Swagger/OpenAPI)

## Tools and Technologies

| Category | Tools |
|----------|-------|
| Communication | REST, gRPC, GraphQL |
| Message Queues | RabbitMQ, Kafka, AWS SQS |
| Service Discovery | Consul, Eureka, Zookeeper |
| API Gateway | Kong, Netflix Zuul, Spring Cloud Gateway |
| Circuit Breaker | Resilience4j, Hystrix |
| Containers | Docker, Podman |
| Orchestration | Kubernetes, Docker Swarm |
| Service Mesh | Istio, Linkerd |
| Monitoring | Prometheus, Grafana, ELK Stack |
| Tracing | Jaeger, Zipkin |
| Configuration | Spring Cloud Config, Consul |

## Resources
- Building Microservices by Sam Newman
- Microservices Patterns by Chris Richardson
- Domain-Driven Design by Eric Evans
- Martin Fowler's Microservices articles
- microservices.io
