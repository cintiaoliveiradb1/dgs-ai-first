# Registro de Sessão — Jornada do Atendente e Guardrails do Assistente de IA

> **Continuação de:** Registro de Sessão — Product Specialist: Assistente IA Logística (NovaTech)
> **Etapa:** 5 — Design da jornada do atendente com base nos dados de discovery
> **Contexto acumulado:** Mapa de gaps, análise de conflito PROC-042 v1 vs v2 e cruzamento FAQ × normativos já realizados nas etapas anteriores.

---

## Prompt enviado

```
Considerando cenário acima, realizamos o discovery e identificamos os dados abaixo:
- Os atendentes abrem em média 4,1 fontes diferentes por chamado. Nos chamados de
  cálculo de frete especial, esse número sobe para 6,2 — reflexo direto da
  coexistência das duas versões do PROC-042 sem hierarquia declarada.
- 22% dos chamados são reabertos em até 48 horas — indicativo de respostas
  inconsistentes, majoritariamente em tópicos sem documentação oficial
  (rastreamento, seguro, sinistro).
- As dúvidas mais comuns são sobre prazos de entrega (35%), regras de frete (25%),
  política de devolução (20%), SLA e tiers (10%) e outros (10%).

Crie a jornada do atendente em formato textual estruturado incluindo:
- Fluxo principal
- Fluxo de fallback
- Fluxo de feedback
- 2 guardrails de comportamento do assistente
```

## Decisão de contexto

Os dados de discovery foram fornecidos de forma direta e quantificada, sem indicar qual fluxo priorizar ou como estruturar os guardrails. O objetivo foi observar se o agente conectaria os dados de comportamento (6,2 fontes em frete especial, 22% de reabertura) às causas raiz já mapeadas nas etapas anteriores — e se isso se refletiria nas decisões de design dos fluxos.

---

## Output gerado

**Arquivo gerado:** `jornada-atendente-assistente-ia-logistica.md`

### Estrutura do documento produzido

**Fluxo principal** — 6 passos cobrindo os 90% de volume mapeado no discovery (prazos de entrega, frete especial, devolução, SLA/tiers), com exemplo concreto de resposta estruturada para cálculo de frete especial incluindo a regra de transição entre PROC-042 v1 e v2.

**Fluxo de fallback** — acionado quando o assistente não possui documentação suficiente. Inclui 4 gatilhos de entrada, 4 passos de fluxo e tabela com mensagens de fallback pré-escritas para os temas sem cobertura identificados anteriormente (seguro de carga, carga perigosa com frete expresso, sinistro/carga danificada, frete especial acima de 5.000 kg).

**Fluxo de feedback** — 4 passos com avaliação não obrigatória pós-chamado (sim / parcialmente / não), campo aberto para detalhe e agregação de dados para revisão quinzenal da base. Inclui tabela de métricas de acompanhamento com baseline do discovery e metas iniciais.

**2 guardrails de comportamento**, cada um com princípio, comportamentos esperados, violação típica a evitar e comportamento correto:

- **Guardrail 1 — Não responder além da documentação indexada:** o assistente nunca infere, interpola ou usa o FAQ não validado para completar lacunas. Fallback imediato quando o tema não tem cobertura.
- **Guardrail 2 — Transparência sobre fonte e limitação:** toda resposta inclui documento, versão e data de vigência. Respostas parciais (FAQ com ressalva) são sinalizadas explicitamente. Disposições transitórias são informadas antes do cálculo.

**Diagrama de resumo** da jornada em ASCII mostrando os três fluxos integrados (principal, fallback e feedback) em sequência.

---

## Decisões de design e rastreabilidade

Cada elemento da jornada foi ancorado nos dados de discovery e nos achados das etapas anteriores:

| Elemento | Dado que motivou | Achado anterior relacionado |
|---|---|---|
| Fluxo principal cobre prazos + frete + devolução + SLA | 90% das dúvidas concentradas nesses 4 temas | Mapa de cobertura — temas com documentação oficial |
| Exemplo de resposta com regra de transição v1/v2 | 6,2 fontes abertas por chamado em frete especial | Conflito PROC-042: disposição transitória por data |
| Fallback para seguro, sinistro e cargas perigosas | 22% de reabertura em temas sem documentação oficial | Cruzamento FAQ: itens 22, 32 e 38 classificados como "bloquear" |
| Mensagens de fallback pré-escritas | Especificidade reduz risco de improviso do atendente | Guardrail 1 — não responder além da base indexada |
| Feedback não obrigatório | Não bloquear o fluxo do atendente em chamados urgentes | Premissa de design: assistente elimina busca, não adiciona fricção |
| Meta de reabertura < 10% | Baseline de 22% mapeado no discovery | Itens bloqueados no cruzamento FAQ × normativos |
| Revisão quinzenal nas primeiras 8 semanas | Base nova, comportamento a calibrar | G7 — 38 perguntas ainda sem cobertura oficial |
| Guardrail 2 inclui par violação/comportamento correto | Facilitar uso em especificação técnica e treinamento | Item 45 do FAQ com gatilho de desconto desatualizado (v1 vs v2) |

---

## Métricas de acompanhamento definidas

| Métrica | Baseline (discovery) | Meta inicial |
|---|---|---|
| Taxa de reabertura de chamados em 48h | 22% | < 10% |
| Fontes consultadas por chamado — frete especial | 6,2 | ≤ 2 |
| Fontes consultadas por chamado — geral | 4,1 | ≤ 1,5 |
| Taxa de utilidade das respostas do assistente | — (a medir) | ≥ 80% |
| Chamados resolvidos sem escalada em temas cobertos | — (a medir) | ≥ 70% |

---

## Conexão com entregas anteriores

Esta etapa fecha o ciclo das análises anteriores transformando achados em design operacional:

```
Etapa 1 — Mapa de gaps
    → identificou temas cobertos, parciais e ausentes
    → alimentou os gatilhos de fallback e a seleção de temas do fluxo principal

Etapa 2 — Análise de conflito PROC-042 v1 vs v2
    → quantificou o impacto da coexistência de versões (6,2 fontes/chamado)
    → alimentou o exemplo de resposta estruturada e a regra de transição por data

Etapa 3 — Cruzamento FAQ × normativos
    → classificou 4 itens como "bloquear" e 1 como "reescrever"
    → alimentou as mensagens de fallback pré-escritas e os dois guardrails

Etapa 4 — Jornada do atendente  ◄ este documento
    → transforma achados em fluxo operacional com guardrails de comportamento
    → define métricas de acompanhamento ancoradas no baseline de discovery
```

---

*Registro gerado em maio/2025. Sessão conduzida com Claude Sonnet (Anthropic) na interface claude.ai.*
