# DevOps 4 Devs

### :memo: Aula 01 - 01/04/2024

Realizado o fork do projeto: `https://github.com/KubeDev/conversao-temperatura`

Com o projeto local na maquina, dentro do diretório `src` executar a construção da imagem do container Docker: `docker build -t conversor-temperatura .`

### :memo: Aula 02 - 02/04/2024 
Instalação do k3d: `curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash`

Criando um cluster kubernetes localmente: `k3d cluster create`

Listar nós kubernetes: `kubectl get nodes`

Após a criação do cluster, são criados três containers para o gerenciamento do cluster. `docker container ls`

```
CONTAINER ID   IMAGE                            COMMAND                  CREATED         STATUS         PORTS                             NAMES
1708451c1063   ghcr.io/k3d-io/k3d-tools:5.6.0   "/app/k3d-tools noop"    3 minutes ago   Up 3 minutes                                     k3d-k3s-default-tools
ffc7b3b2b68b   ghcr.io/k3d-io/k3d-proxy:5.6.0   "/bin/sh -c nginx-pr…"   3 minutes ago   Up 3 minutes   80/tcp, 0.0.0.0:34891->6443/tcp   k3d-k3s-default-serverlb
7ecc2d47da40   rancher/k3s:v1.27.4-k3s1         "/bin/k3s server --t…"   3 minutes ago   Up 3 minutes                                     k3d-k3s-default-server-0
```

Listar os clusters criados: `k3d cluster list`
```
NAME          SERVERS   AGENTS   LOADBALANCER
k3s-default   1/1       0/0      true
```

Criando um cluster com mais informações: `k3d cluster create meucluster --servers 3 --agents 3`

Excluir um cluster: `k3d cluster delete nome_cluster`

Criando o Deployment e o Service do postgre: `kubectl apply -f k8s/deploy.yaml`
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgre
spec:
  selector:
    matchLabels:
      app: postgre
  template:
    metadata:
      labels:
        app: postgre
    spec:
      containers:
        - name: postgre
          image: postgres:15.0
          ports:
            - containerPort: 5432
          env:
            - name: POSTGRES_USER
              value: "newuser"
            - name: POSTGRES_PASSWORD
              value: "newspwd"
            - name: POSTGRES_DB
              value: "news"
---
apiVersion: v1
kind: Service
metadata:
  name: postgre
spec:
  selector:
    app: postgre
  ports:
    - port: 5432
```

Listar os Services: `kubectl get service`
```
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes   ClusterIP   10.43.0.1       <none>        443/TCP    32m
postgre      ClusterIP   10.43.213.223   <none>        5432/TCP   78s
```

Listar os Deployments: `kubectl get deployments`
```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
postgre            1/1     1            1           7m4s
```

Listar os Replicasets: `kubectl get replicaset`
```
NAME                         DESIRED   CURRENT   READY   AGE
postgre-6dc7c9f6dc           1         1         1       7m12s
```


Listar os Pods: `kubectl get pods`
```
NAME                               READY   STATUS    RESTARTS   AGE
postgre-6dc7c9f6dc-kw2c9           1/1     Running   0          7m38s
```

Configurar o port-forward do banco de dados: `kubectl port-forward nome_do_pod 5432:5432`

Criação de uma imagem localmente para subir no dockerhub: `docker build -t kaan086/devops4devs-news:v1 .`

Logar no DockerHub: `docker login`

Subir imagem para o DockerHub: `docker push kaan86/devops4devs-news:v1` 

Criar imagem latest: `docker tag kaan86/devops4devs-news:v1 kaan86/devops4devs-news`

Subir imagem latest para o DockerHub: `docker push kaan86/devops4devs-news`

Configurar o manifesto da aplicação web: `kubectl apply -f k8s/deploy.yaml`
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: web
          image: kaan86/devops4devs-news:v1
          ports:
            - containerPort: 8080
          env:
            - name: DB_DATABASE
              value: "news"
            - name: DB_USERNAME
              value: "newuser" 
            - name: DB_PASSWORD
              value: "newspwd"
            - name: DB_HOST
              value: postgre
```

Criar o Service do container web e atualiza o manifesto:
```
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 8080
  type: NodePort
```

Recriar o cluster com loadbalancer: `k3d cluster create meucluster --servers 3 --agents 3 -p "30000:30000@loadbalancer"`

Consultar o histórico dos deployments: `kubectl rollout history deployment web`

<<<<<<< HEAD
Fazer o rollback: `kubectl rollout undo deployment web`

### :memo: Aula 03 - 04/04/2024 

# AWS - IAM

Instalação do AWS LCI: `https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html`

Criação de um usuario para gerenciamento no IAM.
Policies - Politicas de acesso.
Users - Usuários com acesso ao gerenciamento.
User Groups - Permissões baseadas em grupos.
Roles - Autenticação de um servico AWS.

Criar um grupo: administrador com a police AdministratorAccess.
Criar um usuario: administrador
 - Provider user acesss to the AWS.
 - Tipo de acesso: I want to create an IAM user;
 - Console password: escolha da senha ou gerar uma senha.
 - Permissions options: adicionar o grupo.
 - No final irá aparecer o Console sign-in details.
 - Logar usando as credenciais recebidas acima para não utilizar um conta root.

Criando uma secret para o usuario utilizar o cli
- No IAM na parte de usuarios.
- Access key, será gerado um Access key e secrect access key.
- Depois de gerado as informações acima, no terminal executar o comando `aws configure` e preencher as informações abaixo:
```
aws configure
AWS Access Key ID [None]: xxxxxxx
AWS Secret Access Key [None]: xxxxxxx
Default region name [None]: us-east-1
Default output format [None]:
```

Criação e configuração do S3(armazenamento)
Pesquisar por S3.
Criar um bucket, escolher a região, o nome único, ACL enabled, liberar o acesso ao bucker, desabilitar o versionamento.

Alterações no projeto:
Adicionar no manifesto as configurações da AWS:
```
- name: AWS_ACCESS_KEY
  value: "xxxxxx"
- name: AWS_ACCESS_SECRET
  value: "xxxxxxx"
- name: AWS_S3_BUCKET_NAME
  value: kleber86-devops
- name: STORAGE_TYPE
  value: S3
```
Atualização, construção e subir para o DockerHub:
````
docker build -t kaan086/devops4devs-news:v3 --push . 
```
Criação das Roles para EKS
- No IAM 
- Roles: Create roles, vincular a um serviço EKS Cluster.
- Adicionar o role name e finalizar.
- Criando um nova role para EC2:
- ADD permissions: AmazonEC2ContainerRegistryReadOnly, AmazonEKS_CNI_Policy, AmazonEKSWorkerNodePolicy
- Adicionar o role name e finalizar.

Criação templetes para Redes:
- Em CloudFormation: Create Stack
- Adicionar a url: `https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/amazon-eks-vpc-private-subnets.yaml`
- Stack name para adicionar uma nome e pode seguir até o final.

Criação do cluster:
Elastic Kubernetes Serice:
Adicionar um nome
Escolher a versão
A Role criada anteriomente será adicionada automaticamente.
Specify Network: 
- VPC criada anteriormente
- Security Group criado anteriormente
Cluster endpoint access: Public e Private e seguir até o final.

Configurar o projeto pelo AWS CLI:
`aws eks update-kubeconfig --name nome_do_cluster_criado`

Atualizar o manifesto retirando o NodePorte:
```
      targetPort: 8080
  type: LoadBalancer
```
AWS CLI executar o manifesto: `kubectl applay -f k8s/deploy.yml`

No cluster criado criar os worknodes:
Adicionar Node Groups:
- Adicionar o nome
- Adicionar a Roles que mostrar automaticamente
- Escolher as imagens
- Instance types: Escolher o pefil das maquinas
- Node group scaling: Desired: 2, Min: 1, Max: 3
- Specify Network: Excluir as publicas por ser uma boa pratica.

Consultar os nodes criados: `kubectl get nodes`

Consultar o Serice para pegar o endereço externo: `kubectl get all`

Para o projeto funcionar foi necessario alterar os arquivos do app, os arquivos da aula passada foram do conversor e o desta aula é o kubenews. Com isso as imagens precisaram ser recriadas.

Consultar o histórico dos deployments: `kubectl rollout history deployment web`

Fazer o rollback: `kubectl rollout undo deployment web`
