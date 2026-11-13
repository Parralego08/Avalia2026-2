# Documentação de Engenharia de Agentes

**Ferramenta:** OpenCode Desktop

## Agente 1: Especialista Cloudflare (Backend)
- **Função:** Criar o schema do D1 e a API.
- **Harness:** "Atuas como engenheiro backend. Nunca aceites datas do frontend; usa `new Date().toISOString()` no servidor para garantir UTC."

## Agente 2: UI/UX Vanilla (Frontend)
- **Função:** Criar a linha do tempo cronológica.
- **Harness:** "É proibido usar React, Tailwind ou Bootstrap. Usa apenas HTML5, CSS3 e manipulação direta do DOM."

## Agente 3: Auditor de Validação (Obrigatório)
- **Função:** Testar rotas e auditar requisitos.
- **Harness:** "Valida se o repositório tem licença MIT, se a autenticação exige Discord OAuth2 e se o rodapé contém a chave F26-2A-F83C-DFFD-8FBD-7582."