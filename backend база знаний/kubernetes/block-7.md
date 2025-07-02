# 🏢 Блок 7: Производственные практики и DevOps

**⏱️ Продолжительность:** 2-3 недели  
**🎯 Цель:** Применить знания в реальных production-сценариях

---

## 🚀 Введение в Production-Ready Kubernetes

На этом этапе мы переходим от изучения отдельных компонентов к пониманию того, как Kubernetes работает в реальном мире. Production-окружение требует особого подхода к архитектуре, безопасности, мониторингу и управлению жизненным циклом приложений.

### Ключевые принципы Production Kubernetes:

```
┌─────────────────────────────────────────────────────────────────┐
│                    PRODUCTION PILLARS                           │
├─────────────┬─────────────┬─────────────┬─────────────────────────┤
│ RELIABILITY │ SCALABILITY │  SECURITY   │     OBSERVABILITY       │
│             │             │             │                         │
│ • High      │ • Auto      │ • RBAC      │ • Comprehensive         │
│   Availability│   Scaling  │ • Network   │   Monitoring            │
│ • Disaster  │ • Load      │   Policies  │ • Distributed           │
│   Recovery  │   Balancing │ • Pod       │   Tracing              │
│ • Backup &  │ • Resource  │   Security  │ • Centralized           │
│   Restore   │   Management│ • Secrets   │   Logging              │
└─────────────┴─────────────┴─────────────┴─────────────────────────┘
```

---

## 📊 Глава 7.1: Managed Kubernetes (4-5 дней)

### Концепция Managed Kubernetes

Managed Kubernetes - это облачные сервисы, где провайдер берет на себя управление control plane, оставляя вам контроль над worker nodes и приложениями.

#### Архитектурное разделение ответственности:

```
┌──────────────────────────────────────────────────────────────────┐
│                   RESPONSIBILITY MATRIX                          │
├─────────────────────┬─────────────────┬─────────────────────────┤
│      COMPONENT      │   SELF-MANAGED  │      MANAGED (EKS)      │
├─────────────────────┼─────────────────┼─────────────────────────┤
│ API Server          │     YOU ⚙️       │    PROVIDER ☁️          │
│ etcd                │     YOU ⚙️       │    PROVIDER ☁️          │
│ Scheduler           │     YOU ⚙️       │    PROVIDER ☁️          │
│ Controller Manager  │     YOU ⚙️       │    PROVIDER ☁️          │
├─────────────────────┼─────────────────┼─────────────────────────┤
│ Worker Nodes        │     YOU ⚙️       │       YOU ⚙️            │
│ kubelet             │     YOU ⚙️       │       YOU ⚙️            │
│ Container Runtime   │     YOU ⚙️       │       YOU ⚙️            │
│ Applications        │     YOU ⚙️       │       YOU ⚙️            │
└─────────────────────┴─────────────────┴─────────────────────────┘
```

### Amazon EKS: Особенности и Best Practices

#### Архитектурные особенности EKS:

```
┌─────────────────────────────────────────────────────────────────┐
│                        EKS ARCHITECTURE                         │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                   CONTROL PLANE                             ││
│  │              (Managed by AWS)                               ││
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ ││
│  │  │ API Server  │  │   Scheduler │  │ Controller Manager  │ ││
│  │  └─────────────┘  └─────────────┘  └─────────────────────┘ ││
│  │  ┌─────────────────────────────────────────────────────────┐ ││
│  │  │                    etcd                                 │ ││
│  │  └─────────────────────────────────────────────────────────┘ ││
│  └─────────────────────────────────────────────────────────────┘│
│                                │                                │
│                        ┌───────▼───────┐                       │
│                        │ Load Balancer │                       │
│                        └───────┬───────┘                       │
│                                │                                │
│  ┌─────────────────────────────┼─────────────────────────────┐  │
│  │                   DATA PLANE                               │  │
│  │                 (Your Responsibility)                      │  │
│  │                                                            │  │
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    │  │
│  │  │    Node 1   │    │    Node 2   │    │    Node 3   │    │  │
│  │  │             │    │             │    │             │    │  │
│  │  │ ┌─────────┐ │    │ ┌─────────┐ │    │ ┌─────────┐ │    │  │
│  │  │ │  Pod A  │ │    │ │  Pod B  │ │    │ │  Pod C  │ │    │  │
│  │  │ └─────────┘ │    │ └─────────┘ │    │ └─────────┘ │    │  │
│  │  └─────────────┘    └─────────────┘    └─────────────┘    │  │
│  └─────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

#### EKS Best Practices:

**1. Network Security:**
- Используйте AWS VPC CNI для оптимальной производительности
- Настройте security groups на уровне pod (Pod Security Groups)
- Реализуйте network segmentation через subnets

**2. IAM Integration:**
- Интегрируйте RBAC с AWS IAM через IRSA (IAM Roles for Service Accounts)
- Используйте принцип least privilege для service accounts
- Настройте fine-grained permissions для каждого workload

**3. Storage Optimization:**
- Используйте EBS CSI driver для persistent volumes
- Настройте appropriate storage classes для разных типов workloads
- Реализуйте backup strategies с AWS Backup

### Google GKE: Autopilot vs Standard

#### Сравнение режимов GKE:

```
┌─────────────────────────────────────────────────────────────────┐
│                    GKE: STANDARD vs AUTOPILOT                  │
├─────────────────────┬─────────────────┬─────────────────────────┤
│      ASPECT         │    STANDARD     │       AUTOPILOT         │
├─────────────────────┼─────────────────┼─────────────────────────┤
│ Node Management     │ Manual scaling  │ Fully automated         │
│ Cost Model          │ Pay for nodes   │ Pay per pod resource    │
│ Security            │ Your setup      │ Hardened by default     │
│ Flexibility         │ Full control    │ Opinionated setup       │
│ Maintenance         │ Your burden     │ Google managed          │
├─────────────────────┼─────────────────┼─────────────────────────┤
│ Use Case            │ Custom needs    │ Standard workloads      │
│ Complexity          │ High            │ Low                     │
│ Time to Production  │ Longer          │ Faster                  │
└─────────────────────┴─────────────────┴─────────────────────────┘
```

#### GKE Autopilot Концепции:

**Bin Packing Optimization:**
```
Traditional Node-based:           Autopilot Pod-based:
┌─────────────────────────┐      ┌─────────────────────────┐
│ Node 1 (4 CPU, 8GB)    │      │    Optimized Bins       │
│ ┌─────┐ ┌─────┐ ░░░░░  │      │ ┌─────┐ ┌─────┐ ┌─────┐ │
│ │Pod A│ │Pod B│ Waste  │  →   │ │Pod A│ │Pod B│ │Pod C│ │
│ └─────┘ └─────┘        │      │ └─────┘ └─────┘ └─────┘ │
└─────────────────────────┘      └─────────────────────────┘
      Pay for entire node              Pay only for pods
```

### Azure AKS: Интеграция с Azure Services

#### AKS Integration Ecosystem:

```
┌─────────────────────────────────────────────────────────────────┐
│                     AKS INTEGRATION MAP                         │
│                                                                 │
│          ┌─────────────────────────────────────────┐            │
│          │              AKS CLUSTER                │            │
│          │  ┌─────────┐  ┌─────────┐  ┌─────────┐  │            │
│          │  │  Pods   │  │  Pods   │  │  Pods   │  │            │
│          │  └─────────┘  └─────────┘  └─────────┘  │            │
│          └─────────────────┬───────────────────────┘            │
│                            │                                    │
│  ┌─────────────────────────┼─────────────────────────────────┐  │
│  │               AZURE SERVICES                              │  │
│  │                                                           │  │
│  │ ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │  │
│  │ │Azure Monitor│  │ Key Vault   │  │ Container Registry  │ │  │
│  │ │     &       │  │   (Secrets) │  │       (ACR)         │ │  │
│  │ │ Log Analytics│  └─────────────┘  └─────────────────────┘ │  │
│  │ └─────────────┘                                           │  │
│  │                                                           │  │
│  │ ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │  │
│  │ │   Azure AD  │  │Azure Policy │  │  Application        │ │  │
│  │ │   (RBAC)    │  │(Governance) │  │  Gateway            │ │  │
│  │ └─────────────┘  └─────────────┘  └─────────────────────┘ │  │
│  └─────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

#### AKS-специфичные возможности:

**1. Azure AD Integration:**
- Seamless authentication через Azure AD
- Conditional access policies для cluster access
- Group-based RBAC mapping

**2. Azure Policy for AKS:**
- Gatekeeper integration для policy enforcement
- Built-in policy definitions для security и compliance
- Audit и remediation capabilities

---

## 🏗️ Глава 7.2: Infrastructure as Code (3-4 дня)

### Концепция Infrastructure as Code (IaC)

IaC - это подход к управлению инфраструктурой через код, обеспечивающий повторяемость, версионность и автоматизацию.

#### IaC Principles:

```
┌─────────────────────────────────────────────────────────────────┐
│                       IaC PRINCIPLES                            │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  DECLARATIVE  │  Define WHAT you want, not HOW            ││
│  └─────────────────────────────────────────────────────────────┘│
│                                   │                            │
│  ┌─────────────────────────────────▼───────────────────────────┐│
│  │  IDEMPOTENT   │  Same input → Same output, always         ││
│  └─────────────────────────────────┬───────────────────────────┘│
│                                   │                            │
│  ┌─────────────────────────────────▼───────────────────────────┐│
│  │  VERSIONED    │  Track changes like application code      ││
│  └─────────────────────────────────┬───────────────────────────┘│
│                                   │                            │
│  ┌─────────────────────────────────▼───────────────────────────┐│
│  │  IMMUTABLE    │  Replace rather than modify               ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

### Terraform для Kubernetes Infrastructure

#### Terraform Kubernetes Provider Architecture:

```
┌─────────────────────────────────────────────────────────────────┐
│                  TERRAFORM K8S WORKFLOW                        │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                 TERRAFORM CODE                              ││
│  │  resource "kubernetes_deployment" "app" {                  ││
│  │    metadata { name = "my-app" }                             ││
│  │    spec { ... }                                             ││
│  │  }                                                          ││
│  └─────────────────────────┬───────────────────────────────────┘│
│                            │                                    │
│                      ┌─────▼─────┐                             │
│                      │ terraform │                             │
│                      │   plan    │                             │
│                      └─────┬─────┘                             │
│                            │                                    │
│                      ┌─────▼─────┐                             │
│                      │ terraform │                             │
│                      │   apply   │                             │
│                      └─────┬─────┘                             │
│                            │                                    │
│  ┌─────────────────────────▼───────────────────────────────────┐│
│  │                KUBERNETES CLUSTER                          ││
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ ││
│  │  │ Deployment  │  │   Service   │  │    ConfigMap        │ ││
│  │  └─────────────┘  └─────────────┘  └─────────────────────┘ ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

#### Terraform Best Practices для Kubernetes:

**1. State Management:**
- Используйте remote state (S3, GCS, Azure Storage)
- Implement state locking для team collaboration
- Separate state files для different environments

**2. Module Organization:**
```
┌─────────────────────────────────────────────────────────────────┐
│                   TERRAFORM STRUCTURE                          │
│                                                                 │
│  project/                                                       │
│  ├── environments/                                              │
│  │   ├── dev/                                                   │
│  │   ├── staging/                                               │
│  │   └── prod/                                                  │
│  ├── modules/                                                   │
│  │   ├── k8s-cluster/                                           │
│  │   ├── monitoring/                                            │
│  │   └── networking/                                            │
│  └── shared/                                                    │
│      ├── variables.tf                                           │
│      └── outputs.tf                                             │
└─────────────────────────────────────────────────────────────────┘
```

### Pulumi для Cloud-Native IaC

#### Pulumi vs Traditional IaC:

```
┌─────────────────────────────────────────────────────────────────┐
│                   PULUMI ADVANTAGES                             │
├─────────────────────┬─────────────────┬─────────────────────────┤
│     FEATURE         │   TERRAFORM     │        PULUMI           │
├─────────────────────┼─────────────────┼─────────────────────────┤
│ Language            │ HCL (DSL)       │ Real programming langs  │
│ Logic & Loops       │ Limited         │ Full language features  │
│ Testing             │ External tools  │ Native unit testing     │
│ Debugging           │ Difficult       │ Standard debuggers      │
│ IDE Support         │ Basic           │ Full IntelliSense      │
│ Package Management  │ Limited         │ Native package managers │
└─────────────────────┴─────────────────┴─────────────────────────┘
```

#### Pulumi Kubernetes Programming Model:

```
┌─────────────────────────────────────────────────────────────────┐
│                  PULUMI K8S CONCEPTS                           │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                    STACK                                    ││
│  │                (Environment)                                ││
│  │  ┌─────────────────────────────────────────────────────────┐││
│  │  │                  PROGRAM                                │││
│  │  │              (Your Code)                                │││
│  │  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐│││
│  │  │  │  Resources  │  │   Config    │  │     Outputs     │││
│  │  │  │             │  │             │  │                 │││
│  │  │  │ k8s.Deployment│ database.url │ │ service.loadBalancer│││
│  │  │  │ k8s.Service │  │ replicas: 3 │  │ ingress.ip      │││
│  │  │  └─────────────┘  └─────────────┘  └─────────────────┘│││
│  │  └─────────────────────────────────────────────────────────┘││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

### Crossplane для Infrastructure Composition

#### Crossplane Architecture:

```
┌─────────────────────────────────────────────────────────────────┐
│                   CROSSPLANE ARCHITECTURE                      │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                KUBERNETES CLUSTER                          ││
│  │                                                             ││
│  │  ┌─────────────────────────────────────────────────────────┐││
│  │  │                 CROSSPLANE                              │││
│  │  │                                                         │││
│  │  │ ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │││
│  │  │ │   Providers │  │ Compositions│  │   Composite     │  │││
│  │  │ │             │  │             │  │   Resources     │  │││
│  │  │ │ AWS, GCP,   │  │ Infrastructure│ │                │  │││
│  │  │ │ Azure       │  │ Blueprints  │  │ Database,       │  │││
│  │  │ └─────────────┘  └─────────────┘  │ Network, etc.   │  │││
│  │  │                                   └─────────────────┘  │││
│  │  └─────────────────────────────────────────────────────────┘││
│  │                                │                            ││
│  └────────────────────────────────┼────────────────────────────┘│
│                                   │                             │
│  ┌────────────────────────────────▼────────────────────────────┐│
│  │                      CLOUD PROVIDERS                       ││
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ ││
│  │  │     AWS     │  │     GCP     │  │       AZURE         │ ││
│  │  └─────────────┘  └─────────────┘  └─────────────────────┘ ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

#### Crossplane Benefits:

**1. Kubernetes-Native:**
- Все ресурсы управляются через Kubernetes API
- Consistent experience с kubectl
- Integration с existing K8s tooling

**2. Composition Model:**
- Define reusable infrastructure patterns
- Abstract complexity от development teams
- Self-service infrastructure provisioning

---

## 🔄 Глава 7.3: GitOps практики (3-4 дня)

### Концепция GitOps

GitOps - это operational framework, который использует Git как single source of truth для declarative infrastructure и applications.

#### GitOps Principles:

```
┌─────────────────────────────────────────────────────────────────┐
│                       GITOPS FLOW                              │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  1. DECLARATIVE CONFIGURATION IN GIT                       ││
│  │     ┌───────────────────────────────────────────────────┐   ││
│  │     │ manifests/                                        │   ││
│  │     │ ├── deployment.yaml                               │   ││
│  │     │ ├── service.yaml                                  │   ││
│  │     │ └── configmap.yaml                                │   ││
│  │     └───────────────────────────────────────────────────┘   ││
│  └─────────────────────────┬───────────────────────────────────┘│
│                            │                                    │
│  ┌─────────────────────────▼───────────────────────────────────┐│
│  │  2. AUTOMATED DEPLOYMENT AGENT                             ││
│  │     ┌───────────────────────────────────────────────────┐   ││
│  │     │             ArgoCD / Flux                         │   ││
│  │     │                                                   │   ││
│  │     │ • Pull from Git                                   │   ││
│  │     │ • Compare desired vs actual state                 │   ││
│  │     │ • Apply changes automatically                     │   ││
│  │     └───────────────────────────────────────────────────┘   ││
│  └─────────────────────────┬───────────────────────────────────┘│
│                            │                                    │
│  ┌─────────────────────────▼───────────────────────────────────┐│
│  │  3. KUBERNETES CLUSTER                                     ││
│  │     ┌───────────────────────────────────────────────────┐   ││
│  │     │        Desired State Applied                      │   ││
│  │     │                                                   │   ││
│  │     │ ┌─────────┐ ┌─────────┐ ┌─────────────────────┐   │   ││
│  │     │ │  Pods   │ │Services │ │   ConfigMaps        │   │   ││
│  │     │ └─────────┘ └─────────┘ └─────────────────────┘   │   ││
│  │     └───────────────────────────────────────────────────┘   ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

### Repository Structure для GitOps

#### Mono-repo vs Multi-repo Strategy:

```
┌─────────────────────────────────────────────────────────────────┐
│                 GITOPS REPOSITORY PATTERNS                     │
├─────────────────────┬─────────────────┬─────────────────────────┤
│      PATTERN        │    MONO-REPO    │      MULTI-REPO         │
├─────────────────────┼─────────────────┼─────────────────────────┤
│ Structure           │ All in one      │ Separate repositories   │
│ Complexity          │ Simple          │ More complex            │
│ Access Control      │ Repository level│ Fine-grained            │
│ CI/CD               │ Single pipeline │ Multiple pipelines      │
│ Team Independence   │ Limited         │ High                    │
│ Consistency         │ Easy            │ Requires discipline     │
└─────────────────────┴─────────────────┴─────────────────────────┘
```

#### Recommended GitOps Repository Structure:

```
┌─────────────────────────────────────────────────────────────────┐
│                    GITOPS REPO STRUCTURE                       │
│                                                                 │
│  k8s-gitops/                                                    │
│  ├── apps/                          # Application definitions    │
│  │   ├── base/                      # Base configurations      │
│  │   │   ├── kustomization.yaml                                │
│  │   │   ├── deployment.yaml                                   │
│  │   │   └── service.yaml                                      │
│  │   └── overlays/                  # Environment-specific     │
│  │       ├── dev/                                              │
│  │       ├── staging/                                          │
│  │       └── prod/                                             │
│  ├── infrastructure/                # Infrastructure as Code    │
│  │   ├── networking/                                           │
│  │   ├── storage/                                              │
│  │   └── monitoring/                                           │
│  ├── bootstrap/                     # GitOps bootstrapping     │
│  │   ├── argocd/                                               │
│  │   └── sealed-secrets/                                       │
│  └── clusters/                      # Cluster-specific configs │
│      ├── dev-cluster/                                          │
│      ├── staging-cluster/                                      │
│      └── prod-cluster/                                         │
└─────────────────────────────────────────────────────────────────┘
```

### Environment Promotion Strategies

#### Progressive Delivery Pipeline:

```
┌─────────────────────────────────────────────────────────────────┐
│                 ENVIRONMENT PROMOTION FLOW                     │
│                                                                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────┐  │
│  │     DEV     │    │  STAGING    │    │        PROD         │  │
│  │             │    │             │    │                     │  │
│  │ • Feature   │    │ • Integration│    │ • Production        │  │
│  │   branches  │    │   testing    │    │   traffic           │  │
│  │ • Fast      │    │ • Performance│    │ • Blue/Green or     │  │
│  │   feedback  │    │   testing    │    │   Canary deploy     │  │
│  │ • Developer │────▶│ • Security   │────▶│ • Full monitoring   │  │
│  │   iteration │    │   scanning   │    │ • Rollback ready    │  │
│  └─────────────┘    └─────────────┘    └─────────────────────┘  │
│                                                                 │
│           Auto                   Manual Gate                    │
│        Promotion              (Approval Required)               │
└─────────────────────────────────────────────────────────────────┘
```

#### Promotion Mechanisms:

**1. Automated Promotion:**
- Triggered by successful tests
- Version bumping in target environment
- Automatic PR creation

**2. Manual Gates:**
- Required approvals for production
- Security and compliance checks
- Business stakeholder sign-off

### Secret Management в GitOps

#### Sealed Secrets Pattern:

```
┌─────────────────────────────────────────────────────────────────┐
│                    SEALED SECRETS WORKFLOW                     │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  1. DEVELOPER CREATES SECRET                                ││
│  │     kubectl create secret generic mysecret --from-literal=key=value ││
│  │     --dry-run=client -o yaml > secret.yaml                 ││
│  └─────────────────────────┬───────────────────────────────────┘│
│                            │                                    │
│  ┌─────────────────────────▼───────────────────────────────────┐│
│  │  2. SEAL THE SECRET                                         ││
│  │     kubeseal -f secret.yaml -w sealedsecret.yaml           ││
│  │     # Creates encrypted version safe for Git               ││
│  └─────────────────────────┬───────────────────────────────────┘│
│                            │                                    │
│  ┌─────────────────────────▼───────────────────────────────────┐│
│  │  3. COMMIT TO GIT                                           ││
│  │     git add sealedsecret.yaml                               ││
│  │     git commit -m "Add sealed secret"                      ││
│  └─────────────────────────┬───────────────────────────────────┘│
│                            │                                    │
│  ┌─────────────────────────▼───────────────────────────────────┐│
│  │  4. ARGOCD DEPLOYS                                          ││
│  │     SealedSecret → Controller → Secret                     ││
│  │     (Automatic decryption in cluster)                      ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

#### External Secrets Operator Pattern:

```
┌─────────────────────────────────────────────────────────────────┐
│                EXTERNAL SECRETS INTEGRATION                    │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │              EXTERNAL SECRET STORES                        ││
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ ││
│  │  │ AWS Secrets │  │ HashiCorp   │  │    Azure Key        │ ││
│  │  │   Manager   │  │   Vault     │  │      Vault          │ ││
│  │  └─────────────┘  └─────────────┘  └─────────────────────┘ ││
│  └─────────────────────────┬───────────────────────────────────┘│
│                            │                                    │
│  ┌─────────────────────────▼───────────────────────────────────┐│
│  │              EXTERNAL SECRETS OPERATOR                     ││
│  │  ┌───────────────────────────────────────────────────────┐  ││
│  │  │  ExternalSecrets CRD                                  │  ││
│  │  │  • Defines what secrets to fetch                     │  ││
│  │  │  • Maps external keys to K8s secret keys             │  ││
│  │  │  • Handles refresh intervals                         │  ││
│  │  └───────────────────────────────────────────────────────┘  ││
│  └─────────────────────────┬───────────────────────────────────┘│
│                            │                                    │
│  ┌─────────────────────────▼───────────────────────────────────┐│
│  │                KUBERNETES SECRETS                          ││
│  │  ┌───────────────────────────────────────────────────────┐  ││
│  │  │  Automatically created and refreshed secrets         │  ││
│  │  │  Available for pods as usual                          │  ││
│  │  └───────────────────────────────────────────────────────┘  ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Глава 7.4: Chaos Engineering (2-3 дня)

### Концепция Chaos Engineering

Chaos Engineering - это дисциплина экспериментирования на системе для построения уверенности в способности системы выдерживать турбулентные условия в production.

#### Принципы Chaos Engineering:

```
┌─────────────────────────────────────────────────────────────────┐
│                   CHAOS ENGINEERING PRINCIPLES                 │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  1. HYPOTHESIS-DRIVEN                                      ││
│  │     "System should remain stable when X fails"             ││
│  └─────────────────────────┬───────────────────────────────────┘│
│                            │                                    │
│  ┌─────────────────────────▼───────────────────────────────────┐│
│  │  2. MINIMIZE BLAST RADIUS                                  ││
│  │     Start small, increase scope gradually                  ││
│  └─────────────────────────┬───────────────────────────────────┘│
│                            │                                    │
│  ┌─────────────────────────▼───────────────────────────────────┐│
│  │  3. AUTOMATE EXPERIMENTS                                   ││
│  │     Reproducible and scalable testing                     ││
│  └─────────────────────────┬───────────────────────────────────┘│
│                            │                                    │
│  ┌─────────────────────────▼───────────────────────────────────┐│
│  │  4. LEARN AND IMPROVE                                      ││
│  │     Use results to make system more resilient             ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

### Chaos Monkey для Kubernetes

#### Chaos Types в Kubernetes:

```
┌─────────────────────────────────────────────────────────────────┐
│                    KUBERNETES CHAOS TYPES                      │
├─────────────────────┬─────────────────┬─────────────────────────┤
│     LAYER           │   CHAOS TYPE    │       EXAMPLES          │
├─────────────────────┼─────────────────┼─────────────────────────┤
│ Pod Level           │ Pod Failure     │ Kill random pods        │
│                     │ CPU/Memory      │ Stress pod resources    │
│                     │ Network         │ Packet loss, latency   │
├─────────────────────┼─────────────────┼─────────────────────────┤
│ Node Level          │ Node Failure    │ Drain/cordon nodes      │
│                     │ Disk Pressure   │ Fill up disk space     │
│                     │ Clock Skew      │ Modify system time     │
├─────────────────────┼─────────────────┼─────────────────────────┤
│ Cluster Level       │ etcd Issues     │ etcd corruption         │
│                     │ API Server      │ Rate limiting           │
│                     │ DNS Failure     │ CoreDNS issues         │
├─────────────────────┼─────────────────┼─────────────────────────┤
│ Application Level   │ Dependency      │ External API failures   │
│                     │ Database        │ Connection issues       │
│                     │ Cache           │ Redis/Memcached fails  │
└─────────────────────┴─────────────────┴─────────────────────────┘
```

### Litmus для Chaos Experiments

#### Litmus Architecture:

```
┌─────────────────────────────────────────────────────────────────┐
│                     LITMUS ARCHITECTURE                        │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                 LITMUS PORTAL                               ││
│  │  ┌───────────────────────────────────────────────────────┐  ││
│  │  │              WEB DASHBOARD                            │  ││
│  │  │  • Experiment management                              │  ││
│  │  │  • Workflow visualization                             │  ││
│  │  │  • Results analysis                                   │  ││
│  │  └───────────────────────────────────────────────────────┘  ││
│  └─────────────────────────┬───────────────────────────────────┘│
│                            │                                    │
│  ┌─────────────────────────▼───────────────────────────────────┐│
│  │                CHAOS WORKFLOWS                             ││
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ ││
│  │  │   Chaos     │  │   Steady    │  │     Hypothesis      │ ││
│  │  │ Experiments │  │    State    │  │    Validation       │ ││
│  │  │             │  │  Checking   │  │                     │ ││
│  │  │ Pod Kill    │  │             │  │ SLO Monitoring      │ ││
│  │  │ Network     │  │ Health      │  │ Alert Checking      │ ││
│  │  │ Latency     │  │ Probes      │  │                     │ ││
│  │  └─────────────┘  └─────────────┘  └─────────────────────┘ ││
│  └─────────────────────────┬───────────────────────────────────┘│
│                            │                                    │
│  ┌─────────────────────────▼───────────────────────────────────┐│
│  │              TARGET KUBERNETES CLUSTER                     ││
│  │  ┌───────────────────────────────────────────────────────┐  ││
│  │  │               CHAOS INJECTION                         │  ││
│  │  │                                                       │  ││
│  │  │ ┌─────────┐ ┌─────────┐ ┌─────────────────────────┐   │  ││
│  │  │ │ Pod A   │ │ Pod B   │ │        Pod C            │   │  ││
│  │  │ │(Target) │ │(Normal) │ │      (Observer)         │   │  ││
│  │  │ └─────────┘ └─────────┘ └─────────────────────────┘   │  ││
│  │  └───────────────────────────────────────────────────────┘  ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

#### Chaos Experiment Lifecycle:

```
┌─────────────────────────────────────────────────────────────────┐
│                  CHAOS EXPERIMENT LIFECYCLE                    │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  PHASE 1: STEADY STATE HYPOTHESIS                          ││
│  │  "Under normal conditions, system responds < 100ms"        ││
│  └─────────────────────────┬───────────────────────────────────┘│
│                            │                                    │
│  ┌─────────────────────────▼───────────────────────────────────┐│
│  │  PHASE 2: DEFINE CHAOS VARIABLES                           ││
│  │  • What to break: Pod failure                              ││
│  │  • Scope: 10% of pods                                      ││
│  │  • Duration: 5 minutes                                     ││
│  └─────────────────────────┬───────────────────────────────────┘│
│                            │                                    │
│  ┌─────────────────────────▼───────────────────────────────────┐│
│  │  PHASE 3: HYPOTHESIS VALIDATION                            ││
│  │  "Even with 10% pod failure, response time < 200ms"       ││
│  └─────────────────────────┬───────────────────────────────────┘│
│                            │                                    │
│  ┌─────────────────────────▼───────────────────────────────────┐│
│  │  PHASE 4: RUN EXPERIMENT                                   ││
│  │  • Monitor baseline                                        ││
│  │  • Inject chaos                                            ││
│  │  • Observe system behavior                                 ││
│  │  • Validate hypothesis                                     ││
│  └─────────────────────────┬───────────────────────────────────┘│
│                            │                                    │
│  ┌─────────────────────────▼───────────────────────────────────┐│
│  │  PHASE 5: LEARN AND IMPROVE                                ││
│  │  • Analyze results                                         ││
│  │  • Identify weaknesses                                     ││
│  │  • Implement improvements                                  ││
│  │  • Repeat with increased scope                             ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

### Fault Injection Patterns

#### Common Fault Injection Strategies:

```
┌─────────────────────────────────────────────────────────────────┐
│                 FAULT INJECTION PATTERNS                       │
├─────────────────────┬─────────────────────────────────────────────┤
│     PATTERN         │              DESCRIPTION                    │
├─────────────────────┼─────────────────────────────────────────────┤
│ Random Failures     │ Kill random pods to test redundancy        │
│ Cascading Failures  │ Test dependency chain reactions             │
│ Resource Exhaustion │ CPU/Memory/Disk pressure scenarios         │
│ Network Partitions  │ Split brain and communication failures     │
│ Slow Dependencies   │ Simulate slow external services            │
│ Data Corruption     │ Test backup and recovery procedures        │
│ Time Skew          │ Clock synchronization issues               │
│ Security Failures   │ Certificate expiration, auth failures     │
└─────────────────────┴─────────────────────────────────────────────┘
```

---

## 📋 Глава 7.5: Compliance и аудит (2-3 дня)

### CIS Benchmarks для Kubernetes

#### CIS Kubernetes Benchmark Structure:

```
┌─────────────────────────────────────────────────────────────────┐
│                  CIS KUBERNETES BENCHMARK                      │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                CONTROL PLANE SECURITY                      ││
│  │  ┌───────────────────────────────────────────────────────┐  ││
│  │  │ 1. Master Node Security                               │  ││
│  │  │    • API Server configuration                         │  ││
│  │  │    • etcd security settings                           │  ││
│  │  │    • Controller Manager settings                      │  ││
│  │  └───────────────────────────────────────────────────────┘  ││
│  └─────────────────────────┬───────────────────────────────────┘│
│                            │                                    │
│  ┌─────────────────────────▼───────────────────────────────────┐│
│  │                 WORKER NODE SECURITY                       ││
│  │  ┌───────────────────────────────────────────────────────┐  ││
│  │  │ 2. Worker Node Security                               │  ││
│  │  │    • kubelet configuration                            │  ││
│  │  │    • Container runtime security                       │  ││
│  │  │    • Node hardening                                   │  ││
│  │  └───────────────────────────────────────────────────────┘  ││
│  └─────────────────────────┬───────────────────────────────────┘│
│                            │                                    │
│  ┌─────────────────────────▼───────────────────────────────────┐│
│  │                  POLICIES & PROCEDURES                     ││
│  │  ┌───────────────────────────────────────────────────────┐  ││
│  │  │ 3. Policies                                           │  ││
│  │  │    • RBAC configuration                               │  ││
│  │  │    • Network policies                                 │  ││
│  │  │    • Pod security policies                            │  ││
│  │  └───────────────────────────────────────────────────────┘  ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

#### Key CIS Controls для Kubernetes:

**1. API Server Security:**
- Enable TLS for all API communications
- Configure appropriate authentication mechanisms
- Enable audit logging with comprehensive event capture
- Restrict API server binding to secure interfaces

**2. etcd Security:**
- Enable client certificate authentication
- Use TLS for peer communication
- Implement proper file permissions
- Configure backup encryption

### SOC 2 Compliance в Kubernetes

#### SOC 2 Trust Criteria применительно к Kubernetes:

```
┌─────────────────────────────────────────────────────────────────┐
│                    SOC 2 TRUST CRITERIA                        │
├─────────────────────┬─────────────────────────────────────────────┤
│     CRITERIA        │         KUBERNETES IMPLEMENTATION          │
├─────────────────────┼─────────────────────────────────────────────┤
│ Security            │ • RBAC and authentication                  │
│                     │ • Network policies                         │
│                     │ • Pod security standards                   │
│                     │ • Secret management                        │
├─────────────────────┼─────────────────────────────────────────────┤
│ Availability        │ • Multi-zone deployments                   │
│                     │ • Health checks and probes                 │
│                     │ • Autoscaling capabilities                 │
│                     │ • Disaster recovery procedures             │
├─────────────────────┼─────────────────────────────────────────────┤
│ Processing Integrity│ • Input validation in applications         │
│                     │ • Admission controllers                     │
│                     │ • Policy enforcement                       │
├─────────────────────┼─────────────────────────────────────────────┤
│ Confidentiality     │ • Encryption at rest and in transit       │
│                     │ • Secrets management                       │
│                     │ • Network segmentation                     │
├─────────────────────┼─────────────────────────────────────────────┤
│ Privacy             │ • Data minimization practices              │
│                     │ • Access controls                          │
│                     │ • Data retention policies                  │
└─────────────────────┴─────────────────────────────────────────────┘
```

### Audit Logging и Analysis

#### Kubernetes Audit Policy Levels:

```
┌─────────────────────────────────────────────────────────────────┐
│                   KUBERNETES AUDIT LEVELS                      │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  NONE    │  No logging                                     ││
│  └──────────┼─────────────────────────────────────────────────┘│
│             │                                                  │
│  ┌──────────▼─────────────────────────────────────────────────┐│
│  │  METADATA│  Log request metadata only                     ││
│  │          │  • User, timestamp, resource                   ││
│  │          │  • No request/response bodies                  ││
│  └──────────┼─────────────────────────────────────────────────┘│
│             │                                                  │
│  ┌──────────▼─────────────────────────────────────────────────┐│
│  │  REQUEST │  Log metadata + request body                   ││
│  │          │  • What was requested                          ││
│  │          │  • Who requested it                            ││
│  └──────────┼─────────────────────────────────────────────────┘│
│             │                                                  │
│  ┌──────────▼─────────────────────────────────────────────────┐│
│  │ REQUESTRESPONSE │ Log metadata + request + response      ││
│  │          │  • Complete audit trail                        ││
│  │          │  • Highest verbosity                          ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

#### Audit Log Analysis Workflow:

```
┌─────────────────────────────────────────────────────────────────┐
│                   AUDIT LOG ANALYSIS                           │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │  1. LOG COLLECTION                                          ││
│  │     Kubernetes API Server → Log Files → Log Aggregation    ││
│  └─────────────────────────┬───────────────────────────────────┘│
│                            │                                    │
│  ┌─────────────────────────▼───────────────────────────────────┐│
│  │  2. LOG PARSING & ENRICHMENT                               ││
│  │     • Parse JSON audit events                              ││
│  │     • Add context (user groups, resource types)           ││
│  │     • Geographic information                               ││
│  └─────────────────────────┬───────────────────────────────────┘│
│                            │                                    │
│  ┌─────────────────────────▼───────────────────────────────────┐│
│  │  3. ANOMALY DETECTION                                      ││
│  │     • Unusual access patterns                              ││
│  │     • Privilege escalation attempts                        ││
│  │     • Off-hours activity                                   ││
│  └─────────────────────────┬───────────────────────────────────┘│
│                            │                                    │
│  ┌─────────────────────────▼───────────────────────────────────┐│
│  │  4. COMPLIANCE REPORTING                                   ││
│  │     • Generate compliance reports                          ││
│  │     • Track policy violations                              ││
│  │     • Audit trail documentation                            ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

### Policy Enforcement

#### Policy as Code Ecosystem:

```
┌─────────────────────────────────────────────────────────────────┐
│                   POLICY AS CODE STACK                         │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                  POLICY DEFINITION                          ││
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ ││
│  │  │    OPA      │  │   Falco     │  │    Pod Security     │ ││
│  │  │ Gatekeeper  │  │   Rules     │  │     Standards       │ ││
│  │  │             │  │             │  │                     │ ││
│  │  │ Admission   │  │ Runtime     │  │ Built-in K8s        │ ││
│  │  │ Control     │  │ Security    │  │ Security            │ ││
│  │  └─────────────┘  └─────────────┘  └─────────────────────┘ ││
│  └─────────────────────────┬───────────────────────────────────┘│
│                            │                                    │
│  ┌─────────────────────────▼───────────────────────────────────┐│
│  │                POLICY ENFORCEMENT                          ││
│  │  ┌───────────────────────────────────────────────────────┐  ││
│  │  │              ADMISSION PHASE                          │  ││
│  │  │  • Validate resources before creation                 │  ││
│  │  │  • Mutate resources (add labels, sidecars)           │  ││
│  │  │  • Reject non-compliant resources                     │  ││
│  │  └───────────────────────────────────────────────────────┘  ││
│  │  ┌───────────────────────────────────────────────────────┐  ││
│  │  │               RUNTIME PHASE                           │  ││
│  │  │  • Monitor running workloads                          │  ││
│  │  │  • Detect policy violations                           │  ││
│  │  │  • Generate alerts and reports                        │  ││
│  │  └───────────────────────────────────────────────────────┘  ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Финальный проект: Production-Ready кластер

### Архитектурные требования:

```
┌─────────────────────────────────────────────────────────────────┐
│               PRODUCTION ARCHITECTURE OVERVIEW                 │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                      INGRESS LAYER                         ││
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ ││
│  │  │   ALB/NLB   │  │   Ingress   │  │        WAF          │ ││
│  │  │             │  │ Controller  │  │                     │ ││
│  │  └─────────────┘  └─────────────┘  └─────────────────────┘ ││
│  └─────────────────────────┬───────────────────────────────────┘│
│                            │                                    │
│  ┌─────────────────────────▼───────────────────────────────────┐│
│  │                  APPLICATION LAYER                         ││
│  │  ┌───────────────────────────────────────────────────────┐  ││
│  │  │ Microservices with:                                   │  ││
│  │  │ • Health checks                                       │  ││
│  │  │ • Resource limits                                     │  ││
│  │  │ • Security contexts                                   │  ││
│  │  │ • Service mesh (optional)                             │  ││
│  │  └───────────────────────────────────────────────────────┘  ││
│  └─────────────────────────┬───────────────────────────────────┘│
│                            │                                    │
│  ┌─────────────────────────▼───────────────────────────────────┐│
│  │                    DATA LAYER                              ││
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ ││
│  │  │  Databases  │  │    Cache    │  │   Message Queue     │ ││
│  │  │ (Stateful)  │  │   (Redis)   │  │      (Kafka)        │ ││
│  │  └─────────────┘  └─────────────┘  └─────────────────────┘ ││
│  └─────────────────────────┬───────────────────────────────────┘│
│                            │                                    │
│  ┌─────────────────────────▼───────────────────────────────────┐│
│  │                INFRASTRUCTURE LAYER                        ││
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ ││
│  │  │ Monitoring  │  │   Logging   │  │      Security       │ ││
│  │  │ (Prometheus)│  │    (ELK)    │  │   (Falco, OPA)      │ ││
│  │  └─────────────┘  └─────────────┘  └─────────────────────┘ ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

### Проектные этапы:

**Этап 1: Инфраструктурное планирование (1-2 дня)**
- Выбор cloud provider и managed Kubernetes
- Network design и security groups
- Storage strategy и backup планирование

**Этап 2: Bootstrap кластера (2-3 дня)**
- Infrastructure as Code setup
- GitOps repository structure
- Initial cluster configuration

**Этап 3: Security hardening (2-3 дня)**
- RBAC configuration
- Pod Security Standards
- Network policies implementation

**Этап 4: Observability setup (2-3 дня)**
- Monitoring stack deployment
- Logging aggregation
- Alerting configuration

**Этап 5: Application deployment (3-4 дня)**
- Multi-tier application with database
- CI/CD pipeline setup
- Progressive deployment strategy

**Этап 6: Chaos testing (1-2 дня)**
- Fault injection experiments
- Disaster recovery testing
- Performance validation

---

## 🎓 Итоговые компетенции эксперта после Блока 7

После завершения этого блока вы будете обладать следующими production-ready навыками:

### Операционная готовность:
- **Multi-cloud expertise:** Понимание особенностей EKS, GKE, AKS
- **GitOps mastery:** Полный цикл от кода до production
- **Compliance knowledge:** SOC 2, PCI DSS, CIS benchmarks
- **Chaos engineering:** Proactive resilience testing

### Архитектурное мышление:
- **Infrastructure patterns:** Scalable и resilient design
- **Security-first approach:** Defense in depth strategies  
- **Cost optimization:** Resource efficiency и FinOps practices
- **Disaster recovery:** Business continuity planning

### DevOps интеграция:
- **CI/CD automation:** End-to-end pipeline design
- **Policy as Code:** Automated governance
- **Observability:** Comprehensive monitoring strategies
- **Team enablement:** Self-service platform creation

Поздравляем с завершением продвинутого блока Kubernetes! 🚀 Теперь вы готовы к архитектурным и лидерским ролям в enterprise Kubernetes environments