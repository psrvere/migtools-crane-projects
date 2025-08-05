## 1. Basic Stateless Pod Migration

**Setup:** Set up two local Kubernetes clusters (using Minikube or Kind). Deploy a simple Nginx pod in the source cluster. Use Crane to export the resources, transform them (e.g., for namespace changes), and apply to the destination. Run: `crane export -n default`, `crane transform`, `crane apply`.

**What you'll learn:** Core CLI workflow (export-transform-apply pipeline), basic manifest handling, and non-destructive operations. Reference the guestbook example in the Crane repo for guidance.

Crane Repo - https://github.com/migtools/crane
Crane Lib Repo - https://github.com/migtools/crane-lib

---

## Implementation Guide

### Prerequisites

- Docker installed and running
- Kind CLI installed
- kubectl installed
- Crane CLI installed

#### Installing Crane CLI

**Using Go (Recommended for macOS)**
```bash
# Install Crane CLI using Go
go install github.com/konveyor/crane@latest

# Verify installation
crane version
```

**Note**: The Go installation method installs Crane globally in your system PATH (`/Users/prathore/go/bin/crane`), making it available from any directory.

### Project Structure

```
statelesspod/
├── README.md
├── go.mod
├── clusters/
│   ├── source-cluster.yaml
│   └── destination-cluster.yaml
├── manifests/
│   ├── nginx-deployment.yaml
│   ├── nginx-service.yaml
│   └── nginx-configmap.yaml
└── crane/
    ├── export/
    ├── transform/
    └── apply/
```

### Step-by-Step Migration Guide

#### 1. Setup Local Clusters

Create two Kind clusters: `source-cluster` and `destination-cluster`.

```bash
# Create source cluster
kind create cluster --name source-cluster --config clusters/source-cluster.yaml

# Create destination cluster  
kind create cluster --name destination-cluster --config clusters/destination-cluster.yaml

# Verify clusters are running
kind get clusters

# List available contexts
kubectl config get-contexts

# Verify you can connect to both clusters
kubectl config use-context kind-source-cluster
kubectl get nodes

kubectl config use-context kind-destination-cluster
kubectl get nodes
```

#### 2. Deploy Application to Source Cluster

```bash
# Switch to source cluster
kubectl config use-context kind-source-cluster

# Deploy Nginx application
kubectl apply -f manifests/nginx-deployment.yaml
kubectl apply -f manifests/nginx-service.yaml
kubectl apply -f manifests/nginx-configmap.yaml

# Wait for pods to be ready
kubectl wait --for=condition=ready pod -l app=nginx -n default --timeout=300s

# Verify deployment
kubectl get pods,svc,configmap -n default

# Test the application (optional)
kubectl port-forward svc/nginx-service 8080:80 -n default
# Access http://localhost:8080 in your browser
# Then stop port-forward: kill %1
```

#### 3. Export Resources from Source Cluster

```bash
# Create export directory
mkdir -p crane/export

# Export all resources from default namespace
crane export --namespace default --export-dir crane/export

# Verify exported resources
ls -la crane/export/

# View the exported YAML files
cat crane/export/*.yaml
```

#### 4. Transform Resources

```bash
# Create transform directory
mkdir -p crane/transform

# Option 1: Use Crane transform command
crane transform --export-dir crane/export --transform-dir crane/transform

# Option 2: Manually edit the exported YAML files
# Copy exported files to transform directory
cp crane/export/*.yaml crane/transform/

# Edit the files manually to change namespace from 'default' to 'migrated-app'
# Example: sed -i 's/namespace: default/namespace: migrated-app/g' crane/transform/*.yaml

# Verify transformed resources
ls -la crane/transform/
cat crane/transform/*.yaml
```

#### 5. Apply Resources to Destination Cluster

```bash
# Switch to destination cluster
kubectl config use-context kind-destination-cluster

# Generate final YAML files from transformed resources
crane apply --export-dir crane/export --transform-dir crane/transform --output-dir crane/apply

# Manually apply the resources to destination cluster
kubectl apply -f crane/apply/resources/default/

# Wait for pods to be ready
kubectl wait --for=condition=ready pod -l app=nginx -n default --timeout=300s

# Verify migration
kubectl get pods,svc,configmap -n default

# Check the status of all resources
kubectl get all -n default
```

#### 6. Verify Migration

```bash
# Test the migrated application
kubectl port-forward svc/nginx-service 8080:80

# Access http://localhost:8080 in your browser
# You should see the Nginx welcome page

# Stop port-forward when done
```

## Q&A: Node Labels Configuration

### Q: What is the purpose of `node-labels: "ingress-ready=true"` in the cluster configuration?

**A:** This configuration adds a label to the control-plane node that:
- **Enables ingress controllers** to identify nodes capable of handling ingress traffic
- **Works with port mapping** (`hostPort: 8080`) to allow external access to cluster services
- **Enables testing** your application via `localhost:8080` before and after migration
- **Mimics production** ingress patterns for realistic migration practice

```text
External Request: localhost:8080
    ↓
Host Port Mapping: 8080 → 80
    ↓
Control-Plane Node: ingress-ready=true
    ↓
Nginx Service: ClusterIP:80
    ↓
Nginx Pod: Container:80
```

### Q: How does this help with the migration workflow?

**A:** 
- **Before migration**: Test Nginx on source cluster via `localhost:8080`
- **After migration**: Test Nginx on destination cluster via `localhost:8081`
- **Validation**: Compare functionality across clusters to verify successful migration
- **Realistic testing**: Simulates how you'd access applications in production

---

### Q: Why do we need to use `kubectl port-forward` when the cluster is ingress-ready?

**A:** The `ingress-ready=true` label and port mapping are **different from actual ingress**:

**What we have:**
- ✅ **Port mapping**: Direct container-to-host port mapping (`8080:80`)
- ✅ **Ingress-ready label**: Prepares node for future ingress controller installation
- ❌ **No ingress controller**: Actually installed and running

**What we DON'T have:**
- ❌ **Ingress controller**: nginx-ingress, traefik, etc. (not installed)
- ❌ **Ingress resources**: YAML manifests for routing rules
- ❌ **Domain routing**: Multiple domains, paths, SSL

### Q: What's the difference between port-forward and true ingress?

**A:**

**Port-Forward (What we're using):**
```bash
kubectl port-forward svc/nginx-service 8080:80 -n default
# Direct service access via localhost:8080
```

**True Ingress (Not installed):**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
spec:
  rules:
  - host: nginx.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```

**Ingress controllers are add-ons**, not core Kubernetes components, so they require separate installation and configuration.