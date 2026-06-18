# Sessão de Trabalho — Tech Lead · NovaTech Logística
**Data:** 11/06/2026  
**Escopo:** Análise de ambiguidades e revisão de especificação do Query Endpoint — Assistente de Atendimento IA  
**Artefatos de entrada:** requirements.md v1.0 · bounded-contexts-anexo-a.md v1.0 · linguagem-ubiqua-anexo-a.md v1.0 · mockups (3 imagens)  
**Artefatos gerados:** requirements.md v1.1 · bounded-contexts-anexo-a.md v1.1 · linguagem-ubiqua-anexo-a.md v1.1

---

## Turno 1 — Solicitação de análise de ambiguidades

**Usuário:**
> Atue como tech lead, analise os arquivos anexos e aponte ambiguidades.

**Assistente:**

Após leitura completa dos arquivos `requirements.md`, `bounded-contexts-anexo-a.md`, `linguagem-ubiqua-anexo-a.md` e dos três mockups do Microsoft Teams, foram identificadas 13 ambiguidades organizadas por severidade.

---

### 🔴 Críticas — Bloqueiam implementação ou testes

#### AMB-001 — Limiar de frete especial: 500 kg é inclusivo ou exclusivo?

A RD-001 diz "acima de 500 kg". A linguagem ubíqua diz "peso acima de 500 kg". A tabela de fatores de peso na RD-005 começa em "500–1.000 kg". Uma carga de **exatamente 500 kg** está dentro ou fora do frete especial?

Os dois documentos usam "acima de" (exclusivo), mas a tabela de faixas inclui o 500. Há contradição interna. O cenário U-04 usa 800 kg e o CA-FUNC-001 usa Nordeste sem mencionar o limiar — o edge case nunca é coberto nos testes.

**Decisão necessária:** 500 kg é frete especial? O threshold é `> 500` ou `>= 500`?

---

#### AMB-002 — Relógio de SLA: pausa se aplica a incidentes críticos de Silver e Standard?

RD-015 diz: "Incidentes críticos de clientes **Gold**: relógio contínuo, sem pausa." Isso implica que Silver e Standard têm pausa mesmo em incidentes críticos? Ou o relógio contínuo é para todos os tiers em incidente crítico, sendo Gold apenas o exemplo explicitado?

O RF-009 confirma "com relógio contínuo para Gold" sem mencionar os outros tiers. O BC-A6 declara "relógio contínuo (sem pausa) para incidentes críticos de clientes Gold" — silêncio sobre Silver/Standard. É um gap documental ou uma regra deliberada?

**Impacto direto:** o comportamento do assistente no cenário CA-FUNC-006, que solicita o tier antes de informar o SLA, ficará inconsistente se o time implementar a pausa para Silver/Standard e o Product não tiver decidido isso.

---

#### AMB-003 — Score de similaridade como único critério de gap total (RF-005 vs. RF-002)

O RF-002 aciona a solicitação de dado contextual quando "a resposta depende de um dado não fornecido". O RF-005 aciona a resposta de gap total quando "nenhum trecho com score ≥ 0,75 é recuperado". O problema: **um tema de gap total poderia ter score alto** se o FAQ-Item contiver texto semanticamente próximo à pergunta.

Exemplo: a pergunta "qual o seguro para carga perigosa?" pode recuperar o FAQ-Item 22 com score ≥ 0,75. O pipeline seguiria RF-001 (resposta com fonte informal) ou RF-005 (gap total)? Os requisitos usam score como proxy de cobertura, mas isso não distingue "documento formal ausente" de "tema não coberto".

**Decisão necessária:** o gap total é definido pela ausência de **documento primário** ou pela ausência de qualquer trecho acima do threshold? A lista de gaps da ADR-002 deve ser tratada como hardcoded no pipeline ou inferida dinamicamente pelo score?

---

#### AMB-004 — Base de cálculo do desconto: sobre o multiplicador ou sobre o total?

RD-007 define: "desconto de 5% sobre o multiplicador regional". Mas a fórmula é `Valor base × Multiplicador regional × Fator de peso`. O desconto reduz o multiplicador antes de multiplicar pelo fator de peso? Ou é aplicado sobre o valor final do frete especial?

Matematicamente não é a mesma coisa:
- Opção A: `VB × (MR × 0,95) × FP`
- Opção B: `(VB × MR × FP) × 0,95`

O documento diz "sobre o multiplicador regional" — sugere a Opção A — mas a linguagem ubíqua não esclarece. A diferença pode ser material em cargas pesadas.

---

### 🟡 Relevantes — Afetam comportamento em casos reais

#### AMB-005 — CA-05: data sem hora e o problema do fuso/corte de dia

O CA-05 pergunta "chamado aberto ontem, sem hora exata" e a decisão pendente é se a **data apenas** é suficiente. Isso tem uma segunda camada: se o sistema registra em UTC e o atendente usa horário de Brasília (UTC-3), um chamado aberto às 23h de 30/11/2023 em Brasília é 02h de 01/12/2023 em UTC — e muda de versão. Os requisitos são completamente silenciosos sobre fuso horário.

---

#### AMB-006 — Triagem de 4h úteis: começa a contar quando?

RD-009 e o BC-A3 definem triagem em "4 horas úteis após abertura". O exemplo da linguagem ubíqua ("Chamado aberto às 10h — triagem até 14h") é simples. Mas e um chamado aberto às 17h? A triagem termina às 21h (sem pausa) ou às 09h do dia seguinte (com pausa pelo horário comercial)? O relógio de SLA de 08h–18h se aplica à triagem de devolução ou apenas aos SLAs de atendimento do BC-A6?

O BC-A3 diz que o prazo de triagem "dialoga com os SLAs de primeira resposta" (relacionamento com BC-A6), mas não declara explicitamente que as mesmas regras de pausa se aplicam.

---

#### AMB-007 — "Mesmo problema" no critério 3 de incidente crítico

O critério 3 define: "> 5 chamados do mesmo cliente sobre o **mesmo problema** nas últimas 24h." "Mesmo problema" não tem definição na linguagem ubíqua nem nos RDs. O pipeline precisará decidir se dois chamados são "o mesmo problema" com base em quê? Número de CT-e, texto semântico, número de contrato, categoria do chamado?

Isso importa especialmente porque o assistente é instruído a identificar incidentes críticos (RF-009) — e ele precisaria raciocinar sobre chamados anteriores que podem não estar no contexto da conversa.

---

#### AMB-008 — Devolução parcial: proporcional ao peso **ou** ao valor?

A linguagem ubíqua diz "proporcional ao peso/valor do volume devolvido, conforme o CT-e". O BC-A3 diz "cálculo proporcional ao peso/valor no CT-e". O uso de barra (`peso/valor`) é ambíguo: é peso **ou** valor (o CT-e define qual)? É peso **e** valor (alguma fórmula que combina ambos)? Se o CT-e tiver apenas peso, usa peso. Se tiver valor, usa valor? Nenhum documento explica a lógica de escolha.

---

#### AMB-009 — RF-008: janela de 24h de defasagem do FAQ — de onde vem esse número?

O RF-008 menciona "janela de até 24h de defasagem do FAQ" como critério para suprimir o FAQ em favor da fonte primária. Esse número não aparece em nenhum outro documento. É um parâmetro configurável? Como o sistema sabe que o FAQ está em defasagem — há um mecanismo de timestamp de atualização indexado? O RF nunca explica o mecanismo de detecção, apenas o comportamento esperado.

---

#### AMB-010 — Feedback "Incorreta" vs. "Incompleta": o que dispara revisão do FAQ?

O RF-010 registra feedback vinculado à interação e ao documento. O ADR-004 menciona que "o assistente sinaliza internamente o conflito para revisão do FAQ". O RF-008 menciona "sinaliza internamente o item do FAQ para revisão prioritária". Mas o RF-010 não descreve nenhum fluxo de sinalização automática. São dois sistemas diferentes ou o mesmo? Quem consome a fila de revisão?

---

### 🟢 Menores — Inconsistências de linguagem ou edge cases documentados

#### AMB-011 — "Carga com lacre violado" no mockup vs. no requisito

O BC-A3 encaminha lacre violado **sem documentação** à Gestão de Riscos, mas lacre violado **com documentação no ato** é elegível para devolução padrão. O RF e os cenários cobrem o caso inelegível mas não especificam o comportamento quando o atendente informa que **há** documentação — o assistente deve perguntar proativamente?

---

#### AMB-012 — Mockup mostra "PROC-042 v2.0" mas os documentos chamam de "v2"

A imagem 2 mostra a citação de fonte como `PROC-042, v2.0, seção 2.1, atualizada em 10/11/2023`. O CA-FUNC-001 especifica exatamente esse formato. Mas o requirements.md e os bounded contexts referenciam como "PROC-042 v2" (sem ".0"). Qual é o valor canônico?

---

#### AMB-013 — RF-012 (modo degradado): o caminho/URL da documentação não está definido

O RF-012 especifica a mensagem de fallback com `[caminho/URL]` como placeholder. Sem esse valor definido, o modo degradado retorna uma mensagem incompleta para o atendente em situação crítica.

---

### Resumo executivo

| Severidade | Qtd | Temas principais |
|------------|-----|-----------------|
| 🔴 Crítica | 4 | Threshold 500 kg, relógio de SLA por tier em incidente crítico, gap total via score vs. lista hardcoded, base de cálculo do desconto |
| 🟡 Relevante | 6 | Fuso horário, triagem e pausa de SLA, "mesmo problema" indefinido, devolução parcial peso/valor, RF-008 sem mecanismo, fila de revisão de FAQ |
| 🟢 Menor | 3 | Lacre com documentação, versioning de string de citação, URL de modo degradado |

As AMB-001, AMB-002 e AMB-003 são as que devem ser priorizadas para resolução antes de qualquer sprint de desenvolvimento.

---

## Turno 2 — Solicitação de soluções

**Usuário:**
> Me sugira soluções para as ambiguidades apresentadas acima.

**Assistente:**

---

### 🔴 Críticas

#### AMB-001 — Solução: adotar `>= 500 kg` (inclusivo)

A tabela de fatores de peso já começa em "500–1.000 kg", o que implica que 500 kg tem fator definido. Em conflito entre narrativa e tabela, a tabela é mais específica e prevalece. Adotar o exclusivo criaria um buraco: 500 kg exato não teria frete especial nem frete padrão documentado.

**Ações:**
- Substituir "acima de 500 kg" por "com peso igual ou superior a 500 kg" na RD-001, linguagem ubíqua e escopo do assistente.
- Adicionar cenário `CA-FUNC-001-B`: carga de exatamente 500 kg → frete especial, fator 1,0.
- Registrar como ADR-007.

---

#### AMB-002 — Solução: relógio contínuo para todos os tiers em incidente crítico (Opção B)

O critério de incidente crítico é objetivo (4 gatilhos formais) e nenhum deles é menos urgente por ser de cliente Silver. A distinção de Gold na documentação original provavelmente reflete o benefício contratual do relógio contínuo em chamados *gerais*, não uma restrição de incidentes críticos.

**Ações:**
- Atualizar RD-015: "Incidentes críticos: relógio contínuo para todos os tiers. Chamados gerais: pausa fora do horário comercial."
- Atualizar RF-009: remover "com relógio contínuo para Gold".
- Registrar como ADR-008.

---

#### AMB-003 — Solução: arquitetura em duas camadas

```
1. Pipeline verifica se o tema corresponde a um gap da lista ADR-002
   → SIM: resposta de gap total imediata, sem consultar índice vetorial
   → NÃO: continua para busca vetorial

2. Busca vetorial:
   → Score >= 0,75 com fonte primária: RF-001 (resposta oficial)
   → Score >= 0,75 com fonte apenas secundária (FAQ): RF-007 (resposta com sinalização)
   → Score < 0,75 em todas as fontes: RF-005 (gap total por ausência de cobertura)
```

A lista de gaps da ADR-002 deve ser mantida como configuração versionada no repositório, não embutida no prompt do modelo. Registrar como ADR-009.

---

#### AMB-004 — Solução: desconto sobre o multiplicador (Opção A)

Fórmula canônica: `Frete especial = Valor base × (Multiplicador regional × (1 − desconto%)) × Fator de peso`

**Exemplo:** destino Norte (MR = 1,8), ≥ 8 fretes/mês, carga 800 kg (FP = 1,0): MR com desconto = 1,8 × 0,95 = 1,71. Frete = Valor base × 1,71 × 1,0.

**Ações:** adicionar RD-007-A com fórmula e exemplo canônico; adicionar CA-FUNC-001-C.

---

### 🟡 Relevantes

#### AMB-005 — Solução: fuso UTC-3 canônico + data sem hora = 00h01

- Todos os timestamps em UTC-3 (America/Sao_Paulo).
- Data sem hora informada → tratar como 00h01 daquela data para fins de versão do PROC-042.
- Para relógio de SLA: hora desconhecida → RF-002 (solicitar ao atendente).
- Azure DevOps registra em UTC → converter para UTC-3 na camada de apresentação.
- Registrar como RD-003-A.

---

#### AMB-006 — Solução: mesmas regras de pausa do relógio de SLA aplicam-se à triagem

O BC-A3 declara que o prazo de triagem "dialoga com os SLAs de primeira resposta" do BC-A6. Os operadores de triagem trabalham no mesmo horário comercial.

**Ação:** adicionar RD-009-A: "Chamado aberto às 17h: triagem concluída até 09h do próximo dia útil."

---

#### AMB-007 — Solução: definição objetiva de "mesmo problema" + confiança na declaração do atendente

> Dois chamados são "o mesmo problema" quando compartilham o mesmo número de CT-e **ou** a mesma categoria de chamado E o mesmo número de contrato, abertos nas últimas 24 horas.

O assistente classifica como incidente crítico com base na **declaração do atendente** e o orienta a confirmar antes de escalar. Adicionar à linguagem ubíqua e ao RF-009.

---

#### AMB-008 — Solução: hierarquia de cálculo para devolução parcial

1. Proporcional ao **valor declarado por volume** no CT-e (quando discriminado)
2. Proporcional ao **peso por volume** (quando só peso discriminado)
3. Nenhum dos dois discriminado → encaminhar ao Operações para apuração manual

**Ação:** atualizar linguagem ubíqua, RD-009-B e adicionar CA-FUNC-U-02-C.

---

#### AMB-009 — Solução: substituir janela temporal por comparação de timestamps

O índice deve armazenar `document_updated_at` e `indexed_at` por chunk. Defasagem = `document_updated_at(fonte primária) − indexed_at(chunk FAQ)`. Parâmetro `faq_staleness_threshold_hours = 24` configurável pelo administrador sem redeploy. Reescrever RF-008.

---

#### AMB-010 — Solução: fila de revisão unificada com duas origens

```
Fila de revisão do FAQ
  ├── Entrada automática (RF-008): tipo=conflito_automático, prioridade ALTA
  └── Entrada feedback (RF-010): tipo=feedback_atendente, prioridade NORMAL
Consumidor: painel de administração com filtros por tipo e documento
```

Registrar como ADR-010 e adicionar RF-010-B.

---

### 🟢 Menores

#### AMB-011 — Solução: pergunta proativa sobre documentação de lacre

Quando atendente menciona lacre violado, antes de classificar como inelegível, o assistente pergunta: *"Houve documentação no ato da entrega com assinatura do motorista e do recebedor?"* Adicionar cenário U-05-B e atualizar RF-002.

#### AMB-012 — Solução: adotar "v2.0" como string canônica

CA-FUNC-001 e o mockup já usam "v2.0". Inconsistência está nos documentos de análise. Padronizar metadado `document_version` no índice como "v2.0" / "v1.0".

#### AMB-013 — Solução: substituir placeholder por caminho real com nota bloqueante

Usar `\\novatech-fs\operacoes\manuais\` como fallback provisório. Criar tarefa de dependência bloqueante para go-live: confirmar caminho com TI e verificar ponto de falha único. Registrar como dívida técnica.

---

### Resumo das ações por tipo

| Tipo de ação | Qtd |
|---|---|
| Novo ADR (ADR-007 a ADR-010) | 4 |
| Atualização de RD existente | 6 |
| Novo RF ou sub-requisito | 4 |
| Novo cenário de aceite | 7 |
| Atualização de linguagem ubíqua | 3 |
| Tarefa de dependência bloqueante | 2 |

---

## Turno 3 — Geração dos artefatos atualizados

**Usuário:**
> Vou aceitar suas sugestões, realize as alterações e gere arquivos atualizados.

**Assistente:**

Todos os três artefatos foram atualizados para a versão 1.1 com as seguintes alterações:

---

### requirements.md v1.1 — Alterações

| Artefato | Alteração | AMB |
|---|---|---|
| RF-002 | Lacre violado adicionado como dado contextual obrigatório | AMB-011 |
| RF-002-B | Novo — gap total por lista hardcoded como barreira primária antes da busca vetorial | AMB-003 |
| RF-005 | Reposicionado como barreira secundária por score, com pré-condição explícita | AMB-003 |
| RF-008 | Substituída "janela de 24h" por mecanismo `document_updated_at` vs `indexed_at`; parâmetro `faq_staleness_threshold_hours` configurável | AMB-009 |
| RF-009 | Relógio contínuo estendido a todos os tiers; critério 3 com definição de "mesmo problema" e instrução de confirmação | AMB-002, AMB-007 |
| RF-010-B | Novo — fila de revisão unificada com duas origens e prioridade | AMB-010 |
| RF-012 | Placeholder substituído por caminho real com nota de dependência bloqueante | AMB-013 |
| RD-001 | Threshold `>` substituído por `>=` (inclusivo) | AMB-001 |
| RD-003-A | Novo — fuso UTC-3, data sem hora = 00h01 | AMB-005 |
| RD-007-A | Novo — fórmula explícita de desconto sobre o multiplicador com exemplo canônico | AMB-004 |
| RD-009-A | Novo — triagem com pausa de horário comercial | AMB-006 |
| RD-009-B | Novo — hierarquia de cálculo da devolução parcial | AMB-008 |
| RD-015 | Relógio contínuo para todos os tiers em incidente crítico | AMB-002 |
| ADR-007 | Novo — threshold inclusivo >= 500 kg | AMB-001 |
| ADR-008 | Novo — relógio contínuo em incidente crítico para todos os tiers | AMB-002 |
| ADR-009 | Novo — arquitetura de gap total: lista hardcoded como barreira primária | AMB-003 |
| ADR-010 | Novo — fila de revisão unificada do FAQ | AMB-010 |
| Casos Ambíguos | 7 originais + 5 novos; 7 marcados como ✅ RESOLVIDO | Múltiplas |
| Cenários de aceite | CA-FUNC-001-B, CA-FUNC-001-C, CA-FUNC-006-B, U-02-B, U-02-C, U-05-B, U-06-B adicionados | Múltiplas |

---

### bounded-contexts-anexo-a.md v1.1 — Alterações

| Contexto | Alteração | AMB |
|---|---|---|
| BC-A1 | Threshold de frete especial corrigido para inclusivo | AMB-001 |
| BC-A2 | Nota de relógio contínuo para todos os tiers em incidente crítico | AMB-002 |
| BC-A3 | Triagem com pausa explícita de horário comercial | AMB-006 |
| BC-A3 | Lacre violado: comportamento proativo do assistente para verificar documentação | AMB-011 |
| BC-A3 | Devolução parcial com hierarquia valor > peso > Operações | AMB-008 |
| BC-A6 | Relógio de SLA reescrito: chamados gerais pausam; incidentes críticos contínuos para todos os tiers | AMB-002 |
| Tabela C/G | C1 e C2 marcados como resolvidos; G6 e G7 adicionados como gaps fechados | AMB-002, AMB-006 |

---

### linguagem-ubiqua-anexo-a.md v1.1 — Alterações

| Termo | Alteração | AMB |
|---|---|---|
| Frete especial | Threshold inclusivo; exemplo explícito de 500 kg | AMB-001 |
| Desconto por volume | Fórmula `VB × (MR × (1−d%)) × FP` com exemplo numérico canônico | AMB-004 |
| Devolução parcial | Hierarquia: valor declarado > peso > Operações | AMB-008 |
| Incidente crítico | Definição de "mesmo problema" com identificadores objetivos; instrução de confirmação ao atendente | AMB-007 |
| Relógio de SLA | Contínuo para todos os tiers em incidente crítico (não apenas Gold) | AMB-002 |
| Disposições transitórias | Fuso UTC-3 canônico; data sem hora = 00h01 | AMB-005 |

---

*Fim da sessão — artefatos gerados: requirements.md v1.1 · bounded-contexts-anexo-a.md v1.1 · linguagem-ubiqua-anexo-a.md v1.1*
