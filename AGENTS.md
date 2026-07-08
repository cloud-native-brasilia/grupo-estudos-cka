# AGENTS.md — Instruções para Agentes de IA neste Repositório

Este arquivo é a **fonte canônica** de regras para qualquer agente de IA (Claude Code, Google Antigravity, ou outro) que trabalhe neste repositório. `CLAUDE.md` apenas importa este arquivo — não duplique regras lá; edite sempre aqui.

Se uma instrução do usuário em uma conversa contradisser este arquivo, siga a instrução da conversa, mas **pergunte se este arquivo deve ser atualizado** para refletir a mudança permanentemente.

---

## Sobre este repositório

Repositório do **grupo de estudos para a certificação CKA (Certified Kubernetes Administrator)** da CNCF/Linux Foundation, mantido pela comunidade Cloud Native Brasília. É um repositório **público**, colaborativo e assíncrono (veja a dinâmica completa no [README.md](README.md)).

**Filosofia do grupo:** a métrica de sucesso é a **aprovação coletiva**, não individual — todo conteúdo deve favorecer quem está aprendendo do zero, não só quem já tem experiência prévia com Kubernetes.

---

## 🇧🇷 Regra inegociável: idioma

**Todo conteúdo escrito neste repositório é em Português do Brasil (pt-BR)** — READMEs, roadmap, exercícios, atas, comentários em YAML, mensagens de commit, nomes de seções, etc.

Exceções aceitáveis:
- Nomes de comandos, flags, recursos do Kubernetes e termos técnicos sem tradução consagrada (`Pod`, `Deployment`, `NodeAffinity`, `kubectl`, `rollout`, etc.) permanecem em inglês, como é padrão na comunidade Kubernetes brasileira.
- Materiais de terceiros incorporados verbatim (ex.: `cheatsheet/cka-cheatsheet-comunidade.md`, PDFs em `guias-de-estudo/`) podem permanecer no idioma original da fonte, desde que isso esteja indicado no arquivo/README que os apresenta.

---

## Estrutura do repositório e onde adicionar conteúdo novo

| Diretório/Arquivo | Propósito | Convenção |
| :--- | :--- | :--- |
| [`README.md`](README.md) | Dinâmica assíncrona, métrica de sucesso, agenda de encontros | Tabela de encontros: `# \| Data \| Google Meet \| Ata` — **sem coluna de gravação** (o grupo decidiu não gravar, apenas publicar atas) |
| [`ROADMAP.md`](ROADMAP.md) | Currículo oficial dividido em blocos + cronograma | Blocos apresentados em **ordem didática** (Workloads & Scheduling → Services & Networking → Storage → Cluster Architecture → Troubleshooting), não na ordem de peso oficial — ideal para quem começa do zero. O peso oficial de cada bloco fica indicado no título de cada seção; a tabela de pesos oficiais (ordem CNCF) fica à parte, para referência |
| [`curriculo-oficial/`](curriculo-oficial/) | Cópia do PDF oficial do currículo CKA (CNCF) | Fonte primária de verdade para qualquer conteúdo sobre domínios/pesos da prova. Antes de alterar ROADMAP.md ou os exercícios, confira este PDF |
| [`atas/`](atas/) | Atas dos encontros síncronos mensais | Nome de arquivo: `AAAA-MM-DD-nome-do-encontro.md`. O grupo **não grava** os encontros — apenas publica a ata |
| [`ambiente-local/`](ambiente-local/) | Setup de cluster local com `kind` | Cluster multi-nó (1 control-plane + 2 workers) para praticar scheduling e manutenção de nós |
| [`links-e-referencias/`](links-e-referencias/) | Links externos (docs oficiais, killer.sh, killercoda, repositórios comunitários) | Documentação oficial do Kubernetes é a única fonte de consulta permitida na prova — priorize sempre esses links |
| [`simulados/`](simulados/) | Exercícios práticos estilo CKA | `EXERCICIOS-EXEMPLO.md` (poucos exercícios com solução bem detalhada) + `BANCO-DE-EXERCICIOS.md` (50 exercícios, distribuídos proporcionalmente ao peso oficial de cada domínio, na mesma ordem didática do ROADMAP.md) |
| [`guias-de-estudo/`](guias-de-estudo/) | Guias/resumos trazidos por membros do grupo | Documentar a origem e o propósito de cada material no README da pasta |
| [`gabaritos-yaml/`](gabaritos-yaml/) | Templates YAML prontos | Apenas para recursos **sem forma imperativa** no `kubectl` (NetworkPolicy, PV, StorageClass, RBAC completo, CRDs, etc.) |
| [`cheatsheet/`](cheatsheet/) | Referência rápida de comandos | Complementa, nunca substitui, o [kubectl Cheat Sheet oficial](https://kubernetes.io/docs/reference/kubectl/cheatsheet/) |
| [`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md) | Avisos de licença de material de terceiros | Ver seção "Material de terceiros" abaixo |

---

## Convenções de conteúdo técnico

- **`kubectl` imperativo em primeiro lugar.** Toda solução de exercício deve priorizar a forma imperativa do `kubectl` (`k run`, `k create`, `k expose`, `k autoscale`, `$do` para gerar YAML) — é o jeito mais rápido de resolver na prova real. Use YAML apenas quando o recurso não tiver forma imperativa (NetworkPolicy, PV/PVC, StorageClass, RBAC composto, CRDs, Gateway API, static pods, ValidatingAdmissionPolicy, etc.) — nesses casos, aponte para um gabarito em [`gabaritos-yaml/`](gabaritos-yaml/) quando existir um.
- **Atalhos padrão.** Sempre que exemplos de comando forem introduzidos, assuma/mencione os atalhos já convencionados no repositório: `alias k=kubectl`, `export do="--dry-run=client -o yaml"`, autocomplete do `kubectl`.
- **Markdown profissional.** Blocos de código bem definidos e delimitados por linguagem (` ```bash `, ` ```yaml `), tabelas para informação estruturada, cabeçalhos consistentes com emojis moderados (seguindo o estilo já usado nos arquivos existentes).
- **Rastreabilidade ao currículo oficial.** Ao criar ou revisar exercícios, cite a habilidade exata do domínio oficial (texto do PDF em `curriculo-oficial/`) a que o exercício se refere.
- **Nunca reproduza questões reais de prova.** Por política da CNCF (reforçada na ata de kickoff, [`atas/2026-07-01-kickoff.md`](atas/2026-07-01-kickoff.md)), nenhum exercício pode ser uma questão real do exame CKA — todos os cenários devem ser criações originais, apenas inspiradas nos domínios do currículo.
- **Datas de encontros futuros.** Encontros ainda não confirmados devem aparecer na agenda apenas com o mês/período aproximado (ex.: "Agosto/2026"), nunca com data e hora inventadas. Só preencha data, hora e link do Google Meet quando o usuário fornecer essa informação explicitamente.

---

## Material de terceiros e licenciamento

Antes de copiar conteúdo de um repositório ou material externo para dentro deste repositório:

1. Verifique a licença da fonte (ex.: MIT permite cópia com atribuição; sempre confirme antes de assumir).
2. Prefira **linkar** em vez de copiar, a menos que o conteúdo agregue valor direto e não duplique o que já existe no repositório (ex.: templates YAML, cheat sheets).
3. Se copiar, registre a atribuição completa (incluindo aviso de copyright/licença, quando exigido) em [`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md) e adicione uma nota de origem no topo do(s) arquivo(s) copiado(s).

---

## Fluxo de trabalho com Git

- **Só commite quando o usuário pedir explicitamente** ("pode fazer o commit", "commit e push", etc.). Nunca commite proativamente.
- **Só dê `push` quando o usuário pedir explicitamente** — commit e push são pedidos separados, a menos que o usuário junte os dois no mesmo pedido.
- Mensagens de commit em português, curtas e focadas no "porquê", terminando com a linha `Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>` (ou a assinatura equivalente da ferramenta em uso).
- Prefira `git add` de arquivos específicos a `git add -A`/`git add .`, para evitar incluir arquivos indesejados.
