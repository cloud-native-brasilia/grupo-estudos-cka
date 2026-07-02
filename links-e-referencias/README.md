# 🔗 Links e Referências

Coletânea de materiais de estudo para a CKA. Mantenha esta lista viva — se encontrar um recurso bom, abra um PR adicionando-o aqui!

---

## 📚 Documentação Oficial (permitida na prova!)

Durante a prova CKA você pode consultar **apenas** os domínios oficiais abaixo. Dominar a navegação nesses sites é parte fundamental da preparação — treine o `Ctrl+F`!

| Recurso | URL | Observação |
| :--- | :--- | :--- |
| **Kubernetes Docs** | https://kubernetes.io/docs/ | A principal referência. Permitida na prova. |
| **Kubernetes Blog** | https://kubernetes.io/blog/ | Permitido na prova. |
| **kubectl Reference** | https://kubernetes.io/docs/reference/kubectl/ | Cheat sheet e referência de comandos. |
| **kubectl Cheat Sheet** | https://kubernetes.io/docs/reference/kubectl/cheatsheet/ | Atalhos essenciais. Estude a fundo. |

> ⚠️ **Atenção:** durante a prova você só pode abrir **uma aba adicional** além do terminal, e apenas nos domínios `kubernetes.io/docs`, `kubernetes.io/blog` (e subdomínios permitidos vigentes). Confirme sempre as regras atualizadas no *Candidate Handbook*.

### Páginas da doc que valem ter na ponta da língua

- **RBAC**: https://kubernetes.io/docs/reference/access-authn-authz/rbac/
- **Backup/Restore ETCD**: https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/
- **Upgrade com kubeadm**: https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
- **Assign Pods to Nodes (Affinity)**: https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/
- **Taints & Tolerations**: https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/
- **Services**: https://kubernetes.io/docs/concepts/services-networking/service/
- **Ingress**: https://kubernetes.io/docs/concepts/services-networking/ingress/
- **NetworkPolicies**: https://kubernetes.io/docs/concepts/services-networking/network-policies/
- **Persistent Volumes**: https://kubernetes.io/docs/concepts/storage/persistent-volumes/

---

## 🎓 Ambientes de Simulação e Prática

| Recurso | URL | Descrição |
| :--- | :--- | :--- |
| **Killer.sh** | https://killer.sh/ | Simulador oficial que **vem incluso** na compra da prova CKA (2 sessões). É **mais difícil que a prova real** — se você vai bem aqui, está pronto. |
| **Killercoda** | https://killercoda.com/ | Ambientes interativos **gratuitos** no navegador. Cenários CKA prontos para praticar. |
| **Killercoda — CKA scenarios** | https://killercoda.com/killer-shell-cka | Playground específico de cenários no estilo CKA. |
| **Play with Kubernetes** | https://labs.play-with-k8s.com/ | Sandbox gratuito para experimentar clusters no navegador. |

> 💡 O **Killer.sh** que acompanha a inscrição fica disponível por 36 horas após ativado. Ative estrategicamente, perto da data da prova.

---

## 🧑‍💻 Repositórios Públicos de Simulados e Exercícios

| Repositório | URL | Descrição |
| :--- | :--- | :--- |
| **CKA Exercises ( dgkanatsios)** | https://github.com/dgkanatsios/CKAD-exercises | Clássico de CKAD, ótimo para fundamentos de workloads (útil também para CKA). |
| **Kubernetes the Hard Way** | https://github.com/kelseyhightower/kubernetes-the-hard-way | Monte um cluster do zero, componente por componente. Excelente para o bloco de Arquitetura. |
| **CKA Practice (bmuschko)** | https://github.com/bmuschko/cka-crash-course | Crash course com exercícios práticos alinhados ao edital da CKA. |
| **Kubernetes Examples** | https://github.com/kubernetes/examples | Exemplos oficiais de manifests para estudar padrões. |

> ⚠️ Alguns repositórios comunitários podem estar desatualizados em relação à versão vigente do Kubernetes na prova. Sempre valide os comandos no seu cluster local do [ambiente-local](../ambiente-local/README.md).

---

## 🎥 Cursos e Vídeos Recomendados

| Recurso | Plataforma | Observação |
| :--- | :--- | :--- |
| **Kubernetes Administrator (CKA) — Mumshad / KodeKloud** | Udemy / KodeKloud | Curso mais popular para a CKA, com labs interativos. |
| **Kubernetes Documentation Tutorials** | kubernetes.io | Tutoriais oficiais passo a passo. |
| **CNCF YouTube** | https://www.youtube.com/@cncf | Palestras e deep-dives da comunidade. |

---

## 📄 Informações Oficiais da Certificação

| Recurso | URL |
| :--- | :--- |
| **Página oficial da CKA** | https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/ |
| **Currículo oficial (GitHub CNCF)** | https://github.com/cncf/curriculum |
| **Currículo oficial (cópia local, PDF)** | [`../curriculo-oficial/CKA_Curriculum_v1.35.pdf`](../curriculo-oficial/CKA_Curriculum_v1.35.pdf) |
| **Candidate Handbook** | https://docs.linuxfoundation.org/tc-docs/certification/lf-candidate-handbook |
| **Important Instructions: CKA & CKAD** | https://docs.linuxfoundation.org/tc-docs/certification/important-instructions-cka-and-ckad |

> 📌 Leia o **Candidate Handbook** e as **Important Instructions** com antecedência: regras de ambiente, itens permitidos, requisitos de webcam e política de reagendamento.

---

## 🛠️ Ferramentas para Ganhar Velocidade

- **kubectl aliases & autocomplete** — configure no dia 1 (veja o [ROADMAP.md](../ROADMAP.md)).
- **`kubectl explain`** — documentação de campos direto no terminal, sem sair do prompt.
- **vim/nano cheat sheets** — você vai editar muito YAML sob pressão.
- **tmux** (opcional) — gerenciar múltiplos terminais durante a prova.

---

Achou um recurso que não está aqui? **Abra um PR!** 🙌
