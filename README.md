
---

# Argo CD ApplicationSets Explained | Scaling GitOps Across Clusters & Microservices

## Video reference for this lecture is the following:

---

## ⭐ Support the Project

If this **repository** helps you, give it a ⭐ to show your support and help others discover it!

---

## Table of Contents

* [Introduction](https://www.google.com/search?q=%2523introduction&utm_source=gemini)
* [Why ApplicationSets?](https://www.google.com/search?q=%2523why-applicationsets&utm_source=gemini)
* [What is Multi-Cluster in Kubernetes?](https://www.google.com/search?q=%2523what-is-multi-cluster-in-kubernetes&utm_source=gemini)
* [Why Plain Argo CD Applications Do Not Scale](https://www.google.com/search?q=%2523why-plain-argo-cd-applications-do-not-scale&utm_source=gemini)
* [What are ApplicationSets?](https://www.google.com/search?q=%2523what-are-applicationsets&utm_source=gemini)
* [Responsibilities of ApplicationSets](https://www.google.com/search?q=%2523responsibilities-of-applicationsets&utm_source=gemini)
* [Demo 1: Multi-Region Application Deployment Using Argo CD ApplicationSets](https://www.google.com/search?q=%2523demo-1-multi-region-application-deployment-using-argo-cd-applicationsets&utm_source=gemini)
* [Step 1: Create Local Kubernetes Clusters Using Kind](https://www.google.com/search?q=%2523step-1-create-local-kubernetes-clusters-using-kind&utm_source=gemini)
* [Step 2: Install and Prepare Argo CD in the Hub Cluster](https://www.google.com/search?q=%2523step-2-install-and-prepare-argo-cd-in-the-hub-cluster&utm_source=gemini)
* [Step 3: Onboard Spoke Clusters into Argo CD](https://www.google.com/search?q=%2523step-3-onboard-spoke-clusters-into-argo-cd&utm_source=gemini)
* [Step 4: Create the Git Repository and Application Structure](https://www.google.com/search?q=%2523step-4-create-the-git-repository-and-application-structure&utm_source=gemini)
* [Step 5: Understanding and Applying ApplicationSet YAML](https://www.google.com/search?q=%2523step-5-understanding-and-applying-applicationset-yaml&utm_source=gemini)


* [Other Generator Types](https://www.google.com/search?q=%2523other-generator-types&utm_source=gemini)
* [Cluster Generator (Demo 2)](https://www.google.com/search?q=%25231-cluster-generator-demo-2&utm_source=gemini)
* [Git Generator (Directories) (Demo 3)](https://www.google.com/search?q=%25232-git-generator-directories-demo-3&utm_source=gemini)
* [Git Generator (Files) (Demo 4)](https://www.google.com/search?q=%25233-git-generator-files-demo-4&utm_source=gemini)
* [Matrix Generator (Demo 5)](https://www.google.com/search?q=%25234-matrix-generator-demo-5&utm_source=gemini)


* [Choosing the Right ApplicationSet Generator](https://www.google.com/search?q=%2523choosing-the-right-applicationset-generator&utm_source=gemini)
* [Other ApplicationSet Generators (for completeness)](https://www.google.com/search?q=%2523other-applicationset-generators-for-completeness&utm_source=gemini)
* [Conclusion](https://www.google.com/search?q=%2523conclusion&utm_source=gemini)
* [References](https://www.google.com/search?q=%2523references&utm_source=gemini)

---

## Why ApplicationSets?

To understand why ApplicationSets exist, we must first understand **multi-cluster Kubernetes** and the operational pressure it creates on GitOps.

---

## What is Multi-Cluster in Kubernetes?

Multi-cluster is a **generic term** that simply means operating more than one Kubernetes cluster.
However, in this context, multi-cluster refers specifically to **multiple Kubernetes clusters that are logically related and managed together**.

These clusters are independent from an infrastructure perspective, but they are connected by a **shared operational intent**. For example, the same application may be deployed across clusters for environment separation, high availability, or disaster recovery.

Each cluster still has its own control plane, nodes, networking, and lifecycle.

Multi-cluster here is **not about unrelated or isolated clusters** that have no operational relationship with each other. It is about **coordinating deployments and configurations across a defined set of clusters** in a consistent and repeatable way.

#### Why teams adopt multi-cluster

* **Failure Domain Isolation**
A single cluster represents a shared blast radius. Control plane failures, misconfigurations, or cluster-level issues can affect all workloads at once. Multiple clusters isolate failures and improve overall system resilience.
* **Environment Separation**
Development, staging, and production environments are commonly isolated into separate clusters. This provides stronger security boundaries, clearer operational ownership, and safer promotion of changes across environments. In addition to application workloads, each cluster also requires a standard set of platform components such as ingress controllers, monitoring, logging, and security tooling.
* **Geographic and Latency Requirements**
Applications serving users across regions often run workloads closer to users. Deploying the same application in multiple clusters improves latency and availability without coupling regions operationally.
* **Compliance and Regulatory Needs**
Certain workloads must run in specific regions or controlled environments. Multi-cluster architectures make it possible to satisfy regulatory constraints while keeping deployment models consistent.

As organizations grow, new clusters are added regularly to support scale, isolation, or regional expansion. These clusters must be brought online quickly with a predictable baseline of applications and platform services, without relying on manual configuration.

---

## Why Plain Argo CD Applications Do Not Scale

At this stage, GitOps complexity increases significantly.

#### Operational challenges without ApplicationSets

* **Argo CD Application YAML Explosion**
In multi-environment or multi-cluster setups, the same *application workload* needs to be deployed repeatedly with only minor differences such as cluster destination, namespace, or values files. Without ApplicationSets, each deployment requires a separate **Argo CD Application** manifest, leading to duplicated YAML and increased maintenance effort.
* **Manual Multi-Cluster Wiring**
Argo CD supports multi-cluster deployments, but without ApplicationSets, each cluster must be explicitly wired using individual **Argo CD Application** definitions. Adding a new cluster often means copying existing YAML, modifying destinations, and manually validating changes. This approach does not scale and introduces human error.
* **Configuration Drift at Scale**
When many **Argo CD Application** manifests are manually maintained, it becomes easy for configurations to diverge over time. Small changes applied to one **Argo CD Application** may not be consistently propagated, resulting in drift across clusters and environments, undermining the promise of GitOps.
* **Static GitOps in a Dynamic World**
Real-world platforms are dynamic. Clusters are added or removed, environments evolve, and repository structures change. Plain **Argo CD Applications** and even the *App of Apps* pattern rely on static definitions and cannot react automatically to these changes without manual intervention.

ApplicationSets address these problems by making **Argo CD Application creation itself declarative, dynamic, and scalable**.

> ApplicationSets are not mandatory for GitOps, but they become valuable whenever GitOps must scale across **repeated deployment patterns**.

> **Note:** ApplicationSets are commonly used in multi-cluster and multi-environment setups, but their value is not limited to those scenarios.
> They are equally useful when managing **multiple similar Applications within a single cluster**, such as microservices, tenants, or environment variants.
> Their real strength appears wherever **deployment intent must be expressed once and applied repeatedly**, whether across clusters, environments, repositories, or services.

---

## What are ApplicationSets?

An **ApplicationSet** is an Argo CD custom resource that defines **how Argo CD Applications should be created at scale**.

Instead of manually defining each **Argo CD Application**, an ApplicationSet describes:

* *where application definitions come from*
* *how they should be templated*
* *when they should be created or removed*

An ApplicationSet itself does **not deploy workloads**.
Its sole purpose is to **generate and manage Argo CD Applications declaratively**.

By taking responsibility for **Application creation itself**, ApplicationSets transform multi-cluster GitOps from a *manual and error-prone process* into a **declarative and scalable system**.

> ApplicationSets define **what Argo CD Applications should exist**, and Argo CD ensures they always do.

---

## Responsibilities of ApplicationSets

Once you understand what an ApplicationSet is, its responsibilities become clear.

#### How ApplicationSets work

* **Define Desired Argo CD Applications**
ApplicationSets let you declare **which Argo CD Applications should exist**, rather than manually creating them one by one. This shifts GitOps from managing deployments to managing **application intent**.
* **Generate Argo CD Applications from Rules**
An ApplicationSet uses generators to produce multiple parameter sets and combines them with a template. Each rendered output results in a concrete **Argo CD Application** resource.
* **Template Application Definitions**
The template section looks similar to a standard Argo CD Application but supports variables. This allows the same application definition to be reused across clusters, environments, or regions with controlled variation.
* **Continuously Reconcile Application State**
The ApplicationSet controller continuously evaluates generator inputs. When clusters are added, Git paths change, or entries are removed, the corresponding **Argo CD Applications are created, updated, or deleted automatically**.
* **Enable Scalable Multi-Cluster GitOps**
By combining generators with templated destinations, ApplicationSets make it possible to manage many related clusters using a single declarative definition, without duplicating Application manifests.

---

## Demo 1: Multi-Region Application Deployment Using Argo CD ApplicationSets

---

## Demo Introduction

In this demo, we will work with an application called **app1** modeled across multiple environments/regions.

To build a robust multi-cluster lab environment locally, we deploy app1 across a **Hub-and-Spoke topology**:

* **Hub Cluster (`kind-argocd-hub`)** – runs the Argo CD control plane and ApplicationSet controller.
* **Spoke Cluster 1 (`mumbai`)** – target environment simulating a regional deployment.
* **Spoke Cluster 2 (`nvirginia`)** – target environment simulating a secondary regional deployment.

To manage this multi-cluster deployment, we will use **Argo CD ApplicationSets** and follow a **GitOps-driven approach**.

Instead of manually creating and managing separate Argo CD Applications for each cluster, we will define a **single ApplicationSet** that automatically generates and manages cluster-specific Applications.

---

## Demo Prerequisites

Before proceeding, ensure the following tools are available:

* **Kind (Kubernetes in Docker)** or access to Kubernetes clusters.
* **kubectl** configured with multiple contexts (`kind-argocd-hub`, `mumbai`, `nvirginia`).
* **Helm** for installing Argo CD.
* **Argo CD CLI** for command-line interaction.

---

## Step 1: Create Local Kubernetes Clusters Using Kind

To simulate a production-grade multi-region environment locally, we create three Kind clusters: one central management hub and two regional spoke clusters.

```bash
kind create cluster --name argocd-hub
kind create cluster --name mumbai
kind create cluster --name nvirginia

```

Switch to your hub cluster context:

```bash
kubectl config use-context kind-argocd-hub

```

---

## Step 2: Install and Prepare Argo CD in the Hub Cluster

1. Create the Argo CD namespace on the hub cluster:
```bash
kubectl create namespace argocd

```


2. Install Argo CD using official manifests or Helm:
```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

```


3. Access the Argo CD UI via port-forwarding:
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443

```



---

## Step 3: Onboard Spoke Clusters into Argo CD

To allow the hub Argo CD instance to securely manage workloads on the spoke clusters (`mumbai` and `nvirginia`):

1. **Create Service Accounts & RBAC:** Apply a service account (`argocd-manager`) with cluster-admin permissions on each spoke cluster.
2. **Generate Tokens:** Extract service account bearer tokens.
3. **Register via Secrets on the Hub:** Create Kubernetes secret objects inside the `argocd` namespace on the hub cluster mapping the spoke API endpoints and credentials.
4. **Label Cluster Secrets:** Add custom metadata labels to enable selector-based discovery by ApplicationSet generators:
```bash
kubectl label secret cluster-mumbai app=app1 region=mumbai -n argocd --overwrite
kubectl label secret cluster-nvirginia app=app1 region=nvirginia -n argocd --overwrite

```



Verify successful registration:

```bash
argocd cluster list

```

---

## Step 4: Create the Git Repository and Application Structure

We utilize a dedicated configuration repository (`emmanuel-adekiitan/appset-config.git`) structured with a nested path layout:

```text
02-app1-config-repo/
├── mumbai/
│   └── manifests/
│       ├── 01-cm.yaml
│       ├── 02-svc.yaml
│       └── 03-deploy.yaml
└── nvirginia/
    └── manifests/
        ├── 01-cm.yaml
        ├── 02-svc.yaml
        └── 03-deploy.yaml

```

Each `manifests/` directory contains the complete Kubernetes manifests required to deploy the application into that target environment.

---

## Step 5: Understanding and Applying ApplicationSet YAML (Generators and Templates)

An **ApplicationSet** is a Kubernetes custom resource that declares **what Argo CD Applications should exist**, using **generators** to produce parameters and **templates** to render concrete Argo CD Applications.

### ApplicationSet Manifest Example (List Generator)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: app1-multi-region
  namespace: argocd
spec:
  goTemplate: true
  goTemplateOptions:
    - missingkey=error

  generators:
  - list:
      elements:
      - region: mumbai
        destinationName: mumbai
        namespace: app1-mumbai-ns
      - region: nvirginia
        destinationName: nvirginia
        namespace: app1-nvirginia-ns

  template:
    metadata:
      name: "app1-{{.region}}"
    spec:
      project: default

      source:
        repoURL: https://github.com/emmanuel-adekiitan/appset-config.git
        targetRevision: main
        path: "02-app1-config-repo/{{.region}}/manifests"

      destination:
        name: "{{.destinationName}}"
        namespace: "{{.namespace}}"

      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        syncOptions:
          - CreateNamespace=true

```

---

## Other Generator Types

Beyond the List generator, Argo CD ApplicationSets support powerful dynamic generators tested in our lab:

### 1. Cluster Generator

Automatically discovers registered target clusters dynamically using Kubernetes label selectors (`matchLabels: { app: app1 }`) matched against hub cluster secrets.

### 2. Git Generator (Directories)

Scans repository subdirectories automatically using wildcards (e.g., `path: "02-app1-config-repo/*"`) and evaluates Go templates like `{{.path.path}}` and `{{.path.basename}}` to handle nested repository structures seamlessly.

### 3. Matrix Generator

Computes the Cartesian product across multiple generator dimensions (such as multiplying Git configuration directories across target cluster lists) to generate full multi-region deployments automatically.

---

## Choosing the Right ApplicationSet Generator

* **List Generator:** Best for simple, explicit, static mappings.
* **Cluster Generator:** Ideal when scaling across many dynamic clusters using label governance.
* **Git Generator (Directories):** Perfect for monorepos structured by environment or microservice.
* **Matrix Generator:** Unlocks advanced combinations (e.g., deploying every app version to every cluster).

---

## Conclusion

By leveraging Argo CD ApplicationSets in a Hub-and-Spoke architecture, multi-cluster GitOps becomes scalable, declarative, and resilient against configuration drift.

---

## References

* [Argo CD Documentation](https://www.google.com/search?q=https://argo-cd.readthedocs.io/&utm_source=gemini)
* [Argo CD ApplicationSet Specification](https://www.google.com/search?q=https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/&utm_source=gemini)
* [Kind (Kubernetes in Docker)](https://www.google.com/search?q=https://kind.sigs.k8s.io/&utm_source=gemini)