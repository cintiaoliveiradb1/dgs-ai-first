# Classificação de Enforcement dos Guardrails Operacionais
## NovaTech Logística — Assistente de Atendimento IA

| Campo | Valor |
|-------|-------|
| Versão | 1.0 |
| Data de elaboração | Junho/2025 |
| Documento de referência | Guardrails Operacionais do Assistente v1.0 |
| Responsável | AI Governance Specialist |

---

## Critério de classificação

| Tipo | Definição | Característica central |
|------|-----------|----------------------|
| **Determinístico (via código)** | A regra é aplicada por lógica programada no pipeline — filtros, verificações de metadados, thresholds numéricos, bloqueios condicionais — sem depender da geração do LLM para ser cumprida. O comportamento é garantido independentemente do que o modelo produziria. | **Verificável e executável antes ou depois da geração do texto** |
| **Probabilístico (via prompt)** | A regra é comunicada ao LLM por instrução em prompt (sistema ou contexto). O cumprimento depende do comportamento emergente do modelo — que pode variar conforme o phrasing da pergunta, temperatura, versão do modelo ou contexto da conversa. | **Depende da interpretação e obediência do modelo ao seguir instruções** |

> **Nota metodológica:** A classificação considera a camada em que o enforcement é *primariamente* operacionalizado. Guardrails determinísticos também se beneficiam de instrução em prompt para casos-borda, e guardrails probabilísticos podem ter camadas de código como suporte — mas a classificação reflete o mecanismo que garante (ou não) a regra de forma robusta.

---

## Tabela resumo

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

---

## Classificações detalhadas

---

### GR-001 — Citar fonte oficial em toda resposta
**Classificação: Probabilístico (com suporte determinístico parcial)**

**Justificativa:**
A instrução de citar documento, versão, seção e data depende do LLM para ser executada — é o modelo que decide incluir ou não a citação no texto gerado, e que formata corretamente os campos exigidos. O pipeline pode fornecer os metadados dos chunks recuperados (nome do documento, versão, seção) como contexto obrigatório no prompt, o que é determinístico, mas a incorporação desses metadados na resposta final como citação explícita permanece uma instrução ao modelo. Um LLM pode omitir a citação, citá-la parcialmente ou formatá-la de forma diferente da especificada — sem que nenhum bloqueio de código seja acionado.

**Risco residual:** O modelo pode gerar resposta correta em conteúdo mas omitir a citação, especialmente em respostas curtas ou quando o contexto da pergunta é muito direto. Requer monitoramento por amostragem ou verificação por parser pós-geração (ex.: checar se o padrão "Fonte: [doc]" está presente na saída).

**Mitigação recomendada:** Implementar validador de saída (output parser) que verifique se a resposta contém ao menos um bloco de citação formatado. Respostas sem citação são descartadas e regeneradas ou sinalizadas para revisão.

---

### GR-002 — Sinalizar fonte informal (FAQ)
**Classificação: Determinístico**

**Justificativa:**
O metadado `tipo=informal` é atribuído ao FAQ no momento da indexação (F-005, SPEC v1.3). O pipeline, ao montar o contexto da resposta, pode verificar programaticamente se algum chunk recuperado possui esse metadado e, em caso afirmativo, injetar automaticamente o bloco de aviso de fonte informal no payload enviado ao LLM — ou anexá-lo à resposta no pós-processamento, antes da exibição ao atendente. A sinalização não precisa depender do LLM para ser executada: é uma operação de template sobre metadado conhecido.

**Condição de determinismo:** O enforcement é determinístico desde que o pipeline implemente a lógica de verificação de metadado e injeção do aviso *fora* do prompt de instrução ao modelo. Se a sinalização for apenas instruída via prompt ("sempre avise quando usar o FAQ"), degrada para probabilístico.

---

### GR-003 — Usar versão vigente do PROC-042 conforme data do chamado
**Classificação: Determinístico**

**Justificativa:**
A regra de seleção de versão é inteiramente baseada em comparação de datas: chamado ≤ 30/11/2023 → PROC-042-v1; chamado ≥ 01/12/2023 → PROC-042-v2. Essa lógica é implementável como filtro de metadado no momento da recuperação (retrieval filtering): o pipeline compara a data do chamado com os metadados `vigente_até` e `data` dos documentos e restringe os chunks elegíveis antes mesmo de enviar o contexto ao LLM. O modelo nunca recebe chunks da versão incorreta, eliminando a possibilidade de confusão.

**Condição de determinismo:** Depende de a data do chamado estar disponível no contexto. Quando ausente, o guardrail GR-017 (também determinístico) intercepta a consulta antes do cálculo.

---

### GR-004 — Sinalizar conflito entre documentos
**Classificação: Determinístico (com componente probabilístico na redação do aviso)**

**Justificativa:**
A detecção de conflito é determinística: o pipeline pode identificar programaticamente quando dois chunks recuperados possuem metadados de versão ou status distintos para o mesmo tema (mesmo documento-base, versões diferentes). A decisão de sinalizar e qual versão usar (status vigente > status transição; data mais recente como desempate) é uma regra de negócio codificável. O bloco de aviso padrão pode ser gerado por template com preenchimento dos campos `documento_A`, `documento_B`, `campo_conflitante` e `regra_de_desempate`.

**Componente probabilístico:** A formulação da frase de aviso integrada ao fluxo da resposta — tornando-a legível e contextualizada para o atendente — depende do LLM. O template resolve o *conteúdo* do aviso; a *integração fluida* com o restante da resposta é geração de linguagem.

---

### GR-005 — Classificar resposta negativa por tipo de gap
**Classificação: Determinístico (parcialmente) + Probabilístico (na classificação de escopo)**

**Justificativa:**
A distinção entre **gap documental** e **fora de escopo** possui dois momentos com naturezas diferentes:

- **Gap documental** (temas da seção 2.3 da SPEC): é determinístico. O pipeline mantém uma lista estática de temas mapeados como gaps (carga danificada, seguro de carga, frete padrão, Gestão de Riscos). Se a pergunta não retornar chunks com score ≥ 0,75 *e* o tema for reconhecido nessa lista, o pipeline roteia para a mensagem de gap documental sem precisar do LLM para essa classificação.
- **Fora de escopo** (temas logisticamente alheios): é probabilístico. A determinação de que uma pergunta "não pertence ao domínio de operações logísticas da NovaTech" é um julgamento semântico que o LLM executa — não há lista exaustiva de temas fora do escopo que o código possa verificar deterministicamente para todos os casos possíveis.

---

### GR-006 — Solicitar dados faltantes antes de calcular frete ou SLA
**Classificação: Determinístico**

**Justificativa:**
Os parâmetros obrigatórios para cálculo de frete especial (peso, região, data do chamado) e para consulta de SLA (tier do cliente) são campos verificáveis no contexto da conversa antes da geração. O pipeline pode implementar um verificador de pré-condições: se o intent da pergunta for classificado como "cálculo de frete" ou "consulta de SLA" e os campos obrigatórios estiverem ausentes no contexto, interrompe o fluxo e injeta uma mensagem de solicitação de dados — sem acionar o LLM para a resposta principal. Essa lógica é equivalente a validação de formulário: campos obrigatórios faltando bloqueiam o envio.

**Condição de determinismo:** Depende de o pipeline possuir classificação de intent (que pode ser feita por um modelo leve ou por regras de palavras-chave para os tópicos cobertos).

---

### GR-007 — Exibir trecho original que embasou a resposta
**Classificação: Determinístico**

**Justificativa:**
Os chunks recuperados existem como objetos com texto e metadados no pipeline *antes* da geração da resposta. O trecho original pode ser exibido pela interface diretamente a partir do objeto recuperado, sem depender do LLM para transcrevê-lo ou selecioná-lo. A lógica de exibição (bloco recuado, rótulo com documento e seção, resumo para trechos > 150 palavras) é implementável como renderização de template sobre os dados de recuperação. O LLM não precisa ser a origem do trecho — apenas da resposta síntese.

**Componente probabilístico residual:** A decisão de *qual* trecho entre os recuperados corresponde a cada afirmação específica da resposta pode exigir rastreabilidade por sentença (claim-level attribution), que é mais complexa e pode requerer instrução ao modelo.

---

### GR-008 — Coletar feedback do atendente ao final de cada resposta
**Classificação: Determinístico**

**Justificativa:**
O mecanismo de coleta de feedback é inteiramente responsabilidade da camada de interface e do pipeline de dados — não do LLM. A interface renderiza os botões "Útil / Incorreta / Incompleta" após cada resposta por lógica de UI, registra a seleção no banco de dados e expõe no painel de administração. O LLM não participa desse fluxo. Trata-se de uma funcionalidade de produto, não de instrução ao modelo.

---

### GR-009 — Registrar log completo de cada interação
**Classificação: Determinístico**

**Justificativa:**
O registro de log (timestamp, ID do atendente, pergunta, chunks recuperados com scores, resposta gerada, fontes citadas) é uma operação de infraestrutura executada pelo pipeline a cada interação, independentemente do conteúdo gerado pelo LLM. Controle de acesso (restrito à equipe de segurança), prazo de retenção (mínimo 90 dias) e conformidade com a LGPD são políticas de dados implementadas na camada de armazenamento. Nenhum desses elementos depende do comportamento do modelo.

---

### GR-010 — Exibir mensagem diferenciada em modo degradado
**Classificação: Determinístico**

**Justificativa:**
O estado de disponibilidade do sistema (operacional / falha / atualização planejada) é monitorado pela infraestrutura, não pelo LLM. Quando o assistente está indisponível, o LLM não está sendo invocado — portanto a mensagem ao atendente é necessariamente gerada por lógica de sistema (página de erro, health check, mensagem de manutenção). As duas mensagens distintas (falha vs. atualização planejada) são templates estáticos exibidos condicionalmente pelo sistema operacional do pipeline.

---

### GR-011 — Fabricar ou extrapolar informações não documentadas
**Classificação: Probabilístico**

**Justificativa:**
Esta é a proibição mais estruturalmente dependente do comportamento do LLM. "Não fabricar" é uma instrução de alinhamento ao modelo — e a tendência de alucinação é uma propriedade emergente dos modelos de linguagem que nenhum filtro de código previne completamente de forma antecipada. O pipeline pode reduzir o risco estruturalmente (fornecendo apenas chunks recuperados como contexto, sem acesso à internet, com instrução de "responda apenas com base nos documentos fornecidos"), mas não pode garantir deterministicamente que o modelo não extrapole, especialmente em perguntas ambíguas ou quando os chunks recuperados são tangencialmente relacionados.

**Mitigação recomendada:** Combinar instrução de prompt rigorosa ("responda exclusivamente com base nos trechos fornecidos; se a informação não estiver presente, declare isso") com verificação pós-geração por NLI (Natural Language Inference) que compare a resposta gerada com os chunks recuperados e sinalize afirmações sem suporte documental.

---

### GR-012 — Usar FAQ informal como fonte primária quando há documento oficial
**Classificação: Determinístico (primariamente)**

**Justificativa:**
O re-ranking pós-recuperação (F-006, SPEC v1.3) é uma etapa de código: após recuperar os chunks mais relevantes, o pipeline compara os scores e, em caso de empate entre chunk de fonte primária (`tipo=normativo/procedimento/contratual`) e chunk de fonte secundária (`tipo=informal`), seleciona o primário por regra explícita. Quando o score da fonte primária é significativamente superior, ela prevalece por relevância — também por código. O LLM recebe o contexto já filtrado e ordenado pelo re-ranking, sem ver os chunks secundários descartados.

**Limite do determinismo:** Se o chunk do FAQ tiver score *significativamente mais alto* que o documento oficial (cenário de relevância semântica superior), o re-ranking por código entrega o FAQ ao LLM. A instrução ao modelo de sempre preferir fontes oficiais em caso de conflito de conteúdo — não de score — permanece probabilística.

---

### GR-013 — Apresentar informações de gaps documentais como fatos confirmados
**Classificação: Probabilístico**

**Justificativa:**
Esta regra exige que o LLM faça uma distinção epistêmica: diferenciar "informação presente em documento oficial" de "informação presente apenas em FAQ informal sobre tema sem cobertura formal". Essa distinção é comunicada via instrução de prompt e depende do modelo seguir o alinhamento corretamente. O risco é particularmente alto porque o FAQ-Item 38 (carga danificada) e o FAQ-Item 22 (seguro de carga) contêm informações plausíveis e específicas (prazos, percentuais) que um LLM pode reproduzir com alto grau de confiança aparente, mesmo instruído a não fazê-lo. Não há filtro de código que diferencie "informação verdadeira mas sem respaldo formal" de "informação com respaldo formal" — essa distinção é semântica e contextual.

**Mitigação recomendada:** Para os temas de gap documentados, implementar filtro de metadado que exclua chunks de fonte informal da composição da resposta quando o tema for identificado como gap total — forçando o fluxo GR-020 (determinístico) antes que o LLM receba qualquer conteúdo sobre o tema.

---

### GR-014 — Confirmar tiers inexistentes ou políticas não documentadas
**Classificação: Probabilístico**

**Justificativa:**
A negação de entidades inexistentes (como o tier "Platinum") depende do LLM reconhecer que não há documento indexado que suporte a existência dessa entidade e recusar a descrição. Isso é uma instrução de comportamento ao modelo — e LLMs podem confirmar inadvertidamente entidades não documentadas especialmente quando o usuário as apresenta com convicção ("meu contrato Platinum diz que..."), pois tendem a ser cooperativos e podem interpretar o contexto como uma premissa válida. Não há bloqueio de código que intercepte uma pergunta sobre "tier Platinum" antes de ela chegar ao modelo.

**Mitigação recomendada:** Para entidades de alto risco (tiers de cliente, produtos contratuais), manter uma lista de entidades válidas verificável por código no pré-processamento, que injete no contexto: "Os únicos tiers existentes são Gold, Silver e Standard. Qualquer menção a outro tier deve ser tratada como erro do cliente."

---

### GR-015 — Usar trecho com score de similaridade abaixo de 0,75
**Classificação: Determinístico**

**Justificativa:**
O threshold de similaridade (0,75) é um parâmetro numérico aplicado pelo pipeline no momento da recuperação, antes da geração. Chunks com score abaixo do limiar são descartados da lista de contexto antes de serem enviados ao LLM. O modelo nunca vê os chunks descartados — não há possibilidade de violação por comportamento do modelo. A regra é inteiramente controlada pela lógica de filtragem do retriever.

**Observação operacional:** O valor 0,75 deve ser validado e potencialmente ajustado durante a homologação (R-GAP-002b, SPEC v1.3). A classificação como determinístico é independente do valor escolhido — o mecanismo de threshold permanece determinístico em qualquer valor.

---

### GR-016 — Aplicar multiplicadores do PROC-042 sem verificar a versão correta
**Classificação: Determinístico**

**Justificativa:**
Esta regra é o espelho negativo do GR-003. Como a seleção da versão correta do PROC-042 é feita por filtro de metadado no retrieval (determinístico), a violação deste guardrail é prevenida pela mesma lógica: o pipeline nunca fornece ao LLM chunks das duas versões simultaneamente para o mesmo chamado, eliminando a possibilidade de mistura de valores. A regra é cumprida como consequência direta da implementação determinística do GR-003.

---

### GR-017 — Ausência de data do chamado em consulta de frete especial
**Classificação: Determinístico**

**Justificativa:**
A verificação de presença da data do chamado no contexto é uma checagem de campo obrigatório — equivalente a validação de parâmetro de entrada. O pipeline detecta programaticamente que o intent é "cálculo de frete especial" e que a data está ausente no contexto da conversa, então interrompe o fluxo antes de invocar o LLM para o cálculo e injeta a mensagem de solicitação de dado. A mensagem-padrão de solicitação pode ser um template estático. O LLM não precisa participar da decisão de interromper — apenas da formulação natural da solicitação, se desejável.

---

### GR-018 — Conflito entre FAQ e documento oficial na janela de atualização
**Classificação: Determinístico**

**Justificativa:**
A janela de inconsistência é detectável por comparação de timestamps: o pipeline sabe quando cada documento foi ingerido (log de ingestão, R-ATU-006) e pode verificar se o FAQ possui versão mais antiga que o documento primário para o mesmo tema. Quando detectado, o chunk do FAQ é excluído do contexto antes da geração e o bloco de aviso é injetado no contexto como instrução explícita ao modelo. A sinalização interna do item do FAQ como "pendente de revisão" é uma operação de banco de dados. Nenhum desses passos depende do LLM.

---

### GR-019 — Categoria de carga não elegível para devolução padrão
**Classificação: Probabilístico**

**Justificativa:**
A elegibilidade de uma categoria de carga para devolução padrão está documentada na POL-001, seção 3.2, mas a identificação da categoria da carga mencionada pelo atendente é uma tarefa de compreensão de linguagem natural. O atendente pode descrever a carga de formas variadas ("produto químico classe 5", "oxidante", "carga com temperatura controlada que ficou desligada", "lacre aberto na entrega") e o LLM precisa reconhecer que essas descrições correspondem a categorias não elegíveis. A decisão de encaminhar ao ramal 4500 em vez de processar como devolução padrão é um julgamento semântico do modelo, não uma verificação de campo estruturado.

**Mitigação recomendada:** Enriquecer os chunks da POL-001, seção 3.2, com exemplos de linguagem natural para cada categoria não elegível, aumentando a probabilidade de recuperação correta e de instrução contextual adequada ao LLM.

---

### GR-020 — Tema reconhecido como gap documental
**Classificação: Determinístico (primariamente) + Probabilístico (na detecção de novos gaps)**

**Justificativa:**
Para os quatro gaps documentados explicitamente na seção 2.3 da SPEC (carga danificada, seguro de carga, frete padrão abaixo de 500 kg, processo da Gestão de Riscos), a classificação é determinística: o pipeline mantém uma lista de temas-gap e, quando a pergunta não retorna chunks com score ≥ 0,75 *e* o tema é encontrado na lista, roteia para a mensagem padronizada de gap documental com indicação da área responsável.

**Componente probabilístico:** Para gaps ainda não mapeados na lista (temas que surgem após a publicação da SPEC), o reconhecimento de que o tema "pertence ao domínio NovaTech mas não possui cobertura" depende do julgamento semântico do LLM — o que é probabilístico por natureza.

---

## Síntese por perfil de risco

### Guardrails com enforcement determinístico (alta confiabilidade)
Estes guardrails podem ser considerados **garantidos pelo pipeline**, desde que a implementação técnica correspondente seja validada em testes de aceitação:

| ID | Nome | Mecanismo principal |
|----|------|---------------------|
| GR-002 | Sinalizar fonte informal (FAQ) | Verificação de metadado `tipo=informal` + template de aviso |
| GR-003 | Usar versão vigente do PROC-042 | Filtro de metadado `status` + `data` no retrieval |
| GR-007 | Exibir trecho original | Renderização de objeto de chunk na interface |
| GR-008 | Coletar feedback do atendente | Componente de UI pós-resposta |
| GR-009 | Registrar log completo | Middleware de logging por interação |
| GR-010 | Exibir mensagem diferenciada em modo degradado | Health check + templates de mensagem por estado |
| GR-015 | Usar trecho com score ≥ 0,75 | Filtro numérico no retriever |
| GR-016 | Não misturar versões do PROC-042 | Filtro de metadado (consequência do GR-003) |
| GR-017 | Solicitar data do chamado faltante | Validação de pré-condição de campo obrigatório |
| GR-018 | Descartar FAQ na janela de atualização | Comparação de timestamps de ingestão |

### Guardrails com enforcement probabilístico (requerem monitoramento contínuo)
Estes guardrails **não podem ser garantidos apenas por configuração** — exigem monitoramento por amostragem, avaliação de qualidade periódica e potencial ajuste de prompt ou arquitetura:

| ID | Nome | Principal vetor de falha |
|----|------|--------------------------|
| GR-001 | Citar fonte oficial em toda resposta | LLM pode omitir ou formatar incorretamente citações |
| GR-011 | Não fabricar informações | Alucinação é propriedade emergente do modelo |
| GR-013 | Não confirmar gaps como fatos | LLM pode reproduzir dados do FAQ com falsa autoridade |
| GR-014 | Não confirmar tiers inexistentes | LLM pode ser persuadido pela premissa do usuário |
| GR-019 | Reconhecer carga não elegível para devolução | Variabilidade na descrição da carga pelo atendente |

### Guardrails híbridos (determinístico + probabilístico)
Estes guardrails possuem **camada determinística que mitiga o risco principal**, mas mantêm componente probabilístico em casos-borda:

| ID | Nome | Determinístico | Probabilístico |
|----|------|----------------|----------------|
| GR-004 | Sinalizar conflito entre documentos | Detecção de chunks conflitantes por metadado | Redação integrada do aviso na resposta |
| GR-005 | Classificar resposta negativa por gap | Gaps mapeados → roteamento por lista | Classificação de "fora do escopo" |
| GR-006 | Solicitar dados faltantes | Validação de campos obrigatórios | Classificação de intent |
| GR-012 | Não usar FAQ como fonte primária | Re-ranking por tipo de fonte + score | Conflito de conteúdo sem conflito de score |
| GR-020 | Reconhecer gap documental | Gaps mapeados na seção 2.3 → lista | Novos gaps não mapeados |

---

## Recomendações prioritárias

Com base na análise de enforcement, três ações são prioritárias antes do go-live:

**1. Implementar output parser para GR-001**
Validador pós-geração que verifique presença de bloco de citação. Respostas sem citação devem ser rejeitadas ou sinalizadas automaticamente. Transforma parcialmente o guardrail probabilístico em verificável.

**2. Implementar filtro de pré-composição para GR-013**
Para os quatro temas de gap documental mapeados, excluir programaticamente chunks de fonte informal antes da composição do contexto — forçando o fluxo de gap total e impedindo que o LLM receba dados do FAQ sobre esses temas. Reduz significativamente o risco de confirmação inadvertida.

**3. Criar lista de entidades controladas para GR-014**
Manter lista de tiers válidos (Gold, Silver, Standard) e injetá-la como contexto obrigatório em toda sessão. Adicionar verificação de entidade no pré-processamento para perguntas sobre tiers. Aproxima o guardrail do comportamento determinístico.

---

*Documento elaborado com base exclusivamente nos Guardrails Operacionais do Assistente v1.0 e nos arquivos de referência da NovaTech: POL-001 v3.1, PROC-042-v1, PROC-042-v2, SLA-2024 v2024.1, FAQ-Atendimento e SPEC RAG NovaTech v1.3.*

*NovaTech Logística — Confidencial — Versão 1.0*
