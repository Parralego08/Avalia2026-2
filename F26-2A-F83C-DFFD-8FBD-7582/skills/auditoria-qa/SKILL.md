---
name: auditoria-qa
description: Protocolo do Agente de Teste e Validação para auditar segurança, integridade no banco D1, contratos JSON e conformidade com a rubrica.
---

# Skill: Agente de Teste, Auditoria e Validação

## Checklist de Verificação Obrigatória
1. **Segurança de Sessão e Anti-Adulteração:**
   - Verificar que rotas protegidas retornam `401` sem cookie ou com cookie `session` adulterado/expirado.
   - Confirmar que `POST /api/logs` ignora qualquer `criado_em` enviado no payload e gera timestamp UTC válido no servidor.
2. **Validação de Entrada e Isolamento de Dados:**
   - Confirmar retorno `400` para `conteudo` vazio/excedendo 2000 caracteres e `impedimento` fora de `['sem', 'atencao', 'bloqueado']`.
   - Validar que `GET /api/logs` retorna apenas registros do `user_id` autenticado e que o filtro `fim` inclui todo o dia selecionado (`T23:59:59.999Z`).
3. **Conformidade Estrutural da Rubrica:**
   - Ausência de frameworks proibidos no frontend.
   - Presença de `LICENSE` (MIT), `AGENTS.md`, `skills/*/SKILL.md` (mínimo 3), `README.md` com Chave Única e `URL_CLOUDFLARE`, e chave `F26-2A-F83C-DFFD-8FBD-7582` no `<footer>`.
4. **Execução de Testes Automatizados:**
   - Rodar `node --test tests/api.test.mjs` com 100% de aprovação.
