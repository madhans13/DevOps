# Kubernetes Cluster Resource Organization - Study Notes

## 📦 Overview

Once you connect to a Kubernetes cluster, you need a way to **organize, isolate, and control resources** like Pods, Services, and ConfigMaps. Kubernetes provides multiple mechanisms for resource organization.

---

## 🔹 1. Namespaces

### Purpose
- **Logical partitioning** of resources within a cluster
- Useful for **multi-tenant clusters** or separating environments (dev/test/prod)
- Resources in different namespaces are isolated from each other unless explicitly shared
- Some resources are **namespaced** (Pods, Services, ConfigMaps) and some are **cluster-wide** (Nodes, PersistentVolumes)

### Key Characteristics
- **Virtual clusters** within a physical cluster
- **DNS isolation**: Services get DNS names like `service-name.namespace.svc.cluster.local`
- **Network policies** can control traffic between namespaces
- **Resource quotas** can be applied per namespace

### Common Use Cases
- **Environment separation**: dev, staging, production
- **Team isolation**: team-a, team-b, shared-services
- **Application separation**: frontend, backend, database
- **Customer isolation**: customer-1, customer-2 (multi-tenancy)

### 📌 Namespace Commands

```bash
# Create namespace
kubectl create namespace dev
kubectl create ns staging  # Short form

# List namespaces
kubectl get namespaces
kubectl get ns

# Get resources in specific namespace
kubectl get pods -n dev
kubectl get all -n kube-system

# Set default namespace for current context
kubectl config set-context --current --namespace=dev

# Delete namespace (WARNING: Deletes all resources inside!)
kubectl delete namespace test

# Describe namespace
kubectl describe namespace dev
```

### Namespace YAML Example

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: development
  labels:
    env: dev
    team: backend
  annotations:
    description: "Development environment for backend team"
```

### Resource Scope
```bash
# Check which resources are namespaced
kubectl api-resources --namespaced=true

# Check cluster-scoped resources
kubectl api-resources --namespaced=false
```

---

## 🔹 2. Labels & Selectors

### Labels
- **Key-value pairs** attached to objects (e.g., `app=frontend`)
- Used for **organization** and **selection** of resources
- Can be added at creation time or later
- **Maximum 63 characters** for keys and values

### Selectors
- **Queries** to filter resources based on labels
- Controllers (ReplicaSets, Deployments, Services) use selectors to **group resources**
- Two types: **equality-based** and **set-based**

### 📌 Label Examples

```yaml
# Pod with labels
apiVersion: v1
kind: Pod
metadata:
  name: frontend-pod
  labels:
    app: frontend
    tier: web
    environment: production
    version: v1.2.3
spec:
  containers:
  - name: nginx
    image: nginx:1.21
```

### Label Management Commands

```bash
# Add label to existing resource
kubectl label pod frontend-pod release=stable

# Update existing label
kubectl label pod frontend-pod version=v1.2.4 --overwrite

# Remove label
kubectl label pod frontend-pod version-

# View labels
kubectl get pods --show-labels
kubectl describe pod frontend-pod

# Filter by labels (equality-based)
kubectl get pods -l app=frontend
kubectl get pods -l environment=production
kubectl get pods -l app=frontend,tier=web

# Filter by labels (set-based)
kubectl get pods -l 'environment in (production,staging)'
kubectl get pods -l 'tier notin (cache,proxy)'
kubectl get pods -l version
kubectl get pods -l '!version'
```

### Selector in Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend
    tier: web
  ports:
  - port: 80
    targetPort: 8080
```

### Label Best Practices
- **Use consistent naming**: `app`, `version`, `environment`, `tier`
- **Include metadata**: release version, build number, team
- **Avoid sensitive data**: Don't put passwords or keys in labels
- **Use semantic versioning**: v1.2.3, not just latest

---

## 🔹 3. Annotations

### Purpose
- Store **non-identifying metadata** (like version info, links, build SHA, documentation pointers)
- Unlike labels, they **cannot** be used for selection
- Can store **larger values** than labels (up to 256KB)
- Used by **tools and libraries** for additional information

### Common Use Cases
- **Build information**: commit SHA, build number, CI/CD pipeline info
- **Documentation**: links to runbooks, API docs, dashboards
- **Tool metadata**: deployment tools, monitoring configs
- **Change tracking**: last update time, change reason

### 📌 Annotation Examples

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
  annotations:
    description: "Frontend web application"
    build-version: "1.0.3"
    git-commit: "a1b2c3d4e5f6"
    maintainer: "platform-team@company.com"
    documentation: "https://wiki.company.com/frontend"
    deployment.kubernetes.io/revision: "3"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment"...}
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
        prometheus.io/path: "/metrics"
    spec:
      containers:
      - name: frontend
        image: nginx:1.21
```

### Annotation Management Commands

```bash
# Add annotation
kubectl annotate pod frontend-pod description="Main frontend pod"

# Update annotation
kubectl annotate pod frontend-pod build-version="1.0.4" --overwrite

# Remove annotation
kubectl annotate pod frontend-pod build-version-

# View annotations
kubectl describe pod frontend-pod
kubectl get pod frontend-pod -o yaml | grep -A 10 annotations:
```

---

## 🔹 4. Resource Quotas & Limits

### ResourceQuota
- Limits **total resources per namespace** (CPU, memory, object count)
- Prevents any single namespace from consuming all cluster resources
- Enforced by **admission controllers**

### LimitRange
- Sets **default/min/max values** for individual containers/pods
- Applied to **new resources** created in the namespace
- Ensures consistent resource allocation

### 📌 ResourceQuota Example

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: dev
spec:
  hard:
    # Compute resources
    requests.cpu: "4"
    requests.memory: "8Gi"
    limits.cpu: "8"
    limits.memory: "16Gi"
    
    # Storage resources
    persistentvolumeclaims: "4"
    requests.storage: "100Gi"
    
    # Object counts
    pods: "10"
    services: "5"
    secrets: "10"
    configmaps: "10"
    replicationcontrollers: "2"
```

### LimitRange Example

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: dev-limits
  namespace: dev
spec:
  limits:
  - default:  # Default limits (if not specified)
      memory: "512Mi"
      cpu: "500m"
    defaultRequest:  # Default requests (if not specified)
      memory: "256Mi"
      cpu: "100m"
    max:  # Maximum allowed
      memory: "1Gi"
      cpu: "1"
    min:  # Minimum required
      memory: "128Mi"
      cpu: "50m"
    type: Container
  - max:  # Pod-level limits
      memory: "2Gi"
      cpu: "2"
    type: Pod
```

### Resource Management Commands

```bash
# Create resource quota
kubectl apply -f resource-quota.yaml

# View resource quotas
kubectl get resourcequotas -n dev
kubectl describe resourcequota dev-quota -n dev

# Check resource usage
kubectl top pods -n dev
kubectl top nodes

# Create limit range
kubectl apply -f limit-range.yaml

# View limit ranges
kubectl get limitranges -n dev
kubectl describe limitrange dev-limits -n dev
```

---

## 🔹 5. RBAC (Role-Based Access Control)

### Purpose
- Controls **who can access what** in the cluster
- **Fine-grained permissions** for users and service accounts
- **Principle of least privilege** - grant minimal required access

### Components
- **Role/ClusterRole**: Define **permissions** (what can be done)
- **RoleBinding/ClusterRoleBinding**: Assign permissions to **subjects** (who can do it)
- **Subjects**: Users, Groups, ServiceAccounts

### 📌 RBAC Examples

#### Role (Namespace-scoped)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: dev-reader
rules:
- apiGroups: [""]  # Core API group
  resources: ["pods", "services", "configmaps"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list"]
```

#### ClusterRole (Cluster-scoped)

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["metrics.k8s.io"]
  resources: ["nodes", "pods"]
  verbs: ["get", "list"]
```

#### RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-readers
  namespace: dev
subjects:
- kind: User
  name: jane.doe@company.com
  apiGroup: rbac.authorization.k8s.io
- kind: Group
  name: dev-team
  apiGroup: rbac.authorization.k8s.io
- kind: ServiceAccount
  name: dev-service-account
  namespace: dev
roleRef:
  kind: Role
  name: dev-reader
  apiGroup: rbac.authorization.k8s.io
```

#### ClusterRoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-readers
subjects:
- kind: User
  name: admin@company.com
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

### RBAC Management Commands

```bash
# Check current permissions
kubectl auth can-i create pods
kubectl auth can-i create pods -n dev
kubectl auth can-i create nodes

# Check permissions for specific user
kubectl auth can-i get pods --as=jane.doe@company.com
kubectl auth can-i create deployments --as=jane.doe@company.com -n dev

# List all permissions for current user
kubectl auth can-i --list
kubectl auth can-i --list -n dev

# View RBAC resources
kubectl get roles,rolebindings -n dev
kubectl get clusterroles,clusterrolebindings

# Describe RBAC resources
kubectl describe role dev-reader -n dev
kubectl describe rolebinding dev-readers -n dev
```

### RBAC Verbs
- **get**: Read a specific resource
- **list**: Read multiple resources
- **watch**: Watch for changes
- **create**: Create new resources
- **update**: Modify existing resources
- **patch**: Apply partial updates
- **delete**: Remove resources
- **deletecollection**: Remove multiple resources

---

## ✅ Summary

### Core Organization Mechanisms
- **Namespaces** → Logical separation and isolation
- **Labels & Selectors** → Grouping and filtering resources
- **Annotations** → Metadata for humans and tools
- **Resource Quotas & Limits** → Prevent resource abuse
- **RBAC** → Fine-grained access control

### Best Practices

#### Namespace Organization
```bash
# Environment-based
kubectl create ns development
kubectl create ns staging  
kubectl create ns production

# Team-based
kubectl create ns team-frontend
kubectl create ns team-backend
kubectl create ns shared-services
```

#### Label Standards
```yaml
metadata:
  labels:
    app: frontend                    # Application name
    version: v1.2.3                 # Version
    environment: production          # Environment
    tier: web                       # Application tier
    component: nginx                # Component type
    part-of: ecommerce-platform     # Larger application
    managed-by: helm                # Management tool
```

#### Resource Management
```bash
# Apply quotas to all non-system namespaces
kubectl apply -f resource-quota.yaml -n development
kubectl apply -f limit-range.yaml -n development

# Monitor resource usage
kubectl top pods --all-namespaces
kubectl describe quota --all-namespaces
```

#### RBAC Security
- **Principle of least privilege**: Grant minimal required permissions
- **Regular audits**: Review permissions periodically
- **Use groups**: Assign roles to groups, not individual users
- **Service accounts**: Use dedicated service accounts for applications

---

## 🎯 Common Interview Questions

**Q: What's the difference between labels and annotations?**
- **Labels**: Used for selection and grouping, limited to 63 chars, key-value pairs
- **Annotations**: Metadata only, cannot select, up to 256KB, used by tools

**Q: How do you prevent a namespace from using too many resources?**
```yaml
# ResourceQuota limits total namespace consumption
apiVersion: v1
kind: ResourceQuota
metadata:
  name: namespace-quota
spec:
  hard:
    requests.cpu: "4"
    requests.memory: "8Gi"
    pods: "10"
```

**Q: What happens when you delete a namespace?**
- All namespaced resources inside are deleted
- Cluster-scoped resources are unaffected
- Persistent volumes may be retained (depends on reclaim policy)
- Finalizers can prevent deletion until cleanup is complete

**Q: How do you find all pods with a specific label?**
```bash
kubectl get pods -l app=frontend --all-namespaces
kubectl get pods -l 'environment in (dev,staging)' -A
```

**Q: What's the difference between Role and ClusterRole?**
- **Role**: Namespace-scoped permissions (pods, services in specific namespace)
- **ClusterRole**: Cluster-scoped permissions (nodes, persistent volumes) or cross-namespace access