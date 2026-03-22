# RELATORIO DE WORKFLOWS N8N - COSIARARAS
**Data:** 22 de Marco de 2026
**Instancia:** energetictriggerfish-n8n.cloudfy.live
**Total de workflows:** 44

---

## WORKFLOWS TESTADOS E ATIVOS (3)

| # | Workflow | ID | Status | Teste | Tag |
|---|----------|----|--------|-------|-----|
| 1 | Consultar Cardapio (tool) | `9JQ8ZZr77KVYnNTh` | ATIVO | OK - Sub-workflow funcional | Cosi - Atendimento |
| 2 | Resposta Whats | `PL9pDNWyJ7nM1SWc` | ATIVO | OK - Exec #5741 sucesso (1.8s) | Cosi - Atendimento |
| 3 | Emporio Cosi - Instagram | `C03ebvbUqUhi1gP5` | ATIVO | ERRO - Token Instagram expirado | Cosi - Atendimento |

### Detalhes dos testes:

**Resposta Whats** - FUNCIONANDO
- Webhook: `/whatsapp`
- Teste: mensagem "Oi, qual o cardapio de hoje?"
- Resultado: sucesso, processou em 1.8s
- Fluxo: Webhook > Normalize > Filtro Spam > Horario > AI Agent (OpenAI) > Cross-sell > Evolution API

**Emporio Cosi - Instagram** - TOKEN EXPIRADO
- Webhook: `/instagram`
- Teste: mensagem "Oi, voces tem pao de fermentacao natural hoje?"
- Resultado: ERRO no no "Send Instagram1"
- Erro: `Error validating access token: The session has been invalidated`
- Causa: Token Instagram (IGAAWbkyb74n5...) expirado/invalidado pelo Facebook
- PENDENCIA: Gerar novo token no Meta Business Suite com permissao `instagram_manage_messages`

---

## WORKFLOWS CORRIGIDOS HOJE (2)

| # | Workflow | ID | Correcao | Tag |
|---|----------|----|----------|-----|
| 1 | MP notificacao | `K43zyfUVq07szg2k` | Removido override R$1,00 + conectado fluxo marmitas ao Merge | Cosi - Pagamentos |
| 2 | MP preferencia | `kfudThI2lNnw21HG` | Removido no de teste que forcava valor R$1,00 | Cosi - Pagamentos |

---

## WORKFLOWS FUNCIONAIS (nao testados em execucao, mas estrutura OK) (11)

### Cosi - Pagamentos
| # | Workflow | ID | Descricao | Obs |
|---|----------|----|-----------|-----|
| 1 | MP notificacao | `K43zyfUVq07szg2k` | Recebe webhook MP, atualiza Supabase, financial_entries, print_queue, WhatsApp | Corrigido hoje. Suporta MARM/CORP/ENC + marmitariaararas |
| 2 | MP preferencia | `kfudThI2lNnw21HG` | Cria checkout Mercado Pago, retorna preferenceId | Corrigido hoje. back_urls apontam para cosiararas.lovable.app |

### Cosi - Operacional
| # | Workflow | ID | Descricao | Obs |
|---|----------|----|-----------|-----|
| 3 | WF-Central WhatsApp Router | `47oJzQO8XzuDFz6a` | Router de comandos WhatsApp (estoque, pedidos, orcamentos) | 32 nos, gestao interna |
| 4 | WF4 - Estoque WhatsApp | `W1uSdileIneKtIGq` | Movimentacao de estoque via WhatsApp + Gemini AI | Usa Supabase |
| 5 | Guardrail-Hub | `eZ5ZDBKUUPN7APD2` | Sub-workflow de seguranca chamado pelos bots Instagram | Essencial |

### Marmitaria - Atendimento
| # | Workflow | ID | Descricao | Obs |
|---|----------|----|-----------|-----|
| 6 | Marmitaria Araras - Instagram | `AfQvI2P0JAzFabsM` | Bot Instagram Marmitaria com Gemini AI + deteccao propaganda | Token Instagram pode estar expirado tambem |

### Marmitaria - Marketing
| # | Workflow | ID | Descricao | Obs |
|---|----------|----|-----------|-----|
| 7 | WF1: Boas-vindas Pos-Compra | `dcEWdEaq4t3yoNkx` | Envia boas-vindas via WhatsApp apos compra | Evolution API |
| 8 | WF2: Broadcast B2B - Menu Semanal | `tRjEx799T0kDz5xu` | Cron segunda 7h, envia cardapio para clientes B2B | Evolution API |
| 9 | WF3: Despertador de Fome B2C | `mANNed8z6A2Q9gfB` | Cron seg/qua/sex 10h30, prato do dia com foto | Evolution API |
| 10 | WF4: Recuperacao Carrinho Abandonado | `vPHABPRxU8JjcmiC` | Cron a cada 30min (10h-13h), lembrete de carrinho | Evolution API |
| 11 | WF5: Follow-up B2B Sexta | `7yFY1bLFSHjUBU3W` | Cron sexta 16h, follow-up para empresas inativas | Evolution API |

### Marmitaria - Operacional
| # | Workflow | ID | Descricao | Obs |
|---|----------|----|-----------|-----|
| 12 | Notificacao Novo Pedido | `gq243gd2bcissbo6` | Webhook de novo pedido, notifica dono via WhatsApp | Evolution API |

---

## WORKFLOWS QUEBRADOS (URL Supabase antiga / Google Sheets) (5)

Estes usam `energetictriggerfish-supabase.cloudfy.live` (URL ANTIGA) e/ou Google Sheets.
Serao migrados para Airtable.

| # | Workflow | ID | Problema | Acao |
|---|----------|----|---------|------|
| 1 | busca cardapio | `CLuPWZAARdETNWww` | Orfao (ninguem chama), URL antiga | DELETAR |
| 2 | Padaria | `lENk0kQAsdM8RWvo` | Google Sheets + URL antiga, orfao | MIGRAR para Airtable |
| 3 | Confeitaria | `l0JAILH0P2kMUJZj` | Google Sheets + URL antiga, orfao | MIGRAR para Airtable |
| 4 | Consultar Cardapio | `3r2SfdrIubTZ1a0z` | Google Sheets + URL antiga, cron 6h/9h | MIGRAR para Airtable |
| 5 | Sync Precos Google Sheets > Supabase | `IrRbnmb2NQVWLjgS` | Google Sheets, sera substituido por Airtable | MIGRAR para Airtable |

---

## WORKFLOWS NAO CATEGORIZADOS (ainda nao analisados) (20)

### Gestao Cosi (WF-series) - Provavelmente do gestao-cosiararas
| # | Workflow | ID | Descricao provavel |
|---|----------|----|-------------------|
| 1 | WF1 - Orcamentos | `BNijyaJLZqeereka` | Gestao de orcamentos |
| 2 | WF1-Inventory-CORRIGIDO | `kjVSFAmwYKXm1qz0` | Inventario corrigido |
| 3 | WF2 - Inventory Motor | `XHAfyr2nwAiWZLme` | Motor de inventario |
| 4 | WF3 - Purchases Management | `Wu3UM7W27EWQnMSt` | Gestao de compras |
| 5 | WF3 - Purchases Management (ext) | `q0nJkmOm1BLNwQnv` | Extensao de compras |
| 6 | WF3-CRUD Purchase Orders | `CvYBjXpVotimW5rp` | CRUD pedidos de compra |
| 7 | WF-Auto Sync Google Sheets | `fyrAaYdI4GZ6kbcC` | Sync automatico Sheets |
| 8 | WF-API Gateway | `PpjrEftGQz5Bxdsc` | Gateway multi-servico |
| 9 | Marmitas | `HisPQKaohtnl2E6d` | Sub-workflow marmitas |

### Temporarios / Setup
| # | Workflow | ID | Descricao |
|---|----------|----|-----------|
| 10 | TEMP Popula Sheets Gabarito | `IPd99YqdEKdnqyPs` | Temporario |
| 11 | TMP Populate Sheets MANUAL | `YmuPR5NP6JDoP0py` | Temporario |
| 12 | TEMP Cria Aba Encomendas | `hT0zOUQg9mOWCyVS` | Temporario |
| 13 | SETUP Cria Aba Encomendas | `15YQDVB6SBQkLZ6E` | Setup unico |
| 14 | My Sub-Workflow 1 | `I9CNBY8eqVkB6zGs` | Generico |
| 15 | My workflow | `6ijvJlucSY7BGokY` | Generico |

### Outros
| # | Workflow | ID | Descricao |
|---|----------|----|-----------|
| 16 | Antigravity Voice Inbox | `2Bz10GnpJ1R2fXCk` | Voice inbox (outro projeto?) |
| 17 | Public Privacy Policy | `xz4jvgWplmHJgYCM` | Politica de privacidade |
| 18 | Avaliacao Custos OCR | `bPiWpbOcCgaylZUg` | OCR de custos |
| 19 | Importacao PDFs RH | `2Dy4UMDAmAKhueS4` | Import de PDFs |
| 20 | Webhook Vendas Marmitaria | `0UenRSL4dbvkXUw6` | Vendas app |

### Backups do Zelador
| # | Workflow | ID | Descricao |
|---|----------|----|-----------|
| 21 | Zelador Monitor de Saude | `lg6poPSiF9yejF67` | Health check principal |
| 22 | Zelador v3 Final (backup) | `GKIVgVZGWwUSj3Se` | Backup v3 |
| 23 | Zelador v2 Final (backup) | `38SreW0qaGd1n3In` | Backup v2 |
| 24 | Zelador Corrigido (backup) | `kuaklSVnVgQQAhg0` | Backup corrigido |

---

## WORKFLOWS DELETADOS HOJE (16)

| # | Workflow | Motivo |
|---|----------|--------|
| 1 | notificacao novo app | URL Supabase antiga |
| 2 | Workflow Corrigido (backup) | Duplicata |
| 3 | MP Consultar Status Pagamento | Duplicata |
| 4 | MP notificacao (antiga) | Duplicata |
| 5 | Instagram Resp (Unificado) | A pedido do usuario |
| 6 | Instagram Resp (duplicata) | Duplicata |
| 7 | Emporio Cosi - Instagram (antiga) | Duplicata |
| 8 | Marketing Cosi - Funil WhatsApp | Esqueleto com placeholders |
| 9 | WF WHATS | Duplicata antiga do WF4 |
| 10 | Fluxo 8 - Orcamentos WhatsApp | Esqueleto com URL placeholder |
| 11-16 | 6 workflows adicionais | Limpeza inicial (obsoletos) |

---

## WORKFLOWS CRIADOS HOJE (1)

| # | Workflow | ID | Descricao |
|---|----------|----|-----------|
| 1 | Consultar Cardapio (tool) | `9JQ8ZZr77KVYnNTh` | Sub-workflow que busca daily_menu + paes + confeitaria do Supabase (URL nova) |

---

## PENDENCIAS

### URGENTE
- [ ] **Token Instagram Cosi** — Gerar novo token no Meta Business Suite (permissao `instagram_manage_messages`, page_id `17841400838864590`). Atualizar no no "Send Instagram1" do workflow `C03ebvbUqUhi1gP5`.
- [ ] **Token Instagram Marmitaria** — Verificar se o token do workflow `AfQvI2P0JAzFabsM` tambem esta expirado.

### PROXIMO PASSO
- [ ] Migrar Padaria/Confeitaria/Cardapio de Google Sheets para Airtable
- [ ] Recriar workflows de sync Airtable > Supabase (Precos + Estoque)
- [ ] Analisar e categorizar os 20 workflows nao analisados
- [ ] Deletar temporarios (TEMP, TMP, My workflow, My Sub-Workflow)
- [ ] Escolher e manter apenas 1 versao do Zelador
- [ ] Ativar workflows de pagamento (MP notificacao + MP preferencia) quando prontos
- [ ] Ativar workflows de marketing da Marmitaria quando prontos

### SEGURANCA
- [ ] Mover tokens do Mercado Pago para variaveis de ambiente n8n
- [ ] Mover tokens do Instagram para credentials n8n (evitar hardcode)
- [ ] Verificar se Evolution API URLs estao corretas em todos os workflows

---

## TAGS CRIADAS NO N8N

| Tag | ID |
|-----|-----|
| Cosi - Atendimento | `UtOyVPvtPwUPspOj` |
| Cosi - Operacional | `eh4kcFJSEn79ApWO` |
| Cosi - Pagamentos | `s9sWyXxKz9wiMcIs` |
| Marmitaria - Atendimento | `nUc0IJGb2IULKKPV` |
| Marmitaria - Marketing | `U1NG1IDPSh5e0c46` |
| Marmitaria - Operacional | `wt6zjErCScRwcZlE` |

*Tags precisam ser atribuidas manualmente no UI do n8n (API nao suporta)*

---

**Documento gerado automaticamente em 22/03/2026 as 19:00**
**Proximo review agendado: 23/03/2026**
