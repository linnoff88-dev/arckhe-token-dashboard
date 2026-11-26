ï»¿# Token Dashboard

Um dashboard local que lêÂª as transcriêÂ§êÂµes JSONL que o Claude Code grava em `~/.claude/projects/` e as transforma em anêÂ¡lise de custo por prompt, mapas de calor de ferramentas/arquivos, atribuiêÂ§êÂ£o de subagentes, anêÂ¡lise de cache, comparaêÂ§êÂ£o entre projetos e um motor de dicas baseado em regras.

**Tudo roda localmente.** Nenhum dado sai da sua mêÂ¡quina ê¢â¬â sem telemetria, sem chamadas de API com seus dados, sem login.

![Aba Overview ê¢â¬â totais e grêÂ¡ficos diêÂ¡rios](docs/images/dashboard-overview-top.jpg)

![Aba Overview ê¢â¬â por projeto, por modelo, top de ferramentas, sessêÂµes recentes](docs/images/dashboard-overview-bottom.jpg)

## Para que isto serve

- Ver quais dos seus prompts sêÂ£o caros (surpresa: geralmente envolvem resultados grandes de ferramentas).
- Comparar uso de tokens entre projetos em que vocêÂª trabalhou.
- Identificar padrêÂµes desperdiêÂ§adores ê¢â¬â o mesmo arquivo lido vinte vezes em uma sessêÂ£o, uma chamada de ferramenta retornando 80k tokens.
- Entender o que um "cache hit" realmente economiza.
- Se vocêÂª estêÂ¡ no Pro ou Max, confirmar que estêÂ¡ tendo retorno do dinheiro em dêÂ³lares equivalentes êÂ  API.

## PrêÂ©-requisitos

- **Python 3.8 ou mais novo** ê¢â¬â jêÂ¡ instalado no macOS e na maioria do Linux. No Windows: `winget install Python.Python.3.12` ou baixe de python.org.
- **Claude Code** ê¢â¬â instalado e com pelo menos uma sessêÂ£o executada. O dashboard lêÂª essas sessêÂµes. Se vocêÂª acabou de instalar o Claude Code e ainda nêÂ£o o usou, rode pelo menos um prompt primeiro.
- **Um navegador web.** Qualquer um moderno.

Sem `pip install`. Sem Node.js. Sem etapa de build.

## InêÂ­cio rêÂ¡pido

```bash
git clone https://github.com/linnoff88-dev/token-dashboard.git
cd token-dashboard
python3 cli.py dashboard
```

> No Windows, se `python3` nêÂ£o estiver no PATH, substitua `py -3` por `python3` em todos os comandos abaixo.

O comando:
1. Varre `~/.claude/projects/` (a primeira execuêÂ§êÂ£o pode levar 20ê¢â¬â60 segundos na mêÂ¡quina de um usuêÂ¡rio pesado).
2. Inicia um servidor local em http://127.0.0.1:8080.
3. Abre seu navegador padrêÂ£o nessa URL.

Deixe rodando; ele re-varre a cada 30 segundos e envia atualizaêÂ§êÂµes ao vivo. Pare com `Ctrl+C`.

## De onde vêÂªm os dados

O Claude Code grava um arquivo JSONL por sessêÂ£o aqui:

| SO | Caminho |
|---|---|
| macOS / Linux | `~/.claude/projects/<project-slug>/<session-id>.jsonl` |
| Windows | `C:\Users\<you>\.claude\projects\<project-slug>\<session-id>.jsonl` |

O dashboard nunca modifica esses arquivos ê¢â¬â apenas os lêÂª e mantêÂ©m um cache SQLite local em `~/.claude/token-dashboard.db`.

Para apontar para outro local:

```bash
python3 cli.py dashboard --projects-dir /path/to/projects --db /path/to/cache.db
```

### VariêÂ¡veis de ambiente

| Var | PadrêÂ£o | PropêÂ³sito |
|---|---|---|
| `PORT` | `8080` | Porta em que o servidor web local escuta |
| `HOST` | `127.0.0.1` | EndereêÂ§o de bind. Mantenha o padrêÂ£o. Definir `0.0.0.0` expêÂµe todo o seu histêÂ³rico de prompts a qualquer um na sua rede local ê¢â¬â nêÂ£o faêÂ§a isso em nenhuma rede que vocêÂª nêÂ£o controle totalmente (nada de Wi-Fi de cafeteria, nada de coworking). |
| `CLAUDE_PROJECTS_DIR` | `~/.claude/projects` | Onde varrer os arquivos JSONL de sessêÂ£o |
| `TOKEN_DASHBOARD_DB` | `~/.claude/token-dashboard.db` | Local do cache SQLite |

Os preêÂ§os ficam em [`pricing.json`](pricing.json). Edite-o diretamente se os preêÂ§os dos modelos mudarem ou para adicionar um novo plano.

## ReferêÂªncia da CLI

```bash
python3 cli.py scan          # popula / atualiza o DB local e sai
python3 cli.py today         # totais de hoje (terminal)
python3 cli.py stats         # totais de todo o perêÂ­odo (terminal)
python3 cli.py tips          # sugestêÂµes ativas (terminal)
python3 cli.py dashboard     # scan + serve a UI em http://localhost:8080

# flags do dashboard
python3 cli.py dashboard --no-open   # nêÂ£o abre o navegador automaticamente
python3 cli.py dashboard --no-scan   # pula o scan inicial (usa sêÂ³ o DB em cache)
```

Trocar a porta: `PORT=9000 python3 cli.py dashboard`.

## As 7 abas

O dashboard êÂ© uma pêÂ¡gina êÂºnica com uma barra de abas no topo usando hash-router. Cada aba êÂ© alimentada por sua prêÂ³pria API JSON em `/api/`:

- **Overview** ê¢â¬â tokens de input/output/cache de todo o perêÂ­odo, sessêÂµes, turnos, custo estimado no plano escolhido, grêÂ¡ficos diêÂ¡rios de trabalho e leitura de cache, tokens por projeto, participaêÂ§êÂ£o de tokens por modelo, top de ferramentas por contagem de chamadas e sessêÂµes recentes. Esta êÂ© a aba inicial.
- **Prompts** ê¢â¬â seus prompts de usuêÂ¡rio mais caros, ranqueados por tokens. Clique em qualquer linha para ver a resposta do assistente, as chamadas de ferramenta feitas e o tamanho de cada resultado de ferramenta.
- **Sessions** ê¢â¬â visêÂ£o turno a turno de qualquer sessêÂ£o, com tokens e chamadas de ferramenta por turno.
- **Projects** ê¢â¬â comparaêÂ§êÂ£o por projeto: tokens, contagem de sessêÂµes e quais arquivos foram mais tocados.
- **Skills** ê¢â¬â quais skills vocêÂª invoca com mais frequêÂªncia e (onde dêÂ¡ pra medir) seu custo em tokens. Veja [limitaêÂ§êÂµes](docs/KNOWN_LIMITATIONS.md#skills-token-counts-are-partial).
- **Tips** ê¢â¬â sugestêÂµes baseadas em regras para reduzir uso de tokens (leituras repetidas de arquivos, resultados de ferramentas grandes demais, taxa baixa de cache hit, etc.).
- **Settings** ê¢â¬â alterna o pricing entre API / Pro / Max / Max-20x para que os valores de custo em todo o resto reflitam seu plano real.

A aba Overview tambêÂ©m tem um painel embutido "What do these numbers mean?" que explica tokens de input/output/cache em linguagem simples.

## SoluêÂ§êÂ£o de problemas

**"Sem dados" ou grêÂ¡ficos vazios.** Rode `python3 cli.py scan` uma vez para popular o DB e recarregue.

**Porta 8080 jêÂ¡ em uso.** `PORT=9000 python3 cli.py dashboard`.

**NêÂºmeros parecem errados / travados.** O DB fica em `~/.claude/token-dashboard.db`. Apague e rode `python3 cli.py scan` para reconstruir do zero.

**Rodar o dashboard duas vezes ao mesmo tempo.** NêÂ£o faêÂ§a ê¢â¬â os dois processos vêÂ£o brigar pelo DB SQLite. Pare todas as instêÂ¢ncias antes de iniciar uma nova.

## Nota sobre precisêÂ£o

O Claude Code grava cada resposta do assistente 2ê¢â¬â3 vezes em disco enquanto ela faz streaming (a mesma mensagem da API êÂ© snapshotada conforme o output cresce). O dashboard deduplica isso por `message.id` para que o total final bata com o que a API realmente cobrou. Se vocêÂª comparar com outra ferramenta que soma cada linha do JSONL, espere que os nêÂºmeros deste dashboard sejam menores ê¢â¬â e mais prêÂ³ximos da realidade.

## Privacidade

Nada sai da sua mêÂ¡quina. Sem telemetria. Sem chamadas remotas com seus dados. O navegador busca seu JSON de `127.0.0.1`, e todo JS/CSS/fonte êÂ© servido por esse mesmo servidor local ê¢â¬â o ECharts estêÂ¡ vendorizado em `web/`, e a UI usa fontes do sistema como fallback em vez de buscar de uma CDN de fontes. Se quiser verificar: `grep -r "https://" token_dashboard/ web/` ê¢â¬â nêÂ£o vai encontrar nada.

## Stack têÂ©cnica

Python 3 (sêÂ³ stdlib) para a CLI, scanner e servidor HTTP. SQLite para o cache local. Vanilla JS + ECharts para a UI, sem etapa de build. Tema escuro, router baseado em hash, server-sent events para refresh ao vivo.

Fluxo de dados: `cli.py` ê¢â â `token_dashboard/scanner.py` ê¢â â DB SQLite; `token_dashboard/server.py` expêÂµe rotas JSON `/api/*` e serve `web/`.

## Leitura adicional

- [`CLAUDE.md`](CLAUDE.md) ê¢â¬â convenêÂ§êÂµes e visêÂ£o geral da arquitetura (tambêÂ©m êÂ© lido automaticamente pelo Claude Code)
- [`CONTRIBUTING.md`](CONTRIBUTING.md) ê¢â¬â como desenvolver e testar
- [`docs/KNOWN_LIMITATIONS.md`](docs/KNOWN_LIMITATIONS.md) ê¢â¬â pontas soltas
- [`docs/inspiration.md`](docs/inspiration.md) ê¢â¬â trabalho anterior e como este projeto diverge
- [`docs/claude-mem-tuning.md`](docs/claude-mem-tuning.md) ê¢â¬â caso real de uso do dashboard para auditar e ajustar o plugin claude-mem

## Contribuindo

Veja [`CONTRIBUTING.md`](CONTRIBUTING.md). VersêÂ£o curta: faêÂ§a fork, rode `python3 -m unittest discover tests` antes de abrir um PR, mantenha sêÂ³ stdlib.

## LicenêÂ§a

[MIT](LICENSE).

---

Projeto  - ARCKHÉe