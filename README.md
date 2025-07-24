# Introdução
Este repositório serve de suporte para o repositorio [hello-app](https://github.com/fassir/hello-app). Neste repositório consta os arquivos necessários para enviar instruções para um pipeline CI/CD utilizando FastAPI, Docker, GitHub Actions e ArgoCD. 

# Requisitos
Os arquivos são manifestos que descrevem uma infraestrutura para configurar e executar a aplicação FastAPI.

Os Arquivos de execução são estes:

**hello-app/deployment.yaml**
Arquivo que gera a aplicação com o kubernets no ambiente do ArgoCD
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-app-deployment
  labels:
    app: hello-app
spec:
  replicas: 1 # Número de réplicas do pod
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
        # A imagem será atualizada automaticamente pelo GitHub Actions
        # A tag ':latest' é um fallback, mas o CI/CD usará o SHA do commit
        image: seu-usuario-docker/hello-app:latest # Substitua pelo seu Docker Hub username
        ports:
        - containerPort: 8000 # Porta que sua aplicação FastAPI expõe
```
**hello-app/service.yaml**

Arquivo que altera a aplicação do ArgoCD
```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-app-service
  labels:
    app: hello-app
spec:
  selector:
    app: hello-app
  ports:
    - protocol: TCP
      port: 80 # Porta que o serviço Kubernetes expõe
      targetPort: 8000 # Porta do contêiner onde a aplicação está rodando
  type: LoadBalancer # Ou NodePort, ClusterIP, dependendo do seu ambiente Kubernetes
```
# Variáveis globais
* Chave pública para repositório hello-app

Como adicionar a chave pública ao repositório hello-manifests:

Vá para hello-manifests > Settings > Deploy keys.

![image](images/image.png)
![image](images/image-1.png)
Clique em Add deploy key.
![image](images/image-2.png)

**Title**: github-actions-for-hello-app (ou um nome descritivo)

**Key**: Cole o conteúdo do arquivo público: cat ~/.ssh/github_actions_manifests_rsa_key.pub

Marque a opção "Allow write access". Isso é crucial para que o GitHub Actions possa fazer commits/PRs nesse repositório.

Clique em Add key.
