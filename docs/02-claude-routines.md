# Claude Routines — Como Funcionam

## O que são Routines

Claude Routines são agentes remotos que rodam na infraestrutura da Anthropic em um schedule definido por você. Cada execução é uma sessão isolada que:
1. Clona o repositório GitHub configurado
2. Recebe o prompt definido
3. Executa com as ferramentas permitidas
4. Commita e faz push dos resultados
5. Encerra — sem estado persistente entre execuções

**Analogia:** É como contratar um freelancer que aparece todo dia às 7h, pega o contexto do repositório, faz o trabalho, entrega os arquivos, e vai embora. No dia seguinte, o mesmo ciclo recomeça do zero.

---

## Routines vs GitHub Actions

| Aspecto | Claude Routines | GitHub Actions |
|---|---|---|
| **Onde roda** | Cloud Anthropic | Cloud GitHub |
| **Custo** | Incluso no plano Pro | Cobra por token de API |
| **Modelo** | Configurável (Sonnet, Opus, Haiku) | Chama a API — você paga |
| **Trigger** | Cron (mínimo 1h) | Cron, push, PR, webhook |
| **Acesso a arquivos** | Clona o repo via git | Clona o repo via git |
| **MCP integrations** | Sim (OAuth nativo) | Não nativamente |
| **Observabilidade** | claude.ai/code/routines | GitHub Actions logs |
| **Complexidade** | Baixa (só o prompt) | Alta (YAML workflow) |

**Por que migramos de GitHub Actions para Routines:**  
GitHub Actions cobrava via API key — cada run consumia tokens e gerava custo. Routines usam o plano Pro, que já está pago. Para automação de agentes de linguagem, Routines são a escolha certa quando você tem o Pro.

---

## Anatomy de uma Routine

Cada Routine tem esta estrutura:

```json
{
  "name": "Daily Job Hunter",
  "cron_expression": "0 10 * * *",
  "enabled": true,
  "job_config": {
    "ccr": {
      "environment_id": "env_...",
      "session_context": {
        "model": "claude-sonnet-4-6",
        "sources": [
          {"git_repository": {"url": "https://github.com/user/repo"}}
        ],
        "allowed_tools": ["Bash", "Read", "Write", "Edit", "Glob", "Grep", "WebSearch", "WebFetch"]
      },
      "events": [
        {
          "data": {
            "type": "user",
            "message": {
              "role": "user",
              "content": "Seu prompt aqui — deve ser self-contained."
            }
          }
        }
      ]
    }
  },
  "mcp_connections": [
    {
      "connector_uuid": "...",
      "name": "Notion",
      "url": "https://mcp.notion.com/mcp"
    }
  ]
}
```

### Campos importantes

**`cron_expression`** — UTC, 5 campos, mínimo 1 hora de intervalo.  
Exemplos:
- `0 10 * * *` → todo dia 10h UTC (= 7h BRT em horário de Brasília)
- `0 23 * * 0` → toda domingo 23h UTC (= domingo 20h BRT)
- `0 9 * * 1` → toda segunda 9h UTC (= segunda 6h BRT)

**`sources`** — O repo que o agente vai clonar no início de cada execução. O agente tem acesso read/write a todos os arquivos do repo.

**`allowed_tools`** — O que o agente pode fazer. Restrinja ao mínimo necessário:
- Job Hunter precisa de `WebSearch` e `WebFetch` para buscar vagas
- CV Optimizer só precisa de `Read`, `Write`, `Edit` — não precisa de web
- Sempre inclua `Bash` para poder rodar `git commit` e `git push`

**`mcp_connections`** — Conectores externos. Use os `connector_uuid` dos conectores configurados em `claude.ai/customize/connectors`. O nome não pode ter espaços ou pontos.

**`model`** — Qual Claude usar. No jobAI usamos `claude-sonnet-4-6`. Você pode usar `claude-haiku-4-5-20251001` para tarefas mais simples e menor consumo do plano.

---

## As 3 Routines do jobAI

### Daily Job Hunter
```
ID: trig_01M7cV6VLteb92jJZMMuvEsQ
Cron: 0 10 * * * (todo dia 10h UTC = 7h BRT)
Model: claude-sonnet-4-6
Tools: Bash, Read, Write, Edit, Glob, Grep, WebSearch, WebFetch
MCP: Notion
```

### Weekly CV Optimizer
```
ID: trig_01QN7yGs2Ts68K2fZkV6TTxX
Cron: 0 23 * * 0 (todo domingo 23h UTC = 20h BRT)
Model: claude-sonnet-4-6
Tools: Bash, Read, Write, Edit, Glob, Grep
MCP: Notion
```

### Weekly Report
```
ID: trig_01VmonY9AJUTapxC4zbt6rWD
Cron: 0 9 * * 1 (toda segunda 9h UTC = 6h BRT)
Model: claude-sonnet-4-6
Tools: Bash, Read, Write, Edit, Glob, Grep
MCP: Notion
```

---

## Como criar uma Routine via /schedule

No Claude Code (local ou web), use o skill `/schedule`:

```
/schedule

Crie uma routine que roda todo dia às 9h BRT, clona github.com/user/repo,
lê .claude/commands/minha-tarefa.md e executa todos os passos.
```

O skill vai:
1. Perguntar o schedule e confirmar a conversão UTC
2. Sugerir o modelo (padrão: Sonnet)
3. Verificar quais MCPs são necessários
4. Mostrar a configuração completa para revisão
5. Criar via API

Você também pode gerenciar em: `claude.ai/code/routines`

---

## O que o agente vê quando executa

O agente começa cada execução com:
1. O prompt definido nos `events` (self-contained — não tem histórico de conversas anteriores)
2. Todos os arquivos do repositório clonado
3. As ferramentas permitidas no `allowed_tools`
4. Os conectores MCP configurados

**Por isso o prompt precisa ser self-contained.** O agente não se lembra do que fez ontem. Ele lê o `CLAUDE.md` e os arquivos de comando para reconstruir o contexto a cada execução.

---

## Boas práticas

**Prompts efetivos para Routines:**
- Comece com "Read CLAUDE.md for full context, then read .claude/commands/X.md and execute every step exactly as written."
- Seja explícito sobre o que salvar, commitar e fazer push
- Defina um critério de parada: "Print the summary and stop."
- Inclua a regra mais importante inline se for crítica: "Never invent skills or experience not in data/cv.md."

**Allowed tools — princípio do mínimo privilégio:**
- Nunca dê `WebSearch` para um agente que não precisa navegar na web
- Sempre inclua `Bash` se o agente precisa fazer git operations
- `Glob` e `Grep` são seguros e úteis para qualquer agente que lê arquivos

**Monitoramento:**
- Ver resultados: `claude.ai/code/routines`
- Os commits no GitHub são o log de execução
- Se o agente falhar silenciosamente, não haverá commit naquele dia — isso é detectável pelo git log

---

## Limitações

- **Mínimo de 1 hora** entre runs — não dá para rodar a cada 30 minutos
- **Sem estado entre sessões** — cada execução começa do zero clonando o repo
- **Sem acesso a arquivos locais** — só o que está no repositório GitHub
- **Sem variáveis de ambiente customizadas** — segredos precisam estar em GitHub Secrets (acessados via workflow) ou no próprio repo (não recomendado para dados sensíveis)
- **Sem `workflow_dispatch`** — para rodar manualmente, use `RemoteTrigger` com `action: "run"` ou o botão no claude.ai/code/routines
