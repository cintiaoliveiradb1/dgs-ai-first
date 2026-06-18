# Mapeamento de Guardrails × Incidentes
## NovaTech Logística — Assistente de Atendimento IA

| Campo | Valor |
|-------|-------|
| Versão | 1.0 |
| Data de elaboração | Junho/2025 |
| Documentos de referência | Guardrails Operacionais v1.0; Classificação de Enforcement v1.0; Incidentes Internos (3 casos) |
| Responsável | AI Governance Specialist |

---

## Incidentes de referência

| ID | Descrição do incidente |
|----|------------------------|
| **INC-01** | O assistente respondeu que o prazo de devolução para carga perigosa é 7 dias, quando na verdade cargas perigosas **não podem** ser devolvidas pelo processo padrão. |
| **INC-02** | O assistente citou "PROC-042, seção 2" mas os multiplicadores informados eram da versão 1 (desatualizada), não da v2 (vigente). |
| **INC-03** | O assistente disse "Não encontrei informação sobre isso" para uma pergunta sobre SLA Gold, sendo que o documento SLA-2024 estava indexado e continha a resposta. |

---

## Análise dos incidentes: o que falhou em cada caso

Antes do mapeamento, é necessário decompor cada incidente em suas causas-raiz, pois cada um revela falhas em camadas distintas do pipeline.

### INC-01 — Prazo incorreto para carga perigosa
**Causa-raiz primária:** O modelo generalizou a regra geral de 7 dias úteis (POL-001, seção 3.1) sem aplicar a exceção da seção 3.2, que exclui cargas perigosas do processo padrão. A resposta mesclou informação de uma seção com omissão deliberada de outra — padrão clássico de recuperação parcial ou de geração sem ancoragem nos chunks completos.

**Causas secundárias:** ausência de sinalização explícita de inelegibilidade, ausência de citação que teria permitido ao atendente verificar a seção correta, ausência de rastreabilidade para auditoria retroativa.

### INC-02 — Multiplicadores da versão errada do PROC-042
**Causa-raiz primária:** O pipeline não filtrou os chunks por versão vigente antes de enviá-los ao LLM — ou o LLM recebeu chunks de ambas as versões e utilizou os da v1 sem detectar o conflito. A seleção da versão correta não foi tratada como operação determinística.

**Causas secundárias:** ausência de sinalização de conflito entre versões, ausência de verificação da data do chamado, citação incorreta que atribuiu os valores da v1 à referência genérica "PROC-042" sem indicar a versão.

### INC-03 — Falso negativo para pergunta coberta pela base
**Causa-raiz primária:** Falha no retrieval — o assistente não recuperou chunks relevantes do SLA-2024 apesar de o documento estar indexado. As causas possíveis são: threshold de similaridade mal calibrado (descartou chunks relevantes), estratégia de chunking inadequada para tabelas (as tabelas de SLA podem ter sido fragmentadas de forma que o texto recuperado perdeu contexto suficiente para atingir score ≥ 0,75), ou classificação incorreta da pergunta como gap ou fora de escopo.

**Causas secundárias:** resposta de gap sem classificação correta do tipo (o assistente deveria ter distinguido "não encontrei" de "gap documental" de "fora de escopo"), ausência de mecanismo de fallback que verificasse se o tema está coberto pela base antes de emitir resposta negativa.

---

## Mapeamento guardrail × incidente

### Leitura da tabela

- **Previne diretamente:** o guardrail, se implementado, interromperia ou corrigiria a falha que gerou o incidente.
- **Previne indiretamente:** o guardrail não impede a falha em si, mas limita seu impacto — por exemplo, permitindo detecção posterior ou alertando o atendente de que algo pode estar errado.
- **—** O guardrail não tem relação causal com o incidente.

| ID | Nome do Guardrail | INC-01 | INC-02 | INC-03 |
|----|-------------------|--------|--------|--------|
| GR-001 | Citar fonte oficial em toda resposta | Indireto | Direto | Indireto |
| GR-002 | Sinalizar fonte informal (FAQ) | — | — | — |
| GR-003 | Usar versão vigente do PROC-042 conforme data do chamado | — | Direto | — |
| GR-004 | Sinalizar conflito entre documentos | — | Direto | — |
| GR-005 | Classificar resposta negativa por tipo de gap | — | — | Direto |
| GR-006 | Solicitar dados faltantes antes de calcular frete ou SLA | — | Direto | — |
| GR-007 | Exibir trecho original que embasou a resposta | Indireto | Indireto | Indireto |
| GR-008 | Coletar feedback do atendente ao final de cada resposta | Indireto | Indireto | Indireto |
| GR-009 | Registrar log completo de cada interação | Indireto | Indireto | Indireto |
| GR-010 | Exibir mensagem diferenciada em modo degradado | — | — | — |
| GR-011 | Não fabricar ou extrapolar informações não documentadas | Direto | — | — |
| GR-012 | Não usar FAQ informal como fonte primária | — | — | — |
| GR-013 | Não apresentar gaps documentais como fatos confirmados | — | — | — |
| GR-014 | Não confirmar tiers inexistentes | — | — | — |
| GR-015 | Usar trecho com score de similaridade ≥ 0,75 | — | — | Direto |
| GR-016 | Não aplicar multiplicadores do PROC-042 sem verificar versão | — | Direto | — |
| GR-017 | Solicitar data do chamado em consulta de frete especial | — | Direto | — |
| GR-018 | Descartar FAQ na janela de atualização | — | — | — |
| GR-019 | Reconhecer carga não elegível para devolução padrão | Direto | — | — |
| GR-020 | Reconhecer gap documental | — | — | Direto |

---

## Detalhamento por guardrail

---

### GR-001 — Citar fonte oficial em toda resposta

| Incidente | Relação | Explicação |
|-----------|---------|------------|
| INC-01 | **Indireto** | Se o assistente tivesse citado "POL-001, v3.1, seção 3.1", o atendente poderia verificar a seção e descobrir a exceção da seção 3.2. A citação não impede a resposta errada, mas cria o caminho para a detecção imediata. |
| INC-02 | **Direto** | O assistente citou "PROC-042, seção 2" sem indicar a versão. Se GR-001 estivesse em vigor, a citação obrigatória incluiria a versão do documento — o que teria exposto imediatamente que os valores eram da v1, não da v2. A falha teria sido detectável pelo próprio atendente ao conferir a fonte. |
| INC-03 | **Indireto** | Uma resposta negativa com citação de fonte ("não encontrei informação em SLA-2024, seção X") seria imediatamente questionável pelo atendente, pois ele saberia que o documento existe. A ausência de citação mascarou a falha de retrieval como se fosse ausência de cobertura. |

---

### GR-002 — Sinalizar fonte informal (FAQ)

| Incidente | Relação | Explicação |
|-----------|---------|------------|
| INC-01 | — | A resposta incorreta sobre carga perigosa não se originou do FAQ — a informação de 7 dias consta na POL-001 (seção 3.1). O problema foi omissão da exceção, não uso de fonte informal. |
| INC-02 | — | O PROC-042 é documento oficial em ambas as versões. O incidente não envolveu o FAQ. |
| INC-03 | — | O SLA-2024 é documento oficial indexado. O incidente foi de retrieval, não de fonte informal. |

> **Observação:** GR-002 não previne nenhum dos três incidentes porque todos originaram-se de falhas em documentos oficiais, não no FAQ. Isso não reduz sua importância — o guardrail previne uma classe distinta de incidente (confusão entre prática informal e política oficial) que não foi capturada nesta amostra.

---

### GR-003 — Usar versão vigente do PROC-042 conforme data do chamado

| Incidente | Relação | Explicação |
|-----------|---------|------------|
| INC-02 | **Direto** | Este é o guardrail central para o INC-02. Se o pipeline filtrasse os chunks do PROC-042 pela data do chamado antes de enviá-los ao LLM, o modelo nunca teria recebido os multiplicadores da v1 para um chamado vigente sob a v2. A falha seria estruturalmente impossível com GR-003 implementado deterministicamente. |

---

### GR-004 — Sinalizar conflito entre documentos

| Incidente | Relação | Explicação |
|-----------|---------|------------|
| INC-02 | **Direto** | Se o pipeline detectasse que dois chunks com valores divergentes de multiplicadores regionais foram recuperados (um da v1, outro da v2), o bloco de aviso de conflito seria gerado — e o atendente veria explicitamente que existe uma versão mais antiga com valores diferentes. Isso teria impedido o uso silencioso dos valores incorretos. GR-004 funciona como segunda linha de defesa quando GR-003 falha. |

---

### GR-005 — Classificar resposta negativa por tipo de gap

| Incidente | Relação | Explicação |
|-----------|---------|------------|
| INC-03 | **Direto** | O assistente respondeu "Não encontrei informação sobre isso" — uma mensagem genérica de ausência que não distingue entre gap documental, fora de escopo e falha de retrieval. Com GR-005, antes de emitir qualquer resposta negativa o pipeline verificaria se o tema "SLA Gold" está coberto pela base (está — SLA-2024 está indexado com `tipo=contratual`), o que classificaria a pergunta como categoria 1 (coberta) e impediria a resposta negativa. O guardrail força a verificação que o incidente mostrou estar ausente. |

---

### GR-006 — Solicitar dados faltantes antes de calcular frete ou SLA

| Incidente | Relação | Explicação |
|-----------|---------|------------|
| INC-02 | **Direto** | A data de abertura do chamado é um dado obrigatório para a seleção da versão correta do PROC-042. Se GR-006 estivesse em vigor e a data não tivesse sido fornecida, o assistente teria interrompido o fluxo e solicitado a informação — impedindo o cálculo com versão incorreta. GR-006 é a primeira linha de defesa; GR-003 e GR-016 são as linhas seguintes. |

---

### GR-007 — Exibir trecho original que embasou a resposta

| Incidente | Relação | Explicação |
|-----------|---------|------------|
| INC-01 | **Indireto** | Se o trecho exibido fosse apenas o da seção 3.1 (regra geral de 7 dias), um atendente familiarizado com a POL-001 poderia notar a ausência do contexto da seção 3.2 e questionar a completude da resposta. |
| INC-02 | **Indireto** | O trecho exibido conteria os multiplicadores da v1. Um atendente que conhecesse os valores da v2 os identificaria como incorretos ao comparar com o documento original. |
| INC-03 | **Indireto** | A ausência de trecho exibido seria em si um sinal de alerta — se o assistente diz "não encontrei" mas deveria ter encontrado, a falta de trecho tornaria a resposta negativa auditável e questionável. |

> **Observação:** GR-007 tem papel consistentemente indireto nos três incidentes — não previne as falhas, mas cria condições para detecção pelo atendente ou em auditoria retroativa.

---

### GR-008 — Coletar feedback do atendente ao final de cada resposta

| Incidente | Relação | Explicação |
|-----------|---------|------------|
| INC-01 | **Indireto** | Um atendente que percebesse a inconsistência da resposta (ao consultar a POL-001 após o atendimento) poderia marcar "Incorreta" — gerando registro para correção da base ou do prompt. |
| INC-02 | **Indireto** | Idem: atendente que identificasse o multiplicador errado poderia sinalizar, acelerando a detecção do problema sistêmico de versionamento. |
| INC-03 | **Indireto** | Atendente que soubesse que o SLA-2024 existe e contém a resposta marcaria "Incorreta", alertando a equipe de produto para investigar a falha de retrieval. |

> **Observação:** GR-008 não previne os incidentes — atua como mecanismo de detecção e melhoria contínua pós-ocorrência. Sua ausência nos testes internos significa que as falhas não geraram sinal de correção automático.

---

### GR-009 — Registrar log completo de cada interação

| Incidente | Relação | Explicação |
|-----------|---------|------------|
| INC-01 | **Indireto** | Com o log disponível, seria possível verificar retroativamente quais chunks foram recuperados — e confirmar se a seção 3.2 (exceções) foi ou não recuperada. Isso direciona a correção: falha de retrieval ou de geração. |
| INC-02 | **Indireto** | O log registraria quais chunks foram usados e seus scores. Seria possível confirmar se chunks da v1 foram recuperados junto com os da v2, e se o re-ranking funcionou corretamente. |
| INC-03 | **Indireto** | O log mostraria o score dos chunks recuperados para "SLA Gold". Se o score ficou abaixo de 0,75, o problema é de threshold ou chunking. Se nenhum chunk foi recuperado, o problema é de indexação. Sem o log, a causa-raiz do INC-03 é opaca. |

> **Observação:** GR-009 é o instrumento de diagnóstico dos três incidentes. Sua ausência (ou inacessibilidade) transforma falhas corrigíveis em falhas opacas.

---

### GR-010 — Exibir mensagem diferenciada em modo degradado

| Incidente | Relação | Explicação |
|-----------|---------|------------|
| INC-01 | — | O assistente estava operacional e gerou resposta — não era caso de modo degradado. |
| INC-02 | — | Idem. |
| INC-03 | — | A resposta negativa foi gerada pelo assistente em operação normal, não por indisponibilidade. |

> **Observação:** GR-010 não previne nenhum dos três incidentes. Tal como GR-002, protege contra uma classe diferente de falha — indisponibilidade sistêmica — não capturada nesta amostra.

---

### GR-011 — Não fabricar ou extrapolar informações não documentadas

| Incidente | Relação | Explicação |
|-----------|---------|------------|
| INC-01 | **Direto** | O assistente extrapolou a regra geral de 7 dias (seção 3.1) para uma categoria de carga que possui exceção explícita (seção 3.2). Isso é uma forma de extrapolação indevida: aplicar uma regra a um escopo além do documentado. GR-011 instrui o modelo a não generalizar além do que o documento estabelece — e a seção 3.2 estabelece explicitamente que a regra geral *não se aplica* a cargas perigosas. |

---

### GR-012 — Não usar FAQ informal como fonte primária

| Incidente | Relação | Explicação |
|-----------|---------|------------|
| INC-01 | — | A resposta não originou do FAQ. |
| INC-02 | — | Idem. |
| INC-03 | — | Idem. |

---

### GR-013 — Não apresentar gaps documentais como fatos confirmados

| Incidente | Relação | Explicação |
|-----------|---------|------------|
| INC-01 | — | O incidente não envolveu um gap documental — a POL-001 cobre o tema completo, incluindo as exceções. O problema foi omissão da exceção pelo modelo, não ausência de documento. |
| INC-02 | — | Idem — ambas as versões do PROC-042 estão indexadas. |
| INC-03 | — | O INC-03 é o inverso: o assistente negou informação que existe. GR-013 trata do oposto (afirmar informação que não existe formalmente). |

---

### GR-014 — Não confirmar tiers inexistentes

| Incidente | Relação | Explicação |
|-----------|---------|------------|
| INC-01 | — | Não envolve tiers. |
| INC-02 | — | Não envolve tiers. |
| INC-03 | — | O INC-03 trata de SLA Gold — tier existente e documentado. O guardrail protege contra tiers *inexistentes*. |

---

### GR-015 — Usar trecho com score de similaridade ≥ 0,75

| Incidente | Relação | Explicação |
|-----------|---------|------------|
| INC-03 | **Direto** | Uma das causas mais prováveis do INC-03 é threshold mal calibrado: o pipeline pode ter descartado chunks relevantes do SLA-2024 por score ligeiramente abaixo de 0,75, ou o chunking das tabelas de SLA pode ter produzido fragmentos com baixa densidade semântica. GR-015 endereça o sintoma (threshold), mas o incidente também aponta para DTP-001 (estratégia de chunking de tabelas) como causa estrutural subjacente que GR-015 sozinho não resolve. |

---

### GR-016 — Não aplicar multiplicadores do PROC-042 sem verificar versão correta

| Incidente | Relação | Explicação |
|-----------|---------|------------|
| INC-02 | **Direto** | Este guardrail é a versão negativa do GR-003 — proíbe explicitamente o que o INC-02 materializou: fornecer valores de multiplicadores sem verificar a versão aplicável ao chamado. É a regra de proibição que espelha a regra de obrigação do GR-003, funcionando como reforço semântico no prompt para o caso em que o filtro determinístico do GR-003 falhe. |

---

### GR-017 — Solicitar data do chamado em consulta de frete especial

| Incidente | Relação | Explicação |
|-----------|---------|------------|
| INC-02 | **Direto** | Se a data do chamado não estava disponível no contexto, GR-017 determinaria a interrupção do cálculo e a solicitação da data ao atendente. Com a data em mãos, GR-003 selecionaria a versão correta. GR-017 é a pré-condição que garante que GR-003 sempre terá os dados necessários para funcionar. |

---

### GR-018 — Descartar FAQ na janela de atualização

| Incidente | Relação | Explicação |
|-----------|---------|------------|
| INC-01 | — | A resposta não originou do FAQ. |
| INC-02 | — | Idem. |
| INC-03 | — | Idem. |

---

### GR-019 — Reconhecer carga não elegível para devolução padrão

| Incidente | Relação | Explicação |
|-----------|---------|------------|
| INC-01 | **Direto** | Este é o guardrail mais diretamente ligado ao INC-01. Ele instrui o assistente a identificar categorias de carga inelegíveis (entre elas, cargas perigosas classes 1 a 6, conforme POL-001, seção 3.2) e encaminhá-las ao ramal 4500 em vez de aplicar o prazo geral de 7 dias. Se GR-019 estivesse operacional, a resposta seria: "Cargas perigosas não são elegíveis para devolução pelo processo padrão" — não "o prazo é de 7 dias". |

---

### GR-020 — Reconhecer gap documental

| Incidente | Relação | Explicação |
|-----------|---------|------------|
| INC-03 | **Direto** | O INC-03 é um caso em que o assistente emitiu resposta negativa genérica ("Não encontrei informação") para um tema coberto pela base. GR-020 força o pipeline a classificar a pergunta antes de responder negativamente: se "SLA Gold" não é um gap documentado na seção 2.3 *e* o documento SLA-2024 está indexado como `tipo=contratual`, a classificação como gap é inválida e o fluxo deve tentar nova recuperação ou alertar para falha de retrieval, em vez de responder "não encontrei". |

---

## Visão consolidada por incidente

### INC-01 — Prazo incorreto para carga perigosa
**Guardrails que previnem diretamente:** GR-011, GR-019
**Guardrails que limitam o impacto (indiretos):** GR-001, GR-007, GR-008, GR-009

**Cadeia de falha e cobertura:**

```
Pergunta sobre devolução de carga perigosa
        │
        ▼
[GR-019] Verificar elegibilidade da categoria → NÃO ELEGÍVEL → encaminhar ramal 4500
        │ (se GR-019 falhar)
        ▼
[GR-011] Não extrapolar regra geral para categoria com exceção explícita
        │ (se GR-011 falhar)
        ▼
[GR-001] Citar seção utilizada → atendente pode detectar ausência da seção 3.2
[GR-007] Exibir trecho → torna omissão da exceção visível
        │ (se todos falharem)
        ▼
[GR-008] Feedback "Incorreta" → sinaliza para correção
[GR-009] Log com chunks → diagnóstico da causa-raiz
```

**Diagnóstico do incidente:** Duas linhas de defesa estavam ausentes simultaneamente (GR-019 e GR-011). O incidente é de **omissão de exceção** — o modelo aplicou a regra geral sem verificar as condições de inelegibilidade explícitas no mesmo documento.

---

### INC-02 — Multiplicadores da versão errada do PROC-042
**Guardrails que previnem diretamente:** GR-003, GR-004, GR-006, GR-016, GR-017
**Guardrails que limitam o impacto (indiretos):** GR-001, GR-007, GR-008, GR-009

**Cadeia de falha e cobertura:**

```
Pergunta sobre frete especial
        │
        ▼
[GR-017] Data do chamado presente no contexto? → NÃO → solicitar data
        │ (se data presente ou após informá-la)
        ▼
[GR-003] Filtrar chunks pelo status/data do chamado → apenas v2 no contexto
        │ (se filtro falhar e ambas as versões chegarem ao LLM)
        ▼
[GR-004] Detectar conflito de multiplicadores → sinalizar ao atendente
[GR-016] Instrução explícita: não usar valores sem verificar versão
        │ (se sinalização for ignorada e modelo usar v1)
        ▼
[GR-001] Citar versão → expõe que os valores são da v1
[GR-006] Verificar campos obrigatórios → (redundância com GR-017)
        │ (se todos falharem)
        ▼
[GR-008] Feedback "Incorreta" → sinaliza erro financeiro
[GR-009] Log com chunks e scores → identifica qual versão foi usada
```

**Diagnóstico do incidente:** É o incidente com mais guardrails preventivos (5 diretos). O fato de ter ocorrido indica que nenhuma das verificações de versionamento estava implementada — o pipeline entregou ambas as versões ao modelo sem distinção, e o modelo usou a mais antiga sem detectar o conflito.

---

### INC-03 — Falso negativo para pergunta coberta pela base
**Guardrails que previnem diretamente:** GR-005, GR-015, GR-020
**Guardrails que limitam o impacto (indiretos):** GR-001, GR-007, GR-008, GR-009

**Cadeia de falha e cobertura:**

```
Pergunta sobre SLA Gold
        │
        ▼
[GR-015] Score dos chunks recuperados ≥ 0,75? → SIM → usar na resposta
        │ (se nenhum chunk atingir o threshold)
        ▼
[GR-020] Tema é gap documentado na seção 2.3? → NÃO (SLA-2024 está indexado)
         → não emitir resposta de gap; verificar falha de retrieval
        │ (se classificação for incorreta e resposta negativa for emitida)
        ▼
[GR-005] Classificar tipo de resposta negativa:
         → "SLA Gold" está coberto → categoria 1 (coberta) → não emitir negativa
        │ (se todos falharem e resposta negativa for emitida mesmo assim)
        ▼
[GR-001] Citar fonte → "não encontrei em SLA-2024, seção X" é questionável
[GR-007] Ausência de trecho → sinal de alerta visível
        │ (se todos falharem)
        ▼
[GR-008] Feedback "Incorreta" → atendente que conhece o SLA sinaliza
[GR-009] Log → mostra score dos chunks; revela se é problema de threshold ou chunking
```

**Diagnóstico do incidente:** Falha de retrieval mascarada como ausência de cobertura. O INC-03 aponta para DTP-001 (chunking de tabelas) como causa estrutural não coberta por nenhum guardrail — tabelas mal fragmentadas produzem chunks com baixa densidade semântica que não atingem o threshold independentemente de sua instrução em prompt.

---

## Guardrails sem cobertura nos três incidentes

Os guardrails abaixo não previnem nenhum dos três incidentes porque protegem contra classes de falha distintas, não representadas nesta amostra:

| ID | Nome | Classe de falha protegida (não observada nos incidentes) |
|----|------|----------------------------------------------------------|
| GR-002 | Sinalizar fonte informal (FAQ) | Uso de prática informal como política oficial |
| GR-010 | Exibir mensagem diferenciada em modo degradado | Indisponibilidade sistêmica do assistente |
| GR-012 | Não usar FAQ como fonte primária | Re-ranking incorreto entre fonte oficial e informal |
| GR-013 | Não apresentar gaps como fatos confirmados | Afirmação de informação sem respaldo formal (seguro de carga, carga danificada) |
| GR-014 | Não confirmar tiers inexistentes | Confirmação de tier "Platinum" ou similar |
| GR-018 | Descartar FAQ na janela de atualização | Janela de inconsistência entre ingestão primária e FAQ |

> **Implicação:** A ausência desses guardrails nos incidentes observados não os torna desnecessários — indica apenas que a amostra de três incidentes não cobre a totalidade dos vetores de falha possíveis.

---

## Lacuna identificada: falha não coberta por nenhum guardrail

O INC-03 expõe uma causa-raiz estrutural — chunking inadequado de tabelas — que nenhum dos 20 guardrails aborda diretamente. A DTP-001 (decisão técnica pendente sobre estratégia de chunking) é o único item da documentação que reconhece esse risco, mas permanece em aberto. Um falso negativo causado por fragmentação inadequada de tabelas numéricas (SLA, multiplicadores) não é evitável apenas por instrução em prompt ou por filtros de metadado — requer decisão técnica de arquitetura.

**Recomendação:** Incluir um guardrail técnico específico (sugerido: GR-021) que defina tratamento especial para tabelas no pipeline de chunking, como requisito de aceitação vinculado à resolução da DTP-001.

---

*Documento elaborado com base nos Guardrails Operacionais v1.0, na Classificação de Enforcement v1.0 e nos três incidentes internos reportados.*

*NovaTech Logística — Confidencial — Versão 1.0*
