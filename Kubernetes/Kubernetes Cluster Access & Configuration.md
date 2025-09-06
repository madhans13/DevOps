# Kubernetes Cluster Access & Configuration - Study Notes

## 🌐 Overview

In Kubernetes, **access & configuration** define **how users and tools connect to the cluster**, authenticate, and select the right environment (dev, test, prod).

## 🔹 1. Kubeconfig

### What is Kubeconfig?
- A **configuration file** used by `kubectl` (default location: `~/.kube/config`)
- Stores **clusters**, **users**, and **contexts**
- YAML format configuration that defines cluster access

### Components of Kubeconfig:

#### **Clusters Section**
- **API server address**: Where to connect
- **Certificate Authority**: How to verify server identity
- **TLS settings**: Security configuration

#### **Users Section** 
- **Authentication methods**: Certificates, tokens, cloud IAM
- **Credentials**: How to prove your identity
- **Client certificates**: Mutual TLS authentication

#### **Contexts Section**
- **Cluster + User + Namespace combination**
- **Default namespace**: Where commands run by default
- **Active context**: Currently selected environment

### 📌 Example Kubeconfig Structure:

```yaml
apiVersion: v1
kind: Config
clusters:
- name: dev-cluster
  cluster:
    server: https://192.168.1.100:6443
    certificate-authority: /path/to/ca.crt
    # Alternative: certificate-authority-data: <base64-encoded-ca>
- name: prod-cluster
  cluster:
    server: https://prod.k8s.company.com:6443
    certificate-authority: /etc/ssl/k8s-ca.crt

users:
- name: dev-user
  user:
    client-certificate: /path/to/dev.crt
    client-key: /path/to/dev.key
- name: prod-user
  user:
    token: eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
- name: cloud-user
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      command: aws
      args:
      - eks
      - get-token
      - --cluster-name
      - prod-cluster

contexts:
- name: dev-context
  context:
    cluster: dev-cluster
    user: dev-user
    namespace: development
- name: prod-context
  context:
    cluster: prod-cluster
    user: prod-user
    namespace: production

current-context: dev-context
```

### Kubeconfig Management Commands:

```bash
# View current kubeconfig
kubectl config view

# View raw kubeconfig (with secrets)
kubectl config view --raw

# Set kubeconfig file location
export KUBECONFIG=/path/to/custom/config

# Merge multiple kubeconfig files
export KUBECONFIG=~/.kube/config:~/.kube/config-dev:~/.kube/config-prod
kubectl config view --merge --flatten > ~/.kube/merged-config

# Set cluster info
kubectl config set-cluster my-cluster \
  --server=https://1.2.3.4:6443 \
  --certificate-authority=/path/to/ca.crt

# Set user credentials
kubectl config set-credentials my-user \
  --client-certificate=/path/to/cert.crt \
  --client-key=/path/to/cert.key

# Create context
kubectl config set-context my-context \
  --cluster=my-cluster \
  --user=my-user \
  --namespace=my-namespace
```

### Authentication Methods in Kubeconfig:

#### **1. Client Certificates (Mutual TLS)**
```yaml
user:
  client-certificate: /path/to/client.crt
  client-key: /path/to/client.key
  # Or embedded:
  client-certificate-data: <base64-cert>
  client-key-data: <base64-key>
```

#### **2. Bearer Tokens**
```yaml
user:
  token: eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
```

#### **3. Basic Authentication** (Deprecated)
```yaml
user:
  username: admin
  password: secret123
```

#### **4. Exec Plugins** (Cloud IAM integration)
```yaml
user:
  exec:
    apiVersion: client.authentication.k8s.io/v1beta1
    command: aws
    args: ["eks", "get-token", "--cluster-name", "my-cluster"]
```

#### **5. OIDC (OpenID Connect)**
```yaml
user:
  auth-provider:
    name: oidc
    config:
      client-id: kubernetes
      client-secret: secret
      id-token: eyJhbGciOiJSUzI1NiIs...
      idp-issuer-url: https://accounts.google.com
```

---

## 🔹 2. Contexts

### What are Contexts?
- A **shortcut** that ties **cluster + user + namespace** together
- Allows switching between environments easily
- Think of it as a **profile** for your cluster access

### Context Benefits:
- **Environment Switching**: Easily switch between dev/test/prod
- **Namespace Default**: Set default namespace per environment
- **User Separation**: Different users for different environments
- **Reduced Errors**: Less chance of running commands in wrong environment

### 📌 Context Commands:

```bash
# List all contexts
kubectl config get-contexts

# Show current context
kubectl config current-context

# Switch to different context
kubectl config use-context prod-context

# Create new context
kubectl config set-context staging \
  --cluster=staging-cluster \
  --user=staging-user \
  --namespace=staging

# Delete context
kubectl config delete-context old-context

# Rename context
kubectl config rename-context old-name new-name

# Set default namespace for context
kubectl config set-context dev-context --namespace=development
```

### Context Output Example:
```bash
$ kubectl config get-contexts
CURRENT   NAME           CLUSTER        AUTHINFO       NAMESPACE
          dev-context    dev-cluster    dev-user       development
*         prod-context   prod-cluster   prod-user      production
          test-context   test-cluster   test-user      testing
```

### Temporary Context Override:
```bash
# Use different context for single command
kubectl get pods --context=dev-context

# Use different namespace for single command
kubectl get pods -n kube-system

# Use different kubeconfig file
kubectl get pods --kubeconfig=/path/to/other/config
```

---

## 🔹 3. Namespaces

### What are Namespaces?
- Provide **logical isolation** inside a cluster
- Used to **organize workloads** and **separate environments**
- Every Pod, Service, and resource lives in a namespace
- **Virtual clusters** within a physical cluster

### Default Namespaces:
- **`default`** → For general workloads when no namespace specified
- **`kube-system`** → For system components (API server, scheduler, etc.)
- **`kube-public`** → For public info (readable without authentication)
- **`kube-node-lease`** → For node heartbeats and health checks

### Namespace Use Cases:

#### **1. Environment Separation**
```yaml
# development namespace
apiVersion: v1
kind: Namespace
metadata:
  name: development
  labels:
    env: dev
---
# production namespace
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    env: prod
```

#### **2. Team Isolation**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: team-frontend
  labels:
    team: frontend
---
apiVersion: v1
kind: Namespace
metadata:
  name: team-backend
  labels:
    team: backend
```

#### **3. Application Separation**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: app-web
---
apiVersion: v1
kind: Namespace
metadata:
  name: app-api
---
apiVersion: v1
kind: Namespace
metadata:
  name: app-database
```

### 📌 Namespace Commands:

```bash
# List all namespaces
kubectl get namespaces
kubectl get ns  # Short form

# Create namespace
kubectl create namespace dev
kubectl create ns staging  # Short form

# Create from YAML
kubectl apply -f namespace.yaml

# Delete namespace (deletes all resources inside!)
kubectl delete namespace dev

# Describe namespace
kubectl describe namespace kube-system

# Get resources in specific namespace
kubectl get pods -n kube-system
kubectl get all -n development

# Set default namespace for current context
kubectl config set-context --current --namespace=development

# Get resources from all namespaces
kubectl get pods --all-namespaces
kubectl get pods -A  # Short form
```

### Resource Naming in Namespaces:
- **Within namespace**: Resource names must be unique
- **Across namespaces**: Same names allowed in different namespaces
- **Fully Qualified Name**: `<resource-name>.<namespace>.svc.cluster.local`

### Namespace Scope:
#### **Namespaced Resources:**
- Pods, Services, Deployments
- ConfigMaps, Secrets
- PersistentVolumeClaims
- Ingress, NetworkPolicies

#### **Cluster-Scoped Resources:**
- Nodes, PersistentVolumes
- StorageClasses, ClusterRoles
- Namespaces themselves

```bash
# Check if resource is namespaced
kubectl api-resources --namespaced=true   # Namespaced resources
kubectl api-resources --namespaced=false  # Cluster-scoped resources
```

---

## 🔹 4. Authentication & Authorization

### Authentication (AuthN) - "Who are you?"

#### **Authentication Methods:**

##### **1. X509 Client Certificates**
```bash
# Generate client certificate
openssl genrsa -out user.key 2048
openssl req -new -key user.key -out user.csr -subj "/CN=username/O=group"
# Have cluster CA sign the CSR
kubectl certificate approve user-csr
```

##### **2. Service Account Tokens**
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
  namespace: default
---
# Token automatically generated
kubectl get secret $(kubectl get sa my-service-account -o jsonpath='{.secrets[0].name}')
```

##### **3. OIDC Tokens** (Google, Azure AD, etc.)
```bash
# Configure OIDC in API server
--oidc-issuer-url=https://accounts.google.com
--oidc-client-id=kubernetes
--oidc-username-claim=email
--oidc-groups-claim=groups
```

##### **4. Cloud Provider IAM**
```bash
# AWS IAM integration
aws eks update-kubeconfig --name my-cluster
# Uses AWS IAM roles for authentication
```

### Authorization (AuthZ) - "What can you do?"

#### **Authorization Modes:**

##### **1. RBAC (Role-Based Access Control)** - Recommended
```yaml
# Role - namespace scoped
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: development
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
---
# ClusterRole - cluster scoped
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list"]
---
# RoleBinding - ties Role to user/group
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: development
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

##### **2. ABAC (Attribute-Based Access Control)**
```json
{
  "apiVersion": "abac.authorization.kubernetes.io/v1beta1",
  "kind": "Policy",
  "spec": {
    "user": "alice",
    "namespace": "development",
    "resource": "pods",
    "verb": "get"
  }
}
```

##### **3. Webhook Authorization**
```yaml
# API server configuration
--authorization-mode=Webhook
--authorization-webhook-config-file=/etc/kubernetes/webhook-config.yaml
```

##### **4. Node Authorization**
- Authorizes kubelet requests
- Ensures nodes can only access their own pods/secrets
- Built-in security for node-level operations

#### **RBAC Best Practices:**

```bash
# Check current user permissions
kubectl auth can-i create deployments
kubectl auth can-i create deployments --as=jane
kubectl auth can-i create deployments --as=jane -n development

# List user permissions
kubectl auth can-i --list
kubectl auth can-i --list --as=jane -n development

# Debug RBAC issues
kubectl get rolebindings -A
kubectl get clusterrolebindings
kubectl describe rolebinding <binding-name> -n <namespace>
```

### Admission Controllers - "Extra validation before creation"

#### **Common Admission Controllers:**

##### **1. ResourceQuota**
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: development
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    persistentvolumeclaims: "10"
```

##### **2. LimitRange**
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
  namespace: development
spec:
  limits:
  - default:
      memory: "512Mi"
      cpu: "500m"
    defaultRequest:
      memory: "256Mi"
      cpu: "100m"
    type: Container
```

##### **3. NetworkPolicy**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: development
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

##### **4. PodSecurityPolicy** (Deprecated - use Pod Security Standards)
```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  privileged: false
  runAsUser:
    rule: MustRunAsNonRoot
  fsGroup:
    rule: RunAsAny
```

---

## ✅ Summary

### **Core Concepts:**
- **Kubeconfig** → Stores clusters, users, contexts for cluster access
- **Contexts** → Switch between clusters/environments easily
- **Namespaces** → Organize workloads logically within cluster
- **AuthN/AuthZ** → Control who can access and what they can do

### **Best Practices:**

#### **Kubeconfig Management:**
- Use separate contexts for different environments
- Keep production credentials secure and separate
- Use cloud provider IAM integration when possible
- Regularly rotate certificates and tokens

#### **Namespace Organization:**
- Use descriptive namespace names (team-app-env pattern)
- Apply resource quotas to prevent resource exhaustion
- Use labels for consistent organization
- Don't use default namespace for applications

#### **Security:**
- Follow principle of least privilege with RBAC
- Use service accounts for applications, not user accounts
- Enable audit logging for compliance
- Regularly review and clean up unused permissions

---

## 🎯 Common Interview Questions

**Q: How do you switch between different Kubernetes clusters?**
```bash
# Use contexts in kubeconfig
kubectl config get-contexts
kubectl config use-context prod-cluster
kubectl config current-context
```

**Q: What's the difference between Role and ClusterRole?**
- **Role**: Namespace-scoped permissions (pods, services in specific namespace)
- **ClusterRole**: Cluster-scoped permissions (nodes, persistent volumes) or namespaced resources across all namespaces

**Q: How do you troubleshoot "Forbidden" errors?**
```bash
# Check current permissions
kubectl auth can-i create pods
kubectl auth can-i create pods --as=serviceaccount:default:my-sa

# Check RBAC bindings
kubectl get rolebindings,clusterrolebindings -A | grep username
kubectl describe rolebinding <binding-name> -n <namespace>
```

**Q: What happens if you delete a namespace?**
- All resources within the namespace are deleted
- Running pods are terminated
- Persistent volumes may be retained (depends on reclaim policy)
- Use finalizers to prevent accidental deletion

**Q: How do you secure access to a Kubernetes cluster?**
1. **Network Security**: Private API server, network policies
2. **Authentication**: Strong auth methods (OIDC, certificates)
3. **Authorization**: RBAC with least privilege
4. **Admission Control**: Resource quotas, security policies
5. **Audit Logging**: Track all API server access
6. **Regular Updates**: Keep cluster and components updated