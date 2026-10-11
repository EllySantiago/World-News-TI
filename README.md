# World-News: Discord Intel & Tech News Bots

Automações em Python, executadas pelo **GitHub Actions**, que leem feeds RSS de portais globais, **resumem cada matéria em um parágrafo (5 a 7 frases)**, traduzem para PT-BR e publicam em canais do **Discord** via Webhook. O link original vai junto, para você abrir a matéria completa se o resumo interessar.

---

## Canais e fontes

| Canal | Comando | Fontes |
|---|---|---|
| Cibersegurança | `python bot.py cyber` | The Hacker News, BleepingComputer, The Record, SecurityWeek, CISO Advisor |
| IA & OSINT | `python bot.py ia_osint` | OpenAI Blog, Hugging Face Blog, TechCrunch AI, Google AI Blog, Nmap, KitPloit, Darknet |
| Frontend | `python bot.py front` | Smashing Magazine, CSS-Tricks, Dev.to, React Blog, LogRocket |

Para adicionar ou remover fontes, edite só o `feeds.json`.

### Monitor de palavras-chave

Cada canal aceita uma lista `"watch"` no `feeds.json` (o de cyber começa com `ShinyHunters`). Matéria cujo título ou trecho do feed contenha uma dessas palavras:
- ganha o prefixo `🚨 palavra` no título;
- passa por cima do limite de matérias por fonte.

---

## Horários (automático, todos os dias)

| Workflow | UTC | Brasília |
|---|---|---|
| `cyber_news.yml` | 09:00 | 06:00 |
| `ia_osint_news.yml` | 09:15 | 06:15 |
| `front_news.yml` | 09:30 | 06:30 |

O GitHub pode atrasar alguns minutos. Qualquer um pode ser disparado na hora em Actions > escolher o workflow > Run workflow. Marque `dry_run` para só ver os posts no log, sem enviar ao Discord.

---

## Como o resumo é feito

1. Baixa a página da matéria e extrai o texto principal (`trafilatura`). Se o site bloquear o bot, usa o texto do feed.
2. **Sem chave de API:** escolhe as frases mais importantes (começo da matéria, palavras repetidas, palavras do título, números e CVEs) e traduz.
3. **Com chave** (`ANTHROPIC_API_KEY` ou `GEMINI_API_KEY`): uma IA escreve o resumo direto em PT-BR. Se falhar, cai no passo 2.

O nome e o avatar do bot são os que você configurou no webhook do Discord (o script não envia `username`).
O bot guarda o que já enviou em `state/<canal>.json`, então não repete matéria.

---

## Configuração do zero

1. **Secrets** (Settings > Secrets and variables > Actions):

   | Secret | Para quê |
   |---|---|
   | `DISCORD_WEBHOOK_URL` | canal de Cibersegurança |
   | `DISCORD_WEBHOOK_IA_OSINT` | canal de IA & OSINT |
   | `DISCORD_WEBHOOK_FRONT` | canal de Frontend |
   | `ANTHROPIC_API_KEY` ou `GEMINI_API_KEY` | opcional, resumo por IA |

2. **Permissão de escrita** (Settings > Actions > General > Workflow permissions): marque *Read and write permissions*, para o bot salvar o `state/`.
3. Rode um workflow manualmente com `dry_run` para conferir o log e depois rode de verdade.

O repositório pode ser público: as URLs dos webhooks ficam nos Secrets e não aparecem no código nem nos logs.

---

## Testar no computador

```bash
pip install -r requirements.txt
python bot.py --dry-run           # todos os canais, sem enviar nem gravar estado
python bot.py cyber --dry-run     # só um canal
```

Ajustes por variável de ambiente: `SUMMARY_CHARS` (900), `SUMMARY_SENTENCES` (7), `MAX_PER_FEED` (2), `MAX_AGE_HOURS` (30), `LLM_MODEL`.

---

## Estrutura

```text
├── .github/workflows/
│   ├── cyber_news.yml       # 06:00 BRT
│   ├── ia_osint_news.yml    # 06:15 BRT
│   ├── front_news.yml       # 06:30 BRT
│   └── _run_channel.yml     # passos compartilhados pelos três
├── state/                   # o que já foi enviado, um arquivo por canal
├── bot.py                   # único script (substitui main.py, ia_osint.py e front.py)
├── feeds.json               # canais, fontes, cores, categorias e palavras monitoradas
├── requirements.txt
└── README.md
```

Tecnologias: Python 3.12, GitHub Actions, Feedparser, Trafilatura, BeautifulSoup4, Deep-Translator, Discord Webhooks.
