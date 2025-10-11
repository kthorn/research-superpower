# Testing Research Superpowers

## Manual Test Plan

To validate the complete workflow, test with a real research question:

### Example Test Case: BTK Inhibitor Selectivity

**Research Question:** "Find selectivity data for BTK inhibitors against other kinases"

**Expected Workflow:**
1. Claude reads `answering-research-questions` skill
2. Parses query → Keywords: "BTK inhibitor", "selectivity", "kinase"
3. Proposes folder: `research-sessions/2025-MM-DD-btk-inhibitor-selectivity/`
4. Searches PubMed for relevant papers
5. Screens abstracts (scores 0-10)
6. Deep dives into promising papers (score ≥7)
7. Extracts IC50 values, selectivity data
8. Downloads PDFs and supplementary data
9. Follows citations (backward + forward)
10. Checks in every 50 papers or 5 minutes
11. Synthesizes findings in SUMMARY.md

**Success Criteria:**
- [ ] Folder created with correct structure
- [ ] papers-reviewed.json tracks all papers by DOI
- [ ] SUMMARY.md contains organized findings
- [ ] PDFs downloaded to papers/ folder
- [ ] Citations tracked in citation-graph.json
- [ ] No duplicate paper processing
- [ ] Progress reported clearly
- [ ] Final summary with key findings

### Other Test Cases

**Simple query:** "What is the mechanism of ibrutinib?"
- Should find mechanism papers quickly
- Extract mechanism description
- Not traverse too deeply

**Data-heavy query:** "IC50 values for covalent BTK inhibitors"
- Should focus on tables and figures
- Extract specific numerical data
- Download supplementary data

**Method query:** "How to synthesize acalabrutinib?"
- Should find synthesis papers
- Extract methods section
- Find reaction schemes

## API Testing

### PubMed E-utilities
- [ ] Search returns results
- [ ] Metadata extraction works
- [ ] DOI resolution works
- [ ] Rate limiting handled (3 req/sec without key)

### Semantic Scholar
- [ ] DOI lookup works
- [ ] References fetched correctly
- [ ] Citations fetched correctly
- [ ] Context strings useful for relevance
- [ ] Rate limiting handled (100 req/5min free tier)

## Edge Cases

- [ ] No results found → suggests broader search
- [ ] Too many results (>500) → suggests narrowing
- [ ] Full text behind paywall → notes in summary, continues with abstract
- [ ] No DOI available → uses PMID as fallback
- [ ] API rate limited → waits and resumes
- [ ] Duplicate papers → skipped via papers-reviewed.json

## Future Improvements

After initial testing, consider:
- Helper scripts for common API calls
- Caching layer to avoid re-fetching
- Better error handling and retry logic
- More sophisticated relevance scoring
- Natural language query parsing
- Export to BibTeX, EndNote, etc.
- Integration with reference managers

## TDD Phase (Future)

Once workflows are validated:
1. Run baseline tests (subagent without skills)
2. Document failure modes
3. Refine skills to address failures
4. Iterate until bulletproof

See `skills/meta/writing-skills` and `skills/meta/testing-skills-with-subagents` from the superpowers project for the TDD process.
