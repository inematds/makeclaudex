---
name: Regra de formato de curso — invocar skill ANTES de construir
description: /formato-curso deve ser invocado antes de criar qualquer HTML; estrutura multi-página deve ser proposta e confirmada antes de escrever código
type: feedback
---

Sempre invocar `/formato-curso` ANTES de criar qualquer página HTML de curso. Sempre propor a estrutura completa de arquivos (`trilha/index.html` + páginas `modulo-X-X.html` separadas) e aguardar confirmação do usuário antes de escrever qualquer código.

**Why:** Na construção do Trail 2, o HTML foi criado como página única baseando-se no `curso-claudex.html` como referência — que é ele próprio fora do padrão. O skill `/formato-curso` foi invocado só após a entrega, detectando o problema tarde demais. A estrutura correta exige `trilha/index.html` (cards com modal+iframe e link "Ver Completo") + 6 páginas de módulo separadas.

**How to apply:** Fluxo obrigatório antes de qualquer build de curso:
1. Invocar `/formato-curso` para carregar as referências
2. Ler `MASTER_COMPLETO.md` seções relevantes
3. Propor estrutura de arquivos explícita ao usuário (listar todos os arquivos que serão criados)
4. Aguardar confirmação
5. Só então começar a escrever HTML
