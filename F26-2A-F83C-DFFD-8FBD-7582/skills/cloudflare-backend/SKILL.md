---
name: cloudflare-backend
description: Padrões obrigatórios para API serverless no Cloudflare Pages Functions com D1, sessão assinada e Discord OAuth2.
---

# Skill: Arquitetura Serverless Cloudflare Functions + D1 + Discord OAuth2

## Regras Obrigatórias
1. **Cloudflare Pages Functions + D1:** Toda rota de API reside em `functions/api/*` utilizando `onRequestGet`/`onRequestPost` e queries parametrizadas no binding `env.DB` (`prepare(...).bind(...)`).
2. **Autenticação Exclusiva via Discord OAuth2:** Fluxo completo em `/api/auth/login` (com `state` anti-CSRF), `/api/auth/callback` (troca `code` por `access_token` e consulta `/api/users/@me`) e `/api/auth/me`, extraindo `user_id`, `username` e `avatar`.
3. **Sessão Assinada em Cookie HttpOnly:** O cookie `session` é assinado com HMAC-SHA256 (`lib/session.js` via Web Crypto `crypto.subtle`) e expiração (`exp`), impedindo adulteração no cliente.
4. **Timestamp UTC Exclusivo no Servidor:** Nunca aceitar datas enviadas pelo cliente. O campo `criado_em` é sempre gerado no backend via `new Date().toISOString()` no momento do `INSERT`.
5. **Isolamento e Busca Textual:** `GET /api/logs` filtra obrigatoriamente por `WHERE user_id = ?`, valida `impedimento` (`sem`, `atencao`, `bloqueado`) e suporta filtros por `busca` (`LIKE`), `impedimento`, `inicio` (`T00:00:00.000Z`) e `fim` (`T23:59:59.999Z`).
