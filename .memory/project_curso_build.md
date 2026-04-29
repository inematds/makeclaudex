---
name: Curso Build Claudex — Plano do Projeto
description: Plano completo do curso "Engenharia com Claude Code — Do Prompt ao Plugin de Produção", status e próximos passos
type: project
---

Curso em 3 trilhas sobre como o Claudex foi construído, usando os build_prompts como material-fonte.

**Why:** Complementa o curso-claudex.html (que ensina a *usar* o Claudex) — este ensina a *construir* sistemas desse nível. Os dois se reforçam mutuamente com CTA cruzado.

**How to apply:** Ao retomar este projeto, ler o plano completo em `../doc/PLANO_CURSO_BUILD.md` (relativo a makeclaudex) antes de qualquer coisa. Fontes estão em `build_prompts/`.

## Estado em 2026-04-29

- **Trail 2 (Construindo)** — ENTREGUE: `docs/como-construimos-o-claudex.html`
- **Trail 1 (Fundamentos)** — não iniciado
- **Trail 3 (Avançado)** — não iniciado

Ordem de construção decidida: Trail 2 → Trail 3 → Trail 1

## Decisões tomadas

- Cor: **Purple T3** (diferencia do Teal do curso-claudex.html)
- Prompts: incluídos em PT-BR com bloco de código + explicação
- M6: hook de linting como projeto guiado

## Arquivo de saída atual

`iclaudex/docs/como-construimos-o-claudex.html` — 6 módulos, 38 tópicos, ~5h

## Estrutura do Trail 2 (entregue)

- M1 (~50 min, 7 tópicos): O Blueprint — README + SPEC + Prompts 01–02
- M2 (~60 min, 7 tópicos): O Cérebro do Loop — Prompts 03–05
- M3 (~50 min, 6 tópicos): Acabamento e Segurança — Prompts 06–07
- M4 (~40 min, 6 tópicos): O que isso nos deu — antes/depois + CTA para curso-claudex.html
- M5 (~45 min, 6 tópicos): A Metodologia — padrões extraídos reutilizáveis
- M6 (~60 min, 6 tópicos): Seu Próximo Projeto — hook de linting

## Próximos passos

1. Trail 3 (Avançado): projeto guiado mais complexo (ex: plugin de revisão de PRs com isolamento de branch)
2. Trail 1 (Fundamentos): base zero — definir quem é o aluno antes de escrever
3. Adicionar CTA de volta no `curso-claudex.html` apontando para este novo curso
