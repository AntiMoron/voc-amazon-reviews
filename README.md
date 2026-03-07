# voc-amazon-reviews

Amazon review VOC (Voice of Customer) analysis skill for [Claude Code](https://claude.ai/code) and [OpenClaw](https://openclaw.ai). Input an ASIN, get deep bilingual insights — straight from your terminal.

## What it does

Scrapes Amazon customer reviews using a real browser (bypassing anti-bot) and runs them through Claude for semantic VOC analysis. No keyword counting — actual language understanding.

- **Sentiment breakdown** — positive / neutral / negative with percentages
- **Top 5 pain points** — what buyers complain about, with real quotes
- **Top 5 selling points** — what buyers love, with real quotes
- **Listing optimization tips** — actionable copy suggestions backed by review data
- **Bilingual output** — every insight in both Chinese and English

## Install

### Claude Code
```bash
mkdir -p .claude/skills
cd .claude/skills
git clone https://github.com/mguozhen/voc-amazon-reviews.git voc-amazon-reviews
```

### OpenClaw
```bash
mkdir -p skills
cd skills
git clone https://github.com/mguozhen/voc-amazon-reviews.git voc-amazon-reviews
```

## Setup

1. **browser skill** — required for Amazon scraping:
   ```bash
   npx skills add browserbase/skills@browser
   ```

2. **Browserbase account** *(recommended)* — handles Amazon's anti-bot, CAPTCHAs, and residential proxies. [Sign up free](https://browserbase.com).
   ```bash
   export BROWSERBASE_API_KEY="your-key"
   export BROWSERBASE_PROJECT_ID="your-project-id"
   browse env remote
   ```
   Without Browserbase, the scraper runs on local Chrome but may get blocked by Amazon sign-in walls.

3. **Anthropic API Key** — for Claude-powered analysis:
   ```bash
   export ANTHROPIC_API_KEY="sk-ant-..."
   ```

## Usage

### Natural language (just talk to Claude)
- "Analyze the reviews for ASIN B08N5WRWNW"
- "Do a VOC analysis on this product: B0F6VWT6SP"
- "What are customers complaining about for B09G9HD6PD?"
- "Find the top selling points from Amazon reviews for B08XYZ"

### CLI commands
```bash
# Basic analysis (scrapes 100 reviews)
bash skills/voc-amazon-reviews/voc.sh B08N5WRWNW

# Scrape more reviews for deeper analysis
bash skills/voc-amazon-reviews/voc.sh B08N5WRWNW --limit 200

# UK marketplace
bash skills/voc-amazon-reviews/voc.sh B08N5WRWNW --market amazon.co.uk

# Save report to file
bash skills/voc-amazon-reviews/voc.sh B08N5WRWNW --output report.md
```

## Sample output

```
╔══════════════════════════════════════════════════════════════╗
║          VOC AI 分析报告 / VOC AI Analysis Report           ║
║  ASIN: B08N5WRWNW  |  分析评论: 100 条                     ║
║  产品: Example Product  |  生成时间: 2026-03-08             ║
╚══════════════════════════════════════════════════════════════╝

📊 情感分布 / Sentiment Distribution
  正面 Positive  ████████████████░░░░  74%
  中性 Neutral   ███░░░░░░░░░░░░░░░░░  16%
  负面 Negative  ██░░░░░░░░░░░░░░░░░░  10%

🔴 Top 5 痛点 / Pain Points
═══════════════════════════════════════════════
1. 电池续航不足 / Short battery life（28条提及）
   「只用了两天电池就耗尽了」
   "Battery drained in 2 days, very disappointed"
...

🟢 Top 5 卖点 / Selling Points
═══════════════════════════════════════════════
1. 音质出色 / Excellent sound quality（52条提及）
   「低音浑厚，高音清晰，性价比极高」
   "Amazing bass and crystal clear highs for the price"
...

💡 Listing 优化建议 / Optimization Suggestions
═══════════════════════════════════════════════
1. 在标题中明确标注电池容量，减少因预期不匹配的差评
   Add battery capacity to title to reduce mismatch-driven 1-stars
...
```

## Options

```
--limit N          Number of reviews to scrape (default: 100)
--market DOMAIN    Amazon marketplace (default: amazon.com)
                   Options: amazon.co.uk, amazon.de, amazon.co.jp, etc.
--output FILE      Save report to markdown file
--help             Show help
```

## How it works

```
① Input ASIN
      ↓
② browse CLI opens Amazon review page
   (Browserbase: stealth mode + residential proxy)
      ↓
③ Paginated scraping — ratings, titles, review bodies, dates
      ↓
④ Claude semantic analysis
   (not keyword counting — actual language understanding)
      ↓
⑤ Bilingual structured report
```

## File structure

```
voc-amazon-reviews/
├── SKILL.md       # Skill definition (Claude reads this)
├── voc.sh         # Main entry point
├── scraper.sh     # Amazon review scraper (uses browse CLI)
└── analyze.sh     # Claude analysis + bilingual report renderer
```

## Supported marketplaces

| Marketplace | Domain |
|---|---|
| United States | amazon.com |
| United Kingdom | amazon.co.uk |
| Germany | amazon.de |
| Japan | amazon.co.jp |
| Canada | amazon.ca |
| France | amazon.fr |

## Why not use the Amazon API?

Amazon's Product Advertising API doesn't provide review text — only aggregate ratings. Seller Central exports are manual and incomplete. Real browser scraping is the only way to get the actual voice of your customers.

## Security

**API keys:** `ANTHROPIC_API_KEY` and `BROWSERBASE_API_KEY` are read from environment variables and never written to disk or printed to stdout. Be aware that AI agent session logs may capture env vars passed via tool calls — use shell-level exports, not inline assignments.

**Amazon ToS:** This tool accesses publicly visible review pages the same way a browser does. Use responsibly — avoid high-frequency scraping of the same ASIN.

## Cost

| Component | Cost |
|---|---|
| Claude analysis (100 reviews) | ~$0.01–0.03 per run |
| Browserbase (remote browser) | ~$0.01 per session |
| **Total per ASIN** | **~$0.02–0.05** |

Running on local Chrome (no Browserbase) is free but may be blocked by Amazon.

## License

MIT
