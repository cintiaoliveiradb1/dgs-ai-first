#### Exercício 2.2 — Definição de guardrails como artefato de produto

**Contexto:** Na fase anterior, você identificou guardrails informais. Agora você precisa formalizá-los como um artefato estruturado consumível por humanos e agentes.

**Ferramentas a utilizar:** Claude (chat)

**Inputs fornecidos:**
- O cenário completo.
- A documentação da NovaTech (ver **Anexo A**) como fonte de verdade para os guardrails.
- Os guardrails informais do cenário 1: *"(1) Sempre citar fonte. (2) Nunca inventar prazos ou valores. (3) Quando não encontrar resposta, dizer explicitamente. (4) Responder em português formal."*
- 3 incidentes simulados onde o assistente falhou durante testes internos:
  1. *"O assistente respondeu que o prazo de devolução para carga perigosa é 7 dias, quando na verdade cargas perigosas NÃO podem ser devolvidas."*
  2. *"O assistente citou 'PROC-042, seção 2' mas os multiplicadores informados eram da versão 1 (desatualizada), não da v2 (vigente)."*
  3. *"O assistente disse 'Não encontrei informação sobre isso' para uma pergunta sobre SLA Gold, mas o documento SLA-2024 estava indexado e continha a resposta."*

**Tarefa:**
1. Usando o **Claude**, elabore um documento de guardrails organizado em:
   - **DEVE** (comportamentos obrigatórios).
   - **NÃO DEVE** (comportamentos proibidos).
   - **QUANDO EM DÚVIDA** (comportamentos de fallback).

2. Para cada guardrail, classifique como: enforcement via prompt (probabilístico) ou enforcement via código (determinístico). Justifique.

3. Conecte cada guardrail a ao menos um dos 3 incidentes (qual incidente esse guardrail previne?).

**Entregável:** O documento de guardrails completo, com classificação de enforcement e rastreabilidade aos incidentes.

**Critérios de avaliação:**
- Os guardrails são específicos ao domínio da NovaTech, não genéricos.
- A classificação prompt vs código demonstra compreensão de que prompts são probabilísticos e código é determinístico.
- Cada guardrail é rastreável a um risco concreto (incidente).

---