# Classificação de Erros do Agente e Sugestões de Ajuste

> Baseado na avaliação das respostas do agente de suporte NovaTech, com referência ao Anexo A — Documentação Simulada.

---

## Resposta 1 — Prazo de devolução standard

**Classificação:** Informação incompleta

**Erro identificado:** O agente citou POL-001, seção 3.2, mas o prazo geral está na seção 3.1 e o procedimento na seção 3.3. Além disso, omitiu o requisito obrigatório do número do CT-e na abertura do chamado.

**Ajustes propostos:**

- **Pipeline (RAG):** Configurar o chunking da POL-001 por subseção (3.1, 3.2, 3.3), e não por seção inteira. Isso garante que o retriever traga o chunk correto e que o número de seção no metadado seja preciso.
- **Prompt:** Adicionar instrução de verificação de completude: *"Ao responder sobre procedimento de devolução, confirme se mencionou: prazo, documentos necessários (CT-e, fotos), canal de abertura e categorias excluídas."*

---

## Resposta 2 — SLA Silver

**Classificação:** Informação incompleta

**Erro identificado:** A resposta é tecnicamente correta para chamados gerais (48h), mas não menciona que incidentes críticos têm SLA radicalmente diferente (8h). A diferença é contratual e de alto impacto operacional.

**Ajustes propostos:**

- **Prompt:** Instruir o agente a sempre discriminar SLA geral e crítico ao responder perguntas sobre prazo: *"Ao citar SLA, informe sempre as duas modalidades: chamado geral e incidente crítico, com os prazos de ambas."*
- **Interface:** Exibir ao agente humano, no painel lateral, o card de SLA do cliente assim que o tier é identificado — com as duas faixas destacadas visualmente. O agente complementa; o sistema informa.

---

## Resposta 3 — Devolução de carga perigosa classe 3

**Classificação:** Informação incompleta

**Erro identificado:** O agente recomendou escalar ao supervisor, mas a POL-001 (seção 3.2) especifica o setor de Gestão de Riscos pelo ramal 4500 como destino correto. A informação de ação estava presente no documento e foi ignorada.

**Ajustes propostos:**

- **Pipeline (RAG):** Garantir que o chunk da seção 3.2 da POL-001 inclua o parágrafo final com o ramal 4500. Chunks cortados antes da instrução de encaminhamento causam exatamente esse tipo de omissão.
- **Prompt:** Criar regra explícita: *"Quando a documentação indicar um setor ou ramal específico como destino de escalação, reproduza esse dado exatamente — nunca substitua por 'supervisor' ou termo genérico."*

---

## Resposta 4 — Política para carga danificada durante transporte

**Classificação:** Alucinação

**Erro identificado:** Não existe POL ou PROC sobre carga danificada. A única referência é o FAQ informal (item 38), que descreve um processo diferente do apresentado (encaminhamento para sinistros@novatech.com.br e Jurídico). O agente fabricou uma política com alta confiança e sem citar fonte.

**Ajustes propostos:**

- **Pipeline (RAG):** Implementar detecção de gap: quando o retriever retorna zero chunks de documentos normativos (POL/PROC) para um tema, sinalizar ausência de base formal antes de responder. O FAQ não deve ser usado como substituto de norma.
- **Prompt:** Adicionar regra de calibração de confiança: *"Se não houver documento POL ou PROC sobre o tema, declare explicitamente que não há política formal. Nunca atribua alta confiança a respostas baseadas apenas no FAQ."*
- **Interface:** Exibir a fonte e seu status de confiabilidade ao lado de cada resposta gerada (ex: "Baseado em: FAQ · não validado"). Isso permite ao agente humano avaliar antes de repassar ao cliente.

---

## Resposta 6 — Carga perigosa com frete expresso

**Classificação:** Fonte não confiável

**Erro identificado:** A resposta cita o FAQ item 32 como se fosse política oficial. Não existe PROC ou POL que regule o envio de carga perigosa via frete expresso. A documentação da NovaTech explicita que o FAQ não foi validado por Compliance ou Operações.

**Ajustes propostos:**

- **Pipeline (RAG):** Classificar documentos por confiabilidade no momento da ingestão (normativo / informativo / informal). Para temas envolvendo cargas perigosas, configurar filtro que exclua ou rebaixe fontes informais do ranking de retrieval.
- **Prompt:** Instrução específica: *"Para temas que envolvam carga perigosa, exija base em documento normativo (POL ou PROC). Se apenas o FAQ cobrir o tema, informe que não há política formal e sugira consulta ao Compliance ou Gestão de Riscos antes de confirmar qualquer procedimento ao cliente."*

---

## Resumo dos padrões e prioridades

| Resposta | Classificação | Camadas de ajuste |
|----------|--------------|-------------------|
| 1 | Informação incompleta | Pipeline, Prompt |
| 2 | Informação incompleta | Prompt, Interface |
| 3 | Informação incompleta | Pipeline, Prompt |
| 4 | Alucinação | Pipeline, Prompt, Interface |
| 6 | Fonte não confiável | Pipeline, Prompt |

### Padrões identificados

**Informação incompleta (respostas 1, 2 e 3):** O agente recuperou o conteúdo correto mas não o entregou de forma completa. Causas prováveis: chunks mal delimitados no RAG e ausência de instruções de completude no prompt.

**Alucinação (resposta 4):** O caso mais grave. Sem base documental formal, o agente produziu uma política inexistente com alta confiança. Exige correção nas três camadas: pipeline identificando o gap, prompt calibrando a confiança e interface tornando a fonte visível ao agente humano.

**Fonte não confiável (resposta 6):** O agente tratou o FAQ informal como norma oficial, especialmente perigoso num tema de carga perigosa. A solução mais robusta é classificar documentos por tier de confiabilidade já na ingestão — não confiar que o prompt sozinho contenha esse comportamento em cenários de risco.

### Prioridade de implementação recomendada

1. **Classificação de documentos por confiabilidade no pipeline** — impacta as respostas 4 e 6, que são os erros de maior risco.
2. **Chunking por subseção nos documentos normativos** — impacta as respostas 1 e 3, corrigindo referências incorretas e encaminhamentos errados.
3. **Regras de completude e calibração de confiança no prompt** — complementa o pipeline em todos os casos.
4. **Exibição de fonte e confiabilidade na interface** — camada de segurança final para o agente humano validar antes de repassar ao cliente.
