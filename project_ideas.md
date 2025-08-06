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

**What you'll learn:** Handling multi-resource apps, understanding service and endpoint preservation during migration, and basic Crane CLI workflow for Kubernetes resource transformations.

---