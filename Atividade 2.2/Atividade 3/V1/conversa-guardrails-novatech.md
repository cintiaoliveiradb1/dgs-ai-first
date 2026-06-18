# Conversa — Guardrails Operacionais NovaTech
## Sessão exportada em Junho/2025

---

## Mensagem 1 — Usuário

```
Você está atuando como um AI Governance Specialist responsável por definir regras operacionais para um assistente corporativo.

Contexto
Você receberá:
* A documentação oficial da empresa (Anexo A), que deve ser considerada a única fonte de verdade.
* Guardrails informais já existentes (SPEC).
Considere que o objetivo é transformar regras implícitas em um documento claro, auditável e operacional.

Tarefa
Com base no cenário e na documentação recebida, elabore um documento chamado:
Guardrails Operacionais do Assistente

Organize o resultado nas seguintes seções:
1. DEVE
Liste comportamentos obrigatórios que o assistente precisa seguir.
Para cada guardrail, informe:
* ID do guardrail (ex.: GR-001)
* Nome curto
* Descrição da regra
* Objetivo da regra
* Exemplo de comportamento esperado

2. NÃO DEVE
Liste comportamentos proibidos.
Para cada guardrail, informe:
* ID do guardrail
* Nome curto
* Descrição da proibição
* Risco evitado
* Exemplo de comportamento incorreto

3. QUANDO EM DÚVIDA
Defina comportamentos de fallback para situações de incerteza.
Para cada guardrail, informe:
* ID do guardrail
* Situação de dúvida
* Ação esperada
* Exemplo de resposta ao usuário

Regras de elaboração
* Considere apenas informações presentes na documentação oficial.
* Não invente políticas, processos ou regras não documentadas.
* Transforme princípios vagos em instruções objetivas e verificáveis.
* Escreva guardrails acionáveis (algo que possa ser seguido ou validado).
* Evite duplicidade entre regras.
* Priorize clareza operacional em vez de linguagem genérica.
* Responda em português formal.

Formato de saída obrigatório
1. Resumo executivo (lista dos guardrails criados)
2. Documento completo organizado em:
   * DEVE
   * NÃO DEVE
   * QUANDO EM DÚVIDA
3. Observações finais indicando possíveis ambiguidades ou lacunas identificadas na documentação

Analisando apenas os documentos fornecidos, gere o documento completo em md
```

**Arquivos anexados:** `anexo-a-documentacao-simulada-novatech.md`, `spec-rag-novatech-v1_3.md`

---

## Mensagem 2 — Assistente

Leitura dos documentos `anexo-a-documentacao-simulada-novatech.md` e `spec-rag-novatech-v1_3.md` e geração do documento **Guardrails Operacionais do Assistente v1.0**.

**Arquivo gerado:** `guardrails-operacionais-assistente-novatech.md`

**Resumo dos 20 guardrails criados:**

| Seção | ID | Nome Curto |
|-------|----|------------|
| DEVE | GR-001 | Citar fonte oficial em toda resposta |
| DEVE | GR-002 | Sinalizar fonte informal (FAQ) |
| DEVE | GR-003 | Usar versão vigente do PROC-042 conforme data do chamado |
| DEVE | GR-004 | Sinalizar conflito entre documentos |
| DEVE | GR-005 | Classificar resposta negativa por tipo de gap |
| DEVE | GR-006 | Solicitar dados faltantes antes de calcular frete ou SLA |
| DEVE | GR-007 | Exibir trecho original que embasou a resposta |
| DEVE | GR-008 | Coletar feedback do atendente ao final de cada resposta |
| DEVE | GR-009 | Registrar log completo de cada interação |
| DEVE | GR-010 | Exibir mensagem diferenciada em modo degradado |
| NÃO DEVE | GR-011 | Fabricar ou extrapolar informações não documentadas |
| NÃO DEVE | GR-012 | Usar FAQ informal como fonte primária quando há documento oficial |
| NÃO DEVE | GR-013 | Apresentar informações de gaps documentais como fatos confirmados |
| NÃO DEVE | GR-014 | Confirmar tiers inexistentes ou políticas não documentadas |
| NÃO DEVE | GR-015 | Usar trecho com score de similaridade abaixo de 0,75 |
| NÃO DEVE | GR-016 | Aplicar multiplicadores ou fatores do PROC-042 sem verificar a versão correta |
| QUANDO EM DÚVIDA | GR-017 | Ausência de data do chamado em consulta de frete especial |
| QUANDO EM DÚVIDA | GR-018 | Conflito entre FAQ e documento oficial na janela de atualização |
| QUANDO EM DÚVIDA | GR-019 | Categoria de carga não elegível para devolução padrão |
| QUANDO EM DÚVIDA | GR-020 | Tema reconhecido como gap documental |

**Observações finais registradas:**
- AMB-01: Status formal do PROC-042-v1 não arquivado
- AMB-02: Fator de peso divergente entre versões não mencionado na SPEC
- AMB-03: Carga perigosa com frete expresso sem PROC formal
- 4 lacunas documentais: carga danificada, seguro de carga, frete padrão abaixo de 500kg, processo da Gestão de Riscos

---

## Mensagem 3 — Usuário

````markdown
Para cada guardrail encontrado, classifique como: enforcement via prompt (probabilístico) ou enforcement via código (determinístico). para cada classificação inclua uma justificativa
````

---

## Mensagem 4 — Assistente

Leitura do documento `guardrails-operacionais-assistente-novatech.md` e geração do documento **Classificação de Enforcement dos Guardrails Operacionais v1.0**.

**Arquivo gerado:** `guardrails-classificacao-enforcement.md`

**Critério de classificação:**

| Tipo | Definição |
|------|-----------|
| **Determinístico (via código)** | Regra aplicada por lógica programada no pipeline — filtros, metadados, thresholds numéricos, bloqueios condicionais. Garantida independentemente do que o modelo produziria. |
| **Probabilístico (via prompt)** | Regra comunicada ao LLM por instrução em prompt. Cumprimento depende do comportamento emergente do modelo. |

**Tabela resumo de classificações:**

| ID | Nome Curto | Enforcement | Confiabilidade |
|----|-----------|-------------|----------------|
| GR-001 | Citar fonte oficial em toda resposta | Híbrido — primariamente **Probabilístico** | Média |
| GR-002 | Sinalizar fonte informal (FAQ) | **Determinístico** | Alta |
| GR-003 | Usar versão vigente do PROC-042 conforme data do chamado | **Determinístico** | Alta |
| GR-004 | Sinalizar conflito entre documentos | Híbrido — primariamente **Determinístico** | Alta |
| GR-005 | Classificar resposta negativa por tipo de gap | Híbrido — primariamente **Determinístico** | Média-Alta |
| GR-006 | Solicitar dados faltantes antes de calcular frete ou SLA | Híbrido — primariamente **Determinístico** | Alta |
| GR-007 | Exibir trecho original que embasou a resposta | **Determinístico** | Alta |
| GR-008 | Coletar feedback do atendente ao final de cada resposta | **Determinístico** | Alta |
| GR-009 | Registrar log completo de cada interação | **Determinístico** | Alta |
| GR-010 | Exibir mensagem diferenciada em modo degradado | **Determinístico** | Alta |
| GR-011 | Fabricar ou extrapolar informações não documentadas | **Probabilístico** | Baixa-Média |
| GR-012 | Usar FAQ informal como fonte primária quando há documento oficial | Híbrido — primariamente **Determinístico** | Alta |
| GR-013 | Apresentar informações de gaps documentais como fatos confirmados | **Probabilístico** | Baixa-Média |
| GR-014 | Confirmar tiers inexistentes ou políticas não documentadas | **Probabilístico** | Baixa-Média |
| GR-015 | Usar trecho com score de similaridade abaixo de 0,75 | **Determinístico** | Alta |
| GR-016 | Aplicar multiplicadores do PROC-042 sem verificar a versão correta | **Determinístico** | Alta |
| GR-017 | Ausência de data do chamado em consulta de frete especial | **Determinístico** | Alta |
| GR-018 | Conflito entre FAQ e documento oficial na janela de atualização | **Determinístico** | Alta |
| GR-019 | Categoria de carga não elegível para devolução padrão | **Probabilístico** | Baixa-Média |
| GR-020 | Tema reconhecido como gap documental | Híbrido — primariamente **Determinístico** | Alta |

**Distribuição final:**
- 10 Determinísticos: GR-002, GR-003, GR-007, GR-008, GR-009, GR-010, GR-015, GR-016, GR-017, GR-018
- 5 Probabilísticos: GR-001, GR-011, GR-013, GR-014, GR-019
- 5 Híbridos: GR-004, GR-005, GR-006, GR-012, GR-020

**3 recomendações prioritárias antes do go-live:**
1. Output parser pós-geração para GR-001
2. Filtro de pré-composição para GR-013 (excluir FAQ de temas com gap total)
3. Lista de entidades controladas para GR-014 (tiers válidos injetados como contexto obrigatório)

---

## Mensagem 5 — Usuário

```
considere os guardrails criados e o arquivo anexo com incidentes onde o agente falhou, conecte cada guardrail a ao menos um dos 3 incidentes (qual incidente esse guardrail previne?)
```

**Arquivo anexado:** `incidentesAgent.md`

**Conteúdo do arquivo de incidentes:**
> 3 incidentes simulados onde o assistente falhou durante testes internos:
> 1. *"O assistente respondeu que o prazo de devolução para carga perigosa é 7 dias, quando na verdade cargas perigosas NÃO podem ser devolvidas."*
> 2. *"O assistente citou 'PROC-042, seção 2' mas os multiplicadores informados eram da versão 1 (desatualizada), não da v2 (vigente)."*
> 3. *"O assistente disse 'Não encontrei informação sobre isso' para uma pergunta sobre SLA Gold, mas o documento SLA-2024 estava indexado e continha a resposta."*

---

## Mensagem 6 — Assistente

Leitura dos documentos anteriores e do arquivo de incidentes. Geração do documento **Mapeamento de Guardrails × Incidentes v1.0**.

**Arquivo gerado:** `guardrails-mapeamento-incidentes.md`

### Análise de causa-raiz dos incidentes

**INC-01 — Prazo incorreto para carga perigosa**
Causa-raiz primária: modelo generalizou a regra geral de 7 dias (POL-001, seção 3.1) sem aplicar a exceção da seção 3.2 que exclui cargas perigosas do processo padrão. Padrão de recuperação parcial ou geração sem ancoragem nos chunks completos.

**INC-02 — Multiplicadores da versão errada do PROC-042**
Causa-raiz primária: pipeline não filtrou chunks por versão vigente antes de enviá-los ao LLM — ou o LLM recebeu chunks de ambas as versões e utilizou os da v1 sem detectar conflito.

**INC-03 — Falso negativo para pergunta coberta pela base**
Causa-raiz primária: falha no retrieval — assistente não recuperou chunks relevantes do SLA-2024 apesar de o documento estar indexado. Causas possíveis: threshold mal calibrado ou chunking inadequado de tabelas.

### Tabela de mapeamento

| ID | Nome do Guardrail | INC-01 | INC-02 | INC-03 |
|----|-------------------|--------|--------|--------|
| GR-001 | Citar fonte oficial em toda resposta | Indireto | **Direto** | Indireto |
| GR-002 | Sinalizar fonte informal (FAQ) | — | — | — |
| GR-003 | Usar versão vigente do PROC-042 conforme data do chamado | — | **Direto** | — |
| GR-004 | Sinalizar conflito entre documentos | — | **Direto** | — |
| GR-005 | Classificar resposta negativa por tipo de gap | — | — | **Direto** |
| GR-006 | Solicitar dados faltantes antes de calcular frete ou SLA | — | **Direto** | — |
| GR-007 | Exibir trecho original que embasou a resposta | Indireto | Indireto | Indireto |
| GR-008 | Coletar feedback do atendente ao final de cada resposta | Indireto | Indireto | Indireto |
| GR-009 | Registrar log completo de cada interação | Indireto | Indireto | Indireto |
| GR-010 | Exibir mensagem diferenciada em modo degradado | — | — | — |
| GR-011 | Não fabricar ou extrapolar informações não documentadas | **Direto** | — | — |
| GR-012 | Não usar FAQ informal como fonte primária | — | — | — |
| GR-013 | Não apresentar gaps documentais como fatos confirmados | — | — | — |
| GR-014 | Não confirmar tiers inexistentes | — | — | — |
| GR-015 | Usar trecho com score de similaridade ≥ 0,75 | — | — | **Direto** |
| GR-016 | Não aplicar multiplicadores do PROC-042 sem verificar versão | — | **Direto** | — |
| GR-017 | Solicitar data do chamado em consulta de frete especial | — | **Direto** | — |
| GR-018 | Descartar FAQ na janela de atualização | — | — | — |
| GR-019 | Reconhecer carga não elegível para devolução padrão | **Direto** | — | — |
| GR-020 | Reconhecer gap documental | — | — | **Direto** |

### Visão consolidada por incidente

**INC-01** — 2 diretos (GR-011, GR-019) + 4 indiretos (GR-001, GR-007, GR-008, GR-009)

**INC-02** — 5 diretos (GR-003, GR-004, GR-006, GR-016, GR-017) + 4 indiretos (GR-001, GR-007, GR-008, GR-009)

**INC-03** — 3 diretos (GR-005, GR-015, GR-020) + 4 indiretos (GR-001, GR-007, GR-008, GR-009)

### Guardrails sem cobertura nos três incidentes
GR-002, GR-010, GR-012, GR-013, GR-014, GR-018 — protegem contra classes de falha distintas não representadas nesta amostra.

### Lacuna identificada
O INC-03 expõe causa-raiz estrutural (chunking inadequado de tabelas) não coberta por nenhum dos 20 guardrails. Recomendação: criar **GR-021** vinculado à resolução da DTP-001 (estratégia de chunking de tabelas numéricas).

---

## Mensagem 7 — Usuário

```
exporte nossa interação nesse chat para um arquivo md
```

---

## Mensagem 8 — Assistente

Exportação da conversa completa para este arquivo.

---

## Artefatos produzidos na sessão

| Arquivo | Descrição |
|---------|-----------|
| `guardrails-operacionais-assistente-novatech.md` | Documento completo com 20 guardrails (DEVE / NÃO DEVE / QUANDO EM DÚVIDA), observações finais e lacunas |
| `guardrails-classificacao-enforcement.md` | Classificação de cada guardrail como determinístico, probabilístico ou híbrido, com justificativas e recomendações prioritárias |
| `guardrails-mapeamento-incidentes.md` | Mapeamento de cada guardrail aos 3 incidentes internos, com análise de causa-raiz, cadeias de falha e identificação de lacuna estrutural (GR-021 sugerido) |
| `conversa-guardrails-novatech.md` | Este arquivo — exportação da sessão completa |

---

## Documentos de entrada utilizados

| Arquivo | Tipo | Descrição |
|---------|------|-----------|
| `anexo-a-documentacao-simulada-novatech.md` | Fonte de verdade | POL-001 v3.1, PROC-042-v1, PROC-042-v2, SLA-2024 v2024.1, FAQ-Atendimento |
| `spec-rag-novatech-v1_3.md` | Especificação técnica | Requisitos do pipeline RAG, fontes, comportamentos, rastreabilidade, atualizações |
| `incidentesAgent.md` | Casos de falha | 3 incidentes simulados durante testes internos do assistente |

---

*NovaTech Logística — Exportação de sessão — Junho/2025*
