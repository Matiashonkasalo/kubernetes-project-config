# Project Configuration Repository

This repository contains all Kubernetes and GitOps configuration for the TODO application.
The application consists of a frontend, backend, broadcaster service, PostgreSQL database,
and supporting jobs and routing components. All deployment logic is managed with **Kustomize**
and **ArgoCD**, following a clear base/overlay structure.

---

## 📁 Repository Structure

### **1. ArgoCD Applications**
Located in `argocd/`.

These manifest files tell ArgoCD how to deploy the project to
different environments. They point to the staging and production overlays and enable
automatic syncing, pruning, and self-healing.

Environments:
- **todo-staging**
- **todo-production**

---

### **2. Kustomize Base**
Located in `environments/base/`.

This folder defines the shared configuration used by all environments:

- **Backend**
  - Deployment, Service, ConfigMap
  - PostgreSQL StatefulSet + PVC
  - CronJobs (DB backup, random todo generator)

- **Frontend**
  - Deployment, Service, ConfigMap
  - Gateway + HTTPRoute
  - PVC for image caching

- **Broadcaster**
  - Deployment and config for consuming NATS events

The base defines the application's full architecture independently of environment-specific settings.

---

### **3. Environment Overlays**
Located in `environments/overlays/`.

These modify the base for specific environments:

#### **Staging**
- Enables "log_only" mode in broadcaster
- Changes database connection URL to staging DB
- Removes backup and random-todo CronJobs
- Overrides images with staging tags
- Custom HTTPRoute patch

#### **Production**
- Uses production PostgreSQL connection
- Adds production-specific Gateway config
- Overrides images with production tags

Kustomize composes each environment by applying patches and image overrides on top of the shared base.

---

### **4. Storage Configuration**
Located in `storage/`.

Includes:
- PersistentVolume
- PersistentVolumeClaim
- Additional project-specific storage definitions

Used for:
- PostgreSQL persistence
- Frontend image caching

---

## Deployment Flow (GitOps)

1. The **code repository** builds Docker images.
2. Updated image tags are applied to Kustomize overlays.
3. ArgoCD monitors this repository and automatically:
   - pulls changes
   - updates Kubernetes resources
   - prunes outdated objects
   - ensures cluster state matches Git (self-heal)

Git is the single source of truth for all deployments.

---

## Summary

This repository holds every configuration file necessary to run the entire TODO platform
in a Kubernetes cluster. It separates concerns cleanly by storing only declarative
infrastructure and deployment logic, allowing the application code to evolve independently
in a separate repository.
