# Transcrição de Sessão — Projeto Assistente IA NovaTech
## Query Endpoint — Specification Driven Development

| Campo | Valor |
|-------|-------|
| Projeto | Assistente de Atendimento IA · NovaTech Logística |
| Sessão | Discovery → Bounded Contexts → Linguagem Ubíqua → Requirements |
| Data | Junho/2025 |
| Arquivos de entrada | `bounded-contexts-anexo-a.md` · `linguagem-ubiqua-anexo-a.md` · `spec-rag-novatech-v1_3.md` · `Interação - Jornada Assistente IA.md` |
| Arquivo produzido | `requirements.md` |

---

## Contexto anterior (sessão prévia)

> A sessão anterior produziu os três documentos de insumo usados nesta sessão. O resumo abaixo foi reconstituído a partir do histórico disponível.

Na sessão anterior, o usuário forneceu o Anexo A (documentação simulada NovaTech: POL-001 v3.1, PROC-042 v1 e v2, SLA-2024, FAQ-Atendimento), os fluxos de jornada do atendente e a Spec RAG v1.3, e solicitou três entregas sequenciais:

1. **Recorte de Domínio completo** (`recorte-dominio-novatech-bounded-contexts.md`) — 8 Bounded Contexts combinando os três arquivos de entrada, com objetivo, escopo, exclusões e relacionamentos para cada contexto.

2. **Bounded Contexts exclusivos do Anexo A** (`bounded-contexts-anexo-a.md`) — análise restrita aos documentos formais do Anexo A, produzindo 6 BCs (BC-A1 a BC-A6) com tabela de 7 conflitos e gaps (2 conflitos documentais + 5 gaps totais ou parciais).

3. **Linguagem Ubíqua** (`linguagem-ubiqua-anexo-a.md`) — 36 termos do domínio logístico com definição oficial, exemplo de uso, sinônimos proibidos, fonte e marcações de `[CONFLITO]`, `[GAP]` e `[HIPÓTESE]`, além de 4 tabelas comparativas de conflitos com regras de desambiguação.

---

## Sessão atual

### Turno 1

**Usuário:**

> Você atuará como Product Specialist especialista em Specification Driven Envelopamento (SDD).
> Sua tarefa é criar o arquivo `requirements.md` para o QUERY ENDPOINT do assistente NovaTech.
> Considere como fonte oficial os documentos anexados:
> - Bounded Contexts
> - Linguagem Ubíqua
> - Demais documentos fornecidos
>
> Objetivo do endpoint:
> Permitir que usuários façam perguntas em linguagem natural no Teams e recebam respostas consistentes com as regras do domínio.
>
> Regras:
> - Não invente informações.
> - Quando faltar definição, marque como [DECISÃO NECESSÁRIA].
> - Quando existir conflito entre documentos, explique o conflito e proponha decisão.
> - Priorize documentos normativos sobre FAQ ou conhecimento informal.
> - Use sempre os termos da Linguagem Ubíqua.
> - Não detalhe implementação técnica.
>
> Estruture o resultado em:
> - ## Objetivo
> - ## Escopo (Dentro / Fora)
> - ## Usuários e Cenários
> - ## Requisitos Funcionais (RF-NNN · DADO / QUANDO / ENTÃO)
> - ## Regras de Domínio
> - ## Prior Decisions (ADRs)
> - ## Scope Boundaries (tabela)
> - ## Casos Ambíguos
> - ## Critérios de Aceite

**Assistente:**

Leu os quatro documentos fornecidos (`bounded-contexts-anexo-a.md`, `linguagem-ubiqua-anexo-a.md`, `spec-rag-novatech-v1_3.md`, `Interação - Jornada Assistente IA.md`) e produziu o arquivo `requirements.md`.

**Estrutura entregue:**

| Seção | Conteúdo produzido |
|-------|--------------------|
| **Objetivo** | Propósito do endpoint: ferramenta de suporte ao atendente humano via Teams, reduzindo o número de fontes consultadas por chamado (4,1 → meta ≤ 1,5) e a taxa de reabertura em 48h (22% → meta < 10%). |
| **Escopo — Dentro** | 6 blocos temáticos mapeados aos BCs: Cálculo de Frete Especial (BC-A1), Prazos e Roteirização (BC-A2), Devolução de Mercadorias (BC-A3), Cargas Especiais e Restritas (BC-A4), Contratos e SLA por Tier (BC-A6), Conhecimento Informal com sinalização (BC-A5). |
| **Escopo — Fora** | Tabela com 10 temas excluídos, motivo (gap documental ou competência de outra área) e encaminhamento correto para cada. Inclui 4 itens com `[DECISÃO NECESSÁRIA]`. |
| **Usuários e Cenários** | 10 cenários (U-01 a U-10) com pergunta de exemplo e comportamento esperado, cobrindo os casos mais frequentes e os principais gaps/conflitos. |
| **Requisitos Funcionais** | 12 RFs (RF-001 a RF-012) no formato DADO/QUANDO/ENTÃO: resposta coberta, gap contextual, seleção de versão por data, sinalização de conflito, gap total, fora do escopo, fonte informal, supressão do FAQ, incidente crítico, feedback, aprovação de gerente regional e modo degradado. |
| **Regras de Domínio** | 16 regras (RD-001 a RD-016) com valores precisos, tabelas de multiplicadores/fatores/tiers/penalidades e citação de fonte para cada. |
| **Prior Decisions (ADRs)** | 6 ADRs: prioridade entre tipos de documento (ADR-001), tratamento de gaps (ADR-002), conflito PROC-042 v1 vs v2 (ADR-003), conflito FAQ-Item 45 vs PROC-042 v2 (ADR-004), limites do escopo atendente vs cliente final (ADR-005), política de não fabricação (ADR-006). |
| **Scope Boundaries** | Tabela de 6 linhas (uma por BC) com colunas "Pode consultar" e "Não pode consultar". |
| **Casos Ambíguos** | 7 casos (CA-01 a CA-07) com pergunta, motivo da ambiguidade e decisão necessária: frete padrão, seguro de carga, carga danificada, desconto de volume, data sem hora, tabela histórica de fretes-base e frete expresso para carga perigosa. |
| **Critérios de Aceite** | 10 critérios (CA-FUNC-001 a CA-FUNC-008 + CA-NF-001 e CA-NF-002) com cenário, resultado esperado e resultado inaceitável explícitos. |

---

### Turno 2

**Usuário:**

> exporte nossa interação para arquivo md

**Assistente:**

Recuperou o histórico da sessão anterior via `recent_chats`, compôs este arquivo reunindo o contexto das duas sessões e exportou como `interacao-novatech-requirements.md`.

---

## Arquivos produzidos no projeto

| Arquivo | Sessão | Conteúdo |
|---------|--------|----------|
| `recorte-dominio-novatech-bounded-contexts.md` | Anterior | Recorte de domínio completo — 8 BCs combinando todos os arquivos de entrada |
| `bounded-contexts-anexo-a.md` | Anterior | 6 BCs extraídos exclusivamente do Anexo A, com tabela de conflitos e gaps |
| `linguagem-ubiqua-anexo-a.md` | Anterior | 36 termos com definição, exemplo, sinônimos proibidos, fonte e marcações de conflito/gap/hipótese |
| `requirements.md` | Atual | Especificação completa do Query Endpoint (12 RFs · 16 RDs · 6 ADRs · 7 casos ambíguos · 10 critérios de aceite) |
| `interacao-novatech-requirements.md` | Atual | Este arquivo — transcrição das sessões de discovery e requirements |

---

> **Nota sobre fidelidade:** as respostas longas do assistente (detalhamento completo dos BCs, tabela de linguagem ubíqua e corpo do `requirements.md`) estão resumidas neste arquivo por volume. O conteúdo integral está nos arquivos exportados listados acima. As perguntas do usuário são reproduzidas literalmente.

---

*Transcrição v1.0 — NovaTech Logística — Confidencial*
