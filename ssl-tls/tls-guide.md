# Prerequisites
You need: kubectl connected to a cluster (minikube, kind, or any K8s) + openssl installed locally. Run kubectl cluster-info to verify connectivity.

## Step 1: Generate a private key (2048-bit RSA)
```
openssl genrsa -out my-app.key 2048
```

## Step 2: Create a CSR with your app's identity
```
openssl req -new \
  -key my-app.key \
  -out my-app.csr \
  -subj "/CN=my-app/O=my-org"
```

## Step 3:

## Verify the CSR was created
```
openssl req -text -noout -in my-app.csr | head -20
```

## Encode your CSR to base64
```
CSR_BASE64=$(cat my-app.csr | base64 | tr -d '\n')
```

## Create the Kubernetes CSR object
```
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: my-app-csr
spec:
  request: ${CSR_BASE64}
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400
  usages:
  - client auth
EOF
```
## Check the CSR was submitted
```
kubectl get csr my-app-csr
```

## Step 4:


## View pending CSR (status will be 'Pending')
```
kubectl get csr
```

## NAME          AGE   SIGNERNAME                            REQUESTOR   CONDITION
## my-app-csr    10s   kubernetes.io/kube-apiserver-client   admin       Pending

## Admin approves the CSR

kubectl certificate approve my-app-csr

## Verify it's approved
```
kubectl get csr my-app-csr
```

## CONDITION should now show: Approved,Issued

## Step 5:

# Extract the signed certificate
```
kubectl get csr my-app-csr \
  -o jsonpath='{.status.certificate}' | \
  base64 --decode > my-app.crt
```

# Verify the certificate details
```
openssl x509 -in my-app.crt -text -noout
```

# You should see:
# Subject: CN=my-app, O=my-org
# Issuer: CN=kubernetes (your cluster CA)
# Validity: Not After : <24 hours from now>

## Step 6:

## Create a TLS secret with cert + private key
```
kubectl create secret tls my-app-tls \
  --cert=my-app.crt \
  --key=my-app.key
```

## Verify the secret

```
kubectl get secret my-app-tls
```

## View the secret structure
```
kubectl describe secret my-app-tls
```
## Type:  kubernetes.io/tls
## Data:
##   tls.crt: 1234 bytes
##   tls.key: 1679 bytes

# Implementation Example:

## Setup on Minikube or Kubedm Cluster

### Step 01: 

### Enable the nginx ingress controller
```
minikube addons enable ingress
```
### Wait for it to be ready
```
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s
```
### Start the tunnel (KEEP THIS TERMINAL OPEN — don't close it!)
```
minikube tunnel
```
### Step 2: 

### If NOT found, create a quick nginx app:
```
kubectl create deployment my-app --image=nginx
kubectl expose deployment my-app --name=my-app-service --port=80
```
### Check if service exists
```
kubectl get svc my-app-service
```

### Step 03: 

### Use TLS secret in an Nginx Ingress

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
spec:
  tls:
  - hosts:
    - my-app.example.com
    secretName: my-app-tls      # <-- your TLS secret!
  rules:
  - host: my-app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app-service
            port:
              number: 80
```

### Apply it
```
kubectl apply -f ingress.yaml
kubectl get ingress my-app-ingress

kubectl get ingress
```
### ADDRESS column should now show 127.0.0.1


### Step 4: 
### Get the ingress IP
```
INGRESS_IP=$(kubectl get ingress my-app-ingress -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo "Ingress IP: $INGRESS_IP"
```

### Add to /etc/hosts so your machine knows where my-app.example.com points
```
echo "$INGRESS_IP my-app.example.com" | sudo tee -a /etc/hosts
```
### Verify the entry was added
```
cat /etc/hosts | grep my-app
```

### Step 5 :

### Test HTTP
```
curl http://my-app.example.com
```
### Test HTTPS (with -k to skip cert verification for self-signed certs)
```
curl -k https://my-app.example.com
```
### Test HTTPS and see the certificate details
```
curl -kv https://my-app.example.com 2>&1 | grep -A5 "Server certificate"
```

### Full verbose TLS details
```
openssl s_client -connect my-app.example.com:443 -servername my-app.example.com
```
---

### 🎯 Expected Output
```
# curl -k https://my-app.example.com
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...

# openssl s_client output should show:
# subject=CN=my-app.example.com
# issuer=CN=kubernetes  ← signed by your cluster CA!