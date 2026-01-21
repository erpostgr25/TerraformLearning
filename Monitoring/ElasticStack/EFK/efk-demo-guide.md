# EFK Stack for Kubernetes

### Overview
The EFK stack (Elasticsearch, Fluent Bit/Fluentd, Kibana) is optimized for Kubernetes environments, collecting logs from containers and pods.

**Architecture:**
```
K8s Pods → Fluent Bit (DaemonSet) → Elasticsearch → Kibana
```

### Prerequisites
- Running Kubernetes cluster (minikube, kind, or cloud provider)
- kubectl configured
- Helm 3.x installed
- At least 8GB RAM for the cluster

---

### 2.1 Installation Steps

#### Step 1: Create Namespace
```bash
kubectl create namespace logging
```

#### Step 2: Add Helm Repositories
```bash
# Add Elastic Helm repository
helm repo add elastic https://helm.elastic.co

# Add Fluent Helm repository
helm repo add fluent https://fluent.github.io/helm-charts

# Update repositories
helm repo update
```

#### Step 3: Install Elasticsearch
```bash
# Install Elasticsearch using the elasticsearch-values.yaml in the current folder
helm install elasticsearch elastic/elasticsearch \
  --namespace logging \
  --values elasticsearch-values.yaml

# Wait for Elasticsearch to be ready
kubectl wait --for=condition=ready pod -l app=elasticsearch-master -n logging --timeout=300s

# Check status
kubectl get pods -n logging
```

#### Step 4: Install Kibana
```bash
# Install Kibana using the Create kibana-values.yaml in the current folder
helm install kibana elastic/kibana \
  --namespace logging \
  --values kibana-values.yaml

# Wait for Kibana to be ready
kubectl wait --for=condition=ready pod -l app=kibana -n logging --timeout=300s

# Get Kibana URL
kubectl get svc kibana-kibana -n logging
```

#### Step 5: Install Fluent Bit

```bash
# Install Fluent Bit using the fluent-bit-values.yaml file in the current folder
helm install fluent-bit fluent/fluent-bit \
  --namespace logging \
  --values fluent-bit-values.yaml

# Verify DaemonSet
kubectl get daemonset fluent-bit -n logging
kubectl get pods -n logging -l app.kubernetes.io/name=fluent-bit
```

---

### 2.2 Usage and Verification

#### Access Kibana

For Minikube:
```bash
minikube service kibana-kibana -n logging
```

For NodePort:
```bash
# Get the node IP
kubectl get nodes -o wide

# Access Kibana at: http://<NODE_IP>:30561
```


#### Login to Kibana
- Username: `elastic`
- Password: `changeme` (or what you set in values.yaml)

#### Create Index Pattern
1. Navigate to **Stack Management** → **Index Patterns**
2. Click **Create index pattern**
3. Enter pattern: `fluent-bit-*`
4. Select `@timestamp` as time field
5. Click **Create**

#### View Container Logs
1. Go to **Discover**
2. Select `fluent-bit-*` index pattern
3. Add fields to columns:
   - `kubernetes.namespace_name`
   - `kubernetes.pod_name`
   - `kubernetes.container_name`
   - `log`

#### Create Useful Filters
```
# Filter by namespace
kubernetes.namespace_name: "default"

# Filter by pod
kubernetes.pod_name: "my-pod-*"

# Filter by container
kubernetes.container_name: "nginx"

# Error logs only
log: *error* OR log: *ERROR*
```


### 2.3 Deploy Sample Application for Testing

```bash
# Deploy sample applications using sample-app.yaml file in the current folder
kubectl apply -f sample-app.yaml

# Verify deployment
kubectl get pods -n demo-app

# Generate some traffic to nginx
kubectl run curl-test --image=curlimages/curl -i --rm --restart=Never -- \
  curl http://nginx-service.demo-app.svc.cluster.local
```