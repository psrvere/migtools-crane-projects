## 2. Migrating a Deployment with Services

**Setup:** Deploy a sample guestbook app (Redis + frontend) as a Deployment and Service in one cluster. Migrate it using Crane, focusing on how services and endpoints are preserved during transformation. Test connectivity post-migration.

**What you'll learn:** Handling multi-resource apps, understanding service and endpoint preservation during migration, and basic Crane CLI workflow for Kubernetes resource transformations.

---

### Project Structure

```
deploymentwithservice/
├── README.md
├── clusters/
│   ├── source-cluster.yaml
│   └── destination-cluster.yaml
├── manifests/
│   ├── guestbook-deployment.yaml
│   └── guestbook-services.yaml
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

#### 2. Deploy Guestbook Application to Source Cluster

```bash
# Switch to source cluster
kubectl config use-context kind-source-cluster

# Deploy Guestbook application (Redis + Frontend)
kubectl apply -f manifests/guestbook-deployment.yaml
kubectl apply -f manifests/guestbook-services.yaml

# Wait for pods to be ready
kubectl wait --for=condition=ready pod -l app=guestbook -n default --timeout=300s
kubectl wait --for=condition=ready pod -l app=redis -n default --timeout=300s

# Verify deployment
kubectl get pods,svc,deploy -n default

# Check service endpoints
kubectl get endpoints -n default

# Test the application (optional)
kubectl port-forward svc/guestbook-frontend 8080:80 -n default
# Access http://localhost:8080 in your browser
# You should see the guestbook application
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

# Check what resources were exported
kubectl get all -n default
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

# Edit the files manually to change namespace from 'default' to 'guestbook-app'
# Example: sed -i 's/namespace: default/namespace: guestbook-app/g' crane/transform/*.yaml

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
kubectl wait --for=condition=ready pod -l app=guestbook -n default --timeout=300s
kubectl wait --for=condition=ready pod -l app=redis -n default --timeout=300s

# Verify migration
kubectl get pods,svc,deploy -n default

# Check the status of all resources
kubectl get all -n default

# Verify service endpoints
kubectl get endpoints -n default
```

#### 6. Verify Migration and Test Connectivity

```bash
# Test the migrated application
kubectl port-forward svc/guestbook-frontend 8081:80 -n default

# Access http://localhost:8081 in your browser
# You should see the guestbook application working

# Test Redis connectivity (optional)
kubectl port-forward svc/redis-master 6379:6379 -n default
# In another terminal, test Redis connection
# redis-cli -h localhost -p 6379 ping
# Should return PONG

# Stop port-forward when done
```

#### 7. Advanced Testing: Service Connectivity

```bash
# Test internal service communication
kubectl run test-pod --image=busybox --rm -it --restart=Never -- nslookup redis-master.default.svc.cluster.local

# Test service endpoints
kubectl get endpoints redis-master -n default
kubectl get endpoints redis-slave -n default
kubectl get endpoints guestbook-frontend -n default

# Check service selectors
kubectl get svc -n default -o wide
```

### Cleanup

```bash
# Delete clusters when done
kind delete cluster --name source-cluster
kind delete cluster --name destination-cluster

# Verify clusters are deleted
kind get clusters
```

---

### Q: How does Crane handle service endpoints during migration?

**A:** Crane preserves service endpoints by:
- **Exporting Services**: Captures service definitions with selectors
- **Exporting Pods**: Captures pod labels and metadata
- **Maintaining Relationships**: Preserves selector-label relationships
- **Recreating Endpoints**: Kubernetes automatically recreates endpoints when services and pods are applied

### Q: What happens to service DNS names during migration?

**A:** Service DNS names are preserved because:
- **Internal DNS**: `redis-master.default.svc.cluster.local` remains the same
- **Service Names**: Service names are preserved in the destination
- **Namespace Handling**: DNS names adapt to namespace changes if transformed
- **Application Compatibility**: Applications continue to work without DNS changes