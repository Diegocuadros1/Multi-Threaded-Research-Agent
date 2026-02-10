# Company Research Intelligence Platform

An AI-powered competitive intelligence engine that autonomously researches companies by orchestrating LLM agents, real-time web search, and SEC filing analysis — delivering structured, citation-backed reports in seconds via CLI or Discord.

## Key Highlights

- **Multi-Agent Research Pipeline** — Five specialized AI personas execute in parallel, each targeting a distinct research angle (company overview, strategic initiatives, 5-year competitive history, tech stack analysis via job postings, and investor materials collection).
- **Real-Time Web Intelligence** — Integrates the Tavily search API with automatic query generation, result deduplication, and full-page content extraction via `httpx` + `BeautifulSoup` to ground every answer in live data.
- **SEC Edgar Integration** — Programmatically fetches and filters SEC filings (10-Q, 10-K, 8-K) by type and date range, providing direct access to public company financial disclosures.
- **Concurrent Execution Engine** — Combines `asyncio`, `AsyncOpenAI`, and `ThreadPoolExecutor` to run all research streams in parallel, reducing end-to-end latency by up to 5x compared to sequential execution.
- **Discord Bot Interface** — Production-ready Discord bot with semaphore-controlled concurrency, chunked message delivery, and execution-time reporting for team-wide access to research capabilities.

## Architecture

```
                         ┌──────────────┐
                         │  Discord Bot │  ← !research <company>
                         │   (bot.py)   │  ← !sec <cik>
                         └──────┬───────┘
                                │
                    ┌───────────┴───────────┐
                    │    main.py (CLI)      │
                    │  ThreadPoolExecutor   │
                    └───────────┬───────────┘
                                │
               ┌────────────────┼────────────────┐
               │                │                 │
        ┌──────┴──────┐  ┌─────┴──────┐  ┌──────┴───────┐
        │  prompts.py │  │ sec_files  │  │ run_prompts  │
        │  5 Research │  │ SEC Edgar  │  │  AI Research │
        │   Personas  │  │   API      │  │    Engine    │
        └─────────────┘  └────────────┘  └──────┬───────┘
                                                 │
                              ┌──────────────────┼──────────────┐
                              │                  │              │
                        ┌─────┴─────┐    ┌───────┴────┐  ┌──────┴──┐
                        │  OpenAI   │    │   Tavily   │  │  httpx  │
                        │  GPT-5    │    │  Search    │  │ Scraper │
                        │  (query   │    │  (web      │  │ (full   │
                        │  gen +    │    │  results)  │  │  page   │
                        │  synth)   │    │            │  │  fetch) │
                        └───────────┘    └────────────┘  └─────────┘
```

## Research Prompts

Each research prompt is crafted with a domain-expert persona to maximize output quality:

| Prompt | Persona | Output |
|--------|---------|--------|
| **Company Overview** | Enterprise SaaS Founder | HQ, headcount, ARR, industries, top 10 customers, competitors |
| **Strategic Initiatives** | Chief Research Officer | Plans, pains, risks, user quotes, solution gaps |
| **5-Year History** | Competitive Intelligence Analyst | M&A, funding, C-suite changes, layoffs, product launches |
| **Job Postings Analysis** | Competitive Intelligence Analyst | Tech stack (PSA/ERP/CRM), hiring trends, strategic signals |
| **Investor Materials** | Financial Analyst | Earnings decks, transcripts, investor day presentations |

## How It Works

1. **Query Generation** — The LLM analyzes each research prompt and proposes 3-6 targeted search queries with optimal freshness parameters.
2. **Parallel Web Search** — Tavily executes all queries concurrently, returning ranked results with raw content.
3. **Source Enrichment** — Any result missing full text is fetched directly via `httpx`, parsed with BeautifulSoup, and cleaned of non-visible elements.
4. **Deduplication** — Results are deduplicated by URL and domain to ensure breadth of sourcing.
5. **Synthesis** — The LLM synthesizes all sources into a structured, citation-backed report with inline `[1]`, `[2]` references.
6. **Delivery** — Results stream back to the user via CLI stdout or Discord messages (auto-chunked to respect Discord's 2000-char limit).

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Language | Python 3.12 |
| LLM | OpenAI GPT-5 (async) |
| Web Search | Tavily API (advanced depth) |
| Web Scraping | httpx, BeautifulSoup4 |
| Financial Data | SEC Edgar REST API |
| Bot Framework | discord.py |
| Concurrency | asyncio, ThreadPoolExecutor |

## Usage

### CLI
```bash
python main.py
```
Runs all five research prompts concurrently for a target company and prints results to stdout.

### Discord Bot
```bash
python bot.py
```

| Command | Description |
|---------|-------------|
| `!research <company>` | Run full research pipeline (5 parallel prompts) |
| `!sec <cik>` | Fetch and filter SEC filings by CIK number |
| `!hello` | Health check |

### Environment Variables

```
OPENAI_API_KEY=       # OpenAI API key
TAVILY_API_KEY=       # Tavily search API key
DISCORD_TOKEN=        # Discord bot token (bot.py only)
```

## Project Structure

```
company_research/
├── main.py            # CLI entry point — concurrent research execution
├── bot.py             # Discord bot — team-accessible research interface
├── run_prompts.py     # Core engine — search, fetch, dedupe, synthesize
├── api/
│   ├── prompts.py     # 5 expert-persona research prompt templates
│   └── sec_files.py   # SEC Edgar API client with filing filters
└── test.py            # Concurrency testing utility
```
