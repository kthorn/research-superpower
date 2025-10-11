---
name: Getting Started with Research Superpowers
description: Introduction to research skills - literature search, paper evaluation, citation traversal workflows
when_to_use: At start of each Claude Code session. When user asks research questions. When searching scientific literature. When analyzing papers or citations.
version: 1.0.0
---

# Getting Started with Research Superpowers

Research Superpowers gives Claude Code systematic workflows for scientific literature research.

## What You Can Do

- **Search literature** - PubMed and Semantic Scholar integration
- **Evaluate papers** - Two-stage screening (abstract → deep dive)
- **Extract data** - Find specific methods, results, data from papers
- **Traverse citations** - Smart backward/forward citation following
- **Track findings** - Organized folders with summaries and papers

## Available Skills

**Research Skills** (`skills/research/`)
- **answering-research-questions** - Main orchestration workflow for research queries
- **searching-literature** - PubMed search with keyword optimization
- **evaluating-paper-relevance** - Abstract screening + full text analysis
- **checking-chembl** - Check if medicinal chemistry papers have curated SAR data in ChEMBL
- **traversing-citations** - Semantic Scholar citation network traversal
- **finding-open-access-papers** - Unpaywall API to find free versions of paywalled papers

## Basic Workflow

When user asks a research question:

1. **Read answering-research-questions skill** - Main orchestration
2. **Announce**: "I'm using the Answering Research Questions skill"
3. **Parse query** - Extract keywords, data types, constraints
4. **Create research folder** - Propose name, initialize tracking
5. **Search → Evaluate → Traverse** - Follow the workflow
6. **Check in regularly** - Every 50 papers or 5 minutes

## Research Session Folders

Each query creates a folder in `research-sessions/`:

```
research-sessions/YYYY-MM-DD-query-description/
├── SUMMARY.md              # Main findings
├── papers-reviewed.json    # Deduplication tracking (DOI → status)
├── papers/                 # Downloaded PDFs and supplementary data
└── citations/              # Citation graph tracking
```

## Core Principles

- **Precision over breadth** - Find papers with specific data, not just topical matches
- **Smart citation following** - Only traverse relevant citations
- **Deduplicate aggressively** - Track all reviewed papers by DOI
- **Report progress** - Update user as work proceeds
- **Checkpoint frequently** - Ask to continue or stop

## API Information

**PubMed E-utilities** (no key required):
- Search: `https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi`
- Details: `https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esummary.fcgi`
- Full text: `https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi`

**Semantic Scholar** (free tier works, optional key for higher limits):
- Paper: `https://api.semanticscholar.org/graph/v1/paper/DOI:{doi}`
- References: `https://api.semanticscholar.org/graph/v1/paper/{id}/references`
- Citations: `https://api.semanticscholar.org/graph/v1/paper/{id}/citations`

## Finding Skills

Use the find-skills script to search for relevant skills:

```bash
# From project directory
./scripts/find-skills              # List all skills
./scripts/find-skills literature   # Search for "literature"
./scripts/find-skills 'cite|ref'   # Regex search
```

## Remember

- **Always start** by reading the relevant research skill
- **Announce skill usage** when you begin
- **Track everything** in the research folder
- **Check in with user** regularly during long searches
- **Deduplicate** using papers-reviewed.json (DOI as key)
