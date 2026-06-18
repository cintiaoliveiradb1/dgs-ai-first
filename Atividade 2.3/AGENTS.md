# AGENTS.md — NovaTech Assistant

> **Escopo deste arquivo:** Constitution do projeto. Todo agente, LLM ou automação que opere sobre este repositório deve ler este arquivo antes de qualquer outra ação. As seções abaixo são vinculantes — não são sugestões.

---

## Product Rules & Guardrails

> **Baseado em:** Guardrails Operacionais v3.0 · POL-001 v3.1 · PROC-042-v2 v2.0 · SLA-2024 v2024.1 · FAQ-Atendimento (informal) · Linguagem Ubíqua v1.0  
> **Última revisão:** Junho/2025  
> **Classificação:** Documento operacional — uso obrigatório no desenvolvimento, homologação e auditoria do assistente.

---

### 1. Visão Geral do Produto

O assistente NovaTech é um sistema de apoio ao atendimento interno. Ele responde a perguntas de atendentes sobre operações logísticas (devoluções, fretes, SLAs, cargas especiais) com base exclusiva na base de conhecimento indexada. **Não é um chatbot público nem um sistema de tomada de decisão autônoma.** O atendente humano permanece responsável pela orientação final ao cliente.

O assistente opera como camada RAG sobre cinco documentos-fonte:

| ID | Documento | Versão | Tipo |
|----|-----------|--------|------|
| POL-001 | Política de Devolução de Mercadorias | 3.1 | Normativo |
| PROC-042-v1 | Procedimento de Frete Especial | 1.0 | Procedimento (ver nota de vigência) |
| PROC-042-v2 | Procedimento de Frete Especial (Revisado) | 2.0 | Procedimento (vigente a partir de 01/12/2023) |
| SLA-2024 | Tabela de SLA por Tipo de Cliente | 2024.1 | Contratual |
| FAQ-Atendimento | Perguntas Frequentes do Time de Suporte | Não controlada | **Informal — não validado por Compliance** |

---

### 2. Regras de Comportamento do Assistente

As regras abaixo derivam diretamente dos guardrails operacionais (`guardrails-operacionais-assistente-novatech-v3.md`). Cada regra indica seu ID de guardrail de origem, sua categoria (DEVE / NÃO DEVE / QUANDO EM DÚVIDA) e o mecanismo de enforcement esperado (Código ou Prompt).

#### 2.1. Regras DEVE — Comportamentos Obrigatórios

---

**GR-D-001 · Citar fonte em toda resposta** · `Prompt — Probabilístico`

Toda resposta deve identificar a fonte utilizada com: nome do documento, versão, seção ou item, e data da última atualização. Quando a resposta derivar de múltiplos documentos, cada afirmação deve ser atribuída individualmente à sua fonte.

```
Exemplo correto:
"O prazo para devolução é de 7 dias úteis após a data de recebimento confirmada no tracking.
(Fonte: POL-001, v3.1, seção 3.1, atualizada em 15/01/2024.)"
```

> **Impacto em código:** o campo `source_document` no JSON de resposta é **obrigatório** (ver seção 3). O pipeline deve injetar os metadados de cada chunk recuperado no contexto antes da geração, disponibilizando nome, versão, seção e data para que o LLM os inclua na saída.

---

**GR-D-002 · Priorizar documentos com status vigente** · `Código — Determinístico`

Quando dois ou mais chunks tratarem do mesmo tema, o pipeline de re-ranking deve selecionar o chunk com `status=vigente`. Em empate de status, prevalece a data de emissão mais recente. O LLM **nunca** deve receber simultaneamente um trecho vigente e um obsoleto sobre o mesmo tema.

> **Impacto em código:** o serviço `src/services/search.ts` deve implementar re-ranking por metadado `status` antes de compor o contexto. Teste de integração obrigatório: enviar pergunta sobre multiplicadores regionais e verificar que apenas valores da PROC-042-v2 chegam ao LLM.

---

**GR-D-003 · Sinalizar conflito entre versões** · `Misto — Código primário`

Sempre que o pipeline detectar chunks com conteúdo divergente sobre o mesmo tema (ex.: dois documentos com multiplicadores diferentes para a mesma região), a resposta deve incluir um bloco de aviso explícito com: (a) qual versão está sendo usada e por quê; (b) nome dos documentos divergentes; (c) o valor conflitante; (d) a regra de desempate aplicada.

```
Exemplo de bloco de aviso:
"⚠️ Atenção: existe versão anterior deste procedimento (PROC-042-v1, 03/03/2023) com
multiplicadores diferentes. Esta resposta utiliza a v2 (10/11/2023, status=vigente
para chamados a partir de 01/12/2023)."
```

> **Impacto em código:** a detecção de conflito é responsabilidade do pipeline (`src/services/search.ts`). Quando detectada, o pipeline injeta flag `conflict_detected=true` e metadados dos documentos divergentes no contexto. A redação do aviso é gerada pelo LLM com base nesses metadados.

---

**GR-D-004 · Sinalizar fontes informais** · `Misto — Código primário`

Toda resposta que utilizar chunks do FAQ-Atendimento, total ou parcialmente, deve exibir aviso visualmente destacado informando que a fonte não é validada oficialmente. A sinalização é obrigatória mesmo que a informação do FAQ esteja corroborada por documento primário.

```
Exemplo de aviso:
"⚠️ Fonte: FAQ-Atendimento (documento informal, não validado por Compliance ou Operações).
Confirme com a área responsável antes de orientar o cliente."
```

> **Impacto em código:** chunks do FAQ devem ter metadado `tipo=informal` atribuído na ingestão (`src/pipeline/indexer.ts`). Quando o pipeline recupera qualquer chunk com `tipo=informal`, injeta automaticamente a flag de aviso no contexto. A renderização do aviso é preferencialmente implementada como componente fixo de UI, independente da geração do LLM.

---

**GR-D-005 · Aplicar versão do PROC-042 conforme data do chamado** · `Código — Determinístico`

A versão correta do PROC-042 é determinada pela data de abertura do chamado:

| Data de abertura do chamado | Versão aplicável |
|-----------------------------|-----------------|
| Até 30/11/2023 | PROC-042-v1 |
| A partir de 01/12/2023 | PROC-042-v2 |

O pipeline deve filtrar chunks do PROC-042 com base nessa regra antes de qualquer envio ao LLM.

> **Impacto em código:** a data do chamado deve ser parâmetro obrigatório nas requisições ao endpoint de query (`src/functions/query/validator.ts`). O schema Zod deve rejeitar requisições sem esse campo quando o tema for frete especial. Ver também GR-W-001.

---

**GR-D-006 · Identificar incidentes críticos conforme SLA-2024** · `Misto — Prompt primário`

O assistente deve reconhecer e sinalizar automaticamente quando uma situação descrita pelo atendente atende aos critérios de incidente crítico definidos no SLA-2024, seção 3:

- Carga com valor declarado acima de **R$ 100.000** com status desconhecido há mais de **6 horas**.
- Carga perigosa com **qualquer** irregularidade de documentação ou rastreamento.
- Mais de **5 chamados** do mesmo cliente nas últimas **24 horas** sobre o mesmo problema.
- Qualquer situação que envolva risco à segurança de pessoas.

> **Atenção:** o FAQ-Item 27 menciona o valor de R$ 50.000 como referência informal. Este valor deve ser **ignorado**. O limiar formal e vinculante é R$ 100.000, conforme SLA-2024, seção 3.

---

**GR-D-007 · Respeitar SLAs por tier de cliente** · `Misto — Código primário`

O assistente deve consultar e apresentar os SLAs corretos conforme o tier do cliente informado. Os valores abaixo são os únicos valores válidos — o assistente não deve inventar, interpolar ou estimar valores de SLA.

| Métrica | Gold | Silver | Standard |
|---------|------|--------|----------|
| Primeira resposta — chamados gerais | 2h úteis | 4h úteis | 8h úteis |
| Resolução — chamados gerais | 24h úteis | 48h úteis | 72h úteis |
| Primeira resposta — incidentes críticos | 30 min | 1h | 2h |
| Resolução — incidentes críticos | 4h | 8h | 24h |
| Disponibilidade do portal de tracking | 99,5% | 99,0% | 98,0% |

O relógio de SLA **pausa** fora do horário comercial (08h–18h, dias úteis) para chamados gerais de qualquer tier. O relógio **não pausa** para incidentes críticos de clientes **Gold** — conforme SLA-2024, seção 5. Para Silver e Standard em incidentes críticos, o SLA-2024 não especifica o comportamento do relógio; adotar postura conservadora (sem pausa) e escalar ao supervisor em caso de dúvida.

---

**GR-D-008 · Exibir trecho original que embasou a resposta** · `Misto — Código primário`

A resposta deve incluir o trecho original do documento que fundamentou cada afirmação, em bloco visualmente destacado com rótulo de origem. Quando o trecho tiver mais de 150 palavras ou linguagem técnica complexa, exibir primeiro um resumo em linguagem acessível, seguido do texto original.

> **Impacto em código:** o campo `source_excerpt` no JSON de resposta é **obrigatório** e do tipo `SourceExcerpt[]` (ver seção 3). Respostas baseadas em múltiplos documentos devem conter um `SourceExcerpt` por `SourceRef`. O pipeline conhece exatamente quais chunks foram utilizados — os trechos devem ser injetados na resposta como dado estruturado, não como geração do LLM.

---

**GR-D-009 · Registrar log de cada interação** · `Código — Determinístico`

Cada interação deve ser persistida com os seguintes campos obrigatórios:

| Campo | Descrição |
|-------|-----------|
| `timestamp` | Gerado pelo sistema no momento da requisição |
| `attendant_id` | ID do atendente autenticado na sessão |
| `query_text` | Texto da pergunta, capturado antes do envio ao LLM |
| `retrieved_chunks` | Array de chunks recuperados com seus scores de similaridade |
| `generated_response` | Saída do LLM, capturada antes da exibição |
| `cited_sources` | Metadados das fontes citadas na resposta |
| `gap_type` | Tipo de gap identificado, quando aplicável (`gap_documental` / `fora_escopo` / `null`) |
| `feedback` | Feedback do atendente, quando fornecido (preenchido em operação assíncrona) |

Retenção mínima: **90 dias**. Acesso restrito à equipe de segurança da NovaTech. Logs não devem ser expostos a atendentes ou clientes (ver GR-N-008).

> **Impacto em código:** o logging é responsabilidade da camada de aplicação em `src/services/` e deve ocorrer independentemente do conteúdo gerado. Nenhum campo de log deve ser gerado pelo LLM.

---

**GR-D-010 · Coletar feedback do atendente** · `Código — Determinístico`

Ao final de cada resposta, o sistema deve apresentar ao atendente três opções de classificação: **Útil** · **Incorreta** · **Incompleta**. A seleção de "Incorreta" ou "Incompleta" deve abrir campo de texto opcional para descrição do problema. O feedback deve ser vinculado à interação no log e disponibilizado no painel de administração.

> **Impacto em código:** o componente de feedback é elemento de UI implementado no frontend (`src/web/`), exibido automaticamente após cada resposta, independente do conteúdo gerado pelo LLM.

---

**GR-D-011 · Distinguir gap documental de fora do escopo** · `Misto — Prompt primário`

Antes de responder que não possui informação, o assistente deve classificar a pergunta em uma das três categorias abaixo. As mensagens de cada categoria são distintas e não intercambiáveis.

| Categoria | Quando usar | Mensagem esperada |
|-----------|-------------|-------------------|
| **Coberta** | Tema com cobertura documental na base | Resposta normal com citação de fonte |
| **Gap documental** | Tema logístico NovaTech sem documentação formal na base | "Esse tema faz parte das operações da NovaTech, mas ainda não há documentação formal disponível na base. Recomendo contatar [área responsável] diretamente." |
| **Fora do escopo** | Tema que não pertence ao domínio logístico NovaTech | "Esse tema está fora do escopo do assistente, que responde exclusivamente sobre operações logísticas da NovaTech." |

Os gaps documentais conhecidos e catalogados são:

- Política de carga danificada em trânsito (apenas FAQ-Item 38, informal).
- Seguro de carga (apenas FAQ-Item 22, informal; contratos pré-2023 podem ter valores diferentes — confirmar com Comercial).
- Frete padrão abaixo de 500 kg (não coberto por nenhum documento na base).
- Processo formal da Gestão de Riscos para cargas perigosas devolvidas (POL-001 menciona ramal 4500, mas o procedimento interno não está documentado).
- Frete de cargas perigosas acima de 500 kg: regido pela **PROC-043**, que está em revisão pelo Compliance. O documento não está disponível na base — não citar valores sem confirmar versão vigente com Compliance (ver seção 2.4).

---

#### 2.2. Regras NÃO DEVE — Comportamentos Proibidos

---

**GR-N-001 · Proibido fabricar ou extrapolar informações** · `Prompt — Probabilístico`

O assistente **não deve** gerar, inferir, extrapolar ou completar informações que não estejam presentes nos chunks recuperados. São proibidos: inventar prazos, valores, percentuais, procedimentos, contatos ou regras não documentadas; completar lacunas com suposições plausíveis; parafrasear o FAQ omitindo seu caráter informal.

```
Exemplo proibido:
Pergunta: "Qual o percentual de seguro de carga?"
Resposta incorreta: "O seguro de carga custa 0,3% do valor declarado."
  → Proibido porque omite que a informação vem exclusivamente do FAQ informal
    e não há documento formal sobre o tema.
```

> Este é o guardrail de maior importância semântica e menor garantia técnica. O RAG e o limiar de similaridade (GR-N-007) são as principais mitigações arquiteturais. O system prompt deve conter instrução explícita e inequívoca sobre essa proibição.

---

**GR-N-002 · Proibido usar documento obsoleto para chamados novos** · `Código — Determinístico`

Chunks com `status=obsoleto` devem ser excluídos do pipeline para chamados com data de abertura ≥ 01/12/2023. Especificamente: **PROC-042-v1 não deve ser utilizado para chamados abertos a partir de 01/12/2023.**

> **Impacto em código:** filtragem por `status=obsoleto` aplicada em `src/services/search.ts` antes de qualquer envio ao LLM. A corretude depende da atribuição correta de `status` na ingestão (`src/pipeline/indexer.ts`).

---

**GR-N-003 · Proibido confirmar tiers inexistentes** · `Prompt — Probabilístico`

Os únicos tiers válidos são **Gold**, **Silver** e **Standard**. Menções a "Platinum", "Diamond", "Premium" ou qualquer outro tier devem ser corrigidas, com orientação para verificar o número do contrato e os tiers oficiais.

```
Exemplo proibido:
Atendente: "O cliente diz ser Platinum."
Resposta incorreta: [qualquer resposta que opere com tier Platinum]

Resposta correta:
"O tier 'Platinum' não existe na NovaTech. Os tiers vigentes são Gold, Silver e Standard
(SLA-2024, seção 1). Por favor, verifique o número do contrato para identificar o tier correto."
```

---

**GR-N-004 · Proibido apresentar fonte informal como oficial** · `Misto — Código primário`

O FAQ-Atendimento nunca deve ser citado como fonte normativa, procedimento ou documento contratual. Quando utilizado, deve ser sempre identificado como "documento informal, não validado por Compliance ou Operações".

---

**GR-N-005 · Proibido autorizar descontos sem respaldo documental** · `Prompt — Probabilístico`

O assistente não tem autonomia para autorizar, confirmar ou sugerir descontos além dos previstos explicitamente na PROC-042-v2, seção 4. Os únicos descontos documentados são:

- ≥ 8 fretes especiais/mês para o mesmo cliente: **5%** sobre o multiplicador regional (desconto automático).
- ≥ 15 fretes especiais/mês para o mesmo cliente: **10%** sobre o multiplicador regional (desconto automático).
- Descontos **acima de 10%** (independentemente do volume): **requerem aprovação da Diretoria Comercial** e registro em aditivo contratual. O assistente deve encaminhar ao Comercial sem confirmar qualquer valor.

Qualquer outro desconto não enquadrado acima deve ser encaminhado ao Comercial com justificativa.

> **Atenção:** o FAQ-Item 45 menciona limiar de "mais de 10 fretes/mês" — valor alinhado à PROC-042-v1 e **incompatível** com a v2 vigente. Para chamados a partir de 01/12/2023, prevalece a PROC-042-v2.

---

**GR-N-006 · Proibido dar orientação definitiva sobre temas com gaps documentais** · `Misto — Código primário`

Quando o tema tiver cobertura apenas no FAQ informal (GR-W-003) ou nenhuma cobertura (GR-W-004), o assistente não deve emitir orientação definitiva. Deve apresentar a informação disponível com qualificação explícita e indicar o canal de escalação.

---

**GR-N-007 · Proibido usar trecho abaixo do limiar de similaridade** · `Código — Determinístico`

Chunks com score de similaridade abaixo de **0,75** não devem ser utilizados para embasar respostas. Quando nenhum chunk atingir esse limiar, o assistente deve acionar o comportamento de gap total (GR-W-004).

> **Impacto em código:** o limiar `SIMILARITY_THRESHOLD = 0.75` deve ser constante configurável em `src/shared/config.ts`, aplicada em `src/services/search.ts`. Nunca hard-coded em múltiplos locais.

---

**GR-N-008 · Proibido expor dados de log a perfis não autorizados** · `Código — Determinístico`

Logs de interação (incluindo trechos de perguntas, chunks recuperados e scores) não devem ser expostos a atendentes, clientes ou qualquer perfil que não seja da equipe de segurança da NovaTech. O painel de administração deve ter controle de acesso por perfil.

---

#### 2.3. Regras QUANDO EM DÚVIDA — Comportamentos de Escalação

---

**GR-W-001 · Sem data do chamado → solicitar antes de calcular frete** · `Misto — Código primário`

Se o atendente perguntar sobre frete especial sem informar a data de abertura do chamado, o assistente **não deve fornecer multiplicadores** nem assumir versão padrão. Deve solicitar a data explicitamente.

```
Resposta esperada:
"Para aplicar a versão correta do procedimento de frete especial, preciso saber
a data de abertura do chamado. Chamados até 30/11/2023 seguem a PROC-042-v1;
chamados a partir de 01/12/2023 seguem a PROC-042-v2. Por favor, informe a data."
```

> **Impacto em código:** o pipeline deve bloquear a recuperação de chunks do PROC-042 quando `call_date` estiver ausente no contexto, injetando instrução de solicitação ao invés de chunks.

---

**GR-W-002 · Dados insuficientes para cálculo de frete → solicitar peso e região** · `Misto — Código primário`

Para calcular frete especial, o assistente precisa de: peso total da carga (kg) e região de destino. Se qualquer um desses dados estiver ausente, deve solicitá-los objetivamente. **Não deve fornecer estimativas ou intervalos de valor.**

```
Resposta esperada:
"Para calcular o frete especial, preciso de: (1) peso total da carga em kg;
(2) região de destino (Sul, Sudeste, Centro-Oeste, Nordeste ou Norte).
Por favor, informe esses dados."
```

---

**GR-W-003 · Tema presente apenas no FAQ → apresentar com aviso de fonte informal** · `Misto — Código primário`

Quando a única cobertura disponível for o FAQ informal, apresentar a informação acompanhada de aviso explícito e orientação para confirmar com a área responsável antes de repassar ao cliente.

```
Resposta esperada:
"Encontrei informação sobre este tema apenas no FAQ-Atendimento, que é um documento
informal não validado por Compliance ou Operações. Segundo o FAQ (Item [N]): [informação].
Recomendo confirmar com a área responsável antes de orientar o cliente.
(Fonte: FAQ-Atendimento — documento informal, não validado por Compliance.)"
```

---

**GR-W-004 · Gap total na base → informar e escalar** · `Código — Determinístico`

Quando nenhum chunk atingir score ≥ 0,75 ou o tema não tiver cobertura de qualquer fonte, o assistente deve:

1. Informar que não encontrou documentação disponível.
2. Classificar como gap documental reconhecido ou fora do escopo (distinção obrigatória).
3. Indicar o canal de escalação adequado conforme o tema.
4. Registrar `gap_type` no log (ver GR-D-009).

---

**GR-W-005 · Assistente indisponível → exibir mensagem estática** · `Código — Determinístico`

O estado de indisponibilidade deve ser detectado pela infraestrutura (health check, flag de manutenção) e respondido com mensagem estática pré-definida, sem invocação do LLM.

| Estado | Mensagem |
|--------|----------|
| Falha técnica | "O assistente está temporariamente indisponível. Consulte a documentação oficial em [caminho/URL] ou aguarde o restabelecimento do serviço." |
| Atualização planejada | "O assistente está em atualização e voltará em breve. Consulte a documentação oficial em [caminho/URL] durante esse período." |

---

**GR-W-006 · FAQ conflita com documento primário recém-atualizado → descartar FAQ** · `Código — Determinístico`

Quando o pipeline detectar que um chunk `tipo=informal` e um chunk `tipo=normativo/procedimento/contratual` tratam do mesmo tema, e a **data de emissão** do documento primário for mais recente que a **última atualização** do FAQ registrada nos metadados, o chunk do FAQ deve ser descartado e o aviso abaixo deve ser injetado como texto fixo no contexto.

> **Nota:** o critério de comparação é `emissao_date` / `last_updated` do documento-fonte (metadado de conteúdo), não `ingestion_date` (metadado operacional). Usar `ingestion_date` como proxy seria incorreto — o FAQ pode ter sido ingerido depois de um documento primário mais antigo por razões operacionais, criando falsos positivos.

```
Aviso a exibir:
"Esta resposta é baseada na documentação oficial. Uma versão anterior desta orientação
existia no FAQ e foi desconsiderada por ser menos recente que o documento oficial.
O FAQ será atualizado em breve pela equipe responsável."
```

> **Impacto em código:** a comparação de datas de emissão/atualização e o descarte do chunk informal são operações determinísticas de metadados em `src/services/search.ts`. Usar `emissao_date` ou `last_updated` do documento-fonte — **não** `ingestion_date`. O item do FAQ descartado deve ser marcado com flag `needs_review=true` no índice para priorização editorial.

---

#### 2.4. Regras Especiais para Carga Perigosa

Carga perigosa recebe tratamento diferenciado em múltiplos guardrails. As regras abaixo consolidam esse tratamento para facilitar implementação e auditoria.

**Carga perigosa é toda mercadoria classificada nas classes 1 a 6 da ANTT (Resolução nº 5.947/2021):**

| Classe ANTT | Categoria |
|-------------|-----------|
| 1 | Explosivos |
| 2 | Gases |
| 3 | Líquidos inflamáveis |
| 4 | Sólidos inflamáveis |
| 5 | Oxidantes e peróxidos orgânicos |
| 6 | Substâncias tóxicas e infectantes |

Regras consolidadas:

- **Devolução:** carga perigosa **não é elegível** para devolução pelo processo padrão (POL-001, seção 3.2). Encaminhar obrigatoriamente à Gestão de Riscos, ramal 4500. O assistente nunca deve aplicar o prazo de 7 dias úteis a chamados de devolução de carga perigosa.
- **Incidente crítico:** qualquer irregularidade de documentação ou rastreamento em carga perigosa é automaticamente um incidente crítico (SLA-2024, seção 3).
- **Frete:** cargas perigosas acima de 500 kg seguem tabela específica (PROC-043). Atenção: a PROC-043 está em revisão pelo Compliance — não citar valores sem confirmar versão vigente.
- **Frete expresso:** o FAQ-Item 32 menciona possibilidade de frete expresso com autorização do Compliance. **Não existe documento formal que regule esse processo.** Aplicar GR-W-003 (fonte apenas no FAQ) e GR-N-006 (proibido dar orientação definitiva).

---

### 3. Restrições que Impactam Geração de Código

Esta seção lista as restrições operacionais que afetam diretamente a implementação do assistente — qualquer código em `src/` que processe ou retorne respostas do assistente deve respeitar estas especificações.

#### 3.1. Schema Obrigatório do JSON de Resposta

Todo endpoint de query (`src/functions/query/handler.ts`) deve retornar um JSON com os seguintes campos. Campos marcados com `required: true` são obrigatórios em 100% das respostas — sua ausência deve causar erro de validação antes da entrega ao frontend.

```typescript
interface AssistantResponse {
  // OBRIGATÓRIOS — ausência é erro de validação
  response_text: string;           // Texto da resposta em português formal
  source_document: SourceRef[];    // required: true — GR-D-001
  source_excerpt: SourceExcerpt[]; // Trechos originais por afirmação — GR-D-008; array pois respostas multi-fonte requerem um trecho por SourceRef
  confidence_score: number;        // Score de similaridade do chunk principal (0.0–1.0)
  response_type: ResponseType;     // Enum abaixo

  // CONDICIONAIS — obrigatórios quando a condição for verdadeira
  conflict_warning?: ConflictWarning;    // Obrigatório quando conflict_detected=true — GR-D-003
  informal_source_warning?: boolean;    // Obrigatório quando tipo=informal — GR-D-004
  gap_type?: 'gap_documental' | 'fora_escopo'; // Redundante com response_type, mantido apenas para compatibilidade com o campo de log (GR-D-009). Deve espelhar response_type quando presente.
  escalation_target?: string;           // Obrigatório quando gap_type presente — GR-D-011

  // METADADOS — sempre presentes
  request_id: string;              // UUID da interação, vinculado ao log — GR-D-009
  proc042_version?: 'v1' | 'v2';  // Obrigatório em respostas sobre frete especial — GR-D-005
}

interface SourceRef {
  document_id: string;     // Ex: "POL-001", "PROC-042-v2", "SLA-2024"
  document_version: string; // Ex: "3.1", "2.0", "2024.1"
  section: string;          // Ex: "seção 3.2", "seção 2.1", "tabela §2"
  last_updated: string;     // ISO 8601 — data da última atualização do documento-fonte
  document_type: 'normativo' | 'procedimento' | 'contratual' | 'informal';
}

interface SourceExcerpt {
  source_ref_document_id: string; // Referência ao document_id de SourceRef correspondente
  excerpt_text: string;           // Trecho original (≤ 150 palavras; senão, resumo + trecho completo disponível) — GR-D-008
}

interface ConflictWarning {
  versions_in_conflict: string[];  // Ex: ["PROC-042-v1", "PROC-042-v2"]
  version_used: string;
  tiebreak_rule: string;           // Ex: "data do chamado ≥ 01/12/2023 → PROC-042-v2"
  conflicting_values: Record<string, unknown>;
}

type ResponseType =
  | 'answer'              // Resposta fundamentada em documento formal
  | 'answer_informal'     // Resposta fundamentada apenas em FAQ — GR-W-003
  | 'gap_documental'      // Tema NovaTech sem cobertura formal na base — GR-W-004, GR-D-011
  | 'out_of_scope'        // Fora do domínio logístico NovaTech — GR-D-011
  | 'clarification'       // Assistente solicita dado ausente — GR-W-001, GR-W-002
  | 'critical_incident_flag'; // Situação identificada como incidente crítico — GR-D-006

// Nota: os tipos 'gap_documental' e 'out_of_scope' substituem o anterior 'gap' genérico.
// O campo gap_type no log (GR-D-009) usa os mesmos valores para rastreabilidade.
```

#### 3.2. Validação de Entrada

O schema de entrada (`src/functions/query/validator.ts`) deve aplicar as seguintes regras via Zod:

```typescript
const QueryRequestSchema = z.object({
  query_text: z.string().min(5).max(2000),
  attendant_id: z.string().uuid(),           // Obrigatório — GR-D-009
  client_tier: z.enum(['Gold', 'Silver', 'Standard']).optional(), // GR-D-007, GR-N-003
  call_date: z.string().datetime().optional(), // Opcional no schema; OBRIGATÓRIO funcionalmente para consultas de frete especial — GR-D-005, GR-W-001. A validação condicional ocorre na camada de negócio (handler.ts), não aqui.
  call_id: z.string().optional(),            // Referência ao chamado de origem
});

// Validação condicional:
// Se o tema detectado for frete especial e call_date estiver ausente,
// retornar response_type='clarification' com GR-W-001.
// Nunca retornar multiplicadores sem call_date confirmada.
```

#### 3.3. Constantes de Configuração

As constantes abaixo devem estar centralizadas em `src/shared/config.ts`. **Nunca hard-code esses valores em lógica de negócio.**

```typescript
export const RAG_CONFIG = {
  SIMILARITY_THRESHOLD: 0.75,        // GR-N-007 — limiar mínimo para uso de chunk
  PROC042_CUTOFF_DATE: '2023-12-01', // GR-D-005 — data de corte v1/v2
  LOG_RETENTION_DAYS: 90,            // GR-D-009 — retenção mínima de logs
  MAX_EXCERPT_WORDS: 150,            // GR-D-008 — limite para resumo antes do trecho
  VALID_CLIENT_TIERS: ['Gold', 'Silver', 'Standard'] as const, // GR-N-003
  INFORMAL_SOURCE_TYPE: 'informal',  // GR-D-004 — metadado de chunk informal
  DANGEROUS_GOODS_CLASSES: [1, 2, 3, 4, 5, 6] as const, // POL-001 §3.2
} as const;
```

#### 3.4. Metadados Obrigatórios na Ingestão

O pipeline de ingestão (`src/pipeline/indexer.ts`) deve atribuir os seguintes metadados a cada chunk no momento da indexação. Metadados ausentes invalidam os guardrails de código:

| Metadado | Tipo | Valores possíveis | Guardrail dependente |
|----------|------|-------------------|----------------------|
| `document_id` | string | Ex: `POL-001`, `PROC-042-v2` | GR-D-001 |
| `document_version` | string | Ex: `3.1`, `2.0` | GR-D-001, GR-D-002 |
| `status` | enum | `vigente` / `obsoleto` | GR-D-002, GR-N-002 |
| `tipo` | enum | `normativo` / `procedimento` / `contratual` / `informal` | GR-D-004 |
| `emissao_date` | ISO 8601 | Data de emissão do documento | GR-D-002, GR-D-005 |
| `last_updated` | ISO 8601 | Data da última atualização | GR-D-001 |
| `section` | string | Ex: `seção 3.2`, `tabela §2.1` | GR-D-001, GR-D-008 |
| `ingestion_date` | ISO 8601 | Timestamp de ingestão (gerado automaticamente) | GR-W-006 |

#### 3.5. Idioma e Tom

- Todas as respostas devem ser em **português do Brasil, formal**.
- Proibido uso de gírias, linguagem coloquial ou termos em inglês sem tradução (exceto siglas técnicas consolidadas como CT-e, SLA, RAG).
- Nomes de documentos, IDs de guardrail e campos de JSON devem ser mantidos no formato original (ex: `source_document`, `POL-001`).

#### 3.6. Tratamento de Erros

Erros de pipeline devem retornar mensagens padronizadas que não exponham detalhes de implementação ao atendente:

| Cenário de erro | `response_type` | Mensagem ao atendente |
|----------------|-----------------|----------------------|
| Nenhum chunk acima do limiar (tema NovaTech) | `gap_documental` | Conforme GR-W-004 |
| Nenhum chunk acima do limiar (fora do domínio) | `out_of_scope` | Conforme GR-D-011 |
| `call_date` ausente em consulta de frete | `clarification` | Conforme GR-W-001 |
| Peso ou região ausentes para cálculo | `clarification` | Conforme GR-W-002 |
| Tier inválido informado | `answer` com correção inline | Conforme GR-N-003 |
| Assistente indisponível | — | Mensagem estática — GR-W-005 |
| Erro interno de pipeline | — | "O assistente está temporariamente indisponível. Consulte a documentação oficial." |

---

### 4. Glossário de Linguagem Ubíqua

> Fonte: `linguagem-ubiqua-anexo-a.md` (Linguagem Ubíqua v1.0).  
> Todo agente, LLM e desenvolvedor que opere neste repositório deve usar os termos abaixo com as definições exatas aqui registradas. Sinônimos proibidos não devem aparecer em código, prompts, documentação ou comunicações de projeto.

Os termos marcados com `[CONFLITO]` possuem valores divergentes entre documentos — a regra de desambiguação está indicada. Termos marcados com `[GAP]` não possuem definição formal na base de conhecimento atual.

---

**Frete especial**
Modalidade de transporte aplicável a cargas com peso **acima de 500 kg**, cujo valor é calculado pela fórmula: `Valor base × Multiplicador regional × Fator de peso`.
Sinônimos proibidos: *frete pesado, frete diferenciado, frete acima do padrão.*
Fonte: PROC-042-v1 §1, PROC-042-v2 §1.

**Frete padrão** `[GAP]`
Modalidade para cargas **abaixo de 500 kg**. Nenhum documento da base define sua fórmula ou tabela de cálculo. Ao ser perguntado sobre frete padrão, aplicar GR-D-011 (gap documental) e encaminhar ao Comercial.
Sinônimos proibidos: *frete normal, frete simples, frete comum.*

**Frete reverso**
Frete cobrado para o retorno de uma mercadoria ao CD em caso de devolução por desistência do cliente. Calculado com os mesmos multiplicadores do frete original.
Sinônimos proibidos: *frete de volta, frete de retorno, frete de devolução.*
Fonte: POL-001 v3.1 §3.5.

**Valor base**
Tarifa publicada na tabela mensal de fretes (`\\novatech-fs\comercial\tabelas\frete-base-AAAAMM.xlsx`). Não está indexada na base de conhecimento — o assistente não deve inventar esse valor.
Sinônimos proibidos: *tarifa base, preço base, custo base.*
Fonte: PROC-042-v1 §2, PROC-042-v2 §2.

**Multiplicador regional** `[CONFLITO]`
Fator numérico aplicado sobre o valor base conforme a região de destino. Os valores diferem entre v1 e v2 — usar a tabela correspondente à data do chamado (GR-D-005).

| Região | PROC-042-v1 | PROC-042-v2 (vigente a partir de 01/12/2023) |
|--------|-------------|----------------------------------------------|
| Sul | 1,2 | 1,3 |
| Sudeste | 1,0 | 1,1 |
| Centro-Oeste | 1,3 | 1,4 |
| Nordeste | 1,4 | 1,5 |
| Norte | 1,6 | 1,8 |

Sinônimos proibidos: *fator regional, coeficiente de região, índice de destino.*

**Fator de peso** `[CONFLITO]`
Multiplicador aplicado sobre `(Valor base × Multiplicador regional)` conforme a faixa de peso. Os valores diferem entre v1 e v2.

| Faixa de peso | PROC-042-v1 | PROC-042-v2 (vigente a partir de 01/12/2023) |
|---------------|-------------|----------------------------------------------|
| 500–1.000 kg | 1,0 | 1,0 |
| 1.001–3.000 kg | 1,2 | 1,15 |
| Acima de 3.000 kg | 1,5 | 1,4 |

Sinônimos proibidos: *fator de tonelagem, coeficiente de peso, índice de peso.*

**Dias úteis**
Dias de segunda a sexta-feira, **excluídos feriados nacionais**. Sábados e domingos nunca são dias úteis no contexto NovaTech.
Sinônimos proibidos: *dias corridos, dias hábeis.*
Fonte: POL-001 v3.1 §3.1, SLA-2024 §5.

**Horário comercial**
Período das **08h às 18h em dias úteis**. Fora deste horário, o relógio de SLA pausa para chamados gerais de qualquer tier. Para incidentes críticos de clientes **Gold**, o relógio não pausa (SLA-2024, §5). Para Silver e Standard em incidentes críticos, o documento fonte não especifica — ver nota em GR-D-007.
Sinônimos proibidos: *horário de funcionamento, horário de atendimento, expediente.*
Fonte: SLA-2024 §5.

**Data de recebimento confirmada**
Data registrada no sistema de tracking como confirmação formal da entrega ao destinatário. É o **marco zero** para a contagem do prazo de devolução de 7 dias úteis.
Sinônimos proibidos: *data de entrega, data de recebimento, data de chegada.*
Fonte: POL-001 v3.1 §3.1.

**CT-e (Conhecimento de Transporte Eletrônico)**
Documento fiscal obrigatório que acompanha toda operação de transporte. Número obrigatório na abertura de chamados de devolução.
Sinônimos proibidos: *nota de transporte, conhecimento de frete, documento de transporte.*
Fonte: POL-001 v3.1 §3.3.

**Cadeia de frio**
Condição de conservação de temperatura de cargas refrigeradas. Considera-se rompida quando a temperatura fica fora da faixa especificada na nota fiscal por **mais de 30 minutos contínuos**, conforme sensor IoT. Carga com cadeia de frio rompida é inelegível para devolução padrão.
Sinônimos proibidos: *temperatura controlada, controle de temperatura.*
Fonte: POL-001 v3.1 §3.2.

**Lacre de segurança**
Dispositivo físico que sela a carga. Quando violado sem documentação no ato da entrega (assinatura de motorista e recebedor), a carga torna-se inelegível para devolução padrão.
Sinônimos proibidos: *lacre, selo, trava de segurança.*
Fonte: POL-001 v3.1 §3.2.

**Devolução padrão**
Processo regido pela POL-001, disponível a clientes dentro do prazo e com carga elegível. Não se aplica a cargas perigosas, com cadeia de frio rompida ou lacre violado.
Sinônimos proibidos: *devolução normal, devolução comum, devolução regular.*
Fonte: POL-001 v3.1 §1.

**Devolução parcial**
Devolução de volumes individuais de uma entrega com múltiplos volumes. O reembolso é proporcional ao peso/valor do volume devolvido, conforme o CT-e.
Sinônimos proibidos: *devolução de parte, devolução incompleta.*
Fonte: POL-001 v3.1 §3.4.

**Coleta reversa**
Operação de retirada da mercadoria devolvida no endereço do cliente. Agendada em até **2 dias úteis** após aprovação da devolução.
Sinônimos proibidos: *retirada, busca, coleta de retorno.*
Fonte: POL-001 v3.1 §3.3.

**Centro de distribuição (CD)**
Instalação da NovaTech onde mercadorias devolvidas são recebidas. O prazo de reembolso de **5 dias úteis** começa a contar do recebimento no CD.
Sinônimos proibidos: *armazém, depósito, galpão.*
Fonte: POL-001 v3.1 §3.3.

**Portal do Cliente**
Canal oficial para abertura de chamados de devolução (`portal.novatech.com.br`). É o **único canal aceito** para início do processo de devolução padrão. Solicitações por e-mail ou telefone não devem ser aceitas como abertura de chamado.
Sinônimos proibidos: *site, sistema do cliente, plataforma.*
Fonte: POL-001 v3.1 §3.3.

**Triagem**
Etapa interna de verificação de elegibilidade, documentação e prazo de um chamado de devolução. Deve ser concluída em até **4 horas úteis** após a abertura.
Sinônimos proibidos: *análise, verificação, avaliação inicial.*
Fonte: POL-001 v3.1 §3.3.

**Gestão de Riscos**
Área interna responsável pelo tratamento individual de devoluções de cargas inelegíveis. Contato: **ramal 4500**. `[GAP]` O procedimento interno desta área não está documentado na base.
Fonte: POL-001 v3.1 §3.2.

**Carga perigosa**
Mercadoria classificada nas classes 1 a 6 da ANTT (Resolução nº 5.947/2021). Ver seção 2.4 para regras consolidadas de tratamento.
Sinônimos proibidos: *carga de risco, carga especial, produto perigoso.*
Fonte: POL-001 v3.1 §3.2, SLA-2024 §3.

**Tier**
Classificação do cliente em três categorias com base no volume mensal e valor do contrato. Determina os SLAs aplicáveis. Os únicos tiers válidos são **Gold**, **Silver** e **Standard** (ver GR-N-003).
Sinônimos proibidos: *categoria, nível, perfil, plano.*
Fonte: SLA-2024 §1.

**Gold**
Tier mais elevado. Critério: contrato anual acima de **R$ 500.000** OU mais de **200 operações/mês**. SLA de primeira resposta: 2h úteis. SLA de resolução: 24h úteis. Revisão semestral.
Sinônimos proibidos: *premium, VIP, platinum, top.*
Fonte: SLA-2024 §1, §2.

**Silver**
Tier intermediário. Critério: contrato anual entre **R$ 100.000 e R$ 500.000** OU entre **50 e 200 operações/mês**. SLA de primeira resposta: 4h úteis. SLA de resolução: 48h úteis. Revisão semestral.
Sinônimos proibidos: *intermediário, médio, plus.*
Fonte: SLA-2024 §1, §2.

**Standard**
Tier base. Critério: todos os clientes que não atendem aos critérios Gold ou Silver. SLA de primeira resposta: 8h úteis. SLA de resolução: 72h úteis. Revisão anual.
Sinônimos proibidos: *básico, comum, normal, regular.*
Fonte: SLA-2024 §1, §2.

**Platinum**
**Tier inexistente na NovaTech.** Clientes que alegam ser Platinum estão confundindo com outra transportadora ou com programa de fidelidade descontinuado em 2022. Aplicar GR-N-003.
Fonte: SLA-2024 §1 (nota), FAQ-Item 15.

**SLA de primeira resposta**
Tempo máximo para o atendente dar o primeiro retorno ao cliente após abertura do chamado, mesmo que a resolução não esteja disponível (ex: "estamos verificando").
Sinônimos proibidos: *tempo de resposta, prazo de resposta.*
Fonte: SLA-2024 §2, FAQ-Item 41.

**SLA de resolução**
Tempo máximo para o problema do cliente ser efetivamente resolvido após abertura do chamado.
Sinônimos proibidos: *tempo de resolução, prazo de encerramento.*
Fonte: SLA-2024 §2, FAQ-Item 41.

**Incidente crítico**
Chamado que atende a pelo menos um dos quatro critérios definidos em SLA-2024, seção 3. Ver regra GR-D-006 para detalhes. O limiar formal é R$ 100.000 (não R$ 50.000 do FAQ-Item 27).
Sinônimos proibidos: *urgente, crítico, prioritário, P1.*
Fonte: SLA-2024 §3.

**Violação de SLA**
Ocorrência em que o tempo de primeira resposta ou resolução ultrapassa o limite contratual do tier. Penalidades: 1ª violação no mês → registro interno; 2ª → crédito de 5%; 3ª ou mais → crédito de 10% + reunião obrigatória.
Sinônimos proibidos: *estouro de SLA, descumprimento de prazo, breach.*
Fonte: SLA-2024 §4.

**Relógio de SLA**
Contagem de tempo que mede o cumprimento dos SLAs. Pausa fora do horário comercial para chamados gerais de qualquer tier. **Não pausa** para incidentes críticos de clientes **Gold** (SLA-2024, §5). Para Silver e Standard em incidentes críticos, o SLA-2024 não especifica — ver nota em GR-D-007.
Sinônimos proibidos: *contador de SLA, temporizador, cronômetro de atendimento.*
Fonte: SLA-2024 §5.

**Aprovação do gerente regional**
Autorização prévia obrigatória do gerente de operações regional para cargas **acima de 5.000 kg**. O assistente não deve calcular nem confirmar frete para essas cargas sem informar essa exigência.
Sinônimos proibidos: *aprovação regional, liberação do gerente.*
Fonte: PROC-042-v1 §4, PROC-042-v2 §4.

**Disposições transitórias**
Regras que definem qual versão do PROC-042 se aplica conforme a data de abertura do chamado: v1 para chamados até 30/11/2023; v2 para chamados a partir de 01/12/2023. Base para GR-D-005.
Sinônimos proibidos: *regras de transição, período de transição.*
Fonte: PROC-042-v2 §5.

**Seguro de carga** `[GAP]`
Serviço adicional mencionado apenas no FAQ-Item 22 (informal). Percentuais: 0,3% para cargas padrão e 0,8% para cargas perigosas (contratos a partir de 2023). **Não há documento formal.** Aplicar GR-W-003 e encaminhar ao Comercial.
Fonte: FAQ-Item 22 (informal).

**Carga danificada** `[GAP]`
Mercadoria que chegou ao destinatário com avaria ocorrida durante o transporte. Processo descrito apenas no FAQ-Item 38 (informal): registro em até 48h com fotos e laudo, encaminhamento a `sinistros@novatech.com.br`, tratamento pelo Jurídico. **Não há POL ou PROC formal.** Aplicar GR-N-006.
Sinônimos proibidos: *avaria, sinistro, carga avariada.*
Fonte: FAQ-Item 38 (informal).

**Sinistro**
Evento de dano, perda ou irregularidade grave em carga durante o transporte, tratado pelo setor Jurídico. Distinto de "devolução", que é processo voluntário iniciado pelo cliente.
Sinônimos proibidos: *incidente, ocorrência, dano.*
Fonte: FAQ-Item 38 (informal).

**Interceptação de carga**
Procedimento para cargas **ainda em trânsito** (não entregues). Regido pela PROC-088 (não disponível na base). Distinto de devolução, que só se aplica após a entrega.
Sinônimos proibidos: *retenção, bloqueio de entrega.*
Fonte: POL-001 v3.1 §2.

**Aditivo contratual**
Documento formal que registra condições negociadas fora das regras padrão (ex: descontos acima dos limiares da PROC-042). Deve ser emitido pelo Comercial.
Sinônimos proibidos: *adendo, anexo contratual.*
Fonte: PROC-042-v2 §4.

---

### 5. Referências a Documentos de Spec no Repositório

| Artefato | Caminho no repositório | Descrição |
|----------|----------------------|-----------|
| Guardrails operacionais | `docs/novatech/guardrails-operacionais-assistente-novatech-v3.md` | Fonte primária desta seção do AGENTS.md. Contém detalhamento de enforcement, exemplos e rastreabilidade para os 3 incidentes de referência. |
| Linguagem ubíqua | `docs/novatech/linguagem-ubiqua-anexo-a.md` | Glossário completo com conflitos identificados, sinônimos proibidos e notas de desambiguação. |
| Documentação NovaTech | `docs/novatech/anexo-a-documentacao-simulada-novatech.md` | Fonte de verdade: POL-001, PROC-042-v1, PROC-042-v2, SLA-2024, FAQ-Atendimento. |
| Estrutura do repositório | `docs/novatech/anexo-c-estrutura-repositorio.md` | Árvore de diretórios, convenções de organização e configuração MCP. |
| System prompt versionado | `prompts/system-prompt.md` | Prompt principal do assistente. Toda alteração deve ser registrada em `prompts/prompt-changelog.md` com data, autor, motivo e resultado esperado. |
| Spec de requisitos — Pipeline de ingestão | `specs/pipeline-ingestao/requirements.md` | Requisitos funcionais do pipeline RAG, incluindo R-CONT, R-RASTR, R-GAP e R-NF referenciados nos guardrails. |
| Spec de requisitos — Query endpoint | `specs/query-endpoint/requirements.md` | Requisitos do endpoint de consulta, incluindo schema de resposta e validações. |
| Spec de requisitos — Feedback API | `specs/feedback-api/requirements.md` | Requisitos da API de coleta de feedback do atendente (GR-D-010). |
| Skills de domínio | `skills/domain/azure-ai-search-integration.md` | Padrões de integração com Azure AI Search — re-ranking por metadados, filtros de `status`, limiar de similaridade. |
| Skills de domínio | `skills/domain/azure-functions-endpoint.md` | Padrões de implementação do endpoint HTTP com validação Zod e tratamento de erros. |
| Golden queries | `prompts/eval/golden-queries.json` | Perguntas de referência para avaliação do assistente, cobrindo os cenários dos 3 incidentes de referência (INC-01, INC-02, INC-03). |

#### 5.1. Conexão com Incidentes de Referência

Os três incidentes que motivaram os guardrails desta seção estão documentados em `guardrails-operacionais-assistente-novatech-v3.md`. Todo desenvolvedor deve lê-los antes de implementar ou modificar qualquer componente do pipeline.

| Incidente | Causa raiz | Guardrails primários | Onde testar |
|-----------|-----------|---------------------|-------------|
| INC-01 — Prazo de 7 dias para carga perigosa | Extrapolação indevida de regra geral sem considerar seção de exceções | GR-N-001, GR-D-004, GR-W-003 | `prompts/eval/golden-queries.json` → query sobre devolução de carga perigosa |
| INC-02 — Multiplicadores da v1 em chamado pós-01/12/2023 | Ausência de re-ranking por `status=vigente` e filtragem de obsoletos | GR-D-002, GR-D-005, GR-N-002 | `prompts/eval/golden-queries.json` → query sobre frete especial região Norte |
| INC-03 — "Não encontrei" para SLA Gold com documento indexado | Score abaixo do limiar por provável falha de chunking de tabelas | GR-D-007, GR-D-011, GR-N-007 | `prompts/eval/golden-queries.json` → query sobre SLA cliente Gold |

> **Decisões técnicas pendentes com impacto direto nos guardrails:** DTP-001 (estratégia de chunking — risco específico em tabelas como SLA-2024) e DTP-002 (seleção e versionamento do modelo LLM) estão registradas como bloqueadoras para a resolução definitiva de INC-03. Consultar `specs/pipeline-ingestao/plan.md` antes de implementar o chunker (`src/pipeline/chunker.ts`).

---

*Product Rules & Guardrails — NovaTech Assistant · Versão 1.0 · Junho/2025*  
*Derivado de: Guardrails Operacionais v3.0 · Linguagem Ubíqua v1.0 · Documentação NovaTech (Anexo A)*  
*Classificação: Confidencial — uso interno*
