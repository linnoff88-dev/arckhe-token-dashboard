# Token Dashboard

Um dashboard local que lÃª as transcriÃ§Ãµes JSONL que o Claude Code grava em `~/.claude/projects/` e as transforma em anÃ¡lise de custo por prompt, mapas de calor de ferramentas/arquivos, atribuiÃ§Ã£o de subagentes, anÃ¡lise de cache, comparaÃ§Ã£o entre projetos e um motor de dicas baseado em regras.

**Tudo roda localmente.** Nenhum dado sai da sua mÃ¡quina â€” sem telemetria, sem chamadas de API com seus dados, sem login.

![Aba Overview â€” totais e grÃ¡ficos diÃ¡rios](docs/images/dashboard-overview-top.jpg)

![Aba Overview â€” por projeto, por modelo, top de ferramentas, sessÃµes recentes](docs/images/dashboard-overview-bottom.jpg)

## Para que isto serve

- Ver quais dos seus prompts sÃ£o caros (surpresa: geralmente envolvem resultados grandes de ferramentas).
- Comparar uso de tokens entre projetos em que vocÃª trabalhou.
- Identificar padrÃµes desperdiÃ§adores â€” o mesmo arquivo lido vinte vezes em uma sessÃ£o, uma chamada de ferramenta retornando 80k tokens.
- Entender o que um "cache hit" realmente economiza.
- Se vocÃª estÃ¡ no Pro ou Max, confirmar que estÃ¡ tendo retorno do dinheiro em dÃ³lares equivalentes Ã  API.

## PrÃ©-requisitos

- **Python 3.8 ou mais novo** â€” jÃ¡ instalado no macOS e na maioria do Linux. No Windows: `winget install Python.Python.3.12` ou baixe de python.org.
- **Claude Code** â€” instalado e com pelo menos uma sessÃ£o executada. O dashboard lÃª essas sessÃµes. Se vocÃª acabou de instalar o Claude Code e ainda nÃ£o o usou, rode pelo menos um prompt primeiro.
- **Um navegador web.** Qualquer um moderno.

Sem `pip install`. Sem Node.js. Sem etapa de build.

## InÃ­cio rÃ¡pido

```bash
git clone https://github.com/inematds/token-dashboard.git
cd token-dashboard
python3 cli.py dashboard
```

> No Windows, se `python3` nÃ£o estiver no PATH, substitua `py -3` por `python3` em todos os comandos abaixo.

O comando:
1. Varre `~/.claude/projects/` (a primeira execuÃ§Ã£o pode levar 20â€“60 segundos na mÃ¡quina de um usuÃ¡rio pesado).
2. Inicia um servidor local em http://127.0.0.1:8080.
3. Abre seu navegador padrÃ£o nessa URL.

Deixe rodando; ele re-varre a cada 30 segundos e envia atualizaÃ§Ãµes ao vivo. Pare com `Ctrl+C`.

## De onde vÃªm os dados

O Claude Code grava um arquivo JSONL por sessÃ£o aqui:

| SO | Caminho |
|---|---|
| macOS / Linux | `~/.claude/projects/<project-slug>/<session-id>.jsonl` |
| Windows | `C:\Users\<you>\.claude\projects\<project-slug>\<session-id>.jsonl` |

O dashboard nunca modifica esses arquivos â€” apenas os lÃª e mantÃ©m um cache SQLite local em `~/.claude/token-dashboard.db`.

Para apontar para outro local:

```bash
python3 cli.py dashboard --projects-dir /path/to/projects --db /path/to/cache.db
```

### VariÃ¡veis de ambiente

| Var | PadrÃ£o | PropÃ³sito |
|---|---|---|
| `PORT` | `8080` | Porta em que o servidor web local escuta |
| `HOST` | `127.0.0.1` | EndereÃ§o de bind. Mantenha o padrÃ£o. Definir `0.0.0.0` expÃµe todo o seu histÃ³rico de prompts a qualquer um na sua rede local â€” nÃ£o faÃ§a isso em nenhuma rede que vocÃª nÃ£o controle totalmente (nada de Wi-Fi de cafeteria, nada de coworking). |
| `CLAUDE_PROJECTS_DIR` | `~/.claude/projects` | Onde varrer os arquivos JSONL de sessÃ£o |
| `TOKEN_DASHBOARD_DB` | `~/.claude/token-dashboard.db` | Local do cache SQLite |

Os preÃ§os ficam em [`pricing.json`](pricing.json). Edite-o diretamente se os preÃ§os dos modelos mudarem ou para adicionar um novo plano.

## ReferÃªncia da CLI

```bash
python3 cli.py scan          # popula / atualiza o DB local e sai
python3 cli.py today         # totais de hoje (terminal)
python3 cli.py stats         # totais de todo o perÃ­odo (terminal)
python3 cli.py tips          # sugestÃµes ativas (terminal)
python3 cli.py dashboard     # scan + serve a UI em http://localhost:8080

# flags do dashboard
python3 cli.py dashboard --no-open   # nÃ£o abre o navegador automaticamente
python3 cli.py dashboard --no-scan   # pula o scan inicial (usa sÃ³ o DB em cache)
```

Trocar a porta: `PORT=9000 python3 cli.py dashboard`.

## As 7 abas

O dashboard Ã© uma pÃ¡gina Ãºnica com uma barra de abas no topo usando hash-router. Cada aba Ã© alimentada por sua prÃ³pria API JSON em `/api/`:

- **Overview** â€” tokens de input/output/cache de todo o perÃ­odo, sessÃµes, turnos, custo estimado no plano escolhido, grÃ¡ficos diÃ¡rios de trabalho e leitura de cache, tokens por projeto, participaÃ§Ã£o de tokens por modelo, top de ferramentas por contagem de chamadas e sessÃµes recentes. Esta Ã© a aba inicial.
- **Prompts** â€” seus prompts de usuÃ¡rio mais caros, ranqueados por tokens. Clique em qualquer linha para ver a resposta do assistente, as chamadas de ferramenta feitas e o tamanho de cada resultado de ferramenta.
- **Sessions** â€” visÃ£o turno a turno de qualquer sessÃ£o, com tokens e chamadas de ferramenta por turno.
- **Projects** â€” comparaÃ§Ã£o por projeto: tokens, contagem de sessÃµes e quais arquivos foram mais tocados.
- **Skills** â€” quais skills vocÃª invoca com mais frequÃªncia e (onde dÃ¡ pra medir) seu custo em tokens. Veja [limitaÃ§Ãµes](docs/KNOWN_LIMITATIONS.md#skills-token-counts-are-partial).
- **Tips** â€” sugestÃµes baseadas em regras para reduzir uso de tokens (leituras repetidas de arquivos, resultados de ferramentas grandes demais, taxa baixa de cache hit, etc.).
- **Settings** â€” alterna o pricing entre API / Pro / Max / Max-20x para que os valores de custo em todo o resto reflitam seu plano real.

A aba Overview tambÃ©m tem um painel embutido "What do these numbers mean?" que explica tokens de input/output/cache em linguagem simples.

## SoluÃ§Ã£o de problemas

**"Sem dados" ou grÃ¡ficos vazios.** Rode `python3 cli.py scan` uma vez para popular o DB e recarregue.

**Porta 8080 jÃ¡ em uso.** `PORT=9000 python3 cli.py dashboard`.

**NÃºmeros parecem errados / travados.** O DB fica em `~/.claude/token-dashboard.db`. Apague e rode `python3 cli.py scan` para reconstruir do zero.

**Rodar o dashboard duas vezes ao mesmo tempo.** NÃ£o faÃ§a â€” os dois processos vÃ£o brigar pelo DB SQLite. Pare todas as instÃ¢ncias antes de iniciar uma nova.

## Nota sobre precisÃ£o

O Claude Code grava cada resposta do assistente 2â€“3 vezes em disco enquanto ela faz streaming (a mesma mensagem da API Ã© snapshotada conforme o output cresce). O dashboard deduplica isso por `message.id` para que o total final bata com o que a API realmente cobrou. Se vocÃª comparar com outra ferramenta que soma cada linha do JSONL, espere que os nÃºmeros deste dashboard sejam menores â€” e mais prÃ³ximos da realidade.

## Privacidade

Nada sai da sua mÃ¡quina. Sem telemetria. Sem chamadas remotas com seus dados. O navegador busca seu JSON de `127.0.0.1`, e todo JS/CSS/fonte Ã© servido por esse mesmo servidor local â€” o ECharts estÃ¡ vendorizado em `web/`, e a UI usa fontes do sistema como fallback em vez de buscar de uma CDN de fontes. Se quiser verificar: `grep -r "https://" token_dashboard/ web/` â€” nÃ£o vai encontrar nada.

## Stack tÃ©cnica

Python 3 (sÃ³ stdlib) para a CLI, scanner e servidor HTTP. SQLite para o cache local. Vanilla JS + ECharts para a UI, sem etapa de build. Tema escuro, router baseado em hash, server-sent events para refresh ao vivo.

Fluxo de dados: `cli.py` â†’ `token_dashboard/scanner.py` â†’ DB SQLite; `token_dashboard/server.py` expÃµe rotas JSON `/api/*` e serve `web/`.

## Leitura adicional

- [`CLAUDE.md`](CLAUDE.md) â€” convenÃ§Ãµes e visÃ£o geral da arquitetura (tambÃ©m Ã© lido automaticamente pelo Claude Code)
- [`CONTRIBUTING.md`](CONTRIBUTING.md) â€” como desenvolver e testar
- [`docs/KNOWN_LIMITATIONS.md`](docs/KNOWN_LIMITATIONS.md) â€” pontas soltas
- [`docs/inspiration.md`](docs/inspiration.md) â€” trabalho anterior e como este projeto diverge
- [`docs/claude-mem-tuning.md`](docs/claude-mem-tuning.md) â€” caso real de uso do dashboard para auditar e ajustar o plugin claude-mem

## Contribuindo

Veja [`CONTRIBUTING.md`](CONTRIBUTING.md). VersÃ£o curta: faÃ§a fork, rode `python3 -m unittest discover tests` antes de abrir um PR, mantenha sÃ³ stdlib.

## LicenÃ§a

[MIT](LICENSE).

---

Projeto adaptado da comunidade INEMA TDS - Marca Arckhe