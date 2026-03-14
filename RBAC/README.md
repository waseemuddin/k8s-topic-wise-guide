# Kubernetes RBAC (Role-Based Access Control) 🎮

[![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

## 📋 Table of Contents
- [What is RBAC?](#-what-is-rbac)
- [The 4 Main Pieces of RBAC](#-the-4-main-pieces-of-rbac)
- [Understanding API Groups & Resources](#-understanding-api-groups--resources)
- [Role vs ClusterRole](#-role-vs-clusterrole)
- [Verbs (Actions) Explained](#-verbs-actions-explained)
- [🎮 Practical Lab: The Toy Store Adventure](#-practical-lab-the-toy-store-adventure)
  - [Prerequisites](#prerequisites)
  - [Scenario](#scenario)
  - [Step 0: Setup](#step-0-setup---create-our-toy-store)
  - [Step 1: Create Users](#step-1-create-users-the-employees)
  - [Step 2: Create Roles](#step-2-create-roles-job-descriptions)
  - [Step 3: Create RoleBindings](#step-3-create-rolebindings-assign-jobs-to-people)
  - [Step 4: Configure kubectl Contexts](#step-4-configure-kubectl-for-each-user)
  - [Step 5: Test Permissions](#step-5-test-the-permissions---lets-play-)
  - [Step 6: Verify with Commands](#step-6-check-permissions-like-a-detective-)
- [📝 Cheat Sheet](#-cheat-sheet)
- [🎯 Quick Lab: Try It Yourself!](#-quick-lab-try-it-yourself)
- [Best Practices](#-best-practices)
- [Troubleshooting](#-troubleshooting)
- [Additional Resources](#-additional-resources)

## 🎈 What is RBAC? (Explained Like You're 5)

Imagine your Kubernetes cluster is a **big toy box** 🧸 with different toys:

- **Pods** = Your favorite action figures
- **Services** = The toy car garage
- **Deployments** = A whole set of Lego blocks

**RBAC (Role-Based Access Control)** is like giving **keys** 🔑 to different people who want to play with your toys:

- **Mommy** gets keys to EVERYTHING (admin)
- **Your friend** only gets keys to play with action figures (pods only)
- **Your little sibling** can ONLY look at toys, not touch them (read-only)
- **The babysitter** can organize toys but not take them apart (limited access)

### Simple Definition
**RBAC = Who can do What to Which things**

- **Who** = Users (like "john," "mary," or "robot")
- **What** = Actions (get, list, create, delete)
- **Which things** = Resources (pods, deployments, secrets)

## 🧩 The 4 Main Pieces of RBAC

| Component | Description | Real-world Analogy |
|-----------|-------------|-------------------|
| **User** | The person/thing trying to do something | Employee ID badge |
| **Role** | List of permissions (allowed actions) | Job description |
| **RoleBinding** | Glue that sticks a User to a Role | Employment contract |
| **Namespace** | A separate room in the toy box | Different departments |

## 📚 Understanding API Groups & Resources

### Common API Groups
| Group | Contains | Example |
|-------|----------|---------|
| `""` (core) | pods, services, configmaps, secrets | `kubectl get pods` |
| `"apps"` | deployments, statefulsets, replicasets | `kubectl get deployments` |
| `"batch"` | jobs, cronjobs | `kubectl get jobs` |
| `"networking.k8s.io"` | ingresses, networkpolicies | `kubectl get ingresses` |

### Important Notes
- **Resource names must be PLURAL** in the resources list
  - ✅ Correct: `"deployments"`, `"services"`, `"pods"`
  - ❌ Wrong: `"deployment"`, `"service"`, `"pod"`

## 👑 Role vs ClusterRole

| **Role** | **ClusterRole** |
|----------|-----------------|
| Works in ONE namespace | Works in ALL namespaces |
| Like a room key | Like a master key to the whole building |
| `kind: Role` | `kind: ClusterRole` |
| Limited scope | Global scope |
| Example: `role-binding.yaml` | Example: `cluster-role-binding.yaml` |

## 🎯 Verbs (Actions) Explained

| **Verb** | **Meaning** | **Like** | **Command Example** |
|----------|-------------|----------|---------------------|
| `get` | See one specific thing | "Can I see THAT toy?" | `kubectl get pod nginx` |
| `list` | See all things | "Can I see ALL toys?" | `kubectl get pods` |
| `watch` | Keep watching for changes | "Tell me if someone plays with toys" | `kubectl get pods --watch` |
| `create` | Make new things | "Can I make a new toy?" | `kubectl create deployment` |
| `update` | Change existing things | "Can I repaint this toy?" | `kubectl edit deployment` |
| `patch` | Make small changes | "Can I fix this toy's arm?" | `kubectl patch deployment` |
| `delete` | Remove things | "Can I throw this toy away?" | `kubectl delete pod` |

---

# 🎮 PRACTICAL LAB: The Toy Store Adventure

## Prerequisites
- ✅ Kubernetes cluster (Minikube recommended)
- ✅ `kubectl` installed and configured
- ✅ `openssl` for generating certificates
- ✅ Basic understanding of Linux terminal

## Scenario
You run a Kubernetes toy store with 3 employees:
- **Alex** (Store Manager) - Needs full access to everything
- **Bob** (Sales Associate) - Can view and create toys, but can't delete them
- **Charlie** (Customer) - Can only look at toys (read-only)

## Step 0: Setup - Create Our Toy Store

```bash
# Create namespaces (different rooms in our store)
kubectl create namespace store-front
kubectl create namespace store-back

# Verify namespaces
kubectl get namespaces | grep store
```

## Step 1: Create Users (The Employees)

Create certificates for each user (like giving them ID badges):

```bash
# For Alex (Manager)
openssl genrsa -out alex.key 2048
openssl req -new -key alex.key -out alex.csr -subj "/CN=alex"
openssl x509 -req -in alex.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out alex.crt -days 365

# For Bob (Sales Associate)
openssl genrsa -out bob.key 2048
openssl req -new -key bob.key -out bob.csr -subj "/CN=bob"
openssl x509 -req -in bob.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out bob.crt -days 365

# For Charlie (Customer)
openssl genrsa -out charlie.key 2048
openssl req -new -key charlie.key -out charlie.csr -subj "/CN=charlie"
openssl x509 -req -in charlie.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out charlie.crt -days 365
```

## Step 2: Create Roles (Job Descriptions)

### Alex's Role - Store Manager (Can do EVERYTHING)

```bash
cat > manager-role.yaml << 'EOF'
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: store-manager
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
EOF

kubectl apply -f manager-role.yaml
```

### Bob's Role - Sales Associate (Can view and create, but not delete)

```bash
cat > sales-role.yaml << 'EOF'
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: store-front
  name: sales-associate
rules:
# Can look at everything
- apiGroups: ["", "apps"]
  resources: ["pods", "services", "deployments"]
  verbs: ["get", "list", "watch"]
# Can create new things
- apiGroups: ["", "apps"]
  resources: ["pods", "services", "deployments"]
  verbs: ["create"]
# CANNOT delete anything (notice: no delete verb)
EOF

kubectl apply -f sales-role.yaml
```

### Charlie's Role - Customer (Can only look, cannot touch)

```bash
cat > customer-role.yaml << 'EOF'
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: store-front
  name: customer
rules:
- apiGroups: ["", "apps"]
  resources: ["pods", "services", "deployments"]
  verbs: ["get", "list", "watch"]  # ONLY view permissions!
EOF

kubectl apply -f customer-role.yaml
```

## Step 3: Create RoleBindings (Assign jobs to people)

### Assign Alex as Manager (works in ALL namespaces)

```bash
cat > alex-binding.yaml << 'EOF'
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: alex-manager
subjects:
- kind: User
  name: alex
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: store-manager
  apiGroup: rbac.authorization.k8s.io
EOF

kubectl apply -f alex-binding.yaml
```

### Assign Bob as Sales Associate (only in store-front)

```bash
cat > bob-binding.yaml << 'EOF'
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: store-front
  name: bob-sales
subjects:
- kind: User
  name: bob
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: sales-associate
  apiGroup: rbac.authorization.k8s.io
EOF

kubectl apply -f bob-binding.yaml
```

### Assign Charlie as Customer (only in store-front)

```bash
cat > charlie-binding.yaml << 'EOF'
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: store-front
  name: charlie-customer
subjects:
- kind: User
  name: charlie
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: customer
  apiGroup: rbac.authorization.k8s.io
EOF

kubectl apply -f charlie-binding.yaml
```

## Step 4: Configure kubectl for each user

```bash
# Create contexts for each user
# Alex's context
kubectl config set-credentials alex --client-certificate=alex.crt --client-key=alex.key --embed-certs=true
kubectl config set-context alex-context --cluster=minikube --user=alex

# Bob's context
kubectl config set-credentials bob --client-certificate=bob.crt --client-key=bob.key --embed-certs=true
kubectl config set-context bob-context --cluster=minikube --user=bob --namespace=store-front

# Charlie's context
kubectl config set-credentials charlie --client-certificate=charlie.crt --client-key=charlie.key --embed-certs=true
kubectl config set-context charlie-context --cluster=minikube --user=charlie --namespace=store-front

# View all contexts
kubectl config get-contexts
```

## Step 5: Test the Permissions - Let's Play! 🎮

### Test 1: Alex (The Manager) - Can do ANYTHING

```bash
# Switch to Alex
kubectl config use-context alex-context

# Alex can create in ANY namespace
kubectl create deployment nginx --image=nginx -n store-front
kubectl create deployment redis --image=redis -n store-back

# Alex can see everything
kubectl get pods --all-namespaces

# Alex can delete anything
kubectl delete deployment nginx -n store-front
kubectl delete deployment redis -n store-back

# All commands should SUCCEED
```

### Test 2: Bob (Sales Associate) - Can create but NOT delete

```bash
# Switch to Bob
kubectl config use-context bob-context

# Bob can CREATE (should WORK!)
kubectl create deployment web --image=nginx
kubectl expose deployment web --port=80 --type=NodePort

# Bob can VIEW (should WORK!)
kubectl get pods
kubectl get services
kubectl get deployments

# Bob tries to DELETE (should FAIL!)
kubectl delete deployment web
# Expected: Error from server (Forbidden): deployments.apps "web" is forbidden

# Bob tries to work in store-back (should FAIL!)
kubectl get pods -n store-back
# Expected: Error from server (Forbidden): pods is forbidden
```

### Test 3: Charlie (Customer) - Can ONLY look

```bash
# Switch to Charlie
kubectl config use-context charlie-context

# Charlie can VIEW (should WORK!)
kubectl get pods
kubectl get services
kubectl get deployments

# Charlie tries to CREATE (should FAIL!)
kubectl create deployment hack --image=nginx
# Expected: Error from server (Forbidden): deployments.apps is forbidden

# Charlie tries to DELETE (should FAIL!)
kubectl delete service web
# Expected: Error from server (Forbidden): services "web" is forbidden

# Charlie tries to UPDATE (should FAIL!)
kubectl scale deployment web --replicas=3
# Expected: Error from server (Forbidden): deployments.apps "web" is forbidden
```

## Step 6: Check permissions like a detective 🕵️

```bash
# Switch back to admin context
kubectl config use-context minikube

# Check specific permissions
kubectl auth can-i create deployments --as bob -n store-front
# Returns: yes

kubectl auth can-i delete deployments --as bob -n store-front
# Returns: no

# Check all permissions for a user
kubectl auth can-i --list --as charlie -n store-front

# See all bindings
kubectl get rolebindings,clusterrolebindings -A | grep -E "alex|bob|charlie"

# Describe specific bindings
kubectl describe rolebinding bob-sales -n store-front
kubectl describe clusterrolebinding alex-manager
```

---

## 📝 Cheat Sheet

### Common Role Templates

#### 1. Read-Only Access
```yaml
rules:
- apiGroups: ["", "apps"]
  resources: ["pods", "services", "deployments"]
  verbs: ["get", "list", "watch"]
```

#### 2. Full CRUD on Pods
```yaml
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

#### 3. Deployments Management
```yaml
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

#### 4. Multiple Resources + Logs
```yaml
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log", "services"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "create"]
```

### Quick Commands Reference

| **Command** | **Purpose** |
|-------------|-------------|
| `kubectl auth can-i create pods --as bob` | Check if bob can create pods |
| `kubectl auth can-i --list --as bob` | List all permissions for bob |
| `kubectl get roles -A` | List all roles |
| `kubectl get rolebindings -A` | List all role bindings |
| `kubectl describe role pod-reader` | Describe a specific role |
| `kubectl describe rolebinding read-pods` | Describe a specific binding |
| `kubectl delete role pod-reader` | Delete a role |
| `kubectl delete rolebinding read-pods` | Delete a binding |

---

## 🎯 Quick Lab: Try It Yourself!

**Challenge:** Create a new user "dave" with read-only access to pods in store-front.

### Solution:

```bash
# 1. Create dave's certificate
openssl genrsa -out dave.key 2048
openssl req -new -key dave.key -out dave.csr -subj "/CN=dave"
openssl x509 -req -in dave.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out dave.crt -days 365

# 2. Create role
cat > pod-viewer.yaml << EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: store-front
  name: pod-viewer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
EOF
kubectl apply -f pod-viewer.yaml

# 3. Create binding
cat > dave-binding.yaml << EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: store-front
  name: dave-viewer
subjects:
- kind: User
  name: dave
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-viewer
  apiGroup: rbac.authorization.k8s.io
EOF
kubectl apply -f dave-binding.yaml

# 4. Configure and test
kubectl config set-credentials dave --client-certificate=dave.crt --client-key=dave.key --embed-certs=true
kubectl config set-context dave-context --cluster=minikube --user=dave --namespace=store-front
kubectl config use-context dave-context

# Test
kubectl get pods  # Should WORK!
kubectl create deployment test --image=nginx  # Should FAIL!
```

---

## 💡 Best Practices

1. **Principle of Least Privilege** - Give users ONLY the permissions they need
2. **Use Groups** - Bind roles to groups, not individual users
3. **Regular Audits** - Periodically review who has access to what
4. **Namespace Isolation** - Use namespaces to separate environments
5. **Avoid ClusterAdmin** - Only give cluster-wide access when absolutely necessary
6. **Document Everything** - Keep track of why users have certain permissions

## 🔍 Troubleshooting

### Common Issues and Solutions

| **Issue** | **Error Message** | **Solution** |
|-----------|-------------------|--------------|
| Wrong API Group | `cannot list resource "deployments" in API group ""` | Add `apiGroups: ["apps"]` |
| Wrong Resource Name | `the server could not find the requested resource` | Use plural: "deployments" not "deployment" |
| Missing Verbs | `cannot create resource "pods"` | Add `"create"` to verbs |
| Wrong Namespace | `pods is forbidden: User cannot list resource` | Check namespace in Role and Binding |
| Authentication Failed | `Unauthorized` | Check certificate is valid and signed by correct CA |

### Debugging Commands

```bash
# Check certificate details
openssl x509 -in user.crt -text -noout | grep -A 2 "Subject:"

# Verify certificate issuer
openssl x509 -in user.crt -noout -issuer

# Check certificate dates
openssl x509 -in user.crt -noout -dates

# Test authentication with curl
curl --cert user.crt --key user.key https://kubernetes-api-server/api/v1/namespaces/default/pods
```

## 🎨 Visual Memory Aid

**RBAC is like a restaurant:**
- **Users** = Customers and staff
- **Roles** = Job descriptions (Chef, Waiter, Customer)
- **RoleBindings** = Job assignments (John is the Chef)
- **Namespaces** = Different sections (Kitchen, Dining Room)
- **Verbs** = Actions (Cook, Serve, Eat, Clean)

## 📚 Additional Resources

- [Official Kubernetes RBAC Documentation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [RBAC Good Practices](https://kubernetes.io/docs/concepts/security/rbac-good-practices/)

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🤝 Contributing

Contributions, issues, and feature requests are welcome! Feel free to check [issues page](issues/).

## ⭐ Support

If you found this guide helpful, please give it a star! ⭐

---

**Remember:** Without RBAC, everyone could do everything - chaos! With RBAC, everyone has just the right amount of access to do their job safely. 🚀