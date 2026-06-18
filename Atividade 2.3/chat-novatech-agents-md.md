# Sessão de Trabalho — NovaTech Assistant · AGENTS.md
**Data:** 15 de junho de 2026  
**Objetivo:** Produção da seção _Product Rules & Guardrails_ do `AGENTS.md` do projeto NovaTech Assistant.

---

## Contexto

Projeto de assistente de IA para empresa de logística (NovaTech). O assistente auxilia atendentes apresentando respostas para questionamentos que hoje exigem consulta manual a documentos dispersos. A fase de discovery está concluída.

**Artefatos de entrada fornecidos:**

| Arquivo | Descrição |
|---------|-----------|
| `anexo-a-documentacao-simulada-novatech.md` | Documentação oficial NovaTech (POL-001, PROC-042-v1, PROC-042-v2, SLA-2024, FAQ-Atendimento) |
| `guardrails-operacionais-assistente-novatech-v3.md` | Guardrails operacionais v3.0 com 26 regras, enforcement e rastreabilidade por incidente |
| `linguagem-ubiqua-anexo-a.md` | Glossário de domínio com conflitos, gaps e sinônimos proibidos |
| `anexo-c-estrutura-repositorio.md` | Estrutura do repositório `novatech-assistant` |
| `estrutura.md` | Estrutura de seções do `AGENTS.md` |

---

## Turno 1 — Briefing do projeto

**Usuário:**

> Estamos criando um assistente de IA para empresa de logística. Assistente deve auxiliar atendentes apresentando respostas para questionamentos que hoje só são respondidos após consulta de documentações distribuídas em alguns documentos, o que dificulta a consulta.
> O discovery está concluído e a fase de entendimento produziu artefatos concretos: ADRs com decisões arquiteturais (modelo LLM, estratégia de contexto, tratamento de documentos contraditórios, build vs buy), uma spec de requisitos de produto para o pipeline de RAG, um protótipo funcional de RAG com ferramentas open-source, cenários de falha mapeados pelo QA, e um plano de testes inicial.
> Você vai receber:
> - A documentação da NovaTech (ver **Anexo A**) e a estrutura do repositório (ver **Anexo C**).
> - A estrutura do AGENTS.md (estrutura.md).
> - Documento com linguagem ubíqua identificada (linguagem-ubiqua-anexo-a.md)
> - Guardrails (guardrails-operacionais-assistente-novatech-v3.md) e da documentação oficial da NovaTech.
> -- Incluindo regras sobre:
>   - citação de fonte com identificador de documento e seção
>   - obrigatoriedade do campo source_document no JSON
>   - idioma e tom formal em português
>   - proibição de inventar números, SLAs, multiplicadores ou tiers
>   - tratamento de carga perigosa
>   - priorização da versão mais recente de documentos quando houver versões conflitantes
>   - comportamento de baixa confiança e escalonamento ao supervisor

**Assistente:**

Leitura e análise de todos os arquivos enviados:
- `anexo-a-documentacao-simulada-novatech.md` — 5 documentos-fonte com contradições identificadas (PROC-042-v1 vs v2), gaps documentais (carga danificada, seguro, frete padrão) e documento informal (FAQ).
- `guardrails-operacionais-assistente-novatech-v3.md` — 26 guardrails em 3 categorias (DEVE / NÃO DEVE / QUANDO EM DÚVIDA), com 3 incidentes de referência (INC-01, INC-02, INC-03), classificação de enforcement (Código / Prompt / Misto) e rastreabilidade por incidente.
- `linguagem-ubiqua-anexo-a.md` — 30+ termos com conflitos de valores entre v1 e v2, sinônimos proibidos e notas de desambiguação.
- `anexo-c-estrutura-repositorio.md` — Árvore do repositório com caminhos para `src/`, `specs/`, `skills/`, `prompts/`, `docs/`.
- `estrutura.md` — Seções do AGENTS.md: Project Overview, Tech Stack, Coding Standards, **Product Rules & Guardrails**, Testing Standards, Project Management Rules, Build & Deploy.

---

## Turno 2 — Solicitação de produção do artefato

**Usuário:**

> Utilizando os arquivos informados acima, escreva a seção **"Product Rules & Guardrails"** do AGENTS.md. Ela deve conter:
> 1. Regras de comportamento do assistente (derivadas dos guardrails simulados acima).
> 2. Glossário de linguagem ubíqua do domínio que os agentes precisam conhecer (ex: "cliente Gold", "carga perigosa", "SLA de resolução", "multiplicador regional", "frete especial").
> 3. Restrições que impactam geração de código (ex: "toda resposta DEVE incluir o campo `source_document` no JSON de retorno").
> 4. Referências a documentos de spec no repositório.

**Assistente:**

Produção do arquivo `AGENTS.md` com a seção completa. Estrutura gerada:

---

### Seção produzida: `Product Rules & Guardrails`

```
AGENTS.md — NovaTech Assistant
└── Product Rules & Guardrails
    ├── 1. Visão Geral do Produto
    │   └── Tabela dos 5 documentos-fonte (ID, versão, tipo)
    ├── 2. Regras de Comportamento do Assistente
    │   ├── 2.1. Regras DEVE (11 guardrails: GR-D-001 a GR-D-011)
    │   ├── 2.2. Regras NÃO DEVE (8 guardrails: GR-N-001 a GR-N-008)
    │   ├── 2.3. Regras QUANDO EM DÚVIDA (6 guardrails: GR-W-001 a GR-W-006)
    │   └── 2.4. Regras Especiais para Carga Perigosa (consolidado)
    ├── 3. Restrições que Impactam Geração de Código
    │   ├── 3.1. Schema obrigatório do JSON de resposta (TypeScript)
    │   ├── 3.2. Validação de entrada (schema Zod)
    │   ├── 3.3. Constantes de configuração (src/shared/config.ts)
    │   ├── 3.4. Metadados obrigatórios na ingestão (tabela)
    │   ├── 3.5. Idioma e tom
    │   └── 3.6. Tratamento de erros (matriz de cenários)
    ├── 4. Glossário de Linguagem Ubíqua (30 termos)
    └── 5. Referências a Documentos de Spec no Repositório
        ├── Tabela de artefatos e caminhos no repo
        └── Conexão com incidentes de referência (INC-01, INC-02, INC-03)
```

**Decisões editoriais:**

- Cada guardrail recebe um bloco `> Impacto em código` indicando o arquivo exato do repositório responsável pela implementação.
- Campos obrigatórios do JSON (`source_document`, `source_excerpt`, `confidence_score`, `response_type`, `request_id`) são tipados em TypeScript com rastreabilidade ao guardrail de origem.
- Constantes como `SIMILARITY_THRESHOLD = 0.75` e `PROC042_CUTOFF_DATE = '2023-12-01'` são centralizadas em `src/shared/config.ts` e nunca hard-coded.
- Termos com conflito entre versões de documento (ex: `multiplicador_regional`) incluem tabela de desambiguação embutida no glossário.
- A seção 5 conecta cada guardrail a um caminho de spec (`specs/pipeline-ingestao/requirements.md`, `specs/query-endpoint/requirements.md`) e às golden queries de avaliação.
- A matriz de incidentes (seção 5.1) indica para cada INC o guardrail primário, a causa raiz e o caminho de teste.

**Arquivo entregue:** `AGENTS.md`

---

## Turno 3 — Exportação da sessão

**Usuário:**

> Exporte nossa interação nesse chat para um arquivo .md

**Assistente:**

Geração deste arquivo: `chat-novatech-agents-md.md`

---

## Artefatos Produzidos

| Arquivo | Caminho de saída | Descrição |
|---------|-----------------|-----------|
| `AGENTS.md` | `/mnt/user-data/outputs/AGENTS.md` | Seção _Product Rules & Guardrails_ completa, pronta para integrar ao repositório `novatech-assistant` |
| `chat-novatech-agents-md.md` | `/mnt/user-data/outputs/chat-novatech-agents-md.md` | Este arquivo — registro da sessão de trabalho |

---

## Notas de Contexto

**Incidentes de referência que motivaram os guardrails:**

| ID | Descrição | Guardrail(s) primário(s) |
|----|-----------|--------------------------|
| INC-01 | Assistente respondeu que prazo de devolução para carga perigosa é 7 dias (regra geral aplicada indevidamente a caso de exceção explícita) | GR-N-001, GR-D-004, GR-W-003 |
| INC-02 | Assistente citou multiplicadores da PROC-042-v1 desatualizada para chamado pós-01/12/2023 | GR-D-002, GR-D-005, GR-N-002 |
| INC-03 | Assistente respondeu "Não encontrei informação" para SLA Gold quando documento estava indexado | GR-D-007, GR-D-011, GR-N-007 |

**Conflitos documentais identificados e tratados:**

| Conflito | Entre | Regra de desambiguação |
|----------|-------|------------------------|
| Multiplicadores regionais | PROC-042-v1 vs v2 | Data do chamado: até 30/11/2023 → v1; a partir de 01/12/2023 → v2 |
| Fatores de peso | PROC-042-v1 vs v2 | Mesma regra de data |
| Prazo adicional de frete especial | v1: +2 dias úteis vs v2: +3 dias úteis | Mesma regra de data |
| Limiar de desconto por volume | FAQ (>10/mês) vs PROC-042-v2 (≥8/mês) | Para chamados vigentes, v2 prevalece |
| Limiar de incidente crítico por valor | FAQ-Item 27 (R$50k) vs SLA-2024 (R$100k) | SLA-2024 é o documento formal; FAQ ignorado para este fim |

**Gaps documentais sem cobertura formal:**

- Política de carga danificada em trânsito (apenas FAQ-Item 38)
- Seguro de carga (apenas FAQ-Item 22)
- Frete padrão abaixo de 500 kg (nenhum documento)
- Processo formal da Gestão de Riscos para cargas perigosas devolvidas (apenas ramal 4500 na POL-001)

---

*Exportação gerada em 15/06/2026*

---

## Turno 4 — Revisão do AGENTS.md

**Usuário:**

> Revise o AGENTS.md à procura de inconsistências ou ambiguidades.

**Assistente:**

Leitura completa do arquivo `/mnt/user-data/outputs/AGENTS.md` (743 linhas). Identificados e corrigidos **12 problemas** distribuídos em três categorias de severidade.

---

### Problemas críticos — gerariam bugs em runtime

**Problema 1 · `status` enum continha `'informal'`** (seção 3.4 — metadados de ingestão)

`informal` é valor do campo `tipo`, não de `status`. O campo `status` só admite `vigente` / `obsoleto`. Misturar os dois quebra os guardrails GR-D-002 (re-ranking por status) e GR-N-002 (filtro de obsoletos), pois o pipeline buscaria chunks com `status=informal` — valor inexistente no índice.

*Correção:* removido `informal` do enum de `status`. A linha agora lê `vigente` / `obsoleto` apenas.

---

**Problema 2 · `source_excerpt` era `string` singular**

GR-D-001 exige atribuição individual por afirmação quando a resposta derivar de múltiplos documentos. Um campo `string` singular colapsaria todos os trechos num único bloco, tornando impossível rastrear qual trecho embasou qual afirmação.

*Correção:* campo alterado para `SourceExcerpt[]`. Nova interface adicionada:

```typescript
interface SourceExcerpt {
  source_ref_document_id: string; // Vincula ao document_id do SourceRef correspondente
  excerpt_text: string;           // Trecho ≤ 150 palavras; senão, resumo + trecho completo
}
```

---

**Problema 3 · `ResponseType` tinha `'gap'` genérico**

GR-D-011 exige distinção **obrigatória** entre gap documental (tema NovaTech sem cobertura) e fora do escopo (tema externo ao domínio). Um único valor `'gap'` impossibilitava essa distinção programaticamente — o campo `gap_type` tornava-se redundante e o comportamento do pipeline, ambíguo.

*Correção:* `'gap'` substituído por `'gap_documental'` e `'out_of_scope'` como valores distintos no enum. Tabela de tratamento de erros (seção 3.6) atualizada para refletir os dois cenários separados.

---

**Problema 4 · GR-W-006 comparava `ingestion_date` em vez de data do conteúdo**

O critério de "documento primário mais recente que o FAQ" deve ser baseado em `emissao_date` / `last_updated` do conteúdo do documento-fonte — não em `ingestion_date`, que é um metadado operacional de quando o arquivo foi processado pelo pipeline. Usar `ingestion_date` criaria falsos positivos: se o FAQ fosse reingerido (ex.: após correção de encoding) depois de um documento primário mais antigo, o pipeline o descartaria indevidamente.

*Correção:* critério alterado para `emissao_date`/`last_updated`. Adicionada nota explicativa no `Impacto em código` do GR-W-006.

---

### Problemas importantes — gerariam comportamento ambíguo

**Problema 5 · Relógio de SLA para Silver/Standard em incidentes críticos**

SLA-2024, seção 5, especifica a regra de não-pausa do relógio **exclusivamente para clientes Gold**. A versão anterior generalizava silenciosamente para todos os tiers em GR-D-007, no glossário de `Horário comercial` e no glossário de `Relógio de SLA`. Um agente lendo o documento assumiria que Silver/Standard também têm relógio contínuo em incidentes críticos — comprometimento não documentado.

*Correção:* os três locais foram atualizados. Adicionada instrução de postura conservadora (tratar como sem pausa) e escalação ao supervisor quando o tier for Silver/Standard em incidente crítico.

---

**Problema 6 · `call_date` como `.optional()` no Zod sem delimitação de responsabilidades**

O schema Zod declarava `call_date` como opcional globalmente, mas GR-D-005 o torna funcionalmente obrigatório para consultas de frete especial. A ausência de nota sobre onde ocorre a validação condicional deixava ambíguo se a obrigatoriedade estava no schema ou na camada de negócio.

*Correção:* comentário reescrito para deixar explícito que a validação condicional ocorre em `handler.ts`, não no schema Zod.

---

**Problema 7 · GR-N-005 — "Acima de 10%" lido como terceiro limiar de volume**

A frase original poderia ser interpretada como: volume > 10 fretes/mês → aprovação da Diretoria. Na verdade, refere-se ao **percentual de desconto** (>10% de desconto requer aprovação), independentemente do volume. A ambiguidade era agravada pelo FAQ-Item 45 que usa "mais de 10 fretes/mês" como limiar — confundindo os dois critérios.

*Correção:* reestruturado em três bullet points com distinção clara: (a) desconto automático por volume ≥8; (b) desconto automático por volume ≥15; (c) qualquer desconto acima de 10% de percentual → aprovação da Diretoria.

---

**Problema 8 · PROC-043 não listada nos gaps catalogados em GR-D-011**

A seção 2.4 (carga perigosa) mencionava PROC-043 em revisão pelo Compliance, mas a lista de gaps catalogados em GR-D-011 não a incluía. Um agente que processasse uma pergunta sobre frete de carga perigosa poderia não identificar o gap e tentar responder sem cobertura documental.

*Correção:* adicionada como quinto item na lista de gaps catalogados, com instrução de não citar valores sem confirmar versão vigente com Compliance.

---

### Problemas menores — formatação e clareza

**Problema 9–11 · Glossário desalinhado com correções do relógio de SLA**

As entradas `Horário comercial` e `Relógio de SLA` no glossário (seção 4) repetiam a generalização incorreta do problema 5 com texto ligeiramente diferente do GR-D-007 — risco de contradição interna se as seções fossem lidas isoladamente.

*Correção:* ambas alinhadas ao texto corrigido em GR-D-007, com referência cruzada explícita.

**Problema 12 · Markdown inválido na entrada `Gold` do glossário**

`*premium, VIP, **platinum**, top*` — negrito dentro de itálico é sintaxe inválida em CommonMark e renderiza com asteriscos literais em alguns parsers.

*Correção:* normalizado para `*premium, VIP, platinum, top*`.

---

### Estado final dos artefatos

| Arquivo | Status |
|---------|--------|
| `AGENTS.md` | ✅ Revisado — 12 correções aplicadas |
| `chat-novatech-agents-md.md` | ✅ Este arquivo — atualizado com turno 4 |

