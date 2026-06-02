# Estratégia de Dois Repositórios

## O problema

O jobAI tem dois tipos de conteúdo com necessidades opostas:

| Tipo | Exemplos | Necessidade |
|---|---|---|
| **Arquitetura** | Commands, workflows, CLAUDE.md, docs | Público — portfolio, showcase |
| **Dados pessoais** | cv.md, config.json, application_profile.json | Privado — nunca expor |

E um constraint técnico crítico: **as Routines clonam o repo do GitHub para cada execução**. Ou seja, os dados pessoais precisam estar no repo para a automação funcionar.

Você não pode ter os dois em um repo público — teria que escolher entre automação funcionando e privacidade. A solução é dois repos.

---

## A solução

```
github.com/matgermano/jobAI  (PRIVADO)
├── Todo o conteúdo pessoal (cv.md, config.json, etc.)
├── É daqui que as Routines clonam
├── Todo o conteúdo de arquitetura também está aqui
└── Nunca se torna público

        ↓ auto-sync (GitHub Action)

github.com/matgermano/jobai-agent  (PÚBLICO)
├── Apenas os arquivos de arquitetura (commands, workflows, docs)
├── Arquivos exemplo sem dados reais (config.example.json, cv.example.md)
└── Portfolio showcase — qualquer pessoa pode ver, clonar, adaptar
```

---

## O que está em cada repo

### Repositório Privado (jobAI)
```
Tudo — incluindo:
├── data/cv.md                    ← CV real com dados reais
├── config.json                   ← Salary target, preferências reais
├── application_profile.json      ← Nome, telefone, email, LinkedIn reais
├── application_narratives.md     ← Histórias STAR reais
├── data/knowledge_base.json      ← Histórico de vagas encontradas
├── outputs/                      ← Relatórios e CVs gerados
└── .github/workflows/sync-to-public.yml  ← O mecanismo de sync
```

### Repositório Público (jobai-agent)
```
Apenas:
├── CLAUDE.md                     ← Arquitetura do agente (usa "Alex Rivera")
├── README.md                     ← Documentação do projeto
├── roadmap_completo.md           ← Roadmap completo
├── config.example.json           ← Estrutura do config com dados fictícios
├── application_profile.example.json  ← Estrutura do perfil com dados fictícios
├── data/cv.example.md            ← CV fictício (Alex Rivera)
├── .claude/commands/*.md         ← Toda a lógica dos agentes (o showcase principal)
├── .github/workflows/            ← Workflows (exceto sync-to-public.yml)
└── docs/*.md                     ← Esta pasta de documentação
```

---

## Como funciona o auto-sync

O arquivo `.github/workflows/sync-to-public.yml` no repositório privado faz:

1. **Trigger:** Dispara em todo push para `main` que toca arquivos de arquitetura
2. **Clone:** Clona o repo público em `/tmp/public-repo`
3. **Cópia seletiva:** Copia apenas os arquivos seguros (nunca `config.json`, nunca `cv.md`)
4. **Commit:** Se houve mudança, commita e faz push para o repo público

O resultado: quando você adiciona um novo comando ou melhora um workflow no repo privado, o repo público atualiza automaticamente. **Você nunca precisa lembrar de sincronizar manualmente.**

### Arquivos que disparam o sync:
- `CLAUDE.md`
- `README.md`
- `roadmap_completo.md`
- `LICENSE`, `.gitignore`
- `.claude/commands/**`
- `.github/workflows/**` (exceto `sync-to-public.yml`)
- `config.example.json`
- `data/cv.example.md`
- `application_profile.example.json`
- `docs/**`

### Arquivos que NUNCA são sincronizados:
- `config.json` ← dados pessoais
- `data/cv.md` ← CV real
- `application_profile.json` ← contato pessoal
- `application_narratives.md` ← histórias pessoais
- `data/knowledge_base.json` ← histórico de vagas
- `data/kb_*.json` ← slices semanais
- `outputs/` ← relatórios gerados
- `.github/workflows/sync-to-public.yml` ← o próprio workflow de sync

---

## Setup inicial (one-time)

Para ativar o sync, você precisa fazer isso uma vez:

### 1. Criar o repositório público

No GitHub: New repository → nome `jobai-agent` → Public → Create

### 2. Criar um Personal Access Token (PAT)

1. GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Generate new token (classic)
3. Name: `jobai-public-repo-sync`
4. Expiration: 1 year (ou "No expiration" — lembre de renovar)
5. Scope: `repo` (acesso completo a repositórios)
6. Generate token → copiar o token

### 3. Adicionar o PAT como secret no repositório privado

1. No repo privado (jobAI) → Settings → Secrets and variables → Actions
2. New repository secret
3. Name: `PUBLIC_REPO_PAT`
4. Value: o token copiado no passo anterior
5. Add secret

### 4. Fazer o primeiro sync manual

Edite qualquer arquivo de arquitetura (ex: adicione uma linha ao README.md), commite e faça push para o repo privado. O workflow de sync vai disparar e popular o repo público.

Alternativamente, você pode fazer o primeiro push manual:
```bash
# Clone o público
git clone https://github.com/matgermano/jobai-agent.git /tmp/public
# Copie os arquivos relevantes
# Commit e push
```

---

## Por que não usar submodules ou branches?

**Git submodules** criam uma relação entre repos mas não resolvem o problema de separação de dados — você ainda precisaria de dois repos separados e a sincronização seria mais complexa.

**Branches públicas** dentro do mesmo repo privado: o repo inteiro ou é público ou é privado — não existe granularidade por branch na visibilidade do GitHub.

**A solução de dois repos com sync automático** é a mais simples e mantível. O overhead é mínimo — uma vez configurado, funciona sozinho para sempre.

---

## O valor para portfolio

O repositório público mostra:

1. **Arquitetura de agente autônomo** — como CLAUDE.md funciona como memória
2. **Prompt engineering real** — os arquivos de comando com budgets, hard rules, critérios de sucesso
3. **CI/CD para agentes** — workflows com artifacts, job dependencies, error handling
4. **Padrões de dados** — TTL, deduplication, versioning de outputs
5. **Integração MCP** — Notion, Gmail como ferramentas nativas do agente
6. **Roadmap de produto** — fases, cards, critérios de pronto

É um case de estudo completo, funcionando em produção, com dados reais de execução visíveis via git history — sem expor nenhum dado pessoal.
