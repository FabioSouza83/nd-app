# SPEC v2 — ND Financeiro: `natureza` + Fluxo de Caixa + Compromissos
**Arquivo canônico. Deve viver no repositório (`docs/spec-natureza.md`) e ser emendado por commit, nunca por memória de conversa.**

Estado ao consolidar (05/07/2026): Bloco 1 aplicado · Bloco 2 ✅ (backfill + delete id 54 + gate provado a menos da linha Sicoob) · Blocos 3–9 pendentes.

---

## Princípios (a régua)

1. **Cálculo é rocha.** Nenhuma classificação depende de texto digitado. Erro de classificação falha para o lado VISÍVEL (item aparece no disponível), nunca para o invisível.
2. **Classificação é pela régua, nunca pelo resíduo.** Não se escolhe natureza para fazer número bater.
3. **Régua de natureza:** bem de clínica = aporte (`capital_giro`); serviço e consumível = Dras (`operacao`). O canal de pagamento (PIX direto, cartão, boleto) NÃO decide natureza.
4. **Piso de materialidade:** bem de clínica < R$100 → `operacao` (despesa do período, mesmo sendo bem que fica); ≥ R$100 → `capital_giro`. Classifica pelo valor do bem, não pelo canal nem pelo número resultante. *(Origem: decisão do id 33, corrente R$39 → operacao. Não reclassifica nada existente.)*
5. **Um livro só.** A trilha de lançamentos é a fonte da verdade. Agregados (`nd_saldos_mes`) tornam-se DERIVADOS da trilha após o Bloco 3 — nunca mais mantidos por fora.
6. **Não existe resíduo inexplicável — existe lançamento sem nome.** Toda divergência se resolve nomeando linhas, não ajustando números.
7. **Simetria:** ajustes e painéis de caixa compartilhado aparecem iguais para as duas Dras, mesmo quando afetam só uma. RLS protege dados de pacientes; caixa comum é território comum.
8. **Provisão é reserva, não despesa.** A provisão de impostos desconta do disponível no mês de competência, mas RETORNA no mês seguinte e é consumida pelo pagamento real do tributo. Regra de carry permanente: **`ant_mês = res_mês_anterior + prov_mês_anterior`**. Sem a devolução, contar o imposto pago (princípio: caixa real pesa) cobraria cada tributo duas vezes. *(Origem: teste de fronteira do fechamento de junho, 05/07 — ver "Gabarito oficial".)*

## Gabarito oficial de junho/2026

**Néia R$ 3.681,18 · Dany R$ 6.505,08 · Total R$ 10.186,26** — gabarito oficial (revisado 05/07 pelo teste de fronteira, princípio 8). Todo bloco que altere cálculo valida contra este gabarito.

**Composição:** receita da trilha = extrato Sicoob ao centavo (R$ 14.732,37 excl. aporte); despesa pela régua **contando o imposto pago** (INSS+DAS comp. maio, R$ 1.080,04) como caixa real; provisão de junho mantida como reserva.
**`ant` de junho recebe a devolução da provisão de maio** (princípio 8): `ant_n = res_mai 0,00 + prov_mai 328,10 = 328,10`; `ant_d = res_mai 462,20 + prov_mai 202,80 = 665,00`.

*Por que mudou de 3.353,08/6.302,28 → 3.681,18/6.505,08:* o número anterior contava o imposto de maio pago em junho mas NÃO devolvia a provisão de maio, descontando R$ 530,90 das Dras (328,10 N / 202,80 D) duas vezes e sem nome — viola o princípio 6. "Mês fechado não muda" protege maio (intacto nas duas leituras), não junho (nunca fechado). Opção retroativa adotada; a régua serve à verdade, não à inércia do número.

*Centavo do rateio:* o gabarito foi PRIMEIRO declarado 3.681,19/6.505,08 (total 10.186,27) — 1 centavo fantasma, o mesmo princípio 6 aplicado ao próprio auditor. O R$ 16,99 do Sicoob é conjunto e tem centavos ímpares (8,495/Dra); o centavo tem que cair numa só. **Regra determinística (código + spec): centavo ímpar de rateio conjunto → Néia** — fundamento: espelha o agregado auditado de junho (`desp_n` 2.425,57 > `desp_d` 2.425,56). Total real = 10.186,26.

**Junho FECHADO no app (05/07)** com esse resultado. Saques de junho: retenção de **R$ 2.800/Dra** (pintor de julho: 2.600 pagos 02/07 + 3.000 finais 06/07) → **Néia saca R$ 881,18 · Dany saca R$ 3.705,08**. *(Revoga a retenção anterior de 1.300/Dra — 2.381,18/5.205,08 — quando a quitação do pintor era só parcial.)*

## Classificações definitivas (junho)

| natureza | ids |
|---|---|
| `capital_giro` | 35 (câmeras 143,98) · 36–41 (cadeiras) · 52 (placa 154,14) · 105 (cadeira escritório 449,91) · 98, 99 |
| `operacao` | 31 (fita — consumível de obra) · 33 (corrente — piso de materialidade) · 50 (Projeto Arq Carla — serviço) · 53 (pintor — competência junho) · todas as receitas |
| `imposto` | 55 (INSS) · 56 (DAS) |
| `contabilidade` | 58 |
| `exibicao` | 51 (fatura Sicoob 327,62 — espelho; só R$16,99 é despesa real) |
| DELETE | 54 (placa duplicada — extrato confirma um único débito, 15/06) |

**Pendência acoplada ao Bloco 3:** criar lançamento `operacao` conjunto de **R$ 16,99** (mensalidade cartão Sicoob) NO MESMO deploy do calcDispV2 — nunca antes, pois o calcDisp antigo o somaria por cima do id 51 e dobraria a conta ao vivo.

---

## Blocos de execução

### BLOCO 1 — Schema ✅ (aplicado 05/07)
Coluna `natureza` text NOT NULL DEFAULT 'operacao', CHECK nos 5 valores. Default no banco = rede de segurança (item esquecido nasce visível).

### BLOCO 2 — Backfill + gate ✅ (05/07/2026)
Backfill aplicado pela tabela de classificações acima; id 54 deletado. Reconciliação trilha × agregado provada linha a linha: receita bate ao centavo, despesa bate a menos da mensalidade Sicoob R$16,99 (que não tem lançamento próprio ainda — adiada por sequenciamento pro Bloco 3, ver pendência acima). Gate = **3.353,08 / 6.302,28** confirmado. `calcDispV2` lê `natureza` (timing/rateio inalterados: `origem`, `tipo`, `created_at`, `venc`, `flag`).

### BLOCO 3 — Refatorar calcDisp (commit isolado)
calcDispV2 vira o calcDisp. `ehImpostoOuContab` morre; classificação passa a ser por `natureza`. **Imposto CONTA** como despesa (natureza='imposto'); capital_giro/exibicao nunca entram; contabilidade segue via `contabMeta`. Tela ganha linha "(–) Impostos pagos (comp. anterior)" separada da provisão, e a provisão exibe "reserva — retorna no mês seguinte" (princípio 8). Junto no MESMO deploy: lançamento R$16,99 (acima), atualização do `ant` de junho para 328,10/665,00 (devolução da provisão de maio), e início da derivação do `nd_saldos_mes` a partir da trilha. Rateio conjunto com centavo ímpar → Néia (regra fixa, ver Gabarito oficial). Aceite: gabarito **3.681,18/6.505,08** (total 10.186,26) reproduzido no traço + diff nomeado zero nos demais meses.

**Derivação `nd_saldos_mes` (escopo mínimo aprovado):** recomputar só `rec_n/rec_d/desp_n/desp_d` da trilha; manter `ant/prov/plab` manuais por ora.

**Após Bloco 3 verde: fechar junho no app (res = gabarito) → liberar PIX com retenção.**

**PENDÊNCIAS NOMEADAS (não travam o push):**
- **Derivação do `ant` → view:** hoje `ant = res_anterior + prov_anterior` (princípio 8) aplicado à mão no fechamento manual. Converter `nd_saldos_mes` para view/recálculo quando a regra estiver codificada (Blocos 5/6).
- **Colchão histórico (auditoria read-only):** somar provisões deduzidas em fev–abr que nunca retornaram ao `ant` do mês seguinte (regra antiga não devolvia). Se existirem, é dinheiro das Dras sem atribuição na conta. Devolução, se couber, via ajuste declarado (Bloco 6) — meses fechados permanecem imutáveis. Maio→junho já corrigido (ver Gabarito oficial).

### BLOCO 4 — Inferência por gêmeo + fila de pendentes
Lançamento novo: busca gêmeo no histórico (mesma `cat` + descrição normalizada) → herda `natureza` **e `compromisso_id`**. Sem gêmeo → `operacao` + fila. Fila é exclusiva do Fábio (admin); permite classificar natureza E vincular compromisso. Formulário das Dras não muda em nada.

### BLOCO 5 — Trava de fechamento
"Fechar mês" só habilita com fila zerada para o mês. Contador visível ("N pendentes", link pra fila).

### BLOCO 6 — Mês fechado imutável + lançamento de ajuste
Mês fechado nunca reabre. Correção retroativa = ajuste no mês corrente com 4 campos obrigatórios: **o quê** (FK ao original, clicável) · **por quê** (natureza anterior → correta) · **quanto/pra quem** (efeito por Dra, explícito) · **quando** (mês do ajuste + mês de referência). Simetria obrigatória (princípio 7).

### BLOCO 7 — Resumo de fechamento em 3 blocos
1. **Resultado do mês** (puro, comparável) · 2. **Ajustes de meses anteriores** (só renderiza se houver; cada item com link, motivo, efeito por Dra) · 3. **Total a transferir** (1+2 → é o PIX).

### BLOCO 8 — Tabela `compromissos` (generaliza `aportes`)
Campos: `valor_principal`, `valor_total`, `pago_acumulado` (por vínculo `compromisso_id`, nunca por texto), `saldo`, `condicao_quitacao`, `credor`, cronograma.

**Compromisso 1 — Empréstimo Néia:**
- Entrou: R$ 35.000 (2 PIX de Edineia em **29/06**: 20.000 + 15.000 — data do banco, corrige 27–28/06)
- Devolução: **48 × R$ 1.239,09 = R$ 59.476,32** · 1ª parcela **27/07/2026**, mensal dia 27
- Custo bancário: **R$ 24.476,32** (empréstimo no CPF da Néia, repassado sem margem — exibir explícito)
- Parcelas NÃO são pré-criadas (chegam pelo extrato — pré-criar causaria dupla contagem, provável origem das fantasmas ids 62–97). Cronograma vive na tabela; cada linha de extrato recebe `compromisso_id` (1ª manual, seguintes por gêmeo)
- Parcela = despesa conjunta 50/50, `operacao`. **NUNCA deduz do envelope** (envelope diminui por designação de capital; devolução diminui por parcela paga — livros independentes)

**Compromisso 2 — Contrato do pintor:**
- Total: R$ 13.600 (12.000 + 1.600) · Pago: mai 5.000 (id 28 ✓ contado) + jun 3.000 (id 53 ✓ contado) + jul 2.600 (02/07) + jul 3.000 finais (06/07) → **QUITADO 06/07, contrato completo**. Lançar as parcelas de julho quando o extrato de julho entrar.

### BLOCO 9 — Painel de Fluxo de Caixa (visível às duas Dras)
**Cartão A — Envelope:** saldo livre pra comprometer (R$ 33.251,98 de 35.000) + lista de compromissos designados. Número de decisão; extrato vira só conferência.
**Cartão B — Devolução (Néia):** três linhas fixas — Aporte recebido 35.000 · Custo bancário (juros) 24.476,32 · Total a devolver 59.476,32 (48× 1.239,09) — + contador "devolvido R$X, faltam N, próxima em DD/MM" por soma de vínculos.
Mensagem de design: saldo em conta tem origem declarada; custo bancário explícito impede a leitura errada de "Néia lucrando sobre a clínica".

---

## Registro de decisões (por quê, não só o quê)

- **Gabarito quase caiu por memória institucional:** classificações existiam em chat/obs, não em coluna. Cura = `natureza` no schema. (05/07)
- **"Fantasmas" de R$2.300/2.810 eram os 5 recebimentos de 30/06** digitados só no agregado. Cura = princípio 5, um livro só. (05/07)
- **Resíduo de R$16,99 era a mensalidade Sicoob** enterrada na linha-espelho — não arredondamento. Confirma princípio 6. (05/07)
- **Todo número novo pode ser ponta de contrato** (35k viraram 59k; 3k viraram 13,6k). Toda despesa nova ≥ R$1.000 merece a pergunta: "é despesa ou parcela de algo maior?" — a tabela `compromissos` é onde a história inteira mora.
- **Requisito transversal:** telas novas responsivas desktop + mobile (padrão de todos os apps).
