# Kubernetes ServiceAccount and RBAC Guide

## 1. ServiceAccount Basics

**What it is:**
- An identity for Pods inside a Kubernetes cluster
- Like a "username + password" for Pods to talk to the Kubernetes API

**Default behavior:**
- If no SA is mentioned → Pod uses the default ServiceAccount in its namespace
- Token + CA cert are auto-mounted in `/var/run/secrets/kubernetes.io/serviceaccount/`

**Why we need it:**
- For Pods to securely access Kubernetes API or other services
- Example: Monitoring agents, GitOps tools, custom controllers

## 2. RBAC (Role-Based Access Control)

RBAC controls who can do what:
- **Subjects (Who?)**: User, Group, ServiceAccount
- **Roles/ClusterRoles (What?)**: Define allowed actions (verbs)
- **RoleBindings/ClusterRoleBindings (Who gets what?)**: Link Roles to Subjects

## 3. Key RBAC Resources

### Role (namespace-scoped)
Defines permissions inside one namespace.

```yaml
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

### RoleBinding
Binds a Role to a User/SA. Grants permissions inside a namespace.

```yaml
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: ServiceAccount
  name: my-app-sa
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### ClusterRole (cluster-wide)
Permissions across all namespaces.

```yaml
kind: ClusterRole
metadata:
  name: cluster-pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

### ClusterRoleBinding
Bind ClusterRole → User/SA → permissions across cluster.

```yaml
kind: ClusterRoleBinding
metadata:
  name: cluster-read-pods
subjects:
- kind: ServiceAccount
  name: my-admin-sa
  namespace: default
roleRef:
  kind: ClusterRole
  name: cluster-pod-reader
  apiGroup: rbac.authorization.k8s.io
```

## 4. Real-World Analogy (Office Example)

- **Cluster** = Office Building
- **ServiceAccount** = Employee ID Card
- **Role** = Job Description (permissions in one department)
- **ClusterRole** = Company-wide Job Description (all departments)
- **RoleBinding** = Assign job to employee in one department
- **ClusterRoleBinding** = Assign company-wide role to employee

## 5. Interview Q&A

**Q: What's the difference between Role and ClusterRole?**
- Role → Namespace scoped
- ClusterRole → Cluster-wide

**Q: Why do we need ServiceAccounts if Pods can run apps anyway?**
- Because some apps (monitoring, GitOps, controllers) need to interact with the Kubernetes API. SA gives them an identity + permissions.

**Q: What happens if we don't define serviceAccountName?**
- Pod uses the default SA. Usually has no special permissions.

**Q: Can ServiceAccounts access external clusters?**
- No, SA tokens work only in their own cluster unless external trust is configured.

**Q: Difference between RoleBinding and ClusterRoleBinding?**
- RoleBinding → Gives Role/ClusterRole permissions in one namespace
- ClusterRoleBinding → Gives ClusterRole permissions across all namespaces

## 6. Complete Example

```yaml
# 1. ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app-sa
  namespace: default
---
# 2. Role (can only read pods in default namespace)
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
---
# 3. RoleBinding (connect SA to Role)
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: ServiceAccount
  name: my-app-sa
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
---
# 4. Pod using the ServiceAccount
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
  namespace: default
spec:
  serviceAccountName: my-app-sa
  containers:
  - name: curl
    image: curlimages/curl
    command: ["sleep", "3600"]
```

Inside this Pod, if you run `kubectl exec -it test-pod -- curl ...`, it will be able to list Pods in default namespace, but not in others.

## 7. Key Takeaways

- **ServiceAccount** = Pod identity
- **RBAC** = Permissions model
- **Roles/ClusterRoles** = Define permissions
- **RoleBinding/ClusterRoleBinding** = Assign permissions to SAs

**Interview Trick**: Always say "ServiceAccounts + RBAC = secure way for Pods to interact with Kubernetes API."