# 🐳 Ambiente Local com kind (Kubernetes in Docker)

Este guia ensina a montar um **cluster Kubernetes multi-nó** na sua máquina usando o **kind** (Kubernetes IN Docker). É o ambiente ideal para praticar os labs da CKA localmente, de graça e sem depender de nuvem.

Com o cluster multi-nó (1 control-plane + 2 workers) você poderá praticar conceitos essenciais da prova que **não funcionam bem em cluster de nó único**, como:

- 🧩 **Scheduling**: NodeAffinity, Taints & Tolerations, `nodeSelector`.
- 🔧 **Manutenção de nós**: `kubectl drain`, `cordon`, `uncordon`.
- 🌐 **Networking** entre Pods em nós diferentes.

---

## 📋 Pré-requisitos

| Ferramenta | Para que serve | Verificação |
| :--- | :--- | :--- |
| **Docker** | Runtime que roda os nós como containers | `docker --version` |
| **kind** | Cria o cluster Kubernetes dentro do Docker | `kind --version` |
| **kubectl** | CLI para interagir com o cluster | `kubectl version --client` |

> ⚠️ O Docker precisa estar **rodando** antes de criar o cluster. No macOS/Windows, abra o Docker Desktop; no Linux, garanta que o serviço `docker` esteja ativo (`sudo systemctl start docker`).

---

## 🔧 Instalação das Ferramentas

### 1. Docker

- **macOS / Windows**: instale o [Docker Desktop](https://www.docker.com/products/docker-desktop/).
- **Linux (Ubuntu/Debian)**:

  ```bash
  curl -fsSL https://get.docker.com | sh
  sudo usermod -aG docker $USER   # permite rodar docker sem sudo (relogar depois)
  ```

### 2. kind

**Linux (amd64):**

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

**macOS (Homebrew):**

```bash
brew install kind
```

**macOS (binário — Apple Silicon):**

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-darwin-arm64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

### 3. kubectl

**Linux:**

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/kubectl
```

**macOS (Homebrew):**

```bash
brew install kubectl
```

---

## 🚀 Subindo o Cluster Multi-Nó

O arquivo [`kind-cluster-cka.yaml`](kind-cluster-cka.yaml) descreve um cluster com **1 control-plane** e **2 workers**.

```bash
# A partir do diretório ambiente-local/
kind create cluster --name cka-lab --config kind-cluster-cka.yaml
```

A criação leva de 1 a 3 minutos. Ao final você verá algo como:

```
Creating cluster "cka-lab" ...
 ✓ Ensuring node image (kindest/node) 🖼
 ✓ Preparing nodes 📦 📦 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-cka-lab"
```

---

## ✅ Verificando o Cluster

```bash
# Deve listar 3 nós: 1 control-plane e 2 workers
kubectl get nodes -o wide
```

Saída esperada:

```
NAME                     STATUS   ROLES           AGE   VERSION
cka-lab-control-plane    Ready    control-plane   90s   v1.31.x
cka-lab-worker           Ready    <none>          70s   v1.31.x
cka-lab-worker2          Ready    <none>          70s   v1.31.x
```

```bash
# Ver os componentes do control plane rodando
kubectl get pods -n kube-system
```

---

## 🧪 Praticando Manutenção de Nós (drain / cordon)

Com o cluster multi-nó, você pode simular manutenção — cenário clássico de prova:

```bash
# Marcar o nó como não-agendável (não recebe novos Pods)
kubectl cordon cka-lab-worker

# Drenar o nó: evacua os Pods e o marca como não-agendável
kubectl drain cka-lab-worker --ignore-daemonsets --delete-emptydir-data

# Após a "manutenção", liberar o nó novamente
kubectl uncordon cka-lab-worker
```

---

## 🎯 Praticando Scheduling (labels e affinity)

```bash
# Adicionar um label a um nó (usado por nodeSelector / NodeAffinity)
kubectl label node cka-lab-worker disktype=ssd

# Ver os labels de todos os nós
kubectl get nodes --show-labels

# Aplicar um taint (para praticar tolerations)
kubectl taint node cka-lab-worker2 dedicated=cka:NoSchedule
```

---

## 🧹 Comandos Úteis de Ciclo de Vida

```bash
# Listar todos os clusters kind
kind get clusters

# Ver os nós (containers) no Docker
docker ps

# Trocar o contexto do kubectl para este cluster
kubectl config use-context kind-cka-lab

# Deletar o cluster (limpa tudo)
kind delete cluster --name cka-lab
```

> 💡 **Dica:** você pode destruir e recriar o cluster à vontade. Como tudo é código (o YAML de config), recriar o ambiente do zero leva menos de 3 minutos. Isso é ótimo para praticar cenários "quebrados" sem medo.

---

## 🐛 Troubleshooting da Instalação

| Problema | Causa provável | Solução |
| :--- | :--- | :--- |
| `Cannot connect to the Docker daemon` | Docker não está rodando | Inicie o Docker Desktop / `systemctl start docker` |
| `permission denied` no `docker` | Usuário fora do grupo `docker` | `sudo usermod -aG docker $USER` e relogar |
| Nós ficam em `NotReady` | Recursos insuficientes | Aumente CPU/RAM do Docker Desktop |
| Porta já em uso | Outro cluster ativo | `kind delete cluster --name <nome>` |

---

Ambiente pronto? Bora para os [simulados](../simulados/EXERCICIOS-EXEMPLO.md)! ☸️
