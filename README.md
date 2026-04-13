# AJA Metrics — Dashboard Comercial

Dashboard de métricas comerciais da AJA Educação, construído em HTML estático com conexão direta ao Supabase via JavaScript. Hospedado no GitHub Pages.

---

## Visão geral

Painel executivo de aquisição e retenção, atualizado automaticamente a cada hora, com filtro de período personalizável e comparativo automático com intervalo anterior.

Os dados são puxados em tempo real de dois projetos Supabase:

- **Comercial** — pipeline de leads e vendas (Zoho CRM)
- **Hub Mentorada** — renovações e ascensões (CS)

---

## Métricas disponíveis

### Aquisição — Comercial

| Métrica | Definição |
|---|---|
| Receita Total | Soma de Vendas Novas + Renovações + Ascensões |
| Receita Vendas Novas | Faturamento de novos negócios fechados |
| Vendas Novas | Total de negócios com estágio Fechado Ganho |
| Leads Novos | Total de novos deals cadastrados no período |
| Taxa de Conversão | Vendas Novas ÷ Leads Novos |

### CS — Renovações e Ascensões

| Métrica | Definição |
|---|---|
| Receita Renovações | Faturamento de contratos renovados |
| Renovações | Quantidade de renovações no período |
| Receita Ascensões | Faturamento de upgrades de produto |
| Ascensões | Quantidade de upgrades no período |

### Vendas por Canal

Distribuição das vendas fechadas por `utm_medium`:

| Canal | Critério |
|---|---|
| Pago | `utm_medium = paid` |
| Orgânico | `utm_medium = organic` |
| Sem Rastreamento | Demais valores ou vazio |

> **Em breve:** CAC e ROAS — aguardando integração da tabela Meta Ads no Supabase.

---

## Estrutura do projeto

```
/
├── index.html          # Dashboard completo (HTML + CSS + JS em arquivo único)
└── README.md           # Este arquivo
```

O projeto é intencionalmente simples: **um único arquivo HTML**, sem frameworks, sem build, sem dependências locais.

---

## Fontes de dados

### Projeto Comercial (`deal_webhook_comercial` + `deal_utm_dim`)

- `deal_webhook_comercial` — estágio, montante, datas de cadastro e fechamento, closer, SDR
- `deal_utm_dim` — parâmetros UTM (source, medium, campaign) vinculados por `id_negocio`

> **Atenção:** o campo `dtcadastro` está no formato `DD-MM-YYYY HH:MM:SS`. A filtragem de datas é feita no client-side (JavaScript) para lidar com esse formato não-ISO.

### Projeto Hub Mentorada (`receita CS - Renovações e Ascensões [BASE TOTAL]`)

- Colunas usadas: `tipo_venda`, `montante_numerico`, `data_referencia_fechamento_date`, `ano`
- Registros com `ano = 1899` são descartados automaticamente (bug de conversão de data na origem)

---

## Como funciona

```
Browser do CEO
    │
    ├── Carrega index.html (GitHub Pages)
    │
    ├── Cria 2 clientes Supabase (Comercial + Hub)
    │
    ├── Busca todos os registros das tabelas relevantes
    │
    ├── Filtra por período no JavaScript
    │
    ├── Calcula métricas e deltas
    │
    └── Renderiza os cards — atualiza a cada 1 hora
```

Não há backend. Todas as operações acontecem no browser do usuário.

---

## Segurança

O dashboard usa a **chave `anon` (pública)** do Supabase — projetada para acesso de leitura no client-side.

**Medidas obrigatórias no Supabase:**

- [ ] Ativar **RLS (Row Level Security)** nas tabelas expostas
- [ ] Criar policies de **somente leitura** (`SELECT`) para o role `anon`
- [ ] Garantir que nenhuma tabela sensível esteja exposta sem RLS

**Acesso ao dashboard:**

- Protegido por senha via `sessionStorage` (proteção básica, adequada para uso interno)
- O repositório GitHub **deve ser privado** para não expor as chaves `anon`

> Após qualquer exposição acidental das chaves, regenere-as em **Supabase → Settings → API → Regenerate**.

---

## Deploy — GitHub Pages

### Primeiro deploy

1. Faça upload do `index.html` no repositório
2. Vá em **Settings → Pages**
3. Em **Source**, selecione `Deploy from a branch`
4. Branch: `main` / Pasta: `/ (root)`
5. Salve — o GitHub gera a URL em alguns minutos

### Atualizar o dashboard

```bash
# Substituir o arquivo e fazer push
git add index.html
git commit -m "atualização dashboard"
git push
```

O GitHub Pages atualiza automaticamente em ~1 minuto após o push.

---

## Configuração local (para desenvolvimento)

Não é necessário Node.js ou nenhum build tool. Basta abrir o arquivo diretamente:

```bash
# Opção 1 — abrir direto no browser
open index.html   # macOS
start index.html  # Windows

# Opção 2 — servidor local simples (evita CORS em alguns browsers)
python3 -m http.server 8080
# Acessar: http://localhost:8080
```

---

## Variáveis de configuração

Todas as configurações estão no bloco `CONFIG` no topo do `<script>` em `index.html`:

```javascript
const SENHA         = 'sua-senha-aqui';
const COMERCIAL_URL = 'https://xxxx.supabase.co';
const COMERCIAL_KEY = 'eyJ...';
const HUB_URL       = 'https://yyyy.supabase.co';
const HUB_KEY       = 'eyJ...';
```

---

## Roadmap

- [x] Métricas de aquisição (Leads, Vendas, Receita, Conversão)
- [x] Métricas de CS (Renovações, Ascensões)
- [x] Canais por utm_medium
- [x] Filtro de período personalizável
- [x] Comparativo automático com período anterior
- [x] Tooltips com definição de cada métrica
- [x] Auto-refresh a cada hora
- [ ] CAC — aguardando tabela Meta Ads no Supabase
- [ ] ROAS — aguardando tabela Meta Ads no Supabase
- [ ] NPS — métricas em definição
- [ ] Filtro por Closer / SDR
- [ ] Gráfico de evolução temporal

---

## Elaborado por

Área de Tecnologia e Operações — AJA Educação  
Dados: Zoho CRM + Meta Ads via Supabase
