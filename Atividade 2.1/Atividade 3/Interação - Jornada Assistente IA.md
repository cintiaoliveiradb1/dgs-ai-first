# Interação — Design de jornada com componente de IA (NovaTech Logística)

> Registro da conversa entre o usuário e o assistente de design, do briefing às entregas. Projeto: assistente de IA de apoio ao atendimento logístico da NovaTech.

---

## Linha do tempo da interação

1. **Diagramas da jornada** (3 direções visuais) — a partir de `jornada-atendente-assistente-ia-logistica.md`
2. **Exportação dos diagramas para PDF**
3. **Exportação da interação para Markdown** (primeira versão)
4. **Mockup do assistente no Microsoft Teams** — a partir de `requirements.md`
5. **Exportação do mockup como imagens PNG**
6. **Exportação da interação para Markdown** (esta versão, atualizada)

---

## 1. Diagramas da jornada

### Pedido
"Gere diagramas visuais baseados no documento anexo." — documento `jornada-atendente-assistente-ia-logistica.md` (fluxo principal, fallback, feedback, resumo, guardrails, métricas). O design system do projeto estava vazio → direção visual proposta do zero.

### Perguntas de descoberta e respostas

| Tema | Resposta |
|------|----------|
| Formato | Pôster único navegável (uma página HTML com todos os fluxos) |
| Escopo | Fluxo principal · fallback · feedback |
| Público | Misto |
| Direção visual | Editorial quente — papel creme, contornos, acento dourado |
| IA vs. atendente | Diferenciar visualmente por ator (raias/cores) |
| Variações | Sim — 2–3 direções lado a lado |
| Idioma | Português (BR) |

### Sistema de design
- **Premissa central:** o assistente não substitui o atendente — elimina tempo de busca e reduz inconsistência; a decisão e a comunicação com o cliente permanecem com o atendente.
- **Tipografia:** Newsreader (títulos), Hanken Grotesk (texto/UI), IBM Plex Mono (etiquetas).
- **Paleta:** papel creme `#f3ede0`, tinta `#211d16`, ouro `#f0bf3f`/`#c8851d`.
- **Cores por ator:** Cliente (bege), Atendente (verde), Assistente IA (âmbar), Sistema (ardósia), Área especialista (rúgua/rust).

### Arquitetura (orientada a dados)
```
Jornada Assistente IA.html       ← pôster navegável (alterna direções)
Jornada Assistente IA-print.html ← versão para PDF (3 direções empilhadas)
assets/
├── data.js   ← fonte única dos 3 fluxos, atores, premissa, métricas
├── utils.js  ← construtores de conteúdo (bullets, decisões, rotas, callout, tabelas)
├── poster.css ← sistema visual + estilos das 3 direções
├── dirA.js   ← Direção A — Raias Editoriais
├── dirB.js   ← Direção B — Trilha Vertical
└── dirC.js   ← Direção C — Console Conversacional
```

### As três direções
- **A · Raias Editoriais** — raias por ator; o card salta de coluna a cada handoff, com conectores em L.
- **B · Trilha Vertical** — espinha de playbook com discos numerados por ator e numeral serifado de fundo.
- **C · Console Conversacional** — a jornada como a interface real do assistente.

Compartilham cores por ator, legenda, premissa em destaque e links internos (`↳ fallback`, `↳ feedback`). Guardrails e tabelas isoladas de métricas ficaram fora de escopo por escolha.

---

## 2. Exportação dos diagramas para PDF
Criada a versão `Jornada Assistente IA-print.html`: as três direções empilhadas, cada uma iniciando em página nova (A4 paisagem), exportável por Cmd/Ctrl+P → "Salvar como PDF" (ativar "Gráficos de plano de fundo" para preservar cores).

---

## 3 & 6. Exportação da interação para Markdown
Gerado este documento de registro da conversa (versão inicial e, depois, esta versão atualizada com o mockup do Teams).

---

## 4. Mockup do assistente no Microsoft Teams

### Pedido
"Baseado nesse fluxo e no arquivo anexo, crie um mockup da interface de resposta no Teams." — documento `requirements.md` (especificação do **Query Endpoint**, com comportamentos RF-001 a RF-012).

### Perguntas de descoberta e respostas

| Tema | Resposta |
|------|----------|
| Formato | Conversa contínua e realista (várias mensagens em sequência) |
| Cenários | Resposta completa com fonte oficial · pedido de dado faltante · conflito PROC-042 v1×v2 · fora do escopo |
| Contexto no Teams | Bot dedicado (conversa 1:1 com o "Assistente NovaTech") |
| Dispositivo | Desktop (Teams completo, com barra lateral) |
| Tema | Claro |
| Interatividade | Estático (alta fidelidade) |
| Extra | Ater-se aos requisitos do `requirements.md` |

### Cenários encenados (fiéis ao requirements.md)
1. **RF-002 — dado faltante:** frete especial de 800 kg p/ Nordeste sem data; o bot solicita a data do chamado e não usa suposições.
2. **RF-001 + RF-003 + RF-004 — resposta completa:** com 05/03/2024, aplica **PROC-042 v2** (multiplicador Nordeste **1,5** · fator peso **1,0**), mostra fórmula, trecho citado, selo **Fonte oficial** (PROC-042 v2.0 · §2.1 · 10/11/2023), alerta de **valor-base não indexado** e **aviso de conflito** v1 (1,4) × v2 (1,5) com regra de desempate (§5 Disposições Transitórias).
3. **RF-006 — fora do escopo:** pergunta sobre clima → card neutro e distinto ("FORA DO ESCOPO").
4. **RF-010 — feedback:** chips Útil / Incorreta / Incompleta em cada resposta.

Detalhes fiéis ao domínio: lista lateral com canais de escalada do spec (Compliance, Gestão de Riscos); rodapé reforça a premissa — o assistente apoia a decisão do atendente, não fala com o cliente.

### Arquivos
```
Mockup Teams - Assistente NovaTech.html  ← mockup (janela desktop escalável)
assets/teams.css                          ← chrome do Teams (tema claro) + cards de resposta
```

---

## 5. Exportação do mockup como imagens PNG
Geradas em `export/`:
- `01 - Visao geral.png` — janela completa com o início da conversa
- `02 - Resposta com fonte e conflito.png` — card de resposta completo
- `03 - Fora do escopo.png` — card de tema fora do escopo

Resolução 924×540 (limite da pré-visualização) — adequada para slides/chat; para alta resolução, exportar via PDF.

---

## Entregas do projeto

| Arquivo | Descrição |
|---------|-----------|
| `Jornada Assistente IA.html` | Pôster navegável com as 3 direções visuais |
| `Jornada Assistente IA-print.html` | Versão para PDF (3 direções empilhadas) |
| `Mockup Teams - Assistente NovaTech.html` | Mockup do assistente no Teams (desktop, tema claro) |
| `export/*.png` | Imagens do mockup do Teams |
| `Interação - Jornada Assistente IA.md` | Este registro |

---

*Documento gerado a partir da interação de design. NovaTech Logística · Assistente de Atendimento IA.*
