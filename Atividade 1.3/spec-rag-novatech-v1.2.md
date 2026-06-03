# Especificação de Requisitos — Pipeline RAG
## Assistente de Atendimento IA | NovaTech Logística

| Campo | Valor |
|-------|-------|
| Versão | 1.2 |
| Data | Junho/2025 |
| Autor | Product Specialist — Projeto Assistente IA |
| Status | Rascunho para revisão |
| Baseado em | Documentação NovaTech (Anexo A) |
| Alterações v1.1 | INC-01: pré-requisito de data obrigatória na indexação; seção 3.3 convertida em requisitos formais R-CONT-006 e R-CONT-007. INC-02: atualização de status atribuída ao agente de ingestão; R-CONT-001 desdobrado em R-CONT-001b. INC-03: R-RASTR-005 reformulado com linguagem acessível e limite de tamanho do trecho. |
| Alterações v1.2 | GAP-01: chunking definido como decisão técnica pendente (DTP-001). GAP-02: threshold de similaridade 0,75 formalizado em R-GAP-002b. GAP-03: autenticação Azure formalizada em R-NF-005. GAP-04: requisito de feedback do atendente incluído em R-GAP-006. GAP-05: comportamento em modo degradado formalizado em R-NF-006. GAP-06: seleção de modelo LLM definida como decisão técnica pendente (DTP-002). GAP-07: responsabilidade dos gaps documentais atribuída às áreas Operações, Compliance e Comercial. GAP-08: acesso a logs restrito à equipe de segurança com conformidade LGPD em R-RASTR-009 e R-RASTR-010. |

---

## 1. Propósito e Escopo

Este documento especifica os requisitos que o pipeline de RAG (Retrieval-Augmented Generation) deve atender para suportar o Assistente de Atendimento IA da NovaTech. Serve como referência para desenvolvimento, testes de aceitação e auditorias futuras.

O assistente tem como objetivo reduzir o número de fontes consultadas por chamado (atualmente 4,1 em média, chegando a 6,2 em frete especial) e diminuir a taxa de reabertura de chamados em 48 horas (atualmente 22%), por meio de respostas consistentes e rastreáveis ancoradas na documentação oficial.

---

## 2. Fontes de Dados a Serem Indexadas

### 2.1 Documentos Oficiais (Fontes Primárias)

Documentos normativos e contratuais com responsável formal declarado. Devem ter peso de recuperação mais alto e receber indicação de "fonte oficial" no resultado apresentado ao atendente.

> **Pré-requisito de indexação:** Todo documento submetido à base de conhecimento deve conter obrigatoriamente ao menos uma data — podendo ser data de emissão, data de vigência ou data da última revisão. Documentos sem nenhuma data declarada serão rejeitados pelo pipeline de ingestão e não poderão ser indexados até que a informação seja incluída pelo responsável pelo documento.

| ID | Requisito | Critério de Aceitação |
|----|-----------|----------------------|
| F-001 | **POL-001 v3.1** — Política de Devolução de Mercadorias (Jan/2024). Cobre prazos, exceções, procedimento, custos e devoluções parciais. | Documento indexado com metadado `tipo=normativo`, `versão=3.1`, `data=2024-01-15`. |
| F-002 | **PROC-042-v2** — Procedimento de Cálculo de Frete Especial revisado (Nov/2023). Versão vigente para chamados a partir de 01/12/2023, conforme seção 5 do próprio documento. | Documento indexado com metadado `tipo=procedimento`, `versão=2.0`, `data=2023-11-10`, `status=vigente`. |
| F-003 | **PROC-042-v1** — Versão anterior do procedimento de frete especial. Indexada apenas para chamados abertos antes de 01/12/2023 ainda em processamento. | Documento indexado com metadado `tipo=procedimento`, `versão=1.0`, `status=transição`, `vigente_até=2023-11-30`. |
| F-004 | **SLA-2024 v2024.1** — Tabela de SLA por tipo de cliente (Jan/2024). Documento contratual que define tiers, métricas, penalidades e incidentes críticos. | Documento indexado com metadado `tipo=contratual`, `versão=2024.1`, `data=2024-01-02`. |

### 2.2 Documentos de Apoio (Fontes Secundárias)

Fontes com informações úteis, mas sem validação formal. Devem ser recuperáveis, porém entregues ao atendente com sinalização explícita de seu caráter informal.

| ID | Requisito | Critério de Aceitação |
|----|-----------|----------------------|
| F-005 | **FAQ-Atendimento** — Documento colaborativo mantido pelo time de atendimento, sem responsável formal e sem validação de Compliance. Contém práticas informais valiosas. | Documento indexado com metadado `tipo=informal`, `status=não_validado`. Toda recuperação deste documento deve acionar exibição do aviso de fonte informal. |

### 2.3 Fontes Não Cobertas (Gaps Documentais)

> ⚠️ **Atenção:** Os tópicos abaixo foram identificados como gaps: carga danificada em trânsito, seguro de carga (percentuais), frete padrão (abaixo de 500 kg) e processo formal da Gestão de Riscos para cargas perigosas devolvidas. Esses temas não possuem documento formal na base e devem ser tratados conforme o requisito R-GAP-001 (Seção 4).

A responsabilidade pela criação dos documentos que enderecem esses gaps é das áreas de **Operações**, **Compliance** e **Comercial** da NovaTech, conforme o escopo de cada tema. Recomenda-se que o time de produto formalize a solicitação de documentação a essas áreas, indicando os temas sem cobertura e o impacto operacional identificado no discovery (22% de reabertura de chamados concentrada justamente nesses tópicos). Enquanto os documentos não forem publicados, o assistente seguirá o comportamento definido em R-GAP-001: informar que não há documentação disponível e orientar o atendente a consultar diretamente a área responsável.

### 2.4 Fontes Futuras (Roadmap de Expansão)

A especificação deve ser suficientemente flexível para incorporar, em versões futuras:

- PROC-043 — Frete de Cargas Perigosas (citado como em revisão pelo Compliance)
- Tabela mensal de fretes-base (arquivo Excel do servidor de rede)
- Procedimento formal de Gestão de Riscos para cargas perigosas
- Política formal de carga danificada e sinistros
- Política formal de seguro de carga

---

## 3. Tratamento de Documentos Contraditórios

A coexistência do PROC-042-v1 e PROC-042-v2 sem hierarquia declarada foi o principal fator que elevou para 6,2 o número de fontes consultadas em chamados de frete especial. A estratégia abaixo resolve esse conflito e define o comportamento para casos futuros.

### 3.1 Estratégia de Prioridade por Metadados

| ID | Requisito | Critério de Aceitação |
|----|-----------|----------------------|
| R-CONT-001 | O pipeline deve atribuir, no momento da indexação, os metadados `versão`, `data de emissão`, `status` (vigente / transição / obsoleto) e `tipo` (normativo / procedimento / contratual / informal) a cada documento. O status inicial de cada documento deve ser definido pelo responsável pela publicação no ato da submissão. | Todos os documentos indexados contêm os quatro metadados obrigatórios. Indexação sem esses campos é rejeitada. |
| R-CONT-001b | A atualização de status dos documentos (por exemplo, de `vigente` para `obsoleto` quando uma nova versão é publicada) será realizada pelo próprio agente de ingestão, sem necessidade de pipeline separado. A transição de status deve ocorrer no mesmo evento de ingestão do documento substituto: ao indexar a nova versão, o agente atualiza automaticamente o status da versão anterior. A mudança de `transição` para `obsoleto` deve ocorrer automaticamente após a data de corte declarada no metadado `vigente_até` do documento. | Ingestão de nova versão de qualquer documento: status da versão anterior atualizado para `obsoleto` (ou `transição`, quando aplicável) na mesma operação, sem intervenção manual. Após a data definida em `vigente_até`, documento com `status=transição` é automaticamente marcado como `obsoleto` pelo agente. |
| R-CONT-002 | Quando dois chunks recuperados tratarem do mesmo tema e originarem de documentos com status diferente, o pipeline deve priorizar o de `status=vigente` na composição da resposta. | Teste com pergunta sobre multiplicadores regionais: a resposta usa os valores da v2 (Sul=1,3; Sudeste=1,1; etc.), não da v1. |
| R-CONT-003 | Quando dois chunks recuperados tratarem do mesmo tema e tiverem o mesmo status, o pipeline deve priorizar o de data de emissão mais recente. | Teste com dois documentos hipotéticos de mesmo status: resposta usa o mais recente. |

### 3.2 Sinalização de Conflito ao Atendente

| ID | Requisito | Critério de Aceitação |
|----|-----------|----------------------|
| R-CONT-004 | Quando o pipeline identificar chunks contraditórios recuperados (mesmo tema, valores divergentes), deve sinalizar o conflito explicitamente na resposta, indicando qual versão está sendo usada e por quê. | Resposta sobre frete especial contém bloco de aviso: *"Existe uma versão anterior (PROC-042 v1) com valores diferentes. Esta resposta usa a v2 (vigente para chamados a partir de 01/12/2023)."* |
| R-CONT-005 | A sinalização de conflito deve incluir o nome dos documentos divergentes, o valor conflitante e a regra de desempate aplicada (data ou status). | Aviso exibe: documento A, documento B, campo conflitante, regra de desempate. |

### 3.3 Caso Especial: PROC-042 v1 vs v2

| ID | Requisito | Critério de Aceitação |
|----|-----------|----------------------|
| R-CONT-006 | O pipeline deve identificar a data de abertura do chamado antes de selecionar qual versão do PROC-042 utilizar: chamados abertos até 30/11/2023 devem usar a v1; chamados abertos a partir de 01/12/2023 devem usar a v2. | Teste com chamado datado de 20/11/2023: resposta usa PROC-042-v1. Teste com chamado datado de 05/12/2023: resposta usa PROC-042-v2. |
| R-CONT-007 | Quando a data de abertura do chamado não estiver disponível no contexto da conversa, o assistente deve solicitá-la ao atendente antes de responder sobre cálculo de frete especial. | Pergunta sobre frete especial sem data de chamado informada: assistente solicita "Por favor, informe a data de abertura do chamado para que eu possa aplicar a versão correta do procedimento." |

---

## 4. Comportamento para Perguntas sem Resposta na Base

### 4.1 Taxonomia de Gaps

| Tipo de Gap | Definição | Exemplos na NovaTech |
|-------------|-----------|---------------------|
| **Gap total** | Tópico completamente ausente da base. | Política de carga danificada; Seguro de carga; Processo da Gestão de Riscos. |
| **Gap parcial** | Tópico coberto na base, mas com informações incompletas ou apenas no FAQ informal. | Carga perigosa com frete expresso (só FAQ); Tier Platinum (só FAQ — e a resposta está correta: não existe). |
| **Gap contextual** | A base tem a informação, mas a pergunta exige dados que o atendente não forneceu. | Frete especial sem informar região ou peso; SLA sem informar tier do cliente. |

### 4.2 Requisitos de Comportamento

| ID | Requisito | Critério de Aceitação |
|----|-----------|----------------------|
| R-GAP-001 | Quando nenhum chunk relevante for recuperado (gap total), o assistente deve responder que não encontrou informação na base de conhecimento e indicar o canal de escalação adequado. | Pergunta sobre seguro de carga: *"Não encontrei uma política formal de seguro de carga na documentação disponível. Para essa informação, recomendo contatar o Comercial ou verificar o contrato do cliente."* |
| R-GAP-002 | O assistente não deve fabricar respostas. É proibido inferir, extrapolar ou gerar informações não presentes nos documentos recuperados. | Respostas com informações não contidas em nenhum documento indexado devem ser consideradas falha grave em testes de aceitação. |
| R-GAP-002b | O pipeline deve adotar um score mínimo de similaridade de **0,75** para que um trecho recuperado seja considerado relevante e utilizado na resposta. Trechos com score abaixo desse valor devem ser descartados; se nenhum trecho atingir o mínimo, o assistente deve adotar o comportamento de gap total (R-GAP-001). O valor de 0,75 deve ser validado e, se necessário, ajustado durante a fase de homologação com base nos primeiros resultados reais de consulta. | Teste com pergunta sem correspondência na base: nenhum trecho com score < 0,75 é usado na resposta. Teste com pergunta diretamente coberta pela documentação: trecho correto retorna com score ≥ 0,75. Relatório de homologação documenta o score médio dos primeiros 100 chamados reais e registra decisão de manter ou ajustar o threshold. |
| R-GAP-003 | Quando a resposta existir apenas no FAQ informal, o assistente deve fornecê-la acompanhada de aviso explícito de que a fonte não é validada oficialmente. | Pergunta sobre carga perigosa com frete expresso: resposta inclui informação do FAQ-Item 32 com rótulo *"Fonte: FAQ informal — não validado por Compliance."* |
| R-GAP-004 | Quando o gap for contextual (falta de dado do atendente), o assistente deve solicitar a informação faltante de forma objetiva antes de responder. | Pergunta *"qual o frete especial?"* sem peso ou região: assistente solicita *"Por favor, informe o peso da carga e a região de destino para calcular o multiplicador correto."* |
| R-GAP-005 | Toda resposta de gap deve registrar o tipo de gap no log interno para alimentar o processo de melhoria contínua da base de conhecimento. | Logs de gap total disponíveis no painel de administração com data, pergunta e tipo de gap. |
| R-GAP-006 | O assistente deve coletar feedback do atendente ao final de cada resposta, permitindo que ele informe se a informação foi **útil**, **incorreta** ou **incompleta**. O feedback deve ser registrado vinculado à interação correspondente e disponibilizado no painel de administração para análise periódica pela equipe responsável pela base de conhecimento. | Interface exibe opção de feedback após cada resposta com as três opções. Seleção de "incorreta" ou "incompleta" abre campo opcional para o atendente descrever o problema. Painel de administração exibe histórico de feedbacks com filtro por tipo, data e documento citado na resposta avaliada. |

---

## 5. Requisitos de Atualização da Base

### 5.1 SLAs de Ingestão

| ID | Requisito | Critério de Aceitação |
|----|-----------|----------------------|
| R-ATU-001 | Documentos normativos (POL) e contratuais (SLA): disponíveis no assistente em até **4 horas úteis** após publicação oficial. | Publicação simulada de nova versão da POL-001 às 09h → chunks disponíveis e retornados em consultas até 13h. |
| R-ATU-002 | Procedimentos (PROC): disponíveis em até **4 horas úteis** após publicação. | Publicação simulada de PROC-043 às 14h → disponível até 18h do mesmo dia. |
| R-ATU-003 | FAQ e documentos informais: disponíveis em até **24 horas úteis** após atualização. | Atualização do FAQ às 09h de segunda → disponível até 09h de terça (em dias úteis). |
| R-ATU-004 | Quando um documento substituir outro (nova versão), o documento anterior deve ter seu status atualizado para `obsoleto` ou `transição` simultaneamente à ingestão do novo, no mesmo pipeline. | Ingestão da v2 de qualquer PROC: consultas simultâneas testadas — nenhuma retorna chunks da v1 com `status=vigente` após a ingestão. |

### 5.2 Processo de Ingestão

| ID | Requisito | Critério de Aceitação |
|----|-----------|----------------------|
| R-ATU-005 | O pipeline deve suportar ingestão incremental (apenas o documento novo ou alterado), sem necessidade de reindexação completa da base. | Ingestão de documento único em menos de 15 minutos sem impacto perceptível no tempo de resposta de outros documentos. |
| R-ATU-006 | Cada ingestão deve gerar um log com: nome do arquivo, versão, data/hora de início e conclusão, número de chunks gerados e status (sucesso / erro). | Logs de ingestão acessíveis no painel de administração com os campos listados. |
| R-ATU-007 | Em caso de falha na ingestão, o sistema deve alertar o administrador responsável em até 30 minutos, sem interromper o funcionamento do assistente com a base anterior. | Simulação de falha de ingestão: alerta enviado e assistente continua operando com versão anterior dos chunks. |

---

## 6. Requisitos de Rastreabilidade

### 6.1 Citação de Fonte

| ID | Requisito | Critério de Aceitação |
|----|-----------|----------------------|
| R-RASTR-001 | Toda resposta gerada pelo assistente deve citar a(s) fonte(s) utilizada(s), indicando: nome do documento, versão, seção/item específico e data da última atualização. | Resposta sobre prazo de devolução cita: *"POL-001, v3.1, seção 3.1, atualizada em 15/01/2024."* |
| R-RASTR-002 | Quando a resposta for baseada em múltiplos documentos, todas as fontes devem ser citadas, indicando qual parte da resposta deriva de qual fonte. | Resposta sobre frete especial de carga perigosa cita PROC-042-v2 para o multiplicador e PROC-043 para a regra de carga perigosa (ou indica que PROC-043 está em revisão). |
| R-RASTR-003 | Fontes informais (FAQ) devem ser citadas com indicação explícita do seu status. | Resposta baseada no FAQ exibe o rótulo: *"FAQ-Atendimento (documento informal, não validado por Compliance)."* |

### 6.2 Exibição do Trecho Relevante

| ID | Requisito | Critério de Aceitação |
|----|-----------|----------------------|
| R-RASTR-004 | O assistente deve exibir, junto à resposta, o trecho do documento que embasou cada afirmação — formatado de forma destacada e com indicação da localização no documento. | Interface exibe o trecho em bloco recuado ou destacado, com rótulo *"Trecho de [Documento], [Seção]."* |
| R-RASTR-005 | O trecho exibido deve reproduzir exatamente o texto do documento original, sem alterações de conteúdo, para que o atendente possa conferir se a resposta está alinhada com o que a documentação diz. Quando o trecho original for muito extenso ou contiver linguagem técnica complexa, o assistente deve exibir um resumo em linguagem simples seguido do trecho original completo disponível para consulta. | Atendente consegue localizar no documento original o trecho exibido sem dificuldade. Em trechos com mais de 150 palavras ou linguagem muito técnica, a interface exibe primeiro um resumo em linguagem acessível e depois o texto original. |
| R-RASTR-006 | O atendente deve ter acesso a um link ou referência para o documento completo, sempre que disponível no sistema de gestão documental. | Resposta inclui referência ao caminho do arquivo ou URL do SharePoint, quando disponível. |

### 6.3 Rastreabilidade de Log

| ID | Requisito | Critério de Aceitação |
|----|-----------|----------------------|
| R-RASTR-007 | Cada interação (pergunta + resposta) deve ser registrada em log com: timestamp, ID do atendente, pergunta, trechos recuperados (com score de similaridade), resposta gerada e fontes citadas. | Log auditável disponível por no mínimo 90 dias. Campos de recuperação incluem identificador do documento, identificador do trecho e score. |
| R-RASTR-008 | O log deve ser utilizável para auditoria retroativa em caso de reabertura de chamado, permitindo reconstruir qual resposta foi fornecida e com base em qual versão dos documentos. | Dado o ID de um chamado reaberto, é possível consultar o log e identificar exatamente qual trecho e versão de documento embasou a resposta original. |
| R-RASTR-009 | O acesso aos logs é restrito exclusivamente à **equipe de segurança da NovaTech**. Nenhum outro perfil de usuário — incluindo administradores do assistente e atendentes — deve ter acesso direto aos registros de log. | Tentativa de acesso aos logs por perfil diferente do de segurança resulta em negação de acesso. Controle de acesso validado em teste de permissões. |
| R-RASTR-010 | Dados pessoais eventualmente presentes nos logs (como nome ou identificador do atendente) devem ser tratados em conformidade com a **Lei Geral de Proteção de Dados (LGPD — Lei nº 13.709/2018)**. Isso inclui: finalidade declarada para o tratamento, prazo de retenção definido (mínimo 90 dias, máximo a ser definido pelo DPO da NovaTech), e procedimento de descarte seguro ao final do prazo. | Documentação de tratamento de dados disponível antes do go-live. Prazo de retenção homologado pelo DPO. Processo de descarte documentado e testado. |

---

## 7. Requisitos Não Funcionais

| ID | Requisito | Critério de Aceitação |
|----|-----------|----------------------|
| R-NF-001 | **Tempo de resposta:** o assistente deve retornar uma resposta completa em até 5 segundos para 95% das consultas, em condições normais de operação. | Teste de carga com 50 consultas simultâneas: percentil 95 ≤ 5s. |
| R-NF-002 | **Disponibilidade:** o assistente deve estar disponível durante todo o horário comercial (08h–18h, dias úteis) com SLA de 99% de uptime mensal. | Monitoramento mensal: downtime ≤ 18 minutos no horário comercial. |
| R-NF-003 | **Idioma:** todas as respostas devem ser em português brasileiro, independentemente do idioma do documento-fonte. | Documentos em outros idiomas (caso futuros) são traduzidos na ingestão ou na geração da resposta. |
| R-NF-004 | **Escopo restrito:** o assistente deve responder apenas sobre temas cobertos pela documentação NovaTech indexada. Perguntas fora do escopo devem ser sinalizadas como tal. | Pergunta sobre clima ou futebol: assistente informa que o tema está fora do seu escopo operacional. |
| R-NF-005 | **Autenticação via Azure AD:** o acesso ao assistente é controlado pelo Azure Active Directory da NovaTech. Somente usuários com conta ativa e perfil autorizado no Azure poderão realizar consultas. Tentativas de acesso sem autenticação válida devem ser bloqueadas. | Acesso sem credencial Azure resulta em bloqueio. Acesso com credencial válida e perfil autorizado concede uso normalmente. Revogação de acesso no Azure reflete no assistente sem necessidade de ação adicional. |
| R-NF-006 | **Comportamento em modo degradado:** quando o assistente estiver indisponível (manutenção, falha ou janela de atualização da base), o sistema deve exibir uma mensagem clara informando a situação e indicando onde o atendente pode consultar a documentação enquanto o serviço não é restabelecido. A mensagem deve diferenciar os dois estados: indisponibilidade por falha e indisponibilidade por atualização planejada. | Estado de falha exibe: *"O assistente está temporariamente indisponível. Consulte a documentação oficial em [caminho/URL definido pela NovaTech] ou aguarde o restabelecimento do serviço."* Estado de atualização planejada exibe: *"O assistente está em atualização e voltará em breve. Consulte a documentação oficial em [caminho/URL] durante esse período."* |

---

## 8. Decisões Técnicas Pendentes

As definições abaixo têm impacto direto na qualidade do pipeline, mas dependem de análise técnica e validação em ambiente de homologação. Devem ser documentadas e aprovadas antes do início do desenvolvimento.

| ID | Decisão Pendente | Impacto | Responsável |
|----|-----------------|---------|-------------|
| DTP-001 | **Estratégia de chunking:** definir tamanho dos trechos (em tokens ou palavras), percentual de sobreposição entre trechos adjacentes e tratamento especial para tabelas numéricas (como multiplicadores regionais e tabelas de SLA, que são mal recuperadas com fragmentação ingênua). | Impacta diretamente a precisão das respostas sobre frete e SLA — tópicos que concentram 35% das dúvidas. | Time de engenharia / arquitetura de IA |
| DTP-002 | **Seleção e versionamento do modelo LLM:** definir qual modelo será utilizado, política de atualização de versão e processo de aprovação antes de qualquer troca de modelo em produção. Trocas de modelo sem validação podem alterar o comportamento das respostas mesmo sem mudança na base de conhecimento. | Afeta consistência e previsibilidade de todas as respostas geradas. | Time de engenharia / Product Owner |

---

## 9. Glossário

| Termo | Definição |
|-------|-----------|
| **RAG** | Retrieval-Augmented Generation. Arquitetura que combina recuperação de documentos com geração de texto por LLM. |
| **Chunk** | Fragmento de texto extraído de um documento para indexação vetorial. Unidade básica de recuperação. |
| **Embedding** | Representação vetorial de um chunk, usada para calcular similaridade semântica com a pergunta do atendente. |
| **Gap documental** | Tópico relevante para o atendimento que não possui documento oficial na base de conhecimento. |
| **Fonte primária** | Documento normativo, contratual ou de procedimento com responsável formal declarado. |
| **Fonte secundária** | Documento informal (como o FAQ) sem validação oficial, usado como apoio quando não há fonte primária. |
| **Status vigente** | Documento ativo e aplicável a chamados a partir da data de emissão da versão atual. |
| **Status transição** | Documento ainda aplicável a chamados abertos antes de determinada data de corte. |
| **Status obsoleto** | Documento substituído por versão mais recente e não aplicável a chamados novos. |

---

*Versão 1.2 — Confidencial — NovaTech Logística*
