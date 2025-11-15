# Node-Wide Pod Management with DaemonSet

This repository demonstrates how to deploy a **DaemonSet** in Kubernetes to monitor nodes using **Prometheus node-exporter**. The deployment ensures one node-exporter pod runs on each node and collects system-level metrics.

## Objective

* Create a dedicated namespace `monitoring`.
* Deploy a DaemonSet for Prometheus node-exporter that tolerates all node taints.
* Verify that a node-exporter pod runs on each node.
* Access and validate metrics exposure on port 9100.
* No Helm deployment is included in this setup.

## Steps Performed

### 1. Create Monitoring Namespace

```bash
kubectl create namespace monitoring
```

### 2. Deploy Node Exporter DaemonSet

`node-exporter-daemonset.yaml`:
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      tolerations:
        - operator: "Exists"
      containers:
        - name: node-exporter
          image: prom/node-exporter:latest
          ports:
            - containerPort: 9100
              name: metrics

```

Apply the DaemonSet:

```bash
kubectl apply -f node-exporter-daemonset.yaml
```

### 3. Verify Pod Scheduling

```bash
kubectl get pods -n monitoring -o wide
```

You should see **one node-exporter pod per node**.

### 4. Check Metrics Exposure

Use `kubectl port-forward` from any node-exporter pod to access metrics:

```bash
kubectl port-forward -n monitoring pod/<node-exporter-pod-name> 9100:9100
```

Access metrics locally via:

```
http://localhost:9100/metrics
```

### 5. Sample Metrics Output

```text
# TYPE process_network_receive_bytes_total counter
process_network_receive_bytes_total 360
# TYPE process_network_transmit_bytes_total counter
process_network_transmit_bytes_total 360
# TYPE process_open_fds gauge
process_open_fds 7
# TYPE process_resident_memory_bytes gauge
process_resident_memory_bytes 1.5335424e+07
# TYPE process_start_time_seconds gauge
process_start_time_seconds 1.76323503657e+09
# TYPE process_virtual_memory_bytes gauge
process_virtual_memory_bytes 1.27102976e+09
# TYPE process_virtual_memory_max_bytes gauge
process_virtual_memory_max_bytes 1.8446744073709552e+19
# TYPE promhttp_metric_handler_errors_total counter
promhttp_metric_handler_errors_total{cause="encoding"} 0
promhttp_metric_handler_errors_total{cause="gathering"} 0
# TYPE promhttp_metric_handler_requests_in_flight gauge
promhttp_metric_handler_requests_in_flight 1
# TYPE promhttp_metric_handler_requests_total counter
promhttp_metric_handler_requests_total{code="200"} 0
promhttp_metric_handler_requests_total{code="500"} 0
```

### 6. Summary

* The **DaemonSet** ensures a node-exporter pod runs on every node.
* Tolerations allow scheduling on tainted nodes.
* Metrics are exposed on **port 9100** and can be scraped by Prometheus.
* Sample metrics include network, memory, file descriptors, and process stats.
* Setup is fully functional in CodeKeller w
