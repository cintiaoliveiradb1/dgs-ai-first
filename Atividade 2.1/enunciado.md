#### Exercício 2.1 — Recorte de domínio e spec de produto no formato SDD

**Contexto:** Antes de escrever a spec, você precisa recortar o domínio: quais são os bounded contexts do projeto, qual a linguagem ubíqua do domínio de logística que o time (e os agentes) devem usar, e quais são as fronteiras do que o assistente faz e não faz. Depois, você escreve a spec de requisitos do módulo principal usando SDD.

**Ferramentas a utilizar:** Claude (chat) + Claude Design

**Inputs fornecidos:**
- O cenário completo.
- A documentação da NovaTech (ver **Anexo A**) — use para extrair os termos do domínio e identificar os bounded contexts.
- A spec de requisitos de RAG escrita na fase anterior (simulada): *"O assistente responde perguntas sobre SLAs, frete e devoluções. Fontes contraditórias devem mostrar ambas as versões. O assistente nunca inventa informações. Toda resposta cita fonte. Atualização em até 24h."*
- O fluxo SDD: *"requirements.md contém: outcomes, scope boundaries, constraints, prior decisions, verification criteria."*
- Dados do discovery: *"As perguntas mais frequentes caem em 4 categorias: prazos de entrega, regras de frete, política de devolução e SLAs. Em 15% dos casos, a pergunta cruza duas categorias. Os atendentes precisam da resposta em menos de 30 segundos."*
- Conceito de recorte de domínio: *"Bounded contexts definem fronteiras claras entre subdomínios. Linguagem ubíqua é o vocabulário compartilhado que todo membro do time (e todo agente) usa da mesma forma. Para IA, recorte de domínio é especialmente importante porque agentes sem domínio claro geram outputs genéricos."*

**Tarefa:**
1. Usando o **Claude**, faça o recorte de domínio do projeto:
   - Identifique os bounded contexts do assistente NovaTech (ex: "Atendimento ao Cliente", "Gestão Documental", "SLAs e Contratos", "Logística de Frete"). Para cada contexto, defina: o que está dentro, o que está fora, e como se relaciona com os outros.
   - Extraia a linguagem ubíqua do domínio a partir do Anexo A: termos que precisam ser usados de forma consistente por humanos e agentes (ex: "carga perigosa" sempre significa "classes 1-6 da ANTT", "frete especial" sempre significa "acima de 500kg").

2. Usando o **Claude**, escreva o `requirements.md` do query endpoint seguindo a estrutura SDD. As prior decisions devem referenciar as ADRs da fase anterior (simuladas no contexto). Os scope boundaries devem derivar dos bounded contexts definidos acima.

3. Usando o **Claude Design**, crie um mockup da interface de resposta no Teams, coerente com os requirements.

4. Itere: peça ao Claude que atue como Tech Lead e aponte ambiguidades. Ajuste.

**Entregável:** O mapa de bounded contexts com linguagem ubíqua, o requirements.md, o mockup, e o histórico de iteração.

**Critérios de avaliação:**
- Os bounded contexts são coerentes com o domínio de logística (não são divisões técnicas como "frontend/backend").
- A linguagem ubíqua contém termos que um LLM confundiria sem definição explícita (ex: "Gold" é um tier de cliente, não o metal).
- Os outcomes no requirements.md são orientados a resultado do usuário, não a features técnicas.
- Os scope boundaries derivam dos bounded contexts (ex: "este módulo cobre o contexto 'Atendimento ao Cliente' — não cobre 'Gestão Documental' diretamente").
- Os verification criteria são testáveis pelo QA.