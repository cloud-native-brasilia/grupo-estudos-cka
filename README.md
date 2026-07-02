# 🚢 Grupo de Estudos CKA — Certified Kubernetes Administrator

Bem-vindo(a) ao repositório do nosso grupo de estudos assíncrono para a certificação **CKA (Certified Kubernetes Administrator)** da CNCF / Linux Foundation.

Este espaço foi desenhado para pessoas ocupadas que querem estudar de forma colaborativa, com liberdade de horário, mas com **compromisso e ritmo coletivo**.

---

## 🎯 Objetivo

Preparar todo o grupo para passar na prova CKA, praticando de forma consistente e compartilhando conhecimento. Nossa filosofia:

> **Ninguém fica para trás. A meta é a aprovação coletiva.**

---

## 📊 Métrica de Sucesso

O sucesso do grupo **não é individual**. Medimos nosso progresso por:

| Métrica | Meta |
| :--- | :--- |
| 🏆 **Aprovação coletiva** | 100% das pessoas ativas aprovadas na CKA |
| ✅ **Labs submetidos** | Cada pessoa entrega ao menos 1 PR de lab por bloco do edital |
| 💬 **Participação** | Dúvidas viram Issues; conhecimento vira documentação |
| 📅 **Presença síncrona** | Participação nos encontros mensais |

O foco em **aprovação coletiva** significa que quem entende um tópico tem a responsabilidade de ajudar quem ainda está travado. Ensinar é a melhor forma de fixar.

---

## 🔄 Dinâmica Assíncrona

Trabalhamos de forma **majoritariamente assíncrona** usando as ferramentas do GitHub. Isso permite que cada pessoa estude no seu horário sem perder a colaboração.

### 1. 💬 Issues — Dúvidas Técnicas

Toda dúvida técnica vira uma **Issue**.

- **Antes de abrir**, pesquise se a dúvida já existe (evite duplicatas).
- Use um título claro: `[Storage] Diferença entre PV e PVC no bind`.
- Marque com labels: `dúvida`, `bloco-X`, `ajuda-desejada`.
- Qualquer pessoa do grupo pode responder — as respostas ficam registradas para consulta futura.
- Ao resolver, adicione um comentário com a solução e **feche a Issue**.

> 💡 Dúvida registrada é conhecimento que fica. Nunca deixe uma dúvida só no chat.

### 2. 🔀 Pull Requests — Submissão de Labs

Os exercícios práticos (labs) são submetidos via **Pull Request**.

Fluxo sugerido:

```bash
# 1. Crie um fork ou branch para o seu lab
git checkout -b lab/nome-sobrenome/backup-etcd

# 2. Adicione sua solução no diretório apropriado
#    Ex: simulados/solucoes/seu-usuario/backup-etcd.md

# 3. Commit e push
git add .
git commit -m "lab: backup e restore do ETCD - Fulano"
git push origin lab/nome-sobrenome/backup-etcd

# 4. Abra o Pull Request no GitHub
```

- Cada PR deve conter os comandos usados e uma breve explicação do raciocínio.
- Ao menos **1 revisão (review)** de outra pessoa do grupo antes do merge.
- Revisar o PR de alguém também conta como estudo! 👀

### 3. 📅 Encontros Síncronos Mensais

Uma vez por mês temos um encontro ao vivo (call de ~1h):

- 🗣️ Revisão do bloco do edital estudado no período.
- 🧩 Resolução conjunta de um simulado ao vivo (compartilhando tela).
- ❓ Discussão das Issues mais difíceis do mês.
- 🎯 Alinhamento do próximo bloco no [ROADMAP.md](ROADMAP.md).

> Não gravamos os encontros. Em vez disso, publicamos a **ata** de cada reunião no repositório, para quem não puder participar ao vivo.

#### Agenda de Encontros

| # | Data | Google Meet | Ata |
| :---: | :--- | :--- | :--- |
| 1 — Kickoff | 01/07/2026 (quarta-feira), 20h | _(a definir)_ | [2026-07-01-kickoff.md](atas/2026-07-01-kickoff.md) |
| 2 | 2ª semana de julho/2026 | _(a definir)_ | _(a definir)_ |
| 3 | Agosto/2026 | _(a definir)_ | _(a definir)_ |
| 4 | Setembro/2026 | _(a definir)_ | _(a definir)_ |
| 5 | Outubro/2026 | _(a definir)_ | _(a definir)_ |
| 6 | Novembro/2026 (revisão final) | _(a definir)_ | _(a definir)_ |

> 📌 As datas e horários exatos dos encontros a partir do 2º serão confirmados via Issue fixada no repositório com, no mínimo, uma semana de antecedência. Atualize esta tabela via PR assim que o link do Google Meet estiver disponível e assim que a ata de cada encontro for publicada em [`atas/`](atas/).

---

## 🗂️ Estrutura do Repositório

```
grupo-estudos-cka/
├── README.md              # Você está aqui
├── ROADMAP.md             # Edital dividido em blocos + cronograma (ordem didática)
├── curriculo-oficial/     # Currículo oficial da CKA (fonte primária)
│   └── CKA_Curriculum_v1.35.pdf
├── atas/                  # Atas dos encontros síncronos mensais
│   └── 2026-07-01-kickoff.md
├── ambiente-local/        # Como montar um cluster local com kind
│   ├── README.md
│   └── kind-cluster-cka.yaml
├── links-e-referencias/   # Documentação oficial e materiais de apoio
│   └── README.md
└── simulados/             # Questões práticas no estilo CKA
    ├── EXERCICIOS-EXEMPLO.md
    └── BANCO-DE-EXERCICIOS.md    # 50 exercícios cobrindo todo o currículo oficial
```

---

## 🚀 Por Onde Começar

1. Leia este README por completo.
2. Configure seu ambiente local seguindo o guia em [ambiente-local/README.md](ambiente-local/README.md).
3. Estude a estrutura do edital no [ROADMAP.md](ROADMAP.md).
4. Explore os materiais em [links-e-referencias/README.md](links-e-referencias/README.md).
5. Pratique com os [simulados/EXERCICIOS-EXEMPLO.md](simulados/EXERCICIOS-EXEMPLO.md) e o [simulados/BANCO-DE-EXERCICIOS.md](simulados/BANCO-DE-EXERCICIOS.md) (50 questões).
6. Abra sua primeira Issue de dúvida ou submeta seu primeiro lab. 💪

---

## 🤝 Código de Conduta

- Seja gentil. Todo mundo aqui está aprendendo.
- Não existe pergunta boba.
- Compartilhe descobertas, atalhos e macetes.
- Celebre as aprovações do grupo! 🎉

---

**Vamos juntos rumo ao `kubectl get certification`! ☸️**
