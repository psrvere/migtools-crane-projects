# Crane Ecosystem

## Core Components

### konveyor/crane
**Main CLI tool** for migrating Kubernetes workloads between clusters. Part of the Konveyor open-source community focused on application modernization and migration.

**Key Features:**
- **Three-phase pipeline**: Export → Transform → Apply
- **Non-destructive operations**: Inspects running applications without altering live workloads
- **Configuration drift handling**: Addresses differences between intended and actual state
- **GitOps integration**: Supports versioning migration artifacts
- **Persistent volume migrations**: Integrates with tools like backube/pvc-transfer

**Workflow:**
1. **Export**: Extract resources from source cluster as YAML manifests
2. **Transform**: Apply plugins to clean cluster-specific artifacts (UIDs, timestamps)
3. **Apply**: Deploy transformed manifests to destination cluster

### konveyor/crane-lib
**Core transformation engine** - the official library that powers Crane's transform phase. Written in Go, it provides the foundational engine for processing and modifying exported Kubernetes manifests.

**Key Capabilities:**
- **Built-in transformations**: JSONPatch operations to remove cluster-bound metadata
- **Plugin framework**: Infrastructure for loading and executing plugins
- **Core library**: Reusable transformation logic for the Crane ecosystem
- **Official support**: Maintained by the Crane core team

### konveyor/crane-plugins
**Community plugin repository** - a collection of user-contributed transformation plugins. Provides extensible components for specific migration scenarios beyond the built-in capabilities.

**Structure:**
- **index.yml**: Catalog of available community plugins
- **plugins directory**: Individual plugin manifests and binaries
- **Community-driven**: Plugins contributed by users and organizations

**Usage:** Extends Crane's transformation capabilities with specialized plugins for different environments, cloud providers, or custom requirements.

### konveyor/crane-plugin-openshift
**OpenShift-specific plugin** for handling Red Hat's enterprise Kubernetes platform migrations. Ensures smooth transitions between OpenShift clusters or from vanilla Kubernetes to OpenShift.

**Key Transformations:**
- **OpenShift-specific patches**: Removes or converts fields unique to OpenShift (build configs, deployment configs, security constraints)
- **Compatibility enhancements**: Ensures YAML adheres to OpenShift's stricter policies (RBAC, network security)
- **Reference implementation**: Serves as example for building similar plugins

**Integration:** Optional plugin that can be specified during Crane's transform phase via flags or configuration.

### Q: What is configuration drift?

**A:** Configuration drift occurs when your actual running application differs from your intended/declarative configuration. It's a common problem where the real state of your infrastructure doesn't match your version-controlled manifests.

### Q: How does configuration drift happen?

**A:** Configuration drift typically occurs through:
- **Manual changes**: Direct kubectl commands that aren't committed to Git
- **Emergency fixes**: Quick patches during incidents that aren't documented
- **Direct edits**: Using `kubectl edit` without updating manifests
- **Time-based changes**: Gradual modifications over time without proper documentation

### Q: Why is configuration drift a problem for migrations?

**A:** Configuration drift creates several migration challenges:
- **Uncertainty**: You don't know what the "real" configuration should be
- **Reproducibility**: Can't recreate the exact same environment
- **Risk**: Important changes might be lost during migration
- **Compliance**: Difficult to audit what changes were made and why

### Q: How does Crane help with configuration drift?

**A:** Crane addresses configuration drift by:
- **Discovering actual state**: `crane export` captures the real running configuration
- **Reconstructing manifests**: Creates redeployable YAML from actual state
- **Bridging the gap**: Provides the real configuration for migration
- **Source of truth**: Gives you the actual running state, not outdated manifests

### Q: What's the difference between Git manifests and Crane-exported resources?

**A:** 
- **Git manifests**: What you intended to deploy (may be outdated)
- **Crane exported**: What's actually running in the cluster (current reality)
- **Migration approach**: Use Crane to capture real state, then migrate that state 


### Q: What does the built-in Kubernetes plugin do?

**A:** The built-in Kubernetes plugin automatically:
- **Removes cluster-specific metadata**: `metadata.uid`, `metadata.resourceVersion`, `metadata.creationTimestamp`
- **Cleans up resource versions**: Ensures resources can be redeployed
- **Handles basic transformations**: Namespace changes and standard Kubernetes cleanup
- **Generates redeployable manifests**: Creates clean YAML files ready for destination cluster