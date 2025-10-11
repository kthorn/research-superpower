# Research Superpowers

Give Claude Code superpowers for scientific research. Search literature, evaluate papers, traverse citation networks, and answer research questions systematically.

## What You Get

- **Literature Search** - PubMed and Semantic Scholar integration
- **Smart Paper Evaluation** - Abstract screening + deep dive for relevant data
- **Open Access Finding** - Unpaywall API to find free versions of paywalled papers
- **Citation Traversal** - Intelligent backward and forward citation following
- **Research Orchestration** - End-to-end workflow for answering research questions

## Quick Start

```bash
# Clone to your config directory
git clone https://github.com/yourname/research-superpowers ~/.config/research-superpowers

# Start Claude Code and tell it about your research question
# Example: "Find data on BTK inhibitor selectivity against other kinases"
```

## How It Works

1. **Parse your research question** - Extract keywords, data types, constraints
2. **Search PubMed** - Find relevant papers
3. **Screen abstracts** - Score papers for relevance (0-10)
4. **Deep dive on promising papers** - Extract specific data and methods
5. **Traverse citations** - Follow relevant references and citing papers (via Semantic Scholar)
6. **Track everything** - Maintain folder with findings, papers, and deduplication

## Project Structure

```
research-superpowers/
├── skills/
│   ├── getting-started/        # Introduction and workflow overview
│   └── research/
│       ├── answering-research-questions/    # Main orchestration
│       ├── searching-literature/            # PubMed search
│       ├── evaluating-paper-relevance/      # Abstract screening + deep dive
│       └── traversing-citations/            # Citation network traversal
├── scripts/
│   └── find-skills              # Search for relevant skills
└── hooks/
    └── session-start.sh         # Auto-load at Claude Code startup
```

## Research Session Output

Each research query creates a folder:

```
research-sessions/2025-10-11-btk-inhibitor-selectivity/
├── SUMMARY.md                   # Main findings organized by relevance
├── papers-reviewed.json         # Deduplication tracking
├── papers/
│   ├── 10.1234_example.pdf
│   └── ...
└── citations/
    └── citation-graph.json      # Citation relationships
```

## Skills Library

### Research Skills

- **answering-research-questions** - Main orchestration workflow
- **searching-literature** - PubMed API integration
- **evaluating-paper-relevance** - Two-stage relevance filtering
- **traversing-citations** - Smart citation following via Semantic Scholar

## Philosophy

- **Precision over breadth** - Find papers with specific data, not just topically related
- **Smart traversal** - Only follow relevant citations to avoid exponential explosion
- **Track everything** - Deduplicate and organize findings
- **Check in regularly** - Every 50 papers or 5 minutes

## Requirements

- Claude Code CLI
- Internet connection (for PubMed and Semantic Scholar APIs)
- Optional: API keys for higher rate limits (free tier works for most use cases)

## Reducing Command Prompts

When Claude runs API calls (curl commands), you may see approval prompts. To eliminate these for research sessions:

### Option 1: Pre-configure permissions (Recommended)

Copy the template permissions file to your research project:

```bash
# In your research project directory (e.g., research-sessions/YYYY-MM-DD-query/)
mkdir -p .claude
cp ~/.config/research-superpowers/.claude/settings.local.json.template .claude/settings.local.json
```

This pre-approves:
- All PubMed/NCBI API calls
- All Semantic Scholar API calls
- All Unpaywall API calls
- DOI resolution calls
- Basic file operations

### Option 2: Per-command approval

When prompted, choose option 2: "Yes, and don't ask again for similar commands in this directory"

**Note:** If this doesn't work (keeps prompting), use Option 1 instead. Claude Code may add specific commands instead of patterns.

## API Information

**PubMed/E-utilities:**
- Free, no API key required
- Rate limit: 3 requests/second without key, 10 req/sec with key

**Semantic Scholar:**
- Free tier: 100 requests per 5 minutes
- API key (free): 1000 requests per 5 minutes
- [Get API key](https://www.semanticscholar.org/product/api#api-key)

**Unpaywall:**
- Free: 100,000 requests per day
- No API key required (just provide email)
- [Learn more](https://unpaywall.org/products/api)

## Contributing

This is an experimental project! Contributions welcome:
- Try it with real research questions
- Report issues and edge cases
- Suggest new skills
- Improve existing workflows

## License

MIT License - see LICENSE file

## Acknowledgments

Inspired by [Superpowers](https://github.com/obra/superpowers) by Jesse Vincent
