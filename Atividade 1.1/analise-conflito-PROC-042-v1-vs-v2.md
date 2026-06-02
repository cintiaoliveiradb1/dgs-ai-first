# Análise de Conflito — PROC-042 v1 vs v2

> **Origem:** Leitura direta dos arquivos `PROC-042-frete-especial-v1.md` e `PROC-042-v2-frete-especial-revisado.md`
> **Finalidade:** Subsidiar decisão de indexação para assistente de IA de atendimento logístico
> **Conclusão antecipada:** Os dois documentos são mutuamente incompatíveis e não podem coexistir no corpus do assistente.

---

## Metadados dos documentos

| Campo | v1.0 | v2.0 | Observação |
|---|---|---|---|
| Data de emissão | 03/03/2023 | 10/11/2023 | 8 meses de diferença |
| Responsável | Diretoria Comercial | Diretoria Comercial | Igual |
| Indicação formal de vigência | Ausente | Ausente | Ambos sem status formal |
| Indicação de obsolescência | Ausente | Ausente | Ambos coexistem sem hierarquia |

---

## Divergências identificadas

### 1. Multiplicadores regionais

Todas as regiões sofreram aumento na v2. Nenhuma permaneceu igual.

| Região | v1 (mar/2023) | v2 (nov/2023) | Variação |
|---|---|---|---|
| Norte | 1,6 | 1,8 | +12,5% |
| Nordeste | 1,4 | 1,5 | +7,1% |
| Centro-Oeste | 1,3 | 1,4 | +7,7% |
| Sul | 1,2 | 1,3 | +8,3% |
| Sudeste | 1,0 | 1,1 | +10,0% |

---

### 2. Fatores de peso

Os fatores de peso moveram-se em direção oposta aos multiplicadores: reduziram na v2.

| Faixa de peso | v1 | v2 | Variação |
|---|---|---|---|
| 500 kg – 1.000 kg | 1,0 | 1,0 | Sem alteração |
| 1.001 kg – 3.000 kg | 1,2 | 1,15 | −4,2% |
| Acima de 3.000 kg | 1,5 | 1,4 | −6,7% |

> **Atenção:** A redução dos fatores de peso não compensa o aumento dos multiplicadores regionais. O efeito líquido no valor final depende da combinação de região e faixa de peso — tornando imprevisível qual versão gera valor maior sem calcular caso a caso.

---

### 3. Prazo adicional de entrega

| Parâmetro | v1 | v2 | Impacto |
|---|---|---|---|
| Dias úteis adicionais | +2 dias | +3 dias | +1 dia útil (+50%) |
| Justificativa declarada | Manuseio de carga pesada | Manuseio e roteirização | Escopo ampliado na v2 |

---

### 4. Política de descontos por volume

Esta é a divergência mais crítica em termos de processo. As regras são **mutuamente exclusivas** — não se trata de atualização de valores, mas de modelos operacionais diferentes.

| Regra | v1 | v2 |
|---|---|---|
| Gatilho mínimo | 10 fretes/mês | 8 fretes/mês |
| Desconto automático (faixa 1) | Não previsto | 5% sobre multiplicador regional |
| Desconto automático (faixa 2) | Não previsto | 10% (acima de 15 fretes/mês) |
| Processo de concessão | Negociação comercial + aditivo contratual | Automático até 10%; acima exige Diretoria Comercial |

Na v1, nenhum desconto é concedido sem negociação explícita e registro formal. Na v2, o desconto é concedido automaticamente ao atingir o volume mínimo. Aplicar a regra errada gera concessão indevida de desconto ou negação de benefício ao qual o cliente tem direito contratual.

---

### 5. Disposição transitória e referências externas

| Item | v1 | v2 | Status |
|---|---|---|---|
| Chamados anteriores a 01/12/2023 | Sem regra específica | Usar multiplicadores da v1 | Regra unilateral, definida apenas na v2 |
| PROC-043 (cargas perigosas) | Referência direta como vigente | "Em processo de revisão pelo Compliance" | Incerteza adicional — documento referenciado em estado indefinido |
| Aprovação para cargas ≥ 5.000 kg | Gerente de operações regional | Gerente de operações regional | Sem alteração |

---

## Simulação de impacto — exemplo prático

**Cenário:** Carga de 2.000 kg com destino à região Norte. Valor base hipotético: R$ 1.000,00.

### Valor do frete

| Versão | Cálculo | Resultado |
|---|---|---|
| v1 | R$ 1.000 × 1,6 (Norte) × 1,2 (peso 1.001–3.000 kg) | **R$ 1.920,00** |
| v2 | R$ 1.000 × 1,8 (Norte) × 1,15 (peso 1.001–3.000 kg) | **R$ 2.070,00** |
| **Diferença** | | **+ R$ 150,00 (+7,8%)** |

### Prazo de entrega

Assumindo rota com prazo padrão de 5 dias úteis:

| Versão | Cálculo | Resultado |
|---|---|---|
| v1 | 5 dias + 2 adicionais | **7 dias úteis** |
| v2 | 5 dias + 3 adicionais | **8 dias úteis** |
| **Diferença** | | **1 dia útil** |

---

## Impacto no assistente de IA

### Risco 1 — Erro de cálculo garantido se ambas forem indexadas
Com multiplicadores divergentes em todas as regiões e fatores de peso também alterados, qualquer consulta de frete especial pode retornar valores distintos dependendo de qual documento for consultado primeiro. Não há como o assistente "escolher" a versão correta sem uma regra explícita de hierarquia.

### Risco 2 — Prazo comunicado ao cliente pode estar errado
A diferença de +1 dia útil é um dado objetivo que o atendente comunicará ao cliente. Usar a versão errada gera expectativa incorreta e potencial violação de SLA — especialmente para clientes Gold (resolução em 24h).

### Risco 3 — Desconto por volume com regras mutuamente exclusivas
O atendente pode negar desconto a que o cliente tem direito (ignorando v2) ou conceder desconto indevido sem o aditivo contratual exigido (ignorando v1). Ambos os cenários geram risco comercial e contratual.

### Risco 4 — Disposição transitória cria terceiro cenário
Chamados anteriores a 01/12/2023 ainda em aberto devem usar os multiplicadores da v1. O assistente precisaria de lógica condicional baseada na data de abertura do chamado — complexidade não trivial que precisa ser modelada antes da indexação.

### Risco 5 — PROC-043 em estado indefinido
A v2 declara que a PROC-043 (frete de cargas perigosas) está "em processo de revisão pelo Compliance". Isso amplia o gap já identificado: não há apenas ausência de documentação sobre cargas perigosas — há instabilidade declarada na única referência existente.

---

## Recomendações

| Prioridade | Ação | Responsável sugerido |
|---|---|---|
| 🔴 Imediata | Deprecar formalmente o PROC-042-v1 no SharePoint com indicação de substituição pela v2 | Diretoria Comercial |
| 🔴 Imediata | Indexar apenas a v2 no corpus do assistente | Equipe de IA |
| 🔴 Imediata | Implementar regra de data no assistente: chamados anteriores a 01/12/2023 seguem multiplicadores da v1 | Equipe de IA |
| 🟠 Alta | Validar se há chamados anteriores a dez/2023 ainda em aberto e quantificá-los | Operações |
| 🟠 Alta | Verificar status da revisão da PROC-043 com Compliance | Compliance |
| 🟡 Média | Revisar aditivos contratuais existentes para adequação à nova política de descontos da v2 | Comercial / Jurídico |

---

## Parâmetros consolidados da versão vigente (v2)

Para referência, os valores que devem ser utilizados pelo assistente após resolução do conflito:

### Multiplicadores regionais (vigentes desde nov/2023)

| Região | Multiplicador |
|---|---|
| Norte | 1,8 |
| Nordeste | 1,5 |
| Centro-Oeste | 1,4 |
| Sul | 1,3 |
| Sudeste | 1,1 |

### Fatores de peso (vigentes desde nov/2023)

| Faixa de peso | Fator |
|---|---|
| 500 kg – 1.000 kg | 1,0 |
| 1.001 kg – 3.000 kg | 1,15 |
| Acima de 3.000 kg | 1,4 |

### Demais parâmetros vigentes

- **Prazo adicional:** +3 dias úteis sobre o prazo padrão da rota
- **Aprovação gerencial:** obrigatória para cargas acima de 5.000 kg
- **Desconto automático faixa 1:** 5% sobre o multiplicador regional a partir de 8 fretes especiais/mês
- **Desconto automático faixa 2:** 10% sobre o multiplicador regional acima de 15 fretes/mês
- **Desconto acima de 10%:** requer aprovação da Diretoria Comercial
- **Cargas perigosas:** seguem PROC-043 (em revisão — aguardar nova versão)

---

*Análise gerada em maio/2025 com base na leitura direta dos arquivos PROC-042-frete-especial-v1.md e PROC-042-v2-frete-especial-revisado.md.*
