# jobAI — Roadmap Completo
# Produto + Aprendizado + Documentação
# Ritmo: 1h/dia + 2h fim de semana = ~9h/semana

---

## Como usar este roadmap

Cada card tem:
- O que fazer (ação concreta)
- O que aprender (conceito)
- O que documentar no Obsidian (nota a criar)
- Tempo estimado
- Critério de pronto (como saber que terminou)

Regra de ouro: nenhum card está Done sem os três — feito, funcionando, documentado.

Importe no Notion como database com as colunas:
Status | Fase | Tipo | Tempo | Semana

---

## FASE 0 — Organizar a casa
### Objetivo: estabilizar o que existe antes de construir mais

---

### Card 0.1 — Configurar o Obsidian
- **Tipo:** Setup
- **Tempo:** 1h
- **Semana:** 1

**O que fazer:**
Criar o vault "Segundo Cérebro" com esta estrutura de pastas:
```
00_Inbox        ← captura rápida
01_Projetos     ← jobAI e futuros projetos
02_Conhecimento ← Claude Code, MCP, Routines, etc
03_Carreira     ← vagas, CV, networking
04_Daily        ← notas do dia
```

**O que aprender:**
Por que vault único (não um por projeto). O valor está nas conexões entre as coisas — quando você linka o que aprendeu de MCP com onde aplicou no jobAI, isso vira conhecimento navegável.

**Documentar no Obsidian:**
Criar nota `02_Conhecimento/Obsidian — Como usar este vault.md` explicando a estrutura e a regra de linkagem.

**Critério de pronto:** Vault criado, 5 pastas, primeira nota em cada uma.

---

### Card 0.2 — Colocar os arquivos do perfil no projeto
- **Tipo:** Feature
- **Tempo:** 30min
- **Semana:** 1

**O que fazer:**
Copiar os dois arquivos para a raiz do projeto jobAI:
- `application_profile.json`
- `application_narratives.md`

Commitar: `chore(profile): add structured application profile and narratives`

**O que aprender:**
Por que separar dados estruturados (JSON) de narrativos (markdown). O agente usa o JSON pra preencher campos e o markdown pra responder perguntas abertas.

**Documentar no Obsidian:**
`01_Projetos/jobAI/Perfil de candidatura.md` — o que é cada arquivo e como o agente usa.

**Critério de pronto:** Dois arquivos no repo, commitados, datas do CV conferidas.

---

### Card 0.3 — Migrar de GitHub Actions para Routines
- **Tipo:** Infra
- **Tempo:** 2h
- **Semana:** 1 (fim de semana)

**O que fazer:**
1. Abrir o Claude Code na pasta do projeto
2. Criar as 3 Routines com `/schedule`:
   - Daily Job Hunter: todo dia 7h BRT
   - Weekly CV Optimizer: domingo 20h BRT
   - Weekly Report: segunda 6h BRT
3. Desativar os workflows do GitHub Actions
4. Testar rodando uma Routine manualmente

**O que aprender:**
Routines rodam na nuvem da Anthropic usando seu Pro — sem custo de API, sem precisar de máquina ligada. GitHub Actions precisa de API key e cobra por token.

**Documentar no Obsidian:**
`02_Conhecimento/Claude Routines.md` — o que são, como criar, diferença vs GitHub Actions, limite de runs do Pro.

**Critério de pronto:** 3 Routines ativas, GitHub Actions desativados, um run manual bem-sucedido.

---

### Card 0.4 — Montar o Kanban no Notion
- **Tipo:** Setup
- **Tempo:** 1h
- **Semana:** 2

**O que fazer:**
1. Ir em `claude.ai/customize/connectors` e adicionar o Notion
2. Criar uma página "jobAI" no Notion
3. Adicionar a conexão do Claude nessa página (menu "..." → Add connections)
4. Abrir o Claude Code local e rodar:

```
Read jobAI_roadmap_notion.md and create a Notion database 
inside the page called "jobAI" with these properties:
- Card (title)
- Fase (select): Fase 0, Fase 1, Fase 2, Fase 3, Fase 4, Fase 5
- Tipo (select): Feature, Learning, Docs, Infra, Setup
- Tempo (text)
- Status (select): Backlog, To Do, Doing, Done

Create one entry per card in the file. Set Fase 0 cards to 
"To Do" and everything else to "Backlog". Do not invent cards.
```

5. No Notion, mudar a view para Board agrupado por Status

**O que aprender:**
MCP (Model Context Protocol) — o protocolo que permite o Claude operar sistemas externos como o Notion. Quando você conecta via claude.ai, os conectores ficam disponíveis no Claude Code local automaticamente (só funciona com OAuth Pro, não com API key).

**Documentar no Obsidian:**
`02_Conhecimento/MCP — O que é e como funciona.md` — definição, diferença de API tradicional, como conectar, modelo de permissão.

**Critério de pronto:** Kanban no Notion com todos os cards, view de Board funcionando.

---

## FASE 1 — Descoberta mais inteligente
### Objetivo: vagas certas chegando sem esforço

---

### Card 1.1 — Expandir escopo de roles
- **Tipo:** Feature
- **Tempo:** 30min
- **Semana:** 3

**O que fazer:**
Abrir `config.json` e adicionar nos target_roles:
- AI Builder
- AI Engineer
- Automation Specialist
- Process Transformation Specialist
- Solutions Engineer

**O que aprender:**
Como o `config.json` molda o comportamento de todos os agentes. Alterar um arquivo de configuração central é mais poderoso que alterar cada agente individualmente.

**Documentar no Obsidian:**
`01_Projetos/jobAI/Como o config molda os agentes.md`

**Critério de pronto:** Próximo run do job hunter traz vagas das novas categorias.

---

### Card 1.2 — Deduplicação e TTL
- **Tipo:** Feature
- **Tempo:** 1h
- **Semana:** 3

**O que fazer:**
Abrir o Claude Code e pedir:
```
Read .claude/commands/hunt-jobs.md and data/knowledge_base.json.
Add URL-based deduplication before appending new jobs.
Add TTL: remove entries older than 90 days on each weekly run.
Commit the changes.
```

**O que aprender:**
Data hygiene — por que dados sujos distorcem análise. Uma vaga salva 5 vezes parece 5x mais demandada.

**Documentar no Obsidian:**
`02_Conhecimento/Data hygiene em agentes.md`

**Critério de pronto:** knowledge_base não duplica, não cresce infinito.

---

### Card 1.3 — Trend tracking real
- **Tipo:** Feature
- **Tempo:** 1h
- **Semana:** 4

**O que fazer:**
Pedir pro Claude Code atualizar o `analyze-gaps.md` para comparar keywords desta semana vs semana anterior com números reais — não setas estimadas.

**O que aprender:**
Análise temporal de dados. Tendência real vs snapshot isolado.

**Documentar no Obsidian:**
`01_Projetos/jobAI/Como funciona o trend tracking.md`

**Critério de pronto:** Weekly report mostra "↑ 4→7 jobs" em vez de "↑↑↑".

---

### Card 1.4 — Alerta por email para vagas top
- **Tipo:** Feature
- **Tempo:** 1h 30min
- **Semana:** 4

**O que fazer:**
1. Conectar Gmail em `claude.ai/customize/connectors`
2. Pedir pro Claude Code atualizar o hunt-jobs para enviar email quando score >= 9

**O que aprender:**
Trigger condicional — o agente age diferente dependendo do resultado. Segundo MCP conectado (Gmail além do Notion).

**Documentar no Obsidian:**
`02_Conhecimento/MCP — Gmail.md` — como conectar, casos de uso.

**Critério de pronto:** Recebe email quando aparece vaga 9-10, sem esperar segunda.

---

## FASE 2 — Posicionamento
### Objetivo: CV e LinkedIn sempre alinhados ao mercado

---

### Card 2.1 — CV personalizado por vaga
- **Tipo:** Feature
- **Tempo:** 2h
- **Semana:** 5 (fim de semana)

**O que fazer:**
Criar novo comando `.claude/commands/cv-by-job.md`:
Dado uma URL de vaga, o agente lê a vaga, compara com seu perfil, e gera um `cv_[empresa].md` customizado pra aquela oportunidade específica.

**O que aprender:**
Geração condicional por contexto. Por que CV genérico converte menos que CV alinhado.

**Documentar no Obsidian:**
`01_Projetos/jobAI/CV por vaga — como funciona.md`

**Critério de pronto:** Gera cv_[empresa].md diferente do cv genérico, com bullets realinhados.

---

### Card 2.2 — Análise do LinkedIn
- **Tipo:** Feature
- **Tempo:** 2h
- **Semana:** 6

**O que fazer:**
Criar comando que compara seu headline, about e experiências do LinkedIn com as keywords mais pedidas pelo mercado (do gap_report), e sugere melhorias de copy.

**O que aprender:**
LinkedIn como produto. Headline é SEO, about é pitch, experiências são prova.

**Documentar no Obsidian:**
`03_Carreira/LinkedIn — estratégia de posicionamento.md`

**Critério de pronto:** Relatório com sugestões concretas de melhoria do perfil.

---

### Card 2.3 — Alinhamento com seus interesses
- **Tipo:** Feature
- **Tempo:** 1h
- **Semana:** 6

**O que fazer:**
Adicionar no `application_profile.json` uma seção `interests` com o que você quer fazer, gosta de fazer, e quer evitar. O agente passa a ponderar isso nas recomendações.

**O que aprender:**
Otimização multi-objetivo — equilibrar o que o mercado pede com o que você quer.

**Documentar no Obsidian:**
`03_Carreira/O que eu quero — clareza de direção.md`

**Critério de pronto:** Recomendações do agente refletem seus interesses, não só demanda de mercado.

---

## FASE 3 — Aplicação assistida
### Objetivo: eliminar o trabalho mecânico de preencher formulários

---

### Card 3.1 — Aprender Chrome MCP
- **Tipo:** Learning
- **Tempo:** 2h
- **Semana:** 7 (fim de semana)

**O que fazer:**
1. Instalar o Claude in Chrome (extensão beta)
2. Testar o agente preenchendo um formulário simples (não de vaga ainda)
3. Entender o modelo de permissão e os riscos

**O que aprender:**
Browser automation — o agente opera o navegador como um humano. Mais poderoso e mais arriscado que MCP de API.

**Documentar no Obsidian:**
`02_Conhecimento/Chrome MCP — browser automation.md` — o que é, como instalar, riscos, quando usar.

**Critério de pronto:** Consegue fazer o agente abrir uma página e preencher um campo simples.

---

### Card 3.2 — Engine de preenchimento para ATS
- **Tipo:** Feature
- **Tempo:** 3h
- **Semana:** 8

**O que fazer:**
Criar comando `.claude/commands/apply-ats.md`:
Dado uma URL de vaga em Greenhouse, Lever ou Ashby, o agente abre o formulário, lê o `application_profile.json`, preenche todos os campos e para antes de submeter para sua revisão.

**O que aprender:**
Mapeamento de campos — como o agente relaciona campos de formulário com dados estruturados. Automação assistida vs automação total.

**Documentar no Obsidian:**
`01_Projetos/jobAI/Apply ATS — como funciona.md`

**Critério de pronto:** Preenche uma vaga real de ATS, você só revisa e dá o clique final.

---

### Card 3.3 — Pacote de candidatura completo
- **Tipo:** Feature
- **Tempo:** 2h
- **Semana:** 9

**O que fazer:**
Criar comando `/apply-prep`:
Dado uma URL de vaga, o agente gera em um único pacote:
- CV customizado pra vaga
- Respostas rascunhadas das perguntas abertas
- Briefing da empresa (contexto, produto, cultura)
- Suas histórias STAR mais relevantes pra aquela vaga

**O que aprender:**
RAG na prática — o agente combina dados externos (vaga, empresa) com seus dados internos (profile, narratives) para gerar algo personalizado.

**Documentar no Obsidian:**
`01_Projetos/jobAI/Apply prep — pacote de candidatura.md`

**Critério de pronto:** Pacote completo em menos de 5 minutos por vaga.

---

### Card 3.4 — Tracker de candidaturas no Notion
- **Tipo:** Feature
- **Tempo:** 1h 30min
- **Semana:** 9

**O que fazer:**
Adicionar campo `applied` e `status` no knowledge_base. Criar segundo database no Notion "Candidaturas" com: empresa, vaga, data, status, próximo passo.

**O que aprender:**
Gestão de pipeline de candidatura. Por que acompanhar é tão importante quanto aplicar.

**Documentar no Obsidian:**
`03_Carreira/Pipeline de candidatura.md`

**Critério de pronto:** Dashboard no Notion mostrando todas as candidaturas e status.

---

## FASE 4 — Especialização
### Objetivo: você como profissional de IA, não só usuário

---

### Card 4.1 — Criar uma Skill própria
- **Tipo:** Learning
- **Tempo:** 2h
- **Semana:** 10 (fim de semana)

**O que fazer:**
Empacotar "otimização de CV para ATS" como uma Claude Skill reutilizável — que qualquer pessoa pode usar no próprio projeto.

**O que aprender:**
Skills — pacotes de conhecimento reutilizáveis. Diferença entre usar Claude Code e estender o Claude Code.

**Documentar no Obsidian:**
`02_Conhecimento/Claude Skills — como criar.md`

**Critério de pronto:** Skill funciona e poderia ser usada em outro projeto.

---

### Card 4.2 — Explorar Cowork
- **Tipo:** Learning
- **Tempo:** 1h
- **Semana:** 11

**O que fazer:**
Testar o Cowork para uma tarefa de conhecimento não-técnica — por exemplo, organizar suas notas do Obsidian automaticamente.

**O que aprender:**
Diferença entre Claude Code (dev, código, arquivos) e Cowork (trabalho de conhecimento, não-técnico). Quando usar cada um.

**Documentar no Obsidian:**
`02_Conhecimento/Cowork vs Claude Code — quando usar cada um.md`

**Critério de pronto:** Consegue explicar a diferença e citar um caso de uso real de cada.

---

### Card 4.3 — Preparação de entrevista por vaga
- **Tipo:** Feature
- **Tempo:** 2h
- **Semana:** 11

**O que fazer:**
Criar comando `/interview-prep`:
Dado o nome de uma empresa, o agente pesquisa o produto, cultura, notícias recentes, e gera um briefing personalizado com suas histórias mais relevantes para aquela entrevista.

**O que aprender:**
Pesquisa contextual em tempo real + seus dados internos. Como se preparar com IA sem soar artificial.

**Documentar no Obsidian:**
`03_Carreira/Como me preparar para entrevistas com IA.md`

**Critério de pronto:** Briefing gerado em menos de 3 minutos que cobre empresa, perguntas prováveis e suas histórias.

---

### Card 4.4 — Case de estudo publicável
- **Tipo:** Docs
- **Tempo:** 3h
- **Semana:** 12 (fim de semana)

**O que fazer:**
Transformar toda a jornada do jobAI num case documentado:
- AS IS (antes)
- Decisões técnicas e por quê
- O que quebrou e como debugou
- Resultados reais (vagas encontradas, tempo economizado)
- O que aprendeu

**O que aprender:**
Storytelling técnico. Como transformar um projeto em prova de competência.

**Documentar no Obsidian:**
`01_Projetos/jobAI/Case de estudo completo.md`

**Critério de pronto:** Texto publicável que você mostra em entrevistas e no LinkedIn.

---

## Resumo executivo

| Fase | Foco | Semanas | Horas |
|---|---|---|---|
| Fase 0 | Organizar a casa | 1-2 | ~5h |
| Fase 1 | Descoberta inteligente | 3-4 | ~6h |
| Fase 2 | Posicionamento | 5-6 | ~7h |
| Fase 3 | Aplicação assistida | 7-9 | ~10h |
| Fase 4 | Especialização | 10-12 | ~10h |
| **Total** | | **12 semanas** | **~38h** |

---

## O que você aprende em cada fase (trilha de conhecimento)

- **Fase 0:** Obsidian, Claude Routines, MCP básico (Notion)
- **Fase 1:** Config de agente, data hygiene, análise temporal, MCP Gmail
- **Fase 2:** Geração condicional, LinkedIn como produto, otimização multi-objetivo
- **Fase 3:** Chrome MCP, browser automation, RAG na prática, pipeline de candidatura
- **Fase 4:** Skills, Cowork, pesquisa contextual, storytelling técnico

---

## Estrutura do vault Obsidian ao final das 12 semanas

```
00_Inbox/
01_Projetos/
  └── jobAI/
        ├── Perfil de candidatura
        ├── Como o config molda os agentes
        ├── CV por vaga
        ├── Apply ATS
        ├── Apply prep
        ├── Pipeline de candidatura
        └── Case de estudo completo
02_Conhecimento/
  ├── Obsidian — Como usar este vault
  ├── Claude Routines
  ├── MCP — O que é e como funciona
  ├── MCP — Gmail
  ├── Chrome MCP — browser automation
  ├── Data hygiene em agentes
  ├── Claude Skills — como criar
  └── Cowork vs Claude Code
03_Carreira/
  ├── LinkedIn — estratégia de posicionamento
  ├── O que eu quero — clareza de direção
  ├── Pipeline de candidatura
  └── Como me preparar para entrevistas com IA
04_Daily/
```
