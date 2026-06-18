# Bounded Contexts — Análise Exclusiva do Anexo A
## Domínio Logístico NovaTech

**Autor:** Análise de Negócio Sênior — Specification Driven Development  
**Fonte:** Anexo A — Documentação Simulada NovaTech (POL-001 v3.1, PROC-042 v1 e v2, SLA-2024, FAQ-Atendimento)  
**Versão:** 1.1 — Revisão de ambiguidades (AMB-001 a AMB-013)  
**Data:** Junho/2025

---

> **Nota metodológica:** Esta análise parte exclusivamente do Anexo A. Os contextos identificados são do **domínio de negócio logístico** — não são contextos técnicos de software. Cada contexto encapsula um conjunto coeso de regras, dados e decisões com linguagem ubíqua própria.

---

## Visão Geral

| # | Bounded Context | Fonte principal |
|---|-----------------|-----------------|
| BC-A1 | Cálculo e Precificação de Frete | PROC-042 v1 e v2 |
| BC-A2 | Prazos e Roteirização de Entrega | PROC-042 v1 e v2, SLA-2024, FAQ |
| BC-A3 | Devolução de Mercadorias | POL-001 v3.1, FAQ |
| BC-A4 | Cargas Especiais e Restritas | POL-001, PROC-042 v2, SLA-2024, FAQ |
| BC-A5 | Conhecimento Informal do Atendimento | FAQ-Atendimento |
| BC-A6 | Contratos e SLA por Tier de Cliente | SLA-2024 |

---

### BC-A1 — Cálculo e Precificação de Frete

#### Objetivo
Define como o valor de um serviço de transporte é calculado, quais variáveis incidem sobre o preço e quais condições especiais (peso, região, volume) alteram a equação.

#### Está dentro do contexto
- Fórmula de frete especial: Valor base × Multiplicador regional × Fator de peso
- Faixas de peso para frete especial (a partir de **500 kg inclusive**) e seus fatores multiplicadores
- Multiplicadores regionais por destino (Sul, Sudeste, Centro-Oeste, Nordeste, Norte)
- Regras de desconto por volume mensal por cliente (limiares de 8 e 15 fretes/mês na v2; 10 fretes/mês na v1)
- Tabela de fretes-base como referência de valor inicial (arquivo externo mensal)
- Regra de transição entre versões: data de abertura do chamado determina qual tabela de multiplicadores se aplica (v1 até 30/11/2023; v2 a partir de 01/12/2023)
- Aprovação prévia do gerente regional para cargas acima de 5.000 kg
- Condições especiais para cargas perigosas (remissão à PROC-043)

#### Está fora do contexto
- Prazo de entrega associado ao frete especial (pertence ao BC-A2)
- Custo do frete reverso em devoluções — embora use os mesmos multiplicadores, a decisão de cobrar ou não é do contexto de devolução (BC-A3)
- Negociação de contratos e descontos fora das regras documentadas (responsabilidade Comercial, fora do escopo dos documentos formais)
- Frete padrão abaixo de 500 kg — **gap documental: não há procedimento formal nesta base**

#### Relacionamentos
- Referenciado por **BC-A3:** custo do frete reverso usa os mesmos multiplicadores da PROC-042
- Referenciado por **BC-A5:** atendentes consultam o FAQ para orientação prática sobre qual versão usar
- Bloqueado por **BC-A4:** cargas perigosas acima de 500 kg não seguem esta tabela — seguem PROC-043 (não disponível na base)

---

### BC-A2 — Prazos e Roteirização de Entrega

#### Objetivo
Define os prazos de entrega aplicáveis a cada tipo de operação, como o tempo adicional de manuseio se soma ao prazo-base da rota, e o que se entende por "dias úteis" no contexto da NovaTech.

#### Está dentro do contexto
- Conceito de prazo padrão da rota como referência base
- Acréscimo para frete especial: +3 dias úteis (v2, vigente) ou +2 dias úteis (v1, transição)
- Definição operacional de dias úteis: exclui sábados, domingos e feriados nacionais
- Referência informal a prazos por região (Norte: até 10 dias úteis; Sul/Sudeste: > 3 dias parado é anômalo — fonte FAQ, não validada)
- Pausa do relógio de SLA fora do horário comercial (08h–18h, dias úteis) para **chamados gerais**; relógio contínuo para incidentes críticos de qualquer tier (ver ADR-008)

#### Está fora do contexto
- A tabela de rotas cadastradas com prazos por origem/destino — **gap documental: não está no Anexo A**
- Prazos para frete padrão (abaixo de 500 kg) — **gap documental**
- Prazo para coleta reversa em devoluções (pertence ao BC-A3, onde está especificado: 2 dias úteis após aprovação)
- Prazo de atendimento ao cliente (pertence ao BC-A6 — SLA)

#### Relacionamentos
- Dependência de **BC-A1:** o prazo especial só existe quando o cálculo de frete especial se aplica
- Referenciado por **BC-A6:** a definição de dias úteis é compartilhada entre os dois contextos

---

### BC-A3 — Devolução de Mercadorias

#### Objetivo
Governa todo o processo pelo qual um cliente pode solicitar a devolução de uma mercadoria entregue: elegibilidade, prazo para solicitação, procedimento operacional, responsabilidade de custos e casos especiais.

#### Está dentro do contexto
- Prazo para solicitação: 7 dias úteis após confirmação de recebimento no sistema de tracking
- Categorias de carga inelegíveis para devolução padrão:
  - Cargas perigosas classes 1–6 (ANTT Resolução 5.947/2021)
  - Cargas refrigeradas com cadeia de frio rompida (> 30 min fora da faixa, registrado por sensor IoT)
- Cargas com lacre de segurança violado **sem** documentação no ato (assinatura de motorista e recebedor) → inelegíveis para devolução padrão; **com** documentação no ato → elegíveis para devolução padrão. O assistente deve perguntar proativamente sobre a existência de documentação antes de classificar como inelegível.
- Procedimento: abertura no Portal do Cliente, documentos exigidos (CT-e, mínimo 3 fotos, motivo), triagem em 4h úteis (com pausa fora do horário comercial 08h–18h, dias úteis — mesma regra do relógio de SLA), coleta reversa em até 2 dias úteis após aprovação, reembolso em até 5 dias úteis após recebimento no CD
- Devoluções parciais: por volume individual, com hierarquia de cálculo: (1) proporcional ao valor declarado por volume no CT-e; (2) proporcional ao peso por volume se valor não discriminado; (3) encaminhar ao Operações se nenhum dos dois for discriminado por volume
- Responsabilidade de custo:
  - Defeito ou erro da NovaTech → sem custo para o cliente
  - Desistência do cliente → frete reverso por conta do cliente (mesmos multiplicadores da PROC-042)
  - Prazo expirado → não elegível, encaminhar ao Comercial
- Escalada para Gestão de Riscos (ramal 4500) para categorias inelegíveis

#### Está fora do contexto
- Carga danificada em trânsito — **gap documental crítico:** o FAQ-Item 38 descreve uma prática (registro em 48h, fotos, encaminhamento a sinistros@novatech.com.br), mas não há POL ou PROC formal
- Carga ainda em trânsito (interceptação) — regido pela PROC-088, não disponível nesta base
- O que a Gestão de Riscos faz após receber o caso de carga perigosa — **gap documental**
- Seguro de carga como instrumento de cobertura da devolução — **gap documental** (só existe no FAQ informal)

#### Relacionamentos
- Consome de **BC-A1:** multiplicadores de frete para cálculo do frete reverso
- Consome de **BC-A2:** contagem de dias úteis
- Referenciado por **BC-A6:** o prazo de triagem de devolução (4h úteis) dialoga com os SLAs de primeira resposta
- Referenciado por **BC-A5:** atendentes têm orientações práticas sobre cargas perigosas e carga danificada que contradizem ou complementam este contexto

---

### BC-A4 — Cargas Especiais e Restritas

#### Objetivo
Define as regras e restrições aplicáveis a categorias de carga que exigem tratamento diferenciado por risco, regulação ou complexidade operacional. É um contexto transversal que modifica as regras de outros contextos.

#### Está dentro do contexto
- Classificação de cargas perigosas: classes 1–6 da ANTT (Resolução 5.947/2021), com lista de categorias por classe
- Restrição de devolução padrão para cargas perigosas (remissão a Gestão de Riscos)
- Restrição de frete especial padrão para cargas perigosas acima de 500 kg (remissão à PROC-043)
- Incidente crítico envolvendo carga perigosa com irregularidade de documentação ou rastreamento (definição do SLA-2024)
- Referência informal a frete expresso para carga perigosa: requer autorização do Compliance e documentação ANTT atualizada — **fonte apenas no FAQ, sem PROC formal**
- Cargas refrigeradas: regra de cadeia de frio (> 30 min fora da faixa = inelegível para devolução padrão)
- Cargas com lacre violado: condição de inelegibilidade para devolução, com exceção documentada

#### Está fora do contexto
- O procedimento completo de frete de cargas perigosas (PROC-043 — citado, mas em revisão pelo Compliance e não disponível na base)
- O processo formal da Gestão de Riscos para cargas perigosas devolvidas — **gap documental**
- Requisitos de documentação ANTT para transporte (referenciados, mas não detalhados na base)

#### Relacionamentos
- **Modifica BC-A1:** cargas perigosas não seguem a tabela de frete especial padrão
- **Modifica BC-A3:** cargas perigosas são inelegíveis para devolução padrão
- **Modifica BC-A6:** irregularidade em carga perigosa aciona classificação de incidente crítico
- **Bloqueado por gap:** PROC-043 em revisão — a regra completa deste contexto está incompleta

---

### BC-A5 — Conhecimento Informal do Atendimento

#### Objetivo
Representa o corpo de conhecimento prático acumulado pelo time de atendimento ao longo do tempo — válido como orientação operacional, mas sem validação formal e potencialmente inconsistente com os documentos normativos.

#### Está dentro do contexto
- Orientações práticas sobre os temas cobertos (frete especial, devolução, SLA, tiers)
- Informações sobre temas não cobertos por documentos formais: seguro de carga (percentuais 0,3% e 0,8%), carga danificada em trânsito, carga perigosa com frete expresso
- Orientação sobre tier Platinum (não existe — esclarecimento correto, confirmado pelo SLA-2024)
- Referência de prazos informais por região (Norte até 10 dias, Sul/Sudeste anomalia > 3 dias)
- Orientação sobre qual versão do PROC-042 usar e alerta sobre a coexistência sem hierarquia clara
- Orientação sobre autonomia do atendente para desconto (atendente não tem autonomia — confirmado pela PROC-042)

#### Está fora do contexto
- Qualquer informação validada pelo Compliance, Operações ou Comercial — essas pertencem aos contextos normativos
- Decisões contratuais ou negociações — responsabilidade do Comercial

#### Relacionamentos
- **Contradiz BC-A1:** FAQ-Item 45 diz "desconto acima de 10 fretes/mês"; PROC-042-v2 diz "a partir de 8 fretes/mês" — conflito real entre fonte informal e normativa
- **Complementa BC-A3:** cobre gap de carga danificada que não tem documento formal
- **Complementa BC-A4:** cobre gap de frete expresso para carga perigosa que não tem PROC formal
- **Confirma BC-A6:** a explicação de SLA de resposta vs. resolução está alinhada com o SLA-2024

---

### BC-A6 — Contratos e SLA por Tier de Cliente

#### Objetivo
Define os compromissos formais de nível de serviço da NovaTech com seus clientes, segmentados por tier, incluindo critérios de classificação, métricas, penalidades e definição de incidente crítico.

#### Está dentro do contexto
- Critérios de elegibilidade e revisão dos tiers Gold, Silver e Standard
- Inexistência do tier Platinum (esclarecimento formal)
- Tabela de SLAs: tempo de primeira resposta e tempo de resolução, por tier e por tipo de chamado (geral vs. crítico)
- Disponibilidade do portal de tracking por tier (Gold 99,5% / Silver 99,0% / Standard 98,0%)
- Benefícios diferenciados: gerente de conta dedicado (Gold), relatório mensal (Gold detalhado, Silver resumido, Standard sob demanda)
- Definição precisa de incidente crítico (4 critérios: valor declarado > R$100k sem status há > 6h; carga perigosa com irregularidade; > 5 chamados do mesmo cliente sobre o mesmo problema em 24h; risco a pessoas)
- Regime de penalidades por violação (1ª, 2ª, 3ª ou mais no mesmo mês): registro interno → crédito de 5% → crédito de 10% + reunião obrigatória
- Medição pelo Azure DevOps com timestamp de abertura do chamado
- Pausa do relógio de SLA para **chamados gerais** fora do horário comercial (08h–18h, dias úteis)
- Relógio contínuo (sem pausa) para **incidentes críticos de qualquer tier** — o critério é objetivo e independe do volume contratual (ver ADR-008)
- Escalada obrigatória em 3ª violação: gerente de conta (Gold) ou gerente de operações (Silver/Standard)

#### Está fora do contexto
- Negociação do contrato e definição inicial do tier (responsabilidade Comercial)
- SLA diferenciado fora dos três tiers (encaminhar ao Comercial — confirmado pelo documento)
- Execução técnica da medição no Azure DevOps (infraestrutura de TI)
- Tracking de carga (sistema transacional externo ao escopo do Anexo A)

#### Relacionamentos
- Consome de **BC-A2:** definição de dias úteis e horário comercial
- Referenciado por **BC-A4:** irregularidade em carga perigosa aciona incidente crítico independentemente do tier
- Referenciado por **BC-A3:** em fallback de devolução sem resolução imediata, o prazo de retorno ao cliente segue o SLA do tier

---

## Conflitos e Gaps Identificados

> Este é o insumo mais crítico para a especificação funcional. Cada item abaixo exige uma decisão explícita antes de qualquer requisito ser escrito.

| # | Tipo | Contextos envolvidos | Descrição |
|---|------|----------------------|-----------|
| C1 | **Conflito** | BC-A1 | PROC-042-v1 e v2 coexistem sem hierarquia formal no SharePoint; multiplicadores regionais, fatores de peso e prazo adicional são distintos entre as versões. **Resolvido pelo ADR-003:** data de abertura do chamado determina a versão; fuso horário UTC-3, data sem hora tratada como 00h01 (ver ADR-003 e RD-003-A). |
| C2 | **Conflito** | BC-A1 / BC-A5 | FAQ-Item 45 diz "desconto acima de 10 fretes/mês"; PROC-042-v2 diz "a partir de 8 fretes/mês" — valores distintos para o mesmo tema. **Resolvido pelo ADR-004:** v2 prevalece. Desconto calculado sobre o multiplicador regional (fórmula explícita em RD-007-A). |
| G1 | **Gap total** | BC-A3 / BC-A4 | Carga danificada em trânsito: existe prática informal (FAQ-Item 38), mas nenhum POL ou PROC formal |
| G2 | **Gap total** | BC-A1 | Frete padrão abaixo de 500 kg: nenhum documento cobre esta faixa |
| G3 | **Gap total** | BC-A4 | Processo formal da Gestão de Riscos para cargas perigosas devolvidas: ramal 4500 documentado, mas sem PROC |
| G4 | **Gap total** | BC-A1 | Seguro de carga: percentuais existem apenas no FAQ-Item 22 (0,3% padrão / 0,8% perigosas), sem documento oficial |
| G5 | **Gap parcial** | BC-A4 | Frete expresso para carga perigosa: autorização do Compliance mencionada no FAQ-Item 32, mas sem PROC formal (PROC-043 em revisão pelo Compliance) |
| G6 | **Gap** | BC-A6 | Relógio de SLA em incidente crítico para Silver e Standard: silêncio documental resolvido pelo ADR-008 — relógio contínuo para todos os tiers em incidente crítico. |
| G7 | **Gap** | BC-A3 | Triagem de devolução e pausa de horário comercial: não explicitado no POL-001. Resolvido por alinhamento com BC-A6: mesmas regras de pausa se aplicam (ver RD-009-A). |

---

*Bounded Contexts v1.0 — Análise exclusiva do Anexo A — NovaTech Logística — Confidencial*
