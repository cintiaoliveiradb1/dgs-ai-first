# Cruzamento FAQ × Documentos Normativos

> **Documentos analisados:** FAQ-atendimento.md, POL-001 (v3.1), PROC-042-v2, SLA-2024
> **Finalidade:** Identificar inconsistências entre práticas informais do time de atendimento e os documentos normativos vigentes, para orientar a indexação do assistente de IA
> **Metodologia:** Cada item do FAQ foi confrontado contra os documentos oficiais disponíveis. Onde não há documento oficial, a ausência é registrada como gap.

---

## Resumo executivo

| Classificação | Itens | Recomendação para o assistente |
|---|---|---|
| Conflito direto com normativo | 3 | Bloquear — não indexar |
| Informação incompleta / sem respaldo total | 3 | Indexar com ressalva explícita |
| Sem respaldo normativo algum | 1 | Bloquear — não indexar |
| Alinhado com normativo | 2 | Indexar diretamente |
| **Total analisado** | **9** | — |

> Dos 9 itens do FAQ analisados, apenas 2 podem ser indexados diretamente no assistente. Os demais 7 exigem bloqueio, reescrita ou ressalva antes de qualquer uso.

---

## Análise item a item

---

### Item 3 — "Cliente perguntou se pode devolver carga perigosa"
**Classificação:** ⚠️ Incompleto + risco de compromisso indevido

**O que o FAQ diz:**
> Oriente o cliente a ligar no ramal 4500 (Gestão de Riscos). Oficialmente não pode pelo processo padrão, mas já tiveram casos em que o pessoal de Riscos autorizou exceção. Não diga que é impossível — diga que precisa de tratamento especial.

**O que o normativo diz (POL-001, v3.1):**
Cargas perigosas são categoria explicitamente não elegível ao processo padrão de devolução. Nenhum documento oficial menciona exceções, ramal 4500 ou tratamento especial.

**Inconsistências identificadas:**
- O FAQ cria expectativa de exceção não prevista em nenhum documento oficial
- A orientação de "não dizer que é impossível" pode gerar compromisso verbal sem respaldo normativo
- O ramal 4500 não aparece em nenhum dos documentos — se o número mudar, a informação fica inválida silenciosamente
- Ausência de qualquer procedimento formal para o cenário de exceção mencionado

**Recomendação para o assistente:** Reescrever. Substituir pelo fluxo oficial da POL-001. Remover referência a exceções e ao ramal não documentados. Escalar ao responsável por Gestão de Riscos para documentar o fluxo de exceção, se ele existir.

---

### Item 8 — "Como funciona o frete especial?"
**Classificação:** 🔴 Conflito direto — lógica de seleção de versão incorreta

**O que o FAQ diz:**
> Acima de 500kg, aplica a tabela de multiplicadores por região. Existem duas versões da PROC-042. Use a v2 na dúvida, mas se o cliente reclamar do valor, pode ser que o contrato dele esteja na tabela antiga.

**O que o normativo diz (PROC-042-v2 + análise de conflito):**
A regra de transição é baseada em data objetiva: chamados abertos a partir de 01/12/2023 usam obrigatoriamente a v2. Chamados anteriores a essa data usam a v1. A regra não depende de reclamação do cliente.

**Inconsistências identificadas:**
- O FAQ instrui o atendente a trocar de versão mediante reclamação do cliente — critério subjetivo e operacionalmente incorreto
- Essa lógica abre brecha para que clientes manipulem qual tabela será aplicada
- O FAQ não menciona os descontos automáticos por volume introduzidos pela v2 (8 fretes/mês → 5%; 15 fretes/mês → 10%)
- A referência a "contrato na tabela antiga" mistura versão do documento com condições contratuais individuais, sem distinção

**Recomendação para o assistente:** Bloquear. Não indexar este item. Substituir integralmente pela PROC-042-v2 com regra de data objetiva: chamados anteriores a 01/12/2023 usam multiplicadores da v1; demais usam v2.

---

### Item 15 — "Cliente diz que é Platinum. Existe esse tier?"
**Classificação:** ✅ Alinhado com normativo

**O que o FAQ diz:**
> Não existe tier Platinum. Tiers são Gold, Silver e Standard. O programa antigo foi descontinuado em 2022.

**O que o normativo diz (SLA-2024):**
Define exatamente três tiers: Gold, Silver e Standard. Nenhuma menção a Platinum ou programa anterior.

**Consistência:** Total. A informação sobre descontinuação em 2022 é contexto útil não coberto pelo normativo — pode ser mantida com ressalva de que não foi validada formalmente.

**Recomendação para o assistente:** Indexar. Manter com nota de que a informação sobre descontinuação em 2022 é de fonte informal e não consta em documento oficial.

---

### Item 22 — "Cliente quer saber sobre seguro de carga"
**Classificação:** 🔴 Conflito direto — valores sem respaldo normativo

**O que o FAQ diz:**
> A NovaTech oferece seguro como adicional. Valor: 0,3% do valor declarado para cargas padrão e 0,8% para cargas perigosas. Vale para contratos a partir de 2023. Contratos mais antigos podem ter percentuais diferentes — confirme com o Comercial.

**O que o normativo diz:**
Nenhum dos 5 documentos analisados menciona seguro de carga, percentuais de cobertura ou condições de apólice. Este é o gap G2 identificado no mapa de temas.

**Inconsistências identificadas:**
- Os percentuais 0,3% e 0,8% não têm origem rastreável em nenhum documento oficial
- Podem estar desatualizados, incorretos ou variar por contrato individual
- O assistente que replicar esses valores pode gerar compromisso financeiro não intencional — o cliente pode exigir o percentual citado
- A ressalva "confirme com o Comercial" no final não isenta o risco da informação principal

**Recomendação para o assistente:** Bloquear. Não indexar nenhum percentual de seguro até que exista política formal documentada. O assistente deve orientar o cliente a contatar o Comercial para informações sobre seguro, sem citar valores.

---

### Item 27 — "O tracking mostra 'em trânsito' há 5 dias"
**Classificação:** ⚠️ Incompleto — orientação prática sem respaldo formal

**O que o FAQ diz:**
> Rotas para o Norte podem levar até 10 dias úteis. Sul/Sudeste: mais de 3 dias parado é estranho. Abrir chamado de rastreamento como prioridade alta se for Gold ou se o valor da carga for acima de R$ 50.000.

**O que o normativo diz:**
O SLA-2024 define prazos de resposta e resolução por tier, mas não cobre procedimentos de rastreamento. Nenhum documento define prazos de trânsito por rota ou critério de priorização por valor de carga.

**Inconsistências identificadas:**
- Os prazos regionais (10 dias para Norte; 3 dias para Sul/Sudeste) são conhecimento tácito do time, sem fonte normativa verificável
- O critério de R$ 50.000 para classificar prioridade alta não aparece em nenhum documento
- A ausência de um manual técnico de rastreamento (gap G3) significa que o assistente não consegue orientar sobre acesso ao sistema, interpretação de status ou resolução de falhas

**Recomendação para o assistente:** Indexar com ressalva explícita. Sinalizar que prazos regionais e critério de valor são estimativas operacionais do time, não regras formais. Complementar com instrução de abertura de chamado conforme SLA-2024.

---

### Item 32 — "Pode enviar carga perigosa com frete expresso?"
**Classificação:** 🟣 Sem respaldo normativo — fluxo integralmente não documentado

**O que o FAQ diz:**
> Sim, pode — mas precisa de autorização do Compliance e documentação ANTT atualizada. Na prática demora uns 2 dias para conseguir a autorização, então o "expresso" não é tão expresso. Avise o cliente.

**O que o normativo diz:**
A PROC-042-v2 menciona apenas que cargas perigosas com peso acima de 500 kg seguem a PROC-043 — documento declaradamente em processo de revisão pelo Compliance. Nenhum documento descreve frete expresso para cargas perigosas, prazo de autorização ou requisitos ANTT.

**Inconsistências identificadas:**
- O FAQ descreve com confiança e especificidade um fluxo operacional inteiro sem base documental rastreável
- Com a PROC-043 em revisão, o processo descrito pode ter mudado ou estar suspenso
- Requisitos ANTT para transporte de cargas perigosas têm implicações regulatórias — orientação incorreta gera risco legal
- A especificidade da resposta (2 dias, autorização do Compliance, documentação ANTT) dá aparência de regra estabelecida, o que é mais perigoso do que uma resposta vaga

**Recomendação para o assistente:** Bloquear. Não indexar. Aguardar publicação da PROC-043 revisada. Até lá, o assistente deve informar que cargas perigosas com frete expresso requerem consulta direta com a área de Compliance, sem detalhar o processo.

---

### Item 38 — "Carga chegou danificada. Qual a política?"
**Classificação:** ⚠️ Incompleto — útil, mas não validado

**O que o FAQ diz:**
> Registrar a ocorrência em até 48h após o recebimento, com fotos e laudo se possível. NovaTech investiga e, se comprovada responsabilidade, reembolsa integralmente. Encaminhar para sinistros@novatech.com.br — passa pelo Jurídico, não pelo atendimento normal.

**O que o normativo diz (POL-001):**
Define o processo de devolução padrão, mas carga danificada em trânsito é um sinistro — processo distinto da devolução. A POL-001 não cobre esse cenário. Nenhum outro documento formaliza o processo de sinistro.

**Inconsistências identificadas:**
- O prazo de 48h para registro não consta em nenhum documento oficial — pode ter sido alterado
- O e-mail sinistros@novatech.com.br não aparece em nenhum documento normativo e pode estar desatualizado
- A condição de "reembolso integral" é uma promessa específica sem documento que a sustente
- O FAQ mistura conceitualmente devolução e sinistro — são processos distintos com fluxos, responsáveis e prazos diferentes; o assistente precisa fazer essa distinção

**Recomendação para o assistente:** Indexar com ressalva. Manter a distinção entre sinistro e devolução como orientação estrutural. Validar prazo de 48h, e-mail e condição de reembolso com o Jurídico antes de confirmar no corpus.

---

### Item 41 — "Qual a diferença entre SLA de resposta e SLA de resolução?"
**Classificação:** ✅ Alinhado com normativo

**O que o FAQ diz:**
> Resposta é o primeiro retorno ao cliente. Resolução é quando o problema é efetivamente resolvido. Gold: 2h resposta / 24h resolução. Silver: 4h / 48h. Standard: 8h / 72h. Incidentes críticos têm prazos menores — ver SLA-2024.

**O que o normativo diz (SLA-2024):**
Confirma exatamente os mesmos valores. Incidentes críticos: Gold 30 min, Silver 1h, Standard 2h.

**Consistência:** Total. É o item mais bem fundamentado do FAQ. A remissão à tabela SLA-2024 para incidentes críticos é adequada. O FAQ omite os prazos de incidente crítico por tier, mas não os contradiz.

**Recomendação para o assistente:** Indexar. Complementar com os prazos de incidente crítico do SLA-2024 (Gold 30 min, Silver 1h, Standard 2h).

---

### Item 45 — "O cliente quer desconto no frete. Posso dar?"
**Classificação:** 🔴 Conflito direto — gatilho de desconto desatualizado

**O que o FAQ diz:**
> Atendente não tem autonomia para dar desconto. Para clientes com mais de 10 fretes especiais por mês, existe desconto automático na tabela (PROC-042).

**O que o normativo diz (PROC-042-v2):**
- Desconto automático faixa 1: 5% sobre o multiplicador regional a partir de **8 fretes/mês** (não 10)
- Desconto automático faixa 2: 10% a partir de 15 fretes/mês
- Descontos acima de 10% requerem aprovação da Diretoria Comercial

**Inconsistências identificadas:**
- O gatilho "10 fretes/mês" do FAQ corresponde ao da PROC-042-v1, obsoleta
- Um atendente usando o FAQ negaria desconto a clientes com 8 ou 9 fretes/mês — que já têm direito pelo normativo vigente
- O FAQ não menciona a existência de dois níveis de desconto (5% e 10%) nem o limiar de 15 fretes/mês
- A lógica de alçada (aprovação da Diretoria para >10%) está completamente ausente

**Recomendação para o assistente:** Bloquear. Não indexar este item. Substituir integralmente pelos critérios da PROC-042-v2: gatilho em 8 fretes/mês (5%) e 15 fretes/mês (10%), com aprovação da Diretoria para descontos maiores.

---

## Tabela consolidada de decisão de indexação

| Item | Tema | Classificação | Decisão | Condição |
|---|---|---|---|---|
| 3 | Devolução de carga perigosa | ⚠️ Incompleto + risco | Reescrever | Substituir por POL-001; remover ramal e menção a exceções |
| 8 | Como funciona o frete especial | 🔴 Conflito direto | Bloquear | Substituir por PROC-042-v2 com regra de data objetiva |
| 15 | Tier Platinum — existe? | ✅ Alinhado | Indexar | Manter com nota sobre descontinuação em 2022 (fonte informal) |
| 22 | Seguro de carga — percentuais | 🔴 Conflito direto | Bloquear | Aguardar política formal de seguro; orientar a contatar Comercial |
| 27 | Tracking parado há 5 dias | ⚠️ Incompleto | Indexar c/ ressalva | Sinalizar que prazos regionais e critério R$ 50k são estimativas |
| 32 | Carga perigosa com frete expresso | 🟣 Sem respaldo | Bloquear | Aguardar PROC-043 revisada; orientar consulta ao Compliance |
| 38 | Carga chegou danificada | ⚠️ Incompleto | Indexar c/ ressalva | Validar prazo 48h, e-mail e condição de reembolso com Jurídico |
| 41 | SLA de resposta vs resolução | ✅ Alinhado | Indexar | Complementar com prazos de incidente crítico do SLA-2024 |
| 45 | Atendente pode dar desconto? | 🔴 Conflito direto | Bloquear | Substituir por PROC-042-v2 com gatilhos corretos (8 e 15/mês) |

---

## Padrão de origem dos conflitos

Os três conflitos diretos (itens 8, 22 e 45) têm origem comum: o FAQ foi escrito parcialmente com base em versões ou orientações anteriores que foram superadas por documentos mais recentes.

O item 45 usa explicitamente o gatilho da PROC-042-v1 (10 fretes/mês). O item 8 reproduz uma lógica de seleção de versão que nunca existiu formalmente. O item 22 apresenta percentuais de seguro sem fonte rastreável — provavelmente oriundos de uma tabela comercial antiga ou de orientação verbal do time.

O item 32 é o caso mais crítico em termos de aparência enganosa: a especificidade da resposta (2 dias, Compliance, ANTT) dá falsa impressão de regra estabelecida, quando na realidade o único documento que regula o tema (PROC-043) está declaradamente em revisão.

---

## Ações necessárias antes da indexação

| Prioridade | Ação | Responsável | Impacto se não feita |
|---|---|---|---|
| 🔴 Imediata | Bloquear itens 8, 22, 32 e 45 do corpus do assistente | Equipe de IA | Erros de cálculo, valores incorretos e compromissos indevidos com clientes |
| 🔴 Imediata | Substituir item 45 pelos critérios da PROC-042-v2 | Equipe de IA | Negação de desconto legítimo a clientes com 8–9 fretes/mês |
| 🟠 Alta | Criar política formal de seguro de carga (gap G2) | Comercial / Jurídico | Assistente sem resposta para tema de alto impacto financeiro |
| 🟠 Alta | Aguardar e indexar PROC-043 revisada | Compliance | Fluxo de cargas perigosas sem cobertura normativa |
| 🟠 Alta | Validar prazo de 48h, e-mail de sinistros e condição de reembolso do item 38 | Jurídico | Compromisso não intencional com cliente sobre indenização |
| 🟡 Média | Documentar formalmente o fluxo de exceção de devolução para cargas perigosas | Operações / Compliance | FAQ cria expectativa de exceção sem processo definido |
| 🟡 Média | Criar manual técnico de rastreamento (gap G3) | TI / Operações | Assistente incapaz de orientar dúvidas operacionais do dia a dia |

---

*Análise gerada em maio/2025 com base na leitura cruzada de FAQ-atendimento.md, POL-001-politica-devolucao.md, PROC-042-v2-frete-especial-revisado.md e SLA-2024-tabela-sla-clientes.md, e dos documentos de análise produzidos anteriormente.*
