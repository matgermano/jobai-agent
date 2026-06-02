# Data Hygiene em Agentes de IA

## O problema

Agentes que acumulam dados sem limpeza ficam progressivamente piores. O motivo é simples: análises baseadas em dados sujos produzem insights incorretos.

No contexto do jobAI, dois problemas específicos:

**1. Duplicação:** A mesma vaga pode aparecer no RemoteOK e no Remotive na mesma semana. Se ambas forem salvas, o gap_report vai contar essa vaga como duas demandas separadas pela mesma skill. Skills que aparecem em 3 vagas reais parecem aparecer em 6. A prioridade do seu estudo fica distorcida.

**2. Data stale (dados velhos):** O mercado muda. Uma skill muito demandada em outubro pode ter saturado em março. Se o knowledge_base guarda vagas de um ano atrás sem expiração, o relatório semanal mescla tendências antigas com novas, tornando as recomendações genéricas.

---

## As soluções implementadas

### URL-based deduplication

Antes de salvar qualquer vaga no `knowledge_base.json`, o agente verifica se a URL já existe. Se sim, descarta — mesma URL = mesma vaga.

```json
// Antes (com duplicata):
[
  {"url": "https://remoteok.io/jobs/123", "title": "PM at Acme", ...},
  {"url": "https://remotive.com/job/456", "title": "PM at Acme", ...},
  {"url": "https://remoteok.io/jobs/123", "title": "PM at Acme", ...}  // duplicata
]

// Depois (deduplicated):
[
  {"url": "https://remoteok.io/jobs/123", "title": "PM at Acme", ...},
  {"url": "https://remotive.com/job/456", "title": "PM at Acme", ...}
]
```

A deduplicação acontece por URL, não por título — porque títulos podem variar levemente entre fontes mas a URL é única por posting.

### 90-day TTL (Time To Live)

Toda vaga tem um campo `found_at` com o timestamp de quando foi encontrada. Em cada execução do Weekly CV Optimizer, o agente remove todas as entradas com `found_at` anterior a 90 dias:

```bash
# Dentro do workflow:
CUTOFF=$(date -d '90 days ago' +%Y-%m-%dT%H:%M:%S)
jq --arg cutoff "$CUTOFF" \
  '[.[] | select(.found_at >= $cutoff)] | unique_by(.url)' \
  data/knowledge_base.json > /tmp/kb_clean.json
```

**Por que 90 dias?**  
- 30 dias seria muito curto — não captura ciclos mensais de contratação
- 6 meses seria muito longo — o mercado muda significativamente em 6 meses
- 90 dias = um trimestre = tempo suficiente para identificar tendências sem incluir dados obsoletos

---

## Weekly KB Slices

Além do `knowledge_base.json` (visão completa, 90 dias), o sistema mantém arquivos semanais: `data/kb_YYYY-WNN.json`.

**Por que existem?**

O gap_report precisa responder: "essa skill ficou mais demandada essa semana em relação à semana passada?"

Para responder isso com dados reais, você precisa de dois snapshots separados:
- `kb_2026-W23.json` — vagas encontradas na semana atual
- `kb_2026-W22.json` — vagas encontradas na semana anterior

```
Semana passada: "Python" apareceu em 4 vagas
Essa semana:    "Python" apareceu em 7 vagas
Resultado:      "↑ Python 4→7 jobs (+75%)"
```

Sem os slices semanais, você teria que varrer o `knowledge_base.json` inteiro e tentar separar por data — mais lento, mais complexo, e sujeito a erros nas bordas temporais.

### Naming convention

`kb_YYYY-WNN.json` — onde:
- `YYYY` = ano com 4 dígitos
- `W` = literal "W"
- `NN` = número da semana ISO (01-53), com zero à esquerda

Exemplos: `kb_2026-W22.json`, `kb_2026-W23.json`

O Weekly Report sempre carrega os **2 arquivos mais recentes** para calcular tendências:

```python
# Lógica do weekly-report.md:
1. glob data/kb_*.json
2. sort descending por nome (= descending por data)
3. pegar os 2 primeiros
4. comparar contagens de keywords entre eles
```

---

## Estrutura de um job entry

Cada entrada no `knowledge_base.json` segue esta estrutura:

```json
{
  "title": "Senior Product Manager",
  "company": "Acme Corp",
  "url": "https://himalayas.app/jobs/123",
  "source": "himalayas",
  "location_type": "remote_worldwide",
  "salary_range": "$90k-$120k",
  "fit_score": 8,
  "key_requirements": [
    "5+ years product management",
    "B2B SaaS experience",
    "data-driven decision making",
    "API product experience"
  ],
  "found_at": "2026-06-02T10:15:33Z"
}
```

O `fit_score` (1-10) é calculado pelo Job Hunter baseado em:
- Match de título com os `target_roles` do config.json
- Overlap de skills entre os `key_requirements` e o cv.md
- Conformidade com as regras de localização
- Salary range vs target salary

---

## Impacto no gap analysis

Com dados limpos, o gap analysis torna-se confiável:

```
Gap Report (com dados sujos):
  ❌ "Python" — aparece em 12 vagas ← na verdade eram 6 vagas duplicadas

Gap Report (com dedup + TTL):
  ❌ "Python" — aparece em 6 vagas ← número real, relevante, atual
```

A ordem de prioridade do estudo — o que você deveria aprender primeiro — depende inteiramente da frequência com que cada skill aparece. Dados sujos inverte prioridades.

---

## Evolução futura

Quando o volume crescer (100+ vagas/semana), considerar:
- **PostgreSQL ou SQLite** como storage ao invés de JSON flat files
- **Índice por URL** para dedup em O(1) ao invés de full scan
- **Índice por `found_at`** para TTL sem varrer o array inteiro
- **Análise de salário** — distribuição de salary ranges por role/stack

Por enquanto, JSON flat files no repo são suficientes — simples, sem infra, auditáveis via git diff.
