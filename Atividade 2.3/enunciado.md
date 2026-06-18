#### Exercício 2.3 — Participação na construção do AGENTS.md do projeto

**Contexto:** O Tech Lead está montando o AGENTS.md e pediu que cada papel contribua com sua seção.

**Ferramentas a utilizar:** Claude (chat)

**Inputs fornecidos:**
- O cenário completo.
- A documentação da NovaTech (ver **Anexo A**) e a estrutura do repositório (ver **Anexo C**).
- A estrutura do AGENTS.md (mesma do DM 2.3).
- Guardrails formalizados simulados (output do exercício 2.2 — fornecidos para que este exercício seja autossuficiente):
  ```
  DEVE:
  - Citar fonte com identificador do documento e seção em toda resposta.
  - Incluir campo source_document no JSON de retorno, mesmo com confiança baixa.
  - Responder em português formal.
  
  NÃO DEVE:
  - Gerar valores numéricos (prazos, multiplicadores, SLAs) que não estejam
    literalmente na documentação indexada.
  - Afirmar que carga perigosa (classes 1-6 ANTT) pode ser devolvida
    pelo processo padrão.
  - Inventar tiers de cliente (só existem Gold, Silver, Standard).
  
  QUANDO EM DÚVIDA:
  - Prefixar resposta com aviso de baixa confiança.
  - Sugerir escalação ao supervisor.
  - Se duas versões de um documento existirem, priorizar a mais recente
    e informar que existe versão anterior.
  ```

**Tarefa:**
Usando o **Claude** e referenciando os Anexos A e C, escreva a seção **"Product Rules & Guardrails"** do AGENTS.md. Ela deve conter:

1. Regras de comportamento do assistente (derivadas dos guardrails simulados acima).
2. Glossário de linguagem ubíqua do domínio que os agentes precisam conhecer (ex: "cliente Gold", "carga perigosa", "SLA de resolução", "multiplicador regional", "frete especial").
3. Restrições que impactam geração de código (ex: "toda resposta DEVE incluir o campo `source_document` no JSON de retorno").
4. Referências a documentos de spec no repositório.

**Entregável:** A seção do AGENTS.md pronta para ser adicionada ao repositório.

**Critérios de avaliação:**
- A seção é machine-readable.
- As regras são prescritivas (DEVE/NÃO DEVE).
- O glossário é útil (termos que um LLM confundiria sem contexto de domínio).
- As restrições de código são concretas o suficiente para influenciar o output do Copilot.

---