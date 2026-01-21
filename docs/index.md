# goax

Bash wrapper for [goaccess](https://goaccess.io/) to generate web statistics from nginx/apache access logs.

## Features

- **Easy setup**: interactive configuration wizard
- **Multiple reports**: all traffic, bots only, no bots
- **Navigation UI**: index.html with menu to switch between reports
- **Bot detection**: filters traffic by 20+ known bot user-agents
- **Protected output**: setup password-protected stats folder
- **Cron-ready**: designed for automated scheduled runs

## Quick Start

```bash
# 1. Install
git clone https://github.com/pforret/goax.git
cd goax

# 2. Install goaccess
./goax.sh install

# 3. Configure
./goax.sh config

# 4. Generate reports
./goax.sh run
```

## Commands

| Command | Description |
|---------|-------------|
| `goax install` | Install goaccess (detects OS package manager) |
| `goax config` | Create configuration file interactively |
| `goax folder [user]` | Setup password-protected output folder |
| `goax run` | Generate HTML reports |
| `goax check` | Show current configuration |

## Generated Reports

The `run` command creates 5 files in your output directory:

| File | Content |
|------|---------|
| `index.html` | Navigation wrapper with menu bar |
| `all.html` | All traffic (bots + humans) |
| `nobots.html` | Human traffic only |
| `bots.html` | All bot traffic |
| `llmbots.html` | LLM/AI bot traffic only |

## Configuration

Run `goax config` to create a `goax.env` file:

```bash
ACCESS_LOG="/var/log/nginx/access.log"
OUTPUT_DIR="/var/www/example.com/stats"
LOG_FORMAT="COMBINED"
```

Or use command-line options:

```bash
goax run -l /var/log/nginx/mysite.access.log -o /var/www/mysite/stats
```

## Bot Detection

The following user-agents are classified as bots:

- Search engines: Googlebot, Bingbot, Yandex, Baidu, DuckDuckBot
- SEO tools: Semrush, Ahrefs, Moz, MJ12bot, DotBot
- Social: FacebookExternalHit, TwitterBot, LinkedInBot, WhatsApp
- Others: Applebot, PetalBot, and generic bot/crawler/spider patterns

## LLM Bot Detection

The `llmbots.html` report specifically tracks AI/LLM crawlers:

- OpenAI: GPTBot, ChatGPT-User
- Anthropic: ClaudeBot, Claude-Web
- Other AI: Bytespider, CCBot, PerplexityBot, Cohere-ai
- Meta: Meta-ExternalAgent
- Others: AmazonBot, YouBot, AI2Bot, Diffbot, Omgili, iaskspider

## Cron Setup

For hourly reports:

```cron
0 * * * * /opt/goax/goax.sh run >> /var/log/goax.log 2>&1
```

## Installation Guides

- [Ubuntu](install/ubuntu.md)
- [DigitalOcean](install/digitalocean.md)
- [Generic VPS](install/vps.md)

## Options

```
-h, --help      Show usage
-Q, --QUIET     No output
-V, --VERBOSE   Debug messages
-f, --FORCE     Skip confirmations
-l, --LOG_FILE  Access log path
-o, --OUTPUT    Output directory
-F, --FORMAT    Log format (COMBINED/COMMON/...)
```

## Requirements

- Bash 4+
- [goaccess](https://goaccess.io/)
- htpasswd (for `folder` command)

## License

MIT
