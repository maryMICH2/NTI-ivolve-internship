# Network Policy: Control Pod-to-Pod Traffic

This task demonstrates how to restrict pod-to-pod communication in
Kubernetes using a NetworkPolicy. The goal is to ensure that only the
Node.js application pods can access the MySQL database pod on port 3306,
while blocking all other traffic.

## Objective

-   Create a NetworkPolicy named allow-app-to-mysql
-   Target all pods labeled app=mysql
-   Apply Ingress-only restrictions
-   Allow incoming traffic only from pods labeled app=nodejs
-   Restrict allowed traffic to TCP port 3306 (MySQL)

## NetworkPolicy Manifest

`network-policy.yaml`:

``` yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-app-to-mysql
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: mysql
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: nodejs
      ports:
        - protocol: TCP
          port: 3306
```

## Apply the Network Policy

``` bash
kubectl apply -f network-policy.yaml
```

## Verify Network Policy Creation

``` bash
kubectl get networkpolicy
kubectl describe networkpolicy allow-app-to-mysql
```

## Testing the Network Policy

### 1. Exec into a Node.js pod (allowed)

``` bash
kubectl exec -it <nodejs-pod> -- sh
nc -zv mysql 3306
```
![test-network-policy](images/test-network-policy.png)

### 2. Exec into any other pod (blocked)

``` bash
kubectl exec -it <random-pod> -- sh
nc -zv mysql 3306
```

## Summary

-   MySQL accepts traffic only from app=nodejs pods.
-   All other pod-to-pod connections to MySQL are blocked.
-   The policy secures database access inside the cluster.
