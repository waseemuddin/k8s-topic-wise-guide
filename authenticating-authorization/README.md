# Kubernetes Authentication vs Authorization (Explained Like You Are 5 Years Old)

In **Kubernetes**, security works in two main steps:

1. **Authentication** – Who are you?
2. **Authorization** – What are you allowed to do?

Think of it like **entering a school with a security guard**.

---

### 1. Authentication (Who Are You?)

Imagine you want to **enter a school**.

At the gate, a **security guard** asks:

> "Who are you?"

You show your **school ID card**.

If the guard verifies the ID card, you are allowed to enter the school.

This process is called **Authentication**.

**Authentication = Verifying your identity.**

In Kubernetes, when a user sends a request to the **API Server**, Kubernetes first checks **who the user is**.

---

### Authentication Methods in Kubernetes

Kubernetes supports several authentication methods.

| Method | Example |
|------|------|
TLS Certificates | Client certificate authentication |
Bearer Tokens | Service Account tokens |
Username & Password | Basic authentication |
OIDC | Google / OAuth login |
Webhook | External authentication system |

---

#### Example (kubectl command)

A DevOps engineer runs the command:

```bash
kubectl get pods
```

Kubernetes API Server checks:

Is this user valid?

Does the user have a valid certificate or token?

If the identity is valid → Authentication successful


### 2. Authorization (What Are You Allowed To Do?)

Now imagine you are inside the school.

The guard asks another question:

"What are you allowed to do?"

#### Example:

Person	Permission
Student	Enter classroom
Teacher	Enter staff room
Principal	Access everything

This step is called Authorization.

#### Authorization = Checking what actions you are allowed to perform.

#### Example in Kubernetes

Suppose a user runs:

```bash
kubectl delete pod nginx
```
Kubernetes checks:

### Authentication
Is this a valid user?

### Authorization
Is the user allowed to delete pods?

If permission is missing:

Error from server (Forbidden): pods "nginx" is forbidden

### 3. Kubernetes RBAC Example

Kubernetes commonly uses RBAC (Role Based Access Control) for authorization.

#### Example Role:
```bash

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

#### This role allows users to:
View pods
List pods
But they cannot delete pods.

### Bind Role to User
```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
subjects:
- kind: User
  name: dev-user
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

#### Now dev-user can:
Get pods
List pods

#### But cannot:
Delete pods
Create pods

#### Kubernetes Request Flow

When a user sends a request:

User → Kubernetes API Server
          │
          ▼
1. Authentication (Who are you?)
          │
          ▼
2. Authorization (What can you do?)
          │
          ▼
Request Allowed or Denied


### Simple Real-Life Example

#### Imagine a library system.

Step 1 – Authentication

The librarian asks for your library card.

Step 2 – Authorization

The librarian checks your permissions.

Person	Permission
Student	Read books
Teacher	Borrow books
Admin	Manage library
Difference Between Authentication and Authorization
Feature	Authentication	Authorization
Purpose	Verify identity	Verify permissions
Question	Who are you?	What can you do?
Example	Login with certificate/token	RBAC roles and permissions

##### DevOps Interview Tip

A common interview question is:

What is the difference between Authentication and Authorization in Kubernetes?

Best short answer:

Authentication verifies the identity of a user, while Authorization determines what actions the user is allowed to perform using mechanisms like RBAC.