# RAG-SPEC-001 — Especificação de Requisitos: Pipeline RAG
## Assistente de Atendimento — NovaTech Logística

| Campo | Valor |
|---|---|
| Documento | RAG-SPEC-001 |
| Versão | 1.0 — Inicial |
| Responsável | Product Specialist — Projeto Assistente IA |
| Revisores | Diretoria de Operações, Compliance, TI |
| Status | Em revisão — aguardando aprovação |

---

## 1. Contexto e objetivo

Este documento especifica os requisitos que o pipeline de RAG (Retrieval-Augmented Generation) do assistente de atendimento da NovaTech deve satisfazer. Ele é a referência contratual entre o time de produto, o time de engenharia e os revisores de compliance durante o desenvolvimento e a operação do sistema.

O assistente tem como usuário primário o atendente humano — não o cliente final. Seu papel é reduzir o tempo de consulta a documentações dispersas e eliminar respostas inconsistentes causadas pela coexistência de versões conflitantes de procedimentos.

> **Dados de discovery que motivam esta especificação**
>
> - **4,1 fontes** consultadas por chamado em média (6,2 em chamados de frete especial) — reflexo direto da duplicidade do PROC-042 sem hierarquia declarada.
> - **22% dos chamados** reabertos em até 48 horas, majoritariamente em tópicos sem documentação oficial (rastreamento, seguro, sinistro).
> - Dúvidas mais frequentes: prazos de entrega (35%), regras de frete (25%), política de devolução (20%), SLA e tiers (10%) e outros (10%).

---

## 2. Fontes de dados a indexar

O pipeline deve indexar os documentos listados abaixo. Cada fonte recebe um nível de confiabilidade que o assistente usa para priorizar respostas em caso de conflito e para qualificar seu output ao atendente.

### 2.1 Documentos normativos — confiabilidade ALTA

Estes documentos são a fonte de verdade do assistente. Em qualquer conflito com outras fontes, prevalecem. O atendente deve ser informado quando a resposta vem exclusivamente daqui.

| Documento | Descrição | Versão ativa | Observação |
|---|---|---|---|
| POL-001 | Política de devolução de mercadorias | 3.1 (jan/2024) | — |
| PROC-042-v2 | Cálculo de frete especial (revisado) | 2.0 (nov/2023) | Substitui PROC-042-v1. V1 deve ser arquivada. |
| SLA-2024 | Tabela de SLA por tipo de cliente | 2024.1 (jan/2024) | Documento contratual — compromisso formal. |

### 2.2 FAQ interno — confiabilidade MÉDIA

O FAQ-Atendimento pode ser indexado como fonte auxiliar, com restrições. Por ser um documento informal, não validado por Compliance ou Operações, o assistente deve utilizá-lo apenas quando não houver resposta nos documentos normativos, e deve sempre sinalizar ao atendente que a informação tem origem em documentação informal.

> ⚠️ **Restrição crítica — FAQ-Atendimento**
>
> Nenhum item do FAQ deve ser apresentado ao atendente sem o aviso: *"Esta informação vem do FAQ interno, que não foi validado formalmente. Confirme com a documentação oficial antes de comprometer com o cliente."* Itens que contradizem diretamente um documento normativo devem ser suprimidos na resposta, com o conteúdo normativo prevalecendo.

### 2.3 Fontes a NÃO indexar nesta versão

Os itens abaixo foram identificados no discovery como ausentes ou sem cobertura documental suficiente. Não devem ser adicionados ao índice sem a criação prévia de documentação formal e aprovação de Compliance.

- **Política de carga danificada:** existe apenas no FAQ (item 38). Ausência de POL ou PROC formal.
- **Seguro de carga:** mencionado no FAQ (item 22), sem documento formal com percentuais validados.
- **Frete padrão (abaixo de 500 kg):** não há procedimento documentado na base atual.
- **PROC-042-v1:** deve ser arquivado e removido do índice após confirmação da equipe Comercial de que PROC-042-v2 é a versão vigente (ver seção 3).
- **Tabela mensal de fretes-base** (arquivo `.xlsx` em `\\novatech-fs\comercial`): fonte externa ao SharePoint, requer solução de integração separada.

---

## 3. Tratamento de documentos contraditórios

Esta é a seção de maior criticidade operacional. O discovery identificou conflitos ativos na base documental que, se não tratados na camada de pipeline, serão reproduzidos pelo assistente — replicando exatamente o problema que o projeto pretende resolver.

> 🔴 **Conflito ativo: PROC-042-v1 vs. PROC-042-v2**
>
> As duas versões coexistem no SharePoint sem hierarquia declarada. As diferenças impactam diretamente o valor calculado de frete e o prazo informado ao cliente:
> - Multiplicadores regionais divergem em até 0,2 pontos.
> - Fatores de peso divergem entre 1,5 (v1) e 1,4 (v2) para cargas acima de 3.000 kg.
> - Prazo adicional é +2 dias úteis na v1 e +3 dias úteis na v2.

### 3.1 Regras de hierarquia de fontes

O pipeline deve implementar e respeitar a seguinte hierarquia ao montar o contexto para geração de resposta:

1. Documentos normativos com versão mais recente e responsável formal declarado (POL, PROC, SLA).
2. Documentos normativos com versão mais antiga da mesma família, somente se explicitamente não substituídos.
3. FAQ interno — somente quando nenhuma fonte normativa cobrir o tópico.

### 3.2 Comportamento ao detectar conflito

O pipeline considera que há conflito sempre que dois ou mais chunks recuperados para a mesma query contiverem **qualquer divergência numérica referente ao mesmo assunto** — independentemente da magnitude da diferença. Isso inclui: valores monetários, percentuais, multiplicadores, fatores de peso, prazos em dias ou horas, e quantidades. A comparação é feita entre chunks de documentos diferentes ou de versões diferentes do mesmo documento.

Para que o pipeline possa determinar operacionalmente quando dois valores numéricos tratam do "mesmo assunto", cada chunk deve carregar o metadado `topic_key` — uma tag de vocabulário controlado atribuída no momento da ingestão. A detecção de conflito só é executada entre chunks que compartilham o mesmo `topic_key`. Chunks com `topic_key` distintos nunca são comparados entre si, mesmo que o modelo os recupere para a mesma query.

**Vocabulário controlado de `topic_key` — base NovaTech v1.0:**

| `topic_key` | Descrição | Documentos que o usam |
|---|---|---|
| `frete.multiplicador_regional` | Fatores multiplicadores por região de destino | PROC-042-v1, PROC-042-v2 |
| `frete.fator_peso` | Fatores de peso por faixa de kg | PROC-042-v1, PROC-042-v2 |
| `frete.prazo_adicional` | Dias úteis adicionais para frete especial | PROC-042-v1, PROC-042-v2 |
| `frete.desconto_volume` | Percentual de desconto por volume mensal de fretes | PROC-042-v2 |
| `devolucao.prazo_solicitacao` | Prazo em dias úteis para solicitar devolução | POL-001 |
| `devolucao.prazo_coleta` | Prazo em dias úteis para agendamento de coleta reversa | POL-001 |
| `devolucao.prazo_reembolso` | Prazo em dias úteis para processamento de reembolso | POL-001 |
| `devolucao.prazo_triagem` | Prazo em horas úteis para triagem do chamado | POL-001 |
| `sla.primeira_resposta_geral` | Tempo de primeira resposta para chamados gerais por tier | SLA-2024 |
| `sla.resolucao_geral` | Tempo de resolução para chamados gerais por tier | SLA-2024 |
| `sla.primeira_resposta_critico` | Tempo de primeira resposta para incidentes críticos por tier | SLA-2024 |
| `sla.resolucao_critico` | Tempo de resolução para incidentes críticos por tier | SLA-2024 |
| `sla.disponibilidade_portal` | Percentual de disponibilidade do portal de tracking por tier | SLA-2024 |
| `sla.penalidade_violacao` | Percentual de crédito por violação de SLA | SLA-2024 |
| `sla.criterio_tier` | Valores de contrato e volume que definem elegibilidade de tier | SLA-2024 |
| `sla.incidente_critico` | Valores monetários e quantitativos que definem incidente crítico | SLA-2024 |

> **Regra de manutenção:** sempre que um novo documento com valores numéricos for adicionado à base, o responsável pela ingestão deve verificar se os `topic_key` existentes cobrem os novos tópicos. Se não cobrirem, novos valores devem ser adicionados a esta tabela antes da ingestão, com aprovação do time de produto. Nenhum chunk numérico deve ser ingerido sem `topic_key` atribuído.

Quando conflito for detectado, o assistente deve:

- Usar exclusivamente o chunk de maior confiabilidade (documento normativo mais recente) para compor a resposta.
- Exibir ao atendente um aviso explícito de conflito detectado, identificando as fontes divergentes e indicando qual foi usada.
- **Nunca** fazer média, interpolação ou síntese entre valores conflitantes (ex: calcular frete com multiplicador médio entre v1 e v2). A resposta deve ser baseada em uma única fonte autorizada.
- Registrar o evento de conflito em log para auditoria (ver seção 6).

> 💬 **Exemplo de aviso esperado ao atendente**
>
> *"Atenção: foram encontradas duas versões do PROC-042 com valores divergentes. A resposta abaixo usa a versão 2.0 (nov/2023), que é a mais recente. A versão 1.0 (mar/2023) ainda está presente na base — recomenda-se solicitar o arquivamento formal ao time Comercial."*

### 3.3 Pré-condição de governança (bloqueante)

Antes do go-live em produção, o time de produto deve obter confirmação formal da Diretoria Comercial sobre qual versão do PROC-042 é a vigente. Essa decisão deve ser registrada como metadado no documento vencedor (campo `"substitui": "PROC-042-v1"`) e a versão perdedora deve ser movida para um repositório de arquivos históricos, fora do escopo de indexação.

Enquanto essa confirmação não ocorrer, o pipeline deve operar com a v2 como preferencial por ser a mais recente, mas mantendo o aviso de conflito em todas as respostas relacionadas a frete especial.

---

## 4. Comportamento quando não há resposta na base

O assistente não deve inventar respostas nem extrapolar a partir de conteúdo parcialmente relacionado. Quando a base de conhecimento não contiver informação suficiente para responder à pergunta do atendente, o comportamento esperado é determinado pelo tipo de ausência.

| ID | Requisito | Prioridade |
|---|---|---|
| REQ-04.1 | Quando nenhum chunk relevante for recuperado (score de similaridade abaixo de **0,60**), o assistente deve responder: *"Não encontrei documentação sobre este tópico na base de conhecimento. Consulte o responsável pela área ou abra chamado interno."* Nenhuma resposta especulativa deve ser gerada. O limiar de 0,60 é o valor mínimo aceito; ajustes para cima podem ser feitos após avaliação em homologação, mas nunca abaixo desse valor. | Obrigatório |
| REQ-04.2 | Quando a pergunta tocar um tópico com gap documentado (seguro de carga, carga danificada, frete padrão), o assistente deve identificar o gap e orientar o canal correto: Comercial para seguro, Jurídico/sinistros@novatech.com.br para carga danificada. | Obrigatório |
| REQ-04.3 | O assistente nunca deve usar o FAQ como única fonte sem avisar o atendente do status informal do documento. Se o FAQ for a única fonte disponível, o aviso deve ser proeminente e a resposta deve ser marcada como "não confirmada oficialmente". | Obrigatório |
| REQ-04.4 | Quando a base contiver informação parcial (ex: PROC-042 cobre frete especial mas não frete padrão), o assistente deve deixar claro o escopo da informação disponível: *"Tenho informações sobre frete especial (acima de 500 kg). Para frete padrão, não há procedimento documentado na base atual."* | Obrigatório |
| REQ-04.5 | Ausências de resposta devem ser registradas em log com a query original (anonimizada se necessário), para alimentar o processo de identificação de gaps documentais e priorização de novos documentos a criar. | Recomendado |

---

## 5. Requisitos de atualização

O pipeline deve garantir que documentos novos ou revisados estejam disponíveis para o assistente dentro dos prazos definidos abaixo, conforme a criticidade da alteração. Esses prazos são contados a partir do momento em que o documento é publicado no repositório oficial (SharePoint NovaTech).

### 5.1 Prazos por criticidade

| Criticidade | Exemplos | Prazo máximo | Processo |
|---|---|---|---|
| **Crítica** | Alteração de SLA contratual, mudança em multiplicadores de frete, revogação de política | 4 horas úteis | Ingestão manual com prioridade + notificação ao time de atendimento |
| **Alta** | Revisão de procedimento normativo (ex: nova versão de PROC), nova política operacional | 24 horas úteis | Ingestão automática via trigger no SharePoint |
| **Média** | Atualização de FAQ validado, adição de exemplos ou esclarecimentos | 48 horas úteis | Ingestão no próximo ciclo de reindexação programada |
| **Baixa** | Correções ortográficas, ajustes de formatação sem impacto no conteúdo | 7 dias corridos | Próximo ciclo de reindexação semanal |

### 5.2 Requisitos do processo de atualização

| ID | Requisito | Prioridade |
|---|---|---|
| REQ-05.1 | O pipeline deve suportar ingestão incremental — apenas os chunks afetados pelo documento alterado devem ser re-embeddados e substituídos no índice vetorial. Reindexação total só deve ocorrer em mudanças estruturais do pipeline. | Obrigatório |
| REQ-05.2 | Quando um documento substitui outro (ex: PROC-042-v2 substitui v1), o documento substituído deve ser automaticamente removido do índice no mesmo processo de ingestão do novo documento, mediante metadado `"substitui"` no cabeçalho do novo documento. | Obrigatório |
| REQ-05.3 | Toda ingestão deve ser registrada em log com: nome do documento, versão, timestamp de publicação no SharePoint, timestamp de disponibilidade no índice, e identificação do operador (automático ou manual). | Obrigatório |
| REQ-05.4 | O time de atendimento deve ser notificado (via canal interno) quando um documento de confiabilidade ALTA for atualizado ou removido do índice. | Recomendado |
| REQ-05.5 | O sistema deve expor um endpoint ou dashboard interno que permita ao time de produto consultar quais documentos estão atualmente indexados, suas versões e datas de ingestão. | Recomendado |

---

## 6. Requisitos de rastreabilidade

Todo output do assistente deve ser rastreável até sua fonte documental. Esse requisito existe por três razões:

- **(a) Confiança do atendente** — ele precisa poder verificar a informação antes de repassá-la ao cliente.
- **(b) Auditoria de compliance** — em disputas contratuais, é necessário demonstrar que a resposta foi baseada em documento vigente.
- **(c) Melhoria contínua** — saber quais fontes são mais consultadas orienta a priorização da curadoria documental.

### 6.1 Citação de fonte

| ID | Requisito | Prioridade |
|---|---|---|
| REQ-06.1 | Toda resposta que contenha informação factual (prazos, valores, regras, procedimentos) deve citar obrigatoriamente a fonte documental. Formato mínimo: `[NOME-DO-DOCUMENTO, Seção X.Y]`. Exemplo: `[POL-001, Seção 3.1]`. | Obrigatório |
| REQ-06.2 | Quando a resposta combinar informações de mais de uma fonte, cada trecho deve citar sua fonte individualmente. Não é permitido uma única citação genérica para uma resposta composta. | Obrigatório |
| REQ-06.3 | Respostas baseadas no FAQ devem citar o item específico do FAQ (ex: `[FAQ-Atendimento, Item 22]`) e incluir o aviso de confiabilidade média definido na seção 2.2. | Obrigatório |
| REQ-06.4 | Respostas que não encontraram fonte na base (seção 4) não devem conter citação fabricada. O assistente deve declarar explicitamente a ausência de fonte. | Obrigatório |

### 6.2 Exibição do trecho relevante

Além da citação, o assistente deve oferecer ao atendente acesso ao trecho original do documento que embasou a resposta, como verificação rápida sem necessidade de abrir o documento completo.

| ID | Requisito | Prioridade |
|---|---|---|
| REQ-06.5 | A interface deve exibir, junto à resposta, o trecho do documento original recuperado (chunk), com indicação de: documento, versão, seção e texto do trecho. O trecho pode ser colapsado por padrão (expandir ao clicar) para não poluir o fluxo de atendimento. | Obrigatório |
| REQ-06.6 | O trecho exibido deve ser o chunk real recuperado pelo pipeline — não uma paráfrase ou geração do modelo. O modelo gera a resposta; o trecho é a evidência. | Obrigatório |
| REQ-06.7 | O score de similaridade do chunk em relação à query deve ser exibido ao atendente em modo de depuração (opcional, ativável por configuração). Não precisa ser exibido por padrão. | Opcional |
| REQ-06.8 | Quando houver múltiplos chunks relevantes com scores entre **0,60 e 0,75** (zona de incerteza), o assistente deve indicar ao atendente que a resposta pode não ser exaustiva e sugerir verificação direta no documento. | Recomendado |

### 6.3 Logs de auditoria

| ID | Requisito | Prioridade |
|---|---|---|
| REQ-06.9 | Cada interação deve gerar um log imutável contendo: timestamp, identificador do atendente, query enviada, documentos recuperados (com scores), resposta gerada e flag de conflito (se aplicável). Retenção mínima: 90 dias. | Obrigatório |
| REQ-06.10 | Logs devem ser armazenados em sistema de acesso restrito, com controle de acesso por papel (RBAC). Atendentes não têm acesso aos logs. Gestores de produto e compliance têm acesso de leitura. | Obrigatório |
| REQ-06.11 | O sistema deve produzir relatório semanal automático com: top 20 queries mais frequentes, taxa de ausência de resposta por categoria, documentos mais recuperados, e frequência de avisos de conflito disparados. | Recomendado |

---

## 7. Gaps documentais a resolver antes do go-live

Os itens abaixo são pré-condições para que o assistente opere sem gerar lacunas críticas de atendimento.

| Gap | Impacto | Ação necessária | Responsável |
|---|---|---|---|
| PROC-042 duplicado | Assistente emite aviso de conflito em 100% das queries sobre frete especial | Decisão formal sobre versão vigente + arquivamento da v1 | Diretoria Comercial |
| Política de carga danificada ausente | 22% de reaberturas em tópico sem cobertura | Criar POL ou PROC formal validado por Compliance | Operações + Compliance |
| Seguro de carga sem documento formal | Atendente fica sem resposta ou usa FAQ não validado | Criar documento formal com percentuais e condições | Comercial |
| Frete padrão sem cobertura | Dúvidas sobre frete < 500 kg sem resposta na base | Incluir tabela de fretes padrão como documento indexável | TI + Operações |

---

## 8. Critérios de aceite do pipeline

O pipeline estará apto para go-live quando todos os itens obrigatórios abaixo forem validados pelo time de produto em ambiente de homologação, usando os documentos da base NovaTech como corpus de teste.

- [ ] Todas as queries de teste sobre frete especial retornam resposta baseada na PROC-042-v2, com aviso de conflito enquanto a v1 não for arquivada.
- [ ] Queries sobre tópicos sem documentação (seguro, carga danificada) retornam mensagem de ausência de fonte, sem resposta especulativa.
- [ ] Toda resposta factual cita pelo menos uma fonte no formato `[DOCUMENTO, Seção X.Y]`.
- [ ] O trecho relevante do documento original está disponível junto à resposta.
- [ ] Respostas baseadas no FAQ exibem aviso de confiabilidade média.
- [ ] Todos os logs de interação são gerados corretamente com os campos definidos no REQ-06.9.
- [ ] Ingestão de documento crítico é refletida no índice em até 4 horas úteis (testado em homologação).
- [ ] Reindexação incremental funciona sem necessidade de reprocessamento total do corpus.

> ✅ **Indicador de sucesso pós-implantação (30 dias)**
>
> Redução da média de fontes consultadas por chamado de **4,1 para no máximo 2,0**. Redução da taxa de reabertura em 48h de **22% para abaixo de 10%**. Esses indicadores serão medidos via sistema de chamados (Azure DevOps) e revisados 30 dias após o go-live.

---

*Documento confidencial — uso interno. NovaTech Logística.*
