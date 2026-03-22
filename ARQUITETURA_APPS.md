# ARQUITETURA DOS APPS - COSIARARAS & MARMITARIAARARAS
**Data:** 22 de Marco de 2026

---

## VISAO GERAL

```
                    COSIARARAS                          MARMITARIAARARAS
                  (React + Vite)                      (Next.js 16 + Vercel)
                cosiararas.com.br                   marmitariaararas.com.br
                        |                                     |
          +-------------+-------------+          +------------+------------+
          |             |             |          |            |            |
       SUPABASE       n8n        AIRTABLE    SUPABASE    n8n (min)    VERCEL
      (obrigatorio) (obrigatorio) (gestao)  (obrigatorio) (opcional)  (hosting)
```

---

## 1. COSIARARAS (Emporio Cosi)

### Stack
- **Frontend:** React 18 + Vite + Tailwind + Radix UI
- **Hosting:** Vercel (cosiararas.com.br)
- **Database:** Supabase (PostgreSQL)
- **Pagamento:** Mercado Pago (via n8n)
- **Automacao:** n8n (ESSENCIAL - pagamentos passam por ele)
- **Gestao de dados:** Airtable → n8n → Supabase (em migracao)
- **WhatsApp:** Evolution API (via n8n)
- **Chat IA:** OpenAI GPT-4o-mini (via n8n)

### Fluxo de Dados

```
CLIENTE                    APP (React)                SUPABASE              n8n                    EXTERNO
  |                           |                          |                   |                       |
  |--- Abre cardapio -------->|                          |                   |                       |
  |                           |--- GET daily_menu ------>|                   |                       |
  |                           |--- GET paes_disponiveis->|                   |                       |
  |                           |--- GET confeitaria ----->|                   |                       |
  |                           |<-- dados do cardapio ----|                   |                       |
  |<-- mostra cardapio -------|                          |                   |                       |
  |                           |                          |                   |                       |
  |--- Monta marmita -------->|                          |                   |                       |
  |--- Finaliza pedido ------>|                          |                   |                       |
  |                           |--- Valida (Edge Func) -->|                   |                       |
  |                           |                          |                   |                       |
  |                           |--- POST webhook ---------|------------------>|                       |
  |                           |   (mp-criar-preference)  |                   |--- Cria Preference -->| MP
  |                           |                          |                   |<-- preferenceId ------|
  |                           |                          |                   |--- WhatsApp -------->| Evolution
  |                           |<-- {preferenceId, url} --|-------------------|                       |
  |                           |                          |                   |                       |
  |--- Redireciona checkout ->|------------------------------------------------->| Mercado Pago     |
  |--- Paga ----------------------------------------------------------->|       |                   |
  |                           |                          |               |       |                   |
  |                           |                          |    MP webhook |------>|                    |
  |                           |                          |    (mp-notificacao)   |                    |
  |                           |                          |<-- PATCH status ------|                    |
  |                           |                          |<-- INSERT financial --|                    |
  |                           |                          |<-- INSERT print_queue-|                    |
  |                           |                          |                   |--- WhatsApp confirm ->| Evolution
  |<-- Pedido confirmado -----|                          |                   |                       |
```

### Dependencias Externas

| Servico | Papel | Se cair... |
|---------|-------|------------|
| **Supabase** | Banco de dados (cardapio, pedidos, precos) | App NAO funciona |
| **n8n** | Pagamentos (cria preference + processa webhook MP) | PAGAMENTOS param. App mostra cardapio mas nao vende |
| **Mercado Pago** | Processamento de pagamento | Nao aceita pagamentos |
| **Evolution API** | WhatsApp (confirmacoes + bot atendimento) | Sem confirmacao WhatsApp, bot para |
| **OpenAI** | IA para bot WhatsApp/Instagram | Bot nao responde |
| **Airtable** | Gestao de dados (cardapio, paes, confeitaria) | Dados param de atualizar (mas app continua com dados existentes) |
| **Google Sheets** | Legado (sendo substituido por Airtable) | Idem Airtable |

### Tabelas Supabase Usadas

| Tabela | Quem alimenta | Quem consome |
|--------|--------------|--------------|
| `daily_menu` | n8n (sync Sheets/Airtable) | App (cardapio do dia) + Bot IA |
| `paes_disponiveis` | n8n (sync Sheets/Airtable) | App (pagina paes) + Bot IA |
| `confeitaria_disponiveis` | n8n (sync Sheets/Airtable) | App (pagina confeitaria) + Bot IA |
| `menu_items` | Manual/Admin | App (marmita builder) |
| `meal_sizes` | Manual/Admin | App (tamanhos marmita) |
| `marmita_orders` | App (checkout) + n8n (status) | App (confirmacao) |
| `corporate_orders` | App (checkout corp) + n8n (status) | App (confirmacao) |
| `corporate_routes` | Manual | App (rotas /:slug) |
| `encomendas_products` | Manual | App (catalogo encomendas) |
| `encomendas_pedidos` | App (checkout) + n8n (status) | App (confirmacao) |
| `delivery_zones` | Manual | App (calculo frete) |
| `print_queue` | App + n8n | Sistema impressao interno |
| `financial_entries` | App + n8n | Gestao financeira |
| `app_config` | Manual | App (status loja aberta/fechada) |

### Workflows n8n Necessarios

| Workflow | Funcao | Criticidade |
|----------|--------|-------------|
| **MP preferencia** | Cria checkout Mercado Pago | CRITICO - sem ele nao vende |
| **MP notificacao** | Processa pagamento aprovado | CRITICO - sem ele nao confirma pagamento |
| **Resposta Whats** | Bot IA atendimento WhatsApp | MEDIO - atendimento automatico |
| **Emporio Cosi Instagram** | Bot IA atendimento Instagram | MEDIO - atendimento automatico |
| **Consultar Cardapio (tool)** | Sub-workflow chamado pelos bots | MEDIO - bots precisam dele |
| **Guardrail-Hub** | Seguranca dos bots | MEDIO - protecao contra abuso |
| **Sync Cardapio** (a migrar) | Sheets/Airtable → Supabase | BAIXO - dados existentes continuam |
| **Sync Padaria** (a migrar) | Sheets/Airtable → Supabase | BAIXO |
| **Sync Confeitaria** (a migrar) | Sheets/Airtable → Supabase | BAIXO |

---

## 2. MARMITARIAARARAS (Marmitaria Araras)

### Stack
- **Frontend + Backend:** Next.js 16 (App Router, Server Components)
- **Hosting:** Vercel (marmitariaararas.com.br)
- **Database:** Supabase (PostgreSQL) — MESMO do Cosi
- **Pagamento:** Mercado Pago SDK (DIRETO no codigo)
- **Automacao:** n8n (APENAS notificacao — opcional)
- **WhatsApp:** Evolution API (via n8n, para notificacoes)
- **Chat IA:** Google Gemini (via n8n, bot Instagram)
- **Admin:** Painel /admin integrado no app

### Fluxo de Dados

```
CLIENTE                    APP (Next.js)              SUPABASE              n8n               EXTERNO
  |                           |                          |                   |                   |
  |--- Abre cardapio -------->|                          |                   |                   |
  |                           |--- GET menu_items ------>|                   |                   |
  |                           |    (business_unit=       |                   |                   |
  |                           |     'marmitaria')        |                   |                   |
  |                           |<-- dados do cardapio ----|                   |                   |
  |<-- mostra cardapio -------|                          |                   |                   |
  |                           |                          |                   |                   |
  |--- Monta pedido --------->|                          |                   |                   |
  |--- Finaliza pedido ------>|                          |                   |                   |
  |                           |--- POST /api/checkout -->|                   |                   |
  |                           |   (valida precos server) |                   |                   |
  |                           |--- INSERT marmita_orders>|                   |                   |
  |                           |--- INSERT print_queue -->|                   |                   |
  |                           |--- INSERT financial ---->|                   |                   |
  |                           |                          |                   |                   |
  |                           |--- Cria Preference ------|-------------------|------------------>| MP
  |                           |<-- preferenceId ---------|-------------------|-------------------|
  |                           |                          |                   |                   |
  |                           |--- POST webhook (async)->|------------------>|                   |
  |                           |  (marmitaria-pagamento-  |                   |--- WhatsApp ---->| Evolution
  |                           |   confirmado) OPCIONAL   |                   |                   |
  |                           |                          |                   |                   |
  |--- Redireciona checkout ->|------------------------------------------------------>| MP      |
  |--- Paga ---------------------------------------------------------->|            |          |
  |                           |                          |              |            |          |
  |                           |<-- MP webhook POST ------|--- /api/mp-webhook -------|          |
  |                           |--- PATCH marmita_orders->|              |            |          |
  |                           |--- PATCH financial ----->|              |            |          |
  |<-- Pedido confirmado -----|                          |              |            |          |
```

### Dependencias Externas

| Servico | Papel | Se cair... |
|---------|-------|------------|
| **Supabase** | Banco de dados (cardapio, pedidos, precos) | App NAO funciona |
| **Mercado Pago** | Processamento de pagamento (SDK direto) | Nao aceita pagamentos |
| **Vercel** | Hosting + serverless functions | App offline |
| **n8n** | Notificacao WhatsApp de novos pedidos (OPCIONAL) | App continua vendendo, so nao notifica por WhatsApp |
| **Evolution API** | WhatsApp (notificacoes) | Sem notificacao, mas app funciona |
| **Google Gemini** | IA para bot Instagram | Bot Instagram nao responde |

### Tabelas Supabase Usadas

| Tabela | Quem alimenta | Quem consome |
|--------|--------------|--------------|
| `menu_items` | Admin /admin (direto no Supabase) | App (cardapio, filtrado por business_unit='marmitaria') |
| `menu_groups` | Admin /admin | App (grupos de complementos) |
| `menu_additions` | Admin /admin | App (adicionais) |
| `marmita_orders` | App /api/checkout + /api/mp-webhook | App (confirmacao) |
| `print_queue` | App /api/checkout | Sistema impressao interno |
| `financial_entries` | App /api/checkout + /api/mp-webhook | Gestao financeira |
| `delivery_config` | Admin /admin | App (calculo frete) |

### Workflows n8n Necessarios

| Workflow | Funcao | Criticidade |
|----------|--------|-------------|
| **Marmitaria - Notificacao Pedido** | Notifica loja via WhatsApp | BAIXO - app funciona sem |
| **Marmitaria - Instagram** | Bot IA no Instagram | MEDIO - atendimento automatico |
| **Marketing (5 workflows)** | Boas-vindas, broadcast, despertador, carrinho, follow-up | BAIXO - marketing automatizado |

---

## 3. COMPARACAO DIRETA

| Aspecto | COSIARARAS | MARMITARIAARARAS |
|---------|-----------|------------------|
| **Framework** | React + Vite | Next.js 16 |
| **Hosting** | Vercel | Vercel |
| **Supabase** | Mesmo banco | Mesmo banco |
| **Pagamento** | Via n8n (webhook) | Direto no codigo (SDK) |
| **Se n8n cair** | NAO VENDE | Continua vendendo |
| **Admin de precos** | Nao tem (via Sheets/Airtable) | Tem (/admin) |
| **Bot WhatsApp** | n8n + OpenAI | n8n + Gemini |
| **Bot Instagram** | n8n + OpenAI | n8n + Gemini |
| **Cardapio do dia** | Supabase (alimentado por Sheets/Airtable via n8n) | Supabase (alimentado pelo /admin) |
| **Autonomia** | BAIXA (depende muito de n8n) | ALTA (funciona sozinho) |

### Compartilhamento de Recursos

```
                         COMPARTILHADO
                    +--------------------+
                    |                    |
                    |   SUPABASE         |
                    |   (mesmo banco)    |
                    |                    |
                    |   Separado por:    |
                    |   - business_unit  |
                    |   - source         |
                    |   - target         |
                    |                    |
                    +--------+-----------+
                             |
              +--------------+--------------+
              |                             |
         COSIARARAS                  MARMITARIAARARAS
    business_unit='cosi'        business_unit='marmitaria'
    source='corp/encomendas'    source='marmitaria_araras'
    target='cosi'               target='marmitaria'


                         COMPARTILHADO
                    +--------------------+
                    |                    |
                    |   MERCADO PAGO     |
                    |   (mesma conta)    |
                    |                    |
                    |   Separado por:    |
                    |   external_ref:    |
                    |   MARM- / CORP-    |
                    |   ENC- / uuid      |
                    |                    |
                    +--------+-----------+
                             |
              +--------------+--------------+
              |                             |
         COSIARARAS                  MARMITARIAARARAS
    Pagamento via n8n           Pagamento direto (SDK)
    (mp-criar-preference)       (/api/checkout)


                         COMPARTILHADO
                    +--------------------+
                    |                    |
                    |   N8N              |
                    |   (mesma instancia)|
                    |                    |
                    +--------+-----------+
                             |
              +--------------+--------------+
              |                             |
         COSIARARAS                  MARMITARIAARARAS
    - MP preferencia            - Notificacao pedido
    - MP notificacao            - Instagram bot
    - WhatsApp bot              - Marketing (5 wfs)
    - Instagram bot
    - Sync cardapio
    - Estoque
    - Router central


                    NAO COMPARTILHADO
              +--------------+--------------+
              |                             |
         COSIARARAS                  MARMITARIAARARAS
    - Airtable (gestao)        - Painel /admin
    - Google Sheets (legado)   - Meta Pixel
    - Chatwoot (CRM)           - Next.js API routes
    - Sentry (monitoring)
```

---

## 4. MAPA DE SERVICOS EXTERNOS

```
+------------------+     +------------------+     +------------------+
|   AIRTABLE       |     |   GOOGLE SHEETS  |     |   EVOLUTION API  |
|   (Gestao Cosi)  |     |   (Legado Cosi)  |     |   (WhatsApp)     |
|                  |     |                  |     |                  |
|  - Cardapio dia  |     |  - Cardapio dia  |     |  - Bot Cosi      |
|  - Paes          |     |  - Paes          |     |  - Bot Marmitaria|
|  - Confeitaria   |     |  - Confeitaria   |     |  - Notificacoes  |
|  - Precos        |     |  - Precos        |     |  - Alertas       |
|  - Estoque       |     +--------+---------+     +--------+---------+
|  - Bebidas       |              |                         |
|  - Fornecedores  |              |                         |
+--------+---------+     MIGRANDO PARA AIRTABLE             |
         |                        |                         |
         +------------------------+-------------------------+
                                  |
                          +-------+-------+
                          |     N8N       |
                          | (orquestrador)|
                          +-------+-------+
                                  |
                    +-------------+-------------+
                    |                           |
              +-----+-----+             +------+------+
              |  SUPABASE  |             | MERCADO PAGO|
              | (PostgreSQL)|            | (pagamentos)|
              +-----+------+             +------+------+
                    |                           |
              +-----+-----+             +------+------+
              | COSIARARAS |             |MARMITARIA   |
              | (React)    |             |ARARAS       |
              +------------+             |(Next.js)    |
                                         +-------------+
```

---

## 5. STATUS ATUAL DOS WORKFLOWS (22/03/2026)

### ATIVOS E TESTADOS
| Workflow | Negocio | Exec | Status |
|----------|---------|------|--------|
| Consultar Cardapio (tool) | Cosi | OK | Funcionando |
| Resposta Whats | Cosi | #5741 OK | Funcionando |
| Emporio Cosi - Instagram | Cosi | #5742 ERRO | Token IG expirado |
| Marmitaria - Notificacao | Marmitaria | #5744 OK | Funcionando |

### INATIVOS - FUNCIONAIS
| Workflow | Negocio | Funcao |
|----------|---------|--------|
| MP notificacao | Ambos | Confirma pagamentos (CORRIGIDO hoje) |
| MP preferencia | Cosi | Cria checkout MP (CORRIGIDO hoje) |
| WF-Central WhatsApp Router | Cosi | Router gestao interna |
| WF4 Estoque WhatsApp | Cosi | Estoque via WhatsApp |
| Guardrail-Hub | Ambos | Seguranca dos bots |
| Marmitaria Instagram | Marmitaria | Bot IA Instagram |
| 5x Marketing Marmitaria | Marmitaria | Boas-vindas, broadcast, etc |

### QUEBRADOS (URL antiga / Google Sheets)
| Workflow | Problema | Acao |
|----------|---------|------|
| busca cardapio | Orfao + URL antiga | Deletar |
| Padaria | Sheets + URL antiga | Migrar Airtable |
| Confeitaria | Sheets + URL antiga | Migrar Airtable |
| Consultar Cardapio (cron) | Sheets + URL antiga | Migrar Airtable |
| Sync Precos Sheets | Sheets | Migrar Airtable |

### NAO ANALISADOS (20 workflows)
Ver RELATORIO_WORKFLOWS_N8N_2026-03-22.md para lista completa

---

## 6. PENDENCIAS

### URGENTE
- [ ] Token Instagram Cosi — gerar no Meta Business Suite
- [ ] Token Instagram Marmitaria — verificar se expirado
- [ ] ADMIN_PASSWORD marmitariaararas — trocar no Vercel (esta "trocar-antes-de-publicar")

### ESTA SEMANA
- [ ] Migrar sync Cardapio/Padaria/Confeitaria de Sheets para Airtable
- [ ] Recriar workflows Airtable → Supabase
- [ ] Ativar MP notificacao + MP preferencia
- [ ] Analisar 20 workflows restantes

### ESTE MES
- [ ] Mover tokens hardcoded para credentials n8n
- [ ] Implementar rate limiting na marmitariaararas
- [ ] Validar assinatura webhook MP na marmitariaararas

---

**Documento:** ARQUITETURA_APPS.md
**Criado:** 22/03/2026
**Autor:** Claude Code
