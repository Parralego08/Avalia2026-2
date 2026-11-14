---
name: auditoria-qa
description: Checklist de auditoria, testes de rotas e validação dos requisitos do Projeto #14.
---
# Checklist de Validação Obrigatória
1. Confirmar ausência de frameworks no frontend.
2. Verificar se o login exige Discord OAuth2 (`user_id`, nome e avatar).
3. Auditar `functions/api/logs.js` para garantir que `new Date().toISOString()` é gerado no backend e que há filtro por `user_id`.
4. Verificar presença do arquivo `LICENSE` (MIT), do `AGENTS.md` e da chave `F26-2A-F83C-DFFD-8FBD-7582` no `README.md` e no `<footer>` do `public/index.html`.