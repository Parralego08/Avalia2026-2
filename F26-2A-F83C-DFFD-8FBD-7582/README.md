# Projeto #14: Diário de Bordo de Projetos

## Visão Geral

**Avalia2026** é uma aplicação web de linha do tempo para registro diário de atividades de projetos, com autenticação exclusiva via **Discord OAuth2**. Cada usuário registra suas atividades diárias, classifica impedimentos técnicos e visualiza uma linha do tempo cronológica com filtros de busca, impedimento e intervalo de datas.

O projeto foi construído sem frameworks de UI: **HTML5, CSS3 e JavaScript puro** no frontend, **Cloudflare Pages Functions** no backend e **Cloudflare D1 (SQLite)** como banco relacional.

---

## 🔑 Chave Única e Deploy

- **Chave Única:** `F26-2A-F83C-DFFD-8FBD-7582`
- **URL_CLOUDFLARE:** `https://diario-bordo-f26.pages.dev` *(substitua pela URL real gerada pela Cloudflare caso o nome do projeto Pages seja diferente)*
- A chave também está visível no rodapé da aplicação (`<footer>` em `public/index.html`).

> **Observação sobre o prefixo da chave:** a chave oficial emitida para este projeto inicia com `F26-`. O script `validar-local.mjs` do repositório atualmente aceita os prefixos `P26`/`T26`; caso ele recuse o prefixo `F26` ao rodar a validação local, reporte ao professor antes de abrir o Pull Request. Nenhum arquivo fora desta subpasta foi alterado.

---

## ⚙️ Funcionalidades

1. **Login exclusivo via Discord OAuth2** (`identify`), com parâmetro `state` anti-CSRF e sessão em cookie assinado.
2. **Registro de atividades** com conteúdo textual (até 2000 caracteres) e classificação de impedimento (`sem`, `atencao`, `bloqueado`).
3. **Timestamp UTC gerado exclusivamente no servidor** — o cliente nunca envia datas.
4. **Linha do tempo vertical** em CSS puro, ordenada da mais recente para a mais antiga, com marcadores coloridos por impedimento.
5. **Filtros no backend** por palavra-chave (`busca`), `impedimento`, `inicio` e `fim` (intervalo inclusivo).
6. **Privacidade:** cada usuário visualiza apenas os próprios registros (filtro obrigatório por `user_id` no servidor).
7. **Logout real** sem sair da aplicação, com limpeza do cookie de sessão.

---

## 🏗️ Arquitetura

### Stack Tecnológica

| Camada | Tecnologia |
|--------|------------|
| **Frontend** | HTML5, CSS3, JavaScript (Vanilla/ES6+) — **sem frameworks UI** |
| **Backend** | Cloudflare Pages Functions (Workers runtime) |
| **Banco de Dados** | Cloudflare D1 (SQLite) |
| **Autenticação** | Discord OAuth2 (exclusivo) |
| **Deploy** | Cloudflare Pages |

### Segurança e Integridade

- **Sessão assinada com HMAC-SHA256** via Web Crypto API (`lib/session.js`), contendo `user_id`, `username`, `avatar`, `iat` e `exp`. Cookies não podem ser forjados ou adulterados pelo cliente.
- **Cookie `HttpOnly`, `SameSite=Lax` e `Secure` em HTTPS**, com expiração de 24 horas.
- **Proteção CSRF no OAuth2** com `state` aleatório armazenado em cookie de curta duração e validado no callback.
- **Queries parametrizadas** no D1 (`.prepare().bind()`), prevenindo SQL Injection.
- **Validação no servidor** de `conteudo` (obrigatório, não vazio, limite de 2000 caracteres) e `impedimento` (enumeração fechada).
- **Isolamento de dados** por `user_id` em todas as leituras.

### Estrutura de Ficheiros

```
F26-2A-F83C-DFFD-8FBD-7582/
├── public/
│   ├── index.html          # HTML semântico + footer com a Chave Única
│   ├── style.css           # CSS moderno (variáveis, timeline, responsividade)
│   └── script.js           # Vanilla JS (IIFE, fetch, DOM direto)
├── schema.sql              # Schema D1 + índice
├── wrangler.toml           # Config Cloudflare Pages + binding D1
├── lib/
│   └── session.js          # Sessão assinada HMAC-SHA256 (Web Crypto)
├── functions/
│   └── api/
│       ├── auth/
│       │   ├── login.js    # Inicia OAuth2 Discord (state anti-CSRF)
│       │   ├── callback.js # Callback OAuth2, valida state e cria sessão
│       │   └── me.js       # Valida sessão, retorna perfil e logout
│       └── logs.js         # POST/GET de logs com validação e filtros
├── tests/
│   └── api.test.mjs        # Testes automatizados (node:test)
├── skills/                 # Skills dos agentes (mínimo 3)
│   ├── cloudflare-backend/SKILL.md
│   ├── vanilla-frontend/SKILL.md
│   └── auditoria-qa/SKILL.md
├── .opencode/              # Agentes/skills do harness OpenCode
├── .dev.vars.example       # Exemplo de variáveis locais
├── AGENTS.md               # Documentação dos agentes e harness
├── LICENSE                 # MIT License
└── README.md               # Este ficheiro
```

---

## 🔌 Contrato da API

### `GET /api/auth/login`
Redireciona (302) para `https://discord.com/api/oauth2/authorize` com `state` anti-CSRF.

### `GET /api/auth/callback`
Valida `state`, troca o `code` pelo `access_token`, consulta `/users/@me` e cria a sessão assinada. Em erro, redireciona para `/?auth_error=<motivo>`.

### `GET /api/auth/me`
**Resposta (200):**
```json
{
  "user_id": "123456789012345678",
  "username": "usuario_discord",
  "avatar": "a_abc123hash"
}
```
Sem sessão válida: **401**. Com `?logout=1`: limpa o cookie (`Max-Age=0`).

### `POST /api/logs`
**Body:**
```json
{ "conteudo": "Implementei a assinatura HMAC da sessão", "impedimento": "sem" }
```
**Resposta (201):**
```json
{ "success": true, "id": 42, "criado_em": "2026-11-14T14:30:00.000Z" }
```
Erros: **400** (payload inválido), **401** (sem sessão), **503** (D1 não configurado).

### `GET /api/logs`
**Query params opcionais:** `busca`, `impedimento`, `inicio` (`YYYY-MM-DD`), `fim` (`YYYY-MM-DD` — inclusivo até `23:59:59.999Z`).

**Resposta (200):**
```json
[
  {
    "id": 42,
    "user_id": "123456789012345678",
    "username": "usuario_discord",
    "avatar": "a_abc123hash",
    "conteudo": "Implementei a assinatura HMAC da sessão",
    "impedimento": "sem",
    "criado_em": "2026-11-14T14:30:00.000Z"
  }
]
```

**Os nomes dos campos JSON coincidem exatamente com o schema SQL e o frontend:**
`user_id`, `username`, `avatar`, `conteudo`, `impedimento`, `criado_em`.

---

## 🗄️ Banco de Dados

```sql
CREATE TABLE IF NOT EXISTS logs (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id TEXT NOT NULL,
    username TEXT NOT NULL,
    avatar TEXT,
    conteudo TEXT NOT NULL,
    impedimento TEXT NOT NULL,
    criado_em TEXT NOT NULL
);

CREATE INDEX IF NOT EXISTS idx_logs_user_criado ON logs(user_id, criado_em DESC);
```

---

## 🧪 Testes Automatizados

A suíte usa apenas módulos nativos (`node:test` + `node:assert`), sem dependências externas:

```bash
node --test tests/api.test.mjs
```

Cobertura:
- rejeição de cookie de sessão adulterado/segredo incorreto;
- `401` sem sessão em `GET`/`POST /api/logs`;
- validação de `conteudo` vazio e `impedimento` inválido;
- timestamp UTC gerado no servidor (ignora `criado_em` do cliente);
- isolamento por `user_id`, busca textual e intervalo de datas inclusivo;
- validação de sessão e logout em `/api/auth/me`.

---

## 💻 Como Executar Localmente

```bash
# Pré-requisitos: Node.js >= 18, wrangler CLI, conta Cloudflare com D1
npm install -g wrangler
wrangler login

# Criar banco D1 local
wrangler d1 create diario-bordo-db
# Atualizar database_id em wrangler.toml

# Aplicar schema
wrangler d1 execute diario-bordo-db --local --file=schema.sql

# Configurar variáveis (Discord OAuth2)
# Copiar .dev.vars.example para .dev.vars e preencher:
# DISCORD_CLIENT_ID=...
# DISCORD_CLIENT_SECRET=...
# DISCORD_REDIRECT_URI=http://localhost:8788/api/auth/callback
# SESSION_SECRET=... (opcional; padrão usa DISCORD_CLIENT_SECRET)

# Iniciar dev server
wrangler pages dev
```

> **Dica para o laboratório (Windows + Scoop):** use `npx.cmd wrangler ...` para evitar bloqueio de scripts `.ps1` do PowerShell.

---

## 🚀 Deploy (Cloudflare Pages + D1 + Discord)

1. Em **Workers & Pages → Create → Pages → Connect to Git**, selecione o fork do repositório `Avalia2026-2`.
2. Configure:
   - **Root directory:** `F26-2A-F83C-DFFD-8FBD-7582`
   - **Build command:** *(em branco — HTML/CSS/JS puro)*
   - **Build output directory:** `public`
3. Crie o banco D1 e aplique o schema remoto:
   ```bash
   cd F26-2A-F83C-DFFD-8FBD-7582
   npx.cmd wrangler d1 create diario-bordo-db
   npx.cmd wrangler d1 execute diario-bordo-db --remote --file=schema.sql
   ```
4. No painel do Pages → **Settings → Functions → D1 database bindings**, adicione:
   - **Variable name:** `DB`
   - **D1 database:** `diario-bordo-db`
5. Em **Settings → Environment variables**, adicione:
   - `DISCORD_CLIENT_ID`
   - `DISCORD_CLIENT_SECRET`
   - `DISCORD_REDIRECT_URI` = `https://<seu-projeto>.pages.dev/api/auth/callback`
   - `SESSION_SECRET` *(recomendado; segredo aleatório forte)*
6. No [Discord Developer Portal](https://discord.com/developers/applications), cadastre o redirect exato acima em **OAuth2 → Redirects**.
7. Atualize `URL_CLOUDFLARE` neste README com a URL pública real.
8. Valide localmente e envie:
   ```bash
   node validar-local.mjs F26-2A-F83C-DFFD-8FBD-7582
   node --test F26-2A-F83C-DFFD-8FBD-7582/tests/api.test.mjs
   git add F26-2A-F83C-DFFD-8FBD-7582
   git commit -m "Entrega F26-2A-F83C-DFFD-8FBD-7582"
   git push origin main
   ```

---

## ✅ Conformidade com a Rubrica

| Requisito | Status | Evidência |
|-----------|--------|-----------|
| **1. Deploy Online no Cloudflare** | ⏳ | URL em `URL_CLOUDFLARE` (validar após o deploy) |
| **2. Serverless (Functions + D1 + Discord OAuth2)** | ✅ | `functions/api/*`, `wrangler.toml` (binding `DB`), `schema.sql`, `lib/session.js` |
| **3. Frontend HTML5/CSS3/JS Vanilla** | ✅ | `public/index.html`, `public/style.css`, `public/script.js` sem frameworks |
| **4. Engenharia de Agentes (3 + Teste + Harness + Skills)** | ✅ | `AGENTS.md` + `skills/*/SKILL.md` (3 skills) + `tests/api.test.mjs` |
| **5. Repositório, Licença MIT e Auditoria** | ✅ | `LICENSE`, Chave Única no `README.md` e no `<footer>` |
| **Sem framework UI** | ✅ | Apenas HTML/CSS/JS puro; nenhuma dependência de UI |
| **Login exclusivo Discord OAuth2** | ✅ | `functions/api/auth/login.js`, `callback.js` e `me.js` |
| **`new Date().toISOString()` só no servidor (UTC)** | ✅ | `functions/api/logs.js` — frontend apenas formata datas |
| **Footer com a chave `F26-2A-F83C-DFFD-8FBD-7582`** | ✅ | `public/index.html` |
| **Campos `user_id`, `username`, `avatar`, `conteudo`, `impedimento`, `criado_em`** | ✅ | API, schema e `public/script.js` alinhados |

---

## 📄 Licença

MIT License — ver ficheiro [`LICENSE`](./LICENSE).

---

**Chave Única:** `F26-2A-F83C-DFFD-8FBD-7582`
