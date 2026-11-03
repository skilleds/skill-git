---
description: schematize-git — gera o handoff (context.md + checklist.md) no archive, sem compactar.
---
Gere o handoff em `<projeto>/<projeto>_archive/context/`: `AAAA-MM-DD-<slug>-context.md` (o que foi
feito, branches e PRs abertos, o que o gate acusou, tags criadas) e `AAAA-MM-DD-<slug>-checklist.md`
(FEITO vs ABERTO, nas três classes da casa: `- [ ]` máquina · `- [H ]` humano · `- [~]` on-hold).
Não compacte — só grave. Relatório a cada 30 min e handoff a cada 6 h (piso 26); depois de gravar, **commit + push** do archive e `check-git.sh <archive> --conclusao` (piso 22). Nunca grave segredo nem trecho de diff com credencial no handoff.
