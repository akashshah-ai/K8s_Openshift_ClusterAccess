# Kubernetes/OpenShift Cluster Access

Creates a permanent, user-independent KUBECONFIG file for external scripts to access Kubernetes API with minimal privileges. Follows Principle of Least Privilege for secure programmatic access.

## Prerequisites
- Kubernetes cluster access with admin privileges
- kubectl v1.24+ (for `kubectl create token`)
- Basic YAML and kubectl knowledge
- Sample application deployed
- kubectl CLI configured and connected to the cluster

## Setup Instructions

### 1. Create Namespace and Service Account

kubectl apply -f namespace.yaml
kubectl apply -f service-account.yaml -n cluster-access

### 2. Create RBAC Role and Binding

kubectl apply -f role.yaml -n cluster-access
kubectl apply -f role-binding.yaml -n cluster-access


### 3. Create Token Secret

kubectl apply -f token-secret.yaml -n cluster-access

### 4. Extract Configuration Values

# API Server URL
export SERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')

# CA Certificate Data
export CA_DATA=$(kubectl config view --raw -o jsonpath='{.clusters[0].cluster.certificate-authority-data}')

# Service Account Token
export TOKEN=$(kubectl -n cluster-access get secret cluster-access-token -o jsonpath='{.data.token}' | base64 -d)

### 5. Add Extracted Values to Kubeconfig

vi cluster-access-kubeconfig.yaml 

## Usage

### Test Access

KUBECONFIG=cluster-access-kubeconfig.yaml kubectl -n cluster-access get pods

### Multiple Contexts

# Switch contexts
kubectl config use-context cluster-access-context --kubeconfig=cluster-access-kubeconfig.yaml

# Create pod
kubectl run test-pod --image=nginx -n cluster-access

## Verification Commands

# Check permissions
kubectl auth can-i list pods --namespace=cluster-access --as=system:serviceaccount:cluster-access:cluster-access-sa

# List resources
kubectl -n cluster-access get role,rolebinding,sa,secret

## Cleanup

kubectl delete -f . -n cluster-access

## Debug and Log Analysis – Kubernetes

# Inspect Pods
Use kubectl to inspect pod status and configuration.

- kubectl get pods
- kubectl describe pod <pod-name>

These commands provide pod state, events, restart count, and container details.

# Events
Kubernetes events help diagnose scheduling, image pull, and runtime issues.
- kubectl get events
- kubectl get events -n <namespace>

Important fields: KIND, TYPE, REASON, SOURCE, MESSAGE.

# Application Logs
View application logs using kubectl logs.
- kubectl logs <pod-name>
- kubectl logs --previous <pod-name>
Applications should log to stdout/stderr for Kubernetes to capture logs.

# Exec Into Containers
For deep debugging, exec into running containers if a shell exists.
- kubectl exec -it <pod-name> -- /bin/bash

Shell access is recommended for test environments, not production images.

# Services & Networking
Verify services and endpoints when applications are unreachable.
- kubectl get svc
- kubectl describe svc <service-name>

Ensure correct NodePort, targetPort, and endpoints.

# DNS Debugging
Check DNS resolution from inside pods.
- nslookup kubernetes.default
- Verify /etc/resolv.conf contains correct nameserver, search domains, and ndots value.

# Network Policies
- NetworkPolicy controls pod-to-pod communication using labels.
- Verify podSelector, ingress, and egress rules if traffic is blocked.
