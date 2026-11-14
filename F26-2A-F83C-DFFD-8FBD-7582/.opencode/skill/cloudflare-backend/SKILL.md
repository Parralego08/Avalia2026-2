---
name: cloudflare-backend
description: Padrões obrigatórios para API serverless no Cloudflare Pages Functions com D1 e Discord OAuth2.
---
# Regras de Backend Serverless
- Use apenas Cloudflare Pages Functions (`onRequestGet`, `onRequestPost`) com o binding `env.DB`.
- Autenticação exclusiva via Discord OAuth2 capturando `user_id`, `username` e `avatar`.
- Nunca confie em datas enviadas pelo cliente: gere sempre `new Date().toISOString()` no servidor (UTC) no momento do `INSERT`.
- Isole os dados: a rota GET deve filtrar os registros pelo `user_id` do usuário autenticado e suportar filtros por `busca` (`LIKE`), `inicio` e `fim`.