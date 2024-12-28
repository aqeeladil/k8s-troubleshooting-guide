# Securing a Database Using Kubernetes Network Policies

## Use Case:

- Imagine you have a Kubernetes cluster with two main namespaces:
    - Secure Namespace: Hosts the database (e.g., Redis, MongoDB, MySQL).
    - Sandbox Namespace: Hosts other general-purpose pods.
- The goal is to ensure only specific pods from the `Secure` Namespace can access the database, blocking unauthorized access from other namespaces (e.g., `Sandbox`).

## Problem Scenario: Why Restrict Access?

- A hacker pod could be deployed in the Sandbox namespace, either accidentally or maliciously.
- Without restrictions, this pod can directly access your database, potentially stealing or corrupting data.
- Inside your Kubernetes cluster, not all pods should have access to sensitive resources like your database.

## Solution: Kubernetes Network Policies

- Kubernetes Network Policies control network traffic at the pod level.
- They are divided into two types:
    - **Ingress Policies:** Control inbound traffic to a pod.
    - **Egress Policies:** Control outbound traffic from a pod.
- Ingress Policies are the most commonly used network policies for securing databases.
- Testing can be done on environments like EKS (v1.27+), OpenShift, or Rancher.

## Key Configuration Methods in Network Policies:

- **IP Address-based Access Control:** Allow/deny specific IP ranges.
- **Namespace Selector:** Allow access from specific namespaces.
- **Pod Selector (Labels):** Allow access based on pod labels.

**Example using IPBlock:**
    ```yaml
    ingress:
      - from:
          - ipBlock:
              cidr: 10.0.0.0/16
    ```

## Workflow:

- A Redis Database is deployed in the `Secure` Namespace.
- A "hacker pod" is deployed in the `Sandbox` Namespace.
- Without a Network Policy, the hacker pod can connect to the database.
- A Network Policy is applied to allow access only from pods with a specific label in the Secure Namespace.
- The hacker pod’s connection is successfully blocked.

## Step-by-Step Implementation:

```bash
# Deploy Redis Database in Secure Namespace
kubectl create namespace secure
kubectl apply -f db.yaml -n secure

# Deploy Hacker in Sandbox Namespace
kubectl create namespace sandbox
kubectl apply -f hack.yaml -n sandbox

# Test Without Network Policy
# Log into the hacker pod:
kubectl exec -it hacker-pod -n sandbox -- /bin/bash

# Install Redis CLI (if not already installed):
apt update && apt install -y redis-tools

# Get the IP address of the Redis pod:
kubectl get pod redis -n secure -o wide

# Try to connect to the Redis database:
redis-cli -h <Redis-Pod-IP>

# Observation: The hacker pod can successfully connect to the Redis database. This is a security risk.
```
```bash
# Apply Network Policy to restrict access to the database.
kubectl apply -f network-policy.yaml -n secure

# Test again from the hacker pod:
redis-cli -h <Redis-Pod-IP>

# Observation: The connection will now be blocked.
```
```bash
# Deploy an authorized application pod in the Secure Namespace:
kubectl apply -f auth-app.yaml -n secure

# Try to connect to the Redis database:
kubectl exec -it app-pod -n secure -- /bin/bash
kubectl get pod redis -n secure -o wide
redis-cli -h <Redis-Pod-IP>

# Observation: The connection will succeed.
```





