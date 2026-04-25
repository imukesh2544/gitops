# gitops
GitOps and Argo CD, focusing on the fundamental concepts, principles, and advantages of adopting a GitOps workflow
.
What is GitOps?
GitOps is a methodology that uses Git as the single source of truth for delivering both applications and infrastructure
. In this model, the desired state of a system is stored in a Git repository, and any changes to the environment are made by updating that repository
.
Why GitOps? (The Problem it Solves)
Before GitOps, many organizations lacked a standard mechanism for tracking, auditing, and versioning changes made directly to Kubernetes clusters
.
Lack of Tracking: Manual updates to resources (like node configurations or pod manifests) using shell scripts or kubectl commands are difficult to track over time
.
Audit Challenges: Without GitOps, it is hard to identify who made a specific change or what the cluster state was 10 days prior
.
Inconsistency: While Continuous Integration (CI) usually has a proper Git-integrated approach, Continuous Delivery (CD) often lacks the same level of versioning and verification
.
How GitOps Works
The GitOps workflow standardizes the deployment process using a standard Git philosophy:
Declarative Manifests: A DevOps engineer updates a declarative YAML manifest (e.g., pod.yaml, node.yaml) in a Git repository
.
Pull Request (PR): The change is submitted via a PR, which is reviewed and verified by another team member
.
GitOps Controller: Once merged, a GitOps controller (like Argo CD or Flux) detects the change in Git and automatically deploys it to the Kubernetes cluster
.
The Four Standard Principles of GitOps
To be considered a true GitOps system, the following principles must be followed:
Declarative: The entire system's desired state must be expressed declaratively, meaning "what you see in Git is what you have in the cluster"
.
Versioned and Immutable: All changes must be versioned. While the name implies "Git," any versioned platform like S3 buckets can technically be used to store manifests
.
Automatically Pulled: The system should automatically pull or detect changes from the source. This can be achieved through a pull mechanism (controller watching the repo) or a push mechanism (using webhooks)
.
Continuously Reconciled: The controller continuously compares the state of the Git repository with the state of the live cluster. If there is a difference (drift), the controller overrides the cluster state to match Git
.
Key Advantages
Security: Because the GitOps controller is the only entity managing the cluster, it can automatically override unwanted changes made by unauthorized users or hackers
.
Auto-Healing: If someone manually modifies a resource on the cluster, the continuous reconciliation process identifies the drift and restores the desired state from Git
.
Infrastructure Management: GitOps is equally valuable for managing infrastructure as it is for applications, especially when dealing with hundreds of clusters and thousands of resources
.
Versioning and Rollbacks: Since every change is a Git commit, you have a complete history of the system and can easily revert to a previous state
.
GitOps and Kubernetes
While the principle of GitOps is not strictly limited to Kubernetes, most popular modern tools like Argo CD and Flux are specifically designed to target Kubernetes clusters
. For a neutral, tool-independent definition of these principles, the video recommends referring to the Open GitOps project

<img width="277" height="182" alt="download" src="https://github.com/user-attachments/assets/f7d6d8a1-2c14-42bd-980f-e536e69ae64c" />

