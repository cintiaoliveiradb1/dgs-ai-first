# Transcrição — Sessão de Análise de Domínio NovaTech
## Specification Driven Development — Fase de Discovery

**Data:** Junho/2025  
**Arquivos de entrada:** `anexo-a-documentacao-simulada-novatech.md`, `Design_de_jornada_com_componente_de_IA.pdf`, `spec-rag-novatech-v1_3.md`

---

## Turno 1

### Usuário

> Você atuará como um Analista de Negócio Sênior especializado em Specification Driven Development (SDD).
> Seu objetivo é analisar os materiais fornecidos sobre o projeto do assistente NovaTech e produzir um RECORTE DE DOMÍNIO que será usado como base para especificação funcional.
>
> Contexto: Estamos na fase inicial do projeto. O objetivo é entender o negócio e organizar o conhecimento do domínio.
>
> Entrada:
> - Anexo A (fonte principal para termos do domínio)
> - Fluxos existentes
> - Spec de RAG
>
> Identifique Bounded Contexts. Liste os principais bounded contexts do domínio. Para cada contexto apresente:
> - Nome do contexto
> - Objetivo
> - Está dentro do contexto
> - Está fora do contexto
> - Relacionamentos

### Assistente

Analisou os três arquivos e produziu o **Recorte de Domínio completo** com **8 Bounded Contexts**, divididos em duas camadas:

**Camada técnica do assistente (BC-01 a BC-07):**

| # | Bounded Context |
|---|-----------------|
| BC-01 | Gestão do Corpus Normativo |
| BC-02 | Recuperação e Composição de Respostas (RAG) |
| BC-03 | Atendimento Assistido |
| BC-04 | Gestão de Gaps e Fallback |
| BC-05 | Rastreabilidade e Auditoria |
| BC-06 | Feedback e Melhoria Contínua |
| BC-07 | Identidade e Controle de Acesso |

**Camada de negócio:**

| # | Bounded Context |
|---|-----------------|
| BC-08 | Domínio Logístico NovaTech |

Cada contexto foi detalhado com objetivo, responsabilidades dentro/fora e relacionamentos. O documento foi exportado como `recorte-dominio-novatech-bounded-contexts.md`.

---

## Turno 2

### Usuário

> agora analise apenas o arquivo anexo A e identifique contextos

### Assistente

Restringiu a análise exclusivamente ao Anexo A e reidentificou os contextos sob uma nova lente — desta vez como **contextos do domínio de negócio logístico**, não contextos técnicos de software. Identificou **6 Bounded Contexts**:

| # | Bounded Context | Fonte principal |
|---|-----------------|-----------------|
| BC-A1 | Cálculo e Precificação de Frete | PROC-042 v1 e v2 |
| BC-A2 | Prazos e Roteirização de Entrega | PROC-042 v1 e v2, SLA-2024, FAQ |
| BC-A3 | Devolução de Mercadorias | POL-001 v3.1, FAQ |
| BC-A4 | Cargas Especiais e Restritas | POL-001, PROC-042 v2, SLA-2024, FAQ |
| BC-A5 | Conhecimento Informal do Atendimento | FAQ-Atendimento |
| BC-A6 | Contratos e SLA por Tier de Cliente | SLA-2024 |

Cada contexto foi detalhado com objetivo, dentro/fora e relacionamentos. A análise encerrou com uma tabela de **2 conflitos e 5 gaps** identificados exclusivamente no Anexo A. O documento foi exportado como `bounded-contexts-anexo-a.md`.

---

## Turno 3

### Usuário

> gere um arquivo md com esse último levantamento

### Assistente

Exportou o levantamento do Turno 2 para o arquivo `bounded-contexts-anexo-a.md`, estruturado com:

- Cabeçalho de metadados (autor, fonte, versão, data)
- Nota metodológica com convenções ([HIPÓTESE], [CONFLITO], [GAP])
- Tabela de visão geral dos 6 contextos
- Detalhamento completo de cada BC (objetivo, dentro, fora, relacionamentos)
- Tabela final de conflitos e gaps:

| # | Tipo | Contextos | Descrição |
|---|------|-----------|-----------|
| C1 | Conflito | BC-A1 | PROC-042 v1 e v2 coexistem sem hierarquia formal |
| C2 | Conflito | BC-A1 / BC-A5 | Limiar de desconto diverge entre FAQ e PROC-042-v2 |
| G1 | Gap total | BC-A3 / BC-A4 | Carga danificada sem POL ou PROC formal |
| G2 | Gap total | BC-A1 | Frete padrão abaixo de 500 kg sem cobertura documental |
| G3 | Gap total | BC-A4 | Processo formal da Gestão de Riscos não documentado |
| G4 | Gap total | BC-A1 | Seguro de carga apenas no FAQ informal |
| G5 | Gap parcial | BC-A4 | Frete expresso para carga perigosa apenas no FAQ; PROC-043 em revisão |

---

## Turno 4

### Usuário

> ainda baseado apenas no Anexo A, extraia linguagem ubíqua. Monte uma tabela:
> `| Termo | Definição oficial | Exemplo de uso | Sinônimos proibidos | Fonte |`
>
> Regras:
> - Elimine ambiguidades.
> - Identifique termos conflitantes.
> - Quando uma definição não estiver explícita, proponha uma hipótese e marque como [HIPÓTESE].

### Assistente

Extraiu **36 termos** do Anexo A e os organizou na tabela solicitada, com as convenções:

- **[HIPÓTESE]** — definição inferida, não explícita
- **[CONFLITO]** — termo usado de formas incompatíveis entre documentos
- **[GAP]** — termo referenciado sem definição disponível

**Destaques por categoria:**

*Termos com [CONFLITO]:*
- **Multiplicador regional** — valores distintos entre v1 e v2 (ex.: Norte: 1,6 vs 1,8)
- **Fator de peso** — valores distintos entre v1 e v2 (ex.: faixa 1.001–3.000 kg: 1,2 vs 1,15)
- **Desconto por volume** — limiar diverge entre PROC-042-v2 (≥ 8 fretes/mês) e FAQ-Item 45 (> 10 fretes/mês)

*Termos com [GAP]:*
- **Frete padrão** — sem documento para cargas abaixo de 500 kg
- **Frete de cargas perigosas** — PROC-043 em revisão, não disponível
- **Gestão de Riscos** — ramal documentado, procedimento interno não documentado

*Termos com [HIPÓTESE]:*
- **Seguro de carga** — percentuais apenas no FAQ (0,3% padrão / 0,8% perigosas)
- **Carga danificada** — processo informal descrito no FAQ-Item 38
- **Sinistro** — termo inferido como evento de dano/perda tratado pelo Jurídico

*Sinônimos proibidos de maior risco:*
- **Platinum** → tier inexistente; usar Gold/Silver/Standard
- **Dias corridos** → usar "dias úteis"
- **Urgente / P1** → usar "incidente crítico"
- **Frete de volta / frete de retorno** → usar "frete reverso"

O documento encerrou com **4 tabelas comparativas de conflitos** (multiplicadores regionais, fatores de peso, prazo adicional e limiar de desconto), cada uma com regra de desambiguação explícita. Exportado como `linguagem-ubiqua-anexo-a.md`.

---

## Turno 5

### Usuário

> exporte nossa interação para arquivo md

### Assistente

Este arquivo.

---

## Arquivos produzidos nesta sessão

| Arquivo | Conteúdo |
|---------|----------|
| `recorte-dominio-novatech-bounded-contexts.md` | Recorte de domínio completo — 8 BCs combinando os três arquivos de entrada |
| `bounded-contexts-anexo-a.md` | 6 BCs extraídos exclusivamente do Anexo A, com tabela de conflitos e gaps |
| `linguagem-ubiqua-anexo-a.md` | 36 termos com definição, exemplo, sinônimos proibidos, fonte e marcações de conflito/gap/hipótese |
| `transcricao-sessao-novatech.md` | Este arquivo |

---

*Sessão de Discovery — NovaTech Logística — Confidencial*
