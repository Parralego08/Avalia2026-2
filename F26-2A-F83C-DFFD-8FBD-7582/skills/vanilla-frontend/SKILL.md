---
name: vanilla-frontend
description: Padrões obrigatórios de interface web em HTML5, CSS3 e JavaScript puro (Vanilla JS) sem frameworks e linha do tempo vertical.
---

# Skill: Interface Web Vanilla + Linha do Tempo Vertical CSS

## Regras Obrigatórias
1. **Zero Frameworks de UI:** Utilizar exclusivamente HTML5 semântico, CSS3 moderno (Custom Properties, Flexbox, animações com `prefers-reduced-motion`) e JavaScript puro (IIFE, `fetch` com `credentials: 'include'`, manipulação segura do DOM via `createElement` e `textContent`).
2. **Privacidade e Estado de Sessão:** O formulário de registro e os filtros só são exibidos quando o usuário está autenticado via Discord OAuth2. O botão de logout encerra a sessão via `/api/auth/me?logout=1` sem sair da aplicação.
3. **Linha do Tempo Vertical Estilizada:** Exibir entradas em ordem cronológica decrescente com eixo vertical (`::before`), marcadores alinhados e codificação visual por nível de impedimento (`sem`, `atencao`, `bloqueado`).
4. **Integração com Busca no Backend:** O formulário de filtros consulta diretamente `/api/logs` com os query params `busca`, `impedimento`, `inicio` e `fim`.
5. **Rastreabilidade no Rodapé:** Manter obrigatoriamente a Chave Única `F26-2A-F83C-DFFD-8FBD-7582` visível dentro da tag `<footer>` do `index.html`.
