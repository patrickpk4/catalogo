# 🧩 Catálogo - Aplicação Kubernetes

Este projeto implementa uma aplicação de **Catálogo** com **MongoDB** como banco de dados, totalmente **containerizada e orquestrada com Kubernetes**.  
A arquitetura foi projetada para demonstrar **boas práticas de resiliência, escalabilidade e persistência de dados** em ambiente Kubernetes.

---

## 📋 Arquitetura

### 🧱 Componentes

| Componente | Descrição |
|-------------|------------|
| **API Backend** | Aplicação .NET Core com endpoints REST (imagem `patrickpk4/catalogo:v1.0`) |
| **MongoDB** | Banco de dados NoSQL para armazenamento |
| **NFS Server** | Armazenamento compartilhado para persistência de dados |

---

## ⚙️ Serviços Kubernetes

| Serviço | Tipo | Porta(s) | Função |
|----------|------|-----------|--------|
| `api-service` | NodePort | 80 / 443 | Exposição externa da aplicação |
| `mongo-service` | Headless | 27017 | Comunicação interna com o banco de dados |

---

## 🚀 Deploy

### ✅ Pré-requisitos

- Cluster Kubernetes funcional  
- Servidor NFS configurado em `192.168.1.16` com export `/export`  
- Helm instalado e provisionador NFS configurado (`nfs-subdir-external-provisioner`)  
- Imagem Docker disponível no registro: `patrickpk4/catalogo:v1.0`  

---

### 🪄 Etapas de Deploy

```bash
# 1️⃣ Criar Secrets
kubectl apply -f mongodb-secret.yaml
kubectl apply -f catalogo-secret.yaml

# 2️⃣ Criar StorageClass NFS
kubectl apply -f storage-class-mongodb.yaml

# 3️⃣ Deploy do MongoDB
kubectl apply -f deployment-mongodb.yaml
kubectl apply -f service.yaml  # mongo-service

# 4️⃣ Deploy da API
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml  # api-service

# 5️⃣ Configurar Auto-scaling
kubectl apply -f autoscaling-api.yaml
kubectl apply -f hpa-mongodb.yaml
```

---

## 🔧 Configuração

### 🧩 Variáveis de Ambiente

**Secret do MongoDB (`mongodb-secret.yaml`):**
| Variável | Valor |
|-----------|--------|
| `MONGO_INITDB_ROOT_USERNAME` | `mongouser` |
| `MONGO_INITDB_ROOT_PASSWORD` | `mongopwd` |

**Secret da Aplicação (`catalogo-secret.yaml`):**
| Variável | Valor |
|-----------|--------|
| `Mongo__host` | `mongo-service` |
| `Mongo__port` | `27017` |
| `Mongo__DataBase` | `admin` |

---

## 🩺 Probes de Saúde

| Tipo | Local | Verificação |
|------|--------|--------------|
| **StartupProbe (MongoDB)** | Container MongoDB | `mongo --eval "db.adminCommand('ping')"` |
| **ReadinessProbe (MongoDB)** | Porta TCP 27017 | Conexão disponível |
| **LivenessProbe (MongoDB)** | Container MongoDB | `mongo --eval "db.adminCommand('ping')"` |
| **ReadinessProbe (API)** | Endpoint `/read` | Porta 80 |
| **LivenessProbe (API)** | Endpoint `/health` | Porta 80 |

---

## 📊 Auto-scaling

| Recurso | Tipo | Mínimo | Máximo | Métricas |
|----------|------|---------|---------|-----------|
| **API Deployment** | Deployment | 1 Pod | 10 Pods | CPU 50% / Memória 50% |
| **MongoDB StatefulSet** | StatefulSet | 1 Pod | 10 Pods | CPU 38% / Memória 70% |

---

## 💾 Persistência

| Recurso | Valor |
|----------|--------|
| **StorageClass** | `nfs-mongodb` |
| **Provisionador** | `cluster.local/nfs-new-nfs-subdir-external-provisioner` |
| **Servidor NFS** | `192.168.1.16:/export` |
| **Política** | `Retain` (dados preservados após deleção do PVC) |

---

## 🔍 Monitoramento e Logs

### 🧭 Verificar Status

```bash
kubectl get pods
kubectl get services
kubectl get pvc
kubectl get hpa
```

### 🪵 Logs

```bash
# Logs da API
kubectl logs deployment/api-deployment

# Logs do MongoDB
kubectl logs statefulset/mongodb-statefulset
```

---

## 🛠️ Desenvolvimento

### 🏗️ Build e Push da Imagem

```bash
docker build -t patrickpk4/catalogo:v1.0 .
docker push patrickpk4/catalogo:v1.0
```

### 🔁 Atualizar Deployment

```bash
kubectl rollout restart deployment/api-deployment
```

---

## 🧠 Notas Técnicas

- O **initContainer** da API aguarda o MongoDB estar acessível antes do start.  
- O **MongoDB** é executado como **StatefulSet** para garantir identidade estável.  
- O **NFS** assegura persistência entre reinicializações.  
- O **HPA** aplica escalabilidade automática com base no uso de CPU e memória.  

---

## 🔒 Segurança

- Credenciais armazenadas em **Secrets codificados em Base64**  
- **Resource Limits** definidos para evitar sobrecarga de recursos  
- **Probes configuradas** garantem disponibilidade contínua da aplicação  

---

## ✍️ Autor

**Patrick Amorim**  
Projeto de estudo em **Kubernetes**, **MongoDB** e **.NET Core**, com foco em arquitetura resiliente e escalável.
