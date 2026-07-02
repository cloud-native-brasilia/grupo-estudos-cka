# 🏋️ Banco de 50 Exercícios — Currículo Oficial da CKA

Este banco contém **50 exercícios práticos**, um para (quase) cada habilidade listada no **currículo oficial da CKA** — veja o PDF fonte em [`../curriculo-oficial/CKA_Curriculum_v1.35.pdf`](../curriculo-oficial/CKA_Curriculum_v1.35.pdf).

Os exercícios estão organizados na mesma **ordem didática do [ROADMAP.md](../ROADMAP.md)** (Workloads → Networking → Storage → Cluster Architecture → Troubleshooting), e não na ordem de peso da prova. A distribuição de quantidade por domínio é proporcional ao peso oficial:

| Bloco do Roadmap | Domínio oficial | Peso | Exercícios |
| :---: | :--- | :---: | :---: |
| 1️⃣ | Workloads and Scheduling | 15% | `01`–`07` (7) |
| 2️⃣ | Services and Networking | 20% | `08`–`17` (10) |
| 3️⃣ | Storage | 10% | `18`–`22` (5) |
| 4️⃣ | Cluster Architecture, Installation and Configuration | 25% | `23`–`35` (13) |
| 5️⃣ | Troubleshooting | 30% | `36`–`50` (15) |

> 🎯 **Como usar:** resolva no seu [cluster local (kind)](../ambiente-local/README.md), cronometrando-se. As soluções priorizam a **forma imperativa do `kubectl`** sempre que existir — é o jeito mais rápido de resolver na prova. Quando o recurso não tem forma imperativa (NetworkPolicy, PV, RBAC completo, CRDs, static pods), a solução usa um manifesto YAML enxuto.
>
> 💡 Configure antes: `alias k=kubectl` e `export do="--dry-run=client -o yaml"` (veja o [ROADMAP.md](../ROADMAP.md) para o setup completo de atalhos).

---

## 1️⃣ Workloads & Scheduling (peso oficial: 15%)

### Exercício 01 — Rolling update de um Deployment

**Habilidade do edital:** *Understand application deployments and how to perform rolling update and rollbacks.*

**Enunciado:** Crie um Deployment `web` com imagem `nginx:1.25` e 3 réplicas. Atualize a imagem para `nginx:1.26` usando rolling update e acompanhe o progresso.

**Solução:**

```bash
k create deployment web --image=nginx:1.25 --replicas=3

# Atualiza a imagem (dispara o rolling update)
k set image deployment/web nginx=nginx:1.26

# Acompanha o rollout em tempo real
k rollout status deployment/web

# Confirma a nova imagem e o histórico
k rollout history deployment/web
```

---

### Exercício 02 — Rollback de um Deployment

**Habilidade do edital:** *...perform rolling update and rollbacks.*

**Enunciado:** O Deployment `web` (do exercício anterior) foi atualizado para uma imagem quebrada (`nginx:broken-tag`). Reverta para a revisão estável anterior.

**Solução:**

```bash
# Simula a atualização quebrada
k set image deployment/web nginx=nginx:broken-tag

# Confirma o problema (Pods em ImagePullBackOff)
k get pods -l app=web

# Reverte para a revisão anterior
k rollout undo deployment/web

# Ou para uma revisão específica:
k rollout undo deployment/web --to-revision=1

k rollout status deployment/web
```

---

### Exercício 03 — ConfigMap como variáveis de ambiente

**Habilidade do edital:** *Use ConfigMaps and Secrets to configure applications.*

**Enunciado:** Crie um ConfigMap `app-config` com as chaves `LOG_LEVEL=debug` e `APP_ENV=staging`. Crie um Pod `app-cm` (imagem `busybox:1.36`) que injete essas chaves como variáveis de ambiente.

**Solução:**

```bash
k create configmap app-config --from-literal=LOG_LEVEL=debug --from-literal=APP_ENV=staging

k run app-cm --image=busybox:1.36 --restart=Never $do \
  --command -- sh -c "env; sleep 3600" > app-cm.yaml
```

Adicione o `envFrom` ao container no `app-cm.yaml`:

```yaml
spec:
  containers:
    - name: app-cm
      image: busybox:1.36
      command: ["sh", "-c", "env; sleep 3600"]
      envFrom:
        - configMapRef:
            name: app-config
```

```bash
k apply -f app-cm.yaml
k logs app-cm | grep -E "LOG_LEVEL|APP_ENV"
```

---

### Exercício 04 — Secret montado como volume

**Habilidade do edital:** *Use ConfigMaps and Secrets to configure applications.*

**Enunciado:** Crie um Secret genérico `db-creds` com `username=admin` e `password=s3nha`. Monte-o como volume em `/etc/db-creds` num Pod `app-secret`.

**Solução:**

```bash
k create secret generic db-creds --from-literal=username=admin --from-literal=password=s3nha

k run app-secret --image=busybox:1.36 --restart=Never $do \
  --command -- sh -c "sleep 3600" > app-secret.yaml
```

Edite `app-secret.yaml` adicionando o volume:

```yaml
spec:
  containers:
    - name: app-secret
      image: busybox:1.36
      command: ["sh", "-c", "sleep 3600"]
      volumeMounts:
        - name: creds
          mountPath: /etc/db-creds
          readOnly: true
  volumes:
    - name: creds
      secret:
        secretName: db-creds
```

```bash
k apply -f app-secret.yaml
k exec app-secret -- ls /etc/db-creds
k exec app-secret -- cat /etc/db-creds/username
```

---

### Exercício 05 — Workload autoscaling com HPA

**Habilidade do edital:** *Configure workload autoscaling.*

**Enunciado:** Crie um Deployment `api` (imagem `nginx:1.25`, 2 réplicas) com `requests.cpu=100m`. Configure um HorizontalPodAutoscaler para escalar entre 2 e 6 réplicas, alvo de 50% de uso de CPU.

**Solução:**

```bash
k create deployment api --image=nginx:1.25 --replicas=2

# Define o request de CPU (necessário para o HPA calcular %)
k set resources deployment/api --requests=cpu=100m

# Cria o HPA de forma imperativa
k autoscale deployment api --min=2 --max=6 --cpu-percent=50

k get hpa api
```

> ⚠️ Requer o `metrics-server` instalado no cluster para o HPA funcionar de verdade (veja o Exercício 44).

---

### Exercício 06 — DaemonSet (1 Pod por nó)

**Habilidade do edital:** *Understand the primitives used to create robust, self-healing, application deployments.*

**Enunciado:** Crie um DaemonSet `node-agent` (imagem `busybox:1.36`, comando `sleep 3600`) que garanta exatamente um Pod rodando em cada nó do cluster, incluindo o control-plane.

**Solução:** DaemonSet não tem `kubectl create daemonset`; gere a partir de um Deployment e ajuste o `kind`:

```bash
k create deployment node-agent --image=busybox:1.36 $do \
  -- sleep 3600 > node-agent.yaml
```

Edite `node-agent.yaml`: troque `kind: Deployment` por `kind: DaemonSet`, remova `spec.replicas` e `strategy`. Para rodar também no control-plane, adicione tolerância ao taint padrão:

```yaml
spec:
  template:
    spec:
      tolerations:
        - key: node-role.kubernetes.io/control-plane
          effect: NoSchedule
      containers:
        - name: node-agent
          image: busybox:1.36
          command: ["sleep", "3600"]
```

```bash
k apply -f node-agent.yaml

# Deve haver 1 Pod por nó (DESIRED = número de nós)
k get daemonset node-agent
k get pods -l app=node-agent -o wide
```

---

### Exercício 07 — Pod admission e scheduling (limits + NodeAffinity)

**Habilidade do edital:** *Configure Pod admission and scheduling (limits, node affinity, etc.).*

**Enunciado:** Rotule um worker com `env=prod`. Crie um Pod `app-prod` (imagem `nginx:1.25`) com `requests.cpu=100m`/`limits.cpu=200m` e `requests.memory=128Mi`/`limits.memory=256Mi`, que só possa ser agendado em nós com `env=prod` (NodeAffinity obrigatória).

**Solução:**

```bash
k label node cka-lab-worker env=prod

k run app-prod --image=nginx:1.25 $do > app-prod.yaml
```

Edite `app-prod.yaml`:

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: env
                operator: In
                values: ["prod"]
  containers:
    - name: app-prod
      image: nginx:1.25
      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "200m"
          memory: "256Mi"
```

```bash
k apply -f app-prod.yaml
k get pod app-prod -o wide   # NODE deve ser cka-lab-worker
```

---

## 2️⃣ Services & Networking (peso oficial: 20%)

### Exercício 08 — Conectividade entre Pods

**Habilidade do edital:** *Understand connectivity between Pods.*

**Enunciado:** Crie dois Pods (`pod-a` e `pod-b`, imagem `busybox:1.36`) e comprove que `pod-a` consegue alcançar o IP de `pod-b` diretamente (sem Service).

**Solução:**

```bash
k run pod-a --image=busybox:1.36 --restart=Never -- sleep 3600
k run pod-b --image=busybox:1.36 --restart=Never -- sleep 3600

# Descobre o IP do pod-b
POD_B_IP=$(k get pod pod-b -o jsonpath='{.status.podIP}')

# Testa a conectividade a partir do pod-a
k exec pod-a -- ping -c 2 "$POD_B_IP"
```

> ✅ Todo Pod deve conseguir alcançar qualquer outro Pod pelo IP, sem NAT — esse é o modelo de rede "flat" do Kubernetes.

---

### Exercício 09 — NetworkPolicy default deny-all (ingress)

**Habilidade do edital:** *Define and enforce Network Policies.*

**Enunciado:** No namespace `restrito`, crie uma NetworkPolicy que bloqueie **todo** tráfego de entrada para todos os Pods do namespace.

**Solução:** NetworkPolicy não tem forma imperativa — escreva o YAML:

```bash
k create namespace restrito
```

```yaml
# deny-all-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
  namespace: restrito
spec:
  podSelector: {}        # aplica a TODOS os Pods do namespace
  policyTypes:
    - Ingress
  # sem regra de 'ingress:' => nenhum tráfego de entrada é permitido
```

```bash
k apply -f deny-all-ingress.yaml
k get networkpolicy -n restrito
```

---

### Exercício 10 — NetworkPolicy permitindo apenas um label específico

**Habilidade do edital:** *Define and enforce Network Policies.*

**Enunciado:** No namespace `restrito`, permita tráfego de entrada na porta `80` para Pods com label `app=backend`, mas **apenas** vindo de Pods com label `app=frontend`.

**Solução:**

```yaml
# allow-frontend-to-backend.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: restrito
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 80
```

```bash
k apply -f allow-frontend-to-backend.yaml

# Valide criando os dois Pods e testando o acesso
k run backend --image=nginx:1.25 -n restrito --labels=app=backend
k run frontend --image=busybox:1.36 -n restrito --labels=app=frontend --restart=Never -- sleep 3600
k run outro --image=busybox:1.36 -n restrito --labels=app=outro --restart=Never -- sleep 3600
```

---

### Exercício 11 — Service ClusterIP

**Habilidade do edital:** *Use ClusterIP, NodePort, LoadBalancer service types and endpoints.*

**Enunciado:** Crie um Deployment `web` (imagem `nginx:1.25`, 2 réplicas) e exponha-o internamente com um Service ClusterIP na porta `80`.

**Solução:**

```bash
k create deployment web --image=nginx:1.25 --replicas=2
k expose deployment web --port=80 --target-port=80 --name=web

k get svc web
k get endpoints web   # deve listar os IPs dos 2 Pods
```

---

### Exercício 12 — Service NodePort com porta fixa

**Habilidade do edital:** *Use ClusterIP, NodePort, LoadBalancer service types and endpoints.*

**Enunciado:** Exponha o Deployment `web` externamente via NodePort, fixando a porta de nó em `30080`.

**Solução:**

```bash
k expose deployment web --port=80 --target-port=80 --type=NodePort \
  --name=web-nodeport $do > web-nodeport.yaml
```

Ajuste `spec.ports[0].nodePort: 30080` no YAML gerado, depois:

```bash
k apply -f web-nodeport.yaml
k get svc web-nodeport
curl http://localhost:30080   # funciona graças ao extraPortMappings do kind
```

---

### Exercício 13 — Service headless

**Habilidade do edital:** *Use ClusterIP, NodePort, LoadBalancer service types and endpoints.*

**Enunciado:** Crie um Service headless (`ClusterIP: None`) chamado `web-headless` para o Deployment `web`, e comprove que o DNS retorna os IPs individuais dos Pods (não um IP virtual único).

**Solução:** Headless também exige YAML (o `--cluster-ip=None` do `expose` funciona via flag):

```bash
k expose deployment web --port=80 --target-port=80 \
  --cluster-ip=None --name=web-headless

k get svc web-headless   # CLUSTER-IP deve mostrar 'None'

# Resolve o nome a partir de um Pod: deve retornar múltiplos IPs (um por Pod)
k run dns-test --image=busybox:1.36 --restart=Never --rm -it -- \
  nslookup web-headless
```

---

### Exercício 14 — Ingress roteando por path

**Habilidade do edital:** *Know how to use Ingress controllers and Ingress resources.*

**Enunciado:** Com dois Services (`web` na porta 80 e `api` na porta 80), crie um Ingress `app-ingress` que roteie `/web` para o Service `web` e `/api` para o Service `api`, no host `app.local`.

**Solução:**

```bash
k create deployment api --image=nginx:1.25
k expose deployment api --port=80

k create ingress app-ingress --rule="app.local/web*=web:80" \
  --rule="app.local/api*=api:80"

k get ingress app-ingress
k describe ingress app-ingress
```

> ⚠️ É necessário um Ingress Controller instalado (ex.: ingress-nginx) para o roteamento funcionar de fato — veja o Exercício 34.

---

### Exercício 15 — Verificar CoreDNS (service discovery)

**Habilidade do edital:** *Understand and use CoreDNS.*

**Enunciado:** Comprove que o Service `web` (ClusterIP) é resolvido pelo nome `web.default.svc.cluster.local` a partir de qualquer Pod.

**Solução:**

```bash
# Confirma que os Pods do CoreDNS estão saudáveis
k get pods -n kube-system -l k8s-app=kube-dns

k run dns-check --image=busybox:1.36 --restart=Never --rm -it -- \
  nslookup web.default.svc.cluster.local
```

---

### Exercício 16 — Customizar o CoreDNS

**Habilidade do edital:** *Understand and use CoreDNS.*

**Enunciado:** Edite o ConfigMap do CoreDNS para encaminhar consultas do domínio `empresa.local` para o resolver `10.20.0.53`.

**Solução:**

```bash
k edit configmap coredns -n kube-system
```

Adicione um bloco de `stub domain` no `Corefile`:

```
    empresa.local:53 {
        errors
        cache 30
        forward . 10.20.0.53
    }
```

```bash
# Reinicia os Pods do CoreDNS para recarregar a config
k rollout restart deployment coredns -n kube-system
k get pods -n kube-system -l k8s-app=kube-dns
```

---

### Exercício 17 — Gateway API (Gateway + HTTPRoute)

**Habilidade do edital:** *Use the Gateway API to manage Ingress traffic.*

**Enunciado:** Usando a Gateway API, crie um `Gateway` chamado `app-gateway` escutando HTTP na porta 80, e um `HTTPRoute` que direcione todo o tráfego para o Service `web`.

**Solução:** Gateway API é declarativa (CRDs) — não tem forma imperativa:

```yaml
# app-gateway.yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: app-gateway
spec:
  gatewayClassName: nginx   # depende do controller instalado
  listeners:
    - name: http
      protocol: HTTP
      port: 80
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: web-route
spec:
  parentRefs:
    - name: app-gateway
  rules:
    - backendRefs:
        - name: web
          port: 80
```

```bash
k apply -f app-gateway.yaml
k get gateway app-gateway
k get httproute web-route
```

> ⚠️ Requer um controller compatível com a Gateway API instalado no cluster (ex.: `gatewayClassName` correspondente).

---

## 3️⃣ Storage (peso oficial: 10%)

### Exercício 18 — StorageClass com provisionamento dinâmico

**Habilidade do edital:** *Implement storage classes and dynamic volume provisioning.*

**Enunciado:** Crie uma StorageClass `fast-local` usando o provisioner padrão do kind (`rancher.io/local-path`), com `volumeBindingMode: WaitForFirstConsumer`.

**Solução:** StorageClass não tem forma imperativa:

```yaml
# fast-local-sc.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-local
provisioner: rancher.io/local-path
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete
```

```bash
k apply -f fast-local-sc.yaml
k get storageclass
```

---

### Exercício 19 — PVC com provisionamento dinâmico

**Habilidade do edital:** *Manage persistent volumes and persistent volume claims.*

**Enunciado:** Crie um PVC `data-pvc` de `1Gi` usando a StorageClass `fast-local`, e monte-o em `/data` num Pod `writer` (imagem `busybox:1.36`).

**Solução:**

```yaml
# data-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  accessModes: ["ReadWriteOnce"]
  storageClassName: fast-local
  resources:
    requests:
      storage: 1Gi
```

```bash
k apply -f data-pvc.yaml

k run writer --image=busybox:1.36 --restart=Never $do \
  --command -- sh -c "echo ok > /data/teste.txt; sleep 3600" > writer.yaml
```

Adicione o volume ao `writer.yaml`:

```yaml
spec:
  containers:
    - name: writer
      image: busybox:1.36
      command: ["sh", "-c", "echo ok > /data/teste.txt; sleep 3600"]
      volumeMounts:
        - name: data
          mountPath: /data
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: data-pvc
```

```bash
k apply -f writer.yaml
# Com WaitForFirstConsumer, o PVC só faz o bind quando o Pod é criado:
k get pvc data-pvc
k exec writer -- cat /data/teste.txt
```

---

### Exercício 20 — PV estático (hostPath) com reclaimPolicy Retain

**Habilidade do edital:** *Configure volume types, access modes and reclaim policies.*

**Enunciado:** Crie um PV estático `static-pv` de `500Mi` usando `hostPath` (`/mnt/static-data`), modo de acesso `ReadWriteOnce` e `persistentVolumeReclaimPolicy: Retain`. Crie um PVC `static-pvc` que se vincule a ele.

**Solução:**

```yaml
# static-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: static-pv
spec:
  capacity:
    storage: 500Mi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/static-data
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: static-pvc
spec:
  accessModes: ["ReadWriteOnce"]
  storageClassName: manual
  resources:
    requests:
      storage: 500Mi
```

```bash
k apply -f static-pv.yaml
k get pv static-pv
k get pvc static-pvc   # STATUS deve ser 'Bound'
```

---

### Exercício 21 — Alterar o reclaimPolicy de um PV existente

**Habilidade do edital:** *Configure volume types, access modes and reclaim policies.*

**Enunciado:** O PV `static-pv` foi criado com `reclaimPolicy: Delete` por engano. Altere-o para `Retain` sem recriar o objeto.

**Solução:**

```bash
k patch pv static-pv -p '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}'

k get pv static-pv -o jsonpath='{.spec.persistentVolumeReclaimPolicy}'
```

---

### Exercício 22 — emptyDir compartilhado entre containers

**Habilidade do edital:** *Configure volume types, access modes and reclaim policies.*

**Enunciado:** Crie um Pod `sidecar-demo` com dois containers: `writer` (grava um arquivo a cada 5s em `/var/log/app`) e `reader` (imagem `busybox:1.36`, lê o arquivo). Ambos devem compartilhar um volume `emptyDir`.

**Solução:**

```bash
k run sidecar-demo --image=busybox:1.36 $do \
  --command -- sh -c "while true; do date >> /var/log/app/log.txt; sleep 5; done" > sidecar-demo.yaml
```

Edite `sidecar-demo.yaml` para adicionar o segundo container e o volume:

```yaml
spec:
  containers:
    - name: writer
      image: busybox:1.36
      command: ["sh", "-c", "while true; do date >> /var/log/app/log.txt; sleep 5; done"]
      volumeMounts:
        - name: logs
          mountPath: /var/log/app
    - name: reader
      image: busybox:1.36
      command: ["sh", "-c", "tail -f /var/log/app/log.txt"]
      volumeMounts:
        - name: logs
          mountPath: /var/log/app
  volumes:
    - name: logs
      emptyDir: {}
```

```bash
k apply -f sidecar-demo.yaml
k logs sidecar-demo -c reader --follow
```

---

## 4️⃣ Cluster Architecture, Installation & Configuration (peso oficial: 25%)

### Exercício 23 — RBAC: Role + RoleBinding

**Habilidade do edital:** *Manage role based access control (RBAC).*

**Enunciado:** No namespace `default`, crie uma Role `pod-reader` que permita apenas `get`, `list` e `watch` em Pods. Vincule-a à ServiceAccount `viewer-sa`.

**Solução:**

```bash
k create serviceaccount viewer-sa

k create role pod-reader --verb=get,list,watch --resource=pods

k create rolebinding viewer-binding --role=pod-reader \
  --serviceaccount=default:viewer-sa
```

---

### Exercício 24 — RBAC: ClusterRole + ClusterRoleBinding

**Habilidade do edital:** *Manage role based access control (RBAC).*

**Enunciado:** Crie uma ClusterRole `node-reader` que permita `get` e `list` em `nodes` (recurso cluster-wide). Vincule-a à ServiceAccount `monitor-sa` do namespace `monitoring`.

**Solução:**

```bash
k create namespace monitoring
k create serviceaccount monitor-sa -n monitoring

k create clusterrole node-reader --verb=get,list --resource=nodes

k create clusterrolebinding monitor-binding \
  --clusterrole=node-reader \
  --serviceaccount=monitoring:monitor-sa
```

---

### Exercício 25 — Verificar permissões com `auth can-i`

**Habilidade do edital:** *Manage role based access control (RBAC).*

**Enunciado:** Confirme que a ServiceAccount `viewer-sa` (Exercício 23) **pode** listar Pods, mas **não pode** deletá-los.

**Solução:**

```bash
k auth can-i list pods --as=system:serviceaccount:default:viewer-sa
# yes

k auth can-i delete pods --as=system:serviceaccount:default:viewer-sa
# no
```

---

### Exercício 26 — Preparar infraestrutura para o cluster

**Habilidade do edital:** *Prepare underlying infrastructure for installing a Kubernetes cluster.*

**Enunciado:** Liste os pré-requisitos de um host Linux antes de rodar `kubeadm init`, e os comandos para satisfazê-los.

**Solução:**

```bash
# 1. Desabilitar swap (obrigatório para o kubelet)
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab

# 2. Carregar módulos de kernel exigidos pelo CNI/containerd
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter

# 3. Parâmetros de rede exigidos
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
sudo sysctl --system

# 4. Instalar um runtime de container (ex.: containerd) e o próprio kubeadm/kubelet/kubectl
```

---

### Exercício 27 — `kubeadm init`

**Habilidade do edital:** *Create and manage Kubernetes clusters using kubeadm.*

**Enunciado:** Inicialize um control plane com `kubeadm`, usando `10.244.0.0/16` como CIDR de Pods (compatível com Flannel), e configure o `kubectl` para o usuário atual.

**Solução:**

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

mkdir -p "$HOME"/.kube
sudo cp -i /etc/kubernetes/admin.conf "$HOME"/.kube/config
sudo chown "$(id -u)":"$(id -g)" "$HOME"/.kube/config

# Instala um plugin CNI (necessário para os nós ficarem Ready)
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml

kubectl get nodes
```

---

### Exercício 28 — `kubeadm join` (adicionar worker)

**Habilidade do edital:** *Create and manage Kubernetes clusters using kubeadm.*

**Enunciado:** Gere um novo token de join (o original pode ter expirado) e use-o para adicionar um worker ao cluster.

**Solução:**

```bash
# No control plane: gera um novo token + comando de join completo
kubeadm token create --print-join-command
```

```bash
# No worker (com o output do comando acima):
sudo kubeadm join <control-plane-host>:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>
```

```bash
# De volta ao control plane:
kubectl get nodes
```

---

### Exercício 29 — Upgrade do control plane com `kubeadm`

**Habilidade do edital:** *Manage the lifecycle of Kubernetes clusters.*

**Enunciado:** Verifique a próxima versão disponível para upgrade do cluster e aplique o upgrade do control plane.

**Solução:**

```bash
# 1. Atualiza o pacote do kubeadm para a nova versão (exemplo genérico)
sudo apt-mark unhold kubeadm
sudo apt-get install -y kubeadm=1.32.0-1.1
sudo apt-mark hold kubeadm

# 2. Verifica o plano de upgrade
sudo kubeadm upgrade plan

# 3. Aplica o upgrade
sudo kubeadm upgrade apply v1.32.0

# 4. Atualiza kubelet e kubectl da mesma forma, depois reinicia o kubelet
sudo systemctl restart kubelet
```

---

### Exercício 30 — Upgrade seguro de um worker node

**Habilidade do edital:** *Manage the lifecycle of Kubernetes clusters.*

**Enunciado:** Faça o upgrade do `kubelet` de um worker sem impactar as aplicações: drene o nó antes e libere-o depois.

**Solução:**

```bash
# No control plane: evacua os Pods do nó com segurança
kubectl drain cka-lab-worker --ignore-daemonsets --delete-emptydir-data

# No worker: atualiza kubeadm (config do nó) e kubelet
sudo apt-mark unhold kubeadm kubelet
sudo apt-get install -y kubeadm=1.32.0-1.1 kubelet=1.32.0-1.1
sudo kubeadm upgrade node
sudo apt-mark hold kubeadm kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# De volta ao control plane: libera o nó
kubectl uncordon cka-lab-worker
kubectl get nodes
```

---

### Exercício 31 — Backup do ETCD

**Habilidade do edital:** *Manage the lifecycle of Kubernetes clusters.*

**Enunciado:** Gere um snapshot do ETCD do control plane em `/opt/backups/etcd-snapshot.db` e valide sua integridade.

**Solução:**

```bash
ETCDCTL_API=3 etcdctl snapshot save /opt/backups/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key

ETCDCTL_API=3 etcdctl snapshot status /opt/backups/etcd-snapshot.db --write-out=table
```

---

### Exercício 32 — Restore do ETCD

**Habilidade do edital:** *Manage the lifecycle of Kubernetes clusters.*

**Enunciado:** Restaure o cluster a partir do snapshot `/opt/backups/etcd-snapshot.db` gerado no exercício anterior.

**Solução:**

```bash
ETCDCTL_API=3 etcdctl snapshot restore /opt/backups/etcd-snapshot.db \
  --data-dir=/var/lib/etcd-restore

sudo vim /etc/kubernetes/manifests/etcd.yaml
# Altere '--data-dir' e o hostPath do volume 'etcd-data' para '/var/lib/etcd-restore'

# O kubelet recria o static pod do etcd automaticamente
kubectl get pods -n kube-system | grep etcd
kubectl get nodes
```

---

### Exercício 33 — Control plane de alta disponibilidade (HA)

**Habilidade do edital:** *Implement and configure a highly-available control plane.*

**Enunciado:** Adicione um segundo nó de control plane a um cluster já existente, usando `kubeadm join --control-plane`.

**Solução:**

```bash
# No control plane original: gera o comando de join com o certificate-key
sudo kubeadm init phase upload-certs --upload-certs
kubeadm token create --print-join-command
```

```bash
# No novo nó de control plane, combine o join-command com --control-plane e o certificate-key:
sudo kubeadm join <endpoint-do-lb>:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash> \
  --control-plane \
  --certificate-key <certificate-key>
```

```bash
kubectl get nodes   # os dois nós devem aparecer com ROLES=control-plane
```

> 💡 Em produção, o `<endpoint-do-lb>` é um load balancer na frente dos control planes — não o IP de um nó individual.

---

### Exercício 34 — Instalar componente via Helm

**Habilidade do edital:** *Use Helm and Kustomize to install cluster components.*

**Enunciado:** Instale o `ingress-nginx` no cluster usando Helm, no namespace `ingress-nginx`.

**Solução:**

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace

helm list -n ingress-nginx
kubectl get pods -n ingress-nginx
```

---

### Exercício 35 — CRD e recurso customizado

**Habilidade do edital:** *Understand CRDs, install and configure operators.*

**Enunciado:** Crie uma CustomResourceDefinition `websites.cka.exemplo.io` (kind `Website`, com o campo `spec.url`) e um recurso customizado usando essa definição.

**Solução:** CRDs são sempre declarativos:

```yaml
# website-crd.yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: websites.cka.exemplo.io
spec:
  group: cka.exemplo.io
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                url:
                  type: string
  scope: Namespaced
  names:
    plural: websites
    singular: website
    kind: Website
```

```yaml
# my-website.yaml
apiVersion: cka.exemplo.io/v1
kind: Website
metadata:
  name: meu-site
spec:
  url: "https://exemplo.io"
```

```bash
k apply -f website-crd.yaml
k apply -f my-website.yaml
k get websites
k get website meu-site -o yaml
```

---

## 5️⃣ Troubleshooting (peso oficial: 30%)

### Exercício 36 — Pod em Pending

**Habilidade do edital:** *Troubleshoot clusters and nodes.*

**Enunciado:** Um Pod `big-app` está preso em `Pending`. Diagnostique a causa mais provável e corrija.

**Solução:**

```bash
k describe pod big-app | tail -n 15
# Evento comum: "Insufficient cpu" ou "Insufficient memory"

# Verifica os recursos alocáveis dos nós
k describe nodes | grep -A 5 "Allocated resources"

# Correção típica: reduzir o request do Pod
k set resources deployment/big-app --requests=cpu=100m,memory=128Mi
```

---

### Exercício 37 — CrashLoopBackOff

**Habilidade do edital:** *Manage and evaluate container output streams.*

**Enunciado:** O Pod `flaky-app` está em `CrashLoopBackOff`. Investigue a causa raiz usando os logs, inclusive da execução anterior.

**Solução:**

```bash
k get pod flaky-app
k describe pod flaky-app | tail -n 15

# Logs da tentativa atual
k logs flaky-app

# Logs da tentativa ANTERIOR (essencial quando o container já reiniciou)
k logs flaky-app --previous
```

---

### Exercício 38 — ImagePullBackOff

**Habilidade do edital:** *Troubleshoot clusters and nodes.*

**Enunciado:** O Pod `typo-app` está em `ImagePullBackOff` porque foi criado com a tag `nginx:1.999` (inexistente). Corrija.

**Solução:**

```bash
k describe pod typo-app | grep -A 3 Events
# Evento: "Failed to pull image ... nginx:1.999: not found"

k set image pod/typo-app typo-app=nginx:1.25
# Para Pods "nus" (sem controller), pode ser necessário deletar e recriar:
k delete pod typo-app
k run typo-app --image=nginx:1.25
```

---

### Exercício 39 — Container OOMKilled

**Habilidade do edital:** *Troubleshoot clusters and nodes.*

**Enunciado:** O container do Pod `mem-app` está sendo reiniciado com `Reason: OOMKilled`. Diagnostique e aumente o limite de memória.

**Solução:**

```bash
k describe pod mem-app | grep -A 3 "Last State"
# Reason: OOMKilled

k get pod mem-app -o jsonpath='{.spec.containers[0].resources}'

# Corrige aumentando o limite de memória
k set resources deployment/mem-app --limits=memory=512Mi
```

---

### Exercício 40 — Nó em NotReady

**Habilidade do edital:** *Troubleshoot clusters and nodes.*

**Enunciado:** O nó `cka-lab-worker2` aparece como `NotReady`. Diagnostique se o `kubelet` está rodando e reinicie-o se necessário.

**Solução:**

```bash
k describe node cka-lab-worker2 | grep -A 5 Conditions

# Dentro do nó (SSH ou 'docker exec' no caso do kind):
sudo systemctl status kubelet
sudo journalctl -u kubelet -n 50 --no-pager

# Se estiver parado:
sudo systemctl restart kubelet
sudo systemctl enable kubelet

# De volta ao control plane:
k get nodes
```

---

### Exercício 41 — Componente do cluster quebrado (static pod)

**Habilidade do edital:** *Troubleshoot cluster components.*

**Enunciado:** Após uma edição manual, o `kube-scheduler` não sobe mais. Diagnostique e corrija o manifest do static pod.

**Solução:**

```bash
k get pods -n kube-system | grep scheduler
# Pode nem aparecer, se o kubelet não conseguir nem criar o Pod

# Verifica erros de sintaxe no manifest
sudo cat /etc/kubernetes/manifests/kube-scheduler.yaml

# Logs do kubelet costumam apontar o erro de parsing do YAML
sudo journalctl -u kubelet -n 50 --no-pager | grep -i scheduler

# Corrija o YAML (indentação, chave errada, etc.) e salve —
# o kubelet detecta a mudança e recria o static pod automaticamente
k get pods -n kube-system -w | grep scheduler
```

---

### Exercício 42 — kube-scheduler não agenda Pods

**Habilidade do edital:** *Troubleshoot cluster components.*

**Enunciado:** Pods novos ficam eternamente em `Pending` mesmo havendo recursos disponíveis. Investigue os logs do `kube-scheduler`.

**Solução:**

```bash
k get pods -n kube-system -l component=kube-scheduler

k logs -n kube-system kube-scheduler-cka-lab-control-plane --tail=50

# Verifica também eventos do próprio Pod pendente
k describe pod <pod-pendente> | tail -n 10
```

---

### Exercício 43 — kube-controller-manager com problema

**Habilidade do edital:** *Troubleshoot cluster components.*

**Enunciado:** Deployments não criam novos ReplicaSets/Pods após um `scale`. Suspeita-se do `kube-controller-manager`. Investigue.

**Solução:**

```bash
k get pods -n kube-system -l component=kube-controller-manager

k logs -n kube-system kube-controller-manager-cka-lab-control-plane --tail=50

# Confirma se o static pod está saudável
k describe pod -n kube-system kube-controller-manager-cka-lab-control-plane
```

---

### Exercício 44 — Monitorar uso de recursos (metrics-server ausente)

**Habilidade do edital:** *Monitor cluster and application resource usage.*

**Enunciado:** O comando `kubectl top nodes` falha com erro de métricas indisponíveis. Diagnostique e instale o `metrics-server`.

**Solução:**

```bash
k top nodes
# Error: Metrics API not available

k get deployment metrics-server -n kube-system
# Se não existir, instale:

k apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Em clusters kind/self-signed, normalmente é necessário permitir certificados inseguros:
k patch deployment metrics-server -n kube-system --type='json' \
  -p='[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'

k top nodes
k top pods -A
```

---

### Exercício 45 — Logs de container específico em Pod multi-container

**Habilidade do edital:** *Manage and evaluate container output streams.*

**Enunciado:** O Pod `sidecar-demo` (Exercício 22) tem dois containers. Obtenha apenas os logs do container `writer`, e depois os logs de **todos** os containers do Pod ao mesmo tempo.

**Solução:**

```bash
# Container específico
k logs sidecar-demo -c writer

# Todos os containers do Pod, com prefixo indicando a origem
k logs sidecar-demo --all-containers=true --prefix=true
```

---

### Exercício 46 — Service não roteia tráfego (mismatch de selector)

**Habilidade do edital:** *Troubleshoot services and networking.*

**Enunciado:** O Service `web` não tem `endpoints`, apesar de existirem Pods do Deployment `web` rodando. Diagnostique e corrija.

**Solução:**

```bash
k get endpoints web
# <none>

# Compara os selectors
k get svc web -o jsonpath='{.spec.selector}{"\n"}'
k get pods --show-labels | grep web

# Causa comum: selector do Service não bate com os labels dos Pods.
# Corrige o selector do Service:
k patch svc web -p '{"spec":{"selector":{"app":"web"}}}'

k get endpoints web
```

---

### Exercício 47 — Falha de resolução DNS dentro de um Pod

**Habilidade do edital:** *Troubleshoot services and networking.*

**Enunciado:** Um Pod não consegue resolver `web.default.svc.cluster.local`. Verifique se o problema está no CoreDNS.

**Solução:**

```bash
# Confirma se os Pods do CoreDNS estão saudáveis
k get pods -n kube-system -l k8s-app=kube-dns
k logs -n kube-system -l k8s-app=kube-dns --tail=50

# Confirma se o Service do CoreDNS existe e tem endpoints
k get svc -n kube-system kube-dns
k get endpoints -n kube-system kube-dns

# Confirma o resolv.conf dentro do Pod com problema
k exec <pod-com-problema> -- cat /etc/resolv.conf
```

---

### Exercício 48 — NetworkPolicy bloqueando tráfego legítimo

**Habilidade do edital:** *Troubleshoot services and networking.*

**Enunciado:** Após aplicar uma NetworkPolicy, o Pod `frontend` não consegue mais acessar `backend` na porta `80`, embora devesse ser permitido. Encontre e corrija o erro na regra.

**Solução:**

```bash
k get networkpolicy -n restrito -o yaml

# Erro comum: o 'podSelector' do 'from' aponta para o label errado,
# ou falta a porta na regra de ingress.

# Corrige (exemplo: o label estava como 'role: frontend' em vez de 'app: frontend')
k edit networkpolicy allow-frontend-to-backend -n restrito

# Revalida o acesso
k exec frontend -n restrito -- wget -qO- --timeout=2 http://backend:80
```

---

### Exercício 49 — Nó não consegue fazer join (erro de certificado)

**Habilidade do edital:** *Troubleshoot clusters and nodes.*

**Enunciado:** Um novo worker falha ao executar `kubeadm join` com erro relacionado a certificado/token expirado. Diagnostique pelos logs do kubelet e resolva.

**Solução:**

```bash
# No worker que falhou:
sudo journalctl -u kubelet -n 50 --no-pager
# Procure por: "certificate has expired" ou "token id ... not found"

# No control plane: tokens de bootstrap expiram em 24h por padrão.
# Gere um novo token + hash e repita o join:
kubeadm token create --print-join-command
```

```bash
# No worker:
sudo kubeadm reset -f
sudo kubeadm join <control-plane-host>:6443 \
  --token <novo-token> \
  --discovery-token-ca-cert-hash sha256:<hash>
```

---

### Exercício 50 — Runbook geral de troubleshooting

**Habilidade do edital:** *Troubleshoot clusters and nodes / cluster components / services and networking (visão integrada).*

**Enunciado:** Uma aplicação `checkout` parou de responder externamente. Você não sabe se o problema é no Pod, no Service, no Ingress ou no nó. Descreva a sequência de diagnóstico ponta a ponta.

**Solução:** Siga o funil "de dentro para fora":

```bash
# 1. O cluster está saudável? Nós Ready? Componentes do control plane up?
k get nodes
k get pods -n kube-system

# 2. Os Pods da aplicação estão rodando?
k get pods -l app=checkout -o wide
k describe pod <pod-com-problema>
k logs <pod-com-problema> --previous

# 3. O Service tem endpoints válidos?
k get svc checkout
k get endpoints checkout

# 4. Teste direto no Pod (bypassa Service/Ingress) para isolar a camada com problema
k exec -it <algum-pod> -- wget -qO- --timeout=2 http://<pod-ip>:8080

# 5. Teste via Service (ClusterIP) de dentro do cluster
k run debug --image=busybox:1.36 --restart=Never --rm -it -- \
  wget -qO- --timeout=2 http://checkout

# 6. Se passou pelo Service mas falha externamente, o problema é no Ingress/NodePort
k get ingress
k describe ingress checkout
k get pods -n ingress-nginx

# 7. Eventos recentes do namespace, para achar o que mudou por último
k get events --sort-by=.lastTimestamp -n default | tail -n 20
```

> 🎯 **Ponto-chave:** na prova, sempre isole a camada (Pod → Service → Ingress/rede) testando de dentro para fora. Isso evita perder tempo investigando a camada errada.

---

## 🏁 Próximos Passos

- Refaça os exercícios sem consultar a solução, cronometrando-se.
- Combine exercícios de blocos diferentes (ex.: Exercício 07 + Exercício 46) para simular cenários mais realistas.
- Submeta suas próprias variações via PR, seguindo o fluxo do [README principal](../README.md).

**Bons estudos! ☸️**
