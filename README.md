Etapa 1 – Criar a aplicação FastAPI
Esta etapa envolve a criação do código-fonte da sua aplicação FastAPI e a configuração do Docker para empacotá-la.

1.1 Criar um repositório Git para o projeto (Exemplo: hello-app) com o arquivo main.py
Primeiro, crie um novo repositório no GitHub (ou em sua plataforma Git preferida) chamado hello-app.

Dentro deste repositório, crie um arquivo chamado main.py com o seguinte conteúdo:

# main.py
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    # Mensagem inicial da aplicação
    return {"message": "Hello World"}

Agora, inicialize o Git e adicione o arquivo:

# No diretório do seu projeto hello-app
git init
git add main.py
git commit -m "Initial commit: Add FastAPI app"
git branch -M main
git remote add origin <URL_DO_SEU_REPOSITORIO_HELLO_APP> # Ex: https://github.com/seu-usuario/hello-app.git
git push -u origin main

1.2 Criar um Dockerfile para executar esse aplicativo
No mesmo diretório do seu projeto hello-app, crie um arquivo chamado Dockerfile com o seguinte conteúdo:

# Dockerfile
# Usar uma imagem base oficial do Python
FROM python:3.9-slim-buster

# Definir o diretório de trabalho dentro do contêiner
WORKDIR /app

# Copiar o arquivo de requisitos (se houver) e instalar as dependências
# Para este exemplo simples, a única dependência é FastAPI e Uvicorn
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copiar o código da aplicação para o diretório de trabalho
COPY . .

# Expor a porta em que a aplicação FastAPI será executada
EXPOSE 8000

# Comando para iniciar a aplicação usando Uvicorn
# Uvicorn é um servidor ASGI para aplicações Python
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]

Crie também um arquivo requirements.txt no mesmo diretório:

# requirements.txt
fastapi
uvicorn

Adicione e commite esses novos arquivos ao seu repositório hello-app:

git add Dockerfile requirements.txt
git commit -m "Add Dockerfile and requirements"
git push origin main

1.3 Criar um repositório Git para os manifestos do ArgoCD (exemplo: hello-manifests)
Crie um novo e vazio repositório no GitHub (ou em sua plataforma Git preferida) chamado hello-manifests. Este repositório conterá os arquivos de configuração do Kubernetes (manifestos) que o ArgoCD usará para implantar sua aplicação.

Este repositório pode ser privado ou público. Para maior segurança e para demonstrar a configuração de chaves SSH no GitHub Actions, é comum que este repositório seja privado. Se você optar por torná-lo público, o processo de configuração da chave SSH ainda é válido e robusto para o acesso de escrita.

Você não precisa adicionar nenhum arquivo a ele agora; faremos isso na Etapa 3. Apenas crie o repositório.

1.4 Commitar e Enviar os Arquivos Iniciais
No seu terminal (Git Bash, PowerShell ou CMD) dentro do diretório hello-app:

git add .
git commit -m "Initial commit: Add FastAPI app, Dockerfile, and requirements"
git push origin main

# .github/workflows/main.yml
# Este workflow será executado no repositório 'hello-app'

name: CI/CD Pipeline - Build, Push & Update ArgoCD Manifests

# Define os eventos que irão disparar este workflow
on:
  push:
    branches:
      - main # O workflow será executado sempre que houver um push na branch 'main'

# Define as variáveis de ambiente que serão usadas no workflow
env:
  DOCKER_IMAGE_NAME: seu-usuario-docker/hello-app # Substitua 'seu-usuario-docker' pelo seu username do Docker Hub
  # A tag da imagem será gerada com o hash curto do commit, garantindo unicidade
  IMAGE_TAG: ${{ github.sha }}

jobs:
  build-and-push:
    runs-on: ubuntu-latest # O job será executado em uma máquina virtual Ubuntu

    steps:
      - name: Checkout application code
        uses: actions/checkout@v4 # Faz o checkout do código da sua aplicação (hello-app)

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3 # Configura o Docker Buildx para builds eficientes

      - name: Log in to Docker Hub
        uses: docker/login-action@v3 # Realiza o login no Docker Hub usando os segredos
        with:
          username: ${{ secrets.DOCKER_USERNAME }} # Nome de usuário do Docker Hub (segredo do GitHub)
          password: ${{ secrets.DOCKER_PASSWORD }} # Senha do Docker Hub (segredo do GitHub)

      - name: Build and push Docker image
        uses: docker/build-push-action@v5 # Constrói e envia a imagem para o Docker Hub
        with:
          context: . # O contexto do build é o diretório atual
          push: true # Habilita o push da imagem
          tags: ${{ env.DOCKER_IMAGE_NAME }}:${{ env.IMAGE_TAG }} # Define a tag da imagem (ex: seu-usuario-docker/hello-app:abcdef123)
          # Inclui uma tag 'latest' para facilitar o uso, mas a tag principal será o SHA do commit
          tags: |
            ${{ env.DOCKER_IMAGE_NAME }}:${{ env.IMAGE_TAG }}
            ${{ env.DOCKER_IMAGE_NAME }}:latest

      - name: Checkout ArgoCD manifests repository
        # Faz o checkout do repositório de manifestos (hello-manifests)
        # É importante usar um token com permissão de escrita para este repositório
        uses: actions/checkout@v4
        with:
          repository: seu-usuario-github/hello-manifests # Substitua 'seu-usuario-github' pelo seu username do GitHub
          path: hello-manifests # Onde o repositório será clonado localmente
          # Usar um token SSH para autenticação e permissão de escrita
          # A chave SSH privada deve ser adicionada como um segredo no GitHub
          ssh-key: ${{ secrets.SSH_PRIVATE_KEY }}
          # Desabilitar a verificação do host SSH para ambientes CI/CD
          # Em ambientes de produção, considere verificar o host para maior segurança
          known-hosts: | chave ssh
            DOCKER_USERNAME`, `DOCKER_PASSWORD`, `SSH_PRIVATE_KEY`

Para que o GitHub Actions possa fazer login no Docker Hub e fazer um Pull Request no repositório de manifestos, você precisará configurar alguns segredos no seu repositório `hello-app`.

1.  **DOCKER_USERNAME** e **DOCKER_PASSWORD**:
    * Vá para o seu repositório `hello-app` no GitHub.
    * Clique em `Settings` (Configurações).
    * No menu lateral esquerdo, clique em `Secrets and variables` > `Actions`.
    * Clique em `New repository secret`.
    * Crie um segredo com o nome `DOCKER_USERNAME` e o valor do seu nome de usuário do Docker Hub.
    * Crie outro segredo com o nome `DOCKER_PASSWORD` e o valor da sua senha do Docker Hub.

2.  **SSH_PRIVATE_KEY**:
    * **Gerar um par de chaves SSH:** Abra seu terminal e execute:
        ```bash
        ssh-keygen -t rsa -b 4096 -C "github-actions-key" -f ~/.ssh/github_actions_rsa
        ```
        Quando perguntado por uma passphrase, **deixe em branco** (apenas pressione Enter duas vezes), pois o GitHub Actions não pode lidar com chaves protegidas por passphrase.
    * Isso criará dois arquivos: `github_actions_rsa` (sua chave privada) e `github_actions_rsa.pub` (sua chave pública).
    * **Adicionar a chave pública ao seu repositório `hello-manifests`:**
        * Vá para o repositório `hello-manifests` no GitHub.
        * Clique em `Settings` (Configurações).
        * No menu lateral esquerdo, clique em `Deploy keys`.
        * Clique em `Add deploy key`.
        * Dê um título (ex: `github-actions-hello-app`).
        * Cole o **conteúdo completo** do arquivo `~/.ssh/github_actions_rsa.pub` (começa com `ssh-rsa ...`).
        * **Marque a opção "Allow write access"** (Permitir acesso de escrita). Isso é crucial para que o GitHub Actions possa fazer o Pull Request.
        * Clique em `Add key`.
    * **Adicionar a chave privada como um segredo no repositório `hello-app`:**
        * Volte para o repositório `hello-app` no GitHub.
        * Vá para `Settings` > `Secrets and variables` > `Actions`.
        * Clique em `New repository secret`.
        * Crie um segredo com o nome `SSH_PRIVATE_KEY`.
        * Cole o **conteúdo completo** do arquivo `~/.ssh/github_actions_rsa` (começa com `-----BEGIN OPENSSH PRIVATE KEY-----` e termina com `-----END OPENSSH PRIVATE KEY-----`). Certifique-se de copiar tudo, incluindo as linhas de `BEGIN` e `END`.

3.  **Ter acesso de gravação ao repositório de manifests usado pelo ArgoCD:**
    * Isso foi configurado no passo anterior ao adicionar a chave de deploy com permissão de escrita no repositório `hello-manifests`.

Com esses segredos configurados e o arquivo de workflow no lugar, seu pipeline de CI/CD está pronto para ser acionado em cada push para a branch `main` do seu repositório `hello-app`.

Etapa 3 – Repositório Git com os manifests do ArgoCD
Nesta etapa, você criará os arquivos de manifesto do Kubernetes (deployment.yaml e service.yaml) que o ArgoCD usará para implantar sua aplicação. Estes arquivos serão armazenados no repositório hello-manifests que você criou anteriormente.

3.1 Criar os manifestos do Kubernetes de deployment e service para o Hello App no repositório de manifestos
Clone o repositório hello-manifests para sua máquina local:

git clone git@github.com:seu-usuario-github/hello-manifests.git # Use SSH para clonar
cd hello-manifests

Dentro do diretório hello-manifests, crie uma pasta para sua aplicação, por exemplo, hello-app/.

mkdir hello-app
cd hello-app

Agora, crie o arquivo deployment.yaml dentro da pasta hello-app com o seguinte conteúdo. Lembre-se de substituir seu-usuario-docker pelo seu nome de usuário do Docker Hub.

# hello-app/deployment.yaml
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

Em seguida, crie o arquivo service.yaml no mesmo diretório hello-app para expor sua aplicação:

# hello-app/service.yaml
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

Adicione e commite esses manifestos ao seu repositório hello-manifests:

git add .
git commit -m "Add initial Kubernetes manifests for hello-app"
git push origin main

Este repositório agora está pronto para ser monitorado pelo ArgoCD.

Etapa 4 – Criar o App no ArgoCD
Esta etapa assume que você já tem o ArgoCD instalado e funcionando em seu cluster Kubernetes. Se não tiver, siga a documentação oficial do ArgoCD para instalação.

A criação do aplicativo no ArgoCD pode ser feita via interface web (UI) ou via argocd CLI. Vamos descrever os passos para a UI, que é mais visual.

4.1 Na interface do ArgoCD criar o vínculo com o repositório de manifestos
Acesse a UI do ArgoCD: Abra seu navegador e navegue até a URL do seu painel do ArgoCD (geralmente https://argocd.example.com ou http://localhost:8080 se estiver usando port-forward).

Faça login: Utilize suas credenciais de administrador do ArgoCD.

Adicionar novo repositório:

No menu lateral esquerdo, clique em Settings (Configurações).

Em Repositories, clique em + New Repo.

Preencha os detalhes do seu repositório hello-manifests:

Repository URL: https://github.com/seu-usuario-github/hello-manifests.git (substitua seu-usuario-github).

Name: hello-manifests (ou um nome de sua preferência).

Authentication: Se o repositório for público, não precisa de credenciais. Se for privado, você pode usar um token de acesso pessoal do GitHub ou SSH. Para SSH, use a mesma chave privada que você configurou no GitHub Actions (mas certifique-se de que ela não tem passphrase).

Clique em CONNECT.

O ArgoCD agora tem acesso ao seu repositório de manifestos e pode monitorá-lo.

4.2 Criar o app no ArgoCD
Crie uma nova aplicação:

No menu lateral esquerdo, clique em Applications.

Clique no botão + NEW APP.

Preencha os detalhes da nova aplicação:

Application Name: hello-app

Project: default (ou o projeto que você preferir)

SYNC POLICY: Selecione Automatic e marque PRUNE e SELF HEAL. Isso fará com que o ArgoCD sincronize automaticamente as mudanças no Git para o cluster e remova recursos que não estão mais no Git.

Repository URL: Selecione o repositório hello-manifests que você acabou de adicionar.

Revision: HEAD (ou main para a branch principal).

Path: hello-app (o diretório dentro do seu repositório hello-manifests onde estão seus manifestos).

Cluster: in-cluster (se você estiver implantando no mesmo cluster onde o ArgoCD está rodando) ou selecione o cluster de destino.

Namespace: default (ou o namespace onde você quer implantar a aplicação).

Clique em CREATE.

O ArgoCD irá agora detectar os manifestos no seu repositório hello-manifests e iniciará o processo de sincronização, implantando o hello-app-deployment e o hello-app-service no seu cluster Kubernetes. Você poderá ver o status da aplicação na interface do ArgoCD.

Etapa 5 – Acessar e testar a aplicação localmente
Esta etapa final verifica se sua aplicação foi implantada com sucesso e se a pipeline de CI/CD está funcionando como esperado.

5.1 Usando o port-forward para rotear as portas do serviço, acesse via navegador: http://localhost:8080/
Assumindo que sua aplicação hello-app foi implantada no namespace default e o serviço se chama hello-app-service, você pode usar o kubectl port-forward para acessar a aplicação localmente:

Verifique se o serviço está rodando:

kubectl get svc -n default

Você deve ver hello-app-service na lista.

Encaminhe a porta:

kubectl port-forward svc/hello-app-service 8080:80 -n default

Este comando encaminhará o tráfego da porta 8080 da sua máquina local para a porta 80 do serviço hello-app-service dentro do cluster.

Acesse no navegador:
Abra seu navegador e navegue para http://localhost:8080/. Você deverá ver a mensagem:

{"message": "Hello World"}

Isso confirma que sua aplicação está rodando e acessível.

5.2 Alterar o repositório da aplicação, modificando a mensagem dentro do código python de "Hello World" para qualquer outra mensagem e verificando se após o processo de CI/CD a imagem foi atualizada no ambiente Kubernetes.
Agora, vamos testar o fluxo completo de CI/CD:

Modifique o arquivo main.py:
No seu repositório hello-app (localmente), edite o arquivo main.py e altere a mensagem:

# main.py
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    # Nova mensagem da aplicação
    return {"message": "Olá do CI/CD com ArgoCD!"}

Commite e faça o push da alteração:

cd /caminho/para/seu/hello-app
git add main.py
git commit -m "Update message to Olá do CI/CD com ArgoCD!"
git push origin main

Monitore o GitHub Actions:

Vá para o seu repositório hello-app no GitHub, na aba Actions.

Você verá um novo workflow sendo executado. Monitore o progresso.

A pipeline irá:

Construir uma nova imagem Docker com a nova tag (SHA do commit).

Fazer o push dessa imagem para o Docker Hub.

Clonar o repositório hello-manifests.

Atualizar o arquivo hello-app/deployment.yaml com a nova tag da imagem.

Fazer um commit e push dessa alteração para o hello-manifests.

Monitore o ArgoCD:

Vá para a interface do ArgoCD.

Você verá que a aplicação hello-app entrará em estado OutOfSync (Fora de Sincronia) porque o repositório de manifestos foi atualizado.

Devido à política de sincronização automática (Automatic com SELF HEAL), o ArgoCD detectará a mudança e iniciará automaticamente a sincronização.

Aguarde até que a aplicação volte para o estado Synced (Sincronizado) e Healthy (Saudável). Isso significa que o Kubernetes atualizou o deployment com a nova imagem.

Verifique a aplicação atualizada:

Mantenha o kubectl port-forward rodando (se ele parou, execute novamente).

Atualize a página no seu navegador (http://localhost:8080/).

Você deverá ver a nova mensagem:

{"message": "Olá do CI/CD com ArgoCD!"}

Parabéns! Você configurou com sucesso uma pipeline de CI/CD completa usando FastAPI, Docker, GitHub Actions e ArgoCD. Agora, qualquer alteração no seu código FastAPI será automaticamente construída, publicada e implantada no seu cluster Kubernetes.

Entregas Esperadas
Para validar a implementação da pipeline de CI/CD, você precisará fornecer as seguintes evidências:

✅ Link do repositório Git com a aplicação FastAPI + Dockerfile + GitHub Actions:

Forneça o link para o seu repositório hello-app no GitHub. Este repositório deve conter o main.py, Dockerfile, requirements.txt e a pasta .github/workflows com o arquivo main.yml do GitHub Actions.

✅ Link do repositório com os manifests (deployment.yaml, service.yaml):

Forneça o link para o seu repositório hello-manifests no GitHub. Este repositório deve conter a estrutura de pastas (ex: hello-app/) com os arquivos deployment.yaml e service.yaml.

✅ Evidência de build e push da imagem no Docker Hub:

Uma captura de tela ou link direto para a página da sua imagem no Docker Hub, mostrando que a imagem seu-usuario-docker/hello-app (com as tags latest e a tag do SHA do commit) foi publicada com sucesso.

Alternativamente, uma captura de tela do log do GitHub Actions mostrando o passo de "Build and push Docker image" concluído com sucesso.

✅ Evidência de atualização automática dos manifests com a nova tag da imagem:

Uma captura de tela do histórico de commits do seu repositório hello-manifests no GitHub, mostrando um commit automático feito pelo GitHub Actions que atualizou a tag da imagem no deployment.yaml.

Ou, uma captura de tela do log do GitHub Actions mostrando o passo de "Update ArgoCD manifests" (ou similar) que realizou o push para o repositório de manifests.

✅ Captura de tela do ArgoCD com a aplicação sincronizada:

Uma captura de tela da interface do ArgoCD mostrando o status da sua aplicação hello-app como Synced (Sincronizado) e Healthy (Saudável), indicando que a implantação foi bem-sucedida.

✅ Print do kubectl get pods com a aplicação rodando:

Uma captura de tela do seu terminal executando o comando kubectl get pods -n <seu-namespace> (ex: kubectl get pods -n default) e mostrando o pod da sua aplicação hello-app-deployment em estado Running.

✅ Print da resposta da aplicação via curl ou navegador:

Uma captura de tela do seu navegador acessando http://localhost:8080/ (após executar o kubectl port-forward) e exibindo a mensagem de resposta da sua aplicação FastAPI (ex: {"message": "Olá do CI/CD com ArgoCD!"}).

Alternativamente, uma captura de tela do seu terminal executando curl http://localhost:8080/ e mostrando a mesma mensagem de resposta.