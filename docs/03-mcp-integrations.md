# MCP — Model Context Protocol

## O que é MCP

MCP (Model Context Protocol) é o protocolo que permite que o Claude opere sistemas externos como se fossem ferramentas nativas. Com MCP, o Claude não apenas fala sobre o Notion — ele abre páginas, cria databases, atualiza propriedades. Não apenas descreve emails — ele os envia.

**Analogia:** APIs tradicionais são como dar ao Claude um manual e pedir pra ele descrever como fazer algo. MCP é como dar ao Claude as mãos para fazer ele mesmo.

```
Sem MCP:  Claude → texto descrevendo como atualizar o Notion
Com MCP:  Claude → chama notion.update_page() → Notion atualizado
```

---

## Como funciona tecnicamente

MCP define um protocolo de comunicação entre:
- **Host** (Claude Code) — o agente que quer usar ferramentas
- **Server** (Notion, Gmail, GitHub, etc.) — o sistema sendo controlado

O servidor MCP expõe um conjunto de **tools** com schemas JSON. O Claude chama essas tools exatamente como chama ferramentas nativas (Read, Write, Bash). A diferença é que as ferramentas MCP fazem chamadas a APIs externas.

```
Claude → mcp__notion__notion-update-page({page_id: "...", data: {...}})
       → Notion API → página atualizada
```

---

## Como conectar um serviço

### Via claude.ai (recomendado — OAuth, sem chave de API)

1. Ir em `claude.ai/customize/connectors`
2. Buscar o conector desejado (Notion, Gmail, GitHub, Slack, etc.)
3. Autorizar via OAuth — o Claude nunca vê sua senha
4. O conector fica disponível automaticamente em todas as sessões Claude Code locais e nas Routines

### Via MCP Server manual (para serviços customizados)

Para serviços que não têm conector oficial, você pode criar um MCP server próprio e configurá-lo no `settings.json`. Esse path é para casos avançados — para a maioria dos serviços populares, os conectores do claude.ai já existem.

---

## Conectores ativos no jobAI

### Notion (cf2d86aa-dc6e-46ab-95cf-1de1ab3020c9)

**Usado para:** Kanban board de roadmap (`/updatekanban`)

**Ferramentas disponíveis:**
- `notion-search` — busca páginas/databases por texto
- `notion-fetch` — lê o conteúdo de uma página
- `notion-update-page` — atualiza propriedades de uma página (ex: mudar Status de "Doing" para "Done")
- `notion-create-pages` — cria novas páginas
- `notion-create-database` — cria databases
- `notion-get-comments` / `notion-create-comment` — comentários em páginas

**Como o agente usa:**
```
/updatekanban "Card 1.4" Done
→ Claude chama notion-search("Card 1.4" no database "jobAI Roadmap")
→ Encontra o page_id
→ Chama notion-update-page com Status = "Done"
→ Confirma a mudança
```

**Para que a Routine acesse o Notion:** O `connector_uuid` do Notion precisa estar no campo `mcp_connections` da Routine. Está configurado em todas as 3 Routines do jobAI.

### Gmail (conectado via claude.ai)

**Usado para:** Alertas de vagas com fit_score ≥ 9

**Como funciona no hunt-jobs:**
```
Após scoring, se alguma vaga tiver fit_score >= 9:
→ Claude chama gmail.send_email()
→ Para: email configurado no config.json
→ Assunto: "jobAI Alert: [Título da Vaga] at [Empresa] — Score 9/10"
→ Corpo: título, empresa, URL, fit_score, principais requirements
```

**Por que email e não Notion?**  
O relatório semanal chega segunda de manhã. Uma vaga 9/10 não pode esperar — ela fica aberta por poucos dias. O email garante que você seja notificado imediatamente, sem precisar checar manualmente.

---

## Roadmap de integrações MCP

### Próximas (Fase 2-3 do roadmap)

**LinkedIn** (via Chrome MCP):  
- Ler seu perfil atual e comparar com keywords do gap_report
- Sugerir melhorias de headline, about, e experiências
- Requer o plugin "Claude in Chrome" (extensão beta)

**Chrome MCP** (browser automation):  
- Preencher formulários de ATS (Greenhouse, Lever, Ashby) automaticamente
- O agente opera o browser como um humano — mais poderoso e mais arriscado
- Você sempre revisa antes de submeter

### Como adicionar um novo conector

1. Verificar se existe em `claude.ai/customize/connectors`
2. Conectar via OAuth
3. Se precisar em Routines: pegar o `connector_uuid` do conector
4. Atualizar a Routine via `/schedule` (update) para adicionar o conector

---

## Modelo de permissão e segurança

**OAuth vs API key:**  
Conectores via claude.ai usam OAuth — o Claude recebe um token de acesso temporário, nunca sua senha. Quando você revoga o conector, o acesso é imediatamente encerrado.

**Ferramentas permitidas:**  
Cada Routine pode especificar `permitted_tools` para um conector MCP. Deixar vazio (`[]`) significa que todas as ferramentas do conector ficam disponíveis. Para ambientes de produção, é boa prática restringir — ex: o Weekly Report pode ler o Notion mas não precisa criar páginas.

**Princípio do mínimo privilégio:**  
Conecte apenas os serviços que o agente realmente precisa. Um agente de análise de CV não precisa de acesso ao Gmail. Um agente de notificação por email não precisa de acesso ao Notion.

---

## Por que MCP importa no contexto de IA

Antes do MCP, integrar LLMs com sistemas externos era um trabalho de engenharia pesado — você precisava construir wrappers, gerenciar auth, escrever código de serialização. Cada integração era custom.

MCP padroniza esse processo. Um servidor MCP bem construído pode ser usado por qualquer cliente compatível (Claude Code, outros hosts). É similar ao que o HTTP fez para a web — um protocolo padrão que desacoplou clientes de servidores.

Para um Product Manager ou AI Builder, entender MCP é entender como construir agentes que realmente operam o mundo — não só geram texto. A diferença entre um chatbot e um agente autônomo está, em grande parte, nos MCPs que ele tem disponíveis.
