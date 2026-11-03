# Fluxo: trunk-based, branch curta, PR pequeno

> Parte da `schematize-git`. O fluxo existe para que `main` esteja sempre liberável e para que o
> histórico seja legível daqui a dois anos por quem não estava na sala.

## 1. Trunk-based com branches curtas

- **Uma branch de integração (`main`).** Ela está sempre verde e liberável. Não existe `develop`.
- **Branch de trabalho vive horas ou 1-2 dias**, nunca semanas. Se passou disso, o trabalho é grande
  demais: fatie, esconda atrás de feature flag e integre em partes.
- Nome: `tipo/assunto-curto` (`feat/login-passkey`, `fix/timeout-webhook`). O tipo é o do Conventional
  Commits. Branch de longa duração (`release/1.x`) só quando há versão antiga mantida (backport).
- **Rebase da branch sobre `main` antes do PR** (`git fetch && git rebase origin/main`), nunca merge de
  `main` dentro da branch de trabalho. Reescrever a SUA branch ainda não publicada é livre; a regra de
  histórico publicado está em `protecao.md` §4.
- Integração em `main` por **squash** (um commit por PR, mensagem em Conventional Commits) ou por
  **rebase** (cada commit do PR já é bom). Merge commit só onde a branch NÃO exige linear — e a `main`
  da casa exige (`protecao.md`).

## 2. Conventional Commits 1.0.0

Formato: `tipo(escopo)!: descrição` · linha em branco · corpo · linha em branco · rodapés.

| Tipo | Quando | SemVer |
|---|---|---|
| `feat` | funcionalidade nova para quem usa | MINOR |
| `fix` | correção de defeito | PATCH |
| `perf` | ganho de desempenho sem mudar contrato | PATCH |
| `refactor` | muda estrutura, não comportamento | nenhum |
| `docs` | só documentação | nenhum |
| `test` | só teste | nenhum |
| `build` · `ci` | sistema de build, pipeline | nenhum |
| `style` | formatação sem efeito | nenhum |
| `chore` | manutenção que não cabe acima (deps, tooling) | nenhum |
| `revert` | desfaz commit anterior | conforme o desfeito |

- `!` antes dos dois-pontos **ou** rodapé `BREAKING CHANGE: <o que quebra e como migrar>` = MAJOR.
- A especificação deixa outros tipos permitidos; **o gate da casa fixa a lista acima** para o changelog
  ser previsível. Tipo novo entra por PR nesta skill, não por commit.
- Descrição: imperativo, minúscula, sem ponto final, até 72 colunas (o gate avisa acima disso). O
  **porquê** vai no corpo; o diff já diz o quê. Idioma do projeto, uma vez só.
- Um commit = uma mudança lógica que compila e passa no teste (bisect depende disso). Misturar
  refactor com feature num commit é o defeito clássico.
- `fixup!`/`squash!`/`amend!` são ferramentas locais: o gate reprova se chegarem ao range publicado.
  Use `git rebase -i --autosquash` antes do push.
- Rodapés úteis: `Refs: #123`, `Co-Authored-By: ...`, `Git-Ok: <regra> <motivo>` (waiver do gate,
  `scripts/check-git.sh`).

## 3. PR pequeno

- **Alvo: revisável em 15 minutos** (ordem de grandeza: até ~400 linhas de diff útil, sem lockfile e
  gerado). Maior que isso: divida em PRs empilhados ou prepare com refactor mecânico separado.
- **Descrição responde 4 perguntas:** o que muda e por quê (link da issue/ADR); como foi testado
  (comando e resultado, não "testei"); risco e rollback (migração? flag? reversível?); o que o revisor
  deve olhar com mais atenção.
- **Gate LOCAL verde antes de pedir revisão** (`check-git.sh`, hooks, `tools/ci.sh`). PR em rascunho
  enquanto o gate local está vermelho. **Não existe CI remoto por PR nem por commit** (piso 27): ele
  roda uma vez antes de promover a hml (tag `hml-*`), em release (`v*`) ou sob pedido. Nenhum agent
  dispara nem espera CI remoto por commit.
- **Quem abre não aprova o próprio.** Em repo de uma pessoa só, a revisão é o gate local mais a
  releitura do diff em PR (a regra "dono solo" está em `protecao.md` §3).
- Comentário de revisão vira commit novo na branch durante a revisão; o squash final limpa.
- Template pronto: `assets/github/pull_request_template.md`.

## 4. GitHub é a fonte da verdade

Regra da casa: **commit + push na hora.** Trabalho que existe só na máquina local é trabalho que não
existe: disco morre, máquina é reinstalada, e uma cópia local que diverge do publicado causa a
oscilação de versão que já custou caro. Consequências:

- Terminou uma unidade de trabalho coerente: commit e push (em branch curta, PR aberto).
- Nunca deixe a cópia local divergir do publicado: se a branch local tem commits que o remoto não tem,
  ou o remoto tem os que a local não tem, resolva agora (fetch, rebase, push), não "depois".
- `git status` sujo no fim da sessão é pendência declarada, não estado normal.
- **TRAVA DE CONCLUSÃO (piso 22 da engineering):** nenhuma demanda se declara concluída sem **remote
  acessível, push feito, árvore limpa e `HEAD == origin`**. Gate: `bash
  .claude/skills/schematize-git/scripts/check-git.sh . --conclusao` (exit 1 = **a demanda NÃO
  conclui**: crie o repo — `gh repo create <org>/<repo> --private` —, adicione o `origin` e dê o push
  antes de declarar pronto). Vale para todo repo tocado, inclusive o `_archive` privado. Commit e
  push do relatório (30 min) e do handoff (6 h) também (piso 26).
- **Deploy só do git (piso 23):** o artefato de deploy é um **commit SHA ou tag** presente no remoto,
  verificável (assinatura via `allowed_signers`, `assinatura.md`); o ops clona/atualiza esse SHA na VPS.
  Commit não pushado não vai a lugar nenhum. VETADO `scp`/`rsync`/`tar | ssh`/`docker save | load`/
  `git bundle`.
- Em repo que o dono mantém sozinho e que não tem PR (ex.: pacote de skill), push direto em `main` é
  permitido com `GIT_ALLOW_PUSH_MAIN=1` (ver o hook `pre-push`); force push, nunca.

## 5. Tags SemVer e changelog

- **Tag de promoção a hml (piso 27):** `hml-AAAAMMDD.N` (ex.: `hml-20261103.1`), **anotada e imutável**
  como as `vX.Y.Z`; empurrá-la dispara o CI remoto **uma vez**, sobre tudo o que acumulou desde a
  promoção anterior. Mesmos rulesets de tag (`assets/github/ruleset-tags.json`) e o hook `pre-push`
  barram mover/apagar. Promoção errada se corrige com a tag seguinte.
- Release = **tag anotada** `vX.Y.Z` em commit de `main` (`git tag -a v1.4.0 -m "v1.4.0"`), assinada
  quando a política de assinatura (`assinatura.md`) está ligada (`git tag -s`). Tag publicada não se
  move nem se apaga: errou, publique `vX.Y.(Z+1)`.
- SemVer 2.0.0: MAJOR quebra contrato, MINOR adiciona compatível, PATCH corrige. Pré-release
  `1.0.0-rc.1`. Antes de `1.0.0` o contrato é instável e isso se declara no README.
- O tipo dos commits desde a última tag **calcula** o próximo número: qualquer `!`/BREAKING = MAJOR;
  senão qualquer `feat` = MINOR; senão PATCH. Se o número calculado diverge do que a pessoa quer
  publicar, um dos dois está errado: leia os commits.
- **CHANGELOG.md** no formato Keep a Changelog, derivado dos commits mas **escrito para humanos**: o
  porquê, não o hash. Ferramenta que ajuda a gerar o rascunho: git-cliff ou commitizen (`stack-versoes.md`);
  a curadoria é humana. O formato do documento é da `schematize-docs`.
- Release de skill do catálogo não é este rito: é da `schematize-skill-dev → references/release.md`.
