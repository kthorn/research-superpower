---
name: Answering Research Questions
description: Main orchestration workflow for systematic literature research - search, evaluate, traverse, synthesize
when_to_use: When user asks a research question. When user wants to find specific data in literature. When starting comprehensive literature review. When user says "find papers about" or "what is known about".
version: 1.0.0
---

# Answering Research Questions

## Overview

Orchestrate the complete research workflow from query to findings.

**Core principle:** Systematic, trackable, comprehensive. Search → Evaluate → Traverse → Synthesize.

**Announce at start:** "I'm using the Answering Research Questions skill to find [specific data] about [topic]."

## The Process

### Phase 1: Parse Query

Extract from user's request:

**Keywords:**
- Main concepts (e.g., "BTK inhibitor", "selectivity")
- Synonyms and alternatives (e.g., "Bruton tyrosine kinase")
- Related terms (e.g., "off-target", "kinase panel")

**Data types needed:**
- Specific measurements (IC50, KD, EC50, etc.)
- Methods or protocols
- Structures or sequences
- Results or conclusions

**Constraints:**
- Date ranges
- Specific compounds/targets
- Organisms or systems
- Publication types

**Ask clarifying questions if needed:**
- "Are you looking for in vitro or in vivo data?"
- "Any specific time frame?"
- "Which kinases are you most interested in?"

### Phase 2: Initialize Research Session

**Propose folder name:**
```
research-sessions/YYYY-MM-DD-brief-description/
```

Example: `research-sessions/2025-10-11-btk-inhibitor-selectivity/`

**Show proposal to user:**
```
📁 Creating research folder: research-sessions/2025-10-11-btk-inhibitor-selectivity/
   Proceed? (y/n)
```

**Create folder structure:**
```bash
mkdir -p "research-sessions/YYYY-MM-DD-description"/{papers,citations}
```

**Initialize files:**

**papers-reviewed.json:**
```json
{}
```

**citations/citation-graph.json:**
```json
{}
```

**SUMMARY.md:**
```markdown
# Research Query: [User's question]

**Started:** YYYY-MM-DD HH:MM
**Keywords:** keyword1, keyword2, keyword3
**Data types sought:** IC50 values, selectivity data, synthesis methods

---

## Highly Relevant Papers (Score ≥ 8)

(Papers will be added here as found)

---

## Relevant Papers (Score 7)

(Papers will be added here as found)

---

## Possibly Relevant Papers (Score 5-6)

(Noted for potential follow-up)

---

## Search Progress

- Initial PubMed search: X results
- Papers reviewed: Y
- Papers with relevant data: Z
- Citations followed: N

---

## Key Findings

(Synthesized findings will be added as research progresses)
```

### Phase 3: Search Literature

**Use searching-literature skill:**

1. Construct PubMed query from keywords
2. Execute search (start with 100 results)
3. Save results to `initial-search-results.json`
4. Report: "🔎 Found N papers matching query"

### Phase 4: Evaluate Papers

**Use evaluating-paper-relevance skill:**

For each paper:
1. Check papers-reviewed.json (skip if already processed)
2. Stage 1: Score abstract (0-10)
3. If score ≥ 7: Stage 2 deep dive
4. Extract findings to SUMMARY.md
5. Download PDF and supplementary if available
6. Update papers-reviewed.json
7. If score ≥ 7: proceed to Phase 5 for this paper

**Report progress:**
```
📄 [15/100] Screening: "Paper Title"
   Abstract score: 8 → Fetching full text...
   ✓ Found IC50 data for 8 compounds
   → Added to SUMMARY.md
```

### Phase 5: Traverse Citations

**Use traversing-citations skill:**

For papers scoring ≥ 7:
1. Get references (backward)
2. Get citations (forward)
3. Filter for relevance (score ≥ 5)
4. Add to processing queue
5. Evaluate queued papers (return to Phase 4)

**Report progress:**
```
🔗 Following citations from highly relevant paper
   → Found 12 relevant references
   → Found 8 relevant citing papers
   → Adding 20 papers to queue
```

### Phase 6: Checkpoint

**Check after:**
- Every 50 papers reviewed
- Every 5 minutes of processing
- Queue exhausted

**Ask user:**
```
⏸️  Checkpoint: Reviewed 50 papers, found 12 relevant
    Papers with data: 7
    Continue searching? (y/n/summary)
```

**Options:**
- `y` - Continue processing
- `n` - Stop and finalize
- `summary` - Show current findings, then decide

### Phase 7: Synthesize Findings

**When stopping (user says no or queue empty):**

**Option A: Manual synthesis (small research sessions)**
1. **Review SUMMARY.md** - Organize by relevance and topic
2. **Extract key findings** - Group by data type
3. **Add synthesis section:**

```markdown
## Key Findings Summary

### IC50 Values for BTK Inhibitors
- Compound A: 12 nM (Smith et al., 2023)
- Compound B: 45 nM (Doe et al., 2024)
- [More compounds...]

### Selectivity Data
- Compound A shows >80-fold selectivity vs other kinases
- Tested against panel of 50 kinases (Jones et al., 2023)

### Synthesis Methods
- Lead compounds synthesized via [method]
- Yields: 30-45%
- Full protocols in [papers]

### Gaps Identified
- No data on selectivity vs [specific kinase]
- Limited in vivo data
- Few papers on resistance mechanisms
```

4. **Update search progress stats**
5. **List all files downloaded**

**Option B: Script-based synthesis (large research sessions >50 papers)**

For large research sessions, consider creating a synthesis script:

**create `generate_summary.py`:**
- Read `evaluated-papers.json` from helper scripts
- Aggregate findings by priority and scaffold type
- Generate comprehensive SUMMARY.md with:
  - Executive summary with statistics
  - Papers grouped by relevance score
  - Key chemical scaffolds found
  - Priority recommendations for next steps
  - Methodology documentation
- Include timestamps and reproducibility info

**Benefits:**
- Consistent formatting across sessions
- Easy to regenerate as more papers added
- Can customize grouping/filtering logic
- Documents complete methodology

**Final report:**
```
✅ Research complete!

📊 Summary:
   - Papers reviewed: 127
   - Relevant papers: 18
   - Highly relevant: 7
   - Data extracted: IC50 values for 45 compounds, selectivity data, synthesis methods

📁 All findings in: research-sessions/2025-10-11-btk-inhibitor-selectivity/
   - SUMMARY.md (organized findings)
   - papers/ (14 PDFs + supplementary data)
   - papers-reviewed.json (complete tracking)
```

## Workflow Checklist

**Use TodoWrite to track these steps:**

- [ ] Parse user query (keywords, data types, constraints)
- [ ] Propose and create research folder
- [ ] Initialize tracking files (SUMMARY.md, papers-reviewed.json, citation-graph.json)
- [ ] Search PubMed using searching-literature skill
- [ ] For each paper: evaluate using evaluating-paper-relevance skill
- [ ] For relevant papers (≥7): traverse citations using traversing-citations skill
- [ ] Report progress regularly
- [ ] Checkpoint every 50 papers or 5 minutes
- [ ] When done: synthesize findings in SUMMARY.md
- [ ] Final report with stats and file locations

## Integration Points

**Skills used:**
1. `searching-literature` - Initial PubMed search
2. `evaluating-paper-relevance` - Score and extract from papers
3. `traversing-citations` - Follow citation networks

**All skills coordinate through:**
- Shared `papers-reviewed.json` (deduplication)
- Shared `SUMMARY.md` (findings accumulation)
- Shared `citation-graph.json` (relationship tracking)

## Error Handling

**No results found:**
- Try broader keywords
- Remove constraints
- Check spelling
- Try different synonyms

**API rate limiting:**
- Report to user: "⏸️ Rate limited, waiting..."
- Wait required time
- Resume automatically

**Full text unavailable:**
- Note in SUMMARY.md
- Continue with abstract-only evaluation
- Flag for manual retrieval if highly relevant

**Too many results (>500):**
- Suggest narrowing query
- Process first 100, ask if continue
- Focus on most recent or most cited

## Quick Reference

| Phase | Skill | Output |
|-------|-------|--------|
| Parse | (built-in) | Keywords, data types, constraints |
| Initialize | (built-in) | Folder, SUMMARY.md, tracking files |
| Search | searching-literature | List of papers with metadata |
| Evaluate | evaluating-paper-relevance | Scored papers, extracted findings |
| Traverse | traversing-citations | Additional papers from citations |
| Synthesize | (built-in) | Final SUMMARY.md with key findings |

## Common Mistakes

**Jumping to evaluation without good search:** Too narrow results → Optimize search first
**Not tracking papers:** Re-reviewing same papers → Always use papers-reviewed.json
**Following all citations:** Exponential explosion → Filter before traversing
**No checkpoints:** User loses context → Report and ask every 50 papers
**Poor synthesis:** Just list papers → Group by data type, extract key findings

## User Communication

**Be specific in progress reports:**
- ✅ "Found IC50 = 12 nM for compound 7 (Table 2)"
- ❌ "Found data"

**Ask for clarification when needed:**
- ✅ "Are you looking for in vitro or in vivo IC50 values?"
- ❌ Assume and potentially waste time

**Report blockers:**
- ✅ "⚠️ Paper behind paywall - evaluating from abstract only"
- ❌ Silently skip without mentioning

## Success Criteria

Research session successful when:
- All relevant papers found and evaluated
- Specific data extracted and organized
- Citations followed systematically
- No duplicate processing
- Clear SUMMARY.md with actionable findings
- User questions answered with evidence

## Next Steps

After completing research:
- User reviews SUMMARY.md
- May request deeper dive into specific papers
- May request follow-up searches with refined keywords
- May export findings to other formats
