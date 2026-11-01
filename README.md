# 🚀 Kubernetes Projects & Learning Hub

<div align="center">
  <img src="https://imgur.com/UI0WSZB.png" alt="Kubernetes Logo" width="800px">
  <br>
  <h3>A Comprehensive Guide to Master Kubernetes Through Hands-on Projects</h3>
  <p>Learn, Practice, and Deploy Real-world Applications on Kubernetes</p>
</div>

<div align="center">
  <img src="https://imgur.com/kHtYfa8.png" alt="Kubernetes Architecture" width="800px">
</div>

---

## 📋 Table of Contents
- [What's Inside](#-whats-inside)
- [Prerequisites](#-prerequisites)
- [Setup & Preparation](#-setup--preparation)
- [Learning Path](#-kubernetes-learning-path)
- [Real-Time Projects](#-real-time-kubernetes-projects)
- [Guides & Best Practices](#-kubernetes-guides--best-practices)
- [Troubleshooting](#-troubleshooting-kubernetes-issues)
- [Cloud Platforms](#-kubernetes-in-the-cloud)
- [Certifications](#-cncf-kubernetes-certifications)
- [Infrastructure as Code](#%EF%B8%8F-kubernetes-infrastructure-as-code-iac)
- [Cheat Sheets & Tools](#-kubernetes-cheat-sheets--tools)
- [Contributing](#-contributing)

---

## 📂 What's Inside

| Folder | What it contains |
|--------|------------------|
| [learning/](learning/) | **13 self-contained guides** with runnable YAML — from "Kubernetes for Everyone" to Ingress, Storage, HPA autoscaling, Helm, Network Policies, RBAC and Monitoring |
| [projects/](projects/) | **4 end-to-end projects** — 10-microservices on EKS, Spring Boot on K8s, Jenkins CI/CD to Kubernetes, and an Uber-clone DevSecOps pipeline (with Jenkinsfiles) |
| [CKAD-exercises/](CKAD-exercises/README.md) | **100+ hands-on exercises** mapped to the current [CKAD curriculum](https://github.com/cncf/curriculum), each with a hidden solution (`<details>` toggles) |

> 💡 New to Kubernetes? Start at [learning/kubernetes-for-everyone](learning/kubernetes-for-everyone/README.md). Preparing for the CKAD? Go straight to [CKAD-exercises](CKAD-exercises/README.md).

---

## 📌 Prerequisites

Before diving in, ensure you have:

| Requirement | Description |
|------------|-------------|
| Containers | Understanding of Docker/containerd/cri-o — images, layers, registries |
| `kubectl` | Basic command-line familiarity (`get`, `apply`, `describe`, `logs`) |
| YAML | Ability to read/edit YAML comfortably (it is indentation-sensitive!) |
| Linux | Basic Linux commands familiarity |

---

## 🛠 Setup & Preparation

### Essential Tools Installation

1. **Kubernetes CLI (`kubectl`)** — [installation guide](https://kubernetes.io/docs/tasks/tools/)

   ```bash
   # Linux example
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

   # verify
   kubectl version --client
   ```

2. **Local Kubernetes Cluster** — pick one:

   - [Minikube](https://minikube.sigs.k8s.io/docs/start/) — `minikube start`
   - [kind](https://kind.sigs.k8s.io/docs/user/quick-start/) — `kind create cluster`
   - [k3s](https://docs.k3s.io/) — `curl -sfL https://get.k3s.io | sh -`
   - [Docker Desktop](https://www.docker.com/products/docker-desktop/) / [Rancher Desktop](https://rancherdesktop.io/) — enable the built-in Kubernetes

   Verify with `kubectl get nodes` — you should see at least one node in `Ready` status.

📚 **Detailed Setup Guide**: [Kubernetes CLI & Cluster Setup](https://gist.github.com/NotHarshhaa/854ed5c12fff07acde88faf95b9decff)

> 💡 **Pro Tip**: Enable kubectl autocompletion and aliases for improved productivity!
>
> ```bash
> source <(kubectl completion bash)   # or zsh — add to ~/.bashrc to persist
> alias k=kubectl                     # you'll type it hundreds of times
> ```

---

## 📚 Kubernetes Learning Path

### Core Concepts
<table>
  <tr>
    <th>Level</th>
    <th>Topic</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>Beginner</td>
    <td><a href="learning/kubernetes-for-everyone/README.md">Kubernetes for Everyone</a></td>
    <td>Foundation concepts and basic architecture</td>
  </tr>
  <tr>
    <td>Beginner</td>
    <td><a href="learning/What-is-Pod-in-Kubernetes/README.md">Understanding Pods</a></td>
    <td>Deep dive into Kubernetes Pods</td>
  </tr>
  <tr>
    <td>Intermediate</td>
    <td><a href="learning/Deploying-an-Application-on-Kubernetes/README.md">Application Deployment</a></td>
    <td>Complete deployment workflow</td>
  </tr>
  <tr>
    <td>Intermediate</td>
    <td><a href="learning/Kubernetes-components-overview/README.md">Architecture Overview</a></td>
    <td>Detailed component analysis</td>
  </tr>
  <tr>
    <td>Advanced</td>
    <td><a href="learning/Deploy-DaemonSets-Service-in-Kubernetes/README.md">DaemonSets</a></td>
    <td>Cluster-wide service deployment</td>
  </tr>
</table>

### Advanced Topics
<table>
  <tr>
    <th>Level</th>
    <th>Topic</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>Intermediate</td>
    <td><a href="learning/Managing-ConfigMaps-and-Secrets/README.md">ConfigMaps & Secrets</a></td>
    <td>Configuration and sensitive data management</td>
  </tr>
  <tr>
    <td>Intermediate</td>
    <td><a href="learning/Ingress-and-Traffic-Routing/README.md">Ingress & Traffic Routing</a></td>
    <td>HTTP routing, TLS termination, ingress controllers</td>
  </tr>
  <tr>
    <td>Intermediate</td>
    <td><a href="learning/Storage-PersistentVolumes/README.md">Storage & Persistent Volumes</a></td>
    <td>PV, PVC, StorageClasses and durable state</td>
  </tr>
  <tr>
    <td>Intermediate</td>
    <td><a href="learning/Autoscaling-HPA/README.md">Autoscaling (HPA)</a></td>
    <td>Scale workloads automatically on CPU/memory metrics</td>
  </tr>
  <tr>
    <td>Intermediate</td>
    <td><a href="learning/Helm-Package-Manager/README.md">Helm Package Manager</a></td>
    <td>Package, version and deploy apps as charts</td>
  </tr>
  <tr>
    <td>Advanced</td>
    <td><a href="learning/Network-Policies-in-K8s/README.md">Network Policies</a></td>
    <td>Pod-to-pod traffic control and micro-segmentation</td>
  </tr>
  <tr>
    <td>Advanced</td>
    <td><a href="learning/RBAC-Access-Control/README.md">RBAC Access Control</a></td>
    <td>Roles, bindings and least-privilege access</td>
  </tr>
  <tr>
    <td>Advanced</td>
    <td><a href="learning/Monitoring-and-Logging/README.md">Monitoring & Logging</a></td>
    <td>Prometheus, Grafana and the EFK stack</td>
  </tr>
</table>

> 🔄 New topics are regularly added to keep content fresh and relevant!

---

## 🔥 Real-Time Kubernetes Projects

### Production-Grade Implementations

<table>
  <tr>
    <th>Project</th>
    <th>Complexity</th>
    <th>Key Learning Points</th>
  </tr>
  <tr>
    <td><a href="projects/10-microservices-deployment-eks/README.md">10-Microservices on EKS</a></td>
    <td>⭐⭐⭐⭐⭐</td>
    <td>
      - Microservices Architecture<br>
      - AWS EKS Management<br>
      - Service Mesh Integration
    </td>
  </tr>
  <tr>
    <td><a href="projects/Deploying-Spring-Boot-K8S/README.md">Spring Boot Deployment</a></td>
    <td>⭐⭐⭐</td>
    <td>
      - Java Application Deployment<br>
      - Service Configuration<br>
      - Resource Management
    </td>
  </tr>
  <tr>
    <td><a href="projects/Uber-Clone-DevSecOps/README.md">Uber Clone DevSecOps</a></td>
    <td>⭐⭐⭐⭐</td>
    <td>
      - Security Implementation<br>
      - CI/CD Pipeline<br>
      - Scalability Patterns
    </td>
  </tr>
  <tr>
    <td><a href="projects/Kubernetes-Using-Jenkins/README.md">Jenkins CI/CD Pipeline</a></td>
    <td>⭐⭐⭐</td>
    <td>
      - Automated Deployment<br>
      - Jenkins Integration<br>
      - Pipeline Management
    </td>
  </tr>
</table>

<div align="center">
  <img src="https://imgur.com/W53NNea.png" alt="Kubernetes Tools" width="800px">
  <br>
  <img src="https://imgur.com/4vq8Nxz.png" alt="Kubernetes Resource Map" width="800px">
</div>

## 🌟 Additional Resources & Projects

### Production-Grade Examples

<table>
  <tr>
    <th>Project</th>
    <th>Description</th>
    <th>Key Features</th>
  </tr>
  <tr>
    <td><a href="https://github.com/GoogleCloudPlatform/microservices-demo">Online Boutique</a></td>
    <td>Cloud-native microservices demo app by Google</td>
    <td>
      - 11 microservices in different languages<br>
      - gRPC communication<br>
      - Cloud Operations integration<br>
      - Istio & Service Mesh ready
    </td>
  </tr>
  <tr>
    <td><a href="https://github.com/HariSekhon/Kubernetes-configs">Kubernetes Configs</a></td>
    <td>Production-ready Kubernetes configurations</td>
    <td>
      - Best practices & templates<br>
      - CI/CD integrations<br>
      - Multi-cloud support<br>
      - Advanced security configs
    </td>
  </tr>
  <tr>
    <td><a href="https://github.com/kubernetes/examples">Kubernetes Examples</a></td>
    <td>Official Kubernetes example applications</td>
    <td>
      - Guestbook application<br>
      - Cassandra deployment<br>
      - WordPress with MySQL<br>
      - Various storage examples
    </td>
  </tr>
</table>

### Essential Tools & Utilities

<table>
  <tr>
    <th>Tool</th>
    <th>Category</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><a href="https://github.com/derailed/k9s">K9s</a></td>
    <td>CLI Tool</td>
    <td>Terminal UI to interact with clusters</td>
  </tr>
  <tr>
    <td><a href="https://github.com/ahmetb/kubectx">kubectx / kubens</a></td>
    <td>CLI Tool</td>
    <td>Switch between clusters and namespaces instantly</td>
  </tr>
  <tr>
    <td><a href="https://github.com/derailed/popeye">Popeye</a></td>
    <td>Cluster Sanitizer</td>
    <td>Scans live clusters for misconfigurations</td>
  </tr>
  <tr>
    <td><a href="https://github.com/kubernetes/kops">KOPS</a></td>
    <td>Cluster Management</td>
    <td>Production-grade K8s installation & management</td>
  </tr>
  <tr>
    <td><a href="https://github.com/kubernetes-sigs/kubespray">Kubespray</a></td>
    <td>Deployment</td>
    <td>Deploy production-ready clusters</td>
  </tr>
</table>

### Learning Resources

- [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way) - Step by step guide
- [Kubernetes Learning Path](https://github.com/techiescamp/kubernetes-learning-path) - Structured learning path
- [Awesome Kubernetes](https://github.com/ramitsurana/awesome-kubernetes) - Curated list of resources
- [Kubernetes Failure Stories](https://github.com/hjacobs/kubernetes-failure-stories) - Learn from others' experiences

---

## 📖 Kubernetes Guides & Best Practices

### 🧭 In this repository
| Topic | Guide |
|-------|-------|
| Ingress & traffic routing | [learning/Ingress-and-Traffic-Routing](learning/Ingress-and-Traffic-Routing/README.md) |
| Storage & Persistent Volumes | [learning/Storage-PersistentVolumes](learning/Storage-PersistentVolumes/README.md) |
| Monitoring & Logging (Prometheus, EFK) | [learning/Monitoring-and-Logging](learning/Monitoring-and-Logging/README.md) |
| RBAC & Network Policies | [learning/RBAC-Access-Control](learning/RBAC-Access-Control/README.md) • [learning/Network-Policies-in-K8s](learning/Network-Policies-in-K8s/README.md) |
| ConfigMaps & Secrets | [learning/Managing-ConfigMaps-and-Secrets](learning/Managing-ConfigMaps-and-Secrets/README.md) |
| Autoscaling (HPA) & Helm | [learning/Autoscaling-HPA](learning/Autoscaling-HPA/README.md) • [learning/Helm-Package-Manager](learning/Helm-Package-Manager/README.md) |

### 🌐 Networking
- [The Kubernetes Networking Guide](https://www.tkng.io/)
- [Hands-on Networking Labs](https://www.tkng.io/lab/)

### 🔒 Security
1. [Official Security Checklist](https://kubernetes.io/docs/concepts/security/security-checklist/)
2. [Awesome K8s Security](https://github.com/magnologan/awesome-k8s-security)
3. [Security CTF Challenges](https://eksclustergames.com)

### 🗄️ Storage
- **Comprehensive Guide**: [Understanding Kubernetes Storage](https://medium.com/@seifeddinerajhi/understanding-storage-in-kubernetes-ee2c19001aae)
  - Persistent Volumes (PV)
  - Persistent Volume Claims (PVC)
  - Storage Classes
  - Dynamic Provisioning

---

## 🛠 Troubleshooting Kubernetes Issues

| Issue Type | Resource |
|------------|----------|
| Common Errors | [Solutions Guide](https://cloudtweaks.com/2023/01/common-kubernetes-errors/) |
| Exit Codes | [Complete Guide](https://komodor.com/learn/exit-codes-in-containers-and-kubernetes-the-complete-guide/) |
| Deployments | [Visual Troubleshooter](https://learnk8s.io/troubleshooting-deployments) |
| General Issues | [Comprehensive Guide](https://komodor.com/learn/kubernetes-troubleshooting-the-complete-guide/) |

---

## ☁ Kubernetes in the Cloud

### Major Cloud Providers

<table>
  <tr>
    <th>Platform</th>
    <th>Service</th>
    <th>Resources</th>
  </tr>
  <tr>
    <td>AWS</td>
    <td>EKS</td>
    <td>
      <ul>
        <li><a href="https://github.com/terraform-aws-modules/terraform-aws-eks">Terraform Module</a></li>
        <li><a href="https://aws.github.io/aws-eks-best-practices/">Best Practices</a></li>
        <li><a href="https://github.com/stacksimplify/aws-eks-kubernetes-masterclass">Masterclass</a></li>
      </ul>
    </td>
  </tr>
  <tr>
    <td>Azure</td>
    <td>AKS</td>
    <td>
      <ul>
        <li><a href="https://github.com/stacksimplify/azure-aks-kubernetes-masterclass">Masterclass</a></li>
        <li><a href="https://www.the-aks-checklist.com/">AKS Checklist</a></li>
      </ul>
    </td>
  </tr>
  <tr>
    <td>Google</td>
    <td>GKE</td>
    <td>
      <ul>
        <li><a href="https://github.com/terraform-google-modules/terraform-google-kubernetes-engine">Terraform Module</a></li>
        <li><a href="https://github.com/GoogleCloudPlatform/kubernetes-engine-samples">Sample Apps</a></li>
      </ul>
    </td>
  </tr>
</table>

---

## 🎓 CNCF Kubernetes Certifications

### Certification Preparation Resources

| Certification | Resources |
|---------------|-----------|
| CKAD | [Hands-on Exercises](CKAD-exercises/README.md) • [Official Curriculum](https://github.com/cncf/curriculum) • [CKAD on CNCF](https://www.cncf.io/training/certification/ckad/) |
| CKA | [Practice Exercises](https://github.com/alijahnas/CKA-practice-exercises) • [Additional Exercises](https://github.com/chadmcrowell/CKA-Exercises) |
| CKS | [Study Guide](https://github.com/walidshaari/Certified-Kubernetes-Security-Specialist) • [Video Course](https://www.youtube.com/watch?v=d9xfB5qaOfg) |

---

## ⚙️ Kubernetes Infrastructure as Code (IaC)

| Tool | Purpose | Resource |
|------|----------|----------|
| Helm | Package Manager | [Repository](https://github.com/helm/helm) |
| Kustomize | Config Management | [Repository](https://github.com/kubernetes-sigs/kustomize) |
| Terraform | Infrastructure | [Documentation](https://www.terraform.io/) |
| Pulumi | Multi-language IaC | [Repository](https://github.com/pulumi/pulumi) |
| Skaffold | Development | [Repository](https://github.com/GoogleContainerTools/skaffold) |
| Argo CD | GitOps CD | [Repository](https://github.com/argoproj/argo-cd) — used in the [Jenkins+ArgoCD project](projects/Kubernetes-Using-Jenkins/README.md) |
| Flux | GitOps Toolkit | [Repository](https://github.com/fluxcd/flux2) |

---

## 🔥 Kubernetes Cheat Sheets & Tools

### Quick Reference Guides
- [kubectl Commands](https://github.com/NotHarshhaa/devops-cheatsheet/blob/master/Containerization/Kubernetes.md)
- [Helm Commands](https://github.com/NotHarshhaa/devops-cheatsheet/blob/master/Containerization/Helm.md)
- [Docker Reference](https://github.com/NotHarshhaa/devops-cheatsheet/blob/master/Containerization/Docker.md)

### Tools
- [K8s YAML Generator](https://www.k8syaml.com/)

---

## 🤝 Contributing

We welcome contributions! To contribute:

1. Fork the repository
2. Create your feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

---

## ⭐ Show Your Support

If you find this repository helpful, please give it a star! Your support motivates continued maintenance and improvements.

---

## 🛠️ Author & Community

Created with 💡 by [Harshhaa](https://github.com/NotHarshhaa)

### 📫 Connect With Me

<div align="center">
  
[![LinkedIn](https://img.shields.io/badge/LinkedIn-%230077B5.svg?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/harshhaa-vardhan-reddy)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/NotHarshhaa)
[![Telegram](https://img.shields.io/badge/Telegram-26A5E4?style=for-the-badge&logo=telegram&logoColor=white)](https://t.me/prodevopsguy)
[![Dev.to](https://img.shields.io/badge/Dev.to-0A0A0A?style=for-the-badge&logo=dev.to&logoColor=white)](https://dev.to/notharshhaa)
[![Hashnode](https://img.shields.io/badge/Hashnode-2962FF?style=for-the-badge&logo=hashnode&logoColor=white)](https://hashnode.com/@prodevopsguy)

</div>

---

### 📢 Stay Updated

<div align="center">
  <img src="https://imgur.com/2j7GSPs.png" alt="Follow Me">
</div>
