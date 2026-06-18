# Guardrails Operacionais do Assistente de Atendimento IA
## NovaTech Logística

| Campo | Valor |
|-------|-------|
| Versão | 3.0 |
| Data de elaboração | Junho/2025 |
| Baseado em | POL-001 v3.1; PROC-042-v2 v2.0; PROC-042-v1 v1.0; SLA-2024 v2024.1; FAQ-Atendimento (informal); Especificação SPEC-RAG-NovaTech v1.3 |
| Alterações v2.0 | Inclusão do campo "Enforcement" em cada guardrail, com classificação entre Código (Determinístico) e Prompt (Probabilístico), acompanhada de justificativa técnica. |
| Alterações v3.0 | Inclusão do campo "Incidente Relacionado" em cada guardrail, conectando cada regra a ao menos um dos três incidentes registrados durante testes internos. |
| Classificação | Documento operacional — uso obrigatório no desenvolvimento, homologação e auditoria do assistente |

---

## Registro de Incidentes de Referência

Os três incidentes abaixo foram registrados durante testes internos e servem como âncora para a rastreabilidade dos guardrails. Cada guardrail indica qual incidente ele previne — diretamente (causa raiz) ou indiretamente (fator contribuinte ou de detecção).

| ID | Descrição do Incidente |
|----|------------------------|
| **INC-01** | O assistente respondeu que o prazo de devolução para carga perigosa é 7 dias, quando na verdade cargas perigosas NÃO podem ser devolvidas pelo processo padrão. |
| **INC-02** | O assistente citou "PROC-042, seção 2" mas os multiplicadores informados eram da versão 1 (desatualizada), não da v2 (vigente). |
| **INC-03** | O assistente disse "Não encontrei informação sobre isso" para uma pergunta sobre SLA Gold, quando o documento SLA-2024 estava indexado e continha a resposta. |

> **Legenda de vínculo:**
> - **Causa raiz:** o guardrail aborda diretamente o comportamento que originou o incidente.
> - **Fator contribuinte:** o guardrail, se violado, cria a condição que permite o incidente ocorrer.
> - **Detecção/mitigação:** o guardrail não previne o incidente, mas permite identificá-lo ou limitar seu impacto após o fato.

---

## Critério de Classificação de Enforcement

| Classificação | Significado |
|---------------|-------------|
| **Código — Determinístico** | Implementado fora do LLM (pipeline, banco vetorial, sistema de autenticação, interface, logger). O comportamento ocorre independentemente do que o modelo gerar. |
| **Prompt — Probabilístico** | Instrução no system prompt ou no contexto enviado ao LLM. O comportamento ocorre na grande maioria dos casos, mas depende da geração do modelo e pode desviar sob entradas adversariais ou incomuns. |
| **Misto — Código primário** | O pipeline garante a pré-condição (detecta, filtra, bloqueia) de forma determinística; o LLM formata e comunica o resultado. A correção do comportamento é garantida pelo código. |
| **Misto — Prompt primário** | A classificação ou interpretação semântica é feita pelo LLM; o código fornece os critérios fixos no contexto. A qualidade do comportamento depende principalmente do prompt. |

---

## Resumo Executivo

| Categoria | ID | Nome Curto | Enforcement | Incidente(s) |
|-----------|----|------------|-------------|--------------|
| **DEVE** | GR-D-001 | Citar fonte em toda resposta | Prompt — Probabilístico | INC-01, INC-02 (detecção) |
| **DEVE** | GR-D-002 | Priorizar documentos com status vigente | Código — Determinístico | INC-02 (causa raiz) |
| **DEVE** | GR-D-003 | Sinalizar conflito entre versões | Misto — Código primário | INC-02 (fator contribuinte) |
| **DEVE** | GR-D-004 | Sinalizar fontes informais | Misto — Código primário | INC-01 (fator contribuinte) |
| **DEVE** | GR-D-005 | Aplicar versão do PROC-042 conforme data do chamado | Código — Determinístico | INC-02 (causa raiz) |
| **DEVE** | GR-D-006 | Identificar incidentes críticos conforme SLA-2024 | Misto — Prompt primário | INC-03 (fator contribuinte) |
| **DEVE** | GR-D-007 | Respeitar SLAs por tier de cliente | Misto — Código primário | INC-03 (causa raiz) |
| **DEVE** | GR-D-008 | Exibir trecho original que embasou a resposta | Misto — Código primário | INC-01, INC-02 (detecção) |
| **DEVE** | GR-D-009 | Registrar log de cada interação | Código — Determinístico | INC-01, INC-02, INC-03 (detecção) |
| **DEVE** | GR-D-010 | Coletar feedback do atendente | Código — Determinístico | INC-01, INC-02, INC-03 (detecção) |
| **DEVE** | GR-D-011 | Distinguir gap documental de fora do escopo | Misto — Prompt primário | INC-03 (causa raiz) |
| **NÃO DEVE** | GR-N-001 | Fabricar ou extrapolar informações | Prompt — Probabilístico | INC-01 (causa raiz) |
| **NÃO DEVE** | GR-N-002 | Usar documento com status obsoleto para chamados novos | Código — Determinístico | INC-02 (causa raiz) |
| **NÃO DEVE** | GR-N-003 | Confirmar tiers inexistentes | Prompt — Probabilístico | INC-03 (fator contribuinte) |
| **NÃO DEVE** | GR-N-004 | Apresentar fonte informal como oficial | Misto — Código primário | INC-01 (fator contribuinte) |
| **NÃO DEVE** | GR-N-005 | Autorizar descontos sem respaldo documental | Prompt — Probabilístico | INC-01 (fator contribuinte) |
| **NÃO DEVE** | GR-N-006 | Dar orientação definitiva sobre temas com gaps documentais | Misto — Código primário | INC-01 (fator contribuinte) |
| **NÃO DEVE** | GR-N-007 | Usar trecho abaixo do limiar de similaridade | Código — Determinístico | INC-03 (causa raiz) |
| **NÃO DEVE** | GR-N-008 | Expor dados de log a perfis não autorizados | Código — Determinístico | INC-01, INC-02, INC-03 (não aplicável — segurança) |
| **QUANDO EM DÚVIDA** | GR-W-001 | Versão do PROC-042 sem data do chamado | Misto — Código primário | INC-02 (fator contribuinte) |
| **QUANDO EM DÚVIDA** | GR-W-002 | Dados insuficientes para cálculo de frete | Misto — Código primário | INC-02 (fator contribuinte) |
| **QUANDO EM DÚVIDA** | GR-W-003 | Tema presente apenas no FAQ informal | Misto — Código primário | INC-01 (fator contribuinte) |
| **QUANDO EM DÚVIDA** | GR-W-004 | Tema com gap total na base | Código — Determinístico | INC-03 (causa raiz) |
| **QUANDO EM DÚVIDA** | GR-W-005 | Assistente indisponível | Código — Determinístico | INC-03 (fator contribuinte) |
| **QUANDO EM DÚVIDA** | GR-W-006 | FAQ conflita com documento primário recém-atualizado | Código — Determinístico | INC-01, INC-02 (fator contribuinte) |

---

## DEVE — Comportamentos Obrigatórios

---

### GR-D-001 — Citar fonte em toda resposta

**Descrição da regra:**
Toda resposta gerada pelo assistente deve citar a fonte utilizada, indicando: nome do documento, versão, seção ou item específico e data da última atualização do documento. Quando a resposta for fundamentada em múltiplos documentos, todas as fontes devem ser listadas, com indicação de qual trecho da resposta deriva de qual documento.

**Objetivo da regra:**
Garantir rastreabilidade e auditabilidade das respostas, permitindo que o atendente verifique a informação na documentação original e que auditorias retroativas reconstruam qual versão dos documentos embasou cada orientação fornecida.

**Exemplo de comportamento esperado:**
> Atendente pergunta: "Qual é o prazo para devolução?"
> Assistente responde: "O cliente pode solicitar devolução em até 7 dias úteis após a data de recebimento confirmada no tracking. *(Fonte: POL-001, v3.1, seção 3.1, atualizada em 15/01/2024.)*"

**Enforcement: Prompt — Probabilístico**
A obrigação de incluir nome do documento, versão, seção e data na resposta é uma instrução de formatação de saída dirigida ao LLM. O pipeline pode injetar os metadados dos trechos recuperados no contexto, mas a decisão de incluí-los na resposta gerada — de forma completa e corretamente atribuída — é do modelo. Não existe mecanismo de código que force o modelo a incluir texto específico na saída sem reprocessamento pós-geração. O risco residual é mitigado pelo GR-D-009 (log auditável) e pelo GR-D-010 (feedback do atendente).

**Incidente relacionado: INC-01 e INC-02 — Detecção**
Em INC-01, se a fonte tivesse sido citada como "POL-001, seção 3.2", o atendente poderia ter consultado o documento e verificado imediatamente que cargas perigosas são uma exceção ao prazo geral de 7 dias. Em INC-02, a citação explícita de versão ("PROC-042, seção 2") revelou a falha — mas não foi suficiente para evitá-la, pois a versão citada era incorreta. Este guardrail não previne os incidentes diretamente, mas é o mecanismo que os torna detectáveis: sem citação de fonte rastreável, ambos os erros poderiam passar despercebidos pelo atendente.

**Referências:** R-RASTR-001, R-RASTR-002 (SPEC v1.3)

---

### GR-D-002 — Priorizar documentos com status vigente

**Descrição da regra:**
Quando dois ou mais trechos recuperados tratarem do mesmo tema, o assistente deve priorizar o trecho proveniente do documento com `status=vigente`. Em caso de dois documentos com mesmo status, priorizar o de data de emissão mais recente. O mecanismo de re-ranking deve aplicar essas regras automaticamente na composição de cada resposta.

**Objetivo da regra:**
Evitar que informações de versões antigas ou em transição sejam apresentadas como orientação atual, reduzindo erros operacionais e reaberturas de chamados.

**Exemplo de comportamento esperado:**
> Pergunta sobre multiplicadores regionais de frete especial: o assistente utiliza os valores do PROC-042-v2 (Sul=1,3; Sudeste=1,1; Centro-Oeste=1,4; Nordeste=1,5; Norte=1,8), não os valores da v1, por ser a versão com `status=vigente` para chamados a partir de 01/12/2023.

**Enforcement: Código — Determinístico**
A priorização de trechos com `status=vigente` é implementada como regra de re-ranking no pipeline de recuperação, antes de qualquer token ser enviado ao LLM. O pipeline compara os metadados dos chunks e seleciona os trechos conforme a hierarquia de status. O modelo nunca recebe simultaneamente um trecho vigente e um obsoleto sobre o mesmo tema. O comportamento é verificável em teste de integração (R-CONT-002 da SPEC v1.3).

**Incidente relacionado: INC-02 — Causa raiz**
INC-02 é a materialização exata da falha que este guardrail previne: o pipeline recuperou e utilizou multiplicadores do PROC-042-v1 (desatualizado) em vez do PROC-042-v2 (vigente). Se o re-ranking por `status=vigente` estivesse implementado, o chunk da v1 seria preterido automaticamente e o modelo nunca receberia os valores incorretos — independentemente de como a pergunta foi formulada.

**Referências:** R-CONT-002, R-CONT-003, F-006 (SPEC v1.3)

---

### GR-D-003 — Sinalizar conflito entre versões

**Descrição da regra:**
Sempre que o pipeline identificar trechos recuperados com conteúdo divergente sobre o mesmo tema — mesmo tendo selecionado a versão correta para a resposta —, o assistente deve incluir na resposta um aviso explícito informando: (a) qual versão está sendo utilizada e por quê; (b) o nome dos documentos divergentes; (c) o valor conflitante; e (d) a regra de desempate aplicada (data ou status).

**Objetivo da regra:**
Manter o atendente informado sobre a existência de versões conflitantes na base, evitando que ele utilize a resposta sem compreender o contexto de ambiguidade documental.

**Exemplo de comportamento esperado:**
> Resposta sobre frete especial inclui bloco de aviso: *"Atenção: existe uma versão anterior deste procedimento (PROC-042 v1, emitido em 03/03/2023) com multiplicadores regionais diferentes. Esta resposta utiliza a v2 (emitida em 10/11/2023, status=vigente para chamados a partir de 01/12/2023)."*

**Enforcement: Misto — Código primário**
A **detecção** do conflito é determinística: o pipeline identifica, durante o re-ranking, que dois chunks com scores relevantes tratam do mesmo tema e provêm de documentos com versões ou datas diferentes — verificação de metadados executada em código. A **existência** do aviso é, portanto, garantida pelo pipeline. A **redação** do aviso (nome dos documentos, valor divergente, regra de desempate) é gerada pelo LLM com base nas instruções do prompt e nos metadados injetados no contexto. O risco residual é que o modelo omita algum dos campos exigidos, mitigável por template estruturado no prompt.

**Incidente relacionado: INC-02 — Fator contribuinte**
Em INC-02, o assistente utilizou os multiplicadores da v1 sem sinalizar ao atendente que havia duas versões do PROC-042 com valores diferentes. Mesmo que o re-ranking correto (GR-D-002) tivesse selecionado a v2, este guardrail acrescenta uma camada de transparência: o atendente é alertado sobre a existência do conflito e sabe qual versão está sendo usada e por quê. Isso permite que ele identifique casos de borda — como chamados em período de transição — que exijam atenção especial.

**Referências:** R-CONT-004, R-CONT-005 (SPEC v1.3)

---

### GR-D-004 — Sinalizar fontes informais

**Descrição da regra:**
Toda resposta fundamentada total ou parcialmente no FAQ-Atendimento deve exibir, de forma visualmente destacada, o aviso de que a fonte não é validada oficialmente. A sinalização deve ser exibida independentemente de a informação do FAQ estar ou não corroborada por documento primário.

**Objetivo da regra:**
Impedir que o atendente tome decisões operacionais com base em informações não homologadas sem ter ciência do status informal da fonte consultada.

**Exemplo de comportamento esperado:**
> Resposta sobre carga perigosa com frete expresso inclui: *"Fonte: FAQ-Atendimento (documento informal, não validado por Compliance ou Operações). Confirme com a área responsável antes de orientar o cliente."*

**Enforcement: Misto — Código primário**
O gatilho para exibir o aviso é determinístico: quando o pipeline recupera um chunk com metadado `tipo=informal`, injeta automaticamente no contexto uma flag que instrui o LLM a incluir o aviso. A decisão baseia-se exclusivamente no metadado — sem necessidade de interpretar o conteúdo. O **posicionamento e a redação** do aviso na resposta dependem do LLM seguir a instrução recebida. Como mitigação adicional, o aviso pode ser injetado diretamente no template de UI, desacoplando-o da geração do LLM para esse elemento específico.

**Incidente relacionado: INC-01 — Fator contribuinte**
Em INC-01, é provável que o assistente tenha recuperado o FAQ-Item 3, que orienta a encaminhar cargas perigosas ao ramal 4500 mas menciona que "já tiveram casos em que o pessoal de Riscos autorizou exceção" — uma informação informal que pode ter contribuído para que o modelo generalizasse incorretamente o prazo de 7 dias. Se o aviso de fonte informal tivesse sido exibido, o atendente saberia que a orientação sobre devoluções de carga perigosa não provinha da POL-001 e deveria ser confirmada.

**Referências:** R-RASTR-003, R-GAP-003, F-005 (SPEC v1.3)

---

### GR-D-005 — Aplicar versão do PROC-042 conforme data do chamado

**Descrição da regra:**
Antes de fornecer qualquer informação sobre cálculo de frete especial, o assistente deve verificar a data de abertura do chamado em questão. Chamados abertos até 30/11/2023 devem ser respondidos com base no PROC-042-v1; chamados abertos a partir de 01/12/2023 devem ser respondidos com base no PROC-042-v2. Essa verificação é obrigatória e não pode ser ignorada.

**Objetivo da regra:**
Garantir que os multiplicadores regionais, fatores de peso e prazos informados correspondam exatamente ao procedimento contratualmente aplicável ao chamado específico, evitando contestações e inconsistências.

**Exemplo de comportamento esperado:**
> Chamado datado de 20/11/2023: assistente aplica PROC-042-v1 (multiplicador Norte=1,6; fator de peso acima de 3.000kg=1,5; prazo adicional=+2 dias úteis).
> Chamado datado de 05/12/2023: assistente aplica PROC-042-v2 (multiplicador Norte=1,8; fator de peso acima de 3.000kg=1,4; prazo adicional=+3 dias úteis).

**Enforcement: Código — Determinístico**
Este guardrail possui o maior impacto financeiro e contratual do conjunto. A seleção da versão correta é implementada como lógica de código: ao receber a data de abertura do chamado, o pipeline executa a verificação `data < 01/12/2023 → injetar apenas chunks da v1; data ≥ 01/12/2023 → injetar apenas chunks da v2`. O modelo nunca recebe os chunks da versão errada. A data de corte (01/12/2023) é um valor fixo e não interpretável — deixá-lo a cargo do LLM introduziria risco desnecessário onde o código oferece garantia total.

**Incidente relacionado: INC-02 — Causa raiz**
Complementar ao GR-D-002, este guardrail aborda a dimensão temporal do mesmo incidente: em INC-02, não apenas o documento com status incorreto foi selecionado, mas também não houve verificação da data do chamado antes de determinar qual versão do PROC-042 aplicar. Este guardrail garante que, mesmo que o metadado de status esteja incorreto, a lógica de data de abertura do chamado funcione como segunda linha de defesa para selecionar a versão correta.

**Referências:** R-CONT-006, R-CONT-007, F-002, F-003 (SPEC v1.3); Seção 5 do PROC-042-v2

---

### GR-D-006 — Identificar incidentes críticos conforme SLA-2024

**Descrição da regra:**
O assistente deve reconhecer e sinalizar quando uma situação descrita pelo atendente atender a pelo menos um dos critérios de incidente crítico definidos na seção 3 do SLA-2024: (a) carga com valor declarado acima de R$ 100.000 com status desconhecido há mais de 6 horas; (b) carga perigosa com qualquer irregularidade de documentação ou rastreamento; (c) mais de 5 chamados do mesmo cliente nas últimas 24 horas sobre o mesmo problema; (d) qualquer situação com risco à segurança de pessoas. Ao identificar, o assistente deve informar o SLA de atendimento correspondente ao tier do cliente.

**Objetivo da regra:**
Assegurar que situações críticas sejam tratadas com a urgência contratual correta, evitando violações de SLA com impacto contratual e reputacional.

**Exemplo de comportamento esperado:**
> Atendente relata: "Carga com valor de R$ 150.000 está sem atualização de tracking há 7 horas para um cliente Gold."
> Assistente sinaliza: *"Situação classificada como incidente crítico (SLA-2024, seção 3). Para clientes Gold, o SLA de primeira resposta é de até 30 minutos e o de resolução é de até 4 horas, sem pausa no relógio de SLA."*

**Enforcement: Misto — Prompt primário**
Os critérios de incidente crítico precisam ser **reconhecidos a partir de linguagem natural** — o atendente descreve a situação em texto livre, e o modelo mapeia essa descrição aos critérios formais. Esse mapeamento semântico é uma capacidade do LLM, não do código. A camada de código contribui ao injetar os critérios formais do SLA-2024 no contexto (determinístico) e pode verificar automaticamente critérios estruturados quando disponíveis como campos do sistema (ex.: contagem de chamados nas últimas 24h via integração com sistema de chamados). A classificação de situações descritas em linguagem livre permanece probabilística.

**Incidente relacionado: INC-03 — Fator contribuinte**
Em INC-03, o assistente falhou ao responder sobre SLA Gold — um tema com cobertura formal na base. Se o assistente tivesse classificado corretamente o contexto da pergunta como relacionado a SLA de cliente Gold (inclusive potencialmente como incidente crítico), teria localizado e apresentado os valores corretos do SLA-2024. A falha de INC-03 não foi de classificação de incidente crítico especificamente, mas este guardrail fortalece a capacidade geral de o assistente interpretar corretamente consultas sobre SLA contextualizado, reduzindo a probabilidade de respostas do tipo "não encontrei".

**Referências:** SLA-2024, seções 2 e 3 (Anexo A); F-004 (SPEC v1.3)

---

### GR-D-007 — Respeitar SLAs por tier de cliente

**Descrição da regra:**
Ao informar prazos de atendimento, o assistente deve considerar o tier do cliente (Gold, Silver ou Standard) e fornecer os valores corretos conforme a tabela SLA-2024. Nunca deve fornecer um SLA genérico sem especificar o tier ao qual se aplica. Se o tier não for informado pelo atendente, o assistente deve solicitá-lo antes de indicar prazos.

**Objetivo da regra:**
Garantir que as expectativas de prazo comunicadas ao atendente — e por ele ao cliente — correspondam aos compromissos contratuais efetivos da NovaTech para aquele cliente específico.

**Exemplo de comportamento esperado:**
> Atendente pergunta: "Qual é o SLA para chamado de cliente Silver?"
> Assistente responde: "Para clientes Silver: primeira resposta em até 4 horas úteis; resolução em até 48 horas úteis. Para incidentes críticos: primeira resposta em até 1 hora; resolução em até 8 horas. *(Fonte: SLA-2024, v2024.1, seção 2, atualizada em 02/01/2024.)*"

**Enforcement: Misto — Código primário**
O lookup do tier do cliente a partir do identificador de contrato é uma operação de banco de dados ou integração com CRM — determinística. Uma vez identificado o tier, o pipeline injeta no contexto apenas os valores de SLA correspondentes, eliminando ambiguidade. O LLM formata e comunica esses valores na resposta. A camada probabilística reside na formatação: o modelo precisa apresentar os valores sem introduzir confusão entre métricas (ex.: não trocar SLA de resposta com SLA de resolução), controlável por prompt estruturado.

**Incidente relacionado: INC-03 — Causa raiz**
INC-03 descreve exatamente a falha que este guardrail previne: o assistente não recuperou os valores de SLA do cliente Gold quando deveria. Este guardrail exige que o pipeline localize e injete os valores corretos de SLA para o tier informado, tornando a resposta determinística para essa classe de pergunta. Se o tier estiver disponível no contexto e o documento SLA-2024 estiver indexado — condição confirmada no incidente —, não há razão para o assistente responder "não encontrei informação".

**Referências:** SLA-2024, seções 1, 2 e 5 (Anexo A); F-004 (SPEC v1.3)

---

### GR-D-008 — Exibir trecho original que embasou a resposta

**Descrição da regra:**
Junto a cada resposta, o assistente deve exibir o trecho do documento original que fundamentou a afirmação, em bloco visualmente destacado, com rótulo indicando o documento e a seção de origem. Quando o trecho original contiver mais de 150 palavras ou linguagem técnica complexa, o assistente deve exibir primeiro um resumo em linguagem acessível, seguido do texto original completo disponível para consulta.

**Objetivo da regra:**
Permitir que o atendente confira rapidamente se a resposta está alinhada com o documento oficial, reduzindo retrabalho e erros de interpretação.

**Exemplo de comportamento esperado:**
> Resposta sobre exceções à devolução exibe bloco recuado:
> *Trecho de POL-001, seção 3.2:*
> *"As seguintes categorias de carga NÃO são elegíveis para devolução pelo processo padrão: Cargas perigosas classificadas nas classes 1 a 6 da ANTT..."*

**Enforcement: Misto — Código primário**
O trecho a ser exibido é recuperado e selecionado pelo pipeline — a **decisão de qual trecho exibir** é determinística. O código conhece exatamente qual chunk foi utilizado, com seu identificador, localização e texto original. A **formatação visual** (bloco recuado, rótulo, resumo quando necessário) depende do LLM ou de um componente de UI. A opção mais robusta é que o código injete o trecho diretamente na interface como elemento UI separado da geração do LLM, eliminando a dependência probabilística para esse elemento.

**Incidente relacionado: INC-01 e INC-02 — Detecção**
Em INC-01, se o trecho exibido fosse da seção 3.2 da POL-001 — que lista explicitamente cargas perigosas como exceção ao prazo geral —, o atendente perceberia imediatamente a contradição entre o trecho original e a resposta gerada ("7 dias"). Em INC-02, o trecho exibido com os multiplicadores da v1 seria evidência imediata de que a versão incorreta estava sendo utilizada, mesmo que o modelo não tivesse sinalizado o conflito. Este guardrail funciona como verificação humana no loop.

**Referências:** R-RASTR-004, R-RASTR-005, R-RASTR-006 (SPEC v1.3)

---

### GR-D-009 — Registrar log de cada interação

**Descrição da regra:**
Cada interação (pergunta + resposta) deve ser obrigatoriamente registrada em log com os seguintes campos: timestamp, ID do atendente, texto da pergunta, trechos recuperados com seus scores de similaridade, resposta gerada, fontes citadas, tipo de gap (quando aplicável) e feedback do atendente (quando fornecido). Os logs devem ser retidos por no mínimo 90 dias. O acesso é restrito exclusivamente à equipe de segurança da NovaTech.

**Objetivo da regra:**
Viabilizar auditoria retroativa de chamados reabertos, alimentar o processo de melhoria contínua da base de conhecimento e garantir conformidade com a LGPD.

**Exemplo de comportamento esperado:**
> Dado o ID de um chamado reaberto, a equipe de segurança consegue acessar o log e identificar exatamente qual trecho, de qual versão de documento, com qual score de similaridade, embasou a resposta original fornecida ao atendente.

**Enforcement: Código — Determinístico**
O logging é responsabilidade exclusiva da camada de aplicação. O código registra: timestamp (gerado pelo sistema), ID do atendente (proveniente da sessão autenticada), pergunta (texto de entrada capturado antes do envio ao modelo), trechos recuperados com scores (resultado do pipeline), resposta gerada (saída do LLM capturada antes da exibição) e fontes citadas (metadados dos chunks). Nenhum desses campos depende de instrução ao modelo — são capturados e persistidos pela infraestrutura independentemente do conteúdo gerado.

**Incidente relacionado: INC-01, INC-02 e INC-03 — Detecção**
Este guardrail não previne nenhum dos três incidentes, mas é o instrumento que permite investigá-los retroativamente. Com logs estruturados, é possível reconstruir: qual chunk foi recuperado com qual score (INC-02 — identificar que o chunk da v1 foi selecionado); qual trecho embasou a resposta sobre devolução de carga perigosa (INC-01 — rastrear se a seção 3.2 da POL-001 foi ou não recuperada); e por que o assistente respondeu "não encontrei" para SLA Gold (INC-03 — verificar se o chunk do SLA-2024 foi recuperado mas com score abaixo do limiar, ou se simplesmente não foi recuperado).

**Referências:** R-RASTR-007, R-RASTR-008, R-RASTR-009, R-RASTR-010, R-GAP-005 (SPEC v1.3)

---

### GR-D-010 — Coletar feedback do atendente

**Descrição da regra:**
Ao final de cada resposta, o assistente deve apresentar ao atendente a opção de classificar a resposta como **útil**, **incorreta** ou **incompleta**. A seleção de "incorreta" ou "incompleta" deve abrir campo opcional para descrição do problema. O feedback deve ser registrado vinculado à interação correspondente e disponibilizado no painel de administração para análise periódica.

**Objetivo da regra:**
Criar mecanismo contínuo de detecção de erros e lacunas nas respostas do assistente, permitindo evolução da base de conhecimento com base em evidências do uso real.

**Exemplo de comportamento esperado:**
> Após fornecer orientação sobre cálculo de frete, o assistente exibe: *"Esta resposta foi útil? [Útil] [Incorreta] [Incompleta]"*. O atendente seleciona "Incompleta" e adiciona: "Não mencionou a condição de aprovação para cargas acima de 5.000kg." O registro é salvo no painel de administração.

**Enforcement: Código — Determinístico**
O componente de feedback (botões e campo de texto) é um elemento de interface implementado no frontend da aplicação. Sua exibição após cada resposta é controlada pela lógica de UI — não pelo LLM. O registro do feedback é uma operação de escrita em banco de dados. Nenhuma dessas etapas passa pelo modelo: o LLM gera a resposta, e o sistema de UI exibe o componente de feedback automaticamente em seguida, independentemente do conteúdo gerado.

**Incidente relacionado: INC-01, INC-02 e INC-03 — Detecção**
Os três incidentes foram identificados em testes internos controlados. Em produção, este guardrail seria o principal mecanismo de superfície para detectá-los: em INC-01, um atendente que soubesse a regra correta sobre carga perigosa marcaria a resposta como "incorreta"; em INC-02, um atendente familiarizado com a v2 do PROC-042 marcaria como "incorreta" ao perceber os multiplicadores desatualizados; em INC-03, o atendente que sabe que o SLA Gold está documentado marcaria como "incorreta" a resposta de ausência. O padrão de feedbacks negativos sobre os mesmos temas sinalizaria a necessidade de investigação.

**Referências:** R-GAP-006 (SPEC v1.3)

---

### GR-D-011 — Distinguir gap documental de fora do escopo

**Descrição da regra:**
Antes de responder que não possui informação, o assistente deve classificar a pergunta em uma das três categorias: (1) coberta pela base — responde normalmente; (2) tema logístico NovaTech sem cobertura documental formal (gap) — informa que o tema é operacionalmente relevante mas sem documentação na base e orienta o atendente à área responsável; (3) fora do escopo do assistente — informa que o tema não pertence ao domínio de operações logísticas da NovaTech. As mensagens das categorias 2 e 3 devem ser distintas e nunca intercambiáveis. Os temas classificados como gap são: política de carga danificada em trânsito, seguro de carga, frete padrão abaixo de 500 kg e processo formal da Gestão de Riscos para cargas perigosas devolvidas.

**Objetivo da regra:**
Evitar que perguntas sobre temas operacionalmente relevantes sejam descartadas como fora de escopo, e que o atendente seja orientado ao canal correto de escalação em vez de receber uma resposta vaga.

**Exemplo de comportamento esperado:**
> Categoria 2 — gap documental: *"Esse tema faz parte das operações da NovaTech, mas ainda não há documentação formal disponível na base. Recomendo contatar a área de Comercial ou Compliance diretamente."*
> Categoria 3 — fora do escopo: *"Esse tema está fora do escopo do assistente, que responde exclusivamente sobre operações logísticas da NovaTech."*

**Enforcement: Misto — Prompt primário**
A classificação entre "gap documental NovaTech" e "fora do escopo" exige interpretação semântica da pergunta — é necessário determinar se o tema pertence ao domínio logístico da NovaTech mesmo sem documentação. Essa distinção é uma capacidade do LLM. A camada de código contribui deterministicamente ao: (a) injetar no contexto a lista fixa de gaps catalogados (seção 2.3 da SPEC), permitindo ao modelo identificar sem ambiguidade os casos de gap total conhecido; e (b) sinalizar quando nenhum chunk foi recuperado acima do limiar. Para temas de gap não catalogados ou para a fronteira entre domínios, o julgamento é do modelo.

**Incidente relacionado: INC-03 — Causa raiz**
Em INC-03, o assistente respondeu "Não encontrei informação sobre isso" para uma pergunta cujo tema — SLA Gold — estava coberto por documento formal indexado. Isso indica que o assistente classificou incorretamente a situação como gap total quando na verdade era categoria 1 (cobertura existente). Este guardrail estabelece que, antes de qualquer resposta negativa, o assistente deve verificar ativamente se o tema está coberto, e só então — se de fato nenhum chunk relevante foi recuperado — emitir a resposta de ausência com a classificação correta (gap ou fora do escopo).

**Referências:** R-NF-004, R-GAP-001 (SPEC v1.3); Seção 2.3 (SPEC v1.3)

---

## NÃO DEVE — Comportamentos Proibidos

---

### GR-N-001 — Fabricar ou extrapolar informações

**Descrição da proibição:**
O assistente não deve gerar, inferir, extrapolar ou completar informações que não estejam presentes nos documentos recuperados da base de conhecimento. Isso inclui: inventar prazos, valores, percentuais, procedimentos, contatos ou regras não documentadas; completar lacunas com suposições plausíveis; e parafrasear o FAQ de forma a omitir seu caráter informal.

**Risco evitado:**
Orientações incorretas que levem o atendente a informar dados errados ao cliente, resultando em contestações contratuais, retrabalho operacional e perda de confiança no assistente.

**Exemplo de comportamento incorreto:**
> Atendente pergunta sobre percentual de seguro de carga. O assistente responde: *"O seguro de carga custa 0,3% do valor declarado para cargas padrão"* — sem indicar que essa informação vem exclusivamente do FAQ informal e sem registrar que não há documento formal sobre o tema.

**Enforcement: Prompt — Probabilístico**
A proibição de alucinação é, por natureza, uma restrição ao comportamento generativo do LLM — e portanto probabilística. Não existe mecanismo de código capaz de detectar, antes da geração, que o modelo está prestes a fabricar uma informação. A mitigação arquitetural mais eficaz é o próprio RAG (ancora a geração em trechos recuperados), somado ao limiar de similaridade (GR-N-007) e à instrução explícita no prompt. Nenhuma dessas medidas elimina completamente o risco. Este é o guardrail de maior importância semântica e menor garantia técnica do conjunto — candidato prioritário a uma camada adicional de output filtering pós-geração.

**Incidente relacionado: INC-01 — Causa raiz**
Em INC-01, o assistente afirmou que "o prazo de devolução para carga perigosa é 7 dias" — uma extrapolação indevida da regra geral da POL-001, seção 3.1, sem considerar a seção 3.2, que lista cargas perigosas como exceção explícita. O modelo generalizou uma regra parcialmente recuperada para um caso que a documentação exclui explicitamente. Este guardrail exige que o assistente responda apenas com o que os documentos recuperados afirmam — e a POL-001, seção 3.2, afirma o contrário do que foi respondido.

**Referências:** R-GAP-002 (SPEC v1.3)

---

### GR-N-002 — Usar documento com status obsoleto para chamados novos

**Descrição da proibição:**
O assistente não deve utilizar trechos de documentos com `status=obsoleto` para responder sobre chamados com data de abertura posterior à data de corte do documento substituído. Especificamente, o PROC-042-v1 não deve ser utilizado para chamados abertos a partir de 01/12/2023.

**Risco evitado:**
Aplicação de multiplicadores regionais, fatores de peso ou prazos desatualizados, gerando cobranças incorretas, conflitos contratuais e necessidade de estorno ou renegociação.

**Exemplo de comportamento incorreto:**
> Chamado aberto em março de 2024. O assistente responde sobre frete especial para a região Norte usando o multiplicador 1,6 (PROC-042-v1) em vez do multiplicador 1,8 (PROC-042-v2, vigente).

**Enforcement: Código — Determinístico**
A filtragem de documentos obsoletos é implementada no pipeline como condição sobre metadados: chunks com `status=obsoleto` são excluídos do conjunto de resultados para chamados com data ≥ 01/12/2023. Essa filtragem ocorre antes de qualquer envio ao LLM — o modelo nunca recebe chunks obsoletos para chamados novos. A robustez depende da corretude dos metadados de status, garantida pelo requisito R-CONT-001b (atualização automática de status no momento da ingestão).

**Incidente relacionado: INC-02 — Causa raiz**
Este guardrail e o GR-D-002 formam a dupla defesa contra INC-02, atuando em camadas complementares. GR-D-002 prioriza o documento vigente quando ambos estão disponíveis; GR-N-002 vai além e impede que o documento obsoleto sequer chegue ao LLM. Em INC-02, o PROC-042-v1 foi utilizado apesar de existir uma versão mais recente com `status=vigente`. A proibição explícita de uso do documento obsoleto torna essa falha impossível quando o metadado de status está corretamente atribuído.

**Referências:** R-CONT-001b, R-CONT-002, R-CONT-006, F-002, F-003 (SPEC v1.3)

---

### GR-N-003 — Confirmar tiers inexistentes

**Descrição da proibição:**
O assistente não deve confirmar, validar ou operar com tiers de cliente que não sejam Gold, Silver ou Standard. Menções a tiers como "Platinum", "Diamond", "Premium" ou qualquer outro não previsto no SLA-2024 devem ser corrigidas pelo assistente, com orientação ao atendente de verificar o número do contrato e os tiers oficiais.

**Risco evitado:**
Criação de expectativas incorretas no atendente e no cliente sobre SLAs diferenciados inexistentes, potencialmente levando a compromissos não suportados operacionalmente pela NovaTech.

**Exemplo de comportamento incorreto:**
> Atendente informa: "O cliente diz ser Platinum." O assistente responde orientando sobre SLAs de um suposto tier Platinum, em vez de informar que esse tier não existe e solicitar o número do contrato para verificação do tier correto.

**Enforcement: Prompt — Probabilístico**
A detecção de um tier inválido mencionado em linguagem natural é uma tarefa de reconhecimento semântico — competência do LLM. O código poderia implementar uma blocklist de strings ("Platinum", "Diamond"), mas isso seria frágil diante de variações ortográficas e contextos diferentes. A abordagem mais robusta é instrução explícita no prompt com os três valores válidos enumerados. O risco de falha é baixo dado que o conjunto de valores é pequeno e bem definido, tornando este um dos guardrails de prompt com menor probabilidade de desvio na prática.

**Incidente relacionado: INC-03 — Fator contribuinte**
Em INC-03, a pergunta era sobre SLA Gold — um tier válido. Porém, o padrão de falha é similar: o assistente não localizou nem apresentou a informação de SLA correspondente ao tier informado. Se um atendente tivesse mencionado um tier inválido (ex.: "Platinum") e o assistente tivesse confirmado SLAs fictícios para esse tier, o resultado seria análogo ao INC-03 — orientação incorreta sobre prazos de atendimento. Este guardrail fecha esse vetor de falha adjacente ao INC-03.

**Referências:** SLA-2024, seção 1 (Anexo A); FAQ-Item 15

---

### GR-N-004 — Apresentar fonte informal como oficial

**Descrição da proibição:**
O assistente não deve apresentar informações provenientes do FAQ-Atendimento sem a sinalização explícita de seu caráter informal e não validado. É proibido omitir a advertência de fonte informal, mesmo que a informação do FAQ pareça consistente com documentos primários.

**Risco evitado:**
O atendente tomar decisões operacionais baseadas em práticas informais não homologadas como se fossem políticas oficiais, expondo a NovaTech a inconsistências e eventuais contestações.

**Exemplo de comportamento incorreto:**
> O assistente responde sobre o processo de carga danificada citando apenas os passos do FAQ-Item 38 sem qualquer aviso de que se trata de fonte informal não validada por Compliance ou Operações.

**Enforcement: Misto — Código primário**
Análogo ao GR-D-004: a detecção de que o chunk provém de fonte informal é determinística, baseada no metadado `tipo=informal`. O pipeline injeta a flag de aviso automaticamente ao recuperar qualquer chunk do FAQ. A omissão do aviso seria uma falha do LLM em seguir uma instrução explícita — possível, mas improvável quando a flag é proeminente no contexto. Como mitigação adicional, o aviso pode ser injetado diretamente no template de UI, desacoplando-o completamente da geração do modelo.

**Incidente relacionado: INC-01 — Fator contribuinte**
O FAQ-Item 3 descreve o encaminhamento de cargas perigosas ao ramal 4500, mas acrescenta que "já tiveram casos em que o pessoal de Riscos autorizou exceção — então não diga que é impossível". Essa linguagem informal pode ter levado o modelo a suavizar a regra oficial da POL-001 (que proíbe devolução padrão) e gerar a resposta incorreta de "7 dias". Se o FAQ-Item 3 tivesse sido apresentado com aviso explícito de fonte informal, o atendente saberia que a regra definitiva estava na POL-001 — e o modelo teria menos razão para misturar as duas fontes.

**Referências:** R-RASTR-003, R-GAP-003, F-005 (SPEC v1.3)

---

### GR-N-005 — Autorizar descontos sem respaldo documental

**Descrição da proibição:**
O assistente não deve afirmar, mesmo que implicitamente, que o atendente tem autonomia para conceder descontos de frete. Descontos de volume previstos no PROC-042-v2 (a partir de 8 fretes especiais/mês: 5% sobre o multiplicador regional; a partir de 15 fretes/mês: 10%) devem ser informados como regras aplicáveis, não como ato discricionário do atendente. Descontos acima desses patamares devem ser encaminhados à Diretoria Comercial.

**Risco evitado:**
Concessão irregular de descontos sem respaldo contratual, impacto na margem operacional e criação de precedentes não autorizados.

**Exemplo de comportamento incorreto:**
> Atendente pergunta: "Posso dar desconto para o cliente?" O assistente responde: "Sim, você pode aplicar um desconto de até 10%", sem verificar o volume de fretes e sem indicar que descontos acima dos previstos na tabela requerem aprovação da Diretoria Comercial.

**Enforcement: Prompt — Probabilístico**
A proibição de atribuir autonomia indevida ao atendente é uma restrição semântica ao conteúdo gerado pelo LLM. O código injeta no contexto as regras de desconto do PROC-042-v2, fornecendo ao modelo a informação correta sobre os limites. Mas a verificação de que a resposta gerada não ultrapassa esses limites nem atribui autonomia indevida é uma análise semântica da saída — probabilística. Para o nível de risco deste guardrail, enforcement via prompt com contexto bem estruturado é aceitável, com monitoramento contínuo via feedback (GR-D-010).

**Incidente relacionado: INC-01 — Fator contribuinte**
A conexão é indireta, mas estruturalmente relevante: em INC-01, o modelo aplicou a regra geral (prazo de 7 dias) a um caso que era explicitamente exceção. O mesmo padrão de falha — generalizar uma regra sem verificar suas exceções e condições — poderia levar o assistente a afirmar autonomia de desconto sem verificar os limites documentados. Ambos os casos representam o modelo respondendo além do que a documentação autoriza.

**Referências:** PROC-042-v2, seção 4 (Anexo A); FAQ-Item 45

---

### GR-N-006 — Dar orientação definitiva sobre temas com gaps documentais

**Descrição da proibição:**
Para os temas identificados como gaps documentais — política de carga danificada em trânsito, seguro de carga, frete padrão abaixo de 500 kg e processo formal da Gestão de Riscos para cargas perigosas devolvidas —, o assistente não deve fornecer orientação definitiva, mesmo que existam informações no FAQ informal. Nesses casos, deve informar a ausência de documentação formal e indicar a área responsável.

**Risco evitado:**
Orientação baseada em informações não homologadas em temas de alto impacto operacional ou contratual, gerando comprometimentos que a NovaTech não pode garantir formalmente.

**Exemplo de comportamento incorreto:**
> Atendente pergunta sobre percentual do seguro de carga. O assistente responde com os valores do FAQ-Item 22 (0,3% para cargas padrão, 0,8% para cargas perigosas) sem indicar que não há documento formal sobre o tema e que contratos mais antigos podem ter percentuais diferentes.

**Enforcement: Misto — Código primário**
A detecção de que nenhum chunk de fonte primária foi recuperado para o tema — ou que todos os chunks provêm exclusivamente de fonte informal para um gap catalogado — é determinística: o pipeline verifica scores e metadados. Quando essa condição é identificada, o pipeline pode bloquear a geração de resposta substantiva e forçar um template de resposta de gap com o canal de escalação adequado. A camada probabilística reside na redação final pelo LLM, mitigada por template fixo no prompt.

**Incidente relacionado: INC-01 — Fator contribuinte**
O processo de devolução de cargas perigosas é exatamente um dos gaps documentais desta lista: a POL-001 proíbe a devolução padrão e orienta a chamar o ramal 4500, mas não documenta o que acontece depois. Em INC-01, o assistente respondeu com orientação definitiva ("7 dias") em vez de sinalizar que as devoluções de cargas perigosas não seguem o processo padrão e devem ser encaminhadas à Gestão de Riscos. Este guardrail teria impedido a resposta definitiva incorreta e substituído pelo comportamento correto de escalação.

**Referências:** R-GAP-001, R-GAP-002, R-NF-004 (SPEC v1.3); Seção 2.3 (SPEC v1.3)

---

### GR-N-007 — Usar trecho abaixo do limiar de similaridade

**Descrição da proibição:**
O assistente não deve utilizar trechos recuperados com score de similaridade inferior a 0,75 na composição de respostas. Trechos abaixo desse limiar devem ser descartados. Se nenhum trecho atingir o mínimo, o assistente deve adotar o comportamento de gap total (GR-W-004).

**Risco evitado:**
Respostas baseadas em trechos semanticamente distantes da pergunta, aumentando o risco de informações incorretas ou fora de contexto.

**Exemplo de comportamento incorreto:**
> Pergunta sobre SLA de incidente crítico retorna, com score 0,60, um trecho sobre disponibilidade do portal de tracking. O assistente utiliza esse trecho para compor a resposta em vez de sinalizar ausência de resultado relevante.

**Enforcement: Código — Determinístico**
O limiar de 0,75 é um valor numérico aplicado diretamente sobre os scores de similaridade retornados pelo modelo de embeddings — operação de filtragem em código, executada antes de qualquer envio ao LLM. Chunks com score < 0,75 são removidos do conjunto de resultados; o modelo nunca os recebe. Este é um dos guardrails mais limpos do conjunto: a regra é um número, a verificação é uma comparação aritmética, o resultado é binário (incluir / descartar).

**Incidente relacionado: INC-03 — Causa raiz**
Uma hipótese técnica plausível para INC-03 é que o chunk do SLA-2024 sobre SLA Gold foi recuperado, mas com score abaixo do limiar vigente no momento do teste — e portanto descartado pelo pipeline, levando o assistente a responder "não encontrei informação". Este guardrail define que o limiar correto é 0,75 e que ele deve ser validado durante a homologação (R-GAP-002b). Se o score do chunk de SLA Gold ficou abaixo desse limiar, isso indica que a estratégia de chunking (DTP-001) ou o modelo de embeddings precisam ser revisados — não que a informação não existe na base.

**Referências:** R-GAP-002b (SPEC v1.3)

---

### GR-N-008 — Expor dados de log a perfis não autorizados

**Descrição da proibição:**
O assistente e o sistema que o suporta não devem permitir acesso aos logs de interação por nenhum perfil além da equipe de segurança da NovaTech. Isso inclui administradores do assistente, atendentes e gestores operacionais. Dados pessoais presentes nos logs devem ser tratados em conformidade com a LGPD.

**Risco evitado:**
Violação de privacidade dos atendentes, exposição de dados operacionais sensíveis e descumprimento da LGPD (Lei nº 13.709/2018).

**Exemplo de comportamento incorreto:**
> Um atendente acessa o painel administrativo e consegue visualizar os logs de interação de outro atendente, incluindo nome, ID e perguntas realizadas, sem passar pelo controle de acesso da equipe de segurança.

**Enforcement: Código — Determinístico**
Controle de acesso a dados é responsabilidade exclusiva da infraestrutura: camada de autenticação (Azure AD, R-NF-005), autorização (verificação de perfil antes de servir qualquer endpoint de log) e criptografia em repouso. O LLM não está na cadeia de acesso a esse dado — os logs são escritos pela aplicação e lidos por sistemas de auditoria. Qualquer instrução de prompt seria irrelevante aqui. Este guardrail deve ser verificado em teste de penetração e revisão de configuração de IAM, não em teste de comportamento do modelo.

**Incidente relacionado: INC-01, INC-02 e INC-03 — Não aplicável diretamente**
Este guardrail não previne nem detecta nenhum dos três incidentes de comportamento. Sua função é proteger a integridade do próprio mecanismo de investigação: os logs que permitem reconstruir INC-01, INC-02 e INC-03 (GR-D-009) só cumprem seu papel de auditoria se estiverem protegidos contra acesso não autorizado. Um log acessível por perfis indevidos poderia ser adulterado, comprometendo a capacidade de investigar os incidentes retroativamente.

**Referências:** R-RASTR-009, R-RASTR-010 (SPEC v1.3)

---

## QUANDO EM DÚVIDA — Comportamentos de Fallback

---

### GR-W-001 — Versão do PROC-042 sem data do chamado

**Situação de dúvida:**
O atendente pergunta sobre cálculo de frete especial, mas não informa a data de abertura do chamado, impossibilitando a seleção da versão correta do PROC-042.

**Ação esperada:**
O assistente deve solicitar ao atendente a data de abertura do chamado antes de fornecer qualquer valor de multiplicador regional, fator de peso ou prazo adicional. Não deve assumir versão padrão nem fornecer os dois conjuntos de valores simultaneamente sem a data.

**Exemplo de resposta ao usuário:**
> *"Para aplicar a versão correta do procedimento de frete especial, preciso saber a data de abertura do chamado. Chamados abertos até 30/11/2023 seguem a tabela anterior (PROC-042-v1) e chamados abertos a partir de 01/12/2023 seguem a tabela revisada (PROC-042-v2). Por favor, informe a data de abertura do chamado."*

**Enforcement: Misto — Código primário**
A detecção de ausência da data do chamado no contexto da conversa é determinística: o pipeline verifica se o campo de data foi fornecido antes de executar a seleção de versão do PROC-042. Se ausente, o pipeline bloqueia a recuperação de chunks do PROC-042 e injeta no contexto a instrução de solicitar a data. A **formulação da solicitação** — linguagem, clareza, explicação do motivo — é gerada pelo LLM. A divisão é clara: o código decide "não é possível responder ainda"; o LLM decide como comunicar isso ao atendente.

**Incidente relacionado: INC-02 — Fator contribuinte**
Em INC-02, o assistente respondeu com multiplicadores da v1 sem verificar a data do chamado. Se a data tivesse sido verificada e estivesse ausente, este guardrail teria bloqueado a resposta e solicitado a informação — impedindo que qualquer versão do PROC-042 fosse aplicada sem a confirmação do contexto temporal correto. A ausência de data é a pré-condição que cria o risco de seleção de versão incorreta; este guardrail elimina essa pré-condição.

**Referências:** R-CONT-007 (SPEC v1.3)

---

### GR-W-002 — Dados insuficientes para cálculo de frete

**Situação de dúvida:**
O atendente solicita o cálculo de frete especial sem informar peso da carga e/ou região de destino, tornando impossível a aplicação dos multiplicadores e fatores corretos.

**Ação esperada:**
O assistente deve identificar quais dados estão ausentes e solicitá-los de forma objetiva ao atendente antes de qualquer tentativa de cálculo. Não deve fornecer estimativas ou intervalos de valores.

**Exemplo de resposta ao usuário:**
> *"Para calcular o frete especial, preciso das seguintes informações: (1) peso total da carga em kg; (2) região de destino (Sul, Sudeste, Centro-Oeste, Nordeste ou Norte). Por favor, informe esses dados."*

**Enforcement: Misto — Código primário**
O pipeline pode verificar deterministicamente se os campos necessários (peso e região) foram fornecidos — seja como campos estruturados via integração com sistema de chamados, seja por extração de entidades do texto. Se ausentes, o pipeline bloqueia o cálculo e aciona o template de solicitação. Para o caso em que os dados chegam apenas em linguagem natural não estruturada, a extração de "peso" e "região" envolve NLP — probabilístico. Nesse subcase, o LLM precisa identificar a ausência e solicitar os dados, controlável por prompt mas não garantido.

**Incidente relacionado: INC-02 — Fator contribuinte**
Em INC-02, o erro foi de versão do documento, não de dados ausentes. No entanto, este guardrail é fator contribuinte porque um cálculo de frete realizado com dados incompletos — por exemplo, sem a região de destino — poderia levar o assistente a aplicar um multiplicador padrão ou arbitrário, agravando o impacto de um erro de versão já existente. Dados completos e verificados são pré-condição para que qualquer cálculo de frete, mesmo com a versão correta do PROC-042, produza um resultado confiável.

**Referências:** R-GAP-004 (SPEC v1.3); PROC-042-v2, seção 2 (Anexo A)

---

### GR-W-003 — Tema presente apenas no FAQ informal

**Situação de dúvida:**
O atendente pergunta sobre um tema para o qual não existe documento primário (normativo, procedimento ou contratual) na base, mas existe informação no FAQ-Atendimento.

**Ação esperada:**
O assistente deve fornecer a informação disponível no FAQ, acompanhada de aviso explícito e visualmente destacado sobre o caráter informal e não validado da fonte. Deve orientar o atendente a confirmar a informação com a área responsável antes de repassá-la ao cliente.

**Exemplo de resposta ao usuário:**
> *"Encontrei informação sobre este tema apenas no FAQ-Atendimento, que é um documento informal não validado por Compliance ou Operações. Segundo o FAQ (Item 32): [informação]. Recomendo confirmar com a área responsável antes de orientar o cliente. (Fonte: FAQ-Atendimento — documento informal, não validado por Compliance.)"*

**Enforcement: Misto — Código primário**
O pipeline detecta deterministicamente que o único chunk relevante possui `tipo=informal`, acionando o template de aviso de fonte informal injetado no contexto. A **redação do aviso e a apresentação da informação** dependem do LLM seguir o template. O risco principal é o modelo apresentar a informação do FAQ de forma assertiva demais sem qualificá-la adequadamente. Como mitigação, o aviso pode ser renderizado como componente fixo de UI, deixando ao LLM apenas a apresentação do conteúdo.

**Incidente relacionado: INC-01 — Fator contribuinte**
O FAQ-Item 3 orienta: "não diga que é impossível" — uma instrução que, sem o aviso de fonte informal, pode levar o LLM a suavizar a regra oficial da POL-001 sobre cargas perigosas. Se este guardrail tivesse sido aplicado, o atendente receberia a informação do FAQ acompanhada do aviso explícito de que a orientação definitiva está na documentação formal — e o modelo teria mais contexto para não generalizar o prazo de 7 dias para um caso que a POL-001 exclui explicitamente.

**Referências:** R-GAP-003, R-RASTR-003 (SPEC v1.3)

---

### GR-W-004 — Tema com gap total na base

**Situação de dúvida:**
O atendente pergunta sobre um tema que não possui nenhuma cobertura documental na base — nem em fontes primárias nem no FAQ — ou nenhum trecho recuperado atingiu o limiar mínimo de similaridade de 0,75.

**Ação esperada:**
O assistente deve informar que não encontrou documentação disponível sobre o tema, classificar se é um gap documental reconhecido (tema logístico NovaTech sem cobertura formal) ou fora do escopo, e indicar o canal de escalação adequado conforme o tema. Deve registrar o tipo de gap no log interno.

**Exemplo de resposta ao usuário (gap documental):**
> *"Não encontrei documentação formal sobre este tema na base de conhecimento disponível. Esse assunto faz parte das operações da NovaTech, mas ainda não possui política ou procedimento formalizado. Recomendo contatar diretamente [área responsável] para obter orientação."*

**Exemplo de resposta ao usuário (fora do escopo):**
> *"Esse tema está fora do escopo do assistente, que responde exclusivamente sobre operações logísticas da NovaTech."*

**Enforcement: Código — Determinístico**
A condição de gap total é verificável de forma binária pelo pipeline: nenhum chunk recuperado atingiu score ≥ 0,75. Quando confirmada, o pipeline bloqueia a geração de resposta substantiva e força um template de resposta pré-definido. O LLM, nesse caso, recebe apenas o template de gap com o canal de escalação adequado — determinado pelo pipeline com base na lista de gaps catalogados. A adaptação linguística do template é o único elemento probabilístico, com risco residual mínimo dado um template suficientemente específico.

**Incidente relacionado: INC-03 — Causa raiz**
Em INC-03, o assistente usou a mensagem de gap total ("Não encontrei informação sobre isso") para uma pergunta sobre SLA Gold — cujo documento estava indexado. Isso indica que o pipeline ativou indevidamente o comportamento de gap total, seja por score abaixo do limiar (problema de chunking ou embeddings), seja por falha na indexação. Este guardrail define o comportamento correto quando o gap total é genuíno; investigar por que INC-03 o ativou indevidamente é a ação corretiva necessária — revisão do DTP-001 (chunking) e DTP-002 (modelo LLM).

**Referências:** R-GAP-001, R-GAP-002b, R-GAP-005, R-NF-004 (SPEC v1.3)

---

### GR-W-005 — Assistente indisponível

**Situação de dúvida:**
O assistente está temporariamente indisponível, seja por falha técnica não planejada ou por janela de atualização planejada da base de conhecimento.

**Ação esperada:**
O sistema deve exibir mensagem clara, diferenciando os dois estados. Em ambos os casos, deve indicar onde o atendente pode consultar a documentação oficial durante a indisponibilidade.

**Exemplo de resposta ao usuário (falha):**
> *"O assistente está temporariamente indisponível. Consulte a documentação oficial em [caminho/URL definido pela NovaTech] ou aguarde o restabelecimento do serviço."*

**Exemplo de resposta ao usuário (atualização planejada):**
> *"O assistente está em atualização e voltará em breve. Consulte a documentação oficial em [caminho/URL] durante esse período."*

**Enforcement: Código — Determinístico**
O estado de indisponibilidade é detectado pela infraestrutura (health check, status de deployment, flag de manutenção). O sistema exibe uma mensagem estática pré-definida no nível da aplicação, antes de qualquer tentativa de invocar o LLM. O modelo não está envolvido: a mensagem é um artefato de UI ou página de erro. A distinção entre "falha" e "atualização planejada" é determinada pelo estado do sistema (flag `maintenance_mode=true` vs. exceção de conexão) e mapeada para mensagens distintas em código.

**Incidente relacionado: INC-03 — Fator contribuinte**
Uma hipótese alternativa para INC-03 é que o assistente estava em estado degradado durante o teste — com a base de conhecimento parcialmente disponível ou em processo de reindexação — e o chunk do SLA-2024 temporariamente inacessível. Se este guardrail estivesse implementado, o atendente teria recebido uma mensagem de indisponibilidade com orientação para consultar a documentação diretamente, em vez de uma resposta de "não encontrei" que parece definitiva mas pode ser circunstancial.

**Referências:** R-NF-006 (SPEC v1.3)

---

### GR-W-006 — FAQ conflita com documento primário recém-atualizado

**Situação de dúvida:**
Um documento primário foi atualizado (ingestão em até 4 horas úteis), mas o FAQ ainda não foi atualizado (prazo de até 24 horas úteis), criando uma janela de inconsistência em que ambas as fontes coexistem com conteúdos divergentes sobre o mesmo tema.

**Ação esperada:**
O assistente deve descartar o trecho do FAQ conflitante e basear a resposta exclusivamente na fonte primária atualizada. Deve informar explicitamente ao atendente que a versão do FAQ foi desconsiderada por ser menos recente. O item do FAQ conflitante deve ser marcado internamente para revisão prioritária.

**Exemplo de resposta ao usuário:**
> *"Esta resposta é baseada na documentação oficial. Uma versão anterior desta orientação existia no FAQ e foi desconsiderada por ser menos recente que o documento oficial. O FAQ será atualizado em breve pela equipe responsável."*

**Enforcement: Código — Determinístico**
O pipeline possui a informação necessária para detectar esse conflito deterministicamente: ao compor os chunks para envio ao LLM, verifica se existe um chunk `tipo=informal` e um chunk `tipo=normativo/procedimento/contratual` sobre o mesmo tema, com datas de ingestão diferentes (primário mais recente). Quando essa condição é detectada, o pipeline descarta o chunk do FAQ e injeta o aviso de descarte como texto fixo no contexto. O LLM recebe apenas o chunk primário e o aviso pré-formulado — não decide se deve descartar o FAQ. A marcação do item do FAQ para revisão é também uma operação de banco de dados, independente do modelo.

**Incidente relacionado: INC-01 e INC-02 — Fator contribuinte**
Em INC-01, o FAQ-Item 3 (com linguagem informal sobre exceções) pode ter coexistido com a POL-001 (com a regra formal de proibição) e o pipeline não priorizou o documento primário. Em INC-02, uma versão desatualizada do FAQ poderia ter reforçado os multiplicadores da v1 mesmo após a PROC-042-v2 ter sido ingerida. Este guardrail garante que, em ambos os casos, o documento primário mais recente prevalece e o FAQ conflitante é explicitamente descartado — eliminando a ambiguidade que contribuiu para os dois incidentes.

**Referências:** R-ATU-004b (SPEC v1.3)

---

## Observações Finais — Ambiguidades e Lacunas Identificadas

As observações a seguir registram situações em que a documentação disponível é insuficiente, contraditória ou ambígua, e que podem exigir definições adicionais pelas áreas responsáveis antes do go-live do assistente.

### 1. PROC-042: ausência de marcação formal de obsolescência

O PROC-042-v1 e o PROC-042-v2 coexistem sem que nenhum deles contenha marcação formal de obsolescência no repositório documental da NovaTech. A SPEC v1.3 resolve operacionalmente esse conflito via metadados de ingestão (F-002, F-003) e regra de data do chamado (R-CONT-006), mas a inconsistência na documentação de origem permanece. **Recomendação:** solicitar às áreas de Operações e Comercial o arquivamento formal do PROC-042-v1 no SharePoint. **Conexão com incidentes:** INC-02 é a consequência direta dessa lacuna.

### 2. Gaps documentais não cobertos por guardrails primários

Quatro temas operacionais relevantes não possuem documento formal na base: (a) política de carga danificada em trânsito; (b) seguro de carga; (c) frete padrão abaixo de 500 kg; (d) processo formal da Gestão de Riscos para cargas perigosas devolvidas. Os guardrails GR-N-006, GR-W-003 e GR-W-004 cobrem o comportamento do assistente nesses casos, mas não eliminam o risco operacional subjacente. **Conexão com incidentes:** INC-01 envolveu exatamente um desses gaps — o processo de devolução de cargas perigosas não está documentado além do ramal 4500 na POL-001. **Recomendação:** formalizar a criação dos documentos junto às áreas responsáveis.

### 3. Frete expresso para cargas perigosas: ausência de documento formal

O FAQ-Item 32 menciona a possibilidade de envio de carga perigosa com frete expresso mediante autorização do Compliance, mas não existe PROC ou POL que formalize esse processo. **Recomendação:** avaliar com Compliance a criação de um procedimento formal antes da entrada em produção.

### 4. Processo de escalação para Gestão de Riscos (ramal 4500)

A POL-001 (seção 3.2) orienta o cliente a contatar a Gestão de Riscos pelo ramal 4500 para devoluções de cargas perigosas, mas não existe procedimento documentado sobre o fluxo que essa área segue após o contato. **Conexão com incidentes:** a ausência desse procedimento é um fator estrutural que contribuiu para INC-01 — sem documentação do processo pós-escalação, o assistente não tem como orientar corretamente além do encaminhamento ao ramal.

### 5. Carga danificada: processo descrito apenas no FAQ

O FAQ-Item 38 descreve um processo com prazo de 48 horas, obrigatoriedade de fotos e laudo, e encaminhamento para sinistros@novatech.com.br. Entretanto, esse processo não possui POL ou PROC formal. Qualquer orientação prestada pelo assistente sobre esse tema será baseada exclusivamente em fonte informal, com os riscos associados descritos no GR-N-006.

### 6. Valor-limite para incidente crítico de rastreamento no FAQ

O FAQ-Item 27 menciona o valor de R$ 50.000 como referência informal para abertura de chamado de rastreamento com prioridade alta, enquanto o SLA-2024 (documento formal) define R$ 100.000 como limiar para incidente crítico. O assistente deve seguir o critério formal do SLA-2024 (GR-D-006), ignorando o valor do FAQ para esse fim específico.

### 7. Decisões técnicas pendentes com impacto em guardrails e incidentes

Duas decisões técnicas registradas na SPEC v1.3 (DTP-001 — estratégia de chunking; DTP-002 — seleção e versionamento do modelo LLM) ainda não foram resolvidas. **Conexão com incidentes:** INC-03 pode ter origem direta nessas decisões pendentes — uma estratégia de chunking inadequada para tabelas de SLA (identificada em DTP-001 como risco específico) explicaria por que o chunk do SLA-2024 Gold não foi recuperado com score suficiente. **Recomendação:** tratar DTP-001 e DTP-002 como bloqueadores para a resolução definitiva de INC-03.

---

*Versão 3.0 — Confidencial — NovaTech Logística*
*Elaborado com base exclusivamente nos documentos: POL-001 v3.1, PROC-042-v1, PROC-042-v2, SLA-2024 v2024.1, FAQ-Atendimento e SPEC-RAG-NovaTech v1.3.*
