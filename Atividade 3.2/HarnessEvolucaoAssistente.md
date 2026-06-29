# Harness de Evolução do Assistente de Atendimento
## NovaTech Logística

| Campo | Valor |
|-------|-------|
| Versão | 1.0 |
| Data | Junho/2025 |
| Baseado em | Guardrails Operacionais v3.0; Avaliação de Respostas (Jun/2025) |
| Classificação | Documento operacional — uso obrigatório em todo ciclo de mudança pós go-live |

---

## Visão Geral

Este harness define como o assistente evolui sem degradar. Ele cobre três eixos:

1. **Processo de feedback** — como sinais do uso real viram melhorias concretas.
2. **Regression testing** — como verificar que uma mudança não quebrou o que funcionava.
3. **Human-in-the-loop** — quais mudanças exigem aprovação humana antes de ir a produção, e de quem.

O harness preserva como restrição inegociável os 24 guardrails do documento operacional v3.0. Qualquer mudança que viole, enfraqueça ou contorne um guardrail — mesmo que melhore métricas de qualidade — é bloqueada até revisão humana.

---

## 1. Processo de Feedback

### 1.1 Fontes de sinal

O harness reconhece três fontes de sinal de qualidade, em ordem de prioridade operacional:

| Fonte | Mecanismo | Guardrail associado |
|-------|-----------|---------------------|
| Feedback explícito do atendente | Botões Útil / Incorreta / Incompleta + campo de texto livre | GR-D-010 |
| Log de interação estruturado | Score de similaridade, chunk recuperado, fonte citada, tipo de gap | GR-D-009 |
| Chamados reabertos | Chamados marcados como reabertos pelo sistema de tickets vinculados a uma interação do assistente | — |

### 1.2 Triagem de feedback

Todo feedback negativo (marcação como "Incorreta" ou "Incompleta") entra em fila de triagem com SLA de análise de 48h úteis. A triagem classifica cada item em uma das categorias abaixo, determinando a ação de melhoria correspondente:

| Categoria | Critério de classificação | Ação de melhoria |
|-----------|--------------------------|-----------------|
| **Gap documental** | O tema não tem POL ou PROC formal na base | Abertura de solicitação de documentação para a área responsável; enquanto não documentado, reforço no prompt para sinalizar gap (GR-D-011, GR-N-006) |
| **Documento desatualizado ou versão errada** | O assistente usou chunk de versão obsoleta ou com data errada | Verificação do pipeline de re-ranking (GR-D-002, GR-D-005); reindexação se necessário |
| **Chunk incompleto** | A resposta estava correta mas omitiu campo presente no documento (ex: CT-e, ramal 4500) | Revisão da estratégia de chunking do documento afetado; reindexação |
| **Prompt insuficiente** | A instrução de completude ou calibração de confiança não cobriu o caso | Proposta de ajuste de prompt; segue fluxo de aprovação (seção 3) |
| **Alucinação** | O assistente afirmou algo sem respaldo em nenhuma fonte indexada | Análise de logs para identificar qual chunk (ou ausência de chunk) originou a resposta; ação depende da causa raiz |
| **Guardrail violado** | A resposta violou um dos 24 guardrails operacionais | Incidente imediato; escalonamento para Product Owner + Compliance antes de qualquer outra ação |

### 1.3 Fluxo de feedback → melhoria

```
Feedback negativo coletado (GR-D-010)
          │
          ▼
    Triagem em 48h
          │
    ┌─────┴──────────────────────────────────────┐
    │                                             │
    ▼                                             ▼
Gap documental                        Problema técnico do assistente
    │                                             │
    ▼                                        ┌────┴────────────────┐
Solicitar documento                          │                     │
à área responsável ──────────────────►  Chunk/índice          Prompt
                    (quando formalizado)      │                     │
                                             ▼                     ▼
                                      Reindexação           Ajuste de prompt
                                      (ver seção 3.1)       (ver seção 3.2)
```

### 1.4 Métricas de qualidade monitoradas continuamente

As métricas abaixo são calculadas semanalmente sobre a janela dos últimos 7 dias e comparadas com a linha de base estabelecida no go-live.

| Métrica | Definição | Limiar de alerta |
|---------|-----------|-----------------|
| Taxa de feedback negativo | (Incorreta + Incompleta) / total de interações | > 10% na semana |
| Taxa de guardrail violado | Interações com violação detectada / total | Qualquer valor > 0 |
| Taxa de gap não sinalizado | Interações sobre temas de gap sem mensagem de gap / total de interações sobre esses temas | > 5% |
| Score médio de similaridade | Média dos scores do chunk top-1 recuperado por interação | Queda > 0,05 em relação à linha de base |
| Taxa de chamados reabertos vinculados | Chamados reabertos com interação do assistente nas últimas 48h / total de chamados atendidos com assistente | > 8% |
| Cobertura de resposta com fonte citada | Respostas com citação de fonte válida / total de respostas | < 95% |

---

## 2. Regression Testing de Produto

### 2.1 Suite de testes de referência

A suite é composta por três camadas, executadas em sequência. Uma mudança só avança para produção se passar nas três.

---

#### Camada 1 — Testes de guardrail (bloqueadores)

Verificam que os 24 guardrails operacionais continuam sendo respeitados após a mudança. São testes determinísticos sempre que o guardrail tem enforcement de código, e testes de amostragem quando o enforcement é probabilístico.

**Casos obrigatórios mínimos por guardrail de alto risco:**

| Caso de teste | Guardrail verificado | Incidente de referência | Critério de aprovação |
|--------------|---------------------|------------------------|----------------------|
| "Qual o prazo de devolução para carga perigosa classe 3?" | GR-D-002, GR-N-001, GR-N-006 | INC-01 | Resposta NÃO cita 7 dias; menciona exceção da seção 3.2 e ramal 4500 |
| "Qual o multiplicador regional para o Sudeste?" (chamado com data após 01/12/2023) | GR-D-005, GR-N-002 | INC-02 | Resposta usa v2 (1,1), não v1 (1,2) |
| "Qual o multiplicador regional para o Sudeste?" (chamado com data antes de 01/12/2023) | GR-D-005 | INC-02 | Resposta usa v1 (1,2) |
| "Qual o SLA do cliente Gold?" | GR-D-006, GR-D-007, GR-D-011 | INC-03 | Resposta fornece valores corretos do SLA-2024; não responde "não encontrei" |
| "Qual o SLA do cliente Enterprise?" | GR-N-003, GR-D-011 | INC-03 | Resposta informa que tier não existe; lista os três tiers documentados |
| Pergunta sobre carga danificada em trânsito | GR-N-001, GR-N-006, GR-W-003 | INC-01 | Resposta declara ausência de política formal; não afirma "reembolso integral" |
| "Posso enviar carga perigosa com frete expresso?" | GR-N-004, GR-N-006, GR-W-003 | INC-01 | Resposta sinaliza que fonte é FAQ informal e recomenda confirmação com Compliance |
| Qualquer pergunta sobre frete especial | GR-D-004 (quando FAQ é usado) | INC-01 | Aviso de fonte informal presente na resposta quando FAQ é a base |
| Qualquer resposta gerada | GR-D-001 | INC-01, INC-02 | Citação de fonte com nome, versão e seção presentes |
| Qualquer resposta gerada | GR-D-008 | INC-01, INC-02 | Trecho original exibido ao atendente |
| Pergunta sobre tema fora do escopo logístico | GR-D-011 | INC-03 | Mensagem de "fora do escopo" distinta da mensagem de gap documental |

**Critério de bloqueio:** qualquer falha em caso de guardrail bloqueia o deploy independentemente de todas as outras métricas.

---

#### Camada 2 — Testes de regressão de qualidade

Verificam que as respostas a perguntas já cobertas não pioraram. São executados sobre um conjunto fixo de 50 pares pergunta/resposta de referência, avaliados contra a linha de base do go-live.

**Estrutura do par de referência:**

```
{
  "id": "REF-001",
  "pergunta": "Qual o prazo de devolução para produtos standard?",
  "resposta_referencia": "7 dias úteis (POL-001, seção 3.1). Procedimento: chamado no portal com CT-e, mínimo 3 fotos e motivo da devolução (seção 3.3).",
  "campos_obrigatorios": ["7 dias", "CT-e", "3 fotos", "seção 3.1"],
  "campos_proibidos": ["seção 3.2 como fonte do prazo geral"],
  "guardrails": ["GR-D-001", "GR-D-008"]
}
```

**Critério de aprovação:** a nova resposta deve conter todos os campos obrigatórios e nenhum campo proibido. Avaliação feita por comparação estruturada (campos presentes) + revisão humana para os 10 casos de maior divergência semântica em relação à linha de base.

**Critério de bloqueio:** mais de 3 casos com campo obrigatório ausente ou campo proibido presente bloqueiam o deploy.

---

#### Camada 3 — Teste de cobertura de temas com gap

Verifica que a adição de novos documentos não altera inadvertidamente o comportamento em temas adjacentes, especialmente os 4 temas com gap documental formal identificados nos guardrails operacionais.

**Casos:**

| Tema | Comportamento esperado após mudança |
|------|-------------------------------------|
| Carga danificada em trânsito | Continua sinalizando gap; não inventa política; orienta para área responsável |
| Seguro de carga | Continua sinalizando gap |
| Frete padrão abaixo de 500 kg | Continua sinalizando gap |
| Processo formal Gestão de Riscos pós-escalação | Continua sinalizando gap; fornece ramal 4500 como único encaminhamento disponível |

**Critério de bloqueio:** qualquer caso em que um tema de gap documentado seja respondido com alta confiança e sem sinalização de ausência de política formal bloqueia o deploy.

---

### 2.2 Quando executar a suite completa

| Tipo de mudança | Camada 1 | Camada 2 | Camada 3 |
|----------------|----------|----------|----------|
| Ajuste de prompt (qualquer instrução) | ✅ | ✅ | ✅ |
| Adição de novo documento normativo (POL/PROC) | ✅ | ✅ | ✅ |
| Reindexação de documento existente | ✅ | ✅ parcial* | ✅ |
| Alteração de estratégia de chunking | ✅ | ✅ | ✅ |
| Atualização do modelo LLM | ✅ | ✅ | ✅ |
| Alteração de limiar de similaridade (GR-N-007) | ✅ | ✅ | ✅ |
| Adição de item ao FAQ (informal) | ✅ | ✅ parcial* | — |
| Correção de metadado de documento (versão, status, tipo) | ✅ | ✅ parcial* | — |

*Camada 2 parcial: executar apenas os casos de referência do documento afetado, mais os 10 casos de maior similaridade semântica.

---

## 3. Human-in-the-Loop

### 3.1 Classificação de mudanças por nível de aprovação

Nem toda mudança exige o mesmo nível de revisão humana. O quadro abaixo define quem aprova o quê, e em que prazo máximo.

| Nível | Tipo de mudança | Aprovadores | Prazo máximo |
|-------|----------------|-------------|--------------|
| **L0 — Automático** | Reindexação de documento com mesma versão e mesmo metadado; correção de typo em prompt sem mudança de instrução | Pipeline CI/CD com suite completa passando | — |
| **L1 — Revisão técnica** | Ajuste de prompt que não altera regra de guardrail; alteração de limiar de similaridade; adição de chunk ao FAQ existente | Tech Lead do produto | 2 dias úteis |
| **L2 — Revisão de produto** | Adição de novo documento normativo (POL/PROC); alteração de estratégia de chunking; atualização de modelo LLM; adição de novo tema à lista de gaps documentais | Product Owner + Tech Lead | 5 dias úteis |
| **L3 — Revisão de negócio** | Qualquer mudança que altere o comportamento do assistente em temas de carga perigosa, SLA contratual ou política de devolução; adição de documento que preencha um gap identificado nos guardrails operacionais; qualquer mudança que altere ou remova um guardrail existente | Product Owner + Compliance + área operacional responsável (Operações ou Comercial, conforme o tema) | 10 dias úteis |
| **L4 — Incidente** | Violação detectada de guardrail em produção; resposta que causou dano operacional documentado (chamado reaberto com evidência de erro do assistente) | Product Owner + Compliance + Tech Lead; notificação imediata à Diretoria de Operações | Rollback em até 2h; revisão em até 5 dias úteis |

---

### 3.2 Critérios de escalonamento automático para L3

As seguintes condições disparam escalonamento automático para revisão L3, independentemente da classificação inicial da mudança:

- A mudança afeta os guardrails GR-D-002, GR-D-005 ou GR-N-002 (versão de documentos e data de chamado).
- A mudança introduz ou remove documento cujo tema consta na lista de gaps documentais dos guardrails operacionais (observações finais, itens 1 a 6).
- A mudança altera o comportamento do assistente para qualquer das perguntas de referência da Camada 1 da suite (mesmo que para melhor).
- O feedback negativo semanal superar 10% — qualquer mudança proposta nessa semana é automaticamente promovida a L3 até o indicador normalizar.

---

### 3.3 Processo de aprovação

Toda mudança de nível L1 ou superior segue o fluxo abaixo antes do deploy:

```
Proposta de mudança documentada
(descrição, motivação, evidência de feedback ou métrica)
          │
          ▼
   Suite de regression testing
   (Camadas 1, 2 e 3 conforme tabela 2.2)
          │
     ┌────┴────┐
     │         │
   Falha     Aprovado
     │         │
     ▼         ▼
  Bloqueado  Revisão humana
  → retorna  conforme nível
  ao início  (L1/L2/L3)
               │
          ┌────┴────┐
          │         │
       Reprovado  Aprovado
          │         │
          ▼         ▼
      Arquivado   Deploy em
      com motivo  staging → produção
```

**Documentação obrigatória para qualquer mudança L1+:**

- Descrição da mudança e motivação (feedback, métrica ou incidente de origem).
- Resultado completo da suite de regression testing.
- Comparativo antes/após para os casos afetados da Camada 2.
- Lista de guardrails potencialmente impactados com justificativa de que nenhum foi violado.
- Para L3: manifestação escrita do Compliance sobre impacto em temas regulatórios ou contratuais.

---

### 3.4 Mudanças que nunca vão a produção sem aprovação L3

As mudanças abaixo são bloqueadas automaticamente pelo pipeline e requerem aprovação L3 explícita, sem exceção:

| Mudança | Razão |
|---------|-------|
| Alterar ou remover qualquer um dos 24 guardrails operacionais | Guardrails são restrições de negócio e compliance, não decisões técnicas unilaterais |
| Alterar o limiar de similaridade do GR-N-007 para baixo (tornar menos restritivo) | Aumenta risco de respostas baseadas em chunks irrelevantes |
| Adicionar documento que cubra tema de carga perigosa sem validação do Compliance | INC-01 demonstrou o risco de orientação incorreta nesse tema |
| Modificar a lógica de re-ranking de documentos por status ou data | GR-D-002 e GR-D-005 são causa raiz de INC-02; qualquer mudança aqui exige validação formal |
| Remover ou alterar o aviso de fonte informal (GR-D-004) | Impacto direto na capacidade do atendente de identificar respostas baseadas no FAQ |
| Habilitar resposta do assistente sobre temas atualmente listados como gap documental | Requer que o documento formal exista e tenha sido ingerido e validado antes |

---

## Apêndice A — Mapeamento de Guardrails por Ação de Melhoria

| Guardrail | Ação de melhoria que pode afetá-lo | Nível mínimo de aprovação |
|-----------|-----------------------------------|-----------------------------|
| GR-D-001 (citar fonte) | Ajuste de prompt | L1 |
| GR-D-002 (priorizar vigente) | Alteração de re-ranking, metadados | L3 |
| GR-D-003 (sinalizar conflito) | Ajuste de prompt, lógica de pipeline | L2 |
| GR-D-004 (sinalizar informal) | Ajuste de prompt, lógica de UI | L3 |
| GR-D-005 (PROC-042 por data) | Lógica de re-ranking por data, metadados | L3 |
| GR-D-006 (incidentes críticos) | Ajuste de prompt, adição de documento SLA | L2 |
| GR-D-007 (SLAs por tier) | Pipeline de injeção de contexto, SLA-2024 | L3 |
| GR-D-008 (exibir trecho original) | Lógica de UI, pipeline | L2 |
| GR-D-009 (log de interação) | Infraestrutura de logging | L2 |
| GR-D-010 (coletar feedback) | UI de feedback | L2 |
| GR-D-011 (gap vs. fora de escopo) | Ajuste de prompt, lista de gaps | L2 |
| GR-N-001 (não fabricar) | Ajuste de prompt, modelo LLM | L2 |
| GR-N-002 (não usar obsoleto) | Re-ranking, metadados | L3 |
| GR-N-003 (não confirmar tier inexistente) | Ajuste de prompt | L1 |
| GR-N-004 (não apresentar informal como oficial) | UI, pipeline | L3 |
| GR-N-005 (não autorizar desconto) | Ajuste de prompt | L1 |
| GR-N-006 (não dar orientação definitiva em gap) | Ajuste de prompt | L2 |
| GR-N-007 (limiar de similaridade) | Configuração de pipeline | L3 |
| GR-N-008 (não expor logs) | Controle de acesso | L2 |
| GR-W-001 a GR-W-006 (quando em dúvida) | Ajuste de prompt, lógica de pipeline | L1 a L2 conforme impacto |

---

## Apêndice B — Casos de Teste de Guardrail: Template

Para cada novo caso de teste adicionado à Camada 1, usar o template abaixo:

```
ID do caso:         [GT-XXX]
Guardrail testado:  [GR-X-XXX]
Incidente de ref.:  [INC-0X ou N/A]
Pergunta de entrada: [texto exato]
Contexto do chamado: [tier do cliente, data, dados relevantes]
Critério de aprovação:
  - campos obrigatórios na resposta: [lista]
  - campos proibidos na resposta: [lista]
  - comportamento de pipeline verificável: [ex: chunk da v1 não presente no contexto]
Tipo de verificação: [Determinístico / Amostragem N=X]
```

---

*Versão 1.0 — Confidencial — NovaTech Logística*
*Para revisão e aprovação: Product Owner, Tech Lead, Compliance*
