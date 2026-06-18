# requirements.md
## Query Endpoint — Assistente de Atendimento IA
### NovaTech Logística

| Campo | Valor |
|-------|-------|
| Versão | 1.0 |
| Data | Junho/2025 |
| Autor | Product Specialist — Specification Driven Development |
| Status | Rascunho para revisão |
| Fontes | Bounded Contexts v1.0 · Linguagem Ubíqua v1.0 · spec-rag-novatech-v1.3 · Interação Jornada Assistente IA |

---

## Objetivo

O Query Endpoint é o ponto de entrada por meio do qual atendentes da NovaTech enviam perguntas em linguagem natural pelo Microsoft Teams e recebem respostas estruturadas, rastreáveis e consistentes com as regras do domínio logístico.

O endpoint não é um canal de autoatendimento ao cliente final. É uma ferramenta de suporte ao atendente humano, que permanece responsável pela decisão e pela comunicação com o cliente. O assistente reduz o tempo de busca em documentação dispersa (atualmente 4,1 fontes por chamado, chegando a 6,2 em frete especial) e diminui a taxa de reabertura de chamados em 48h (atualmente 22%), fornecendo respostas ancoradas em documentação oficial com fonte e vigência explícitas.

---

## Escopo

### Dentro do escopo

O assistente pode responder perguntas sobre os seguintes temas, nos limites da documentação disponível na base de conhecimento:

**Cálculo e Precificação de Frete Especial (BC-A1)**
- Fórmula de frete especial: Valor base × Multiplicador regional × Fator de peso
- Multiplicadores regionais por destino (Sul, Sudeste, Centro-Oeste, Nordeste, Norte), com distinção entre PROC-042 v1 e v2 conforme data de abertura do chamado
- Fatores de peso por faixa (500–1.000 kg, 1.001–3.000 kg, acima de 3.000 kg), com a mesma distinção de versão
- Desconto por volume mensal (≥ 8 fretes/mês e ≥ 15 fretes/mês — PROC-042 v2 vigente)
- Necessidade de aprovação do gerente regional para cargas acima de 5.000 kg

**Prazos e Roteirização de Entrega (BC-A2)**
- Acréscimo de prazo para frete especial: +3 dias úteis (v2 vigente) ou +2 dias úteis (v1, para chamados até 30/11/2023)
- Definição operacional de dias úteis (segunda a sexta, excluídos feriados nacionais)
- Referência informal de prazos por região (fonte FAQ — sinalizada como não validada)
- Regra de pausa do relógio de SLA fora do horário comercial (08h–18h)

**Devolução de Mercadorias (BC-A3)**
- Prazo para solicitação de devolução: 7 dias úteis após a data de recebimento confirmada no sistema de tracking
- Categorias de carga inelegíveis para devolução padrão (cargas perigosas classes 1–6, cargas com cadeia de frio rompida, cargas com lacre de segurança violado sem documentação)
- Procedimento de abertura de chamado de devolução pelo Portal do Cliente
- Documentos exigidos na abertura: CT-e, mínimo 3 fotos, motivo
- Prazo de triagem: 4 horas úteis após abertura
- Prazo de coleta reversa: até 2 dias úteis após aprovação
- Prazo de reembolso: até 5 dias úteis após recebimento no centro de distribuição
- Devolução parcial: elegibilidade e cálculo proporcional ao CT-e
- Responsabilidade de custo: defeito NovaTech (sem custo ao cliente) vs. desistência do cliente (frete reverso por conta do cliente)
- Encaminhamento à Gestão de Riscos (ramal 4500) para categorias inelegíveis

**Cargas Especiais e Restritas (BC-A4)**
- Classificação de cargas perigosas por classes ANTT 1–6 (Resolução 5.947/2021)
- Restrições de devolução e frete especial para cargas perigosas
- Regra de cadeia de frio: rompimento após > 30 minutos fora da faixa registrado por sensor IoT
- Regra de lacre de segurança violado: inelegibilidade para devolução padrão, com exceção para documentação no ato

**Contratos e SLA por Tier de Cliente (BC-A6)**
- Critérios de elegibilidade dos tiers Gold, Silver e Standard
- Inexistência do tier Platinum
- SLA de primeira resposta e SLA de resolução por tier e por tipo de chamado (geral vs. incidente crítico)
- Disponibilidade do portal de tracking por tier
- Definição precisa de incidente crítico (4 critérios formais)
- Regime de penalidades por violação de SLA (1ª, 2ª, 3ª ou mais no mesmo mês)
- Regra de pausa e continuidade do relógio de SLA

**Conhecimento Informal do Atendimento — com sinalização (BC-A5)**
- Orientações do FAQ que complementam ou preenchem lacunas dos documentos formais, sempre apresentadas com aviso explícito de fonte informal não validada

---

### Fora do escopo

O assistente **não deve responder** sobre os seguintes temas, por ausência de documentação formal na base ou por competência pertencer a outra área:

| Tema | Motivo | Encaminhamento correto |
|------|--------|------------------------|
| Frete padrão (cargas abaixo de 500 kg) | Gap total — nenhum documento cobre esta faixa | [DECISÃO NECESSÁRIA] Área responsável a definir |
| Seguro de carga (percentuais e condições) | Gap total — existe apenas no FAQ-Item 22, sem documento formal | Comercial ou consulta ao contrato do cliente |
| Carga danificada em trânsito (avaria/sinistro) | Gap total — prática informal no FAQ-Item 38, sem POL ou PROC formal | Jurídico via sinistros@novatech.com.br |
| Processo interno da Gestão de Riscos após receber caso de carga perigosa | Gap total — ramal 4500 documentado, mas sem PROC formal | Gestão de Riscos, ramal 4500 |
| Frete de cargas perigosas acima de 500 kg (PROC-043) | Gap parcial — PROC-043 citada, mas em revisão pelo Compliance; não disponível na base | Compliance |
| Interceptação de carga em trânsito | Regido pela PROC-088 — não disponível na base | [DECISÃO NECESSÁRIA] Área responsável a definir |
| Negociação de contratos e descontos fora das regras documentadas | Competência do Comercial | Comercial |
| Frete padrão e seus prazos | Gap documental | [DECISÃO NECESSÁRIA] |
| Tabela de rotas cadastradas com prazos por origem/destino | Gap documental — não está no Anexo A | [DECISÃO NECESSÁRIA] |
| Tabela mensal de fretes-base | Arquivo externo (`\\novatech-fs\comercial\tabelas\frete-base-AAAAMM.xlsx`) — não indexado | Atendente deve consultar o arquivo diretamente |
| Definição inicial de tier e negociação contratual | Competência do Comercial | Comercial |
| Execução técnica de medição no Azure DevOps | Infraestrutura de TI — fora do domínio do assistente | TI |
| Qualquer tema não relacionado a operações logísticas da NovaTech | Fora do escopo do domínio | — |

---

## Usuários e Cenários

### Perfil de usuário

Atendentes do time de operações da NovaTech com conta ativa no Azure Active Directory e perfil autorizado. O assistente não é acessível a clientes finais.

### Cenários cobertos e comportamento esperado

| # | Pergunta do atendente | Comportamento esperado |
|---|-----------------------|------------------------|
| U-01 | "Qual o multiplicador regional para destino Norte?" | Responde com o valor da v2 (1,8) para chamados a partir de 01/12/2023. Se a data do chamado não for informada, solicita antes de responder. Cita PROC-042 v2, seção 2.1. |
| U-02 | "O cliente quer devolver a carga — qual o prazo?" | Responde: 7 dias úteis após a data de recebimento confirmada no sistema de tracking. Cita POL-001 v3.1, seção 3.1. |
| U-03 | "O cliente diz que é Platinum, qual o SLA dele?" | Informa que o tier Platinum não existe na NovaTech. Os tiers vigentes são Gold, Silver e Standard. Cita SLA-2024, seção 1. |
| U-04 | "Quero calcular o frete especial de 800 kg para o Nordeste" | Se a data do chamado não foi fornecida, solicita. Com a data, usa os multiplicadores corretos (v2: multiplicador 1,5 × fator 1,0) e exibe a fórmula com a fonte. Alerta se o valor base não estiver disponível no contexto. |
| U-05 | "A carga é classe 3 (líquido inflamável) — pode devolver?" | Responde que carga perigosa classe 3 (líquidos inflamáveis, ANTT Resolução 5.947/2021) é inelegível para devolução padrão. Orienta encaminhar à Gestão de Riscos, ramal 4500. Cita POL-001 v3.1, seção 3.2. |
| U-06 | "Qual o SLA de resposta para cliente Gold em incidente crítico?" | Responde com o valor contratual do SLA-2024 para incidente crítico Gold, com relógio contínuo (sem pausa). Cita SLA-2024, seção 2 e seção 5. |
| U-07 | "O cliente quer saber o valor do seguro da carga" | Informa que não há política formal de seguro de carga na documentação disponível. Orienta contatar o Comercial ou verificar o contrato do cliente. Não usa o percentual do FAQ-Item 22 como resposta oficial. |
| U-08 | "A carga chegou danificada — o que eu faço?" | Informa que não há política formal para carga danificada em trânsito. Exibe a orientação informal do FAQ-Item 38 (registro em até 48h com fotos, encaminhamento a sinistros@novatech.com.br) com aviso explícito: "Fonte: FAQ informal — não validado por Compliance." Orienta contatar o Jurídico. |
| U-09 | "Qual o frete para uma carga de 200 kg?" | Informa que frete padrão (abaixo de 500 kg) não está coberto pela documentação disponível na base. Indica que se trata de um gap documental e orienta [DECISÃO NECESSÁRIA] sobre canal de escalação. |
| U-10 | "Qual a diferença entre SLA de resposta e SLA de resolução?" | Explica os dois conceitos com as definições da Linguagem Ubíqua. Cita SLA-2024 e FAQ-Item 41 (alinhados). |

---

## Requisitos Funcionais

### RF-001 — Consulta com documentação disponível

```
DADO que o atendente envia uma pergunta sobre um tema coberto pela base de conhecimento
  E a pergunta contém todas as informações contextuais necessárias (ex.: data do chamado, tier, peso, região)
QUANDO o pipeline recupera ao menos um trecho com score de similaridade ≥ 0,75
ENTÃO o assistente retorna uma resposta estruturada contendo:
  - A informação solicitada em linguagem natural
  - O trecho do documento que embasou a resposta, em bloco destacado
  - A citação da fonte: nome do documento, versão, seção e data da última atualização
  - Indicação visual de "fonte oficial" quando a fonte for primária (normativo, procedimento ou contratual)
```

### RF-002 — Solicitação de dado contextual ausente (gap contextual)

```
DADO que o atendente envia uma pergunta sobre tema coberto pela base
  MAS a pergunta não contém informação contextual obrigatória
    (ex.: pergunta sobre frete especial sem data do chamado, peso ou região)
QUANDO o pipeline identifica que a resposta depende de um dado não fornecido
ENTÃO o assistente solicita objetivamente o dado faltante antes de responder
  - Ex.: "Por favor, informe a data de abertura do chamado para que eu possa aplicar a versão correta do procedimento."
  - Ex.: "Por favor, informe o peso da carga e a região de destino para calcular o multiplicador correto."
  E não tenta responder com suposições ou valores padrão
```

### RF-003 — Seleção de versão do PROC-042 por data do chamado

```
DADO que o atendente faz uma pergunta sobre cálculo de frete especial
  E informa a data de abertura do chamado
QUANDO o pipeline seleciona qual versão do PROC-042 aplicar
ENTÃO:
  - Chamados abertos até 30/11/2023 → o assistente usa PROC-042 v1
  - Chamados abertos a partir de 01/12/2023 → o assistente usa PROC-042 v2
  E a resposta indica explicitamente qual versão foi usada e por quê
```

### RF-004 — Sinalização de conflito entre documentos

```
DADO que o pipeline recupera trechos de documentos distintos sobre o mesmo tema
  E os valores ou regras são divergentes entre as versões
QUANDO a resposta é composta
ENTÃO o assistente exibe um aviso de conflito contendo:
  - Os nomes dos documentos divergentes
  - O campo ou valor conflitante
  - A regra de desempate aplicada (ex.: "Esta resposta usa PROC-042 v2, vigente para chamados a partir de 01/12/2023")
  E usa o documento com status=vigente ou, em empate de status, o de data de emissão mais recente
```

### RF-005 — Resposta para gap total

```
DADO que o atendente pergunta sobre tema logístico da NovaTech
  E o tema é identificado como gap total (lista da seção 2.3 da spec-rag-v1.3)
QUANDO nenhum trecho com score ≥ 0,75 é recuperado da base
ENTÃO o assistente responde que o tema faz parte das operações da NovaTech
  MAS ainda não há documentação formal disponível na base
  E orienta o atendente ao canal de escalação adequado para o gap (Comercial, Compliance, Jurídico ou Gestão de Riscos conforme o tema)
  E NÃO usa a mensagem genérica de "fora do escopo"
```

### RF-006 — Resposta para tema fora do escopo

```
DADO que o atendente pergunta sobre tema não pertencente ao domínio logístico da NovaTech
QUANDO o pipeline classifica a pergunta como fora do escopo
ENTÃO o assistente informa que o tema está fora do escopo do assistente,
  que responde exclusivamente sobre operações logísticas da NovaTech
  E exibe mensagem distinta da mensagem de gap total
```

### RF-007 — Resposta com fonte informal (FAQ)

```
DADO que a pergunta é respondida com base em trecho do FAQ-Atendimento
  (fonte secundária, tipo=informal)
QUANDO a resposta é composta
ENTÃO o assistente exibe a informação do FAQ acompanhada do aviso:
  "FAQ-Atendimento (documento informal, não validado por Compliance)."
  E indica ao atendente que a informação não tem validação formal
```

### RF-008 — Supressão do FAQ durante janela de inconsistência

```
DADO que o pipeline recupera trechos conflitantes de uma fonte primária e do FAQ sobre o mesmo tema
  E a fonte primária foi atualizada mais recentemente
QUANDO a resposta é composta durante a janela de até 24h de defasagem do FAQ
ENTÃO o assistente usa somente o trecho da fonte primária
  E exibe o aviso:
    "Esta resposta é baseada na documentação oficial. Uma versão anterior desta orientação existia no FAQ e foi desconsiderada por ser menos recente que o documento oficial."
  E sinaliza internamente o item do FAQ para revisão prioritária
```

### RF-009 — Identificação e tratamento de incidente crítico

```
DADO que o atendente descreve uma situação envolvendo qualquer dos 4 critérios de incidente crítico:
  (1) carga com valor declarado > R$ 100.000 com status desconhecido há > 6h
  (2) carga perigosa com irregularidade de documentação ou rastreamento
  (3) > 5 chamados do mesmo cliente nas últimas 24h sobre o mesmo problema
  (4) risco à segurança de pessoas
QUANDO o assistente identifica o critério
ENTÃO informa ao atendente que a situação se enquadra como incidente crítico
  E indica o SLA de resolução aplicável ao tier do cliente (com relógio contínuo para Gold)
  E orienta sobre escalada obrigatória conforme o tier
  Citando SLA-2024, seção 3
```

### RF-010 — Coleta de feedback do atendente

```
DADO que o assistente retornou uma resposta (completa, de gap ou de fora do escopo)
QUANDO a interação é concluída
ENTÃO o assistente exibe ao atendente as opções de feedback:
  - Útil
  - Incorreta
  - Incompleta
  E ao selecionar "Incorreta" ou "Incompleta", exibe campo de texto opcional para descrição
  E registra o feedback vinculado à interação, ao documento citado e ao timestamp
```

### RF-011 — Comportamento para carga com aprovação de gerente regional obrigatória

```
DADO que o atendente pergunta sobre frete especial de carga acima de 5.000 kg
QUANDO o assistente identifica que o peso ultrapassa o limiar de aprovação
ENTÃO não fornece valor de frete calculado
  E informa que cargas acima de 5.000 kg requerem aprovação prévia do gerente regional antes do cálculo
  Citando PROC-042 v2, seção 4
```

### RF-012 — Modo degradado

```
DADO que o assistente está indisponível (falha ou atualização planejada)
QUANDO o atendente tenta uma consulta
ENTÃO o sistema exibe mensagem clara distinguindo os dois estados:
  - Falha: "O assistente está temporariamente indisponível. Consulte a documentação oficial em [caminho/URL] ou aguarde o restabelecimento do serviço."
  - Atualização planejada: "O assistente está em atualização e voltará em breve. Consulte a documentação oficial em [caminho/URL] durante esse período."
```

---

## Regras de Domínio

As regras abaixo derivam dos documentos normativos e são reproduzidas com os termos da Linguagem Ubíqua. Conflitos e gaps são identificados explicitamente.

### RD-001 — Elegibilidade de frete especial
Frete especial aplica-se exclusivamente a cargas com peso acima de 500 kg.
*Fonte: PROC-042 v1 §1, PROC-042 v2 §1*

### RD-002 — Fórmula de frete especial
`Frete especial = Valor base × Multiplicador regional × Fator de peso`
O valor base é a tarifa da tabela mensal no servidor de rede (`\\novatech-fs\comercial\tabelas\frete-base-AAAAMM.xlsx`), **não indexada na base atual**.
*Fonte: PROC-042 v2 §2*

### RD-003 — Disposições transitórias (versão aplicável do PROC-042)
- Chamados abertos até 30/11/2023 → PROC-042 v1
- Chamados abertos a partir de 01/12/2023 → PROC-042 v2 (vigente)
*Fonte: PROC-042 v2 §5*

### RD-004 — Multiplicadores regionais (v2 vigente)
| Região | Multiplicador |
|--------|--------------|
| Sul | 1,3 |
| Sudeste | 1,1 |
| Centro-Oeste | 1,4 |
| Nordeste | 1,5 |
| Norte | 1,8 |
*Fonte: PROC-042 v2 §2.1 · Conflito com v1 — ver ADR-003*

### RD-005 — Fatores de peso (v2 vigente)
| Faixa | Fator |
|-------|-------|
| 500–1.000 kg | 1,0 |
| 1.001–3.000 kg | 1,15 |
| Acima de 3.000 kg | 1,4 |
*Fonte: PROC-042 v2 §2 · Conflito com v1 — ver ADR-003*

### RD-006 — Prazo adicional de frete especial
- v2 (vigente): +3 dias úteis sobre o prazo padrão da rota
- v1 (transição): +2 dias úteis
*Fonte: PROC-042 v2 §3 · Conflito com v1 — ver ADR-003*

### RD-007 — Desconto por volume mensal (v2 vigente)
- ≥ 8 fretes/mês: desconto de 5% sobre o multiplicador regional
- ≥ 15 fretes/mês: desconto de 10% sobre o multiplicador regional
- O atendente **não tem autonomia** para conceder desconto fora dessas faixas
- Desconto acima dessas faixas requer aditivo contratual emitido pelo Comercial
*Fonte: PROC-042 v2 §4 · Conflito com FAQ-Item 45 — ver ADR-004*

### RD-008 — Dias úteis
Segunda a sexta-feira, excluídos feriados nacionais. Sábados e domingos nunca contam como dias úteis.
*Fonte: POL-001 v3.1 §3.1, SLA-2024 §5*

### RD-009 — Prazo de solicitação de devolução padrão
7 dias úteis contados a partir da data de recebimento confirmada no sistema de tracking.
*Fonte: POL-001 v3.1 §3.1*

### RD-010 — Categorias inelegíveis para devolução padrão
1. Cargas perigosas classes 1–6 (ANTT Resolução 5.947/2021)
2. Cargas refrigeradas com cadeia de frio rompida (> 30 minutos fora da faixa registrado por sensor IoT)
3. Cargas com lacre de segurança violado sem documentação no ato da entrega (assinatura de motorista e recebedor)
Para todas: encaminhar à Gestão de Riscos, ramal 4500.
*Fonte: POL-001 v3.1 §3.2*

### RD-011 — Responsabilidade de custo no frete reverso
- Defeito ou erro da NovaTech → frete reverso sem custo para o cliente
- Desistência do cliente → frete reverso por conta do cliente, calculado com os mesmos multiplicadores do PROC-042 vigente
*Fonte: POL-001 v3.1 §3.5*

### RD-012 — Tiers de cliente
| Tier | Critério |
|------|----------|
| Gold | Contrato anual > R$ 500.000 OU > 200 operações/mês. Revisão semestral. |
| Silver | Contrato entre R$ 100.000 e R$ 500.000 OU entre 50 e 200 operações/mês. Revisão semestral. |
| Standard | Todos os demais clientes. Revisão anual. |
| Platinum | **Não existe.** |
*Fonte: SLA-2024 §1*

### RD-013 — Incidente crítico (4 critérios cumulativos — basta 1)
1. Carga com valor declarado > R$ 100.000 com status desconhecido há > 6h
2. Carga perigosa com irregularidade de documentação ou rastreamento
3. Mais de 5 chamados do mesmo cliente sobre o mesmo problema nas últimas 24h
4. Risco à segurança de pessoas
*Fonte: SLA-2024 §3*

### RD-014 — Regime de penalidades por violação de SLA
| Ocorrência no mês | Penalidade |
|-------------------|-----------|
| 1ª | Registro interno — sem impacto contratual |
| 2ª | Crédito de 5% |
| 3ª ou mais | Crédito de 10% + reunião obrigatória |
Escala obrigatória na 3ª violação: gerente de conta (Gold) ou gerente de operações (Silver/Standard).
*Fonte: SLA-2024 §4*

### RD-015 — Relógio de SLA
- Chamados gerais: pausa fora do horário comercial (08h–18h, dias úteis)
- Incidentes críticos de clientes Gold: relógio contínuo, sem pausa
*Fonte: SLA-2024 §5*

### RD-016 — Canal exclusivo para devolução padrão
O único canal aceito para abertura de chamado de devolução padrão é o Portal do Cliente (portal.novatech.com.br). Solicitações por e-mail ou telefone não iniciam o processo formal.
*Fonte: POL-001 v3.1 §3.3*

---

## Prior Decisions (ADRs)

### ADR-001 — Prioridade entre tipos de documento

**Decisão:** Documentos normativos, de procedimento e contratuais (fontes primárias) prevalecem sobre o FAQ-Atendimento (fonte secundária) em qualquer conflito de conteúdo. Em empate de score de similaridade entre um trecho primário e um secundário, o trecho primário é selecionado. Quando o trecho secundário tem score significativamente superior, a relevância semântica prevalece — o tipo de fonte é critério de desempate, não de exclusão automática.

**Justificativa:** O FAQ representa conhecimento acumulado pelo time de atendimento sem validação formal de Compliance ou Operações. Documentos normativos têm responsável formal declarado e processo de aprovação explícito.

**Referência:** spec-rag-novatech-v1.3, seção 2.1 (F-006)

---

### ADR-002 — Tratamento de gaps documentais

**Decisão:** Perguntas sobre temas com gap total recebem resposta que: (a) reconhece que o tema pertence ao domínio logístico da NovaTech, (b) informa que não há documentação formal disponível na base, (c) orienta o atendente ao canal de escalação adequado. A mensagem de gap total é distinta da mensagem de "fora do escopo". O assistente **não fabrica** informações para preencher gaps, mesmo que haja orientação informal no FAQ sobre o tema.

**Gaps documentais ativos:**
- Frete padrão (abaixo de 500 kg)
- Seguro de carga (percentuais e condições)
- Carga danificada em trânsito (sinistro/avaria)
- Processo formal da Gestão de Riscos para cargas perigosas devolvidas
- Frete expresso para carga perigosa (PROC-043 em revisão pelo Compliance)

**Responsabilidade documental:** Operações (frete padrão e carga danificada), Compliance (cargas perigosas e PROC-043) e Comercial (seguro de carga).

**Referência:** spec-rag-novatech-v1.3, seções 2.3 e 4; Bounded Contexts v1.0, tabela de gaps

---

### ADR-003 — Conflito entre PROC-042 v1 e v2 (C1)

**Conflito identificado:** PROC-042 v1 e v2 coexistem no SharePoint sem hierarquia formal declarada no repositório. Os multiplicadores regionais, fatores de peso e prazo adicional diferem entre as versões.

**Decisão:** A data de abertura do chamado determina a versão aplicável:
- Até 30/11/2023 → v1 (status: transição)
- A partir de 01/12/2023 → v2 (status: vigente)

Quando a data não for fornecida na pergunta, o assistente deve solicitá-la antes de responder sobre frete especial. Toda resposta sobre frete especial deve indicar qual versão foi usada e por quê.

**Regra derivada:** PROC-042 v2 §5 (Disposições Transitórias) — único documento que declara a regra de desambiguação.

**Referência:** Linguagem Ubíqua v1.0 (Conflitos 1, 2 e 3); Bounded Contexts v1.0 (C1); spec-rag-novatech-v1.3 (R-CONT-006, R-CONT-007)

---

### ADR-004 — Conflito entre FAQ-Item 45 e PROC-042 v2 (C2)

**Conflito identificado:** O FAQ-Item 45 informa "desconto acima de 10 fretes/mês", alinhado com a PROC-042 v1. A PROC-042 v2 (vigente) estabelece limiares de 8 e 15 fretes/mês com percentuais definidos (5% e 10%).

**Decisão:** Para chamados vigentes (a partir de 01/12/2023), a PROC-042 v2 prevalece. O assistente responde com os limiares da v2 (≥ 8 e ≥ 15 fretes/mês). O FAQ-Item 45 está desatualizado e deve ser corrigido pela equipe de atendimento — o assistente sinaliza internamente o conflito para revisão do FAQ.

**Referência:** Linguagem Ubíqua v1.0 (Conflito 4); Bounded Contexts v1.0 (C2)

---

### ADR-005 — Limites do escopo: atendente vs. cliente final

**Decisão:** O assistente serve exclusivamente atendentes autenticados via Azure AD. Nenhuma rota de acesso ao endpoint deve ser exposta a clientes finais sem autenticação. O assistente não toma decisões em nome do atendente — fornece informação estruturada; a decisão e a comunicação com o cliente permanecem com o atendente.

**Referência:** spec-rag-novatech-v1.3, R-NF-005; Interação Jornada Assistente IA (premissa central)

---

### ADR-006 — Política de não fabricação de respostas

**Decisão:** O assistente não deve inferir, extrapolar ou gerar informações não presentes em documentos indexados. Respostas com informações não contidas em nenhum documento indexado são consideradas falha grave. O threshold de similaridade de 0,75 opera como barreira técnica; a regra de não fabricação é o princípio de negócio subjacente.

**Referência:** spec-rag-novatech-v1.3, R-GAP-002

---

## Scope Boundaries

| Contexto (BC) | Pode consultar | Não pode consultar |
|---------------|----------------|--------------------|
| BC-A1 — Cálculo e Precificação de Frete | Fórmula, multiplicadores regionais (v1 e v2), fatores de peso (v1 e v2), desconto por volume (v2), prazo adicional, aprovação gerente regional | Valor base (tabela mensal não indexada), frete padrão < 500 kg (gap), frete de cargas perigosas > 500 kg (PROC-043 em revisão), negociação de contratos |
| BC-A2 — Prazos e Roteirização | Prazo adicional de frete especial, definição de dias úteis, pausa de SLA, referência informal de prazo por região (com sinalização) | Tabela de rotas com prazos por origem/destino (gap), prazos de frete padrão (gap) |
| BC-A3 — Devolução de Mercadorias | Prazo de solicitação, categorias inelegíveis, procedimento e documentos exigidos, prazos de triagem/coleta/reembolso, devolução parcial, responsabilidade de custo, encaminhamento Gestão de Riscos | Carga danificada em trânsito/sinistro (gap), interceptação de carga em trânsito (PROC-088 ausente), atuação interna da Gestão de Riscos (gap) |
| BC-A4 — Cargas Especiais e Restritas | Classificação ANTT classes 1–6, restrições de devolução, regra de cadeia de frio, regra de lacre violado | PROC-043 completa (em revisão), processo formal Gestão de Riscos (gap), requisitos ANTT de documentação de transporte (gap) |
| BC-A5 — Conhecimento Informal (FAQ) | Orientações práticas como complemento quando não há fonte primária, sempre com sinalização de fonte informal | Qualquer informação do FAQ que contradiga fonte primária vigente (suprimida com aviso) |
| BC-A6 — Contratos e SLA por Tier | Critérios de tier (Gold/Silver/Standard), inexistência do Platinum, tabela de SLA por tier e tipo, disponibilidade do portal, incidente crítico, penalidades, relógio de SLA | Negociação de tier e contrato (Comercial), execução técnica Azure DevOps (TI), SLA diferenciado fora dos três tiers |

---

## Casos Ambíguos

Os itens abaixo exigem esclarecimento da equipe de produto, Operações ou Comercial antes de requisitos definitivos serem escritos.

| # | Pergunta ambígua | Motivo da ambiguidade | Decisão necessária |
|---|------------------|-----------------------|--------------------|
| CA-01 | "Qual o frete para esta carga de 350 kg?" | Frete padrão (< 500 kg) não tem procedimento formal — gap total. Não há fórmula, tabela ou responsável declarado nos documentos disponíveis. | Qual área é responsável por documentar o frete padrão? Operações deve publicar o procedimento antes do go-live, ou o assistente deve sempre encaminhar ao Comercial? |
| CA-02 | "O cliente quer seguro para a carga — qual o percentual?" | FAQ-Item 22 menciona 0,3% (padrão) e 0,8% (perigosas), mas sem documento formal. O FAQ informa que contratos anteriores a 2023 podem ter percentuais diferentes. | O assistente pode citar os percentuais do FAQ com sinalização informal, ou deve sempre encaminhar ao Comercial sem fornecer valores? |
| CA-03 | "A carga chegou danificada, o que faço?" | Gap total — apenas orientação informal no FAQ-Item 38. Não há POL ou PROC formal. O processo envolve prazo (48h), fotos, laudo e encaminhamento ao Jurídico (sinistros@novatech.com.br). | O assistente pode usar o FAQ-Item 38 como orientação provisória (com sinalização), ou deve apenas encaminhar ao Jurídico sem fornecer o passo a passo? |
| CA-04 | "Qual o desconto para um cliente com 12 fretes/mês?" | PROC-042 v2 (normativa, vigente) define desconto a partir de ≥ 8 fretes/mês (5%). FAQ-Item 45 diz "> 10 fretes/mês". Para 12 fretes, a v2 concede 5% de desconto; o FAQ sugeriria que o desconto seria concedido com limiar diferente. | Confirmada a decisão do ADR-004 (v2 prevalece), o FAQ precisa ser atualizado pela equipe de atendimento antes do go-live para evitar que atendentes usem o FAQ diretamente e cometam erros. Quem é o responsável e qual o prazo? |
| CA-05 | "O chamado foi aberto ontem, mas não sei a hora exata" | A data é suficiente para aplicar as disposições transitórias entre v1 e v2 do PROC-042, mas a hora pode ser relevante para a contagem do relógio de SLA. | O assistente deve aceitar somente a data (dia/mês/ano) como suficiente para a seleção de versão, tratando qualquer chamado do mesmo dia como pertencente à versão vigente naquela data? |
| CA-06 | "O cliente diz que o frete foi mais barato no mês passado" | A tabela mensal de fretes-base muda todo mês e não está indexada. O assistente não tem acesso ao histórico de valores base. | O assistente deve informar que não tem acesso à tabela histórica e orientar o atendente a consultar o arquivo no servidor de rede, ou há expectativa de indexar versões históricas da tabela? |
| CA-07 | "Carga perigosa com frete expresso — é possível?" | FAQ-Item 32 menciona que frete expresso para carga perigosa requer autorização do Compliance e documentação ANTT atualizada, mas sem PROC formal (PROC-043 em revisão). | O assistente pode citar o FAQ-Item 32 com sinalização informal, ou deve bloquear qualquer orientação sobre esse tema até a PROC-043 ser publicada? |

---

## Critérios de Aceite

Os cenários abaixo são a base para o plano de testes de aceitação do Query Endpoint.

### CA-FUNC-001 — Resposta sobre frete especial com data do chamado

**Cenário:** Atendente pergunta: "Qual o multiplicador regional para destino Nordeste em um chamado aberto em 05/03/2024?"

**Resultado esperado:**
- Resposta usa PROC-042 v2 (status=vigente para chamados a partir de 01/12/2023)
- Informa multiplicador 1,5
- Cita: "PROC-042, v2.0, seção 2.1, atualizada em 10/11/2023"
- Exibe aviso de que existe versão anterior (v1) com valor diferente (1,4) e que a v2 é a vigente para este chamado
- Tempo de resposta ≤ 5 segundos

**Resultado inaceitável:** Retornar multiplicador da v1 (1,4) para data posterior a 01/12/2023.

---

### CA-FUNC-002 — Solicitação de dado contextual ausente

**Cenário:** Atendente pergunta: "Qual o frete especial desta carga?" sem informar peso, região ou data.

**Resultado esperado:**
- O assistente solicita: "Por favor, informe a data de abertura do chamado, o peso da carga e a região de destino para calcular o frete especial corretamente."
- Não tenta responder com valores padrão ou suposições

**Resultado inaceitável:** Responder com qualquer valor de frete sem os dados obrigatórios.

---

### CA-FUNC-003 — Gap total: seguro de carga

**Cenário:** Atendente pergunta: "Qual o percentual de seguro de carga para um cliente padrão?"

**Resultado esperado:**
- Resposta indica que o tema faz parte das operações da NovaTech, mas não há política formal disponível na base
- Orienta contatar o Comercial ou verificar o contrato do cliente
- **Não** informa o percentual do FAQ-Item 22 como resposta oficial

**Resultado inaceitável:** Informar "0,3%" como resposta oficial sem sinalização de fonte informal.

**Resultado alternativo aceitável (dependente do CA-02):** Informar "0,3% segundo o FAQ informal (não validado)" se a decisão do CA-02 autorizar uso do FAQ para este gap.

---

### CA-FUNC-004 — Tema fora do escopo

**Cenário:** Atendente pergunta: "Como está o tempo em Manaus hoje?"

**Resultado esperado:**
- Resposta informa que o tema está fora do escopo do assistente, que responde exclusivamente sobre operações logísticas da NovaTech
- Mensagem **distinta** da mensagem de gap total
- Não tenta responder

**Resultado inaceitável:** Usar a mesma mensagem de gap total, ou tentar responder sobre clima.

---

### CA-FUNC-005 — Tier Platinum inexistente

**Cenário:** Atendente pergunta: "O cliente diz que é Platinum — qual o SLA dele?"

**Resultado esperado:**
- Informa que o tier Platinum não existe na NovaTech
- Informa que os tiers vigentes são Gold, Silver e Standard
- Orienta o atendente a verificar o número do contrato para identificar o tier correto
- Cita: "SLA-2024, v2024.1, seção 1"

**Resultado inaceitável:** Assumir que o cliente é de algum tier existente sem confirmação, ou deixar de informar que o Platinum não existe.

---

### CA-FUNC-006 — Incidente crítico: carga perigosa com irregularidade

**Cenário:** Atendente relata: "Carga de gás liquefeito (classe 2) sem documentação ANTT regular."

**Resultado esperado:**
- Identifica o critério 2 de incidente crítico (carga perigosa com irregularidade de documentação)
- Informa que a situação é um incidente crítico
- Orienta sobre o SLA aplicável ao tier do cliente (solicitando o tier se não informado)
- Cita: "SLA-2024, v2024.1, seção 3"

**Resultado inaceitável:** Tratar como chamado geral sem classificar como incidente crítico.

---

### CA-FUNC-007 — Fonte informal sinalizada

**Cenário:** Atendente pergunta: "Tem alguma orientação sobre frete expresso para carga perigosa?"

**Resultado esperado:**
- Exibe informação do FAQ-Item 32 (requer autorização do Compliance e documentação ANTT atualizada)
- Exibe aviso explícito: "FAQ-Atendimento (documento informal, não validado por Compliance)."
- Menciona que a PROC-043 está em revisão e que o procedimento formal não está disponível

**Resultado inaceitável:** Apresentar a orientação do FAQ como se fosse documento oficial, sem sinalização.

---

### CA-FUNC-008 — Feedback do atendente

**Cenário:** Atendente recebe uma resposta e seleciona "Incorreta".

**Resultado esperado:**
- Interface exibe campo de texto opcional para descrição do problema
- Feedback é registrado vinculado à interação, ao documento citado e ao timestamp
- Painel de administração exibe o feedback com filtro por tipo e documento

**Resultado inaceitável:** Feedback não registrado, ou campo de texto não exibido ao selecionar "Incorreta".

---

### CA-NF-001 — Tempo de resposta

**Cenário:** 50 consultas simultâneas em condições normais de operação.

**Resultado esperado:** Percentil 95 das respostas completas retornado em ≤ 5 segundos.

---

### CA-NF-002 — Autenticação

**Cenário:** Tentativa de acesso sem credencial Azure AD válida.

**Resultado esperado:** Acesso bloqueado. Nenhuma resposta fornecida.

---

*requirements.md v1.0 — NovaTech Logística — Query Endpoint — Confidencial*
