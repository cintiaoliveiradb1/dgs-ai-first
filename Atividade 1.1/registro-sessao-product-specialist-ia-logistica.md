# Registro de Sessão — Product Specialist: Assistente IA Logística

> **Objetivo da sessão:** Analisar a base documental disponível para criação de um assistente de IA voltado ao time de atendimento de uma empresa de logística (NovaTech), identificando temas cobertos, gaps, conflitos e inconsistências entre documentos formais e práticas informais.
>
> **Perfil de condução:** Os prompts foram mantidos intencionalmente simples e objetivos, sem excesso de direcionamento, para observar como o agente se comportaria em cada etapa com contexto mínimo.
>
> **Como a qualidade variou entre as etapas:** Cada etapa complementou a anterior — o mapa geral criou a estrutura de referência, a análise de conflito aprofundou o ponto mais crítico com dados concretos, e o cruzamento com o FAQ fechou o ciclo conectando práticas informais ao que os normativos realmente dizem. Os documentos gerados são progressivamente mais embasados e acionáveis.

---

## Etapa 1 — Mapa de temas e gaps

### Prompt enviado

```
Você está atuando como um Product Specialist. Construa um mapa de temas cobertos
e hipóteses de gaps encontrados.
Você receberá:
- contexto geral
- títulos de documentos
- resumos dos documentos
Sua missão é:
1. Identificar temas e domínios cobertos
2. Detectar possíveis lacunas de cobertura
3. Levantar hipóteses de gaps
4. Identificar sobreposição, redundância ou ausência de informação

Contexto geral: Analisar criação de um assistente de IA para empresa de logística.
Assistente deve auxiliar atendentes apresentando respostas para questionamentos que
hoje só são respondidos após consulta de documentações distribuídas em alguns
documentos, o que dificulta a consulta.

[Títulos e resumos de 5 documentos fornecidos]
```

### Decisão de contexto

O prompt forneceu apenas títulos e resumos — não os documentos completos. A intenção foi verificar se o agente conseguiria estruturar uma análise de cobertura a partir de metadados, sem acesso ao conteúdo integral. Optou-se por não direcionar o formato de saída nem indicar o nível de profundidade esperado, para observar o comportamento natural do agente.

### Output gerado

**Painel visual interativo** com:
- 4 métricas de resumo (5 documentos, 8 temas, 7 gaps, 3 conflitos)
- Listagem de documentos com status de confiabilidade
- Mapa de cobertura por tema (coberto / parcial / ausente / conflito)
- 7 hipóteses de gaps com criticidade e impacto no assistente
- 3 conflitos e riscos de qualidade identificados

**Arquivo gerado:** `mapa-temas-gaps-assistente-ia-logistica.md`

Conteúdo do arquivo:
- Resumo executivo em tabela
- Tabela de documentos fonte com status
- Cobertura detalhada por tema (8 temas)
- 7 gaps hipotéticos com criticidade e impacto
- Conflitos C1, C2 e R1
- Tabela de recomendações para o roadmap com responsáveis

### Achados principais desta etapa

| Indicador | Valor |
|---|---|
| Documentos analisados | 5 |
| Temas com cobertura plena | 2 (devolução de mercadorias; SLA e tiers) |
| Temas com cobertura parcial | 4 (seguro, rastreamento, incidentes, descontos) |
| Temas sem cobertura | 1 (cargas perigosas — procedimento completo) |
| Gaps hipotéticos | 7 |
| Conflitos identificados | 3 |
| Perguntas frequentes sem cobertura oficial | 38 de 47 (~81%) |

---

## Etapa 2 — Análise de conflito entre PROC-042 v1 e v2

### Prompt enviado

```
Com relação aos arquivos conflitantes, vou encaminhar os 2 arquivos completos,
analise as inconsistências encontradas

[Arquivos PROC-042-frete-especial-v1.md e PROC-042-v2-frete-especial-revisado.md anexados]
```

### Decisão de contexto

Prompt mínimo, sem especificar formato, profundidade ou quais inconsistências priorizar. O objetivo foi permitir que o agente determinasse a estrutura de análise mais adequada para comparar dois documentos técnicos conflitantes. Os arquivos completos foram fornecidos nesta etapa (na etapa anterior apenas resumos estavam disponíveis).

### Output gerado

**Painel visual interativo** com comparação linha a linha:
- Tabela de metadados dos dois documentos
- Multiplicadores regionais: todas as 5 regiões com variação percentual
- Fatores de peso: 3 faixas com variação
- Prazo adicional de entrega
- Política de descontos por volume (diferenças de processo, não apenas valores)
- Disposição transitória e referências externas
- Simulação de impacto com exemplo numérico concreto (carga 2.000 kg, Norte)
- 4 riscos classificados para o assistente de IA

**Arquivo gerado:** `analise-conflito-PROC-042-v1-vs-v2.md`

Conteúdo do arquivo:
- Tabela de metadados
- 5 seções de divergências identificadas com tabelas comparativas
- Simulação de impacto: diferença de R$ 150,00 (7,8%) e 1 dia útil no prazo
- 5 riscos detalhados para o assistente
- Tabela de recomendações com responsáveis
- Parâmetros consolidados da v2 prontos para indexação

### Achados principais desta etapa

| Parâmetro | v1 (mar/2023) | v2 (nov/2023) | Impacto |
|---|---|---|---|
| Multiplicador Norte | 1,6 | 1,8 | +12,5% no valor do frete |
| Fator de peso 1.001–3.000 kg | 1,2 | 1,15 | −4,2% (redução) |
| Prazo adicional | +2 dias úteis | +3 dias úteis | +1 dia comunicado ao cliente |
| Gatilho de desconto | 10 fretes/mês | 8 fretes/mês | Desconto negado indevidamente |
| Desconto automático | Não previsto | 5% e 10% por faixa | Processo completamente diferente |
| Indicação de obsolescência | Ausente | Ausente | Ambos coexistem sem hierarquia |

**Descoberta adicional:** os fatores de peso moveram-se em direção oposta aos multiplicadores (reduziram na v2), tornando o efeito líquido no valor imprevisível sem calcular caso a caso — o que amplifica o risco de indexação dupla.

---

## Etapa 3 — Cruzamento FAQ × documentos normativos

### Prompt enviado

```
utilizando os 3 arquivos que envio agora, cruze as inconsistências encontradas
com as práticas informais do FAQ

[FAQ-atendimento.md, analise-conflito-PROC-042-v1-vs-v2.md e
mapa-temas-gaps-assistente-ia-logistica.md anexados]
```

### Decisão de contexto

Foram fornecidos simultaneamente o FAQ original e os dois documentos de análise já produzidos nas etapas anteriores. O prompt não especificou quais itens priorizar nem o formato esperado. A intenção foi verificar se o agente utilizaria o contexto acumulado das etapas anteriores para aprofundar a análise — conectando práticas informais a divergências já mapeadas.

### Output gerado

**Painel visual interativo** com análise item a item dos 9 itens do FAQ:
- Para cada item: o que o FAQ diz × o que o normativo diz × inconsistências × recomendação
- Tabela de decisão de indexação por item (indexar / indexar com ressalva / bloquear / reescrever)
- Resumo com contagem por classificação

**Arquivo gerado:** `cruzamento-faq-vs-normativos.md`

Conteúdo do arquivo:
- Resumo executivo com contagem por classificação
- Análise completa dos 9 itens (cada um com: o que diz, o que o normativo diz, inconsistências, recomendação)
- Tabela consolidada de decisão de indexação
- Seção sobre padrão de origem dos conflitos
- Tabela de ações necessárias antes da indexação com responsáveis e impacto se não executadas

### Achados principais desta etapa

| Classificação | Itens | Decisão |
|---|---|---|
| ✅ Alinhado com normativo | 2 (itens 15 e 41) | Indexar diretamente |
| ⚠️ Incompleto / sem respaldo total | 3 (itens 3, 27 e 38) | Indexar com ressalva |
| 🔴 Conflito direto com normativo | 3 (itens 8, 22 e 45) | Bloquear |
| 🟣 Sem respaldo normativo algum | 1 (item 32) | Bloquear |

**Padrão identificado:** Os três conflitos diretos (itens 8, 22 e 45) têm origem comum — o FAQ foi escrito parcialmente com base em versões ou orientações anteriores superadas por documentos mais recentes, especialmente pela PROC-042-v1 obsoleta.

**Caso mais crítico:** Item 32 (carga perigosa com frete expresso) — a especificidade da resposta no FAQ (2 dias de prazo, envolvimento do Compliance, requisitos ANTT) dá aparência de regra estabelecida, mas não tem nenhum respaldo verificável e a PROC-043 referenciada está em revisão.

---

## Consolidado da sessão

### Documentos produzidos

| Arquivo | Etapa | Conteúdo |
|---|---|---|
| `mapa-temas-gaps-assistente-ia-logistica.md` | 1 | Mapa de cobertura, 7 gaps, 3 conflitos, roadmap |
| `analise-conflito-PROC-042-v1-vs-v2.md` | 2 | Comparação técnica v1 vs v2, simulação de impacto, parâmetros consolidados da v2 |
| `cruzamento-faq-vs-normativos.md` | 3 | Análise item a item dos 9 itens do FAQ, decisão de indexação, ações necessárias |
| `registro-sessao-product-specialist-ia-logistica.md` | 4 | Este arquivo |

### Decisões de indexação consolidadas para o assistente

| Documento | Status recomendado | Condição |
|---|---|---|
| POL-001 v3.1 (devolução) | ✅ Indexar | Fonte canônica para devolução |
| PROC-042-v2 (frete especial) | ✅ Indexar | Única versão vigente; exige regra de data para chamados pré-dez/2023 |
| PROC-042-v1 (frete especial) | 🔴 Não indexar | Deprecar formalmente; manter referência apenas para disposição transitória |
| SLA-2024 (tiers e SLA) | ✅ Indexar | Fonte canônica para SLA e tiers |
| FAQ — itens 15 e 41 | ✅ Indexar | Com notas sobre fonte informal |
| FAQ — itens 27 e 38 | ⚠️ Indexar c/ ressalva | Após validação parcial com áreas responsáveis |
| FAQ — itens 8, 22, 45 | 🔴 Não indexar | Substituir pelos normativos corretos |
| FAQ — item 3 | 🔄 Reescrever | Substituir por fluxo oficial da POL-001 |
| FAQ — item 32 | 🔴 Não indexar | Aguardar PROC-043 revisada |
| PROC-043 (cargas perigosas) | ⏳ Aguardar | Em revisão pelo Compliance |

### Gaps que bloqueiam o assistente e ainda precisam de documentação

| Gap | Criticidade | Responsável sugerido |
|---|---|---|
| G1 — Procedimento completo para cargas perigosas | Alta | Operações / Jurídico |
| G2 — Política formal de seguro de carga | Alta | Comercial / Jurídico |
| G3 — Manual técnico de rastreamento | Média-Alta | TI / Operações |
| G4 — Procedimento de escalonamento de incidentes | Média-Alta | Operações |
| G5 — Política comercial e descontos unificada | Média | Comercial |
| G6 — Cadeia de frio e cargas refrigeradas | Média | Operações |
| G7 — 38 perguntas frequentes sem cobertura oficial | Alta | Atendimento + Operações |

---

*Registro gerado em maio/2025. Sessão conduzida com Claude Sonnet (Anthropic) na interface claude.ai.*
