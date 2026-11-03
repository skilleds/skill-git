# Changelog — schematize-git

Todas as mudanças relevantes deste pacote, no formato [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/),
com versionamento [SemVer](https://semver.org/lang/pt-BR/).

## [0.3.0] — 2026-11-03
### Adicionado
- `check-git.sh --conclusao`: trava de conclusão do piso 22 (remote não vazio e acessível, árvore limpa, upstream, `HEAD == origin`); exit 1 = a demanda não conclui. Chamada nos comandos `/git-commit`, `/git-pr`, `/git-review`, `/git-handoff`, `/git-cc`.
- Scanner de segredo ampliado: Anthropic `sk-ant-`, OpenAI `sk-`, Resend `re_`, Twilio `AC`/`SK`, JWT, URL com senha, webhook Slack/Discord e `password: valor` sem aspas.
- Gate `arquivo-grande` também no índice (`--staged`, hook `pre-commit`).
- Hook `pre-push` barra mover/apagar tag publicada e roda `check-git.sh --range` no que está sendo empurrado (a barreira por push, já que o CI remoto não roda por commit).
- `assets/github/ruleset-tags.json` (tags `v*` e `hml-*` imutáveis) e formato de tag de promoção `hml-AAAAMMDD.N`.
- Bloco "Herda os pisos 22–29"; deploy só do git (23) e CI só antes do hml (27) documentados no fluxo.
### Alterado
- CI do gate (`assets/ci/github-actions-git.yml`) sem `pull_request`: `workflow_dispatch` + tags `hml-*`/`v*`, `concurrency`, `paths-ignore`. `ruleset-main.json` sem status check obrigatório.
- Fluxo trunk-based: "CI verde no PR" virou "gate local verde"; CI remoto só antes de promover a hml (piso 27).

## [0.2.1] — 2026-10-14
Ponteiro para a skill nova `schematize-bots` (piso curto, sem duplicar).

### Alterado
- /git-leak e segredos.md citam token de bot.

## [0.2.0] — 2026-10-14
Integração ao catálogo: orçamento de description e ponteiros entre as irmãs novas.

### Alterado
- `description` <=190 (orçamento do catálogo).
- Motivo: o catálogo estourou o orçamento de listagem com 45 skills; cada fronteira aponta a dona em vez de duplicar.

## [0.1.0] — 2026-10-14
Primeira versão. O fluxo git da casa estava espalhado em memórias e regras soltas (commit+push na hora,
autorização de publicar, segredo vazado em incidente passado) e o catálogo não tinha dono para ele: o
release de skill é da `schematize-skill-dev`, mas o fluxo geral de qualquer repositório não. Esta skill
junta as regras e põe um gate executável na mão de quem commita.

### Adicionado
- `SKILL.md` com 13 pisos: GitHub como fonte da verdade (commit+push na hora), trunk-based com histórico
  linear, Conventional Commits 1.0.0 com tipos fixos, PR pequeno, nunca reescrever histórico publicado
  (exceção única: segredo/dado pessoal, com rito), segredo commitado = rotacionar primeiro, assinatura,
  ruleset no servidor, tags SemVer imutáveis, higiene, monorepo x multirrepo por ADR; divisas com
  engineering, infra, docs, skill-dev, incident e archive.
- `references/`: `fluxo`, `assinatura` (SSH, gitsign, GPG), `protecao` (ruleset, CODEOWNERS, dono solo),
  `segredos` (rito de reescrita), `higiene`, `stack-versoes` (fontes e datas).
- `scripts/check-git.sh`: gate de 6 regras (mensagem fora de Conventional Commits, segredo em linha
  adicionada mesmo removida depois, arquivo proibido, blob grande, merge em branch linear, não
  assinado) com modos `--range`, `--staged`, `--msg-file`; waiver rastreável (`git-ok:` / `Git-Ok:`).
  `check-git.test.sh`: 51 casos em repos temporários, vermelhos e verdes, segredos falsos montados em
  runtime, assinatura SSH real com chave efêmera e os três hooks exercitados.
- `assets/`: hooks `commit-msg`/`pre-commit`/`pre-push`, ruleset importável da `main`, CODEOWNERS,
  template de PR, `gitignore.base`, `gitattributes.base`, workflow `check-git` com Action pinada por SHA.
- Comandos: `/git-help`, `/git-load`, `/git-review`, `/git-commit`, `/git-pr`, `/git-leak`,
  `/git-release`, `/git-claude`, `/git-cc`, `/git-handoff`.
