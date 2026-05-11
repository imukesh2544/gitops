# 🚀 GitOps & Argo CD — Complete Beginner's Guide

> **Course:** GitOps & Argo CD Crash Course — EP 1
> **Topic:** What is GitOps? Why does it matter in Production?
> **Level:** Beginner to Intermediate

---

## 📌 Table of Contents

- [What is GitOps?](#-what-is-gitops)
- [The Real Problem GitOps Solves](#-the-real-problem-gitops-solves)
- [The 4 Core Principles of GitOps](#-the-4-core-principles-of-gitops)
- [How GitOps Works — The Workflow](#-how-gitops-works--the-workflow)
- [GitOps in Real Production Scenarios](#-gitops-in-real-production-scenarios)
- [Key Advantages of GitOps](#-key-advantages-of-gitops)
- [Tools & Technologies](#-tools--technologies)
- [Important Tips for Teams](#-important-tips-for-teams)
- [Summary — Key Takeaways](#-summary--key-takeaways)

---

## 🤔 What is GitOps?

> **Simple Definition:**
> GitOps = Using **Git as the single source of truth** to manage and deliver both your **applications AND your infrastructure**.

Think of it this way:

Imagine your entire company's server setup — all the configurations, deployments, and settings — are written down in a notebook. GitOps says:

> *"Whatever is written in that notebook is the law. The real system must always match the notebook, no exceptions."*

That **notebook is your Git repository**.

- If the Git repo says `replicas: 3`, your cluster will always run **3 pods** — no matter what.
- If someone manually changes it to `replicas: 1` on the server, GitOps will **automatically correct it back to 3**.

> ⚠️ **Note:** While the name says "Git", technically any **versioned and immutable** storage (like AWS S3 buckets) can act as this source of truth.

---

## 🔥 The Real Problem GitOps Solves

### The Traditional CD Problem — A Production Story

Imagine you're a DevOps engineer at a startup. One day, your **Kubernetes cluster** starts behaving unexpectedly. Pods are crashing, resources are misconfigured.

You ask your team:
- ❓ *"Who changed the node configuration?"*
- ❓ *"What was the previous setting?"*
- ❓ *"When did this change happen?"*

**Nobody knows.** 😰

This is the core problem with **traditional Continuous Delivery (CD)**:

| Area | CI (Continuous Integration) | Traditional CD |
|---|---|---|
| Source Code Changes | ✅ Tracked via Git & PRs | ❌ Often manual |
| Infrastructure Changes | ✅ Code is versioned | ❌ Done via `kubectl` or shell scripts |
| Audit Trail | ✅ Full history in Git | ❌ No history, no rollback |
| Who changed what? | ✅ Git blame / PR history | ❌ Unknown |

### The Gap in Simple Terms

```
Developer writes code  →  Git tracks it  ✅
DevOps changes cluster →  Nothing tracks it ❌
```

**GitOps closes this gap** by bringing the same discipline of CI (tracking, versioning, reviewing) to the **infrastructure and deployment side.**

---

## 📐 The 4 Core Principles of GitOps

For a system to truly be called "GitOps," it must follow these **4 non-negotiable principles:**

---

### 1️⃣ Declarative

> *"Describe WHAT you want, not HOW to do it."*

Instead of writing a script that says:

```bash
# Imperative (Traditional way)
kubectl scale deployment my-app --replicas=3
kubectl set image deployment/my-app my-app=nginx:1.21
```

You write a **YAML manifest** that declares the desired state:

```yaml
# Declarative (GitOps way)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: my-app
        image: nginx:1.21
```

> ✅ **Production Benefit:** Everyone on your team can open the YAML file and instantly know the exact state of the system — no guesswork.

---

### 2️⃣ Versioned and Immutable

> *"Every change must have a history."*

Every time you update your YAML file in Git:
- A new **commit** is created
- The **old state is preserved** in history
- You can **roll back** to any previous state with a single command

```bash
# Roll back to a previous working state
git revert <commit-hash>
```

> ✅ **Production Benefit:** When something breaks at 2 AM, you don't panic. You find the last working commit and revert. Done.

---

### 3️⃣ Automatically Pulled

> *"The system should automatically detect and apply changes — no manual deployments."*

Once a change is **merged into Git**, the GitOps controller (e.g., Argo CD) automatically:
1. Detects the new change
2. Pulls the updated manifest
3. Applies it to the cluster

This can happen via:
- **Pull Mechanism** — The controller actively watches the Git repo at regular intervals
- **Push Mechanism** — Git sends a webhook notification to the controller on every merge

> ✅ **Production Benefit:** No engineer needs to manually run `kubectl apply` after every deployment. The system handles it automatically, reducing human error.

---

### 4️⃣ Continuously Reconciled

> *"The controller constantly watches for drift and corrects it."*

The GitOps controller runs a **continuous loop:**

```
Every few seconds:
  Compare → Git State vs Live Cluster State
  If Difference Found → Override Cluster to Match Git
  If No Difference   → Do Nothing
```

This is called the **Reconciliation Loop.**

> ✅ **Production Benefit:** Even if a hacker or a misconfigured script changes something in your cluster, it gets automatically reverted. Your system is **self-healing.**

---

## 🔄 How GitOps Works — The Workflow

Here is the **end-to-end GitOps workflow** in a real team environment:

```
┌─────────────────────────────────────────────────────────────┐
│                      GitOps Workflow                        │
│                                                             │
│  1. DevOps Engineer updates pod.yaml in Git repo            │
│                         ↓                                   │
│  2. Raises a Pull Request (PR) for review                   │
│                         ↓                                   │
│  3. Team reviews → Approves → Merges into main branch       │
│                         ↓                                   │
│  4. GitOps Controller (Argo CD / Flux) detects the change   │
│                         ↓                                   │
│  5. Controller compares:                                    │
│         [Git State]  vs  [Live Cluster State]               │
│                         ↓                                   │
│  6. If Drift Found → Apply Git state to the Cluster         │
│     If No Drift    → Do Nothing                             │
└─────────────────────────────────────────────────────────────┘
```

### 🗂️ What Does the Controller Actually Do?

The GitOps controller maintains **two caches:**

| Cache | What It Stores |
|---|---|
| 📁 **Desired State Cache** | The state defined in your Git repository (YAMLs, Helm charts) |
| 📁 **Live State Cache** | The actual current state of your Kubernetes cluster |

It **continuously compares** these two. Any mismatch = **automatic correction.**

---

## 🏭 GitOps in Real Production Scenarios

### Scenario 1 — The Accidental Manual Change

```
🔴 Problem:
A junior engineer accidentally runs:
  kubectl delete deployment payment-service

🟢 GitOps Solution:
The Argo CD controller detects the drift within seconds.
It reads the Git repo → payment-service should exist.
It re-creates the deployment automatically.
No downtime. No alert needed. System heals itself.
```

---

### Scenario 2 — Managing 500 Clusters

```
🔴 Problem:
Your company has 500 Kubernetes clusters across 10 regions.
Manually updating configurations across all is impossible.

🟢 GitOps Solution:
Update one YAML file in Git.
The GitOps controller propagates the change across
all 500 clusters automatically.
One source of truth. Zero manual effort.
```

---

### Scenario 3 — Security Breach Attempt

```
🔴 Problem:
A hacker gains partial access and changes a
container image to a malicious one.

🟢 GitOps Solution:
The controller detects the image has drifted from
what is declared in Git.
It immediately overrides the change and restores
the approved image.
Your system is protected automatically.
```

---

## ✅ Key Advantages of GitOps

| Advantage | Description |
|---|---|
| 🔒 **Enhanced Security** | Only the GitOps controller manages the cluster. Unauthorized changes are auto-reverted. |
| 🩺 **Auto-Healing** | Drift from desired state is automatically corrected. |
| 📋 **Full Audit Trail** | Every change is a Git commit — you always know who changed what and when. |
| ⚡ **Faster Recovery** | Roll back to any previous state instantly via `git revert`. |
| 📈 **Scalability** | Manage hundreds of clusters from a single Git repository. |
| 🤝 **Team Collaboration** | PRs enforce review and approval before anything reaches production. |
| 🌐 **Standardization** | Brings the same discipline to CD that CI has had for years. |

---

## 🛠️ Tools & Technologies

### GitOps Controllers

| Tool | Description |
|---|---|
| **Argo CD** | Most popular GitOps controller for Kubernetes. Focus of this course. |
| **Flux CD** | Lightweight GitOps controller, CNCF graduated project. |
| **Spinnaker** | Enterprise-grade CD platform with GitOps support. |

### CI Platforms (Used alongside GitOps)

| Tool | Role |
|---|---|
| **Jenkins** | Handles the CI part — build, test, push Docker image. |

### Infrastructure & Config Formats

| Tool | Role |
|---|---|
| **Kubernetes** | The primary platform GitOps controllers manage. |
| **YAML** | Declarative configuration format for Kubernetes resources. |
| **Helm** | Kubernetes package manager — used to template YAML manifests. |
| **AWS S3** | Can serve as an alternative immutable source of truth. |

### A Typical CI + GitOps CD Pipeline

```
Developer pushes code
        ↓
Jenkins (CI) → Builds Docker Image → Pushes to Registry
        ↓
Updates image tag in Git repo (YAML)
        ↓
Argo CD (GitOps CD) → Detects change → Deploys to Kubernetes
```

---

## 💡 Important Tips for Teams

> ⚠️ **Critical Warning:**
> The GitOps controller will **blindly deploy whatever is in Git.**
> If your Git repo has a broken or insecure manifest, it WILL be deployed.

**This means Git checks are non-negotiable:**

- ✅ Always use **Pull Requests** — never push directly to main
- ✅ Set up **automated linting and validation** on PRs (e.g., `kubeval`, `kube-score`)
- ✅ Require **at least one reviewer approval** before merge
- ✅ Use **branch protection rules** on your Git repository
- ✅ Think of GitOps **not just for apps** but for your **entire infrastructure** — nodes, namespaces, RBAC, network policies, everything

> 📖 **Want a vendor-neutral definition?**
> Visit the [OpenGitOps](https://opengitops.dev/) forum for platform-independent GitOps principles.

---

## 📝 Summary — Key Takeaways

```
GitOps is NOT just a deployment tool.
It is a fundamental shift in HOW we manage systems.
```

| Concept | Remember This |
|---|---|
| **GitOps** | Git = Single Source of Truth for apps + infrastructure |
| **Declarative** | Describe WHAT, not HOW |
| **Versioned** | Every change = a commit = full history |
| **Auto-Pulled** | No manual `kubectl apply` needed |
| **Reconciled** | System constantly self-heals back to Git state |
| **Argo CD** | The most popular GitOps controller for Kubernetes |
| **PR = Gate** | All changes must go through review before reaching production |

---

---
<img width="277" height="182" alt="download" src="https://github.com/user-attachments/assets/f7d6d8a1-2c14-42bd-980f-e536e69ae64c" />


#####################################################################################################################
#####################################################################################################################

# ⚙️ GitOps & Argo CD — Internal Architecture Deep Dive

> **Course:** GitOps & Argo CD Crash Course — EP 2
> **Topic:** How Argo CD Works Internally — Architecture & Components
> **Level:** Intermediate
> **Pre-requisite:** EP 1 — What is GitOps? *(Watch that first!)*

---

## 📌 Table of Contents

- [What is a GitOps Tool?](#-what-is-a-gitops-tool)
- [The Problem With Traditional CD](#-the-problem-with-traditional-cd)
- [Argo CD — Microservices Architecture Overview](#-argo-cd--microservices-architecture-overview)
- [Component Deep Dive](#-component-deep-dive)
- [How All Components Work Together](#-how-all-components-work-together)
- [GitOps Tools Comparison](#-gitops-tools-comparison)
- [Real Production Scenarios](#-real-production-scenarios)
- [Tips, Best Practices & Interview Prep](#-tips-best-practices--interview-prep)
- [Summary — Key Takeaways](#-summary--key-takeaways)

---

## 🤔 What is a GitOps Tool?

> **Simple Definition:**
> A GitOps tool is a mechanism that **constantly keeps your Kubernetes cluster in sync** with whatever is defined in your Git repository.

Think of it like a security guard at a warehouse:

> *"The guard has a list of what should be in the warehouse (Git). Every few seconds, he walks around and checks the actual warehouse (Kubernetes cluster). If anything is missing or changed without permission, he immediately fixes it."*

That **security guard is your GitOps tool** — Argo CD, Flux CD, etc.

Key terms you must know:

| Term | Meaning |
|---|---|
| **Single Source of Truth** | Git holds the authoritative desired state of your system |
| **Reconciliation Logic** | Constantly comparing Git state vs Cluster state and syncing them |
| **Drift** | When the live cluster deviates from what Git says it should be |
| **Auto-Healing** | Automatically correcting the cluster when drift is detected |

---

## 🔥 The Problem With Traditional CD

### The "Fire and Forget" Problem

In a traditional CI/CD pipeline using shell scripts or Python:

```
Script runs → Deploys to cluster → Script finishes → Script forgets ❌
```

What happens next?

```
🔴 Problem:
An engineer manually logs into the cluster and changes
a setting — maybe increases memory limits or deletes a pod.

The traditional CD script has NO idea this happened.
It already finished and moved on.
Nobody is watching the cluster anymore.
```

This means:
- ❌ No auto-healing
- ❌ No drift detection
- ❌ No way to know the cluster changed
- ❌ No automatic correction

### The GitOps Solution

```
GitOps Controller runs → Deploys to cluster → KEEPS WATCHING FOREVER ✅
```

```
🟢 GitOps Solution:
The Argo CD controller runs 24/7 as a persistent watchman.
It detects the manual change within seconds.
It compares it against Git → Git says it should be different.
It overrides the manual change and restores the correct state.
Cluster is back to desired state. Automatically.
```

---

## 🏗️ Argo CD — Microservices Architecture Overview

> Argo CD is NOT a single monolithic application.
> It is built as a **set of microservices** that each have a dedicated responsibility.

Here is the high-level architecture:

```
┌──────────────────────────────────────────────────────────────────┐
│                        Argo CD Architecture                      │
│                                                                  │
│   👤 User (Engineer)                                             │
│        ↓  (UI / CLI)                                             │
│   ┌─────────────┐                                                │
│   │  API Server │  ← Entry point for all human interaction       │
│   └──────┬──────┘                                                │
│          │                                                       │
│   ┌──────▼──────────────────────────┐                           │
│   │      Application Controller     │  ← The Brain 🧠           │
│   │  (Reconciliation Loop lives here)│                           │
│   └──────┬──────────────┬───────────┘                           │
│          │              │                                        │
│   ┌──────▼──────┐  ┌────▼────────┐                              │
│   │  Reposerver │  │  Kubernetes │                              │
│   │  (Git State)│  │   Cluster   │                              │
│   └─────────────┘  │  (Live State│                              │
│                    └─────────────┘                              │
│                                                                  │
│   ┌─────────────┐   ┌─────────────┐                             │
│   │     Dex     │   │    Redis    │                             │
│   │   (SSO/Auth)│   │   (Cache)   │                             │
│   └─────────────┘   └─────────────┘                             │
└──────────────────────────────────────────────────────────────────┘
```

---

## 🔬 Component Deep Dive

---

### 1️⃣ Reposerver — The Git Connector

> *"Its only job is to talk to Git and fetch the desired state."*

```
Git Repository
      ↓
  Reposerver  →  Fetches YAML manifests / Helm charts
      ↓
  Passes desired state to Application Controller
```

- Connects to your Git repository
- Retrieves the latest manifests (YAML, Helm, Kustomize)
- Passes this **desired state** to the Application Controller

> ✅ **Production Benefit:** By isolating Git communication into its own microservice, Argo CD can handle multiple repos efficiently without bottlenecks.

---

### 2️⃣ Application Controller — The Brain 🧠

> *"This is the most critical component. Everything happens here."*

This is where the **Reconciliation Loop** lives:

```
Every few seconds:

  Step 1 → Fetch Desired State from Reposerver (Git)
  Step 2 → Fetch Live State from Kubernetes Cluster
  Step 3 → Compare both states

  If MATCH    → System is "Green" ✅ → Do Nothing
  If MISMATCH → Drift Detected 🔴 → Sync cluster to match Git
```

- Monitors the live state of the Kubernetes cluster
- Gets desired state from the Reposerver
- Performs comparison and synchronization
- Triggers auto-healing when drift is detected

> ✅ **Production Benefit:** This continuous loop means your cluster is **never left in a broken state** for more than a few seconds.

> 🎯 **Interview Tip:** If asked *"How does Argo CD work internally?"* — start with the Application Controller and the reconciliation loop. This is what interviewers want to hear.

---

### 3️⃣ API Server — The Entry Point

> *"Think of this like the Amazon website — it's the front door for humans to interact with the system."*

```
Engineer (You)
      ↓
  Argo CD UI  or  argocd CLI
      ↓
  API Server  →  Routes commands to internal components
```

- Acts as the **gateway** between users and Argo CD internals
- Supports both a **Web UI** and a **CLI** (`argocd` command)
- Similar to how Jenkins has a UI for engineers to trigger and monitor pipelines

> ✅ **Production Benefit:** Teams can monitor deployments, check sync status, and manually trigger syncs — all through one central interface.

---

### 4️⃣ Dex — The Authentication Layer

> *"Dex handles who is allowed to access Argo CD."*

```
Engineer tries to log in to Argo CD
      ↓
   Dex (OIDC Proxy)
      ↓
Connects to Identity Providers:
  → Google / Gmail
  → GitHub
  → Facebook
  → LDAP / Active Directory
      ↓
Access Granted ✅ or Denied ❌
```

- Built-in **OpenID Connect (OIDC)** proxy server
- Provides **Single Sign-On (SSO)** capabilities
- Lets your team log in using existing company credentials

> ✅ **Production Benefit:** In large organizations, engineers don't need a separate Argo CD password. They log in with their company Google or GitHub account — saving admin overhead and improving security.

---

### 5️⃣ Redis — The Caching Layer

> *"Redis is what makes Argo CD fast and resilient."*

```
Application Controller fetches cluster state
      ↓
   Stores it in Redis Cache
      ↓
If a component crashes and restarts:
   → Reads last known state from Redis
   → Resumes without re-fetching everything from scratch
```

- Stores **cluster state** and **repository state** in memory
- Prevents expensive repeated API calls to Kubernetes and Git
- Ensures fast recovery if any component goes down

> ✅ **Production Benefit:** Without Redis, every reconciliation loop would need to re-fetch everything from scratch. Redis makes the system performant even at **massive scale with hundreds of clusters.**

---

## 🔄 How All Components Work Together

Here is the **complete end-to-end flow** when a developer makes a change:

```
┌──────────────────────────────────────────────────────────────┐
│              Complete Argo CD Reconciliation Flow            │
│                                                              │
│  1. Developer updates YAML → raises PR → merges to Git       │
│                         ↓                                    │
│  2. Reposerver detects the new commit in Git                 │
│                         ↓                                    │
│  3. Reposerver fetches updated manifests                     │
│                         ↓                                    │
│  4. Application Controller receives desired state            │
│                         ↓                                    │
│  5. Application Controller fetches live cluster state        │
│     (also checks Redis cache for last known state)           │
│                         ↓                                    │
│  6. Comparison happens:                                      │
│       Git State == Cluster State?                            │
│         YES → Status: Synced ✅ → Nothing to do              │
│         NO  → Status: OutOfSync 🔴 → Sync cluster to Git    │
│                         ↓                                    │
│  7. Engineer monitors everything via API Server (UI/CLI)     │
│                         ↓                                    │
│  8. Redis caches updated state for performance               │
└──────────────────────────────────────────────────────────────┘
```

---

## 🛠️ GitOps Tools Comparison

| Tool | Stars | CNCF Status | Best For |
|---|---|---|---|
| **Argo CD** | 13,000+ | ✅ Graduated | Full-featured GitOps with rich UI |
| **Flux CD** | High | ✅ Graduated | Lightweight, CLI-focused GitOps |
| **Jenkins X** | Moderate | - | CI/CD focused, GitOps support |
| **Spinnaker** | High | - | Deployment-focused, not pure GitOps |

> 💡 **Key Insight:** Argo CD and Flux CD are the industry standard choices. Both are **CNCF Graduated** projects — meaning they are production-proven and enterprise-ready.

### Organizations Contributing to Argo CD

The strength of Argo CD comes from its diverse, enterprise-grade contributors:

| Organization | Why It Matters |
|---|---|
| **Intuit** | Original creator of Argo CD |
| **BlackRock** | Enterprise financial services — proves production reliability |
| **Red Hat** | Enterprise Linux/OpenShift integration |
| **Codefresh** | CI/CD platform built around Argo |
| **Akuity** | Founded by Argo CD creators — managed Argo CD platform |

### Installation Options for Argo CD

```bash
# Option 1 — Plain YAML Manifests
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Option 2 — Helm Chart
helm install argocd argo/argo-cd

# Option 3 — Kubernetes Operator
# Install via OperatorHub on OpenShift or any Kubernetes cluster
```

---

## 🏭 Real Production Scenarios

### Scenario 1 — Component Crash Recovery

```
🔴 Problem:
The Application Controller pod crashes at 3 AM.
It loses all in-memory state of the cluster.

🟢 GitOps Solution:
Redis has the last known cluster state cached.
When the Application Controller restarts, it reads
from Redis and resumes the reconciliation loop
exactly where it left off.
Zero manual intervention. Zero data loss.
```

---

### Scenario 2 — Multi-Team Access Control

```
🔴 Problem:
Your company has 50 engineers across 5 teams.
You cannot give everyone full access to Argo CD.
Managing separate passwords for 50 engineers is a nightmare.

🟢 GitOps Solution:
Dex integrates with your company's Google Workspace.
Engineers log in using their company Google account.
Role-Based Access Control (RBAC) restricts what each
team can see and do.
One identity system. Zero separate passwords to manage.
```

---

### Scenario 3 — Switching From Argo CD to Flux CD

```
🔴 Problem:
Your company decides to migrate from Argo CD to Flux CD.
Your team is worried about learning a completely new tool.

🟢 GitOps Solution:
Because you understand the internal architecture principles —
Reposerver logic, Application Controller reconciliation,
caching layer — you recognize the same patterns in Flux CD.
The concepts are identical. Only the implementation differs.
Migration takes days, not weeks.
```

---

## 💡 Tips, Best Practices & Interview Prep

> ⚠️ **Watch EP 1 First:**
> The architecture will not make sense without understanding the core GitOps principles from Episode 1.

**For Production Teams:**

- ✅ Use **Helm charts** to install Argo CD — easier to upgrade and manage
- ✅ Enable **Redis persistence** so cache survives pod restarts
- ✅ Configure **Dex with your company SSO** — never rely on local passwords in production
- ✅ Understand the **Application Controller** — it is the component most likely to need tuning at scale
- ✅ Learn both **Argo CD and Flux CD** — different companies prefer different tools

**For Interviews — Know These Cold:**

| Interview Question | Answer |
|---|---|
| *What is the Reposerver?* | Microservice that connects to Git and fetches desired state manifests |
| *What is the Application Controller?* | The brain — runs the reconciliation loop comparing Git vs cluster state |
| *What is the role of Redis in Argo CD?* | Caching layer that stores cluster and repo state for performance and crash recovery |
| *What is Dex?* | OIDC proxy that provides SSO — lets users log in via Google, GitHub, etc. |
| *What happens when drift is detected?* | Application Controller overrides the cluster to match the Git desired state |
| *Argo CD vs Flux CD?* | Both are CNCF graduated GitOps tools — Argo has richer UI, Flux is more lightweight |

---

## 📝 Summary — Key Takeaways

```
GitOps seems simple on the surface.
Under the hood, it requires a robust system of
microservices working together to function reliably at scale.
```

| Component | One-Line Role |
|---|---|
| **Reposerver** | Talks to Git — fetches the desired state |
| **Application Controller** | The brain — compares Git vs Cluster, syncs if different |
| **API Server** | The front door — lets engineers interact via UI or CLI |
| **Dex** | The bouncer — handles authentication and SSO |
| **Redis** | The memory — caches state for speed and crash recovery |
| **Reconciliation Loop** | The heartbeat — runs constantly, keeps everything in sync |

---

### 🔜 What's Next?

The next episode covers **practical hands-on implementation:**
- Live installation of Argo CD on a Kubernetes cluster
- Deploying a sample application using Argo CD
- Exploring the Argo CD UI in action

---

> 📺 **Original Video:** [EP-2 | GitOps & Argo CD Crash Course](https://youtu.be/eqiqQN1CCmM)
> 🌐 **Argo CD Docs:** https://argo-cd.readthedocs.io/
> 🌐 **OpenGitOps:** https://opengitops.dev/
> ⭐ If these notes helped you, consider starring this repo!


