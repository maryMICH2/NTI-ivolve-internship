# Node.js Application Deployment with ClusterIP Service

This repository demonstrates how to deploy a Node.js application on Kubernetes using a **Deployment** and a **ClusterIP Service**, with persistent storage, ConfigMap/Secret-managed environment variables, and tolerations for tainted nodes.

## Objective

* Deploy Node.js as a Deployment named `nodejs-deployment` with 2 replicas (1 pod may be running due to VM resource limits).
* Use a custom Docker image `maaryii/app-image` from Docker Hub.
* Configure pods to use environment variables from ConfigMap and Secret:

  * `DB_HOST`, `DB_USER` from ConfigMap
  * `DB_PASSWORD` from Secret
* Add a toleration to allow pods to run on a tainted worker node (`workload=worker:NoSchedule`).
* Mount a Persistent Volume Claim (PVC) named `pvc-logs` to store application logs.
* Create a ClusterIP Service named `nodejs-service` to expose the application.
* Test the application locally using `kubectl port-forward`.

## ConfigMap Manifest (`mysql-config-map.yaml`)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config-map
data:
  DB_HOST: mysql-service
  DB_USER: appuser
```

### Apply ConfigMap

```bash
kubectl apply -f mysql-config-map.yaml
```

## Secret Manifest (`mysql-secret.yaml`)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
data:
  DB_PASSWORD: YWRtaW4=
  MYSQL_ROOT_PASSWORD: YWRtaW4=
```

> **Note:** Values are Base64 encoded.

### Apply Secret

```bash
kubectl apply -f mysql-secret.yaml
```

## Persistent Volume & Claim (`pv-pvc.yaml`)

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-logs
spec:
  capacity:
    storage: 500Mi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/app-logs
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-logs
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 500Mi
```

### Apply PV and PVC

```bash
kubectl apply -f pv-pvc.yaml
```

## Node.js Deployment Manifest (`deployment.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejs-deployment
  labels:
    app: nodejs
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nodejs
  template:
    metadata:
      labels:
        app: nodejs
    spec:
      tolerations:
      - key: "workload"
        operator: "Equal"
        value: "worker"
        effect: "NoSchedule"
      containers:
      - name: nodejs-pod
        image: maaryii/app-image
        ports:
        - containerPort: 3000
        envFrom:
          - configMapRef:
              name: mysql-config-map
        env:
          - name: DB_PASSWORD
            valueFrom:
              secretKeyRef: 
                name: mysql-secret
                key: DB_PASSWORD
        volumeMounts:
        - name: app-logs
          mountPath: /usr/src/app/logs
      volumes:
      - name: app-logs
        persistentVolumeClaim:
          claimName: pvc-logs
```

### Apply Deployment

```bash
kubectl apply -f deployment.yaml
```

## ClusterIP Service Manifest (`service.yaml`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nodejs-service
spec:
  type: ClusterIP
  selector:
    app: nodejs
  ports:
    - port: 80
      targetPort: 3000
```

### Apply Service

```bash
kubectl apply -f service.yaml
```

## Verification Steps

1. Check pods:

```bash
kubectl get pods
```

2. Check PVC and PV:

```bash
kubectl get pvc
kubectl get pv
```

3. Check Service:

```bash
kubectl get svc
```

4. View logs of Node.js pod:

```bash
kubectl logs <nodejs-pod-name>
```

5. Test the app with port-forward:

```bash
kubectl port-forward svc/nodejs-service 8081:80
```

Open in browser:
 Screenshot:
![testing-the-pod](images/testing-the-pod.png) 

```
http://localhost:8081/health
http://localhost:8081/ready
```

## Summary

* The Deployment ensures multiple Node.js replicas.
* ClusterIP Service exposes the application internally in the cluster.
* Environment variables are injected via ConfigMap and Secret.
* Logs are persisted using a PVC mounted at `/usr/src/app/logs`.
* Node.js container listens on port 3000; Service forwards from 80 → 3000.
* Tolerations allow pods to run on tainted nodes.
* Storage size was reduced to 500Mi to match VM limitations.


