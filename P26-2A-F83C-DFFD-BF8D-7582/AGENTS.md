# DocumentaÃ§Ã£o de Engenharia de Agentes â€” Projeto #14

**Projeto:** #14 â€” DiÃ¡rio de Bordo de Projetos
**Chave Ãšnica:** `P26-2A-F83C-DFFD-BF8D-7582`
**Harness:** OpenCode Desktop (agentes customizados em `.opencode/agent/*.md` + skills em `.opencode/skill/*/SKILL.md`)
**Modelo Base:** Nemotron 3 Ultra (`nvidia/nemotron-3-ultra-550b-a55b:free` via OpenRouter)
**Skills versionadas na entrega:** `skills/cloudflare-backend/SKILL.md`, `skills/vanilla-frontend/SKILL.md`, `skills/auditoria-qa/SKILL.md`, `skills/design-system/SKILL.md`

> **Requisito da Rubrica (25 pontos):** mÃ­nimo de **3 agentes diferentes**, sendo obrigatoriamente **1 agente dedicado a teste e validaÃ§Ã£o**, com harness documentado e skills relacionadas.

---

## 1. Ambiente de OrquestraÃ§Ã£o (Harness)

- **Harness principal:** OpenCode Desktop, operando com quatro agentes especializados declarados em `.opencode/agent/`.
- **EstratÃ©gia de coordenaÃ§Ã£o:** os agentes trabalharam de forma modular sobre a subpasta `P26-2A-F83C-DFFD-BF8D-7582/`, compartilhando o contrato JSON da API, o schema D1 e a bateria de testes automatizados (`tests/api.test.mjs`).
- **Fluxo:** Backend define schema/rotas â†’ Frontend consome o contrato â†’ Validador audita seguranÃ§a, integridade e rubrica, retroalimentando os dois primeiros.

---

## 2. Agentes Criados e Responsabilidades

### Agente 1: Backend Specialist (`cloudflare-backend`)

- **FunÃ§Ã£o:** modelar o schema relacional no Cloudflare D1, implementar as Cloudflare Pages Functions (`/api/auth/*` e `/api/logs`) e garantir integridade de dados com timestamp UTC gerado exclusivamente no servidor.
- **Harness:** *"Atuas como engenheiro backend serverless. Usa apenas Cloudflare Pages Functions com o binding `env.DB` e queries parametrizadas. Nunca aceites datas vindas do frontend: gera `new Date().toISOString()` no servidor. Autentica exclusivamente via Discord OAuth2 e isola os dados por `user_id`."*
- **Skill utilizada:** [`skills/cloudflare-backend/SKILL.md`](./skills/cloudflare-backend/SKILL.md)
- **EntregÃ¡veis:**
  - `migrations/0001_init.sql` (espelhado em `schema.sql`) â€” tabela `logs` (`id`, `user_id`, `username`, `avatar`, `conteudo`, `impedimento`, `criado_em`) + Ã­ndice `idx_logs_user_criado`.
  - `lib/session.js` â€” sessÃ£o assinada com HMAC-SHA256 (Web Crypto) e expiraÃ§Ã£o, cookie `HttpOnly`/`SameSite=Lax`/`Secure` em HTTPS.
  - `functions/api/auth/login.js` â€” inicia OAuth2 com `scope=identify` e parÃ¢metro `state` anti-CSRF.
  - `functions/api/auth/discord.js` â€” alias oficial que encaminha para `/api/auth/login`.
  - `functions/api/auth/callback.js` â€” valida `state`, troca `code` por token, consulta `/users/@me` e cria a sessÃ£o assinada.
  - `functions/api/auth/discord/callback.js` â€” alias oficial que preserva `code`/`state` e encaminha para `/api/auth/callback`.
  - `functions/api/auth/me.js` â€” valida a sessÃ£o, retorna o perfil e implementa logout real (`?logout=1`).
  - `functions/api/logs.js` â€” `POST` valida payload e gera `criado_em` UTC no servidor; `GET` isola por `user_id` e filtra por `busca`, `impedimento`, `inicio` e `fim`.

### Agente 2: Frontend Specialist (`vanilla-frontend`)

- **FunÃ§Ã£o:** construir a interface completa em HTML5 semÃ¢ntico, CSS3 moderno e JavaScript puro, com timeline vertical, formulÃ¡rio de registro, filtros e estados de autenticaÃ§Ã£o.
- **Harness:** *"Ã‰ proibido usar React, Tailwind, Bootstrap ou qualquer framework CSS/JS. Usa apenas HTML5, CSS3 (Custom Properties, Flexbox, animaÃ§Ãµes com `prefers-reduced-motion`) e manipulaÃ§Ã£o direta do DOM com `createElement`/`textContent` e `fetch`."*
- **Skill utilizada:** [`skills/vanilla-frontend/SKILL.md`](./skills/vanilla-frontend/SKILL.md)
- **EntregÃ¡veis:**
  - `public/index.html` â€” estrutura semÃ¢ntica, formulÃ¡rio acessÃ­vel com contador de caracteres, filtros e `<footer>` com a Chave Ãšnica.
  - `public/style.css` â€” design system com tokens claro/escuro, componentes, timeline vertical alinhada, badges de impedimento e responsividade.
  - `public/script.js` â€” estado centralizado, mÃ³dulo `api`, `checkAuth`/`renderAuthArea`/logout, `loadLogs` (com query params), `renderTimeline` e feedback inline (sem `alert`).

### Agente 3: Validation Auditor (`auditoria-qa`) â€” ObrigatÃ³rio

- **FunÃ§Ã£o:** auditoria independente de seguranÃ§a, contratos de API, integridade do banco e conformidade com a rubrica de 100 pontos.
- **Harness:** *"Valida que rotas protegidas retornam 401 sem sessÃ£o e que cookies adulterados sÃ£o rejeitados. Confirma que `POST /api/logs` ignora `criado_em` do cliente e gera UTC no servidor. Verifica isolamento por `user_id`, validaÃ§Ã£o de `impedimento`, filtro de data final inclusivo, ausÃªncia de frameworks, licenÃ§a MIT, chave no README e no `<footer>`."*
- **Skill utilizada:** [`skills/auditoria-qa/SKILL.md`](./skills/auditoria-qa/SKILL.md)
- **EntregÃ¡veis:**
  - `tests/api.test.mjs` â€” suÃ­te com `node:test` cobrindo adulteraÃ§Ã£o de sessÃ£o, 401 sem sessÃ£o, validaÃ§Ã£o de payload, timestamp UTC server-side, isolamento por usuÃ¡rio, busca e intervalo de datas, logout e aliases oficiais do Discord.
  - RelatÃ³rio de auditoria abaixo (seÃ§Ã£o 4).

### Agente 4: Design System Specialist (`design-system`)

- **FunÃ§Ã£o:** manter o design system em CSS puro, garantindo consistÃªncia de tokens, dark mode, tipografia, componentes, foco visÃ­vel, contraste, movimento e responsividade.
- **Harness:** *"Atuas como guardiÃ£o do design system. Usa exclusivamente Custom Properties, CSS3 e HTML5 sem frameworks. Nenhuma cor, sombra ou raio fora dos tokens. Garantes foco visÃ­vel, contraste AA, alvos de 44px e `prefers-reduced-motion`. Preservas o contrato de IDs/classes do `script.js` e a Chave Ãšnica no rodapÃ©."*
- **Skill utilizada:** [`skills/design-system/SKILL.md`](./skills/design-system/SKILL.md)
- **EntregÃ¡veis:**
  - `public/style.css` â€” tokens claro/escuro, componentes (botÃµes, formulÃ¡rios, painÃ©is, badges, feedback), timeline com geometria exata, rodapÃ© e breakpoints.
  - `public/index.html` â€” marca com SVG inline, headings de seÃ§Ã£o, badges de contexto e rodapÃ© com a Chave Ãšnica.
  - RevisÃ£o visual documentada (claro/escuro, 360/768/1280px, teclado e movimento reduzido).

---

## 3. Skills Versionadas

| Skill | Caminho | Uso |
|-------|---------|-----|
| `cloudflare-backend` | `skills/cloudflare-backend/SKILL.md` | Regras de Functions, D1, OAuth2, sessÃ£o assinada e UTC no servidor |
| `vanilla-frontend` | `skills/vanilla-frontend/SKILL.md` | Regras de HTML/CSS/JS puro, timeline e integraÃ§Ã£o com a API |
| `auditoria-qa` | `skills/auditoria-qa/SKILL.md` | Checklist de seguranÃ§a, testes e conformidade com a rubrica |
| `design-system` | `skills/design-system/SKILL.md` | Tokens, dark mode, componentes, acessibilidade e responsividade |

*As mesmas skills tambÃ©m estÃ£o disponÃ­veis em `.opencode/skill/*/SKILL.md` para uso direto pelo harness durante o desenvolvimento.*

---

## 4. EvidÃªncias de ValidaÃ§Ã£o (Agente 3)

Comando executado:

```bash
node --test tests/api.test.mjs
```

Resultado:

```text
âœ” signSession e verifySession rejeitam token adulterado
âœ” GET /api/logs e POST /api/logs retornam 401 sem sessÃ£o vÃ¡lida
âœ” POST /api/logs valida payload e forÃ§a timestamp UTC do servidor
âœ” GET /api/logs isola registros por user_id e filtra corretamente busca e intervalo de datas
âœ” GET /api/auth/me valida sessÃ£o e limpa cookie no logout
âœ” aliases oficiais /api/auth/discord encaminham para o fluxo canÃ´nico preservando a query

tests 6 | pass 6 | fail 0
```

| CritÃ©rio Auditado | Status |
|-------------------|--------|
| Cookie de sessÃ£o assinado (HMAC-SHA256) e expiraÃ§Ã£o | âœ… |
| `state` anti-CSRF no fluxo OAuth2 | âœ… |
| `POST /api/logs` exige sessÃ£o e valida `conteudo`/`impedimento` | âœ… |
| `criado_em` gerado apenas no servidor (UTC) | âœ… |
| `GET /api/logs` isolado por `user_id` + busca e datas inclusivas | âœ… |
| Frontend sem frameworks (HTML/CSS/JS puro) | âœ… |
| Design system com tokens, dark mode, foco visÃ­vel e responsividade | âœ… |
| Acessibilidade, boas prÃ¡ticas e SEO (Lighthouse) | âœ… 100 / 100 / 100 |
| Chave Ãšnica no `README.md` e no `<footer>` | âœ… |
| LicenÃ§a MIT e 4 skills na pasta `skills/` | âœ… |

---

## 5. ObservaÃ§Ãµes de Entrega

- A pasta do projeto mantÃ©m o nome exato da Chave Ãšnica emitida: `P26-2A-F83C-DFFD-BF8D-7582/`.
- O arquivo `AGENTES.md` Ã© mantido apenas como compatibilidade; a documentaÃ§Ã£o oficial passou a ser este `AGENTS.md`, conforme a rubrica.

