# 🚀 Kubernetes Challenge — Orquestração de APIs

Projeto desenvolvido para implementar e gerenciar três APIs Node.js em um ambiente Kubernetes, utilizando Docker para containerização, Kind para provisionamento do cluster local e recursos nativos do Kubernetes para gerenciamento, persistência e escalabilidade das aplicações.

A infraestrutura contempla banco de dados PostgreSQL, cache Redis, armazenamento persistente, gerenciamento de configurações e escalonamento horizontal automático baseado no consumo de CPU.

## 🎯 Objetivos do projeto

- Containerizar três aplicações Node.js utilizando Docker.
- Provisionar um cluster Kubernetes local com Kind.
- Realizar o deploy e o gerenciamento das três APIs.
- Configurar a comunicação interna entre os serviços.
- Implementar persistência de dados utilizando PersistentVolumes (PV) e PersistentVolumeClaims (PVC).
- Centralizar configurações utilizando ConfigMaps.
- Gerenciar credenciais utilizando Kubernetes Secrets.
- Implementar escalabilidade horizontal com Horizontal Pod Autoscaler (HPA).
- Monitorar recursos computacionais utilizando Metrics Server.

## 🛠️ Tecnologias utilizadas

| Tecnologia | Finalidade |
|---|---|
| Node.js 18 | Desenvolvimento e execução das APIs |
| Docker | Containerização das aplicações |
| Docker Hub | Armazenamento e distribuição das imagens |
| Kubernetes | Orquestração dos containers |
| Kind | Criação do cluster Kubernetes local |
| PostgreSQL 15 | 	Banco de dados relacional |
| Redis 7 | Armazenamento em cache |
| Metrics Server | Coleta de métricas de CPU e memória |
| Horizontal Pod Autoscaler | Escalonamento automático das aplicações |
| kubectl | Gerenciamento dos recursos Kubernetes |

## 🏗️ Arquitetura da solução

A infraestrutura é composta por três APIs independentes, executadas em containers Docker e gerenciadas pelo Kubernetes.

As aplicações compartilham serviços de banco de dados e cache, utilizando Services para comunicação interna dentro do cluster.

O armazenamento persistente é configurado por meio de PersistentVolumes e PersistentVolumeClaims, enquanto ConfigMaps e Secrets são utilizados para disponibilizar configurações e credenciais aos containers.

O Horizontal Pod Autoscaler monitora a utilização de CPU das aplicações e ajusta automaticamente a quantidade de réplicas conforme a demanda.

### Componentes da arquitetura

| Componente | Configuração |
|---|---|
| API 1 | Node.js — Porta 3000 |
| API 2 | Node.js — Porta 3001 |
| API 3 | Node.js — Porta 3002 |
| PostgreSQL | Porta 5432 |
| Redis | Porta 6379 |
| HPA | Mínimo de 2 e máximo de 5 réplicas por API |
| Services | Comunicação interna no cluster |

### Fluxo simplificado

```text
                    DOCKER HUB
                        |
             +----------+----------+
             |          |          |
          API 1       API 2      API 3
             |          |          |
             +----------+----------+
                        |
                  KUBERNETES
                        |
          +-------------+-------------+
          |             |             |
       API 1          API 2          API 3
          |             |             |
          +-------------+-------------+
                        |
                +-------+-------+
                |               |
            POSTGRESQL        REDIS
                |               |
             PVC / PV        PVC / PV
```
*Escalabilidade*: o Metrics Server fornece métricas ao HPA, que pode ajustar automaticamente a quantidade de réplicas de cada API.

## 📂 Estrutura do projeto

```bash
.
├── Desafio 01/
│   ├── Dockerfile
│   └── arquivos da API 1
│
├── Desafio 02/
│   ├── Dockerfile
│   └── arquivos da API 2
│
├── Desafio 03/
│   ├── Dockerfile
│   └── arquivos da API 3
│
└── k8s/
    ├── api.yaml
    ├── configmap.yaml
    ├── secret.yaml
    ├── persistence.yaml
    ├── postgres.yaml
    ├── redis.yaml
    └── hpa.yaml

```
A estrutura acima representa a organização esperada dos arquivos de infraestrutura. Os nomes e caminhos devem corresponder exatamente aos arquivos existentes no repositório.

## ⚙️ Configuração e execução
### 1. Pré-requisitos
Antes de iniciar, certifique-se de possuir as seguintes ferramentas instaladas:

- **Docker** — execução e construção dos containers.
- **Kind** — criação do cluster Kubernetes local.
- **kubectl** — gerenciamento dos recursos Kubernetes.
- **Docker Hub** — publicação das imagens Docker.

Certifique-se também de que o Docker esteja em execução.

### 2. Criação do cluster Kubernetes
Crie um cluster local utilizando Kind:
```bash
kind create cluster --name desafio-k8s
```
Verifique se o cluster foi criado corretamente:
```bash
kubectl cluster-info
kubectl get nodes
```
O nó do cluster deverá apresentar o status Ready.

### 3. Construção das imagens Docker
Cada API possui seu próprio Dockerfile.

Na raiz do projeto, execute os comandos abaixo para construir as imagens:

#### API 1
```bash
docker build \
  -t allexandrevitor/desafio1-api:v1.0.0 \
  "./Desafio 01"
```
#### API 2
```bash
docker build \
  -t allexandrevitor/desafio2-api:v1.0.0 \
  "./Desafio 02"
```
#### API 3
```bash
docker build \
  -t allexandrevitor/desafio3-api:v1.0.0 \
  "./Desafio 03"
```
Verifique as imagens criadas:
```bash
docker images
```

### 4. Publicação das imagens no Docker Hub
Autentique-se no Docker Hub:
```bash
docker login
```
Envie as imagens para o registry:
```bash
docker push allexandrevitor/desafio1-api:v1.0.0
docker push allexandrevitor/desafio2-api:v1.0.0
docker push allexandrevitor/desafio3-api:v1.0.0
```

**Importante:** as referências das imagens nos manifestos Kubernetes devem corresponder exatamente aos nomes, usuários e tags publicados no Docker Hub.

### 5. Configuração dos Secrets
Antes de implantar os serviços, configure as credenciais utilizadas pelo PostgreSQL e Redis.

Para criar o Secret diretamente no cluster:
```bash
kubectl create secret generic api-secrets \
  --from-literal=POSTGRES_USER=admin \
  --from-literal=POSTGRES_PASSWORD=sua_senha_segura \
  --from-literal=REDIS_PASSWORD=sua_senha_redis
```
Verifique a criação:
```bash
kubectl get secrets
```
**Atenção:** não armazene credenciais reais diretamente no Git. O Base64 utilizado nos manifestos Kubernetes não representa criptografia.

Caso utilize o arquivo secret.yaml, configure os valores adequadamente antes de aplicá-lo. Não mantenha credenciais reais versionadas no repositório.

### 6. Implantação dos recursos Kubernetes
Com o cluster, as imagens e as configurações preparados, inicie a implantação dos recursos.

#### 6.1. ConfigMap
Aplique as configurações compartilhadas pelas APIs:
```bash
kubectl apply -f k8s/configmap.yaml
```
#### 6.2. Armazenamento persistente
Crie os volumes e suas respectivas solicitações de armazenamento:
```bash
kubectl apply -f k8s/persistence.yaml
```
Verifique:
```bash
kubectl get pv,pvc
```
#### 6.3. PostgreSQL
Implante o banco de dados:
```bash
kubectl apply -f k8s/postgres.yaml
```
#### 6.4. Redis
Implante o serviço de cache:
```bash
kubectl apply -f k8s/redis.yaml
```
#### 6.5. APIs
Implante as três aplicações:
```bash
kubectl apply -f k8s/api.yaml
```
#### 6.6. Horizontal Pod Autoscaler
Configure o escalonamento automático:
```bash
kubectl apply -f k8s/hpa.yaml
```
Os nomes dos arquivos devem corresponder aos manifestos existentes. Caso os recursos estejam separados em arquivos individuais, ajuste os comandos de acordo com a estrutura do projeto.

### 7. Instalação do Metrics Server
O Horizontal Pod Autoscaler depende do Metrics Server para obter informações sobre a utilização de CPU dos containers.

Instale o Metrics Server:
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
Em ambientes locais com Kind, pode ser necessário ajustar a validação TLS da comunicação com o Kubelet:
```bash
kubectl patch deployment metrics-server \
  -n kube-system \
  --type='json' \
  -p='[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
```
Aguarde a inicialização:
```bash
kubectl rollout status deployment/metrics-server -n kube-system
```
Verifique a disponibilidade das métricas:
```bash
kubectl top nodes
kubectl top pods
```
**Nota de segurança:** a opção --kubelet-insecure-tls desabilita a validação do certificado TLS do Kubelet. Utilize-a somente em ambientes locais de desenvolvimento e testes, nunca como configuração padrão de produção.

## 🔍 Verificação e monitoramento
Após a implantação, utilize os comandos abaixo para acompanhar o funcionamento da infraestrutura.

### Verificar os Pods
```bash
kubectl get pods -o wide
```
### Verificar os Deployments
```bash
kubectl get deployments
```
### Verificar os Services
```bash
kubectl get services
```
### Verificar os volumes persistentes
```bash
kubectl get pv,pvc
```
### Verificar os volumes persistentes
```bash
kubectl get pv,pvc
```
### Verificar o Horizontal Pod Autoscaler
```bash
kubectl get hpa
```
### Consultar métricas de CPU e memória
```bash
kubectl top pods
kubectl top nodes
```
### Consultar os logs das aplicações
```bash
kubectl logs deployment/desafio1-api
kubectl logs deployment/desafio2-api
kubectl logs deployment/desafio3-api
```
### Verificar eventos do cluster
```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

## 🌐 Acesso às aplicações
Os Services das APIs estão configurados para comunicação interna no cluster.

Para acessar uma API localmente, utilize o recurso port-forward do Kubernetes.

### API 1
```bash
kubectl port-forward service/desafio1-service 3000:80
```
Acesse:
```bash
http://localhost:3000
```
### API 2
```bash
kubectl port-forward service/desafio1-service 3001:80
```
Acesse:
```bash
http://localhost:3001
```
### API 3
```bash
kubectl port-forward service/desafio1-service 3002:80
```
Acesse:
```bash
http://localhost:3002
```
**Observação:** execute cada comando em um terminal separado. Os endpoints e rotas disponíveis dependem da implementação de cada API.

## 📈 Escalabilidade automática
O projeto utiliza o Horizontal Pod Autoscaler (HPA) para ajustar automaticamente a quantidade de réplicas das aplicações com base no consumo de CPU.

### Configurações aplicadas

| Parâmetro | Valor |
|---|---|
| Réplicas mínimas | 2 |
| Réplicas máximas | 5 |
| Métrica monitorada | CPU |
| Utilização média desejada | 50% |

Para acompanhar o escalonamento:
```bash
kubectl get hpa -w
```
O Kubernetes poderá aumentar ou reduzir a quantidade de Pods conforme a demanda, respeitando os limites configurados.

**Pré-requisitos:** Metrics Server funcional e resources.requests.cpu definidos nos containers das aplicações.

## 🛠️ Problemas comuns

| Problema | Possível causa | Solução |
|---|---|---|
| ImagePullBackOff | Imagem inexistente ou inacessível | Verificar nome, tag e acesso ao Docker Hub |
| CrashLoopBackOff | Erro durante a inicialização da aplicação | Consultar os logs do Pod |
| Pending | Falta de recursos ou volume indisponível | Verificar eventos e PVCs |
| PVC Pending | Volume persistente não encontrado ou incompatível | Conferir PV, PVC e StorageClass |
| HPA Unknown | Metrics Server indisponível | Verificar instalação e métricas |
| Erro de conexão com PostgreSQL | Configuração incorreta ou serviço indisponível | Conferir ConfigMap, Secret e Service |
| Erro de conexão com Redis | Serviço indisponível ou configuração incompatível | Conferir Redis e variáveis de ambiente |

Para investigar um Pod com problemas:
```bash
kubectl describe pod <nome-do-pod>
```
Para consultar seus logs:
```bash
kubectl logs <nome-do-pod>
```

## 🔐 Segurança e boas práticas
- Não versionar senhas, tokens ou credenciais reais no Git.
- Utilizar ferramentas de gerenciamento de Secrets em ambientes produtivos.
- Manter imagens Docker identificadas por versões.
- Definir limites e solicitações de CPU e memória.
- Evitar configurações que desabilitem validações de segurança em produção.
- Utilizar armazenamento persistente adequado ao ambiente de execução.
- Manter as configurações de infraestrutura organizadas e versionadas.

## 🧹 Remoção do ambiente
Caso queira remover os recursos implantados:
```bash
kubectl delete -f k8s/hpa.yaml
kubectl delete -f k8s/api.yaml
kubectl delete -f k8s/redis.yaml
kubectl delete -f k8s/postgres.yaml
kubectl delete -f k8s/persistence.yaml
kubectl delete -f k8s/configmap.yaml
```
Se o Secret tiver sido criado manualmente:
```bash
kubectl delete secret api-secrets
```
Para remover o cluster Kind:
```bash
kind delete cluster --name desafio-k8s
```
**Atenção:** a remoção do cluster pode ocasionar perda dos dados armazenados localmente, dependendo da configuração dos volumes.

## 📌 Considerações finais
Este projeto demonstra a aplicação prática de conceitos de containerização, orquestração de containers, infraestrutura como código, persistência de dados, gerenciamento de configurações e escalabilidade horizontal.

A implementação permite compreender como diferentes serviços podem ser organizados, implantados e gerenciados em um ambiente Kubernetes, utilizando ferramentas amplamente adotadas no ecossistema DevOps.

#### **Principais conceitos aplicados:**

- Containerização com Docker.
- Orquestração com Kubernetes.
- Gerenciamento de configurações com ConfigMaps.
- Gerenciamento de credenciais com Secrets.
- Persistência com PV e PVC.
- Escalabilidade horizontal com HPA.
- Monitoramento de recursos com Metrics Server.
