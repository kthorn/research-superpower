---
name: Evaluating Paper Relevance
description: Two-stage paper screening - abstract scoring then deep dive for specific data extraction
when_to_use: After literature search returns results. When need to determine if paper contains specific data. When screening papers for relevance. When extracting methods, results, data from papers.
version: 1.0.0
---

# Evaluating Paper Relevance

## Overview

Two-stage screening process: quick abstract scoring followed by deep dive into promising papers.

**Core principle:** Precision over breadth. Find papers that actually contain the specific data/methods user needs, not just topically related papers.

## When to Use

Use this skill when:
- Have list of papers from search
- Need to determine which papers have relevant data
- User asks for specific information (IC50 values, methods, etc.)
- Screening papers one-by-one

## Two-Stage Process

### Stage 1: Abstract Screening (Fast)

**Goal:** Quickly identify promising papers

**Score 0-10 based on:**
- **Keywords match (0-3 points)**: Does abstract mention key terms?
- **Data type match (0-4 points)**: Does it mention the data user needs? (e.g., "IC50 values", "synthesis methods")
- **Specificity (0-3 points)**: Is it specific to user's question or just general background?

**Decision rules:**
- Score < 5: Skip (not relevant)
- Score 5-6: Note in summary as "possibly relevant" but skip for now
- Score ≥ 7: Proceed to Stage 2 (deep dive)

**Output to terminal:**
```
📄 [N/Total] Screening: "Paper Title"
   Abstract score: 8 → Fetching full text...
```

or

```
📄 [N/Total] Screening: "Paper Title"
   Abstract score: 4 → Skipping (insufficient relevance)
```

### Stage 2: Deep Dive (Thorough)

**Goal:** Extract specific data/methods from promising papers

#### 1. Fetch Full Text

**Try in order:**

**A. PubMed Central (free full text):**
```bash
# Check if available in PMC
curl "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?db=pmc&term=PMID[PMID]&retmode=json"

# If found, fetch full text XML
curl "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=pmc&id=PMCID&rettype=full&retmode=xml"
```

**B. DOI resolution:**
```bash
# Try publisher link
curl -L "https://doi.org/10.1234/example.2023"
# May hit paywall - check response
```

**C. Preprints:**
- Check bioRxiv: `https://www.biorxiv.org/content/10.1101/{doi}`
- Check arXiv (for computational papers)

**If full text unavailable:**
- Note in SUMMARY.md: "⚠️ Full text behind paywall - evaluated from abstract only"
- Continue with abstract-only evaluation (limited)

#### 2. Scan for Relevant Content

**Focus on sections:**
- **Methods**: Experimental procedures, protocols
- **Results**: Data tables, figures, measurements
- **Tables/Figures**: Often contain the specific data user needs
- **Supplementary Information**: Additional data, extended methods

**What to look for:**
- Specific data user requested (e.g., IC50 = 45 nM)
- Methods described in detail
- Compound structures, identifiers
- Experimental conditions
- Statistical analysis

**Use grep/text search:**
```bash
# Search for specific terms in full text
grep -i "IC50\|inhibitory concentration" paper.xml
grep -i "synthesis\|synthetic route" paper.xml
```

#### 3. Extract Findings

**Create structured extraction:**

```json
{
  "doi": "10.1234/example.2023",
  "title": "Paper Title",
  "relevance_score": 9,
  "findings": {
    "data_found": [
      "IC50 values for compounds 1-12 (Table 2)",
      "Selectivity against 50 kinases (Figure 3)",
      "Synthesis route for lead compound (Scheme 1)"
    ],
    "key_results": [
      "Compound 7: IC50 = 12 nM (BTK), >1000 nM (other kinases)",
      "10-step synthesis, 34% overall yield"
    ],
    "sections": {
      "methods": "Page 3, paragraph 2",
      "results": "Table 2 (page 7), Figure 3 (page 9)",
      "supplementary": "SI Table S1 has full kinase panel"
    }
  }
}
```

#### 4. Download Materials

**PDFs:**
```bash
# If PDF available
curl -L -o "papers/$(echo $doi | tr '/' '_').pdf" "https://doi.org/$doi"
```

**Supplementary data:**
```bash
# Download SI files if URLs found
curl -o "papers/${doi}_supp.zip" "https://publisher.com/supp/file.zip"
```

#### 5. Update Tracking Files

**Add to papers-reviewed.json:**
```json
{
  "10.1234/example.2023": {
    "pmid": "12345678",
    "status": "relevant",
    "score": 9,
    "source": "pubmed_search",
    "timestamp": "2025-10-11T10:30:00Z",
    "found_data": ["IC50 values", "synthesis methods"],
    "has_full_text": true
  }
}
```

**Add to SUMMARY.md:**
```markdown
## Highly Relevant Papers (Score ≥ 8)

### [Paper Title](https://doi.org/10.1234/example.2023) (Score: 9)
**Authors:** Smith et al., 2023
**Key Findings:**
- IC50 values for 12 BTK inhibitors (Table 2)
- Compound 7 shows >80-fold selectivity vs other kinases
- Synthesis described in detail (Scheme 1, page 4)

**Data Location:**
- Main data: Table 2 (page 7), Figure 3 (page 9)
- Supplementary: SI Table S1 (full kinase panel)

**Files:**
- PDF: `papers/10.1234_example.2023.pdf`
- Supplementary: `papers/10.1234_example.2023_supp.zip`

---
```

## Progress Reporting

**Report to user as you work:**
```
📄 [15/127] Screening: "Selective BTK inhibitors..."
   Abstract score: 8 → Fetching full text...
   ✓ Found IC50 data for 8 compounds (Table 2)
   ✓ Selectivity data vs 50 kinases (Figure 3)
   → Added to SUMMARY.md
   → Downloading PDF and supplementary files...
   → Following 3 relevant citations...
```

## Integration with Other Skills

**After finding relevant paper:**
1. **Extract findings** to SUMMARY.md
2. **Download files** to papers/ folder
3. **Call traversing-citations skill** to find related papers
4. **Update papers-reviewed.json** to avoid re-processing

## Scoring Rubric

| Score | Meaning | Action |
|-------|---------|--------|
| 0-4 | Not relevant | Skip, brief note in summary |
| 5-6 | Possibly relevant | Note for later, skip deep dive for now |
| 7-8 | Relevant | Deep dive, extract data, add to summary |
| 9-10 | Highly relevant | Deep dive, extract data, follow citations, highlight in summary |

## Common Mistakes

**Too strict:** Skipping papers that mention data indirectly → Re-read abstract carefully
**Too lenient:** Deep diving into tangentially related papers → Focus on specific data user needs
**Missing supplementary data:** Many papers hide key data in SI → Always check for supplementary files
**Not tracking progress:** User loses context → Report findings as you go
**Re-reviewing papers:** Wastes time → Always check papers-reviewed.json first

## Quick Reference

| Task | Action |
|------|--------|
| Check if reviewed | Look up DOI in papers-reviewed.json |
| Score abstract | Keywords (0-3) + Data type (0-4) + Specificity (0-3) |
| Get full text | Try PMC → DOI → Preprints |
| Find data | Grep for terms, focus on Methods/Results/Tables |
| Download PDF | `curl -L -o papers/FILE.pdf URL` |
| Update tracking | Add to papers-reviewed.json + SUMMARY.md |

## Next Steps

After evaluating paper:
- If score ≥ 7: Call `skills/research/traversing-citations`
- Continue to next paper in search results
- Check if reached 50 papers or 5 minutes → ask user to continue or stop
