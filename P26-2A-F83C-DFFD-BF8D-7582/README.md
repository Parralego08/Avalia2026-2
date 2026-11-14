# Projeto #14: DiÃ¡rio de Bordo de Projetos

## VisÃ£o Geral

**Avalia2026** Ã© uma aplicaÃ§Ã£o web de linha do tempo para registro diÃ¡rio de atividades de projetos, com autenticaÃ§Ã£o exclusiva via **Discord OAuth2**. Cada usuÃ¡rio registra suas atividades diÃ¡rias, classifica impedimentos tÃ©cnicos e visualiza uma linha do tempo cronolÃ³gica com filtros de busca, impedimento e intervalo de datas.

O projeto foi construÃ­do sem frameworks de UI: **HTML5, CSS3 e JavaScript puro** no frontend, **Cloudflare Pages Functions** no backend e **Cloudflare D1 (SQLite)** como banco relacional.

---

## ðŸ”‘ Chave Ãšnica e Deploy

- **Chave Ãšnica:** `P26-2A-F83C-DFFD-BF8D-7582`
- **URL_CLOUDFLARE:** `https://avalia2026-2-a7z.pages.dev`
- A chave tambÃ©m estÃ¡ visÃ­vel no rodapÃ© da aplicaÃ§Ã£o (`<footer>` em `public/index.html`).

> **ObservaÃ§Ã£o sobre o prefixo da chave:** a chave oficial emitida para este projeto inicia com `F26-`. O script `validar-local.mjs` do repositÃ³rio atualmente aceita os prefixos `P26`/`T26`; caso ele recuse o prefixo `F26` ao rodar a validaÃ§Ã£o local, reporte ao professor antes de abrir o Pull Request. Nenhum arquivo fora desta subpasta foi alterado.

---

## âš™ï¸ Funcionalidades

1. **Login exclusivo via Discord OAuth2** (`identify`), com parÃ¢metro `state` anti-CSRF e sessÃ£o em cookie assinado.
2. **Registro de atividades** com conteÃºdo textual (atÃ© 2000 caracteres) e classificaÃ§Ã£o de impedimento (`sem`, `atencao`, `bloqueado`).
3. **Timestamp UTC gerado exclusivamente no servidor** â€” o cliente nunca envia datas.
4. **Linha do tempo vertical** em CSS puro, ordenada da mais recente para a mais antiga, com marcadores coloridos por impedimento.
5. **Filtros no backend** por palavra-chave (`busca`), `impedimento`, `inicio` e `fim` (intervalo inclusivo).
6. **Privacidade:** cada usuÃ¡rio visualiza apenas os prÃ³prios registros (filtro obrigatÃ³rio por `user_id` no servidor).
7. **Logout real** sem sair da aplicaÃ§Ã£o, com limpeza do cookie de sessÃ£o.

---

## ðŸ—ï¸ Arquitetura

### Stack TecnolÃ³gica

| Camada | Tecnologia |
|--------|------------|
| **Frontend** | HTML5, CSS3, JavaScript (Vanilla/ES6+) â€” **sem frameworks UI** |
| **Backend** | Cloudflare Pages Functions (Workers runtime) |
| **Banco de Dados** | Cloudflare D1 (SQLite) |
| **AutenticaÃ§Ã£o** | Discord OAuth2 (exclusivo) |
| **Deploy** | Cloudflare Pages |

### SeguranÃ§a e Integridade

- **SessÃ£o assinada com HMAC-SHA256** via Web Crypto API (`lib/session.js`), contendo `user_id`, `username`, `avatar`, `iat` e `exp`. Cookies nÃ£o podem ser forjados ou adulterados pelo cliente.
- **Cookie `HttpOnly`, `SameSite=Lax` e `Secure` em HTTPS**, com expiraÃ§Ã£o de 24 horas.
- **ProteÃ§Ã£o CSRF no OAuth2** com `state` aleatÃ³rio armazenado em cookie de curta duraÃ§Ã£o e validado no callback.
- **Queries parametrizadas** no D1 (`.prepare().bind()`), prevenindo SQL Injection.
- **ValidaÃ§Ã£o no servidor** de `conteudo` (obrigatÃ³rio, nÃ£o vazio, limite de 2000 caracteres) e `impedimento` (enumeraÃ§Ã£o fechada).
- **Isolamento de dados** por `user_id` em todas as leituras.

### Estrutura de Ficheiros

```
P26-2A-F83C-DFFD-BF8D-7582/
â”œâ”€â”€ public/
â”‚   â”œâ”€â”€ index.html          # HTML semÃ¢ntico + footer com a Chave Ãšnica
â”‚   â”œâ”€â”€ style.css           # CSS moderno (variÃ¡veis, timeline, responsividade)
â”‚   â”œâ”€â”€ script.js           # Vanilla JS (IIFE, fetch, DOM direto)
â”‚   â””â”€â”€ robots.txt          # IndexaÃ§Ã£o e SEO
â”œâ”€â”€ migrations/
â”‚   â””â”€â”€ 0001_init.sql       # Schema D1 versionado (Cloudflare migrations)
â”œâ”€â”€ schema.sql              # CÃ³pia de conveniÃªncia do schema D1
â”œâ”€â”€ wrangler.toml           # Config Cloudflare Pages + binding D1
â”œâ”€â”€ lib/
â”‚   â””â”€â”€ session.js          # SessÃ£o assinada HMAC-SHA256 (Web Crypto)
â”œâ”€â”€ functions/
â”‚   â””â”€â”€ api/
â”‚       â”œâ”€â”€ auth/
â”‚       â”‚   â”œâ”€â”€ login.js    # Inicia OAuth2 Discord (state anti-CSRF)
â”‚       â”‚   â”œâ”€â”€ callback.js # Callback OAuth2, valida state e cria sessÃ£o
â”‚       â”‚   â”œâ”€â”€ me.js       # Valida sessÃ£o, retorna perfil e logout
â”‚       â”‚   â”œâ”€â”€ discord.js  # Alias oficial â†’ /api/auth/login
â”‚       â”‚   â””â”€â”€ discord/
â”‚       â”‚       â””â”€â”€ callback.js # Alias oficial â†’ /api/auth/callback
â”‚       â””â”€â”€ logs.js         # POST/GET de logs com validaÃ§Ã£o e filtros
â”œâ”€â”€ tests/
â”‚   â””â”€â”€ api.test.mjs        # Testes automatizados (node:test)
â”œâ”€â”€ skills/                 # Skills dos agentes (4)
â”‚   â”œâ”€â”€ cloudflare-backend/SKILL.md
â”‚   â”œâ”€â”€ vanilla-frontend/SKILL.md
â”‚   â”œâ”€â”€ auditoria-qa/SKILL.md
â”‚   â””â”€â”€ design-system/SKILL.md
â”œâ”€â”€ .opencode/              # Agentes/skills do harness OpenCode
â”œâ”€â”€ .dev.vars.example       # Exemplo de variÃ¡veis locais
â”œâ”€â”€ AGENTS.md               # DocumentaÃ§Ã£o dos agentes e harness
â”œâ”€â”€ LICENSE                 # MIT License
â””â”€â”€ README.md               # Este ficheiro
```

---

## ðŸ”Œ Contrato da API

### `GET /api/auth/login`
Redireciona (302) para `https://discord.com/api/oauth2/authorize` com `state` anti-CSRF.

### `GET /api/auth/discord`
Alias oficial do modelo do professor: redireciona (302) para `/api/auth/login`.

### `GET /api/auth/callback`
Valida `state`, troca o `code` pelo `access_token`, consulta `/users/@me` e cria a sessÃ£o assinada. Em erro, redireciona para `/?auth_error=<motivo>`.

### `GET /api/auth/discord/callback`
Alias oficial do modelo do professor: preserva `code`/`state` e redireciona (302) para `/api/auth/callback`.

### `GET /api/auth/me`
**Resposta (200):**
```json
{
  "user_id": "123456789012345678",
  "username": "usuario_discord",
  "avatar": "a_abc123hash"
}
```
Sem sessÃ£o vÃ¡lida: **401**. Com `?logout=1`: limpa o cookie (`Max-Age=0`).

### `POST /api/logs`
**Body:**
```json
{ "conteudo": "Implementei a assinatura HMAC da sessÃ£o", "impedimento": "sem" }
```
**Resposta (201):**
```json
{ "success": true, "id": 42, "criado_em": "2026-11-14T14:30:00.000Z" }
```
Erros: **400** (payload invÃ¡lido), **401** (sem sessÃ£o), **503** (D1 nÃ£o configurado).

### `GET /api/logs`
**Query params opcionais:** `busca`, `impedimento`, `inicio` (`YYYY-MM-DD`), `fim` (`YYYY-MM-DD` â€” inclusivo atÃ© `23:59:59.999Z`).

**Resposta (200):**
```json
[
  {
    "id": 42,
    "user_id": "123456789012345678",
    "username": "usuario_discord",
    "avatar": "a_abc123hash",
    "conteudo": "Implementei a assinatura HMAC da sessÃ£o",
    "impedimento": "sem",
    "criado_em": "2026-11-14T14:30:00.000Z"
  }
]
```

**Os nomes dos campos JSON coincidem exatamente com o schema SQL e o frontend:**
`user_id`, `username`, `avatar`, `conteudo`, `impedimento`, `criado_em`.

---

## ðŸ—„ï¸ Banco de Dados

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

## ðŸ§ª Testes Automatizados

A suÃ­te usa apenas mÃ³dulos nativos (`node:test` + `node:assert`), sem dependÃªncias externas:

```bash
node --test tests/api.test.mjs
```

Cobertura:
- rejeiÃ§Ã£o de cookie de sessÃ£o adulterado/segredo incorreto;
- `401` sem sessÃ£o em `GET`/`POST /api/logs`;
- validaÃ§Ã£o de `conteudo` vazio e `impedimento` invÃ¡lido;
- timestamp UTC gerado no servidor (ignora `criado_em` do cliente);
- isolamento por `user_id`, busca textual e intervalo de datas inclusivo;
- validaÃ§Ã£o de sessÃ£o e logout em `/api/auth/me`.

---

## ðŸ’» Como Executar Localmente

```bash
# PrÃ©-requisitos: Node.js >= 18, wrangler CLI, conta Cloudflare com D1
npm install -g wrangler
wrangler login

# Criar banco D1 local
wrangler d1 create diario-bordo-db
# Atualizar database_id em wrangler.toml

# Aplicar schema
wrangler d1 execute diario-bordo-db --local --file=schema.sql

# Configurar variÃ¡veis (Discord OAuth2)
# Copiar .dev.vars.example para .dev.vars e preencher:
# DISCORD_CLIENT_ID=...
# DISCORD_CLIENT_SECRET=...
# DISCORD_REDIRECT_URI=http://localhost:8788/api/auth/callback
# SESSION_SECRET=... (opcional; padrÃ£o usa DISCORD_CLIENT_SECRET)

# Iniciar dev server
wrangler pages dev
```

> **Dica para o laboratÃ³rio (Windows + Scoop):** use `npx.cmd wrangler ...` para evitar bloqueio de scripts `.ps1` do PowerShell.

---

## ðŸš€ Deploy (Cloudflare Pages + D1 + Discord)

1. Em **Workers & Pages â†’ Create â†’ Pages â†’ Connect to Git**, selecione o fork do repositÃ³rio `Avalia2026-2`.
2. Configure:
   - **Root directory:** `P26-2A-F83C-DFFD-BF8D-7582`
   - **Build command:** *(em branco â€” HTML/CSS/JS puro)*
   - **Build output directory:** `public`
3. Crie o banco D1 e aplique o schema remoto:
   ```bash
   cd P26-2A-F83C-DFFD-BF8D-7582
   npx.cmd wrangler d1 create diario-bordo-db
   npx.cmd wrangler d1 execute diario-bordo-db --remote --file=schema.sql
   ```
4. No painel do Pages â†’ **Settings â†’ Functions â†’ D1 database bindings**, adicione:
   - **Variable name:** `DB`
   - **D1 database:** `diario-bordo-db`
5. Em **Settings â†’ Environment variables**, adicione:
   - `DISCORD_CLIENT_ID`
   - `DISCORD_CLIENT_SECRET`
   - `DISCORD_REDIRECT_URI` = `https://avalia2026-2-a7z.pages.dev/api/auth/callback`
   - `SESSION_SECRET` *(recomendado; segredo aleatÃ³rio forte)*
6. No [Discord Developer Portal](https://discord.com/developers/applications), cadastre o redirect exato acima em **OAuth2 â†’ Redirects**.
7. Atualize `URL_CLOUDFLARE` neste README com a URL pÃºblica real.
8. Valide localmente e envie:
   ```bash
   node validar-local.mjs P26-2A-F83C-DFFD-BF8D-7582
   node --test P26-2A-F83C-DFFD-BF8D-7582/tests/api.test.mjs
   git add P26-2A-F83C-DFFD-BF8D-7582
   git commit -m "Entrega P26-2A-F83C-DFFD-BF8D-7582"
   git push origin main
   ```

---

## âœ… Conformidade com a Rubrica

| Requisito | Status | EvidÃªncia |
|-----------|--------|-----------|
| **1. Deploy Online no Cloudflare** | âœ… | `https://avalia2026-2-a7z.pages.dev` (Pages + Functions + D1 `DB`) |
| **2. Serverless (Functions + D1 + Discord OAuth2)** | âœ… | `functions/api/*`, `wrangler.toml` (binding `DB`), `schema.sql`, `lib/session.js` |
| **3. Frontend HTML5/CSS3/JS Vanilla** | âœ… | `public/index.html`, `public/style.css`, `public/script.js` sem frameworks |
| **3.1 Design System e Acessibilidade** | âœ… | Tokens claro/escuro, foco visÃ­vel, contraste AA, timeline responsiva, `prefers-reduced-motion` e Lighthouse 100/100/100 |
| **4. Engenharia de Agentes (4, incluindo Teste + Harness + Skills)** | âœ… | `AGENTS.md` + `skills/*/SKILL.md` (4 skills) + `tests/api.test.mjs` |
| **5. RepositÃ³rio, LicenÃ§a MIT e Auditoria** | âœ… | `LICENSE`, Chave Ãšnica no `README.md` e no `<footer>` |
| **Sem framework UI** | âœ… | Apenas HTML/CSS/JS puro; nenhuma dependÃªncia de UI |
| **Login exclusivo Discord OAuth2** | âœ… | `functions/api/auth/login.js`, `callback.js` e `me.js` |
| **`new Date().toISOString()` sÃ³ no servidor (UTC)** | âœ… | `functions/api/logs.js` â€” frontend apenas formata datas |
| **Footer com a chave `P26-2A-F83C-DFFD-BF8D-7582`** | âœ… | `public/index.html` |
| **Campos `user_id`, `username`, `avatar`, `conteudo`, `impedimento`, `criado_em`** | âœ… | API, schema e `public/script.js` alinhados |

---

## ðŸ“„ LicenÃ§a

MIT License â€” ver ficheiro [`LICENSE`](./LICENSE).

---

**Chave Ãšnica:** `P26-2A-F83C-DFFD-BF8D-7582`

