# 🧪 Exercícios de Exemplo — Estilo CKA

Este arquivo contém **questões práticas** no formato da prova CKA, com **soluções passo a passo**. Sempre que possível, as soluções usam a **forma imperativa do `kubectl`** — na prova, cada segundo conta, e comandos imperativos são muito mais rápidos que escrever YAML do zero.

> 🎯 **Como usar:** tente resolver cada questão **sozinho(a)** no seu [cluster local](../ambiente-local/README.md) antes de olhar a solução. Cronometre-se!

> 💡 **Antes de começar**, configure os atalhos:
> ```bash
> alias k=kubectl
> export do="--dry-run=client -o yaml"
> source <(kubectl completion bash)
> complete -o default -F __start_kubectl k
> ```

---

## 📝 Questão 1 — Pod com NodeAffinity

**Peso do domínio:** Workloads & Scheduling (15%)

### Enunciado

Crie um Pod chamado `nginx-ssd` usando a imagem `nginx:1.25` que **só possa ser agendado em nós que tenham o label `disktype=ssd`**. Use **NodeAffinity** (requisito obrigatório — *required*), não `nodeSelector`.

### Solução Passo a Passo

**1. Rotule um nó de trabalho** (pré-requisito do cenário):

```bash
k label node cka-lab-worker disktype=ssd
```

**2. Gere o esqueleto do Pod imperativamente** e redirecione para um arquivo:

```bash
k run nginx-ssd --image=nginx:1.25 $do > nginx-ssd.yaml
```

**3. Edite o YAML** adicionando o bloco de `affinity`. NodeAffinity **não** tem forma imperativa, então editamos o manifest:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-ssd
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: disktype
                operator: In
                values:
                  - ssd
  containers:
    - name: nginx-ssd
      image: nginx:1.25
```

**4. Aplique e valide:**

```bash
k apply -f nginx-ssd.yaml

# Confirmar que o Pod subiu no nó com o label correto
k get pod nginx-ssd -o wide
```

**5. (Verificação extra) Confirme o agendamento correto:**

```bash
# O NODE da saída deve ser 'cka-lab-worker' (o que tem disktype=ssd)
k get pod nginx-ssd -o wide

# Se quiser testar a regra, descreva o Pod: em caso de nenhum nó elegível,
# o evento mostrará 'didn't match Pod's node affinity/selector'
k describe pod nginx-ssd
```

> ✅ **Ponto-chave:** `requiredDuringSchedulingIgnoredDuringExecution` = regra **obrigatória**. Se nenhum nó tiver o label, o Pod fica em `Pending`.

---

## 📝 Questão 2 — Backup e Restore do ETCD

**Peso do domínio:** Cluster Architecture (25%) — tópico clássico e recorrente.

### Enunciado

Faça o **backup** do banco de dados ETCD do control plane e salve o snapshot em `/opt/etcd-backup.db`. Em seguida, descreva o procedimento de **restore** a partir desse snapshot.

Os certificados do ETCD estão nos caminhos padrão do `kubeadm`:

- CA: `/etc/kubernetes/pki/etcd/ca.crt`
- Cert: `/etc/kubernetes/pki/etcd/server.crt`
- Key: `/etc/kubernetes/pki/etcd/server.key`
- Endpoint: `https://127.0.0.1:2379`

### Solução Passo a Passo

> ℹ️ Estes comandos são executados **dentro do nó de control plane** (via SSH na prova). No kind, você pode entrar no nó com:
> ```bash
> docker exec -it cka-lab-control-plane bash
> ```

**1. Confirme a versão da API do etcdctl:**

```bash
export ETCDCTL_API=3
etcdctl version
```

**2. Faça o backup (snapshot save):**

```bash
ETCDCTL_API=3 etcdctl snapshot save /opt/etcd-backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

**3. Verifique a integridade do snapshot:**

```bash
ETCDCTL_API=3 etcdctl snapshot status /opt/etcd-backup.db --write-out=table
```

Saída esperada (exemplo):

```
+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| a1b2c3d4 |    12345 |       1580 |     4.6 MB |
+----------+----------+------------+------------+
```

### Procedimento de Restore

**4. Restaure o snapshot em um novo diretório de dados:**

```bash
ETCDCTL_API=3 etcdctl snapshot restore /opt/etcd-backup.db \
  --data-dir=/var/lib/etcd-restore
```

**5. Aponte o etcd para o novo data-dir.** O etcd em cluster kubeadm roda como *static pod*. Edite o manifest:

```bash
vim /etc/kubernetes/manifests/etcd.yaml
```

Altere o `hostPath` do volume `etcd-data` e o `--data-dir` para o novo diretório:

```yaml
# ... dentro de spec.containers[].command
    - --data-dir=/var/lib/etcd-restore
# ...
# ... dentro de spec.volumes, no volume 'etcd-data'
  - hostPath:
      path: /var/lib/etcd-restore   # <- novo caminho
      type: DirectoryOrCreate
    name: etcd-data
```

**6. Aguarde o kubelet recriar o static pod do etcd** (ele detecta a mudança no manifest automaticamente) e valide:

```bash
# Fora do nó, do seu terminal normal:
k get pods -n kube-system
k get nodes
```

> ✅ **Ponto-chave:** o restore **não** sobrescreve o data-dir atual — ele cria um **novo** diretório. Você precisa apontar o static pod do etcd para esse novo diretório e deixar o kubelet reiniciar o etcd.

---

## 📝 Questão 3 — Expor um Deployment via ClusterIP e NodePort

**Peso do domínio:** Services & Networking (20%)

### Enunciado

1. Crie um Deployment chamado `web` com a imagem `nginx:1.25` e **3 réplicas**.
2. Exponha-o **internamente** com um Service **ClusterIP** na porta `80`.
3. Exponha-o **externamente** com um Service **NodePort** chamado `web-nodeport` na porta `80`, usando a porta de nó `30080`.

### Solução Passo a Passo

**1. Crie o Deployment imperativamente:**

```bash
k create deployment web --image=nginx:1.25 --replicas=3
```

**2. Confirme que as réplicas subiram:**

```bash
k get deployment web
k get pods -l app=web -o wide
```

**3. Exponha via ClusterIP** (tipo padrão do `expose`):

```bash
k expose deployment web --port=80 --target-port=80 --name=web
```

**4. Exponha via NodePort.** A forma imperativa não permite fixar a `nodePort` diretamente, então geramos o YAML e ajustamos:

```bash
k expose deployment web --port=80 --target-port=80 \
  --type=NodePort --name=web-nodeport $do > web-nodeport.yaml
```

Edite o `web-nodeport.yaml` para fixar `nodePort: 30080`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-nodeport
spec:
  type: NodePort
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

Aplique:

```bash
k apply -f web-nodeport.yaml
```

> 💡 **Atalho:** se a questão **não** exigisse uma `nodePort` específica, bastaria o comando imperativo do passo 4 sem editar o YAML — o Kubernetes atribui uma porta automática na faixa `30000–32767`.

**5. Valide os dois Services:**

```bash
k get svc web web-nodeport
```

Saída esperada:

```
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
web            ClusterIP   10.96.x.x       <none>        80/TCP         10s
web-nodeport   NodePort    10.96.y.y       <none>        80:30080/TCP   5s
```

**6. Teste a conectividade interna (ClusterIP)** a partir de um Pod temporário:

```bash
k run test --image=busybox:1.36 --rm -it --restart=Never -- \
  wget -qO- http://web:80
```

**7. Teste o NodePort** (no cluster kind com o `extraPortMappings` configurado):

```bash
# Graças ao mapeamento de porta do kind-cluster-cka.yaml:
curl http://localhost:30080
```

> ✅ **Ponto-chave:** um NodePort é acessível em `<IP-do-nó>:30080`. No kind, o `extraPortMappings` do arquivo de config expõe essa porta no `localhost`.

---

## 🏁 Desafio Extra (para os corajosos)

Combine tudo: crie um Deployment que **só rode em nós sem o taint de manutenção**, exponha-o via Service, e depois faça `drain` de um nó para observar os Pods sendo reagendados. Documente o passo a passo e **submeta como PR**! 🚀

---

## ✍️ Como Submeter Sua Solução

1. Crie sua solução em `simulados/solucoes/seu-usuario/questao-X.md`.
2. Inclua os comandos usados **e** uma breve explicação do raciocínio.
3. Abra um Pull Request seguindo o fluxo do [README principal](../README.md).
4. Peça a revisão de pelo menos uma pessoa do grupo.

**Bons estudos e mãos ao terminal! ☸️**
