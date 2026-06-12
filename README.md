<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&height=200&color=gradient&customColorList=0:1F9BD4,50:2E75B6,100:16265F&text=hello-manifests&fontColor=ffffff&fontSize=42&fontAlignY=35&desc=Kubernetes%20Manifests%20para%20GitOps%20com%20ArgoCD&descAlignY=55&descSize=18&animation=twinkling" width="100%" />

[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![ArgoCD](https://img.shields.io/badge/ArgoCD-EF7B4D?style=for-the-badge&logo=argo&logoColor=white)](https://argoproj.github.io/cd/)
[![YAML](https://img.shields.io/badge/YAML-CB171E?style=for-the-badge&logo=yaml&logoColor=white)](https://yaml.org/)

[![Manifests](https://img.shields.io/badge/tipo-manifests-blue?style=flat-square)](https://kubernetes.io/docs/concepts/workloads/)
[![GitOps](https://img.shields.io/badge/padrão-GitOps-orange?style=flat-square)](https://www.gitops.tech/)
[![Status](https://img.shields.io/badge/status-ativo-brightgreen?style=flat-square)]()
[![Licença](https://img.shields.io/badge/licença-MIT-lightgrey?style=flat-square)]()

</div>

---

## 📋 Sobre o Projeto

O **hello-manifests** é um repositório dedicado ao armazenamento de **manifests Kubernetes** utilizados em um fluxo de GitOps com **ArgoCD**. Seguindo as práticas modernas de infraestrutura declarativa, este repositório serve como a **fonte da verdade** para o estado desejado das aplicações no cluster Kubernetes.

Quando o pipeline CI/CD do repositório [hello-app](https://github.com/fassir/hello-app) conclui o build e o push da imagem Docker, o ArgoCD monitora este repositório e automaticamente sincroniza os manifests com o cluster — garantindo que o ambiente de execução reflita sempre o que está versionado aqui.

---

<div align="center">

## 🛠️ Stack de Tecnologias

[![My Skills](https://skillicons.dev/icons?i=kubernetes,docker,github&theme=dark)](https://skillicons.dev)

</div>

| Tecnologia | Versão | Finalidade |
|---|---|---|
| **Kubernetes** | 1.28+ | Orquestração de containers |
| **ArgoCD** | 2.x | Operador GitOps / sincronização contínua |
| **YAML** | — | Linguagem de manifests declarativos |
| **kubectl** | 1.28+ | CLI para interação com o cluster |

---

## 📁 Estrutura do Repositório

```
hello-manifests/
│
├── deployment.yaml          # Manifest de Deployment (raiz)
├── service.yaml             # Manifest de Service (raiz)
│
└── hello-app/
    ├── deployment.yaml      # Deployment da aplicação hello-app
    └── service.yaml         # Service da aplicação hello-app
```

---

## 📄 Manifests Explicados

<details>
<summary>📦 <strong>deployment.yaml — Deployment Principal</strong></summary>

<br>

O manifest de `Deployment` define como a aplicação deve ser executada no cluster:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-app
  labels:
    app: hello-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-app
  template:
    metadata:
      labels:
        app: hello-app
    spec:
      containers:
        - name: hello-app
          image: ghcr.io/fassir/hello-app:latest
          ports:
            - containerPort: 8000
          resources:
            requests:
              memory: "64Mi"
              cpu: "250m"
            limits:
              memory: "128Mi"
              cpu: "500m"
```

**Principais configurações:**
- `replicas: 2` — garante alta disponibilidade com 2 pods em execução
- `image` — referencia a imagem construída pelo pipeline CI/CD
- `resources` — define limites e pedidos de CPU/memória

</details>

<details>
<summary>🌐 <strong>service.yaml — Exposição da Aplicação</strong></summary>

<br>

O manifest de `Service` expõe os pods internamente no cluster (ou externamente via NodePort/LoadBalancer):

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-app-service
spec:
  selector:
    app: hello-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8000
  type: ClusterIP
```

**Principais configurações:**
- `selector` — mapeia o service para os pods com a label `app: hello-app`
- `port: 80` — porta exposta pelo service
- `targetPort: 8000` — porta em que o container está ouvindo
- `type: ClusterIP` — acessível apenas dentro do cluster

</details>

---

## 🔄 Fluxo GitOps com ArgoCD

```
┌─────────────────────────────────────────────────────────────────┐
│                      FLUXO GITOPS                               │
│                                                                 │
│  [Dev faz push]                                                 │
│       │                                                         │
│       ▼                                                         │
│  ┌──────────┐    build/push    ┌───────────────┐               │
│  │ hello-app│ ──────────────▶  │  Container    │               │
│  │  (CI/CD) │                  │  Registry     │               │
│  └──────────┘                  └───────────────┘               │
│       │                               │                        │
│       │ atualiza tag                  │                        │
│       ▼                               │                        │
│  ┌──────────────────┐                 │                        │
│  │ hello-manifests  │◀────────────────┘                        │
│  │  (este repo)     │                                          │
│  └──────────────────┘                                          │
│       │                                                         │
│       │ monitora                                                │
│       ▼                                                         │
│  ┌──────────┐   sincroniza   ┌──────────────────────┐          │
│  │  ArgoCD  │ ─────────────▶ │  Cluster Kubernetes  │          │
│  └──────────┘                └──────────────────────┘          │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🚀 Como Configurar no ArgoCD

<details>
<summary>⚙️ <strong>Passo a passo: criar Application no ArgoCD</strong></summary>

<br>

### 1. Via interface gráfica do ArgoCD

1. Acesse o painel do ArgoCD: `https://<seu-argocd>/`
2. Clique em **"+ NEW APP"**
3. Preencha os campos:

| Campo | Valor |
|---|---|
| Application Name | `hello-app` |
| Project | `default` |
| Sync Policy | `Automatic` |
| Repository URL | `https://github.com/fassir/hello-manifests` |
| Revision | `HEAD` |
| Path | `hello-app` |
| Cluster URL | `https://kubernetes.default.svc` |
| Namespace | `default` |

4. Clique em **"CREATE"**

### 2. Via manifesto YAML (recomendado para GitOps)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: hello-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/fassir/hello-manifests
    targetRevision: HEAD
    path: hello-app
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

```bash
kubectl apply -f argocd-application.yaml
```

</details>

---

## ✅ Verificando o Deploy

```bash
# Verificar pods em execução
kubectl get pods -l app=hello-app

# Verificar o service
kubectl get svc hello-app-service

# Port-forward para acesso local
kubectl port-forward svc/hello-app-service 8080:80

# Testar a aplicação
curl http://localhost:8080
```

---

## 🔗 Repositório Relacionado

| Repositório | Descrição |
|---|---|
| [fassir/hello-app](https://github.com/fassir/hello-app) | Aplicação FastAPI com pipeline CI/CD (GitHub Actions + Docker) |

---

<details>
<summary>📚 <strong>Conceitos de GitOps</strong></summary>

<br>

**GitOps** é uma prática de DevOps que utiliza o Git como única fonte da verdade para a infraestrutura e configuração de aplicações. Os princípios fundamentais são:

| Princípio | Descrição |
|---|---|
| **Declarativo** | O sistema é descrito declarativamente |
| **Versionado** | O estado desejado é versionado no Git |
| **Pull automático** | Agentes aprovados puxam o estado desejado automaticamente |
| **Reconciliação contínua** | Agentes garantem que o estado atual corresponde ao desejado |

**ArgoCD** implementa esses princípios monitorando repositórios Git e sincronizando automaticamente as mudanças com o cluster Kubernetes.

</details>

---

## 👤 Autor

<div align="center">

| | |
|---|---|
| **Nome** | Fabio Piassi |
| **LinkedIn** | [linkedin.com/in/fabio-piassi](https://linkedin.com/in/fabio-piassi) |
| **GitHub** | [github.com/fassir](https://github.com/fassir) |

</div>

---

<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&section=footer&height=120&color=gradient&customColorList=0:16265F,50:2E75B6,100:1F9BD4" width="100%" />

*Manifests Kubernetes para GitOps declarativo com ArgoCD*

</div>
