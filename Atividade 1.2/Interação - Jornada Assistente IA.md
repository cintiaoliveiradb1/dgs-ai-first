# Interação — Diagramas da Jornada do Atendente (Assistente de IA Logística · NovaTech)

> Registro da conversa entre o usuário e o assistente de design, do briefing à entrega.

---

## 1. Pedido inicial

**Usuário:** Gere diagramas visuais baseados no documento anexo.

**Documento-base:** `jornada-atendente-assistente-ia-logistica.md` — especificação da jornada do atendente apoiado por um assistente de IA, contendo fluxo principal, fluxo de fallback, fluxo de feedback, resumo da jornada, guardrails de comportamento e tabelas de métricas/exemplos.

**Observação do assistente:** o design system vinculado ao projeto estava **vazio** — sem estilo visual obrigatório a seguir. Direção visual proposta do zero.

---

## 2. Perguntas de descoberta e respostas

| Tema | Resposta do usuário |
|------|---------------------|
| **Formato de entrega** | Pôster único navegável (uma página HTML com todos os fluxos) |
| **Escopo dos fluxos** | Fluxo principal · Fluxo de fallback · Fluxo de feedback |
| **Público** | Misto (produto/design, liderança e atendimento) |
| **Direção visual** | Editorial quente — papel creme, cards com contorno arredondado, acento dourado |
| **Papel da IA vs. atendente** | Diferenciar visualmente quem faz o quê (raias/cores por ator) |
| **Variações** | Sim — explorar 2–3 direções lado a lado |
| **Idioma** | Português (BR) |

**Fora de escopo (por escolha):** guardrails de comportamento e tabelas isoladas de métricas como diagramas próprios.

---

## 3. Sistema de design definido

**Premissa central:** o assistente não substitui o atendente — elimina o tempo de busca e reduz a inconsistência de resposta. O atendente permanece responsável pela decisão e pela comunicação com o cliente.

**Tipografia**
- Serifada de títulos: *Newsreader*
- Sans de texto/UI: *Hanken Grotesk*
- Mono de etiquetas/documentos: *IBM Plex Mono*

**Paleta — papel & tinta:** papel creme (`#f3ede0`), tinta escura (`#211d16`), acento ouro (`#f0bf3f` / `#c8851d`).

**Cores por ator**

| Ator | Papel | Cor |
|------|-------|-----|
| Cliente | Origem do chamado | Bege neutro `#8a7a5c` |
| Atendente | Humano · decide e comunica | Verde `#1f6f5c` |
| Assistente IA | Busca, estrutura e cita fonte | Âmbar `#b9791a` |
| Sistema | Registra e agrega dados | Ardósia `#4f6670` |
| Área especialista | Escala externa (Comercial, Jurídico…) | Rúgua/rust `#b04a31` |

**Métricas de discovery em destaque:** 4,1 fontes consultadas por chamado (6,2 em frete especial) · 22% de reaberturas em 48h · 90% das dúvidas cobertas pelo fluxo principal.

---

## 4. Arquitetura da solução

Página **orientada a dados**: o conteúdo dos três fluxos é modelado uma única vez e renderizado por três motores visuais distintos, garantindo conteúdo idêntico entre as direções.

```
Jornada Assistente IA.html      ← pôster navegável (alterna direções)
Jornada Assistente IA-print.html ← versão para PDF (3 direções empilhadas)
assets/
├── data.js     ← fonte única dos 3 fluxos, atores, premissa e métricas
├── utils.js    ← construtores de conteúdo (bullets, decisões, rotas, callout, tabelas)
├── poster.css  ← sistema visual + estilos das 3 direções
├── dirA.js     ← Direção A — Raias Editoriais
├── dirB.js     ← Direção B — Trilha Vertical
└── dirC.js     ← Direção C — Console Conversacional
```

---

## 5. As três direções visuais

- **A · Raias Editoriais** — raias horizontais por ator; o card "pula" de coluna a cada handoff, com conectores em L mostrando quem passa o bastão para quem.
- **B · Trilha Vertical** — espinha de playbook editorial: discos numerados coloridos por ator, numeral serifado de fundo, exemplos e métricas em largura cheia.
- **C · Console Conversacional** — a jornada como a interface real do assistente: o atendente digita a dúvida, a IA identifica o tema e responde com card estruturado (fonte + vigência), decisões viram botões, fallback vira alerta e feedback vira prompt real.

Todas compartilham: cores por ator consistentes, legenda no cabeçalho, premissa em destaque e links internos (`↳ fallback`, `↳ feedback`) que saltam entre fluxos.

---

## 6. Conteúdo dos fluxos

**01 · Fluxo principal** — temas com documentação oficial validada (prazos, cálculo de frete especial, política de devolução, SLA/tiers; ~90% das dúvidas). O atendente digita → IA identifica tema → resposta estruturada com fonte e vigência → atendente decide (suficiente / follow-up / fallback) → registra → prompt de feedback. Inclui exemplo de cálculo de frete especial para Manaus.

**02 · Fluxo de fallback** — acionado quando não há documentação suficiente. IA exibe mensagem estruturada indicando o responsável e o canal, sem inventar resposta. Inclui exemplos por tema (seguro de carga → Comercial; carga perigosa → Compliance; sinistro → Jurídico; frete > 5.000 kg → gerente regional).

**03 · Fluxo de feedback** — prompt de avaliação não obrigatório, campo aberto opcional, agregação no painel de qualidade e revisão periódica da base. Inclui faixa de métricas baseline → meta (ex.: reabertura 22% → <10%; fontes/chamado 4,1 → ≤1,5; utilidade ≥80%).

---

## 7. Entregas

- **`Jornada Assistente IA.html`** — pôster navegável com seletor de direção (A / B / C) e navegação entre fluxos.
- **`Jornada Assistente IA-print.html`** — versão de impressão com as três direções empilhadas (cada uma inicia em página nova), exportável para PDF via Cmd/Ctrl+P → "Salvar como PDF" (ativar "Gráficos de plano de fundo" para preservar cores).

---

*Documento gerado a partir da interação de design. NovaTech · jornada-atendente-IA · v1.*
