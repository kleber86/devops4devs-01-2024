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