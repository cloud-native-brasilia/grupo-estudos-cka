# 🗺️ ROADMAP — Edital CKA

Este roadmap organiza o **currículo oficial da CKA** (Certified Kubernetes Administrator) em **5 blocos principais**, seguindo os domínios e pesos definidos pela CNCF.

> 📄 **Fonte oficial:** o PDF do currículo está disponível neste repositório em [`curriculo-oficial/CKA_Curriculum_v1.35.pdf`](curriculo-oficial/CKA_Curriculum_v1.35.pdf) (cópia da versão v1.35, publicada pela CNCF/Linux Foundation).
>
> ⚠️ **Sempre confira a versão vigente do edital** em [training.linuxfoundation.org](https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/) e no repositório oficial [github.com/cncf/curriculum](https://github.com/cncf/curriculum), pois os pesos e tópicos podem ser atualizados a cada versão do Kubernetes.

---

## 📈 Distribuição de Pesos (ordem oficial do edital)

Esta é a ordem e a nomenclatura **como aparecem no documento oficial da CNCF**:

| Domínio | Peso |
| :--- | :---: |
| Storage | **10%** |
| Workloads and Scheduling | **15%** |
| Services and Networking | **20%** |
| Cluster Architecture, Installation and Configuration | **25%** |
| Troubleshooting | **30%** |

> 🎯 **Observação estratégica:** *Troubleshooting* é o domínio de maior peso (30%). Ele é transversal — você só troubleshoota bem depois de dominar os outros quatro domínios. Por isso ele fica por último tanto na ordem oficial quanto no nosso cronograma.

---

## 🎓 Ordem de Estudo Didática (recomendada para quem começa do zero)

> 💡 **Por que a ordem abaixo é diferente da tabela acima?**
> A tabela oficial lista os domínios por peso na prova, não pela ordem ideal de aprendizado. Para quem **nunca teve contato com Kubernetes**, começar por *Cluster Architecture* (RBAC, kubeadm, ETCD) ou por *Troubleshooting* é desanimador: são tópicos avançados de administração que só fazem sentido depois que você já entende o que é um Pod, um Deployment e um Service.
>
> Por isso, neste roadmap os **blocos foram renumerados (1 a 5) na ordem em que sugerimos estudá-los**. O peso oficial de cada um continua indicado entre parênteses no título — use a tabela acima como referência de "quanto cai na prova", e a numeração abaixo como referência de "em que ordem estudar".

```
0️⃣ Fundamentos (Nivelamento)   → base necessária antes de orquestrar containers
1️⃣ Workloads & Scheduling   → conceitos concretos: Pods, Deployments, config
2️⃣ Services & Networking    → conectar o que você acabou de criar
3️⃣ Storage                  → persistir dados das aplicações
4️⃣ Cluster Architecture     → agora sim: administrar o cluster por trás de tudo
5️⃣ Troubleshooting          → transversal, exige domínio dos 4 blocos anteriores
```

---

## 🏗️ Bloco 0 — Fundamentos e Nivelamento (Pré-requisitos)

Como o grupo possui pessoas com diferentes níveis de experiência, os passos iniciais focam na base conceitual indispensável. Para nivelar o conhecimento, membros mais experientes conduzirão explanações sobre estes temas basilares:

**Ementa - Encontro de Nivelamento 1: O Paradigma da Conteinerização**
- História da infraestrutura: Bare-metal vs Máquinas Virtuais (VMs) vs Containers.
- O que é um container de fato no Linux (Namespaces para isolamento, Cgroups para recursos).
- Ecossistema: Diferença entre Docker, containerd e o padrão CRI (Container Runtime Interface).
- *Prática:* Rodando e inspecionando um container localmente.

**Ementa - Encontro de Nivelamento 2: Redes e Linux para DevOps**
- Fundamentos de redes: TCP/IP, Portas, NAT e como o DNS resolve nomes.
- O que é e como funciona um Load Balancer e Proxy Reverso.
- Comandos essenciais de Linux para troubleshooting (curl, ping, ip, grep, visualização de logs).
- *Prática:* Testando a comunicação entre processos locais.

**Ementa - Encontro de Nivelamento 3: Mundo Declarativo e a Arquitetura do Kubernetes**
- O que é orquestração e os problemas que o Kubernetes resolve?
- Paradigma Imperativo (comandos) vs Declarativo (estado desejado).
- Sintaxe YAML na prática (listas, dicionários, indentação, boas práticas).
- Visão panorâmica da arquitetura (API Server, ETCD, Kubelet, Control Plane vs Worker nodes).
- *Prática:* Escrevendo e entendendo o primeiro manifesto YAML.

---

## ⚙️ Bloco 1 — Workloads & Scheduling (peso oficial: 15%)

Como executar e controlar cargas de trabalho. **Ponto de partida ideal**: são os objetos do dia a dia (Pods, Deployments), concretos e fáceis de visualizar.

- Entender application deployments e como fazer **rolling update e rollback**.
- Usar **ConfigMaps e Secrets** para configurar aplicações.
- Configurar **workload autoscaling** (ex.: HorizontalPodAutoscaler).
- Entender os primitivos usados para criar aplicações **robustas e self-healing** (Deployments, ReplicaSets, DaemonSets, StatefulSets).
- Configurar **admissão e agendamento de Pods** (`resources.limits`, NodeAffinity, `nodeSelector`, Taints & Tolerations, etc.).

---

## 🌐 Bloco 2 — Services & Networking (peso oficial: 20%)

Conectividade dentro e fora do cluster. Faz sentido logo após Workloads: agora que você sabe criar Pods, é hora de expô-los.

- Entender a **conectividade entre Pods** (modelo de rede do Kubernetes).
- Definir e aplicar **NetworkPolicies**.
- Usar os tipos de Service **ClusterIP, NodePort e LoadBalancer**, e seus endpoints.
- Usar a **Gateway API** para gerenciar tráfego de Ingress.
- Saber usar **Ingress controllers** e recursos Ingress.
- Entender e usar o **CoreDNS**.

---

## 💾 Bloco 3 — Storage (peso oficial: 10%)

Persistência de dados no cluster. Encaixa naturalmente após Networking: sua aplicação já roda e já é acessível — agora ela precisa guardar dados.

- Implementar **StorageClasses** e provisionamento dinâmico de volumes.
- Configurar **tipos de volume, modos de acesso** (`ReadWriteOnce`, `ReadOnlyMany`, `ReadWriteMany`) **e reclaim policies** (`Retain`, `Delete`).
- Gerenciar **PersistentVolumes (PV)** e **PersistentVolumeClaims (PVC)**.

---

## 🧱 Bloco 4 — Cluster Architecture, Installation & Configuration (peso oficial: 25%)

Fundamentos de como o cluster é montado, gerenciado e protegido. Deixamos para depois dos blocos anteriores **de propósito**: só faz sentido administrar o "por trás das cortinas" do cluster depois de entender o que roda dentro dele.

- Gerenciar **RBAC** (Roles, ClusterRoles, RoleBindings, ServiceAccounts).
- Preparar a **infraestrutura subjacente** para instalar um cluster Kubernetes.
- Criar e gerenciar clusters Kubernetes usando **`kubeadm`**.
- Gerenciar o **ciclo de vida** de clusters Kubernetes (upgrade de versão, **backup e restore do ETCD**).
- Implementar e configurar um **control plane de alta disponibilidade (HA)**.
- Usar **Helm e Kustomize** para instalar componentes do cluster.
- Entender as **interfaces de extensão** (CNI, CSI, CRI, etc.).
- Entender **CRDs**, instalar e configurar **operators**.

---

## 🔧 Bloco 5 — Troubleshooting (peso oficial: 30%)

O maior peso da prova, e o último bloco do nosso roadmap por ser **transversal**: exige domínio de tudo o que veio antes.

- Solucionar problemas de **clusters e nós** (`NotReady`, kubelet, etc.).
- Solucionar problemas de **componentes do cluster** (control plane e worker nodes).
- **Monitorar o uso de recursos** do cluster e das aplicações.
- Gerenciar e avaliar **streams de saída de containers** (logs).
- Solucionar problemas de **serviços e networking**.

---

## 📅 Cronograma Sugerido (20 semanas + revisão)

Sugestão de ritmo com base em **~6 a 8 horas de estudo por semana**, seguindo a **ordem didática** acima. Ajuste conforme a disponibilidade do grupo. Os encontros síncronos agora são **mensais** — consulte a agenda e os links no [README.md](README.md#-agenda-de-encontros).

| Semana | Foco | Entregável |
| :---: | :--- | :--- |
| **1** | 0️⃣ Nivelamento: O Paradigma da Conteinerização | 📅 **Encontro #2 (Nivelamento 1)** |
| **2** | 0️⃣ Nivelamento: Redes e Linux para DevOps | 📅 **Encontro #3 (Nivelamento 2)** |
| **3** | 0️⃣ Nivelamento: Declarativo e Arq. do Kubernetes | 📅 **Encontro #4 (Nivelamento 3)** |
| **4** | 🔧 Ambiente local (kind) + `kubectl` básico + Pods | Cluster local rodando, primeiro Pod criado |
| **5** | 1️⃣ Deployments — rolling update e rollback | PR de lab rolling update |
| **6** | 1️⃣ Scheduling — `nodeSelector`, NodeAffinity, Taints | 📅 **Encontro Mensal** + PR de lab NodeAffinity |
| **7** | 1️⃣ ConfigMaps, Secrets e workload autoscaling (HPA) | PR de lab ConfigMap/Secret/HPA |
| **8** | 2️⃣ Services — ClusterIP e NodePort | PR de lab exposição de Deployment |
| **9** | 2️⃣ Ingress, Gateway API e CoreDNS | 📅 **Encontro Mensal** + PR de lab Ingress |
| **10** | 2️⃣ NetworkPolicies e conectividade entre Pods | PR de lab NetworkPolicy |
| **11** | 3️⃣ Volumes (`emptyDir`, `hostPath`) e PV/PVC | PR de lab PV/PVC |
| **12** | 3️⃣ StorageClasses, provisionamento e reclaim policy | PR de lab StorageClass |
| **13** | 4️⃣ Arquitetura do cluster — infraestrutura, `kubeadm` | 📅 **Encontro Mensal** + lab de bootstrap de cluster |
| **14** | 4️⃣ RBAC — Roles, ClusterRoles, ServiceAccounts | PR de lab RBAC |
| **15** | 4️⃣ Backup/restore do ETCD, upgrade de cluster | PR de lab ETCD |
| **16** | 4️⃣ HA control plane, Helm, Kustomize, CRDs, operators| 📅 **Encontro Mensal** + PR de lab Helm |
| **17** | 5️⃣ Troubleshooting de Pods e containers | PR de lab debug CrashLoopBackOff |
| **18** | 5️⃣ Troubleshooting de nós e componentes do control plane | PR de lab nó `NotReady` |
| **19** | 5️⃣ Troubleshooting de serviços, rede e monitoramento | PR de lab debug de Service |
| **20** | 🎯 Simulado completo cronometrado (Killer.sh) + revisão | 📅 **Encontro de Revisão** — agendamento das provas 🚀 |

---

## ✅ Checklist de Prontidão para a Prova

Você está pronto(a) quando conseguir, **sem consultar anotações** (apenas a documentação oficial permitida):

- [ ] Criar e atualizar Deployments com rolling update/rollback.
- [ ] Configurar ConfigMaps, Secrets e um HorizontalPodAutoscaler.
- [ ] Configurar NodeAffinity, Taints e Tolerations.
- [ ] Expor aplicações via ClusterIP, NodePort e Ingress.
- [ ] Criar e aplicar NetworkPolicies.
- [ ] Configurar PV, PVC e StorageClasses.
- [ ] Criar e depurar RBAC.
- [ ] Fazer backup e restore do ETCD em menos de 10 minutos.
- [ ] Fazer upgrade de um cluster com `kubeadm`.
- [ ] Diagnosticar Pods em Pending / CrashLoopBackOff / ImagePullBackOff.
- [ ] Recuperar um nó `NotReady`.
- [ ] Navegar rápido pela documentação oficial (dominar o `Ctrl+F`).

> 💡 **Dica de ouro:** configure aliases e autocomplete do `kubectl` desde o dia 1. Velocidade é essencial na prova.
>
> ```bash
> alias k=kubectl
> export do="--dry-run=client -o yaml"   # ex: k run nginx --image=nginx $do
> source <(kubectl completion bash)
> complete -o default -F __start_kubectl k
> ```

---

Pronto para praticar? Vá para o [banco de 50 exercícios](simulados/BANCO-DE-EXERCICIOS.md), organizado exatamente pelos domínios deste currículo. ☸️
