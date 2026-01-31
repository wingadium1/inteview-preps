# Containers Interview Preparation

## Docker

### 1. Docker Basics

#### Core Concepts
- **Image**: Read-only template with application and dependencies
- **Container**: Running instance of an image
- **Dockerfile**: Text file with instructions to build image
- **Registry**: Store and distribute images (Docker Hub, ECR, GCR)
- **Volume**: Persistent data storage
- **Network**: Container networking

#### Docker Architecture
- **Docker Client**: Command-line interface (docker CLI)
- **Docker Daemon**: Background service (dockerd)
- **Docker Registry**: Image repository
- **Docker Objects**: Images, containers, networks, volumes

### 2. Dockerfile

#### Basic Dockerfile
```dockerfile
# Use official base image
FROM openjdk:11-jre-slim

# Set working directory
WORKDIR /app

# Copy application files
COPY target/myapp.jar /app/myapp.jar

# Expose port
EXPOSE 8080

# Set environment variables
ENV JAVA_OPTS="-Xmx512m"

# Run application
ENTRYPOINT ["java", "-jar", "myapp.jar"]
```

#### Multi-stage Build
```dockerfile
# Build stage
FROM maven:3.8-openjdk-11 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# Runtime stage
FROM openjdk:11-jre-slim
WORKDIR /app
COPY --from=build /app/target/myapp.jar .
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "myapp.jar"]
```

### 3. Docker Commands

#### Image Commands
```bash
# Build image
docker build -t myapp:1.0 .

# List images
docker images

# Remove image
docker rmi myapp:1.0

# Pull image from registry
docker pull nginx:latest

# Push image to registry
docker push myregistry/myapp:1.0

# Tag image
docker tag myapp:1.0 myregistry/myapp:1.0
```

#### Container Commands
```bash
# Run container
docker run -d -p 8080:8080 --name myapp myapp:1.0

# List running containers
docker ps

# List all containers
docker ps -a

# Stop container
docker stop myapp

# Start container
docker start myapp

# Remove container
docker rm myapp

# View logs
docker logs myapp

# Execute command in container
docker exec -it myapp /bin/bash

# Inspect container
docker inspect myapp

# View resource usage
docker stats
```

#### Volume Commands
```bash
# Create volume
docker volume create mydata

# List volumes
docker volume ls

# Remove volume
docker volume rm mydata

# Mount volume
docker run -v mydata:/data myapp:1.0
```

#### Network Commands
```bash
# Create network
docker network create mynetwork

# List networks
docker network ls

# Connect container to network
docker network connect mynetwork myapp

# Inspect network
docker network inspect mynetwork
```

### 4. Docker Compose

#### docker-compose.yml
```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgresql://db:5432/mydb
    depends_on:
      - db
    networks:
      - app-network
    volumes:
      - ./logs:/app/logs
    restart: unless-stopped
    
  db:
    image: postgres:13
    environment:
      - POSTGRES_DB=mydb
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network
    
  redis:
    image: redis:6-alpine
    networks:
      - app-network

volumes:
  postgres-data:

networks:
  app-network:
    driver: bridge
```

#### Docker Compose Commands
```bash
# Start services
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs -f

# Scale service
docker-compose up -d --scale web=3

# Build images
docker-compose build

# List services
docker-compose ps
```

## Kubernetes (K8s)

### 1. Kubernetes Architecture

#### Control Plane Components
- **kube-apiserver**: API server
- **etcd**: Key-value store
- **kube-scheduler**: Schedule pods
- **kube-controller-manager**: Manage controllers
- **cloud-controller-manager**: Cloud provider integration

#### Node Components
- **kubelet**: Agent on each node
- **kube-proxy**: Network proxy
- **Container Runtime**: Docker, containerd, CRI-O

### 2. Kubernetes Objects

#### Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp
    image: myapp:1.0
    ports:
    - containerPort: 8080
    env:
    - name: DATABASE_URL
      value: "postgresql://db:5432/mydb"
    resources:
      requests:
        memory: "256Mi"
        cpu: "250m"
      limits:
        memory: "512Mi"
        cpu: "500m"
```

#### Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:1.0
        ports:
        - containerPort: 8080
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```

#### Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

#### ConfigMap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
data:
  database_url: "postgresql://db:5432/mydb"
  log_level: "info"
```

#### Secret
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secret
type: Opaque
data:
  username: YWRtaW4=  # base64 encoded
  password: cGFzc3dvcmQ=  # base64 encoded
```

#### Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 80
```

### 3. kubectl Commands

```bash
# Get resources
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get nodes

# Describe resource
kubectl describe pod myapp-pod

# Create resource from file
kubectl apply -f deployment.yaml

# Delete resource
kubectl delete pod myapp-pod
kubectl delete -f deployment.yaml

# Logs
kubectl logs myapp-pod
kubectl logs -f myapp-pod  # Follow logs

# Execute command in pod
kubectl exec -it myapp-pod -- /bin/bash

# Port forwarding
kubectl port-forward pod/myapp-pod 8080:8080

# Scale deployment
kubectl scale deployment myapp-deployment --replicas=5

# Rollout
kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment
kubectl rollout undo deployment/myapp-deployment

# ConfigMap and Secret
kubectl create configmap myapp-config --from-file=config.properties
kubectl create secret generic myapp-secret --from-literal=password=secret123

# Namespace
kubectl create namespace dev
kubectl get pods -n dev
```

### 4. Advanced Kubernetes Concepts

#### StatefulSet
- Stable network identities
- Stable persistent storage
- Ordered deployment and scaling
- Use cases: Databases, message queues

#### DaemonSet
- Run pod on every node
- Use cases: Monitoring agents, log collectors

#### Job and CronJob
- Run-to-completion tasks
- Scheduled tasks

#### Horizontal Pod Autoscaler
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

## Common Interview Questions

### Docker Questions
1. What is the difference between Docker image and container?
2. Explain Docker architecture
3. What is a Dockerfile and how do you optimize it?
4. What is the difference between CMD and ENTRYPOINT?
5. How do you manage data persistence in Docker?

### Kubernetes Questions
1. What is Kubernetes and why is it needed?
2. Explain Kubernetes architecture
3. What is the difference between Pod and Deployment?
4. How does Service discovery work in Kubernetes?
5. What are ConfigMaps and Secrets?

### Advanced Questions
1. How do you implement rolling updates in Kubernetes?
2. Explain the difference between StatefulSet and Deployment
3. How do you handle secrets securely in Kubernetes?
4. What is the difference between LoadBalancer, NodePort, and ClusterIP?
5. How do you debug a failing pod in Kubernetes?

## Best Practices

### Docker Best Practices
1. Use official base images
2. Use multi-stage builds
3. Minimize image layers
4. Don't run as root user
5. Use .dockerignore file
6. Scan images for vulnerabilities
7. Use specific image tags (not latest)
8. Keep images small

### Kubernetes Best Practices
1. Use namespaces for isolation
2. Set resource requests and limits
3. Use liveness and readiness probes
4. Use ConfigMaps and Secrets
5. Implement RBAC
6. Use Ingress for routing
7. Monitor and log everything
8. Use Helm for package management
9. Implement pod security policies
10. Use horizontal pod autoscaling

## Resources
- Docker Documentation
- Kubernetes Documentation
- Kubernetes in Action book
- Docker Deep Dive book
- Play with Docker/Kubernetes
