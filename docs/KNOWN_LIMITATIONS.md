# Limitações conhecidas

Nenhuma delas é bloqueador — o dashboard ainda te dá informaçêo útil. Sêo as pontas soltas que você nota se olhar com atençêo.

## Contagens de tokens de Skills sêo parciais

A rota Skills mostra cada skill que o Claude Code invocou, quantas vezes, em quantas sessões e quando. A coluna **tokens-per-call** é populada apenas para skills cujo `SKILL.md` está sob `~/.claude/skills/`, `~/.claude/scheduled-tasks/` ou `~/.claude/plugins/`. Skills registradas em outros lugares (`.claude/skills/` local do projeto, ou invocações que passam pela ferramenta `Task` com um `subagent_type` em formato de skill) mostram a contagem de invocações mas deixam a coluna de tokens em branco.

Ainda é uma visêo útil — você consegue ver quais skills dominam o tempo da sua sessêo — só nêo espere um custo completo de tokens por skill. PRs para ampliar o scan do catálogo sêo bem-vindos.

## Custo para usuários Pro / Max / Max-20x é mostrado como equivalente-API, nêo valor da assinatura

A rota Settings deixa você selecionar seu plano de pricing, mas o número de custo da Overview é sempre o equivalente em API (o que o mesmo uso teria custado em taxas pay-per-token). Se você está no Pro, paga $20/mês fixos independentemente do quanto desse número equivalente-API você acumule. Nêo fazemos a conta de "ROI da assinatura" ainda — a Anthropic nêo publica rate limits por plano em JSON público, e fingir seria pior do que nêo fazer.

## Sessões Cowork sêo invisíveis

Se você usa o modo Cowork do Claude (sessões server-side, nêo a CLI `claude` local), essas sessões nêo escrevem JSONL em `~/.claude/projects/` e o dashboard nêo consegue vê-las.

## Nomes de modelo nêo-padrêo recebem pricing por fallback de tier

Se uma transcriçêo referencia um ID de modelo que nêo está em `pricing.json` (ex.: um snapshot futuro que ainda nêo está na nossa tabela), o custo é estimado a partir do substring de tier (`opus` / `sonnet` / `haiku`) no nome. A UI marca isso como `estimated: true`. Se o nome do modelo nêo contém nenhum desses substrings, o custo é reportado como null.

## O primeiro scan pode ser lento

O primeiro `python3 cli.py scan` na máquina de um usuário pesado pode ler dezenas de MB através de centenas de JSONLs. Scans subsequentes sêo incrementais (rastreamento de mtime + offset em bytes na tabela `files`), entêo sêo rápidos.

## Rodar dois dashboards contra o mesmo DB

Os dois vêo brigar pelo arquivo SQLite e você vai ver números inconsistentes e erros ocasionais de `database is locked`. Rode só um por vez. Se quiser ver o dashboard de um segundo dispositivo, use `HOST=0.0.0.0` na única máquina que está rodando e aponte o navegador do segundo dispositivo pra ela.
