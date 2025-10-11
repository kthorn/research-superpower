---
name: Building Paper Screening Rubrics
description: Collaboratively build and refine paper screening rubrics through brainstorming, test-driven development, and iterative feedback
when_to_use: Starting new literature search. When automated screening misclassifies papers. When need to screen 50+ papers efficiently. Before creating screening scripts. When rescreening papers with updated criteria.
version: 1.0.0
---

# Building Paper Screening Rubrics

## Overview

**Core principle:** Build screening rubrics collaboratively through brainstorming → test → refine → automate → review → iterate.

Good rubrics come from understanding edge cases upfront and testing on real papers before bulk screening.

## When to Use

Use this skill when:
- Starting a new literature search that will screen 50+ papers
- Current rubric misclassifies papers (false positives/negatives)
- Need to define "relevance" criteria before automated screening
- Want to update criteria and re-screen cached papers
- Building helper scripts for evaluating-paper-relevance

**When NOT to use:**
- Small searches (<20 papers) - manual screening is fine
- Rubric already works well - no need to rebuild
- One-off exploratory searches

## Two-Phase Process

### Phase 1: Collaborative Rubric Design

#### Step 1: Brainstorm Relevance Criteria

**Ask domain-agnostic questions to understand what makes papers relevant:**

**Key Terms (+3 pts each):**
- "What are the MOST IMPORTANT terms for your research?"
  - Specific entities: genes, proteins, compounds, scaffolds, organisms, diseases
  - Critical concepts: specific methods, theories, mechanisms
  - Essential data: measurement types (IC50, MIC, expression level, etc.)
  - Example: "If a paper mentions X, it's almost certainly relevant"
- "Are there synonyms or alternative names for key terms?"

**Relevant Terms (+1 pt each):**
- "What related terms make papers MORE relevant but aren't required?"
  - Related concepts: homologs, analogs, related diseases
  - Context terms: derivative, series, analog, homolog, variant
  - Supporting data: in vitro, activity, assay, synthesis
  - Methods: protocol, measurement, analysis
  - Example: "These terms are good signals but not sufficient alone"

**Exclusion Terms (score = 0):**
- "Any terms that indicate a paper is NOT relevant?"
  - Wrong organisms, diseases, or contexts
  - Example: "If studying bacterial compounds, exclude 'cancer' or 'mammalian'"

**Edge Cases:**
- "Can you think of papers that would LOOK relevant but aren't?"
- "Papers that might NOT look relevant but actually are?"
- "If a paper mentions [key term 1] + [key term 2], is it almost always relevant?"

**Document responses in screening-criteria.json**

#### Step 2: Build Initial Rubric

**Based on brainstorming, propose scoring logic:**

```
Scoring (additive, no upper limit):

Key Terms: +3 pts each
  - Target compounds/scaffolds (e.g., "BM212", "bedaquiline")
  - Core concepts (e.g., "tuberculosis drug resistance")
  - Critical data types (e.g., "MIC", "IC50")

Relevant Terms: +1 pt each
  - Related concepts (e.g., "mycobacteria", "intracellular")
  - Related data types (e.g., "in vitro", "activity")
  - Context terms (e.g., "analog", "derivative", "series")

Exclusion Rules:
  - If mentions exclusion term → score = 0

Threshold: ≥7 = relevant, 5-6 = possibly relevant, <5 = not relevant

Example: Abstract mentions "BM212" (+3) and "SAR" (+3) = 6 pts
  → Already close to threshold with just two key terms!
  → Add any relevant term (e.g., "analog" +1) → 7 pts = relevant ✓
```

**Present to user and ask:** "Does this logic match your expectations?"

**Save initial rubric to screening-criteria.json:**
```json
{
  "version": "1.0.0",
  "created": "2025-10-11T15:30:00Z",
  "key_terms": {
    "terms": ["BM212", "SQ109", "MIC", "tuberculosis drug resistance"],
    "points_each": 3,
    "synonyms": {
      "BM212": ["BM-212"],
      "tuberculosis": ["TB", "Mycobacterium tuberculosis", "M. tuberculosis"]
    }
  },
  "relevant_terms": {
    "terms": ["analog", "derivative", "series", "SAR", "structure-activity", "in vitro", "activity", "mycobacteria"],
    "points_each": 1
  },
  "exclusion_terms": ["cancer", "mammalian cell", "eukaryotic"],
  "scoring": {
    "relevance_threshold": 7,
    "possibly_relevant_threshold": 5
  }
}
```

### Phase 2: Test-Driven Refinement

#### Step 1: Create Test Set

**Do a quick PubMed search to get candidate papers:**
```bash
# Search for 20 papers using initial keywords
curl "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?db=pubmed&term=YOUR_QUERY&retmax=20&retmode=json"
```

**Fetch abstracts for first 10-15 papers:**
```bash
curl "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=pubmed&id=PMID1,PMID2,...&retmode=xml&rettype=abstract"
```

**Present abstracts to user one at a time:**
```
Paper 1/10:
Title: [Title]
PMID: [12345678]
DOI: [10.1234/example]

Abstract:
[Full abstract text]

Is this paper RELEVANT to your research question? (y/n/maybe)
```

**Record user judgments in test-set.json:**
```json
{
  "test_papers": [
    {
      "pmid": "12345678",
      "doi": "10.1234/example",
      "title": "Paper title",
      "abstract": "Full abstract text...",
      "user_judgment": "relevant",
      "timestamp": "2025-10-11T15:45:00Z"
    }
  ]
}
```

**Continue until have 5-10 papers with clear judgments**

#### Step 2: Score Test Papers with Rubric

**Apply rubric to each test paper:**
```python
for paper in test_papers:
    score = calculate_score(paper['abstract'], rubric)
    predicted_status = "relevant" if score >= 7 else "not_relevant"
    paper['predicted_score'] = score
    paper['predicted_status'] = predicted_status
```

**Calculate accuracy:**
```python
correct = sum(1 for p in test_papers
              if p['predicted_status'] == p['user_judgment'])
accuracy = correct / len(test_papers)
```

#### Step 3: Show Results to User

**Present classification report:**
```
RUBRIC TEST RESULTS (5 papers):

✓ PMID 12345678: Score 9 → relevant (user: relevant) ✓
✗ PMID 23456789: Score 4 → not_relevant (user: relevant) ← FALSE NEGATIVE
✓ PMID 34567890: Score 8 → relevant (user: relevant) ✓
✓ PMID 45678901: Score 3 → not_relevant (user: not_relevant) ✓
✗ PMID 56789012: Score 8 → relevant (user: not_relevant) ← FALSE POSITIVE

Accuracy: 60% (3/5 correct)
Target: ≥80%

--- FALSE NEGATIVE: PMID 23456789 ---
Title: "Novel analogs of BM212 with improved potency"
Score breakdown:
  - Key terms: 3 pts (BM212)
  - Relevant terms: 0 pts
  - Total: 4 pts → not_relevant

Why missed: Mentions "analogs" but it's not in relevant_terms list
Abstract excerpt: "We synthesized 12 analogs of BM212..."

Should "analog" be added as a relevant term (+1 pt)?

--- FALSE POSITIVE: PMID 56789012 ---
Title: "BM212 and SQ109 in cancer cell lines"
Score breakdown:
  - Key terms: 6 pts (BM212 +3, SQ109 +3)
  - Relevant terms: 2 pts (activity +1, in vitro +1)
  - Total: 8 pts → relevant

Why wrong: Paper about cancer, not tuberculosis
Abstract excerpt: "Tested BM212 and SQ109 against cancer cell lines..."

Should "cancer" be added as an exclusion term (→ score 0)?
```

#### Step 4: Iterative Refinement

**Ask user for adjustments:**
```
Current accuracy: 60% (below 80% threshold)

Suggestions to improve rubric:
1. Add "analog" to relevant_terms list? (would fix false negative)
2. Add "cancer" to exclusion_terms? (would fix false positive)
3. Add more synonyms for key terms?
4. Adjust threshold (currently 7)?

What would you like to adjust?
```

**Update screening-criteria.json based on feedback**

**Example update:**
```json
{
  "relevant_terms": {
    "terms": ["analog", "derivative", "series", "SAR", ...],  // Added "analog"
    "points_each": 1
  },
  "exclusion_terms": ["cancer", "mammalian cell"],  // Added "cancer"
  "version": "1.1.0"
}
```

#### Step 5: Re-test Until Satisfied

**Re-score test papers with updated rubric**

**Show new results:**
```
UPDATED RUBRIC TEST RESULTS (5 papers):

✓ PMID 12345678: Score 9 → relevant (user: relevant) ✓
✓ PMID 23456789: Score 5 → relevant (user: relevant) ✓ (FIXED!)
  - Now: BM212 (+3) + analog (+1) = 4 pts → wait, still not 7...

Let me recalculate:
✓ PMID 23456789: Score 7 → relevant (user: relevant) ✓ (FIXED!)
  - BM212 (+3) + MIC (+3) + analog (+1) = 7 pts ✓
✓ PMID 34567890: Score 8 → relevant (user: relevant) ✓
✓ PMID 45678901: Score 3 → not_relevant (user: not_relevant) ✓
✓ PMID 56789012: Score 0 → not_relevant (user: not_relevant) ✓ (FIXED!)
  - Contains exclusion term "cancer" → score = 0

Accuracy: 100% (5/5 correct) ✓
Target: ≥80% ✓

Rubric is ready for bulk screening!
```

**If accuracy ≥80%:** Proceed to bulk screening
**If <80%:** Continue iterating

### Phase 3: Bulk Screening

**Once rubric validated on test set:**

1. **Run on full PubMed search results**
2. **Save all abstracts to abstracts-cache.json:**
```json
{
  "10.1234/example": {
    "pmid": "12345678",
    "title": "Paper title",
    "abstract": "Full abstract text...",
    "fetched": "2025-10-11T16:00:00Z"
  }
}
```

3. **Score all papers, save to papers-reviewed.json:**
```json
{
  "10.1234/example": {
    "pmid": "12345678",
    "status": "relevant",
    "score": 9,
    "source": "pubmed_search",
    "timestamp": "2025-10-11T16:00:00Z",
    "rubric_version": "1.0.0"
  }
}
```

4. **Generate summary report:**
```
Screened 127 papers using validated rubric:
- Highly relevant (≥8): 12 papers
- Relevant (7): 18 papers
- Possibly relevant (5-6): 23 papers
- Not relevant (<5): 74 papers

All abstracts cached for re-screening.
Results saved to papers-reviewed.json.

Review offline and provide feedback if any misclassifications found.
```

### Phase 4: Offline Review & Re-screening

**User reviews papers offline, identifies issues:**

```
User: "I reviewed the results. Three papers were misclassified:
- PMID 23456789 scored 4 but is actually relevant (discusses scaffold analogs)
- PMID 34567890 scored 8 but not relevant (wrong target)
- PMID 45678901 scored 6 but is highly relevant (has key dataset)

Can we update the rubric?"
```

**Update rubric based on feedback:**
1. Analyze why misclassifications occurred
2. Propose rubric adjustments
3. Re-score ALL cached papers with new rubric
4. Show diff of what changed

**Re-screening workflow:**
```bash
# Load all abstracts from abstracts-cache.json
# Apply updated rubric to each
# Generate change report

RUBRIC UPDATE: v1.0.0 → v1.1.0

Changes:
- Added "derivative" to scaffold_analogs rule
- Increased dataset bonus from +1 to +2 pts

Re-screening 127 cached papers...

Status changes:
  not_relevant → relevant: 3 papers
    - PMID 23456789 (score 4→7)
    - PMID 45678901 (score 6→8)
  relevant → not_relevant: 1 paper
    - PMID 34567890 (score 8→6)

Updated papers-reviewed.json with new scores.
New summary:
- Highly relevant: 13 papers (+1)
- Relevant: 19 papers (+1)
```

## File Structure

```
research-sessions/YYYY-MM-DD-topic/
├── screening-criteria.json      # Rubric definition (weights, rules, version)
├── test-set.json               # Ground truth papers used for validation
├── abstracts-cache.json        # Full abstracts for all screened papers
├── papers-reviewed.json        # Simple tracking: DOI, score, status
└── rubric-changelog.md         # History of rubric changes and why
```

## Integration with Other Skills

**Before evaluating-paper-relevance:**
- Use this skill to build and validate rubric first
- Creates screening-criteria.json and abstracts-cache.json
- Then use evaluating-paper-relevance with validated rubric

**When creating helper scripts:**
- Use screening-criteria.json to parameterize scoring logic
- Reference abstracts-cache.json to avoid re-fetching
- Easy to update rubric without rewriting script

**During answering-research-questions:**
- Build rubric in initialization phase (after Phase 1: Parse Query)
- Validate on test set before bulk screening
- Save rubric with research session for reproducibility

## Rubric Design Patterns

### Pattern 1: Additive Scoring (Default)

```python
score = 0

# Check exclusion terms first
for term in exclusion_terms:
    if term.lower() in abstract.lower():
        return 0  # Automatic rejection

# Count key term matches
for term in key_terms:
    if term.lower() in abstract.lower():
        score += 3

# Count relevant term matches
for term in relevant_terms:
    if term.lower() in abstract.lower():
        score += 1

return score
```

**Example:**
```
Abstract: "We synthesized 12 analogs of BM212 and measured MIC values..."

Matches:
- "BM212" (key term): +3 pts
- "MIC" (key term): +3 pts
- "analogs" (relevant term): +1 pt
- "synthesized" (relevant term): +1 pt

Total: 8 pts → relevant ✓
```

### Pattern 2: Domain-Specific Examples

**Medicinal chemistry:**
```json
{
  "key_terms": {
    "terms": ["BM212", "SQ109", "bedaquiline", "MIC", "IC50", "drug resistance"],
    "points_each": 3
  },
  "relevant_terms": {
    "terms": ["analog", "derivative", "series", "SAR", "structure-activity",
              "in vitro", "activity", "mycobacteria", "synthesis"],
    "points_each": 1
  },
  "exclusion_terms": ["cancer", "mammalian", "eukaryotic"]
}
```

**Genomics:**
```json
{
  "key_terms": {
    "terms": ["BRCA1", "breast cancer", "RNA-seq", "differential expression", "GEO:", "SRA:"],
    "points_each": 3
  },
  "relevant_terms": {
    "terms": ["gene expression", "transcriptome", "DEG", "fold change", "FDR",
              "pathway", "enrichment", "accession"],
    "points_each": 1
  },
  "exclusion_terms": ["mouse", "zebrafish", "drosophila"]
}
```

**Computational methods:**
```json
{
  "key_terms": {
    "terms": ["machine learning", "alignment algorithm", "phylogenetic", "github", "benchmark"],
    "points_each": 3
  },
  "relevant_terms": {
    "terms": ["accuracy", "performance", "comparison", "dataset", "implementation",
              "code available", "software", "tool"],
    "points_each": 1
  },
  "exclusion_terms": ["theoretical only", "no implementation"]
}
```

## Common Mistakes

**Skipping test-driven validation:** Bulk screen without testing rubric → Many misclassifications, wasted time
**Not caching abstracts:** Re-fetch from PubMed when rescreening → Slow, hits rate limits
**No ground truth testing:** Can't measure rubric accuracy → Don't know if it's working
**Too few test papers:** Test on 2-3 papers → Rubric overfits, doesn't generalize
**Too complex rubric:** Boolean logic with 10+ rules → Hard to debug, update, explain
**Not documenting changes:** Update rubric without tracking why → Can't reproduce, learn from mistakes
**Setting threshold too high:** Require 95% accuracy → Iterate forever, diminishing returns
**Not saving rubric with session:** Rubric lost when session ends → Can't reproduce screening

## Success Criteria

Rubric is ready when:
- ≥80% accuracy on test set (5-10 ground truth papers)
- User confident rubric captures edge cases
- Special rules documented for domain-specific patterns
- All files created (screening-criteria.json, test-set.json, abstracts-cache.json)
- Ready to run bulk screening

## Quick Reference

| Task | File | Contents |
|------|------|----------|
| Define rubric | screening-criteria.json | Keywords, weights, special rules |
| Test rubric | test-set.json | Ground truth papers with user judgments |
| Cache abstracts | abstracts-cache.json | Full abstract text for re-screening |
| Track results | papers-reviewed.json | DOI, score, status, rubric version |
| Document changes | rubric-changelog.md | History of updates and rationale |

## Example Session

```
Agent: "I'll help you build a screening rubric. Let's start with some questions about what makes papers relevant..."

[Brainstorming phase - ask 5 question categories]

User: "I'm looking for papers on tuberculosis drug resistance with MIC data for bedaquiline and clofazimine. Papers about analogs are also relevant."

Agent: "Got it. Here's the initial rubric:

Key Terms (+3 pts each):
- bedaquiline, clofazimine, tuberculosis, drug resistance, MIC

Relevant Terms (+1 pt each):
- analog, derivative, series, in vitro, activity, mycobacteria

Threshold: ≥7 = relevant

Example: Paper with 'bedaquiline' (+3) + 'MIC' (+3) + 'analog' (+1) = 7 pts → relevant

Does this look right?"

User: "Yes, let's test it."

Agent: "I'll search PubMed for 20 papers and show you abstracts to classify..."

[Shows 10 abstracts, user classifies them]

Agent: "Testing rubric on your classifications...
Accuracy: 70% (7/10 correct)
- 2 false negatives (missed relevant papers)
- 1 false positive (cancer paper marked relevant)

False negative example:
- Paper discusses 'derivatives of bedaquiline' with activity data
- Score: bedaquiline (+3) + activity (+1) = 4 pts (below threshold)
- Missing: 'derivative' not in relevant_terms

False positive example:
- Paper about bedaquiline in cancer cells
- Score: bedaquiline (+3) + MIC (+3) + in vitro (+1) = 7 pts
- Problem: Wrong disease context

Suggestions:
1. Add 'derivative' to relevant_terms?
2. Add 'cancer' to exclusion_terms?

User: "Yes to both."

Agent: "Updated rubric. Re-testing...
Accuracy: 90% (9/10 correct) ✓

Ready to run bulk screening on 127 papers?"

User: "Yes, go ahead."

Agent: [Screens 127 papers, saves results]

"Done! Found 18 relevant papers. All abstracts cached for re-screening.
Review offline and let me know if any misclassifications."

[Later...]

User: "Found 2 misclassifications. Can we update the rubric?"

Agent: "Sure! What did I miss?"

User: "PMID 12345 discusses pretomanid combination therapy with bedaquiline - should be relevant."

Agent: "Got it. Adding 'combination therapy' as related term with +2 bonus pts.
Re-screening all 127 cached papers...
Status changes: 3 papers now relevant (including PMID 12345).
Updated papers-reviewed.json."
```

## Next Steps

After building rubric:
- Use for bulk screening in evaluating-paper-relevance
- Parameterize helper scripts with screening-criteria.json
- Update rubric as you discover edge cases
- Re-screen cached papers when criteria change
- Document rubric in research session README for reproducibility
