# AJA Metrics — Dashboard Aquisição

Dashboard executivo de métricas comerciais da AJA Educação. HTML estático hospedado no GitHub Pages, com dados em tempo real via Cloudflare Worker + Supabase.

---

## Visão geral

Painel de aquisição e retenção para o CEO, com filtro de período personalizável e comparativo automático com o intervalo anterior. Os dados são buscados em tempo real de dois projetos Supabase, passando por um proxy seguro no Cloudflare Worker — as chaves de API nunca ficam expostas no código público.

---

## Arquitetura

```
Browser do CEO
    │
    └── GitHub Pages (index.html)
            │
            └── Cloudflare Worker (proxy seguro)
                    ├── /api/comercial → Supabase Comercial
                    └── /api/hub       → Supabase Hub-mentorada
```

As chaves do Supabase ficam armazenadas como **Secrets** no Cloudflare Worker — invisíveis para qualquer pessoa que inspecione o código-fonte do dashboard.

---

## Métricas disponíveis

### Aquisição — Comercial

| Métrica | Definição |
|---|---|
| Receita Total | Soma de Vendas Novas + Renovações + Ascensões |
| Receita Vendas Novas | Faturamento de novos negócios fechados |
| Vendas Novas | Total de negócios com estágio Fechado Ganho / Ganho |
| Leads Novos | Total de novos deals cadastrados no período |
| Taxa de Conversão | Vendas Novas ÷ Leads Novos |

### CS — Renovações e Ascensões

| Métrica | Definição |
|---|---|
| Receita Renovações | Faturamento de contratos renovados |
| Renovações | Quantidade de renovações no período |
| Receita Ascensões | Faturamento de upgrades de produto |
| Ascensões | Quantidade de upgrades no período |

### Canais de Aquisição

Distribuição das vendas fechadas por `utm_medium`:

| Canal | Critério |
|---|---|
| Pago | `utm_medium = paid` ou `cpc` |
| Orgânico | `utm_medium = social` |
| Sem Rastreamento | Demais valores ou vazio |

> **Em breve:** CAC e ROAS — aguardando integração da tabela Meta Ads no Supabase.

---

## Estrutura do projeto

```
/
├── index.html      # Dashboard completo (HTML + CSS + JS em arquivo único)
├── worker.js       # Código do Cloudflare Worker (referência — deploy feito no painel Cloudflare)
└── README.md       # Este arquivo
```

---

## Cloudflare Worker

### O que é

Um servidor intermediário gratuito que fica entre o dashboard e o Supabase. Ele recebe as requisições do browser, adiciona as chaves secretas e repassa ao Supabase. As chaves nunca chegam ao browser.

### URL do Worker

```
https://aja-metrics-api.ferramentas-931.workers.dev
```

### Rotas disponíveis

| Rota | Descrição |
|---|---|
| `/api/comercial` | Proxy para o projeto Supabase Comercial |
| `/api/hub` | Proxy para o projeto Supabase Hub-mentorada |
| `/health` | Health check — retorna `{"status":"ok"}` |

### Variáveis de ambiente (Secrets)

Configuradas em **Cloudflare → Workers & Pages → aja-metrics-api → Settings → Variables and Secrets**:

| Nome | Descrição |
|---|---|
| `COMERCIAL_URL` | URL do projeto Supabase Comercial |
| `COMERCIAL_KEY` | Chave anon do projeto Supabase Comercial |
| `HUB_URL` | URL do projeto Supabase Hub-mentorada |
| `HUB_KEY` | Chave anon do projeto Supabase Hub-mentorada |

### Segurança

- Lista branca de tabelas — só as 3 tabelas do projeto são acessíveis
- CORS restrito ao domínio `https://ferramentasaja.github.io`
- Apenas método GET permitido
- Chaves criptografadas — nunca visíveis após salvas

### Plano gratuito

100.000 requisições/dia. O dashboard faz ~20 requisições por acesso (paginação de 19.944 registros). Limite praticamente inalcançável no uso atual.

---

## Atualização automática

O dashboard não usa intervalo fixo. Ele calcula o tempo exato até o próximo horário programado:

| Horário | Descrição |
|---|---|
| 07:00 | Atualização matinal |
| 11:00 | Atualização do meio-dia |
| 16:00 | Atualização da tarde |
| 23:00 | Atualização noturna |

A próxima atualização é exibida no rodapé do dashboard.

---

## Fontes de dados

### Projeto Comercial

**Tabela `deal_webhook_comercial`**
- Campos: `id_negocio`, `estagio`, `montante`, `dtcadastro`, `dtfechamento`
- Leads = registros cadastrados no período (`dtcadastro`)
- Vendas = registros com `estagio IN ('Fechado Ganho', 'Ganho')` e `dtfechamento` no período

**Tabela `deal_utm_dim`**
- Campos: `id_negocio`, `utm_medium`
- Usada para classificar vendas por canal de origem

> **Atenção:** `dtcadastro` está no formato `DD-MM-YYYY HH:MM:SS`. A filtragem é feita no client-side com parser de data robusto.

> **Paginação:** a tabela tem mais de 19.000 registros. O sistema busca em páginas de 1.000 registros para garantir que todos os dados sejam carregados.

### Projeto Hub-mentorada

**Tabela `receita CS - Renovações e Ascensões [BASE TOTAL]`**
- Campos: `tipo_venda`, `montante_numerico`, `data_referencia_fechamento_date`, `ano`, `assinatura`
- Filtros aplicados: `ano ≠ 1899` (bug de conversão de data) e `assinatura = 'Assinado'`

---

## Deploy — GitHub Pages

### Primeiro deploy

1. Faça upload do `index.html` no repositório
2. Vá em **Settings → Pages**
3. Em **Source**, selecione `Deploy from a branch`
4. Branch: `main` / Pasta: `/ (root)`
5. Salve — a URL é gerada em alguns minutos

### Atualizar o dashboard

```bash
git add index.html
git commit -m "atualização dashboard"
git push
```

O GitHub Pages atualiza automaticamente em ~1 minuto após o push.

---

## Atualizar o Worker

Para editar o código do Worker:

1. Acesse **cloudflare.com → Workers & Pages → aja-metrics-api**
2. Clique em **Edit code**
3. Faça as alterações
4. Clique em **Deploy**

Para atualizar as chaves do Supabase:

1. Acesse **Settings → Variables and Secrets**
2. Clique no ícone de edição ao lado da variável
3. Insira o novo valor e clique em **Deploy**

---

## Roadmap

- [x] Métricas de aquisição (Leads, Vendas, Receita, Conversão)
- [x] Métricas de CS (Renovações, Ascensões)
- [x] Canais por utm_medium
- [x] Filtro de período personalizável
- [x] Comparativo automático com período anterior
- [x] Tooltips com definição de cada métrica
- [x] Atualização automática (7h, 11h, 16h, 23h)
- [x] Cloudflare Worker — chaves do Supabase protegidas
- [x] Layout responsivo (mobile, tablet, desktop)
- [ ] CAC — aguardando tabela Meta Ads no Supabase
- [ ] ROAS — aguardando tabela Meta Ads no Supabase
- [ ] NPS — métricas em definição
- [ ] Filtro por Closer / SDR

---

## Elaborado por

Área de Tecnologia e Operações — AJA Educação
Dados: Zoho CRM + Supabase · Proxy: Cloudflare Worker
