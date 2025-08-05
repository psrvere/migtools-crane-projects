# Mini Projects to Master Crane and Crane-Lib

*Suggested by Grok*

To master the **Crane CLI** (from [https://github.com/migtools/crane](https://github.com/migtools/crane), a tool for migrating Kubernetes workloads and state between clusters) and **Crane-Lib** (from [https://github.com/konveyor/crane-lib](https://github.com/konveyor/crane-lib), a Go library handling transformations and state transfers), I've curated 12 mini projects. These progress from basic usage of the Crane CLI to advanced integrations, custom extensions, and diving into the codebase and architecture.

Each project focuses on practical application to build hands-on understanding of:
- **Usage** (e.g., commands like export, transform, apply)
- **Architecture** (e.g., pipeline model with plugins, state transfer components)
- **Codebase** (e.g., Go packages for transforms and transfers)

**Projects 1-6** emphasize Crane CLI usage and basic architecture.  
**Projects 7-9** explore extensions and integrations.  
**Projects 10-12** delve into Crane-Lib's internals for custom development.

---

## 1. Basic Stateless Pod Migration

**Setup:** Set up two local Kubernetes clusters (using Minikube or Kind). Deploy a simple Nginx pod in the source cluster. Use Crane to export the resources, transform them (e.g., for namespace changes), and apply to the destination. Run: `crane export -n default`, `crane transform`, `crane apply`.

**What you'll learn:** Core CLI workflow (export-transform-apply pipeline), basic manifest handling, and non-destructive operations. Reference the guestbook example in the Crane repo for guidance.

---

## 2. Migrating a Deployment with Services

**Setup:** Deploy a sample guestbook app (Redis + frontend) as a Deployment and Service in one cluster. Migrate it using Crane, focusing on how services and endpoints are preserved during transformation. Test connectivity post-migration.

**What you'll learn:** Handling multi-resource apps, understanding configuration drift detection in the architecture, and CLI flags like `--plugins` for basic Kubernetes transformations.

---

## 3. Persistent Volume (PV) Migration with Rsync

**Setup:** Create a stateful app (e.g., a MySQL pod with PV) in the source cluster. Use Crane's state transfer to migrate data via rsync. Configure endpoints and transports as per the architecture.

**What you'll learn:** State migration components (utility like rsync, secure transport), PV handling in the codebase, and repeatability for auditing.

---

## 4. Namespace Remapping During Migration

**Setup:** Migrate a namespace-specific app (e.g., a ConfigMap-heavy workload) from "dev" to "prod" namespace. Use Crane's transform step with built-in plugins to update metadata.

**What you'll learn:** Transformation logic in the architecture (JSONPatch outputs), how Crane promotes GitOps by generating versionable YAML, and limitations like incomplete namespace features in early versions.

---

## 5. OpenShift-Specific Migration

**Setup:** Set up source as plain Kubernetes and destination as OpenShift (using CRC or Minishift). Migrate an app using the Crane OpenShift plugin. Handle customizations like routes vs. ingresses.

**What you'll learn:** Plugin ecosystem in the architecture (e.g., crane-plugin-openshift), cross-platform adaptations, and usage in real-world scenarios like Kubernetes to OpenShift.

---

## 6. Multi-Namespace App Migration

**Setup:** Deploy an app spanning multiple namespaces (e.g., backend in ns1, frontend in ns2). Use Crane to export selectively and transform for consolidation into one namespace.

**What you'll learn:** Advanced filtering in export (e.g., via labels), architecture's modularity for complex workloads, and best practices for onboarding to GitOps.

---

## 7. Integrating Crane with Tekton Pipelines

**Setup:** Install Tekton in your cluster. Use crane-runner examples to build a pipeline that automates Crane's export-transform-apply for a sample app. Trigger it via CI/CD.

**What you'll learn:** Broader architecture (Crane as part of Konveyor's ecosystem with crane-runner for orchestration), repeatability in workflows, and how to chain tasks.

---

## 8. Writing a Custom Transformation Plugin

**Setup:** Fork the crane-plugins repo. Create a simple plugin (in Go) to transform resource quotas during migration. Integrate it with Crane CLI and test on a quota-limited app.

**What you'll learn:** Plugin architecture (how crane-lib is used for transforms), codebase structure (e.g., transform package interfaces), and extending the tool.

---

## 9. Migrating from Docker Compose Using Related Tools

**Setup:** Start with a Docker Compose app (e.g., from move2kube-demos). Use Move2Kube to generate Kubernetes manifests, then Crane to migrate between clusters.

**What you'll learn:** Crane's role in the Konveyor suite (interoperability with tools like Move2Kube), end-to-end migration planning, and architecture for hybrid sources.

---

## 10. Custom Go Script Using Crane-Lib's Transform Package

**Setup:** Write a Go program that imports `github.com/konveyor/crane-lib/transform`. Load sample YAML manifests, apply a transformation (e.g., patch annotations), and output results. Build and run it standalone.

**What you'll learn:** Crane-Lib's API (e.g., transform functions for resource modification), codebase internals (standard library integration), and building custom tools without the CLI.

---

## 11. State Transfer Utility with Crane-Lib

**Setup:** Create a Go app using `github.com/konveyor/crane-lib/state_transfer`. Set up a transfer between two clusters: configure rsync utility, secure transport, and endpoints. Test with dummy data.

**What you'll learn:** State transfer architecture (components like utilities, transports, endpoints), lib's modularity for PV migrations, and error handling in code.

---

## 12. Extending Crane-Lib for a Custom Migration Tool

**Setup:** Build a minimal Go CLI that uses both transform and state_transfer packages from Crane-Lib. Add logic to migrate a specific resource type (e.g., StatefulSets) with custom logging. Compare it to the official Crane CLI.

**What you'll learn:** Full lib integration, understanding the codebase's dependencies (e.g., on backube/pvc-transfer), and how the lib powers the CLI's architecture for extensibility.