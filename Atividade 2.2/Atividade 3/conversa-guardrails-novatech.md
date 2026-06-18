# Transcrição da Conversa — Projeto Guardrails NovaTech
**Data:** Junho/2025
**Contexto:** Definição de guardrails operacionais para o Assistente de Atendimento IA da NovaTech Logística

---

## Arquivos de entrada

- `anexo-a-documentacao-simulada-novatech.md` — Documentação oficial da NovaTech (POL-001, PROC-042-v1, PROC-042-v2, SLA-2024, FAQ-Atendimento)
- `spec-rag-novatech-v1_3.md` — Especificação técnica do pipeline RAG (SPEC v1.3)
- `incidentesAgent.md` — Registro de 3 incidentes simulados durante testes internos

## Arquivos gerados

| Versão | Arquivo | Descrição |
|--------|---------|-----------|
| v1.0 | `guardrails-operacionais-assistente-novatech.md` | Documento base com 25 guardrails nas seções DEVE, NÃO DEVE e QUANDO EM DÚVIDA |
| — | `classificacao-enforcement-guardrails-novatech.md` | Documento separado com classificação de enforcement de cada guardrail |
| v2.0 | `guardrails-operacionais-assistente-novatech-v2.md` | Enforcement integrado ao corpo de cada guardrail |
| v3.0 | `guardrails-operacionais-assistente-novatech-v3.md` | Campo "Incidente Relacionado" integrado ao corpo de cada guardrail |

---

## Turno 1

### Usuário

> Você está atuando como um AI Governance Specialist responsável por definir regras operacionais para um assistente corporativo.
>
> **Contexto:** Você receberá a documentação oficial da empresa (Anexo A), que deve ser considerada a única fonte de verdade, e guardrails informais já existentes (SPEC). O objetivo é transformar regras implícitas em um documento claro, auditável e operacional.
>
> **Tarefa:** Com base no cenário e na documentação recebida, elabore um documento chamado **Guardrails Operacionais do Assistente**, organizado nas seguintes seções:
>
> - **DEVE** — comportamentos obrigatórios, com: ID, nome curto, descrição, objetivo, exemplo de comportamento esperado
> - **NÃO DEVE** — comportamentos proibidos, com: ID, nome curto, descrição da proibição, risco evitado, exemplo de comportamento incorreto
> - **QUANDO EM DÚVIDA** — comportamentos de fallback, com: ID, situação de dúvida, ação esperada, exemplo de resposta ao usuário
>
> **Regras de elaboração:** considere apenas informações presentes na documentação oficial; não invente políticas; transforme princípios vagos em instruções objetivas e verificáveis; escreva guardrails acionáveis; evite duplicidade; priorize clareza operacional.
>
> **Formato de saída obrigatório:** Resumo executivo + documento completo nas três seções + observações finais com ambiguidades e lacunas.
>
> Analisando apenas os documentos fornecidos, gere o documento completo em md.

### Assistente

Leitura dos dois documentos (Anexo A e SPEC v1.3) e geração do documento `guardrails-operacionais-assistente-novatech.md` (v1.0) com 25 guardrails distribuídos em três seções:

**DEVE (11 guardrails)**

| ID | Nome |
|----|------|
| GR-D-001 | Citar fonte em toda resposta |
| GR-D-002 | Priorizar documentos com status vigente |
| GR-D-003 | Sinalizar conflito entre versões |
| GR-D-004 | Sinalizar fontes informais |
| GR-D-005 | Aplicar versão do PROC-042 conforme data do chamado |
| GR-D-006 | Identificar incidentes críticos conforme SLA-2024 |
| GR-D-007 | Respeitar SLAs por tier de cliente |
| GR-D-008 | Exibir trecho original que embasou a resposta |
| GR-D-009 | Registrar log de cada interação |
| GR-D-010 | Coletar feedback do atendente |
| GR-D-011 | Distinguir gap documental de fora do escopo |

**NÃO DEVE (8 guardrails)**

| ID | Nome |
|----|------|
| GR-N-001 | Fabricar ou extrapolar informações |
| GR-N-002 | Usar documento com status obsoleto para chamados novos |
| GR-N-003 | Confirmar tiers inexistentes |
| GR-N-004 | Apresentar fonte informal como oficial |
| GR-N-005 | Autorizar descontos sem respaldo documental |
| GR-N-006 | Dar orientação definitiva sobre temas com gaps documentais |
| GR-N-007 | Usar trecho abaixo do limiar de similaridade |
| GR-N-008 | Expor dados de log a perfis não autorizados |

**QUANDO EM DÚVIDA (6 guardrails)**

| ID | Nome |
|----|------|
| GR-W-001 | Versão do PROC-042 sem data do chamado |
| GR-W-002 | Dados insuficientes para cálculo de frete |
| GR-W-003 | Tema presente apenas no FAQ informal |
| GR-W-004 | Tema com gap total na base |
| GR-W-005 | Assistente indisponível |
| GR-W-006 | FAQ conflita com documento primário recém-atualizado |

**Observações finais identificadas:** 7 ambiguidades/lacunas, incluindo a coexistência sem hierarquia do PROC-042-v1 e v2, os quatro gaps documentais sem POL/PROC formal, e as duas decisões técnicas pendentes (DTP-001 e DTP-002).

---

## Turno 2

### Usuário

> Para cada guardrail, classifique como: enforcement via prompt (probabilístico) ou enforcement via código (determinístico), para cada classificação, faça uma justificativa.

### Assistente

Geração do documento `classificacao-enforcement-guardrails-novatech.md` com a classificação de todos os 25 guardrails.

**Critérios estabelecidos:**

| Classificação | Significado |
|---------------|-------------|
| **Código — Determinístico** | Implementado fora do LLM: pipeline, banco vetorial, sistema de autenticação, interface, logger. Comportamento garantido independentemente do que o modelo gerar. |
| **Prompt — Probabilístico** | Instrução no system prompt ou contexto. Comportamento estatístico — pode desviar sob entradas adversariais. |
| **Misto — Código primário** | Pipeline garante a pré-condição; LLM formata e comunica o resultado. |
| **Misto — Prompt primário** | Classificação semântica feita pelo LLM; código fornece critérios fixos no contexto. |

**Distribuição final:**

| Tipo | Qtd | Guardrails |
|------|-----|-----------|
| Código — Determinístico | 9 | GR-D-002, GR-D-005, GR-D-009, GR-D-010, GR-N-002, GR-N-007, GR-N-008, GR-W-004, GR-W-005 |
| Prompt — Probabilístico | 5 | GR-D-001, GR-N-001, GR-N-003, GR-N-005, (camada semântica de GR-D-011) |
| Misto — Código primário | 9 | GR-D-003, GR-D-004, GR-D-007, GR-D-008, GR-N-004, GR-N-006, GR-W-001, GR-W-002, GR-W-003, GR-W-006 |
| Misto — Prompt primário | 2 | GR-D-006, GR-D-011 |

**Padrões identificados:**
- Os 5 guardrails de prompt puro envolvem julgamento semântico sobre saída ou entrada em linguagem natural.
- GR-N-001 (não fabricar informações) é o mais crítico semanticamente e o menos garantível tecnicamente — candidato a output filtering pós-geração.
- Divisão recorrente nos casos mistos: o código decide *o quê* (detecta, filtra, bloqueia); o LLM decide *como* (formula e comunica).

---

## Turno 3

### Usuário

> Preciso que continue no formato DEVE, NÃO DEVE e QUANDO EM DÚVIDA. É possível adicionar essa informação no formato do documento guardrails operacionais gerado anteriormente?

### Assistente

Geração da versão `guardrails-operacionais-assistente-novatech-v2.md` (v2.0), incorporando o enforcement diretamente no corpo de cada guardrail como campo adicional, após o exemplo de comportamento.

**Estrutura de cada guardrail na v2.0:**

*DEVE:*
- Descrição da regra
- Objetivo da regra
- Exemplo de comportamento esperado
- **Enforcement** *(novo)*
- Referências

*NÃO DEVE:*
- Descrição da proibição
- Risco evitado
- Exemplo de comportamento incorreto
- **Enforcement** *(novo)*
- Referências

*QUANDO EM DÚVIDA:*
- Situação de dúvida
- Ação esperada
- Exemplo de resposta ao usuário
- **Enforcement** *(novo)*
- Referências

O resumo executivo foi atualizado com coluna de enforcement e foi adicionada seção de critérios de classificação no início do documento.

---

## Turno 4

### Usuário

> Seguindo esse mesmo padrão, considere os guardrails criados e o arquivo anexo com incidentes onde o agente falhou, conecte cada guardrail a ao menos um dos 3 incidentes (qual incidente esse guardrail previne?).
>
> **Incidentes registrados:**
> - **INC-01:** O assistente respondeu que o prazo de devolução para carga perigosa é 7 dias, quando na verdade cargas perigosas NÃO podem ser devolvidas.
> - **INC-02:** O assistente citou "PROC-042, seção 2" mas os multiplicadores informados eram da versão 1 (desatualizada), não da v2 (vigente).
> - **INC-03:** O assistente disse "Não encontrei informação sobre isso" para uma pergunta sobre SLA Gold, mas o documento SLA-2024 estava indexado e continha a resposta.

### Assistente

Geração da versão `guardrails-operacionais-assistente-novatech-v3.md` (v3.0), incorporando o campo **"Incidente Relacionado"** no corpo de cada guardrail, com vínculo classificado como: **causa raiz**, **fator contribuinte** ou **detecção/mitigação**.

**Mapa de cobertura por incidente:**

**INC-01** — Erro de extrapolação (carga perigosa / prazo de 7 dias)
Guardrails conectados: GR-D-001 (detecção), GR-D-004 (contribuinte), GR-D-008 (detecção), GR-D-009 (detecção), GR-D-010 (detecção), GR-N-001 ***(causa raiz)***, GR-N-004 (contribuinte), GR-N-005 (contribuinte), GR-N-006 (contribuinte), GR-W-003 (contribuinte), GR-W-006 (contribuinte)

**INC-02** — Erro de versão (multiplicadores da v1 em vez da v2)
Guardrails conectados: GR-D-001 (detecção), GR-D-003 (contribuinte), GR-D-005 ***(causa raiz)***, GR-D-008 (detecção), GR-D-009 (detecção), GR-D-010 (detecção), GR-N-002 ***(causa raiz)***, GR-D-002 ***(causa raiz)***, GR-W-001 (contribuinte), GR-W-002 (contribuinte), GR-W-006 (contribuinte)

**INC-03** — Falso negativo (SLA Gold disponível mas não encontrado)
Guardrails conectados: GR-D-006 (contribuinte), GR-D-007 ***(causa raiz)***, GR-D-009 (detecção), GR-D-010 (detecção), GR-D-011 ***(causa raiz)***, GR-N-007 ***(causa raiz)***, GR-W-004 ***(causa raiz)***, GR-W-005 (contribuinte)

**Análise consolidada:**

- INC-01 é o **erro composto** com mais guardrails conectados (11): cadeia de falhas — FAQ informal sem aviso + regra geral aplicada sem verificar exceção + sem trecho original exibido.
- INC-02 é o **mais corrigível por código**: 4 dos 5 guardrails conectados como causa raiz ou contribuinte são determinísticos. Com GR-D-002, GR-D-005 e GR-N-002 implementados, INC-02 não pode se repetir independentemente do LLM.
- INC-03 é o **mais técnico**: causa raiz aponta para DTP-001 (chunking de tabelas) e DTP-002 (modelo LLM) — ambas decisões técnicas pendentes na SPEC v1.3.

---

## Turno 5

### Usuário

> Exporte nossa interação nesse chat para arquivo md.

### Assistente

Geração do presente arquivo: `conversa-guardrails-novatech.md`.

---

*Exportado em Junho/2025 — NovaTech Logística*
