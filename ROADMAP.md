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
1️⃣ Workloads & Scheduling   → conceitos concretos: Pods, Deployments, config
2️⃣ Services & Networking    → conectar o que você acabou de criar
3️⃣ Storage                  → persistir dados das aplicações
4️⃣ Cluster Architecture     → agora sim: administrar o cluster por trás de tudo
5️⃣ Troubleshooting          → transversal, exige domínio dos 4 blocos anteriores
```

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

## 📅 Cronograma Sugerido (17 semanas + revisão)

Sugestão de ritmo com base em **~6 a 8 horas de estudo por semana**, seguindo a **ordem didática** acima. Ajuste conforme a disponibilidade do grupo. Os encontros síncronos agora são **mensais** — consulte a agenda e os links no [README.md](README.md#-agenda-de-encontros).

| Semana | Foco | Entregável |
| :---: | :--- | :--- |
| **1** | 🔧 Ambiente local (kind) + `kubectl` básico + Pods e ReplicaSets | Cluster local rodando, primeiro Pod criado |
| **2** | 1️⃣ Deployments — rolling update e rollback | 📅 **Encontro #2** (2ª semana de julho) + PR de lab rolling update |
| **3** | 1️⃣ ConfigMaps, Secrets e workload autoscaling (HPA) | PR de lab ConfigMap/Secret/HPA |
| **4** | 1️⃣ Scheduling — `nodeSelector`, NodeAffinity, Taints & Tolerations, limits | PR de lab NodeAffinity |
| **5** | 2️⃣ Services — ClusterIP e NodePort | PR de lab exposição de Deployment |
| **6** | 2️⃣ Ingress, Gateway API e CoreDNS | 📅 **Encontro #3** (Agosto) + PR de lab Ingress |
| **7** | 2️⃣ NetworkPolicies e conectividade entre Pods | PR de lab NetworkPolicy |
| **8** | 3️⃣ Volumes (`emptyDir`, `hostPath`) e PV/PVC | PR de lab PV/PVC |
| **9** | 3️⃣ StorageClasses, provisionamento dinâmico e reclaim policy | PR de lab StorageClass |
| **10** | 4️⃣ Arquitetura do cluster — infraestrutura, `kubeadm init`/`join` | 📅 **Encontro #4** (Setembro) + lab de bootstrap de cluster |
| **11** | 4️⃣ RBAC — Roles, ClusterRoles, ServiceAccounts | PR de lab RBAC |
| **12** | 4️⃣ Backup/restore do ETCD, upgrade de cluster com `kubeadm` | PR de lab ETCD |
| **13** | 4️⃣ HA control plane, Helm, Kustomize, CRDs e operators | 📅 **Encontro #5** (Outubro) + PR de lab Helm |
| **14** | 5️⃣ Troubleshooting de Pods e containers | PR de lab debug CrashLoopBackOff |
| **15** | 5️⃣ Troubleshooting de nós e componentes do control plane | PR de lab nó `NotReady` |
| **16** | 5️⃣ Troubleshooting de serviços, rede e monitoramento de recursos | PR de lab debug de Service |
| **17** | 🎯 Simulado completo cronometrado (Killer.sh) + revisão geral | 📅 **Encontro #6** (Novembro) — revisão final e agendamento das provas 🚀 |

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
