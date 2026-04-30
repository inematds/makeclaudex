---
name: Curso Build Claudex — Plano do Projeto
description: Plano completo do curso "Engenharia com Claude Code — Do Prompt ao Plugin de Produção", status e próximos passos
type: project
---

Curso em 3 trilhas sobre como o Claudex foi construído, usando os build_prompts como material-fonte.

**Why:** Complementa o curso-claudex.html (que ensina a *usar* o Claudex) — este ensina a *construir* sistemas desse nível. Os dois se reforçam mutuamente com CTA cruzado.

**How to apply:** Ao retomar este projeto, ler o plano completo em `../doc/PLANO_CURSO_BUILD.md` (relativo a makeclaudex) antes de qualquer coisa. Fontes estão em `build_prompts/`.

## Estado em 2026-04-29

- **Trail 1 (Fundamentos)** — ENTREGUE (Emerald, 7 arquivos, ~4h30min)
- **Trail 2 (Construindo)** — ENTREGUE (Purple, 7 arquivos, ~5h)
- **Trail 3 (Avançado)** — ENTREGUE (Blue, 7 arquivos, ~5h)

Todas as trilhas concluídas. Curso completo com 21 arquivos HTML.

## Decisões tomadas

- Cor: **Purple T3** (diferencia do Teal do curso-claudex.html)
- Prompts: incluídos em PT-BR com bloco de código + explicação
- M6: hook de linting como projeto guiado
- Formato: multi-página obrigatório (trilha/index.html + modulo-X-X.html separados)

## Arquivos de saída (Trail 2)

```
makeclaudex/curso/trilha2/
├── index.html          — índice da trilha (cards, modal+iframe, "Ver Completo")
├── modulo-2-1.html     — O Blueprint (7 tópicos, ~50 min)
├── modulo-2-2.html     — O Cérebro do Loop (7 tópicos, ~60 min)
├── modulo-2-3.html     — Acabamento e Segurança (6 tópicos, ~50 min)
├── modulo-2-4.html     — O que Isso nos Deu (6 tópicos, ~40 min)
├── modulo-2-5.html     — A Metodologia (6 tópicos, ~45 min)
└── modulo-2-6.html     — Seu Próximo Projeto (6 tópicos, ~60 min)
```

Git: `github.com/inematds/makeclaudex` — commit `7e67f85`

**Nota:** O arquivo `docs/como-construimos-o-claudex.html` era a tentativa anterior (single-page, formato incorreto) — está obsoleto e pode ser removido do repo.

## Estrutura do Trail 2 (entregue)

- M1 (~50 min, 7 tópicos): O Blueprint — README + SPEC + Prompts 01–02
- M2 (~60 min, 7 tópicos): O Cérebro do Loop — Prompts 03–05
- M3 (~50 min, 6 tópicos): Acabamento e Segurança — Prompts 06–07
- M4 (~40 min, 6 tópicos): O que isso nos deu — antes/depois + CTA para curso-claudex.html
- M5 (~45 min, 6 tópicos): A Metodologia — padrões extraídos reutilizáveis
- M6 (~60 min, 6 tópicos): Seu Próximo Projeto — hook de linting

## Próximos passos

- Fazer commit e push de todos os arquivos novos
- Remover `docs/como-construimos-o-claudex.html` (single-page obsoleto)
- Testar navegação entre as 3 trilhas no browser
