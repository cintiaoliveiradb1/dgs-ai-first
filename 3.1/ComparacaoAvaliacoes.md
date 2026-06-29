# Comparação de Avaliações — Respostas do Agente de Suporte NovaTech

## Tabela Comparativa

| # | Minha Avaliação | Avaliação do Usuário | Concordância |
|---|-----------------|----------------------|--------------|
| 1 | ⚠️ Parcialmente Correta | ⚠️ Parcialmente Correta | ✅ Concordam |
| 2 | ✅ Correta | ⚠️ Parcialmente Correta | ⚠️ Divergem |
| 3 | ⚠️ Parcialmente Correta | ⚠️ Parcialmente Correta | ✅ Concordam |
| 4 | ❌ Incorreta | ❌ Incorreta | ✅ Concordam |
| 5 | ✅ Correta | ✅ Correta | ✅ Concordam |
| 6 | ❌ Incorreta | ❌ Incorreta | ✅ Concordam |

---

## Detalhamento por Resposta

### Resposta 1 — Prazo de devolução para produtos standard
**Ambas:** ⚠️ Parcialmente Correta

O prazo de 7 dias úteis e o procedimento de portal + fotos estão corretos, mas a seção citada (3.2) está errada — o prazo geral está na **seção 3.1** e o procedimento na **seção 3.3**. Além disso, a resposta omite o requisito do CT-e, exigido pela seção 3.3.

---

### Resposta 2 — Prazo de resolução para cliente Silver
**Claude:** ✅ Correta | **Usuário:** ⚠️ Parcialmente Correta — **DIVERGÊNCIA**

O prazo de 48h úteis está correto para chamados gerais (SLA-2024, seção 2). A divergência está no critério de avaliação:

- **Raciocínio do Claude:** a pergunta não especificou o tipo de chamado, então o prazo geral é a resposta adequada. Nenhuma informação estava errada.
- **Raciocínio do Usuário:** num contexto de suporte real, um agente bem calibrado deveria sempre mencionar a distinção entre chamados gerais (48h) e incidentes críticos (8h), dado o impacto contratual significativo dessa diferença.

**Conclusão:** ambos os critérios são defensáveis. O critério do usuário é mais rigoroso e apropriado para avaliação de agentes de produção.

---

### Resposta 3 — Devolução de carga perigosa classe 3
**Ambas:** ⚠️ Parcialmente Correta

A informação sobre inelegibilidade ao processo padrão está correta (POL-001, seção 3.2). No entanto, a recomendação de escalar ao supervisor está incorreta: a POL-001 orienta encaminhar o cliente ao setor de **Gestão de Riscos pelo ramal 4500**.

---

### Resposta 4 — Política para carga danificada durante transporte
**Ambas:** ❌ Incorreta

Não existe documento formal (POL ou PROC) sobre tratamento de carga danificada. A única referência é o FAQ Item 38, documento informal não validado por Compliance ou Operações. Responder com alta confiança sem citar fonte, e com informação divergente do próprio FAQ (que orienta encaminhar para sinistros@novatech.com.br e Jurídico), torna a resposta incorreta.

---

### Resposta 5 — SLA do cliente Enterprise
**Ambas:** ✅ Correta

O SLA-2024 (seção 1) lista explicitamente apenas 3 tiers: Gold, Silver e Standard, com nota de que não existem outros. O agente reconheceu corretamente a ausência do tier e sugeriu escalonamento. Confiança baixa adequada ao contexto.

---

### Resposta 6 — Envio de carga perigosa com frete expresso
**Ambas:** ❌ Incorreta

A resposta baseia-se exclusivamente no FAQ Item 32, documento informal não validado. Conforme as notas da própria documentação, não existe PROC ou POL formal que defina esse processo. Usar alta confiança para uma afirmação sobre carga perigosa com base apenas em fonte não confiável é incorreto.

---

## Conclusão

As avaliações são **altamente convergentes**: 5 de 6 com o mesmo veredicto. A única divergência (resposta 2) é de **grau**, não de direção — ambas reconheceram que a resposta do agente estava tecnicamente correta, diferindo apenas na exigência de completude para SLAs com distinção crítica/geral.

O padrão de erro mais relevante identificado por ambas as avaliações foi a **confiança mal calibrada**: as respostas 4 e 6 apresentaram alta confiança justamente nos casos com base documental mais frágil.
