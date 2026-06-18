# Linguagem Ubíqua — Domínio Logístico NovaTech
## Extração exclusiva do Anexo A

**Autor:** Análise de Negócio Sênior — Specification Driven Development  
**Fonte:** Anexo A — POL-001 v3.1, PROC-042 v1, PROC-042 v2, SLA-2024, FAQ-Atendimento  
**Versão:** 1.0  
**Data:** Junho/2025

---

> **Convenções desta tabela**
> - **[HIPÓTESE]** — definição não está explícita no Anexo A; proposta inferida a partir do contexto.
> - **[CONFLITO]** — o próprio Anexo A usa o termo de formas incompatíveis entre documentos.
> - **[GAP]** — termo referenciado, mas sem definição na base disponível.
> - **Sinônimos proibidos** — termos que circulam no FAQ ou no uso coloquial e que devem ser banidos da especificação para evitar ambiguidade.

---

## Tabela de Linguagem Ubíqua

| Termo | Definição oficial | Exemplo de uso | Sinônimos proibidos | Fonte |
|---|---|---|---|---|
| **Frete especial** | Modalidade de transporte aplicável a cargas com peso acima de 500 kg, cujo valor é calculado pela fórmula: Valor base × Multiplicador regional × Fator de peso. | "O chamado envolve frete especial pois a carga tem 800 kg." | frete pesado, frete diferenciado, frete acima do padrão | PROC-042 v1 §1, PROC-042 v2 §1 |
| **Valor base** | Tarifa publicada na tabela mensal de fretes disponível no servidor de rede (`\\novatech-fs\comercial\tabelas\frete-base-AAAAMM.xlsx`). Não está indexada na base de conhecimento atual. | "Para calcular o frete especial, consulte o valor base da tabela de outubro." | tarifa base, preço base, custo base | PROC-042 v1 §2, PROC-042 v2 §2 |
| **Multiplicador regional** | Fator numérico aplicado sobre o valor base conforme a região de destino da carga. **[CONFLITO]** Os valores diferem entre v1 e v2 — ver nota abaixo da tabela. | "Destino Norte: multiplicador 1,8 (v2 vigente)." | fator regional, coeficiente de região, índice de destino | PROC-042 v1 §2.1, PROC-042 v2 §2.1 |
| **Fator de peso** | Multiplicador aplicado sobre o produto (valor base × multiplicador regional) conforme a faixa de peso da carga. **[CONFLITO]** Os valores diferem entre v1 e v2 — ver nota abaixo da tabela. | "Carga de 2.000 kg: fator de peso 1,15 (v2 vigente)." | fator de tonelagem, coeficiente de peso, índice de peso | PROC-042 v1 §2, PROC-042 v2 §2 |
| **Frete padrão** | **[GAP]** Modalidade de transporte para cargas abaixo de 500 kg. Nenhum documento do Anexo A define sua fórmula ou tabela. | — | frete normal, frete simples, frete comum | Referenciado implicitamente em PROC-042 v1 §1 e v2 §1 |
| **Frete reverso** | Frete cobrado para o retorno de uma mercadoria ao centro de distribuição da NovaTech em caso de devolução por desistência do cliente. Calculado com os mesmos multiplicadores do frete original. | "Como a desistência é do cliente, o frete reverso é por conta dele." | frete de volta, frete de retorno, frete de devolução | POL-001 v3.1 §3.5 |
| **Dias úteis** | Dias de segunda a sexta-feira, excluídos feriados nacionais. Sábados e domingos nunca são dias úteis no contexto NovaTech. | "O prazo de 7 dias úteis não conta sábado, domingo nem feriado nacional." | dias corridos, dias hábeis | POL-001 v3.1 §3.1, SLA-2024 §5 |
| **Horário comercial** | Período das 08h às 18h em dias úteis. Fora deste horário o relógio de SLA pausa para chamados gerais. | "O chamado foi aberto às 17h50 — o SLA de 2h úteis começa a contar às 08h do dia seguinte." | horário de funcionamento, horário de atendimento, expediente | SLA-2024 §5 |
| **Data de recebimento confirmada** | Data registrada no sistema de tracking como confirmação formal da entrega ao destinatário. É o marco zero para contagem do prazo de devolução. | "A contagem dos 7 dias úteis inicia na data de recebimento confirmada no tracking." | data de entrega, data de recebimento, data de chegada | POL-001 v3.1 §3.1 |
| **CT-e** | Conhecimento de Transporte Eletrônico. Documento fiscal obrigatório que acompanha toda operação de transporte e serve como referência de identificação do chamado de devolução. | "O chamado de devolução deve incluir o número do CT-e." | nota de transporte, conhecimento de frete, documento de transporte | POL-001 v3.1 §3.3 |
| **Cadeia de frio** | Condição de conservação de temperatura exigida para cargas refrigeradas, especificada na nota fiscal. Considera-se rompida quando a temperatura fica fora da faixa por mais de 30 minutos contínuos, conforme registro do sensor IoT. | "O sensor IoT registrou 35 minutos fora da faixa — cadeia de frio rompida, devolução inelegível." | temperatura controlada, controle de temperatura | POL-001 v3.1 §3.2 |
| **Lacre de segurança** | Dispositivo físico que sela a carga. Quando violado sem documentação no ato da entrega (assinatura de motorista e recebedor), a carga torna-se inelegível para devolução padrão. | "Lacre violado sem registro no ato — encaminhar à Gestão de Riscos." | lacre, selo, trava de segurança | POL-001 v3.1 §3.2 |
| **Devolução padrão** | Processo de devolução regido pela POL-001, disponível a clientes dentro do prazo e com carga elegível. Distingue-se do tratamento especial aplicado a cargas restritas. | "Carga padrão dentro do prazo: abre chamado pelo processo de devolução padrão." | devolução normal, devolução comum, devolução regular | POL-001 v3.1 §1 |
| **Devolução parcial** | Devolução de volumes individuais de uma entrega com múltiplos volumes. O reembolso é proporcional ao peso/valor do volume devolvido, conforme o CT-e. | "O cliente quer devolver 3 dos 10 volumes — devolução parcial, calcular proporcional ao CT-e." | devolução de parte, devolução incompleta | POL-001 v3.1 §3.4 |
| **Coleta reversa** | Operação de retirada da mercadoria devolvida no endereço do cliente, agendada em até 2 dias úteis após a aprovação da devolução. | "Devolução aprovada — agendar coleta reversa em até 2 dias úteis." | retirada, busca, coleta de retorno | POL-001 v3.1 §3.3 |
| **Centro de distribuição (CD)** | Instalação da NovaTech onde as mercadorias devolvidas são recebidas. O prazo de reembolso de 5 dias úteis começa a contar a partir do recebimento no CD. | "Mercadoria recebida no CD em 10/06 — reembolso até 17/06." | armazém, depósito, galpão | POL-001 v3.1 §3.3 |
| **Portal do Cliente** | Canal oficial para abertura de chamados de devolução, acessível em portal.novatech.com.br. É o único canal aceito para início do processo de devolução padrão. | "O cliente deve abrir o chamado no Portal do Cliente — não aceitar solicitação por e-mail ou telefone." | site, sistema do cliente, plataforma | POL-001 v3.1 §3.3 |
| **Triagem** | Etapa interna de verificação de elegibilidade, documentação e prazo de um chamado de devolução. Deve ser concluída em até 4 horas úteis após a abertura. | "Chamado aberto às 10h — triagem deve ser concluída até 14h." | análise, verificação, avaliação inicial | POL-001 v3.1 §3.3 |
| **Gestão de Riscos** | Área interna da NovaTech responsável pelo tratamento individual de devoluções de cargas inelegíveis (perigosas, com cadeia de frio rompida, com lacre violado). Contato: ramal 4500. **[GAP]** O procedimento interno desta área não está documentado no Anexo A. | "Carga perigosa solicitando devolução — encaminhar à Gestão de Riscos, ramal 4500." | área de risco, setor de risco, compliance de risco | POL-001 v3.1 §3.2 |
| **Carga perigosa** | Mercadoria classificada nas classes 1 a 6 da ANTT, conforme Resolução ANTT nº 5.947/2021: classe 1 (explosivos), classe 2 (gases), classe 3 (líquidos inflamáveis), classe 4 (sólidos inflamáveis), classe 5 (oxidantes e peróxidos), classe 6 (substâncias tóxicas e infectantes). | "A carga é gás liquefeito — classe 2, carga perigosa, não elegível para devolução padrão." | carga de risco, carga especial, produto perigoso | POL-001 v3.1 §3.2, SLA-2024 §3 |
| **Frete de cargas perigosas** | **[GAP]** Modalidade especial de frete para cargas perigosas acima de 500 kg, regida pela PROC-043. Este documento está em revisão pelo Compliance e não está disponível no Anexo A. | — | — | PROC-042 v2 §4 |
| **Tier** | Classificação do cliente da NovaTech em três categorias (Gold, Silver, Standard) com base no volume mensal de operações e no valor do contrato anual. Determina os SLAs aplicáveis. | "Verificar o tier do cliente antes de informar o prazo de resposta." | categoria, nível, perfil, plano | SLA-2024 §1 |
| **Gold** | Tier mais elevado. Critério: contrato anual acima de R$ 500.000 OU mais de 200 operações/mês. Revisão semestral. | "Cliente Gold tem SLA de primeira resposta de 2h úteis." | premium, VIP, platinum, top | SLA-2024 §1 |
| **Silver** | Tier intermediário. Critério: contrato anual entre R$ 100.000 e R$ 500.000 OU entre 50 e 200 operações/mês. Revisão semestral. | "Cliente Silver tem SLA de primeira resposta de 4h úteis." | intermediário, médio, plus | SLA-2024 §1 |
| **Standard** | Tier base. Critério: todos os clientes que não atendem os critérios Gold ou Silver. Revisão anual. | "Sem contrato identificado — tratar como Standard até confirmação." | básico, comum, normal, regular | SLA-2024 §1 |
| **Platinum** | **Tier inexistente na NovaTech.** Clientes que alegam ser Platinum estão confundindo com outra transportadora ou com programa de fidelidade descontinuado em 2022. | "Cliente alega ser Platinum — orientar que os tiers são Gold, Silver e Standard; pedir número do contrato." | — | SLA-2024 §1 (nota), FAQ-Item 15 |
| **SLA de primeira resposta** | Tempo máximo para o atendente dar o primeiro retorno ao cliente após abertura do chamado, mesmo que a resolução ainda não esteja disponível. | "Primeiro retorno em até 2h úteis para Gold — pode ser 'estamos verificando'." | tempo de resposta, prazo de resposta | SLA-2024 §2, FAQ-Item 41 |
| **SLA de resolução** | Tempo máximo para o problema do cliente ser efetivamente resolvido após abertura do chamado. | "Resolução em até 24h úteis para Gold — quando o problema estiver de fato encerrado." | tempo de resolução, prazo de encerramento | SLA-2024 §2, FAQ-Item 41 |
| **Incidente crítico** | Chamado que atende a pelo menos um de quatro critérios: (1) carga com valor declarado acima de R$ 100.000 com status desconhecido há mais de 6 horas; (2) carga perigosa com qualquer irregularidade de documentação ou rastreamento; (3) mais de 5 chamados do mesmo cliente nas últimas 24 horas sobre o mesmo problema; (4) risco à segurança de pessoas. | "Status desconhecido há 7h em carga de R$ 150k — classificar como incidente crítico." | urgente, crítico, prioritário, P1 | SLA-2024 §3 |
| **Violação de SLA** | Ocorrência em que o tempo de primeira resposta ou de resolução ultrapassa o limite contratual do tier do cliente. Primeira violação no mês: registro interno sem impacto contratual. Segunda: crédito de 5%. Terceira ou mais: crédito de 10% mais reunião obrigatória. | "Segundo chamado do mês com SLA estourado para cliente Gold — emitir crédito de 5%." | estouro de SLA, descumprimento de prazo, breach | SLA-2024 §4 |
| **Relógio de SLA** | Contagem de tempo que mede o cumprimento dos SLAs. Pausa fora do horário comercial para chamados gerais. Não pausa para incidentes críticos de clientes Gold. | "Chamado geral aberto às 17h30 — relógio de SLA pausa às 18h e retoma às 08h do próximo dia útil." | contador de SLA, temporizador, cronômetro de atendimento | SLA-2024 §5 |
| **Gerente de conta dedicado** | Benefício exclusivo do tier Gold: profissional designado especificamente para o cliente, acionado obrigatoriamente a partir da terceira violação de SLA no mês. | "Terceira violação — convocar gerente de conta dedicado para reunião obrigatória." | account manager, gestor de conta | SLA-2024 §2, §4 |
| **Desconto por volume** | Redução percentual aplicada sobre o multiplicador regional quando o cliente ultrapassa limiares mensais de fretes especiais. **[CONFLITO]** v1 define limiar único de 10 fretes/mês; v2 define dois limiares: 8 fretes/mês (5% de desconto) e 15 fretes/mês (10% de desconto). FAQ-Item 45 alinha com v1, conflitando com v2. | "Cliente com 9 fretes especiais no mês — desconto de 5% pelo multiplicador regional (v2 vigente)." | desconto de fidelidade, desconto comercial, desconto de frequência | PROC-042 v1 §4, PROC-042 v2 §4, FAQ-Item 45 |
| **Aprovação do gerente regional** | Autorização prévia obrigatória do gerente de operações regional para cargas acima de 5.000 kg. O assistente não pode confirmar valores para essas cargas sem esta aprovação. | "Carga de 6.000 kg — não calcular frete; acionar gerente regional para aprovação prévia." | aprovação regional, liberação do gerente | PROC-042 v1 §4, PROC-042 v2 §4 |
| **Disposições transitórias** | Regras que definem qual versão do PROC-042 se aplica conforme a data de abertura do chamado: v1 para chamados abertos até 30/11/2023; v2 para chamados abertos a partir de 01/12/2023. | "Chamado aberto em 15/11/2023 — aplicar multiplicadores da v1 pelas disposições transitórias." | regras de transição, período de transição | PROC-042 v2 §5 |
| **Seguro de carga** | **[GAP + HIPÓTESE]** Serviço adicional oferecido pela NovaTech com custo de 0,3% do valor declarado para cargas padrão e 0,8% para cargas perigosas (contratos a partir de 2023). Informação presente apenas no FAQ-Item 22 — sem documento formal. Contratos anteriores a 2023 podem ter percentuais diferentes. | "Cliente pergunta sobre seguro — informar que não há política formal disponível; encaminhar ao Comercial." | proteção de carga, cobertura de carga | FAQ-Item 22 |
| **Carga danificada** | **[GAP + HIPÓTESE]** Mercadoria que chegou ao destinatário com avaria ocorrida durante o transporte. Processo informal descrito no FAQ-Item 38: registro em até 48h com fotos e laudo, encaminhamento a sinistros@novatech.com.br e tratamento pelo Jurídico. Não há POL ou PROC formal. | "Cliente reporta carga danificada — orientar registro em até 48h e encaminhar ao Jurídico." | avaria, sinistro, carga avariada | FAQ-Item 38 |
| **Sinistro** | **[HIPÓTESE]** Termo usado internamente para eventos de dano, perda ou irregularidade grave em carga durante o transporte, tratados pelo setor Jurídico. Distinto de "devolução", que é o processo voluntário iniciado pelo cliente. | "Carga danificada em trânsito é sinistro — não confundir com devolução padrão." | incidente, ocorrência, dano | FAQ-Item 38 |
| **Interceptação de carga** | Procedimento para cargas ainda em trânsito (não entregues). Regido pela PROC-088, não disponível no Anexo A. Distinto de devolução, que se aplica somente após a entrega. | "Carga ainda em trânsito — não é devolução; acionar PROC-088 de interceptação." | retenção, bloqueio de entrega | POL-001 v3.1 §2 |
| **Aditivo contratual** | Documento formal que registra condições negociadas fora das regras padrão, como descontos de volume acima dos limiares da PROC-042. Deve ser emitido pelo Comercial. | "Desconto acima de 10% requer aprovação da Diretoria Comercial e registro em aditivo contratual." | adendo, anexo contratual | PROC-042 v2 §4 |

---

## Notas sobre Conflitos Identificados

### Conflito 1 — Multiplicador regional (C1-MR)
| Região | PROC-042 v1 | PROC-042 v2 (vigente) |
|--------|-------------|----------------------|
| Sul | 1,2 | 1,3 |
| Sudeste | 1,0 | 1,1 |
| Centro-Oeste | 1,3 | 1,4 |
| Nordeste | 1,4 | 1,5 |
| Norte | 1,6 | 1,8 |

**Regra de desambiguação:** usar v2 para chamados abertos a partir de 01/12/2023; usar v1 para chamados abertos até 30/11/2023 (PROC-042 v2 §5).

### Conflito 2 — Fator de peso (C1-FP)
| Faixa | PROC-042 v1 | PROC-042 v2 (vigente) |
|-------|-------------|----------------------|
| 500–1.000 kg | 1,0 | 1,0 |
| 1.001–3.000 kg | 1,2 | 1,15 |
| acima de 3.000 kg | 1,5 | 1,4 |

**Regra de desambiguação:** mesma regra de data acima.

### Conflito 3 — Prazo adicional de frete especial (C1-PZ)
| Versão | Prazo adicional |
|--------|----------------|
| PROC-042 v1 | +2 dias úteis |
| PROC-042 v2 (vigente) | +3 dias úteis |

**Regra de desambiguação:** mesma regra de data acima.

### Conflito 4 — Limiar de desconto por volume (C2-DV)
| Fonte | Limiar | Desconto |
|-------|--------|---------|
| PROC-042 v1 | > 10 fretes/mês | negociar com Comercial |
| PROC-042 v2 (vigente) | ≥ 8 fretes/mês | 5% sobre multiplicador regional |
| PROC-042 v2 (vigente) | ≥ 15 fretes/mês | 10% sobre multiplicador regional |
| FAQ-Item 45 | > 10 fretes/mês | desconto automático na tabela |

**Decisão necessária:** o FAQ está alinhado com a v1 e conflita com a v2. Para chamados vigentes, a v2 prevalece — mas o FAQ precisa ser atualizado para evitar orientação errada pelo atendente.

---

*Linguagem Ubíqua v1.0 — Análise exclusiva do Anexo A — NovaTech Logística — Confidencial*
