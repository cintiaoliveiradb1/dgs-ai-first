# Mapa de Temas e Gaps — Assistente IA Logística

> **Contexto:** Análise de cobertura documental para criação de assistente de IA voltado ao time de atendimento de empresa de logística. O assistente deve centralizar respostas hoje dispersas em múltiplos documentos.

---

## Resumo executivo

| Indicador | Valor |
|---|---|
| Documentos analisados | 5 |
| Temas mapeados | 8 |
| Gaps hipotéticos identificados | 7 |
| Conflitos / sobreposições | 3 |
| Perguntas frequentes sem cobertura oficial | 38 de 47 |

---

## Documentos fonte

| ID | Documento | Tipo | Versão | Data | Status |
|---|---|---|---|---|---|
| D1 | FAQ-Atendimento | Informal / colaborativo | Sem versão | ~2 anos | ⚠️ Sem validação de Compliance |
| D2 | POL-001 — Política de Devolução | Normativo oficial | v3.1 | Jan/2024 | ✅ Vigente |
| D3 | PROC-042 — Frete Especial | Procedimento | v1 | Mar/2023 | ⚠️ Obsoleto sem marcação formal |
| D4 | PROC-042-v2 — Frete Especial Revisado | Procedimento | v2 | Nov/2023 | ✅ Vigente (conflito com v1) |
| D5 | SLA-2024 — Tabela de SLA por Cliente | Contratual | — | Jan/2024 | ✅ Vigente |

---

## Cobertura por tema

### ✅ Coberto — Devolução de mercadorias
- **Documentos:** D1 (FAQ), D2 (POL-001)
- **O que está documentado:** prazo de 7 dias úteis, categorias não elegíveis (cargas perigosas, refrigeradas com cadeia de frio rompida, lacre violado), procedimento via Portal do Cliente, responsabilidade de custo (gratuito para erro da NovaTech; cliente paga em desistência).
- **Observação:** D1 pode conter informações divergentes de D2 por falta de validação. Recomenda-se priorizar D2 como fonte canônica.

---

### ⚠️ Conflito — Cálculo de frete especial
- **Documentos:** D3 (PROC-042 v1), D4 (PROC-042-v2)
- **O que está documentado:** fórmula base (Valor base × Multiplicador regional × Fator de peso), multiplicadores por região, fatores de peso, prazo adicional de entrega, aprovação gerencial para cargas acima de 5.000 kg.
- **Conflito identificado:**

| Parâmetro | v1 (mar/2023) | v2 (nov/2023) |
|---|---|---|
| Multiplicador Norte | 1,6 | 1,8 |
| Prazo adicional | +2 dias úteis | +3 dias úteis |
| Desconto por volume | Não previsto | A partir de 8 fretes/mês |
| Disposição transitória | Não aplicável | Chamados anteriores a dez/2023 seguem v1 |

- **Risco:** ambas as versões coexistem no SharePoint sem hierarquia clara. O assistente pode retornar valores incorretos se indexar os dois documentos.

---

### ✅ Coberto — SLA e tiers de clientes
- **Documento:** D5 (SLA-2024)
- **O que está documentado:**

| Tier | Resposta | Resolução | Incidente crítico (resposta) |
|---|---|---|---|
| Gold | 2h | 24h | 30 min |
| Silver | 4h | 48h | 1h |
| Standard | 8h | 72h | 2h |

- Critérios de incidente crítico definidos.
- Penalidades por descumprimento: créditos a partir da segunda violação mensal.
- Medição via Azure DevOps.

---

### 🟡 Parcial — Seguro de carga
- **Documento:** D1 (FAQ apenas)
- **O que existe:** menção superficial no FAQ sem documento normativo próprio.
- **O que falta:** apólice, coberturas, valor mínimo segurado, prazo e canal para acionamento de sinistro, responsabilidade em caso de cobertura negada.

---

### 🟡 Parcial — Rastreamento de cargas
- **Documento:** D1 (FAQ apenas)
- **O que existe:** tema citado no FAQ sem profundidade técnica.
- **O que falta:** plataforma ou sistema utilizado, como acessar, prazo de atualização de status, procedimento em caso de falha ou ausência de rastreio.

---

### 🟡 Parcial — Incidentes operacionais
- **Documentos:** D1 (FAQ), D5 (SLA-2024)
- **O que existe:** critérios de incidente crítico no SLA-2024; menção no FAQ.
- **O que falta:** fluxo de escalonamento interno, procedimento de comunicação proativa ao cliente, registro e acompanhamento de ocorrência.

---

### 🟡 Parcial — Descontos e política comercial
- **Documentos:** D1 (FAQ), D4 (PROC-042-v2)
- **O que existe:** menção a descontos no FAQ; desconto automático por volume (≥8 fretes/mês) no PROC-042-v2.
- **O que falta:** política comercial consolidada com tabela de descontos, alçadas de aprovação, critérios de elegibilidade e exceções.

---

### 🔴 Ausente — Cargas perigosas (procedimento completo)
- **Documentos:** menções em D1 e D2 apenas como exclusão
- **O que existe:** exclusão da devolução padrão em POL-001 e no FAQ.
- **O que falta:** fluxo completo de transporte, requisitos legais e regulatórios, embalagem, armazenagem, responsabilidades operacionais e o que comunicar ao cliente.

---

## Hipóteses de gaps

### G1 — Procedimento completo para cargas perigosas
- **Criticidade:** Alta
- **Hipótese:** Dois documentos mencionam cargas perigosas apenas para excluí-las do fluxo padrão. Atendentes ficam sem resposta para perguntas básicas sobre transporte, armazenagem e regulamentação desse tipo de carga.
- **Impacto no assistente:** Incapacidade de responder perguntas frequentes sobre o tema; risco de orientação incorreta.

### G2 — Política de seguro de carga
- **Criticidade:** Alta
- **Hipótese:** A menção no FAQ pode estar desatualizada. Ausência de apólice, cobertura mínima, prazo de acionamento e canais de sinistro.
- **Impacto no assistente:** Respostas incompletas ou potencialmente incorretas sobre um tema de alto impacto financeiro para o cliente.

### G3 — Manual técnico de rastreamento
- **Criticidade:** Média-Alta
- **Hipótese:** O rastreamento é uma das consultas mais frequentes em operações logísticas. Sem documentação técnica, o assistente não consegue orientar sobre como acessar, interpretar ou resolver problemas.
- **Impacto no assistente:** Impossibilidade de responder dúvidas operacionais do dia a dia.

### G4 — Procedimento de escalonamento de incidentes
- **Criticidade:** Média-Alta
- **Hipótese:** SLA-2024 define critérios e penalidades, mas não há fluxo de comunicação proativa ao cliente nem árvore de escalonamento interno.
- **Impacto no assistente:** Atendente sem orientação sobre o que fazer após identificar um incidente crítico.

### G5 — Política comercial e tabela de descontos unificada
- **Criticidade:** Média
- **Hipótese:** Descontos aparecem fragmentados no FAQ e no PROC-042-v2. Ausência de política consolidada com alçadas de aprovação e critérios claros.
- **Impacto no assistente:** Respostas inconsistentes sobre elegibilidade a descontos.

### G6 — Cargas refrigeradas e cadeia de frio
- **Criticidade:** Média
- **Hipótese:** POL-001 cita "cadeia de frio rompida" como exclusão de devolução, mas não há documentação sobre procedimentos de conservação, responsabilidades ou compensação ao cliente.
- **Impacto no assistente:** Sem resposta para um segmento específico de cliente com alta sensibilidade operacional.

### G7 — 38 perguntas frequentes sem cobertura oficial
- **Criticidade:** Alta
- **Hipótese:** O FAQ cobre apenas 9 das 47 perguntas levantadas pelo time de atendimento. As 38 restantes não têm resposta documentada em nenhum dos 5 documentos analisados.
- **Impacto no assistente:** Cobertura de apenas ~19% das dúvidas reais do time, tornando o assistente ineficaz para a maioria das consultas.

---

## Conflitos e riscos de qualidade

### C1 — PROC-042 v1 vs v2 sem hierarquia clara
- **Tipo:** Conflito de versão
- **Descrição:** As duas versões do procedimento de frete especial coexistem no SharePoint sem marcação de obsolescência. Multiplicadores regionais, fatores de peso e prazos de entrega divergem entre as versões.
- **Ação recomendada:** Deprecar formalmente o PROC-042-v1 antes da indexação. Sinalizar no documento que a v2 é a versão vigente.

### C2 — FAQ sem validação de Compliance ou Operações
- **Tipo:** Risco de qualidade
- **Descrição:** O FAQ tem dois anos de manutenção colaborativa sem processo de revisão formal. Pode conter informações desatualizadas ou diretamente conflitantes com POL-001 e SLA-2024.
- **Ação recomendada:** Submeter o FAQ a revisão de Compliance e Operações antes de incluí-lo no corpus do assistente. Tratar como fonte de menor confiança até validação.

### R1 — Disposição transitória do PROC-042-v2 não tratada
- **Tipo:** Risco operacional
- **Descrição:** Chamados abertos antes de dezembro/2023 devem seguir as regras da v1. Sem controle claro, o assistente pode aplicar a versão errada para casos históricos ainda em aberto.
- **Ação recomendada:** Definir critério de data no assistente para aplicação da versão correta, ou excluir casos históricos do escopo inicial.

---

## Recomendações para o roadmap

| Prioridade | Ação | Responsável sugerido |
|---|---|---|
| 🔴 Urgente | Deprecar formalmente PROC-042-v1 no SharePoint | Operações |
| 🔴 Urgente | Validar FAQ com Compliance e Operações | Compliance |
| 🔴 Urgente | Mapear e documentar as 38 perguntas sem cobertura | Atendimento + Operações |
| 🟠 Alta | Criar documentação para cargas perigosas (G1) | Operações / Jurídico |
| 🟠 Alta | Criar política de seguro de carga (G2) | Comercial / Jurídico |
| 🟡 Média | Criar manual técnico de rastreamento (G3) | TI / Operações |
| 🟡 Média | Criar procedimento de escalonamento de incidentes (G4) | Operações |
| 🟡 Média | Consolidar política comercial e descontos (G5) | Comercial |
| 🟢 Baixa | Documentar procedimentos de cadeia de frio (G6) | Operações |

---

*Análise gerada em maio/2025 com base nos documentos fornecidos. Recomenda-se revisão periódica à medida que novos documentos forem incorporados ao corpus do assistente.*
