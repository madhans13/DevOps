# Kubernetes NodePort and LoadBalancer Services Guide

## Overview

While ClusterIP services handle internal cluster communication, NodePort and LoadBalancer services expose applications to external traffic. This guide covers both service types, their configurations, use cases, and best practices.

## NodePort Service

### What is NodePort?

NodePort is a service type that exposes applications on a static port across all cluster nodes. It makes your service accessible from outside the cluster by opening a specific port (30000-32767) on every node.

### How NodePort Works

1. **Port Allocation**: Kubernetes allocates a port from the NodePort range (30000-32767)
2. **Node Binding**: The port is opened on all cluster nodes
3. **Traffic Routing**: External traffic hitting any node on the NodePort gets forwarded to the service
4. **Load Balancing**: Traffic is distributed across all healthy pods

### NodePort Architecture
```
External Client → Node IP:NodePort → ClusterIP:Port → Pod:TargetPort
```

### Basic NodePort Configuration

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-nodeport-service
spec:
  type: NodePort
  selector:
    app: web
    tier: frontend
  ports:
    - port: 80          # ClusterIP port
      targetPort: 8080  # Pod port
      nodePort: 30080   # External port (optional)
```

### NodePort Configuration Options

#### 1. Automatic Port Assignment
```yaml
apiVersion: v1
kind: Service
metadata:
  name: auto-nodeport-service
spec:
  type: NodePort
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 8080
      # nodePort is automatically assigned
```

#### 2. Specific Port Assignment
```yaml
apiVersion: v1
kind: Service
metadata:
  name: specific-nodeport-service
spec:
  type: NodePort
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 31234  # Must be in range 30000-32767
```

#### 3. Multiple Ports
```yaml
apiVersion: v1
kind: Service
metadata:
  name: multi-port-nodeport
spec:
  type: NodePort
  selector:
    app: web
  ports:
    - name: http
      port: 80
      targetPort: 8080
      nodePort: 30080
    - name: https
      port: 443
      targetPort: 8443
      nodePort: 30443
```

### Complete NodePort Example

```yaml
# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
---
# NodePort Service
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

### Accessing NodePort Services

```bash
# Using any node's IP address
curl http://<node-ip>:30080

# Using cluster node IPs
curl http://192.168.1.100:30080
curl http://192.168.1.101:30080
curl http://192.168.1.102:30080

# All will route to the same service
```

## LoadBalancer Service

### What is LoadBalancer?

LoadBalancer is a service type that provisions an external load balancer (when supported by the cloud provider) to expose services. It automatically creates NodePort and ClusterIP services and configures the external load balancer to route traffic to the NodePort.

### How LoadBalancer Works

1. **Service Creation**: Creates NodePort service automatically
2. **External LB Provisioning**: Cloud provider creates external load balancer
3. **IP Assignment**: External IP is assigned and configured
4. **Traffic Flow**: External LB → NodePort → ClusterIP → Pods

### LoadBalancer Architecture
```
Internet → External Load Balancer → Node:NodePort → ClusterIP → Pods
```

### Basic LoadBalancer Configuration

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-loadbalancer-service
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 8080
```

### LoadBalancer Configuration Examples

#### 1. Basic HTTP Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: http-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
    - name: http
      port: 80
      targetPort: 8080
      protocol: TCP
```

#### 2. HTTPS Service with Multiple Ports
```yaml
apiVersion: v1
kind: Service
metadata:
  name: https-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
    - name: http
      port: 80
      targetPort: 8080
    - name: https
      port: 443
      targetPort: 8443
```

#### 3. LoadBalancer with Specific External IP (if supported)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: static-ip-loadbalancer
spec:
  type: LoadBalancer
  loadBalancerIP: 203.0.113.100  # Pre-allocated static IP
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 8080
```

### Cloud Provider Specific Configurations

#### AWS (ELB/ALB)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: aws-loadbalancer
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: "tcp"
    service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 8080
```

#### Google Cloud (GCE)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: gcp-loadbalancer
  annotations:
    cloud.google.com/load-balancer-type: "External"
    cloud.google.com/backend-config: '{"default": "my-backend-config"}'
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 8080
```

#### Azure
```yaml
apiVersion: v1
kind: Service
metadata:
  name: azure-loadbalancer
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "false"
    service.beta.kubernetes.io/azure-pip-name: "my-public-ip"
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 8080
```

### Complete LoadBalancer Example

```yaml
# Namespace
apiVersion: v1
kind: Namespace
metadata:
  name: production
---
# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: production
spec:
  replicas: 5
  selector:
    matchLabels:
      app: web-app
      version: v2.0
  template:
    metadata:
      labels:
        app: web-app
        version: v2.0
    spec:
      containers:
      - name: web
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
---
# LoadBalancer Service
apiVersion: v1
kind: Service
metadata:
  name: web-app-loadbalancer
  namespace: production
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
spec:
  type: LoadBalancer
  selector:
    app: web-app
    version: v2.0
  ports:
    - name: http
      port: 80
      targetPort: 80
      protocol: TCP
```

## Configuration Parameters Deep Dive

### Port Definitions
```yaml
ports:
  - name: http              # Optional: port name for reference
    port: 80                # Service port (ClusterIP)
    targetPort: 8080        # Pod port (can be name or number)
    nodePort: 30080         # NodePort (30000-32767, NodePort only)
    protocol: TCP           # TCP or UDP
```

### Session Affinity
```yaml
# Sticky sessions based on client IP
spec:
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800  # 3 hours
```

### External Traffic Policy
```yaml
spec:
  externalTrafficPolicy: Local  # or Cluster (default)
  # Local: preserves client IP, no extra hops
  # Cluster: distributes across all nodes
```

### Load Balancer Source Ranges
```yaml
spec:
  loadBalancerSourceRanges:
    - "10.0.0.0/8"      # Allow internal networks
    - "192.168.0.0/16"  # Allow private networks
  # Restricts access to specified CIDR blocks
```

## Advanced Configurations

### 1. Health Check Configurations
```yaml
apiVersion: v1
kind: Service
metadata:
  name: health-check-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-healthcheck-interval: "10"
    service.beta.kubernetes.io/aws-load-balancer-healthcheck-timeout: "5"
    service.beta.kubernetes.io/aws-load-balancer-healthcheck-healthy-threshold: "2"
    service.beta.kubernetes.io/aws-load-balancer-healthcheck-unhealthy-threshold: "3"
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 8080
```

### 2. SSL/TLS Termination
```yaml
apiVersion: v1
kind: Service
metadata:
  name: ssl-terminated-service
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-ssl-cert: "arn:aws:acm:us-east-1:123456789:certificate/12345"
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: "http"
    service.beta.kubernetes.io/aws-load-balancer-ssl-ports: "443"
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
    - name: http
      port: 80
      targetPort: 8080
    - name: https
      port: 443
      targetPort: 8080
```

### 3. Internal Load Balancer
```yaml
apiVersion: v1
kind: Service
metadata:
  name: internal-loadbalancer
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-internal: "true"
    # or for GCP:
    # cloud.google.com/load-balancer-type: "Internal"
spec:
  type: LoadBalancer
  selector:
    app: internal-service
  ports:
    - port: 80
      targetPort: 8080
```

## Comparison: NodePort vs LoadBalancer

| Feature | NodePort | LoadBalancer |
|---------|----------|--------------|
| **External Access** | Manual (Node IP + Port) | Automatic (External IP) |
| **Load Balancing** | Client-side | Provider-managed |
| **Cost** | Free | May incur cloud costs |
| **High Availability** | Manual setup required | Built-in |
| **SSL Termination** | Manual | Provider-managed |
| **Health Checks** | Manual | Automatic |
| **Use Case** | Development/Testing | Production |

## Best Practices

### 1. Security Considerations
```yaml
# Restrict access with source ranges
spec:
  loadBalancerSourceRanges:
    - "10.0.0.0/8"
    - "192.168.0.0/16"

# Use network policies for additional security
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: web-app-netpol
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: frontend
    ports:
    - protocol: TCP
      port: 8080
```

### 2. Resource Management
```yaml
# Set appropriate resource limits
resources:
  requests:
    memory: "256Mi"
    cpu: "200m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

### 3. Health Checks
```yaml
# Always include health checks
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

### 4. Monitoring and Logging
```yaml
# Add monitoring labels
metadata:
  labels:
    app: web-service
    version: v1.0.0
    component: frontend
    monitoring: "true"
```

## Troubleshooting

### Common NodePort Issues

#### 1. Port Not Accessible
```bash
# Check if service exists
kubectl get svc

# Verify NodePort is allocated
kubectl describe svc <service-name>

# Check if pods are running
kubectl get pods -l app=<app-label>

# Test from within cluster
kubectl exec -it <pod-name> -- curl http://<service-name>:<port>
```

#### 2. Firewall Issues
```bash
# Check node firewall rules
sudo iptables -L -n | grep <nodeport>

# For cloud providers, check security groups
# AWS: Security Groups
# GCP: Firewall Rules
# Azure: Network Security Groups
```

### Common LoadBalancer Issues

#### 1. External IP Pending
```bash
# Check service status
kubectl get svc <service-name>

# Look for events and errors
kubectl describe svc <service-name>

# Check cloud provider quotas and permissions
# Verify service account has necessary permissions
```

#### 2. Load Balancer Health Check Failures
```bash
# Check pod health
kubectl get pods -l app=<app-label>

# Verify readiness probes
kubectl describe pod <pod-name>

# Check service endpoints
kubectl get endpoints <service-name>
```

### Debugging Commands
```bash
# View service details
kubectl describe service <service-name>

# Check service endpoints
kubectl get endpoints <service-name> -o yaml

# View events
kubectl get events --sort-by='.lastTimestamp'

# Test connectivity
kubectl run -it --rm debug --image=busybox --restart=Never -- sh
```

## Migration Strategies

### From NodePort to LoadBalancer
```bash
# Update existing service
kubectl patch service <service-name> -p '{"spec":{"type":"LoadBalancer"}}'

# Or update YAML and apply
kubectl apply -f updated-service.yaml
```

### Blue-Green Deployment with Services
```yaml
# Blue service (current)
apiVersion: v1
kind: Service
metadata:
  name: app-blue
spec:
  type: LoadBalancer
  selector:
    app: myapp
    version: blue
  ports:
    - port: 80
      targetPort: 8080
---
# Green service (new)
apiVersion: v1
kind: Service
metadata:
  name: app-green
spec:
  type: LoadBalancer
  selector:
    app: myapp
    version: green
  ports:
    - port: 80
      targetPort: 8080
```

## Summary

### When to Use NodePort:
- Development and testing environments
- Small applications with known node IPs
- Cost-sensitive deployments
- Custom load balancing solutions

### When to Use LoadBalancer:
- Production environments
- High availability requirements
- Automatic scaling needs
- Cloud-native applications
- SSL/TLS termination requirements

Both service types provide external access to your Kubernetes applications, with LoadBalancer offering more enterprise features and automation, while NodePort provides simplicity and cost-effectiveness for smaller deployments.