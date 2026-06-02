# Prompt Engineering — Padrões Usados no jobAI

## O que é prompt engineering aqui

No jobAI, "prompt engineering" não é sobre escrever frases bonitas para o ChatGPT. É sobre arquitetar instruções que um agente autônomo segue de forma confiável, repetível e segura — sem supervisão humana em cada execução.

A diferença entre um agente que funciona bem e um que alucina ou gasta tokens à toa está quase inteiramente na qualidade dos prompts e na estrutura de como o contexto é entregue.

---

## Padrão 1: CLAUDE.md como memória persistente do agente

**O que é:**  
Em vez de repetir o perfil do candidato, as regras de localização e os constraints éticos em cada arquivo de comando, tudo isso fica centralizado no `CLAUDE.md`. Claude lê esse arquivo automaticamente no início de cada sessão — é o sistema prompt do agente.

**Por que funciona:**  
Claude Code trata o `CLAUDE.md` como contexto base. Todo agente que roda no projeto — seja local, seja Routine — começa com esse contexto. Você muda as regras uma vez no `CLAUDE.md` e todos os agentes herdam a mudança.

```markdown
# CLAUDE.md (trecho)

## Prime directive
I only work with what exists. I never invent experience, skills,
projects, or achievements that are not already in data/cv.md.
I reframe and surface — I never fabricate.
```

Essa regra aparece uma vez. Mas o CV Optimizer, o Gap Analyzer e o Job Hunter todos a seguem — porque todos leem o `CLAUDE.md`.

**Quando NÃO usar CLAUDE.md para algo:**  
Parâmetros que mudam frequentemente — como target_roles e salary range — ficam em `config.json`. `CLAUDE.md` é para regras comportamentais estáveis. `config.json` é para parâmetros configuráveis.

---

## Padrão 2: Token budget explícito

**O problema sem esse padrão:**  
Um agente sem limite pode decidir fazer 20 buscas na web, abrir 15 páginas e usar 100k tokens numa única execução. Além do custo, isso cria comportamento não-determinístico — resultados diferentes dependendo de quantas páginas ele decidiu abrir.

**Como está implementado no hunt-jobs.md:**

```markdown
## Token Budget (STRICT — do not exceed)
- Max 4 web searches total across all sources
- Max 2 full page fetches (only if snippet is insufficient)
- Extract all data from snippets — do not fetch pages unnecessarily
- If budget is reached, stop searching and proceed to scoring
```

**O resultado:**  
Cada run do Job Hunter é previsível — entre 1.500 e 2.500 tokens. Sem surpresas no consumo do plano.

**Regra geral:**  
Para qualquer agente que acessa a web ou lê muitos arquivos, defina limites explícitos no prompt: "max X searches", "max Y fetches", "read only files matching pattern Z". O agente vai respeitá-los.

---

## Padrão 3: Hard rules (nunca inventar)

**O problema:**  
LLMs são treinados para serem úteis e completar padrões. Se você pede pra otimizar um CV e não diz explicitamente o que não pode fazer, o modelo vai "ajudar" adicionando skills que parecem relevantes mas que você não tem. Em um CV, isso é fraude.

**Como está implementado:**

```markdown
## Hard Rules — CV Optimizer
- NEVER add skills, tools, experiences, or qualifications not present in data/cv.md
- NEVER change company names, job titles, or dates
- NEVER address ❌ genuine gaps — those belong in the study list only
- Rewrite bullets only — do not add new bullet points
- Preserve ALL metrics exactly as written (never round up, never estimate)
```

Essas regras aparecem múltiplas vezes, em locais diferentes do prompt. Repetição deliberada — o modelo precisa ver a constraint várias vezes para não escorregar.

**Distinção surface vs genuine gap:**  
Esta é a constraint ética central do sistema:

```
⚠️ Surface gap: A skill existe na sua experiência mas não está visível no CV
   → O agente PODE reescrever bullets para surfacear
   → Ex: você usou dados para tomar decisões mas o bullet diz "liderança de produto"
         O agente reescreve para "led data-driven product decisions"

❌ Genuine gap: A skill não existe na sua experiência
   → O agente NUNCA adiciona ao CV
   → Vai para o study list — você decide se quer aprender
   → Ex: você não tem experiência com machine learning pipelines
         Vai pro study list. Ponto.
```

---

## Padrão 4: config.json como fonte de verdade configurável

**O que é:**  
Qualquer parâmetro que pode precisar mudar no futuro fica em `config.json`, não hardcoded nos prompts. Os prompts referenciam o arquivo — não definem os valores.

```markdown
# No hunt-jobs.md:
Read config.json and use:
- search.target_roles for role matching
- search.location_rules.accept and .reject for filtering
- search.salary_min_usd and salary_max_usd for salary scoring
- search.sources for which job boards to search
```

**Por que importa:**  
Quando você adiciona uma nova role ao target (ex: "AI Builder" — card 1.1), você muda `config.json`. Todos os agentes capturam a mudança na próxima execução, sem precisar editar nenhum arquivo de comando.

Separar configuração de lógica é um princípio básico de engenharia de software — aqui aplicado a agentes.

---

## Padrão 5: Critério de sucesso explícito

**O que é:**  
Todo prompt de agente termina com um critério claro de quando parar e o que constitui sucesso.

```markdown
# No hunt-jobs.md:
When complete:
- outputs/jobs_YYYY-MM-DD.json saved with all jobs found today
- data/knowledge_base.json updated (deduplicated)
- data/kb_YYYY-WNN.json updated
- Git commit and push completed
- Print summary: "Job Hunt complete: X new jobs found, Y duplicates skipped, Z high-fit alerts sent"
- Stop.
```

**Por que "Stop." explicitamente?**  
Sem isso, o agente pode continuar "melhorando" o trabalho — verificando mais fontes, refinando scores, adicionando análises. O "Stop." sinaliza que o trabalho foi concluído. Em Routines, o agente encerra a sessão.

---

## Padrão 6: Output estruturado e versionado

**CV versioning:**  
Todo output do CV Optimizer é `cv_v[N].md`, onde N incrementa a cada execução. Nunca sobrescreve o anterior.

**Por que:**  
- Permite rollback: se uma versão ficou pior, você pode comparar
- O changelog explica cada mudança: o que mudou, por que, qual gap estava endereçando
- O histórico de versões é evidência de que o sistema funciona — você pode mostrar em entrevistas: "o agente melhorou a visibilidade de 4 skills ao longo de 6 semanas"

**Changelog auditável:**  
```markdown
# outputs/cv_changelog.md

## cv_v3.md — 2026-06-08
### Changes
- Meridian Software Group bullet 3: added "AI and automation integration" phrase
  **Why:** "AI integration" was a surface gap (present in experience, not visible)
  **Gap addressed:** ⚠️ AI integration (appeared in 7 jobs this week, ↑ from 4)
```

---

## Padrão 7: Prompts auto-suficientes (self-contained)

**O problema com prompts que assumem contexto:**  
Em Routines, cada execução começa do zero. O agente não tem memória de execuções anteriores. Se o prompt diz "continue de onde parou", o agente não sabe onde parou.

**Como os prompts do jobAI são estruturados:**

```markdown
# hunt-jobs.md (trecho inicial)
You are the jobAI automated job hunter.
Read CLAUDE.md for full context on the candidate profile and rules,
then read .claude/commands/hunt-jobs.md and execute every step exactly
as written.
```

Essa frase faz o agente reconstruir todo o contexto a partir dos arquivos do repositório. É determinístico — o mesmo comportamento toda execução.

**Regra geral para Routines:**  
Seu prompt deve funcionar se dado a alguém que acabou de ser contratado para fazer a tarefa e tem acesso apenas ao repositório. Sem histórico, sem contexto implícito.

---

## O que não fazer

**Prompts vagos:**
```
❌ "Busque vagas relevantes para mim"
✅ "Search these 4 sources for these 12 roles, apply these location filters, score 1-10 using this criteria"
```

**Sem limites:**
```
❌ "Pesquise quantas fontes precisar"
✅ "Max 4 searches, max 2 page fetches. Stop after reaching budget."
```

**Sem critério de sucesso:**
```
❌ "Analise os gaps e salve o relatório"
✅ "Save outputs/gap_report.md with these exact sections. Commit and push. Print summary. Stop."
```

**Regras implícitas:**
```
❌ (assume que o agente vai entender que não pode inventar skills)
✅ "NEVER add skills not present in data/cv.md" (explícito, repetido 3 vezes)
```
