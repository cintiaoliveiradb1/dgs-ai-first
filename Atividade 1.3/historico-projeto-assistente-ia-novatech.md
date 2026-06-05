# Histórico de Interação — Projeto Assistente IA NovaTech
## Product Specialist · Pipeline RAG

| Campo | Valor |
|-------|-------|
| Projeto | Assistente de Atendimento IA — NovaTech Logística |
| Exportado em | Junho/2025 |
| Versão final da spec | 1.3 |
| Conteúdo | Registro completo das interações de discovery, especificação e revisão |

---

## Interação 1 — Contextualização do Problema

**Usuário:**

> Atue como um product specialist no processo de criação um assistente de IA para empresa de logística. Assistente deve auxiliar atendentes apresentando respostas para questionamentos que hoje só são respondidos após consulta de documentações distribuídas em alguns documentos, o que dificulta a consulta.
> Durante o discovery, foram identificados os dados abaixo:
> - Os atendentes abrem em média 4,1 fontes diferentes por chamado. Nos chamados de cálculo de frete especial, esse número sobe para 6,2 — reflexo direto da coexistência das duas versões do PROC-042 sem hierarquia declarada.
> - 22% dos chamados são reabertos em até 48 horas — indicativo de respostas inconsistentes, majoritariamente em tópicos sem documentação oficial (rastreamento, seguro, sinistro).
> - As dúvidas mais comuns são sobre prazos de entrega (35%), regras de frete (25%), política de devolução (20%), SLA e tiers (10%) e outros (10%).
> Ainda não realize nenhuma ação, apenas entenda o contexto que estou repassando.

**Assistente:**

Entendido. Contexto absorvido:

**Problema central:** Fragmentação do conhecimento operacional, causando ineficiência nos atendimentos e inconsistência nas respostas.

**Sintomas identificados:**
- Alta consulta de fontes por chamado (4,1 em média, chegando a 6,2 em frete especial), com ambiguidade documental não resolvida no PROC-042
- 22% de reabertura em 48h, concentrada em áreas sem documentação oficial
- Volume de dúvidas concentrado em poucos tópicos, o que indica oportunidade clara de cobertura do assistente

**O que ainda não foi definido / pode precisar de aprofundamento:**
- Quais documentos existem, quais são os oficiais e como lidar com conflitos (ex: PROC-042)
- Escopo de atuação do assistente: apenas consulta, ou também triagem / encaminhamento?
- Perfil dos atendentes e nível de autonomia esperado do assistente
- Critérios de confiabilidade da resposta gerada

Estou pronto. Pode seguir com o que precisar — próximos passos, perguntas de discovery, definição de escopo, arquitetura ou qualquer outro direcionamento.

---

## Interação 2 — Definição da Arquitetura e Solicitação da Spec

**Usuário:**

> Vamos utilizar uma arquitetura RAG, onde em vez do assistente responder apenas com base no que foi treinado, ele primeiro consulta a base de conhecimento e depois usa essas informações para gerar a resposta. Considere a documentação anexa.
> Crie uma especificação de requisitos que o pipeline de RAG deve atender, garanta que especificação cubra os pontos abaixo:
> - Quais fontes de dados devem ser indexadas
> - Como o assistente deve lidar com documentos contraditórios
> - Qual o comportamento esperado quando a pergunta do atendente não tem resposta na base
> - Requisitos de atualização: quando novos documentos são publicados, em quanto tempo devem estar disponíveis no assistente?
> - Requisitos de rastreabilidade: toda resposta deve citar a fonte? Deve mostrar o trecho relevante?

**Assistente:**

Especificação gerada com base na documentação anexada (Anexo A — NovaTech). Documento entregue: `spec-rag-novatech.md` (v1.0).

Resumo do que foi coberto:

- **Fontes de dados (Seção 2):** Cinco documentos mapeados em duas camadas: fontes primárias (POL-001, PROC-042-v1, PROC-042-v2, SLA-2024) e FAQ como fonte secundária com sinalização de informalidade obrigatória. Quatro gaps documentais catalogados com roadmap de expansão.
- **Documentos contraditórios (Seção 3):** Estratégia resolve o caso concreto do PROC-042 por metadados de status e data como critério de desempate, com sinalização explícita do conflito ao atendente.
- **Perguntas sem resposta (Seção 4):** Três tipos de gap com comportamentos distintos — total (declara e escala), parcial (responde com aviso de fonte informal) e contextual (solicita dado faltante). R-GAP-002 proíbe inferências fora dos trechos recuperados.
- **Atualização (Seção 5):** SLA de 4h úteis para documentos normativos e contratuais, 24h para FAQ. R-ATU-004 obriga desativação da versão anterior no mesmo pipeline da ingestão da nova versão.
- **Rastreabilidade (Seção 6):** Citação de fonte obrigatória com nome, versão e seção; exibição do trecho original sem paráfrase; log auditável por 90 dias com scores de similaridade.

---

## Interação 3 — Análise de Gaps e Inconsistências da Spec v1.0

**Usuário:**

> Analise a spec anexa e identifique gaps ou inconsistências.

**Assistente:**

Análise crítica realizada. Foram identificados 14 itens distribuídos em três categorias:

### 🔴 Inconsistências (3)

| ID | Descrição |
|----|-----------|
| INC-01 | R-CONT-002 e R-CONT-003 não cobrem o caso PROC-042 v1 vs v2 — a regra real é por data de corte contextual, não por status ou data de emissão. A seção 3.3 era nota informal sem ID de requisito nem critério testável. |
| INC-02 | F-003 atribui `status=transição` ao PROC-042-v1, mas R-CONT-001 não define quando um documento entra ou sai desse status, nem qual pipeline realiza a transição automática. |
| INC-03 | R-RASTR-005 exige exibição do "chunk original sem paráfrase", o que conflita com o propósito do assistente ao expor linguagem técnica densa diretamente ao atendente, sem limite de tamanho. |

### 🟡 Gaps (8)

| ID | Descrição |
|----|-----------|
| GAP-01 | Estratégia de chunking não especificada — tamanho, overlap e tratamento de tabelas numéricas ausentes. |
| GAP-02 | Threshold de similaridade não definido — comportamento de R-GAP-001 fica ambíguo sem score mínimo declarado. |
| GAP-03 | Ausência de requisito de controle de acesso — spec não menciona segmentação de usuários. |
| GAP-04 | Sem mecanismo de feedback do atendente sobre qualidade das respostas. |
| GAP-05 | Comportamento em modo degradado não especificado — indisponibilidade ou janela de atualização sem tratamento. |
| GAP-06 | Ausência de requisito sobre seleção e versionamento do modelo LLM. |
| GAP-07 | Gaps documentais da seção 2.3 sem dono nem prazo para resolução. |
| GAP-08 | Logs sem requisito de segurança, controle de acesso ou conformidade LGPD. |

### 🔵 Ambiguidades (3)

| ID | Descrição |
|----|-----------|
| AMB-01 | "Peso de recuperação mais alto" para fontes primárias não é quantificado nem mecanizado. |
| AMB-02 | Janela de inconsistência entre ingestão primária (4h) e FAQ (24h) não tratada. |
| AMB-03 | Spec não distingue "fora do escopo" de "gap documental" — ambos geram a mesma mensagem de "não tenho resposta". |

---

## Interação 4 — Resolução das Inconsistências (v1.1)

**Usuário:**

> Trate as inconsistências conforme abaixo:
> - INC-01: Podemos incluir um pré-requisito nos documentos a serem indexados, onde seja obrigatório informar ao menos uma data. Pode transformar a seção 3.3 em requisito formal.
> - INC-02: A atualização dos status será feita pelo próprio agente. Verificar se existe necessidade de determinar pipeline específico para fazer essa atualização.
> - INC-03: Utilize linguagem clara que possa ser compreendida por usuários que não tenham conhecimento técnico, sem utilizar linguagem técnica.

**Assistente:**

Aplicadas as três correções na spec. Documento entregue: `spec-rag-novatech-v1.1.md`.

| Inconsistência | Resolução aplicada |
|---------------|-------------------|
| INC-01 | Pré-requisito de data obrigatória adicionado à seção 2.1. Seção 3.3 convertida nos requisitos formais **R-CONT-006** (lógica de seleção por data de corte com testes para ambos os lados) e **R-CONT-007** (solicitação da data ao atendente quando ausente do contexto). |
| INC-02 | R-CONT-001 desdobrado em **R-CONT-001b**: atualização de status atribuída ao próprio agente de ingestão, sem pipeline separado. Transição de `transição` para `obsoleto` automatizada pela data declarada em `vigente_até`. |
| INC-03 | R-RASTR-005 reformulado em linguagem acessível: elimina o termo "chunk", introduz resumo em linguagem simples para trechos com mais de 150 palavras ou linguagem técnica densa, e define critério de aceitação objetivo. |

---

## Interação 5 — Resolução dos Gaps (v1.2)

**Usuário:**

> - GAP-01: Defina como uma decisão técnica pendente.
> - GAP-02: Me sugira um score mínimo aceitável.
> - GAP-03: Acesso ao agente será garantido através do Azure, dessa forma apenas usuário com acesso ao Azure poderá efetuar consulta.
> - GAP-04: Incluir requisito que colete feedback do cliente, onde ele poderá informar se informação respondida foi útil, se está errada ou incompleta.
> - GAP-05: Apresentar mensagem que sistema está indisponível ou em atualização e indique onde cliente pode consultar documentação.
> - GAP-06: Defina como uma decisão técnica pendente.
> - GAP-07: Indique que responsabilidade é das áreas Operações, Compliance, Comercial da NovaTech. Indique ao cliente solicitar às áreas a criação de uma documentação que esclareça o ponto.
> - GAP-08: Somente a equipe de segurança da NovaTech terá acesso aos logs, dados sensíveis devem ser tratados de acordo com LGPD.

**Assistente:**

Antes da aplicação, foi apresentada justificativa para o score sugerido no GAP-02:

> O contexto da NovaTech tem linguagem operacional padronizada em português. Para esse perfil, o intervalo 0,75–0,80 é o mais comum em bases corporativas. Abaixo de 0,70 há risco de falsos positivos; acima de 0,85 a restrição é excessiva para variações naturais de linguagem (ex: "prazo" vs "tempo de entrega"). **Score recomendado: 0,75**, com revisão após os primeiros 30 dias de operação.

Documento entregue: `spec-rag-novatech-v1.2.md`.

| Gap | Resolução aplicada |
|-----|--------------------|
| GAP-01 | Registrado como **DTP-001** na nova Seção 8 — Decisões Técnicas Pendentes, com responsável e destaque para risco em tabelas numéricas. |
| GAP-02 | Score 0,75 formalizado em **R-GAP-002b**, com obrigação de relatório de homologação nos primeiros 100 chamados reais. |
| GAP-03 | Autenticação via Azure AD formalizada em **R-NF-005**: bloqueio sem credencial válida, revogação no Azure reflete automaticamente no assistente. |
| GAP-04 | **R-GAP-006**: coleta de feedback com três opções (útil / incorreta / incompleta), campo descritivo opcional para opções negativas, painel de administração com histórico filtrável. |
| GAP-05 | **R-NF-006**: dois estados distintos — falha (indisponibilidade temporária) e atualização planejada — ambos com indicação do caminho de documentação alternativa. |
| GAP-06 | Registrado como **DTP-002** na Seção 8, com alerta de que troca silenciosa de modelo pode alterar comportamento das respostas. |
| GAP-07 | Seção 2.3 atualizada com nomeação das áreas responsáveis (Operações, Compliance, Comercial) e orientação para o time de produto formalizar a solicitação de documentação. |
| GAP-08 | **R-RASTR-009**: acesso restrito à equipe de segurança. **R-RASTR-010**: conformidade LGPD com finalidade declarada, prazo de retenção homologado pelo DPO e procedimento de descarte documentado. |

---

## Interação 6 — Sugestões para Resolução das Ambiguidades

**Usuário:**

> Me dê sugestões para resolver as ambiguidades encontradas.

**Assistente:**

Foram apresentadas três opções para cada ambiguidade, com recomendação fundamentada:

### AMB-01 — "Peso de recuperação mais alto" sem mecanismo definido

| Opção | Descrição | Indicação |
|-------|-----------|-----------|
| A | Boost de score fixo por tipo de fonte (multiplicador aplicado antes do threshold) | Simples, mas pode fazer chunk primário pouco relevante superar secundário muito relevante |
| **B** ✅ | **Re-ranking pós-recuperação: relevância semântica prevalece; tipo de fonte é critério de desempate em scores iguais** | **Equilibrado e auditável — recomendado** |
| C | Filtro obrigatório: só consulta secundária se primária não atingir threshold | Máxima prioridade ao oficial, mas pode suprimir complementos úteis do FAQ |

### AMB-02 — Janela de inconsistência FAQ × primária (4h vs 24h)

| Opção | Descrição | Indicação |
|-------|-----------|-----------|
| A | Fonte primária sempre suprime FAQ em conflito, sem aviso ao atendente | Simples e seguro, mas pouco transparente |
| **B** ✅ | **Primária suprime FAQ + sinalização ao atendente + marcação do item para revisão** | **Transparente e sem complexidade adicional — recomendado** |
| C | Ingestão de primária dispara verificação automática no FAQ e suprime itens conflitantes | Robusto, mas exige lógica adicional de detecção — indicado como evolução futura |

### AMB-03 — "Fora do escopo" vs "gap documental" com a mesma mensagem

| Opção | Descrição | Indicação |
|-------|-----------|-----------|
| **A** ✅ | **Duas mensagens distintas baseadas na lista de gaps já mapeada na seção 2.3** | **Aproveita mapeamento existente, implementação imediata — recomendado** |
| B | Classificação dinâmica por similaridade com tópicos-semente de cada categoria | Mais flexível para gaps novos, mas mais complexo e com risco de classificação incorreta — evolução futura |

---

## Interação 7 — Incorporação das Ambiguidades (v1.3)

**Usuário:**

> Incorpore as sugestões.

**Assistente:**

Documento entregue: `spec-rag-novatech-v1.3.md`.

| Ambiguidade | Resolução aplicada |
|-------------|-------------------|
| AMB-01 | Descrição da seção 2.1 reescrita para explicar o mecanismo de re-ranking. **F-006** formaliza: relevância semântica prevalece; tipo de fonte desempata em scores iguais. Dois critérios de aceitação cobrem os dois cenários (empate e scores distintos). |
| AMB-02 | **R-ATU-004b**: durante a janela de inconsistência, FAQ conflitante é suprimido, atendente recebe aviso explícito de descarte e item é marcado no painel de administração para revisão prioritária. |
| AMB-03 | **R-NF-004** reescrito com classificação prévia em três categorias (coberta / gap documentado / fora do escopo). Mensagens distintas para gap e fora do escopo. Critério de aceitação exige que os dois cenários nunca exibam a mesma mensagem. |

---

## Rastreabilidade dos Requisitos por Versão

| Versão | Requisitos adicionados / alterados |
|--------|------------------------------------|
| v1.0 | F-001 a F-005, R-CONT-001 a R-CONT-005, R-GAP-001 a R-GAP-005, R-ATU-001 a R-ATU-007, R-RASTR-001 a R-RASTR-008, R-NF-001 a R-NF-004 |
| v1.1 | Pré-requisito de data (seção 2.1), R-CONT-001b, R-CONT-006, R-CONT-007, R-RASTR-005 (revisado) |
| v1.2 | R-GAP-002b, R-GAP-006, R-NF-005, R-NF-006, R-RASTR-009, R-RASTR-010, DTP-001, DTP-002, seção 2.3 (responsáveis) |
| v1.3 | F-006, R-ATU-004b, R-NF-004 (revisado) |

---

## Artefatos Gerados

| Arquivo | Versão | Descrição |
|---------|--------|-----------|
| `spec-rag-novatech.md` | v1.0 | Especificação inicial |
| `spec-rag-novatech-v1.1.md` | v1.1 | Correção das inconsistências INC-01, INC-02, INC-03 |
| `spec-rag-novatech-v1.2.md` | v1.2 | Resolução dos gaps GAP-01 a GAP-08 |
| `spec-rag-novatech-v1.3.md` | v1.3 | Resolução das ambiguidades AMB-01, AMB-02, AMB-03 |
| `historico-projeto-assistente-ia-novatech.md` | — | Este documento |

---

*Exportado em Junho/2025 — Confidencial — NovaTech Logística*
