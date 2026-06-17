# Recorte de Domínio — Projeto Assistente IA NovaTech
## Bounded Contexts — Fase de Análise de Negócio (SDD)

**Autor:** Análise de Negócio Sênior — Specification Driven Development  
**Baseado em:** Anexo A (documentação NovaTech), Mapa de Jornada do Atendente, Spec RAG v1.3  
**Versão:** 1.0  
**Data:** Junho/2025  

---

> **Nota metodológica:** Este recorte adota a técnica de Bounded Contexts do Domain-Driven Design (DDD) para delimitar fronteiras conceituais no domínio. Cada contexto encapsula um conjunto coeso de responsabilidades, regras e dados, com linguagem ubíqua própria. O objetivo é preparar o terreno para a especificação funcional e evitar acoplamento indevido entre subsistemas.

---

## Visão Geral dos Contextos Identificados

| # | Bounded Context | Responsabilidade Central |
|---|-----------------|--------------------------|
| 1 | Gestão do Corpus Normativo | Ciclo de vida dos documentos que alimentam a base de conhecimento |
| 2 | Recuperação e Composição de Respostas (RAG) | Busca semântica, resolução de conflitos e geração de resposta |
| 3 | Atendimento Assistido | Interação entre atendente e assistente durante a resolução de chamados |
| 4 | Gestão de Gaps e Fallback | Tratamento de consultas sem cobertura documental |
| 5 | Rastreabilidade e Auditoria | Registro, log e reprodutibilidade de cada interação |
| 6 | Feedback e Melhoria Contínua | Coleta e análise de avaliações para evoluir a base |
| 7 | Identidade e Controle de Acesso | Autenticação e autorização dos usuários do assistente |
| 8 | Domínio Logístico NovaTech | Regras de negócio do transporte — conhecimento do domínio de operações |

---

## Detalhamento dos Bounded Contexts

---

### BC-01 — Gestão do Corpus Normativo

#### Objetivo
Controlar o ciclo de vida dos documentos que compõem a base de conhecimento do assistente: ingestão, versionamento, gestão de status e descarte. É o guardião da confiabilidade da informação disponível para recuperação.

#### Está dentro do contexto
- Ingestão de documentos com metadados obrigatórios: `versão`, `data de emissão`, `status` (vigente / transição / obsoleto), `tipo` (normativo / procedimento / contratual / informal)
- Validação de pré-requisito de data em todo documento submetido — rejeição de documentos sem data declarada
- Atualização automática de status ao ingerir nova versão de um documento (novo documento entra como `vigente`; anterior transita para `transição` ou `obsoleto`)
- Gestão da janela de transição do PROC-042: chamados até 30/11/2023 usam v1 (`status=transição`); a partir de 01/12/2023 usam v2 (`status=vigente`)
- Transição automática de `transição` → `obsoleto` após a data definida no metadado `vigente_até`
- SLAs de ingestão: POL e SLA em até 4h úteis; PROC em até 4h úteis; FAQ em até 24h úteis
- Ingestão incremental (documento único sem reindexação completa)
- Log de cada operação de ingestão: nome, versão, horário de início/fim, número de chunks, status
- Alerta ao administrador em até 30 minutos em caso de falha de ingestão
- Tratamento de janela de inconsistência entre fonte primária (4h) e FAQ (24h): supressão do FAQ conflitante com sinalização ao atendente e marcação para revisão prioritária
- Gestão do roadmap de fontes futuras (PROC-043, tabela de fretes-base, política de sinistros etc.)

#### Está fora do contexto
- Cálculo de similaridade semântica ou ranqueamento de chunks (pertence ao BC-02)
- Exibição de respostas ao atendente (pertence ao BC-03)
- Decisão sobre qual versão de documento usar em função da data do chamado (pertence ao BC-02, que consome os metadados produzidos aqui)
- Controle de quem pode acessar ou publicar documentos — permissões de autoria (pertence ao BC-07)
- Redação ou validação do conteúdo dos documentos normativos (responsabilidade das áreas Operações, Compliance e Comercial)

#### Relacionamentos
- **Fornece para BC-02:** chunks indexados com metadados completos (tipo, versão, status, data) que alimentam o pipeline de recuperação
- **Fornece para BC-05:** logs de ingestão disponíveis no painel de administração para auditoria
- **Consome de BC-07:** somente usuários autorizados pelo Azure AD podem submeter documentos à base
- **Notifica BC-06:** sinaliza itens do FAQ marcados para revisão após detecção de inconsistência com fonte primária
- **Depende das áreas de negócio** (Operações, Compliance, Comercial) para publicação dos documentos que eliminam os gaps documentais identificados

---

### BC-02 — Recuperação e Composição de Respostas (RAG)

#### Objetivo
Receber a consulta do atendente, recuperar os trechos relevantes da base de conhecimento, resolver conflitos entre documentos e compor uma resposta estruturada, rastreável e ancorada na fonte normativa correta. É o núcleo inteligente do assistente.

#### Está dentro do contexto
- Cálculo de embedding e similaridade semântica para recuperação de chunks
- Aplicação do threshold mínimo de similaridade (0,75) — trechos abaixo do threshold são descartados
- Re-ranking pós-recuperação: em empate de score entre fonte primária e secundária, a primária prevalece; quando scores são claramente distintos, a relevância semântica prevalece
- Resolução de conflito entre documentos com mesmo tema: prioridade por `status=vigente`; em status iguais, prioridade por data de emissão mais recente
- Seleção de versão do PROC-042 baseada na data de abertura do chamado (consumindo metadados do BC-01):
  - Chamado até 30/11/2023 → PROC-042-v1
  - Chamado a partir de 01/12/2023 → PROC-042-v2
  - Data não informada → solicitar ao atendente antes de responder
- Solicitação ao atendente de dados faltantes para gap contextual (ex.: peso e região para frete especial; tier do cliente para SLA)
- Classificação da consulta em três categorias antes de responder:
  1. **Coberta pela base** — responde normalmente com citação de fonte
  2. **Gap documental** (tema logístico NovaTech sem cobertura formal) — mensagem de gap com indicação da área responsável
  3. **Fora do escopo** — mensagem diferenciada informando que o tema não pertence ao domínio logístico da NovaTech
- Sinalização explícita de conflito na resposta quando chunks contraditórios forem recuperados (indicando documentos divergentes, valor conflitante e regra de desempate aplicada)
- Identificação e sinalização de fonte informal (FAQ) com aviso de status não validado
- Composição da resposta em linguagem acessível, sem jargão interno
- Composição estruturada com: resposta direta, fonte normativa (documento + versão + data de vigência), valor calculado / prazo / passo a passo quando aplicável
- Geração de resposta em português brasileiro independentemente do idioma do documento-fonte
- Supressão de chunks do FAQ quando em conflito com fonte primária atualizada (dentro da janela de inconsistência)

#### Está fora do contexto
- Armazenamento e indexação de documentos (pertence ao BC-01)
- Exibição visual da resposta e interação com o atendente (pertence ao BC-03)
- Registro de logs de auditoria da interação (pertence ao BC-05)
- Coleta de feedback pós-resposta (pertence ao BC-06)
- Autenticação do usuário (pertence ao BC-07)
- Qualquer cálculo ou decisão que requeira acesso a sistemas transacionais da NovaTech (ex.: valor base da tabela mensal de fretes, tracking de carga, sistema de chamados Azure DevOps)

#### Relacionamentos
- **Consome de BC-01:** chunks indexados com metadados para recuperação e desambiguação
- **Fornece para BC-03:** resposta estruturada com fonte, trecho de suporte e classificação do resultado
- **Fornece para BC-04:** classificação de gap (tipo: total, parcial ou contextual) para acionamento do fluxo de fallback
- **Fornece para BC-05:** trechos recuperados com scores, documentos utilizados, versão aplicada e regras de desempate — tudo o que compõe o log de cada interação
- **Dependência técnica:** modelo LLM (DTP-002 — decisão técnica pendente de seleção e versionamento)
- **Dependência técnica:** estratégia de chunking (DTP-001 — pendente de definição de tamanho, sobreposição e tratamento especial de tabelas numéricas)

---

### BC-03 — Atendimento Assistido

#### Objetivo
Gerenciar a experiência de uso do assistente pelo atendente durante a resolução de chamados: entrada da consulta, exibição da resposta, ciclo de follow-up e registro da resolução. É a camada de interação humano-máquina.

#### Está dentro do contexto
- Recebimento da consulta do atendente em linguagem livre, sem exigência de formato específico
- Exibição da resposta estruturada gerada pelo BC-02, incluindo:
  - Resposta direta em linguagem acessível
  - Fonte normativa utilizada (documento + versão + seção + data)
  - Trecho do documento original em bloco destacado, com rótulo de localização
  - Resumo em linguagem simples quando o trecho original for extenso (> 150 palavras) ou muito técnico, seguido do trecho completo disponível para consulta
  - Link ou referência ao documento completo no SharePoint quando disponível
- Exibição de avisos de status: fonte informal (FAQ), conflito entre versões, descarte de FAQ por inconsistência
- Suporte ao ciclo de follow-up: o atendente pode fazer pergunta adicional ao assistente após receber a resposta (opção B — "Precisa de contexto")
- Acionamento do fluxo de fallback pelo atendente quando a resposta não atende (opção C)
- Suporte ao registro do chamado pelo atendente com a resolução obtida
- Exibição de mensagem de modo degradado em dois estados distintos:
  - Indisponibilidade por falha
  - Indisponibilidade por atualização planejada
- Indicação do local onde o atendente pode consultar a documentação durante indisponibilidade
- Tempo de resposta: retorno em até 5 segundos para 95% das consultas
- Disponibilidade: 99% de uptime no horário comercial (08h–18h, dias úteis)

#### Está fora do contexto
- Lógica de recuperação semântica e composição da resposta (pertence ao BC-02)
- Gerenciamento do ciclo de vida dos documentos (pertence ao BC-01)
- Registro de logs de auditoria (pertence ao BC-05) — o BC-03 captura a interação; o BC-05 a armazena
- Coleta e análise de feedback (pertence ao BC-06) — o BC-03 apresenta o prompt de feedback; o BC-06 processa os dados
- Comunicação direta com o cliente final — o atendente permanece responsável pela comunicação com o cliente
- Tomada de decisão sobre a resolução do chamado — o assistente não decide; o atendente decide

#### Relacionamentos
- **Consome de BC-02:** resposta estruturada, classificação do resultado, avisos de fonte e conflito
- **Consome de BC-04:** mensagem de fallback estruturada com indicação de responsável e canal de escalada
- **Dispara BC-06:** ao encerrar ou escalar o chamado, aciona o prompt de feedback ao atendente
- **Informa BC-05:** cada interação gera dados (pergunta, resposta exibida, ações do atendente) para o log
- **Consome de BC-07:** sessão autenticada via Azure AD para identificar o atendente

---

### BC-04 — Gestão de Gaps e Fallback

#### Objetivo
Tratar as consultas para as quais o assistente não possui documentação suficiente, entregando ao atendente uma mensagem estruturada com o correto direcionamento para escalada, sem jamais fabricar respostas ou usar inferência não validada.

#### Está dentro do contexto
- Classificação e tratamento dos três tipos de gap:
  - **Gap total:** tópico completamente ausente da base (ex.: política de carga danificada, seguro de carga, processo da Gestão de Riscos)
  - **Gap parcial:** tópico coberto apenas no FAQ informal (ex.: carga perigosa com frete expresso)
  - **Gap contextual:** informação existe na base, mas falta dado do atendente (ex.: frete especial sem peso ou região informados)
- Gatilhos de entrada do fluxo de fallback:
  - Tema sem documentação oficial indexada
  - Consulta ambígua que não se encaixa em nenhum tema mapeado
  - Parâmetros fora das faixas documentadas (ex.: carga acima de 5.000 kg)
  - Atendente sinaliza que a resposta não atende
- Mensagem de fallback estruturada por tipo de gap, contendo:
  - Informação de que o tema não tem documentação oficial indexada
  - Responsável interno correto para escalada (ex.: Comercial, Compliance, Jurídico, Gerente regional)
  - Canal de contato sugerido (área, e-mail ou ramal documentado)
- Mapeamento de fallbacks predefinidos por tema:
  - Seguro de carga → Comercial
  - Carga perigosa + frete expresso → Compliance (PROC-043 em revisão)
  - Sinistro / carga danificada → Jurídico (e-mail: sinistros@novatech.com.br)
  - Frete especial acima de 5.000 kg → Gerente regional de operações
- Restrição absoluta: o assistente não tenta responder com base em FAQ não validado ou inferência quando o gap é total
- Registro do tipo de gap em log interno para alimentar o processo de melhoria contínua

#### Está fora do contexto
- Resolução do gap em si (é responsabilidade das áreas de negócio — Operações, Compliance, Comercial)
- Criação dos documentos que eliminarão os gaps (responsabilidade das áreas proprietárias dos processos)
- Processo de escalada efetivo — o atendente decide como escalar (internamente, consultar supervisor ou registrar sem resolução imediata)
- Análise agregada dos gaps recorrentes para priorização de documentação (pertence ao BC-06)

#### Relacionamentos
- **Recebe de BC-02:** classificação do tipo de gap identificado na consulta
- **Fornece para BC-03:** mensagem de fallback estruturada para exibição ao atendente
- **Alimenta BC-06:** log de gap (tema, tipo, data) para o painel de gaps recorrentes e priorização de documentação futura
- **Alimenta BC-05:** registro do tipo de gap vinculado à interação para auditoria

---

### BC-05 — Rastreabilidade e Auditoria

#### Objetivo
Garantir que cada interação seja completamente registrada, permitindo reconstrução retroativa de qualquer resposta fornecida, suportando auditorias de chamados reabertos e cumprindo obrigações de conformidade (LGPD).

#### Está dentro do contexto
- Log completo por interação: timestamp, ID do atendente, pergunta formulada, chunks recuperados (com scores de similaridade), resposta gerada, fontes citadas (documento, versão, seção)
- Log de gaps: data, pergunta, tipo de gap (total / parcial / contextual)
- Rastreabilidade retroativa: dado o ID de um chamado reaberto, é possível identificar exatamente qual trecho e versão de documento embasou a resposta original
- Retenção de logs por no mínimo 90 dias (prazo máximo a ser definido pelo DPO)
- Controle de acesso restrito: somente a equipe de segurança da NovaTech pode acessar os logs — administradores e atendentes não têm acesso direto
- Conformidade LGPD: dados pessoais (nome/ID do atendente) tratados com finalidade declarada, prazo de retenção definido e procedimento de descarte seguro documentado
- Log de ingestão de documentos (operado pelo BC-01, armazenado e auditável aqui): nome do arquivo, versão, data/hora, chunks gerados, status da operação

#### Está fora do contexto
- Análise de qualidade ou tendências sobre os logs (pertence ao BC-06, que consome dados agregados)
- Exibição de logs ao atendente ou ao administrador do assistente (acesso restrito à equipe de segurança)
- Qualquer processamento em tempo real dos logs para alterar o comportamento do assistente (logs são imutáveis e passivos)

#### Relacionamentos
- **Recebe de BC-02:** dados de recuperação (chunks, scores, documentos, regras de desempate aplicadas)
- **Recebe de BC-03:** dados da interação (pergunta exibida, ações do atendente, resposta mostrada)
- **Recebe de BC-04:** tipo de gap classificado
- **Recebe de BC-01:** logs de ingestão de documentos
- **Fornece para BC-06:** dados agregados de gaps e feedbacks (via painel) para análise de qualidade
- **Fornece para auditoria externa:** evidência auditável de toda resposta para conformidade e resolução de disputas

---

### BC-06 — Feedback e Melhoria Contínua

#### Objetivo
Capturar a avaliação do atendente sobre a utilidade de cada resposta, agregar os dados de qualidade e gaps, e disponibilizá-los para a revisão periódica da base de conhecimento — fechando o ciclo de aprendizado do sistema.

#### Está dentro do contexto
- Prompt de feedback pós-atendimento (após encerramento ou escalada do chamado): não obrigatório e não bloqueia o fluxo
- Três opções de avaliação: Sim (útil) / Parcialmente / Não (incorreta ou incompleta)
- Campo aberto opcional para avaliação negativa ou parcial (máx. 280 caracteres): "O que estava incorreto ou faltando?"
- Registro do feedback vinculado à interação (tema consultado, item, descrição quando fornecida)
- Painel de qualidade da base com:
  - Taxa de utilidade por tema (% úteis / total consultado)
  - Temas com mais feedbacks negativos → candidatos a revisão
  - Fallbacks mais acionados → candidatos a nova documentação
  - Correlação com reabertura de chamados (meta: reduzir de 22% para < 10%)
- Painel de gaps recorrentes (alimentado pelo BC-04): base para priorização de documentação futura
- Sinalização de itens do FAQ marcados para revisão (após inconsistência detectada pelo BC-01)
- Revisão periódica da base: sugerido quinzenal nas primeiras 8 semanas; área especialista analisa feedbacks, aciona áreas para atualizar documentação e submete novas versões ao BC-01
- Métricas de acompanhamento do projeto:
  - Reabertura de chamados em 48h: 22% → meta < 10%
  - Fontes/chamado em frete especial: 6,2 → meta ≤ 2
  - Fontes/chamado geral: 4,1 → meta ≤ 1,5
  - Utilidade das respostas: — → meta ≥ 80%
  - Resolvidos sem escalada (temas cobertos): — → meta ≥ 70%

#### Está fora do contexto
- Execução das atualizações de documentos (responsabilidade das áreas de negócio; a submissão ao pipeline pertence ao BC-01)
- Armazenamento bruto dos logs de interação (pertence ao BC-05)
- Coleta do feedback em si durante a interação (o prompt é exibido pelo BC-03; aqui o contexto trata do processamento e análise)
- Definição de qual documento substituirá qual — essa é decisão de negócio, não do assistente

#### Relacionamentos
- **Recebe de BC-03:** seleção do atendente no prompt de feedback e texto livre opcional
- **Recebe de BC-04:** log de gaps por tipo e tema para o painel de gaps recorrentes
- **Recebe de BC-05:** dados históricos de interações para correlação com reabertura de chamados
- **Alimenta BC-01:** solicitações de revisão e novas versões de documentos (via processo humano das áreas especialistas)
- **Alimenta BC-01:** itens do FAQ marcados para revisão após janela de inconsistência

---

### BC-07 — Identidade e Controle de Acesso

#### Objetivo
Garantir que apenas usuários autenticados e autorizados da NovaTech possam utilizar o assistente, integrando-se ao Azure Active Directory corporativo como fonte única de verdade de identidade.

#### Está dentro do contexto
- Autenticação via Azure Active Directory (Azure AD) como requisito de acesso
- Bloqueio de tentativas de acesso sem credencial Azure válida
- Verificação de perfil autorizado para uso do assistente
- Propagação imediata de revogações de acesso: remoção do perfil no Azure reflete no assistente sem ação adicional
- Identificação do atendente em cada sessão (ID usado nos logs do BC-05)

#### Está fora do contexto
- Gestão de usuários, perfis e grupos no Azure AD (responsabilidade do time de TI/segurança da NovaTech)
- Controle de quem pode publicar documentos na base de conhecimento — esse é um controle de processo, não de sessão do assistente
- Auditoria de acessos (pertence ao BC-05)

#### Relacionamentos
- **Fornece para BC-03:** sessão autenticada que habilita o uso do assistente
- **Fornece para BC-05:** ID do atendente autenticado para registro nos logs
- **Depende de:** Azure Active Directory da NovaTech (sistema externo ao escopo do assistente)

---

### BC-08 — Domínio Logístico NovaTech

#### Objetivo
Representar o conhecimento de negócio do domínio de transporte e logística da NovaTech — as regras, políticas e procedimentos que o assistente deve conhecer e aplicar. Este contexto não é um subsistema de software, mas o modelo conceitual do negócio que fundamenta todos os outros contextos.

#### Está dentro do contexto

**Subdomínio: Cálculo de Frete Especial (PROC-042-v2, vigente para chamados a partir de 01/12/2023)**
- Fórmula: Valor base × Multiplicador regional × Fator de peso
- Faixas de peso: 500–1.000 kg (fator 1,0); 1.001–3.000 kg (fator 1,15); acima de 3.000 kg (fator 1,4)
- Multiplicadores regionais: Sul 1,3 | Sudeste 1,1 | Centro-Oeste 1,4 | Nordeste 1,5 | Norte 1,8
- Prazo adicional: +3 dias úteis sobre o prazo padrão da rota
- Regra de desconto por volume: ≥ 8 fretes/mês → 5% sobre multiplicador regional; ≥ 15 fretes/mês → 10%; acima disso → aprovação da Diretoria Comercial
- Cargas acima de 5.000 kg: requerem aprovação prévia do gerente regional de operações
- Regra de transição: chamados abertos até 30/11/2023 aplicam PROC-042-v1 (multiplicadores e fatores distintos, +2 dias úteis)

**Subdomínio: Política de Devolução (POL-001 v3.1)**
- Prazo geral: 7 dias úteis após confirmação de recebimento no tracking
- Categorias não elegíveis para devolução padrão: cargas perigosas classes 1–6 ANTT, cargas refrigeradas com cadeia de frio rompida, cargas com lacre violado (salvo documentação no ato)
- Casos não elegíveis → Gestão de Riscos (ramal 4500)
- Procedimento: abertura no Portal do Cliente com CT-e, fotos (mín. 3) e motivo; triagem em 4h úteis; coleta reversa agendada em 2 dias úteis após aprovação; reembolso em 5 dias úteis após recebimento no CD
- Responsabilidade de custo: defeito da NovaTech → sem custo para cliente; desistência do cliente → frete reverso por conta do cliente; prazo expirado → não elegível, encaminhar ao Comercial
- Devoluções parciais: por volume, proporcional ao peso/valor no CT-e

**Subdomínio: SLA por Tier de Cliente (SLA-2024)**
- Tiers: Gold (contrato > R$ 500k/ano OU > 200 ops/mês), Silver (R$ 100k–500k OU 50–200 ops/mês), Standard (demais)
- Não existe tier Platinum — clientes que alegam esse tier devem ser orientados
- SLAs de resposta: Gold 2h / Silver 4h / Standard 8h (chamados gerais); Gold 30min / Silver 1h / Standard 2h (incidentes críticos)
- SLAs de resolução: Gold 24h / Silver 48h / Standard 72h (chamados gerais); Gold 4h / Silver 8h / Standard 24h (críticos)
- Incidente crítico: carga > R$100k com status desconhecido há > 6h; carga perigosa com irregularidade; > 5 chamados do mesmo cliente sobre o mesmo problema em 24h; risco a pessoas
- Penalidades: 1ª violação → registro interno; 2ª → crédito de 5%; 3ª ou mais → crédito de 10% + reunião obrigatória

**Gaps documentais identificados (sem cobertura formal)**
- Política de carga danificada em trânsito (existe prática informal no FAQ-Item 38, mas sem POL ou PROC formal)
- Política formal de seguro de carga (existe orientação informal no FAQ-Item 22, mas sem documento oficial)
- Frete padrão para cargas abaixo de 500 kg (tabela base não indexada)
- Procedimento formal da Gestão de Riscos para cargas perigosas devolvidas (ramal 4500 documentado, mas sem PROC)
- Frete de cargas perigosas (PROC-043 em revisão pelo Compliance)

**Conflitos documentais identificados**
- PROC-042-v1 vs PROC-042-v2: multiplicadores regionais distintos, fatores de peso distintos, prazo adicional distinto (+2 vs +3 dias); coexistem no SharePoint sem hierarquia formal declarada — resolvido pela regra de transição da seção 5 do PROC-042-v2

#### Está fora do contexto
- Implementação das regras em software (responsabilidade dos contextos técnicos BC-01 a BC-07)
- Tabela mensal de fretes-base (arquivo externo no servidor de rede — não indexado na base inicial)
- Negociação de contratos, descontos especiais e condições não previstas nas políticas formais (responsabilidade do Comercial)
- Seguro de carga (lacuna documental — responsabilidade do Comercial cobrir)

#### Relacionamentos
- **Fundamenta BC-02:** é o modelo conceitual que o pipeline de RAG operacionaliza
- **Fundamenta BC-04:** é a lista de temas cobertos e gaps que define os comportamentos de fallback
- **Alimenta BC-01:** os documentos normativos deste subdomínio (POL-001, PROC-042-v2, SLA-2024) são as fontes primárias indexadas
- **Interface com sistemas externos:** Portal do Cliente (abertura de chamados de devolução), Azure DevOps (sistema de chamados e medição de SLA), sistema de tracking da NovaTech, servidor de rede (tabela de fretes-base)

---

## Mapa de Relacionamentos entre Contextos

```
[BC-07 Identidade]
        │ autenticação
        ▼
[BC-03 Atendimento Assistido] ◄──────────────────────────────────────────────────────┐
        │ consulta                                                                     │
        ▼                                                                             │
[BC-02 Recuperação e Composição RAG]                                                  │
        │                    │                                                        │
   chunks com           gap identificado                                              │
   metadados               │                                                          │
        │                   ▼                                                         │
[BC-01 Gestão do Corpus]  [BC-04 Gestão de Gaps]                                      │
        │                   │                                                         │
   logs de ingestão     log de gap                                                    │
        │                   │                                                         │
        ▼                   ▼                                                         │
[BC-05 Rastreabilidade e Auditoria]                                                   │
        │                                                                             │
   dados agregados                                                                    │
        ▼                                                                             │
[BC-06 Feedback e Melhoria Contínua] ──── solicitações de revisão ──► [BC-01]         │
        │                                                                             │
  prompt de feedback ──────────────────────────────────────────────────────────────► │
                                                                  (exibido no BC-03)

[BC-08 Domínio Logístico] ──► fundamenta conceitualmente todos os demais contextos
```

---

## Decisões de Modelagem e Pontos de Atenção

**1. Separação BC-02 e BC-03**
A separação entre o motor de recuperação (BC-02) e a camada de interação (BC-03) é intencional. Permite que a interface evolua (ex.: nova versão da UI, integração com CRM) sem impactar a lógica de recuperação, e vice-versa.

**2. BC-08 como modelo conceitual, não subsistema**
O Domínio Logístico não é implementado como serviço: ele se materializa como documentos indexados no BC-01 e como regras interpretadas pelo BC-02. Tratá-lo como contexto separado é uma decisão analítica para tornar explícitas as regras de negócio que precisarão estar representadas na especificação funcional.

**3. Risco: conflito PROC-042-v1 vs v2**
Ambas as versões coexistem no SharePoint sem hierarquia formal. O pipeline resolve isso via metadados (BC-01) e regra de data (BC-02), mas a raiz do problema — ausência de arquivamento formal da v1 — permanece como dívida organizacional. Recomenda-se formalizar o arquivamento com as áreas de Operações e Comercial antes do go-live.

**4. Risco: gaps documentais com impacto direto em reabertura**
Os quatro gaps identificados (carga danificada, seguro de carga, frete padrão, Gestão de Riscos) concentram parcela significativa dos 22% de reaberturas. A criação dos documentos formais por Operações, Compliance e Comercial é pré-condição para que o assistente atinja a meta de < 10%.

**5. Decisões técnicas pendentes que afetam BC-02**
DTP-001 (chunking de tabelas numéricas) e DTP-002 (seleção de modelo LLM) devem ser resolvidos antes do início do desenvolvimento, pois impactam diretamente a precisão das respostas sobre frete especial e SLA — os dois temas de maior volume de consultas.

---

*Recorte de Domínio v1.0 — NovaTech Logística — Confidencial*
