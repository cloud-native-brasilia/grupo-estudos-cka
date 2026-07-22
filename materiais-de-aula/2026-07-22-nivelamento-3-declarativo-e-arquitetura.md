# 🧭 Nivelamento 3 — Mundo Declarativo e a Arquitetura do Kubernetes

> **Encontro 4** · 22/07/2026 (quarta), 20h–21h30 · Condução: Gustavo Fernandes
> Ementa oficial: [`ROADMAP.md`](../ROADMAP.md) — Bloco 0, Fundamentos e Nivelamento.

**Formato:** ~30 min de teoria + ~15 min de prática ao vivo.

**Pré-requisitos para acompanhar a prática:** cluster `kind` de pé, conforme
[`ambiente-local/README.md`](../ambiente-local/README.md). Quem não tiver, acompanha a projeção — ninguém fica travado.

**Atalhos assumidos daqui pra frente** (convenção do repositório):

```bash
alias k=kubectl
export do="--dry-run=client -o yaml"
source <(kubectl completion bash)   # ou zsh
```

> ⚠️ **Aviso testado em sala:** no **zsh** (padrão do macOS), `$do` **não funciona**. O zsh não faz *word splitting* de variáveis, então `k run x --image=nginx $do` manda `--dry-run=client -o yaml` como **um único argumento** e dá erro:
>
> ```
> error: Invalid dry-run value (client -o yaml). Must be "none", "server", or "client".
> ```
>
> Soluções no zsh: usar `${=do}` (força o split) ou `setopt shwordsplit`. **No bash funciona normalmente** — e o terminal da prova CKA é bash, então o `$do` puro está correto para o exame. Só não estranhe se falhar no seu Mac.

---

## 🎯 Objetivos do encontro

Ao final, quem está começando do zero deve conseguir:

1. Explicar **por que** existe um orquestrador, sem decorar definição.
2. Diferenciar **imperativo** de **declarativo** — e reconhecer o padrão fora do Kubernetes.
3. Ler e escrever YAML sem cair nas armadilhas clássicas da linguagem.
4. Desenhar de cabeça o **control plane vs worker nodes** e dizer o que cada componente faz.
5. Enxergar o **loop de reconciliação** acontecendo ao vivo.

O conceito que amarra os cinco: **estado desejado + um loop que reconcilia**. Se sair só isso na cabeça de todo mundo, o encontro valeu.

---

# Parte 1 — Teoria (~30 min)

## 1. Orquestração: que problema é esse? (5 min)

> 🔗 **Gancho com o encontro anterior:** o [Nivelamento 1](../atas/2026-07-08-encontro-2.md) parou exatamente em "surgimento da orquestração de containers: Google/Borg → Kubernetes". Retomamos daí.

No encontro 2 chegamos ao container: processo isolado por *namespaces*, limitado por *cgroups*, empacotado com suas dependências. Ótimo. Agora coloque isso em produção:

| Você tem | O problema aparece quando |
| :--- | :--- |
| 1 container em 1 máquina | — funciona, `docker run` resolve |
| 40 containers em 1 máquina | quem reinicia quando cai às 3h da manhã? |
| 40 containers em 10 máquinas | **em qual máquina** cada um roda? quem decide? |
| ...e uma máquina morre | quem realoca os containers dela? |
| ...e você quer atualizar a versão | como trocar sem derrubar tudo de uma vez? |
| ...e o tráfego triplicou | quem sobe réplicas? quem distribui a carga entre elas? |
| ...e os IPs mudam a cada restart | como um serviço acha o outro? |

Cada linha dessa tabela é uma resposta que o Kubernetes dá:

- **Scheduling** — decidir onde cada container roda, respeitando recursos e restrições.
- **Self-healing** — recriar o que morreu, sem humano no meio.
- **Rollout / rollback** — trocar versão gradualmente e voltar atrás.
- **Escala** — horizontal, manual ou automática.
- **Service discovery e balanceamento** — nomes estáveis sobre IPs efêmeros.
- **Configuração e segredos** — desacoplados da imagem.

### 💡 A virada de chave: infraestrutura efêmera

A [ata do encontro 2](../atas/2026-07-08-encontro-2.md) já registrou a ideia central:

> *"o design do Kubernetes prioriza a infraestrutura como código, garantindo que o estado de configuração seja sempre consistente com o repositório"*

Isso é a diferença cultural de VM para Pod:

| | VM (pet) | Pod (cattle) |
| :--- | :--- | :--- |
| Quando quebra | você entra e conserta | você joga fora e nasce outro |
| Estado | vive na máquina | vive no Git + no etcd |
| Nome/IP | fixo, você decora | efêmero, ninguém decora |
| Mudança | ssh + comando | commit + `apply` |

E é exatamente por isso que o Kubernetes é **declarativo** — que é o nosso próximo tópico. Você não descreve *como consertar*; você descreve *como deve estar*, e alguém fica conferindo.

### ⚖️ Sendo honesto: quando *não* usar

Já discutido no [kickoff](../atas/2026-07-01-kickoff.md): equipes pequenas com poucos serviços, ou casos que um serverless de containers (Cloud Run, ECS Fargate, Fly.io) resolve com uma fração da complexidade operacional. Kubernetes cobra um preço em complexidade — vale quando você tem escala, múltiplos times, ou precisa de portabilidade entre nuvens/on-prem.

---

## 2. Imperativo vs Declarativo (9 min)

A definição de bolso:

> **Imperativo** = você descreve **o caminho**. *"Faça isto, depois aquilo."*
> **Declarativo** = você descreve **o destino**. *"Quero que fique assim."*

Só que essa frase, sozinha, não gruda. O que gruda é ver o padrão nos lugares onde todo mundo já mexeu.

### 2.1 Fora do Kubernetes — você já usa os dois há anos

#### 🅰️ Linguagens de propósito geral são majoritariamente imperativas

C, Java, Python, Go, JavaScript, Bash — todas descendem do modelo de von Neumann: *sequência de instruções que mutam estado*. Você diz o passo a passo.

```python
# Imperativo: EU controlo o laço, o índice, o acumulador.
maiores = []
for u in usuarios:
    if u.idade >= 18:
        maiores.append(u.nome)
```

Note o quanto desse código é *mecânica* e não *intenção*: criar lista vazia, iterar, testar, anexar. A intenção real ("os nomes dos maiores de idade") está diluída.

Mesmo nessas linguagens, você já viu construções declarativas aparecerem — porque elas expressam intenção:

```python
# Declarativo: EU digo o QUÊ. O runtime cuida do COMO.
maiores = [u.nome for u in usuarios if u.idade >= 18]
```

```sql
-- SQL: o exemplo mais puro de declarativo que existe.
SELECT nome FROM usuarios WHERE idade >= 18;
```

Repare no SQL: você **não** diz se é para varrer a tabela inteira, usar um índice, qual algoritmo de join, em que ordem. Quem decide isso é o **query planner**. E ele pode decidir diferente amanhã, se as estatísticas da tabela mudarem — sem você reescrever nada.

> 🔑 **Guarde essa peça:** todo sistema declarativo tem, do outro lado, um **motor que decide o "como"**. SQL tem o *query planner*. O Kubernetes tem os **controllers** e o **scheduler**. Sem esse motor, declarativo é só um arquivo de texto bonito.

#### 🌀 "Então o 'como' é sempre imperativo?"

Pergunta que sempre aparece — e a resposta curta é *no fim das contas, sim*: o processador executa instruções em sequência. Mas a resposta útil é outra:

> **Não existe *um* "como". Existe uma pilha de "comos" — e cada camada é declarativa para a de cima e imperativa para a de baixo.**

```
você        "quero 3 réplicas"           declarativo
Deployment  "crie um RS com 3"           → vira o "o quê" do ReplicaSet
ReplicaSet  conta labels, cria/deleta    código Go, imperativo
kube-proxy  "escreva estas regras"       → iptables é declarativo DE NOVO
netfilter   avalia a tabela de regras    imperativo, no kernel
CPU         executa as instruções...     ...só que não (veja abaixo)
```

Duas quebras que valem por si:

- **kube-proxy delega o "como" outra vez.** Para materializar um Service, ele escreve um *ruleset* de iptables — declarativo, não um roteiro de passos. (Com Cilium/eBPF vira bytecode imperativo — bom contraste.)
- **A CPU inverte tudo no fim.** Um processador *out-of-order* pega seu programa imperativo e lê a sequência como uma **declaração de dependências**: reordena, paraleliza, especula. Você escreveu "faça A, depois B"; ele entendeu "B precisa do resultado de A" e escolheu a ordem sozinho. É um *query planner*, com outro nome.

**Declarativo vs imperativo não é propriedade do sistema — é propriedade da fronteira que você está olhando.**

E isso é prático, não filosófico: quando um Pod não sobe, você precisa saber **em qual camada** olhar. Cada uma falha com vocabulário próprio.

| Sintoma | Camada que falhou | Quem reclama |
| :--- | :--- | :--- |
| `Pending` | o "como" do **agendamento** | `kube-scheduler` |
| `ImagePullBackOff` | o "como" da **execução** | `kubelet` / containerd |
| `CrashLoopBackOff` | o **seu processo** | o container |

#### 🅱️ DSLs — o contraste fica nítido

**DSLs declarativas** (você descreve o resultado):

```css
/* CSS — você não pinta pixel por pixel. Descreve como deve parecer. */
.botao { background: #326ce5; border-radius: 4px; padding: 8px 16px; }
```

```hcl
# Terraform (HCL) — "quero uma instância assim". O provider calcula o plano.
resource "aws_instance" "web" {
  instance_type = "t3.micro"
  ami           = "ami-0abc123"
  tags          = { Name = "web" }
}
```

Outros da mesma família: **HTML** (estrutura, não instruções de renderização), **Dockerfile** no cabeçalho (`FROM`, `EXPOSE`, `ENV`), **Prometheus rules**, **Nix**, **GitHub Actions** na estrutura de jobs, e — claro — **manifestos Kubernetes**.

**DSLs imperativas / procedurais** (você descreve os passos):

```dockerfile
# Dockerfile no corpo: RUN é passo a passo, e a ORDEM importa.
RUN apt-get update
RUN apt-get install -y curl     # se inverter as duas linhas, quebra
COPY . /app
```

```bash
# Bash: o exemplo canônico. Rodar duas vezes ≠ rodar uma vez.
mkdir /opt/app        # segunda execução: "File exists" → erro
useradd deploy
```

O Dockerfile é um híbrido interessante e vale citar: **declarativo no que descreve** (imagem base, portas, variáveis) e **imperativo no como constrói** (a sequência de `RUN`).

#### 🅲 O par que mais ensina: Ansible vs Terraform

Vale um minuto porque quase todo mundo do grupo já tocou em pelo menos um:

| | **Ansible** | **Terraform** |
| :--- | :--- | :--- |
| Modelo | lista **ordenada** de tarefas | grafo de recursos **desejados** |
| Você escreve | "instale, depois configure, depois inicie" | "quero que exista X, Y e Z" |
| Ordem | você define | o motor deduz pelas dependências |
| Sabe o estado atual? | não (por isso módulos idempotentes) | sim — tem *state* e faz `plan` |
| Remover algo | você escreve a tarefa `state: absent` | **apaga do arquivo** e ele destrói |

Aquele último item é o teste definitivo. Em sistema declarativo, **apagar a linha do arquivo apaga o recurso do mundo**, porque o motor compara o desejado com o real. Em imperativo, apagar a linha não faz nada — o efeito da execução anterior continua lá.

#### 🅳 Bônus para quem vem de front-end

jQuery (imperativo: "ache o elemento e mude a classe dele") vs React (declarativo: "para este estado, a tela é assim" — o *reconciler* do React calcula o diff do DOM). O React até usa o mesmo vocabulário do Kubernetes: **reconciliação**.

### 2.2 De volta ao Kubernetes

Agora a tabela fecha sozinha:

```bash
# Imperativo — um comando, um efeito, agora.
k create deploy web --image=nginx:1.27
k scale deploy web --replicas=5
k set image deploy/web nginx=nginx:1.28
k delete pod web-abc123
```

```bash
# Declarativo — um arquivo com o estado desejado, aplicado.
k apply -f web.yaml
```

| Aspecto | Imperativo (`create`, `run`, `scale`, `expose`) | Declarativo (`apply -f`) |
| :--- | :--- | :--- |
| O que você informa | a ação | o estado final |
| Rodar 2x | erro (`AlreadyExists`) | idempotente, sem efeito |
| Rastreável no Git | ❌ está no seu histórico do shell | ✅ é um arquivo versionado |
| Code review | ❌ | ✅ |
| Ideal para | **prova CKA**, exploração, correção pontual | produção, GitOps, times |

### 🎓 O ponto que interessa para a prova

O `AGENTS.md` deste repositório já cravou a regra, e ela é contraintuitiva para quem vem de produção:

> **Na CKA, imperativo em primeiro lugar.** É o jeito mais rápido de resolver. O relógio é o adversário.

Mas alguns recursos **não têm forma imperativa** — não existe `kubectl create networkpolicy`. Para esses, YAML é obrigatório:

`NetworkPolicy` · `PV` / `PVC` · `StorageClass` · RBAC composto · CRDs · Gateway API · static pods · `ValidatingAdmissionPolicy`

Daí a estratégia canônica do repositório — o **melhor dos dois mundos**, documentada em [`gabaritos-yaml/README.md`](../gabaritos-yaml/README.md):

```bash
# 1. Gere o esqueleto com o imperativo (rápido, sem erro de digitação)
k create deploy web --image=nginx:1.27 $do > web.yaml

# 2. Edite só o que o imperativo não cobre
vim web.yaml

# 3. Aplique
k apply -f web.yaml
```

> *"Não decore YAML inteiro. Decore a estrutura."* — e use `k explain` para o resto:
> ```bash
> k explain pod.spec.containers.resources
> k explain deploy.spec.strategy --recursive
> ```

### 🧠 A ponte para o resto do encontro

Declarativo só funciona se **alguém estiver conferindo o tempo todo** se o real bate com o desejado. Esse "alguém" é um **loop de reconciliação**, e ele é literalmente a arquitetura do Kubernetes:

```
     ┌──────────────────────────────────────┐
     │                                      │
     ▼                                      │
 observa o estado atual ──► compara com o desejado ──► age para corrigir
     (watch)                   (diff)              (create/delete/update)
```

Todo controller do Kubernetes é esse loop. Vamos ver ele rodando na prática, no final.

---

## 3. YAML na prática (9 min)

O Kubernetes aceita JSON e YAML. Todo mundo usa YAML porque tem comentários e é mais legível. Em troca, você herda uma linguagem com armadilhas reais.

### 3.1 Os três tipos que você precisa reconhecer

```yaml
# 1) ESCALAR — um valor único
nome: nginx
replicas: 3
ativo: true

# 2) MAPA (dicionário) — pares chave: valor, aninhados por INDENTAÇÃO
metadata:
  name: meu-pod
  labels:
    app: web          # aninhamento = mais indentação

# 3) LISTA (sequência) — itens começando com hífen
containers:
- name: main          # o "-" abre um item
  image: nginx:1.27   # sem "-": ainda é o MESMO item (mapa dentro da lista)
- name: sidecar       # novo "-": SEGUNDO item
  image: busybox
```

**A confusão nº 1 de quem começa:** `-` cria um *item de lista*. As linhas seguintes sem `-`, alinhadas com o `name`, pertencem a esse mesmo item. Lista de mapas é o padrão mais comum em manifestos Kubernetes.

Sintaxe alternativa (*flow style*) — mesma coisa, escrita em uma linha. Guarde essa forma, ela volta na seção de KYAML:

```yaml
labels: { app: web, tier: frontend }
comandos: ["sh", "-c", "sleep 3600"]
```

### 3.2 A estrutura de todo objeto Kubernetes

Quatro campos de topo, **sempre**:

```yaml
apiVersion: v1        # QUAL versão da API (v1, apps/v1, networking.k8s.io/v1...)
kind: Pod             # QUE tipo de objeto
metadata:             # QUEM ele é: nome, namespace, labels, annotations
  name: meu-pod
  labels:
    app: web
spec:                 # COMO ele deve estar — o ESTADO DESEJADO
  containers:
  - name: main
    image: nginx:1.27
```

E existe um quinto que **você nunca escreve**: `status`. Ele é preenchido pelo cluster e representa o **estado real**.

> 🔑 **`spec` = desejado. `status` = real. O controller existe para fazer `status` convergir para `spec`.**
> É o conceito inteiro do Kubernetes num par de campos. Vale escrever no quadro.

Se `apiVersion` ou `kind` estiverem errados, nada mais importa — o API Server rejeita antes de olhar o resto.

```bash
k api-resources | grep -i deploy    # descobre kind, apiVersion e o atalho
```

### 3.3 Multi-documento e blocos de texto

```yaml
# Vários objetos no mesmo arquivo, separados por três hifens
apiVersion: v1
kind: Service
metadata: { name: web-svc }
---
apiVersion: v1
kind: ConfigMap
metadata: { name: web-cfg }
data:
  # "|" preserva as quebras de linha — para arquivos de config inteiros
  nginx.conf: |
    server {
      listen 80;
    }
  # ">" junta tudo numa linha só — para textos longos
  descricao: >
    este texto vira
    uma linha só
```

Variantes de *chomping* que aparecem em manifesto real: `|-` remove a quebra final, `|+` preserva todas.

### 3.4 ⚠️ Os três erros clássicos

**1. TAB.** YAML **proíbe** tabulação para indentação. Só espaço.

```yaml
spec:
	containers:      # ← TAB = "found character that cannot start any token"
```

Configure o editor antes de errar — dica já registrada no [cheatsheet](../cheatsheet/cka-cheatsheet-comunidade.md):

```vim
" ~/.vimrc
set expandtab tabstop=2 shiftwidth=2 number autoindent
```

**2. Indentação inconsistente.** Não existe "quantidade certa" de espaços — existe *consistência* dentro do bloco. O erro típico é o item de lista desalinhado:

```yaml
# ❌ errado
spec:
  containers:
  - name: main
     image: nginx     # 5 espaços vs 4 → parser se perde
# ✅ certo
spec:
  containers:
  - name: main
    image: nginx
```

**3. Dois-pontos dentro de valor não citado.**

```yaml
# ❌ o parser lê "mensagem: erro" como chave e se perde no resto
mensagem: erro: falha ao conectar
# ✅
mensagem: "erro: falha ao conectar"
```

Como se defender **antes** de aplicar:

```bash
k apply -f meu.yaml --dry-run=server    # valida sintaxe E schema, sem criar nada
k apply -f meu.yaml --dry-run=client    # só cliente
```

### 3.5 😈 O "YAML from Hell" — quando o parser adivinha errado

Os três erros acima o parser reclama. **Estes aqui ele aceita em silêncio e faz outra coisa.** A causa raiz: YAML 1.1 tenta **inferir o tipo** de valores não citados.

```yaml
# Parece um bloco de config inocente. Não é.
pais: NO                 # → false (booleano!)   "Norway problem"
producao: off            # → false
recursos: yes            # → true
porta: 22:22             # → 1342  (base 60! sexagesimal)
versao: 1.20             # → 1.2   (float — o zero some)
permissao: 0644          # → 420   (octal)
build: 1e3               # → 1000.0 (notação científica)
id: 12_345               # → 12345 (underscore vira separador)
valor: ~                 # → null
```

O caso mais famoso é o **Norway problem**: o código de país da Noruega é `NO`, e YAML 1.1 lê `NO` como o booleano falso. `yes`, `no`, `on`, `off`, `y`, `n`, `true`, `false` — em qualquer capitalização — são todos booleanos.

E tem mais dois que mordem em produção:

```yaml
# Chave duplicada: muitos parsers não reclamam, o último vence em silêncio
image: nginx:1.27
image: nginx:1.20        # ← este ganha

# Âncoras e aliases: reuso... e a base do ataque "billion laughs" (YAML bomb)
padrao: &base { cpu: "100m" }
container: { <<: *base, memory: "64Mi" }
```

**A regra prática que resolve 95% disso:** na dúvida, **aspas duplas**. Toda string que possa ser confundida com booleano, número, data ou versão vai entre aspas.

```yaml
pais: "NO"
versao: "1.20"
porta: "22:22"
```

#### E no Kubernetes, isso morde de verdade?

Aqui vale honestidade — e é uma boa notícia:

**Menos do que em outros lugares**, porque a API do Kubernetes é **tipada por schema**. Se um campo é `string` e você manda um booleano, o API Server rejeita em vez de aceitar calado:

```bash
# Demonstração de 20 segundos (opcional, se sobrar tempo)
k create cm teste --dry-run=server -o yaml --from-literal=x=y
# Agora escreva à mão um ConfigMap com  pais: NO  (sem aspas) e aplique:
#   error: cannot unmarshal bool into Go value of type string
```

**Onde morde forte** é onde não existe schema tipado, e a maior parte do seu dia é lá: **Helm** (templates que geram YAML por concatenação de texto), **Kustomize**, **docker-compose**, **GitHub Actions**, **Ansible**. E dentro do Kubernetes, os campos genuinamente `string` — `data` de ConfigMap, `args`, `env.value`, labels e annotations.

> 📌 `env.value` **precisa** ser string. `value: 8080` é erro; `value: "8080"` é o certo. Esse é o que mais aparece em prova e em entrevista.

📖 Referência clássica sobre o tema: [*The YAML document from hell*](https://ruudvanasseldonk.com/2023/01/11/the-yaml-document-from-hell), de Ruud van Asseldonk — leitura de 10 min que vale para a vida inteira, não só para a CKA.

### 3.6 ✨ KYAML — a resposta oficial do Kubernetes (novidade quente)

O SIG-CLI olhou para tudo isso e criou o **KYAML**: um **subconjunto** do YAML desenhado para Kubernetes.

- **v1.34** — alpha, atrás de `KUBECTL_KYAML=true`
- **v1.35** — **beta, ligado por padrão** ← é a versão do nosso [currículo oficial](../curriculo-oficial/)

As regras são só três:

1. **Chaves e colchetes explícitos** — `{}` para mapas, `[]` para listas. Indentação vira estética, não semântica.
2. **Toda string entre aspas duplas** — mata a inferência de tipo. Adeus, Norway problem.
3. **Vírgula final permitida** — diffs de Git mais limpos.

O mesmo Pod, nas duas formas:

```yaml
# YAML tradicional
apiVersion: v1
kind: Pod
metadata:
  name: meu-pod
  labels:
    app: web
spec:
  containers:
  - name: main
    image: nginx:1.27
    ports:
    - containerPort: 80
```

```yaml
# KYAML — mesma coisa, sem ambiguidade
---
{
  apiVersion: "v1",
  kind: "Pod",
  metadata: {
    name: "meu-pod",
    labels: {
      app: "web",
    },
  },
  spec: {
    containers: [{
      name: "main",
      image: "nginx:1.27",
      ports: [{
        containerPort: 80,
      }],
    }],
  },
}
```

Repare: `containerPort: 80` continua **sem aspas**, porque é genuinamente um número. Aspas são para strings.

Dois pontos que tiram o medo:

- **Todo KYAML é YAML válido.** É *flow style* com aspas — exatamente a sintaxe da seção 3.1. Qualquer `kubectl`, linter, IDE ou pipeline existente engole sem mudança nenhuma.
- **É formato de saída, não obrigação de entrada.** Você continua escrevendo YAML normal se quiser.

```bash
k get pod meu-pod -o kyaml          # k8s >= 1.35
KUBECTL_KYAML=true k get pod -o kyaml   # se estiver no 1.34
```

> 🎯 **Para a prova:** KYAML **não cai na CKA** hoje, e você não precisa escrever nada nele. Mostro por dois motivos: (1) explica *por que* as armadilhas de YAML existem, melhor que qualquer slide; (2) vocês vão topar com esse formato em issue, blog e output de `kubectl` nos próximos meses e não vão estranhar. Na prova, continue no YAML tradicional gerado pelo `$do`.

Há debate legítimo na comunidade sobre se KYAML resolve um problema real ou adiciona um dialeto a mais — vale ler os dois lados.

---

## 4. Arquitetura do Kubernetes (7 min)

Um cluster tem dois tipos de nó. O nosso `kind` tem exatamente isso: **1 control-plane + 2 workers**, como está comentado em [`ambiente-local/kind-cluster-cka.yaml`](../ambiente-local/kind-cluster-cka.yaml).

```
┌─────────────────── CONTROL PLANE ────────────────────┐
│                                                       │
│   ┌────────────────┐        ┌──────────────────┐     │
│   │  kube-apiserver│◄──────►│      etcd        │     │
│   │  (a PORTA de   │        │ (a MEMÓRIA:      │     │
│   │   entrada)     │        │  key-value, o    │     │
│   └───────┬────────┘        │  estado inteiro) │     │
│           │                 └──────────────────┘     │
│     ┌─────┴─────┬───────────────────┐                │
│     ▼           ▼                   ▼                │
│ ┌────────┐ ┌──────────────┐ ┌────────────────┐      │
│ │  kube- │ │    kube-     │ │    cloud-      │      │
│ │scheduler│ │ controller-  │ │  controller-   │      │
│ │        │ │  manager     │ │   manager      │      │
│ │ ONDE   │ │ RECONCILIA   │ │ (LB/discos da  │      │
│ │ roda?  │ │ (os loops)   │ │  nuvem)        │      │
│ └────────┘ └──────────────┘ └────────────────┘      │
└───────────────────────┬───────────────────────────────┘
                        │  (workers falam com o API Server)
        ┌───────────────┼───────────────┐
        ▼                               ▼
┌──── WORKER 1 ─────┐          ┌──── WORKER 2 ─────┐
│  kubelet          │          │  kubelet          │
│  (o CAPATAZ do nó)│          │                   │
│  kube-proxy       │          │  kube-proxy       │
│  (regras de rede) │          │                   │
│  containerd (CRI) │          │  containerd (CRI) │
│  ┌────┐ ┌────┐    │          │  ┌────┐           │
│  │Pod │ │Pod │    │          │  │Pod │           │
│  └────┘ └────┘    │          │  └────┘           │
└───────────────────┘          └───────────────────┘
```

### Control plane — o cérebro

| Componente | Analogia | O que faz | Se cair... |
| :--- | :--- | :--- | :--- |
| **kube-apiserver** | a **recepção** | Única porta de entrada. Autentica, autoriza, valida e persiste. **Tudo** passa por ele — inclusive os outros componentes | ninguém consegue ler nem mudar nada; **Pods atuais continuam rodando** |
| **etcd** | a **memória** | Banco key-value que guarda o estado inteiro do cluster. **A única fonte da verdade** | perda total do cluster (por isso backup cai na prova) |
| **kube-scheduler** | o **síndico** | Escolhe **em qual nó** cada Pod novo roda. Só decide — não executa | Pods novos ficam eternamente `Pending` |
| **kube-controller-manager** | os **fiscais** | Roda dezenas de loops de reconciliação (Deployment, ReplicaSet, Node, Job...) | nada se auto-corrige; Pod morto não volta |
| **cloud-controller-manager** | o **despachante** | Integra com a nuvem: LoadBalancer, discos, rotas | recursos da nuvem não são provisionados |

### Worker nodes — os braços

| Componente | O que faz |
| :--- | :--- |
| **kubelet** | O agente em cada nó. Pergunta ao API Server "quais Pods são meus?", manda o runtime executar e **reporta a saúde de volta**. É quem faz o `status` existir |
| **kube-proxy** | Programa as regras de rede (iptables/IPVS) que fazem um Service virar tráfego real |
| **container runtime** | `containerd` / CRI-O — quem de fato roda o container, via **CRI** (visto no [Nivelamento 1](../atas/2026-07-08-encontro-2.md)) |

Somam-se os *addons*: **CoreDNS** (DNS interno) e o plugin **CNI** (rede dos Pods).

> 🔗 As três interfaces já vistas no encontro anterior — **CRI** (runtime), **CNI** (rede), **CSI** (storage) — são justamente os pontos de extensão que evitam vendor lock-in.

### 🎬 O caminho de um `kubectl apply` — a história que amarra tudo

Vale contar essa sequência devagar; é a melhor forma de fixar a arquitetura:

1. `k apply -f pod.yaml` → `kubectl` manda um HTTP POST para o **API Server**.
2. **API Server** autentica (quem é você?), autoriza (RBAC: pode fazer isso?), passa por *admission controllers* e **valida contra o schema**.
3. **API Server grava no etcd**. O objeto existe — com `nodeName` vazio. → responde `pod/meu-pod created`. **Seu terminal já voltou.** Nada rodou ainda.
4. O **scheduler** está observando (*watch*) Pods sem nó. Vê o seu, filtra os nós viáveis (recursos? taints? affinity?), pontua, escolhe — e **grava a decisão no API Server**.
5. O **kubelet** do nó escolhido está observando Pods atribuídos a ele. Vê o seu e chama o **containerd** via CRI: baixa a imagem, cria o container.
6. **kubelet** reporta o `status` de volta ao API Server → etcd. `k get pod` mostra `Running`.

Três observações que valem ouro:

- **Ninguém manda em ninguém.** Cada componente *observa* o API Server e age. É um modelo de *nível* (o estado atual), não de *borda* (um evento pontual) — por isso o sistema se recupera sozinho depois de qualquer falha.
- **O API Server é o único que fala com o etcd.** Todo o resto passa por ele.
- **Seu `apply` retorna antes de qualquer container existir.** Ele registra a *intenção*. O resto é o loop de reconciliação. É isso que "declarativo" significa na prática.

---

# Parte 2 — Prática (~15 min)

> Roda no cluster `kind` de 3 nós. Comandos prontos para copiar/colar — os detalhes de setup estão em [`ambiente-local/README.md`](../ambiente-local/README.md).

## 🔧 Aquecimento — enxergar a arquitetura (2 min)

```bash
# Control plane vs workers: olhe a coluna ROLES
k get nodes -o wide

# Os componentes da teoria, rodando como Pods de verdade
k get pods -n kube-system -o wide
```

Aponte na tela: `etcd-cka-lab-control-plane`, `kube-apiserver-...`, `kube-scheduler-...`, `kube-controller-manager-...` — todos no control plane; e `kindnet` / `kube-proxy` — um em **cada** nó, porque todo nó precisa de rede.

## ✍️ Exercício 1 — Seu primeiro manifesto (5 min)

**Todo mundo digita.** Objetivo: sair do "YAML é assustador" em cinco minutos.

```bash
# 1) Gere o esqueleto com o imperativo — NÃO escreva do zero
k run meu-primeiro --image=nginx:1.27 $do > meu-pod.yaml
cat meu-pod.yaml
```

Leia junto com o grupo, apontando os quatro campos: `apiVersion`, `kind`, `metadata`, `spec`.

```bash
# 2) Edite: adicione uma label e a porta
vim meu-pod.yaml
```

```yaml
metadata:
  labels:
    app: web          # ← adicione
    turma: cka        # ← adicione
spec:
  containers:
  - name: meu-primeiro
    image: nginx:1.27
    ports:            # ← adicione
    - containerPort: 80
```

```bash
# 3) Valide ANTES de aplicar — o API Server confere sem criar nada
k apply -f meu-pod.yaml --dry-run=server

# 4) Aplique de verdade
k apply -f meu-pod.yaml
k get pod meu-primeiro -o wide

# 5) Rode o MESMO apply de novo → "unchanged". Idempotência na veia.
k apply -f meu-pod.yaml
```

**Quebre de propósito** (30 s, e é o que mais ensina): tire um espaço da indentação de `image:`, aplique, leia o erro. Depois troque `containerPort: 80` por `containerPort: "80"` e veja o schema reclamar do *tipo*.

```bash
# Bônus, se estiver no 1.35: o mesmo objeto em KYAML
k get pod meu-primeiro -o kyaml
```

## 🎭 Exercício 2 — A dança Deployment → ReplicaSet → Pod (5 min)

**Demonstração conduzida.** É aqui que o loop de reconciliação deixa de ser slide.

```bash
# Deixe rodando num segundo terminal — o filme acontece aqui
k get pods -l app=demo -w
```

**Passo 1 — Criar e ver a cadeia de posse:**

```bash
k create deploy demo --image=nginx:1.27 --replicas=3
k get deploy,rs,pod --show-labels
```

Saída real (testada):

```
deployment.apps/demo              LABELS   app=demo
replicaset.apps/demo-56fc9df44b   LABELS   app=demo,pod-template-hash=56fc9df44b
pod/demo-56fc9df44b-j9vq7         LABELS   app=demo,pod-template-hash=56fc9df44b
```

Mostre a hierarquia nos nomes: `demo` → `demo-56fc9df44b` (ReplicaSet, hash do template) → `demo-56fc9df44b-j9vq7` (Pods, sufixo aleatório). E repare na label extra que apareceu sozinha nos Pods e no RS: **`pod-template-hash`**. Ela é a chave do Passo 3.

```bash
# Quem é o "dono" de cada Pod? A cadeia é explícita no objeto:
k get pod -l app=demo -o jsonpath='{range .items[*]}{.metadata.name}{"  <- "}{.metadata.ownerReferences[0].kind}{"/"}{.metadata.ownerReferences[0].name}{"\n"}{end}'

# O que o ReplicaSet está procurando? Este seletor é a chave de tudo:
k get rs -l app=demo -o jsonpath='{.items[0].spec.selector.matchLabels}'; echo
```

> 💬 **Frase para fixar:** o ReplicaSet **não conhece "seus" Pods**. Ele conta quantos Pods no namespace batem com o **seletor de labels** e compara com `replicas`. É só isso. Toda a mágica dos próximos passos vem daí.

**Passo 2 — Self-healing:**

```bash
k delete pod -l app=demo --field-selector=status.phase=Running --wait=false
# ou apenas:  k delete pod <um-dos-nomes>
```

No terminal do `-w`: o Pod entra em `Terminating` e **outro nasce em segundos**. Ninguém mandou criar. O controller viu 2 ≠ 3 e agiu.

**Passo 3 — 🔥 O momento "aha": um Pod avulso é adotado e morto**

Este passo tem duas partes, e a **primeira dá errado de propósito**. É o melhor momento pedagógico do encontro — não pule.

**3a) A tentativa ingênua:**

```bash
# Um Pod SOLTO, sem Deployment nenhum, com a label da aplicação:
k run intruso-a --image=nginx:1.27 --labels=app=demo
sleep 8
k get pods -l app=demo
```

Resultado real: **4 Pods, e o `intruso-a` continua vivo.** O ReplicaSet não fez nada.

Pergunte ao grupo: *por quê?* A resposta está no seletor:

```bash
k get rs -o jsonpath='{.items[0].spec.selector.matchLabels}'; echo
# {"app":"demo","pod-template-hash":"56fc9df44b"}
```

O seletor exige **duas** labels. `app=demo` sozinha não casa. Um seletor no Kubernetes é **AND de todas as labels** — não basta acertar uma.

**3b) Agora com o seletor completo:**

```bash
HASH=$(k get rs -o jsonpath='{.items[0].metadata.labels.pod-template-hash}')
echo "hash do template: $HASH"

k run intruso-b --image=nginx:1.27 --labels="app=demo,pod-template-hash=$HASH"
sleep 10
k get pods
```

Agora sim: o `intruso-b` **some em segundos**. Confirme quem matou:

```bash
k get events --sort-by=.lastTimestamp | grep -i delete | tail -3
```

Saída real do teste:

```
Normal  SuccessfulDelete  replicaset/demo-56fc9df44b  Deleted pod: intruso-b
```

> **Foi o ReplicaSet, por nome.** Ele nunca soube que o `intruso-b` foi criado à mão. Contou 4 Pods batendo com o seletor, quis 3, e deletou um. **Isso é reconciliação, sem eufemismo.**

Dois detalhes que valem comentar:

- Ele escolheu justamente o intruso, não um dos originais — o ReplicaSet prioriza deletar os Pods **mais novos** e os que ainda não estão prontos, para causar menos dano.
- E agora dá para explicar **por que o `pod-template-hash` existe**: durante um rollout, dois ReplicaSets do mesmo Deployment coexistem, ambos com `app=demo`. Sem uma label que os diferencie, um roubaria os Pods do outro e o rollout seria impossível. O hash é calculado a partir do template do Pod — muda a imagem, muda o hash, nasce um RS novo.

**Passo 4 — Trocar a label e ver o Pod "escapar":**

```bash
POD=$(k get pod -l app=demo -o name | head -1)
k label $POD app=fugitivo --overwrite

k get pods --show-labels
```

Duas coisas ao mesmo tempo:
- O ReplicaSet agora enxerga só 2 Pods com `app=demo` → **cria um terceiro**.
- O Pod renomeado **sobrevive órfão**, sem `ownerReference` funcional. Ninguém mais cuida dele.

```bash
k get pod ${POD#pod/} -o jsonpath='{.metadata.ownerReferences}'; echo
```

Esse é o truque clássico de debug em produção: *trocar a label de um Pod problemático para tirá-lo do balanceamento e investigar com calma, sem derrubar a aplicação.*

**Passo 5 — Rollout: por que existe o ReplicaSet no meio:**

```bash
k set image deploy/demo nginx=nginx:1.28
k get rs -l app=demo          # agora são DOIS ReplicaSets
k rollout status deploy/demo
k rollout history deploy/demo
```

O antigo fica com `0` réplicas, mas **não some** — é ele que torna o `rollout undo` possível.

```bash
k rollout undo deploy/demo
k get rs -l app=demo          # o antigo volta a ter réplicas
```

> **Fecha a tríade:** o **Deployment** gerencia versões (rollout/rollback). O **ReplicaSet** garante a quantidade. O **Pod** roda os containers. Cada um com um trabalho só.

## 🎯 Exercício 3 — O scheduler em ação (3 min)

```bash
# Onde os Pods caíram? Olhe a coluna NODE — o scheduler distribuiu
k get pods -l app=demo -o wide
```

**Deixar um Pod impossível de agendar:**

```bash
k run impossivel --image=nginx:1.27 --overrides='{"spec":{"nodeSelector":{"disktype":"ssd-inexistente"}}}'

k get pod impossivel          # Pending — e vai ficar
k describe pod impossivel | tail -15
```

Leia a mensagem de evento junto com o grupo:

Saída real (testada em cluster de 4 nós):

```
Warning  FailedScheduling  default-scheduler  0/4 nodes are available:
         1 node(s) had untolerated taint(s),
         3 node(s) didn't match Pod's node affinity/selector.
         preemption: 0/4 nodes are available: 4 Preemption is not helpful for scheduling.
```

Vale destrinchar a mensagem, porque ela é um presente de troubleshooting: o scheduler diz **quantos nós ele avaliou** e **por que cada grupo foi descartado**. O nó de taint é o control-plane (que por padrão não recebe carga); os outros 3 são os workers, reprovados no `nodeSelector`.

Três pontos para destacar:

- **Quem reclamou foi o `default-scheduler`** — o componente da teoria, aparecendo por nome.
- O Pod **existe no etcd**, está gravado, mas com `nodeName` vazio. `Pending` significa exatamente "o scheduler ainda não achou casa".
- **Ele não desiste.** Fica tentando para sempre. Resolva o problema e ele agenda sozinho:

```bash
k label node cka-lab-worker disktype=ssd-inexistente
k get pod impossivel -o wide      # agora vai para Running
```

**Cordon — tirar um nó de circulação:**

```bash
k cordon cka-lab-worker2
k get nodes                       # SchedulingDisabled
k scale deploy demo --replicas=8
k get pods -l app=demo -o wide    # nada novo cai no worker2
k uncordon cka-lab-worker2
```

## 🧹 Limpeza

```bash
k delete deploy demo
k delete pod meu-primeiro intruso-a intruso-b impossivel --ignore-not-found
k delete pod -l app=fugitivo --ignore-not-found
k label node cka-lab-worker disktype-
k uncordon cka-lab-worker2
```

> 💡 **Dica de ouro para a demo:** rode tudo num namespace descartável — `k create ns aula && k config set-context --current --namespace=aula`. No fim, `k delete ns aula` limpa tudo de uma vez e você não corre risco de sujar o cluster ao vivo.

---

## 📌 Recado final (1 min)

Três frases para levar para casa:

1. **`spec` é o desejado, `status` é o real.** Todo o Kubernetes é um loop fechando essa distância.
2. **Na CKA, imperativo primeiro** — `$do` gera o YAML, você só edita o que falta.
3. **Na dúvida, aspas duplas.** YAML adivinha tipo, e às vezes adivinha errado.

**Para o próximo encontro:** subir o cluster `kind` ([`ambiente-local/`](../ambiente-local/)) e brincar com os gabaritos em [`gabaritos-yaml/`](../gabaritos-yaml/) — comece pelos menores: `pod.yaml`, `job.yaml`, `configmap-secret.yaml`.

---

## 🔗 Referências

**Documentação oficial** (a única permitida na prova — treine navegar nela):

- [Componentes do Kubernetes](https://kubernetes.io/docs/concepts/overview/components/) — o diagrama oficial da arquitetura
- [Objetos do Kubernetes: spec e status](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/)
- [Gerenciamento declarativo com arquivos de configuração](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/)
- [Gerenciamento imperativo de objetos](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/imperative-command/)
- [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) · [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) · [kube-scheduler](https://kubernetes.io/docs/concepts/scheduling-eviction/kube-scheduler/)
- [Referência do KYAML](https://kubernetes.io/docs/reference/encodings/kyaml/) · [Release do Kubernetes v1.34](https://kubernetes.io/blog/2025/08/27/kubernetes-v1-34-release/)

**Sobre YAML:**

- [The YAML document from hell](https://ruudvanasseldonk.com/2023/01/11/the-yaml-document-from-hell) — Ruud van Asseldonk
- [Especificação YAML 1.2](https://yaml.org/spec/1.2.2/) — a 1.2 corrigiu boa parte das inferências de tipo; muitos parsers ainda usam 1.1
- [KYAML: uma experiência equivocada?](https://vrabbi.cloud/post/kyaml-in-kubernetes-a-solution-in-search-of-a-problem/) — a visão crítica, para ouvir os dois lados

**Neste repositório:**

- [`ROADMAP.md`](../ROADMAP.md) — ementa e cronograma
- [`ambiente-local/`](../ambiente-local/) — cluster `kind` para a prática
- [`gabaritos-yaml/`](../gabaritos-yaml/) — 23 templates prontos
- [`cheatsheet/`](../cheatsheet/) — referência rápida de comandos
- [`simulados/`](../simulados/) — exercícios estilo CKA
