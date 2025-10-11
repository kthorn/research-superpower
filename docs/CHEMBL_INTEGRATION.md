# ChEMBL Integration for Research Superpowers

## Overview

ChEMBL is a manually curated database of bioactivity data with ~99,000 documents (papers) containing structured SAR data including compound structures, activity values (MIC, IC50, etc.), and assay protocols.

**Key value:** ChEMBL extracts and standardizes data from medicinal chemistry papers, making it queryable. You can find papers that contain specific types of bioactivity data.

## API Access

**Base URL:** `https://www.ebi.ac.uk/chembl/api/data/`

**No authentication required** for basic queries.

**Rate limits:** Reasonable usage without hard limits.

## Querying by DOI

### Find if paper is in ChEMBL

```bash
curl "https://www.ebi.ac.uk/chembl/api/data/document.json?doi=10.1016/j.ejmech.2016.07.009"
```

**Response includes:**
- `document_chembl_id` - ChEMBL's internal ID for the paper
- `doc_type` - "PUBLICATION" (from literature) or "DATASET" (deposited data)
- `pubmed_id` - PMID if available
- `abstract` - Paper abstract
- `title`, `authors`, `journal`, `year`
- Whether data has been extracted

### Get bioactivity data from paper

Once you have `document_chembl_id`, retrieve associated data:

```bash
curl "https://www.ebi.ac.uk/chembl/api/data/activity.json?document_chembl_id=CHEMBL3870308&limit=100"
```

**Each activity record includes:**
- `molecule_chembl_id` - Compound identifier
- `canonical_smiles` - Structure
- `standard_type` - Activity type (MIC, IC50, Ki, etc.)
- `standard_value` - Numeric value
- `standard_units` - Units (ug.mL-1, nM, etc.)
- `standard_relation` - "=", "<", ">"
- `target_chembl_id` - What it was tested against
- `target_organism` - e.g., "Mycobacterium tuberculosis"
- `assay_description` - Protocol details
- `assay_type` - F (functional), B (binding), A (ADMET), P (physicochemical)

### Filter by activity type

```bash
# Get only MIC data
curl "https://www.ebi.ac.uk/chembl/api/data/activity.json?document_chembl_id=CHEMBL3870308&standard_type=MIC"

# Get only IC50 data
curl "https://www.ebi.ac.uk/chembl/api/data/activity.json?document_chembl_id=CHEMBL3870308&standard_type=IC50"
```

## Example Use Cases

### 1. Check if highly relevant paper has structured data

After finding a relevant paper (score ≥8), check ChEMBL:

```bash
doi="10.1016/j.ejmech.2016.07.009"
result=$(curl -s "https://www.ebi.ac.uk/chembl/api/data/document.json?doi=$doi")

# Check if found
if echo "$result" | jq -e '.documents[0]' > /dev/null; then
    chembl_id=$(echo "$result" | jq -r '.documents[0].document_chembl_id')
    echo "✓ Paper in ChEMBL: $chembl_id"

    # Get activity count
    activity_count=$(curl -s "https://www.ebi.ac.uk/chembl/api/data/activity.json?document_chembl_id=$chembl_id&limit=1" | jq -r '.page_meta.total_count')
    echo "  → $activity_count bioactivity data points available"
else
    echo "✗ Paper not in ChEMBL"
fi
```

### 2. Extract structured SAR data

For papers in ChEMBL, you can extract:
- Exact structures (SMILES)
- Exact activity values with units
- Assay protocols
- Statistical data (n, SD, etc.)

This is often more reliable than parsing tables from PDFs.

### 3. Find related compounds

```bash
# Get all compounds from a paper
curl "https://www.ebi.ac.uk/chembl/api/data/molecule.json?document_chembl_id=CHEMBL3870308"
```

## Python Client

For more complex queries, use the official Python client:

```bash
pip install chembl_webresource_client
```

```python
from chembl_webresource_client.new_client import new_client

# Search by DOI
documents = new_client.document
results = documents.filter(doi="10.1016/j.ejmech.2016.07.009")

for doc in results:
    print(f"ChEMBL ID: {doc['document_chembl_id']}")
    print(f"Title: {doc['title']}")

    # Get activities
    activities = new_client.activity
    data = activities.filter(document_chembl_id=doc['document_chembl_id'])

    for act in data:
        print(f"  {act['standard_type']}: {act['standard_value']} {act['standard_units']}")
        print(f"  Structure: {act['canonical_smiles']}")
```

## Integration with Research Skills

### When to check ChEMBL

1. **After abstract screening (score ≥7)**
   - Check if paper is in ChEMBL
   - If yes, note in SUMMARY.md

2. **Before deep dive (score ≥8)**
   - If in ChEMBL, extract structured data directly
   - Faster and more accurate than PDF parsing

3. **In SUMMARY.md, note ChEMBL status:**

```markdown
### [Paper Title](DOI) (Score: 9)

**ChEMBL:** CHEMBL3870308 (45 activity data points)

**Key Findings:**
- MIC data for 23 compounds (from ChEMBL)
- IC50 values: 31-250 ng/mL (Table 2)
- ChEMBL structures: [Download](https://www.ebi.ac.uk/chembl/api/data/molecule.json?document_chembl_id=CHEMBL3870308)
```

### Helper script pattern

For papers with many compounds, create extraction script:

```python
# extract_chembl_data.py
import requests
import json

def get_chembl_data(doi):
    """Extract all SAR data from ChEMBL for a given DOI"""

    # 1. Check if paper in ChEMBL
    doc_url = f"https://www.ebi.ac.uk/chembl/api/data/document.json?doi={doi}"
    doc_response = requests.get(doc_url).json()

    if not doc_response.get('documents'):
        return None

    doc_id = doc_response['documents'][0]['document_chembl_id']

    # 2. Get all activities
    act_url = f"https://www.ebi.ac.uk/chembl/api/data/activity.json?document_chembl_id={doc_id}&limit=1000"
    act_response = requests.get(act_url).json()

    # 3. Structure data
    compounds = {}
    for act in act_response['activities']:
        mol_id = act['molecule_chembl_id']

        if mol_id not in compounds:
            compounds[mol_id] = {
                'smiles': act['canonical_smiles'],
                'activities': []
            }

        compounds[mol_id]['activities'].append({
            'type': act['standard_type'],
            'value': act['standard_value'],
            'units': act['standard_units'],
            'relation': act['standard_relation'],
            'assay': act['assay_description']
        })

    return {
        'document_id': doc_id,
        'compound_count': len(compounds),
        'activity_count': len(act_response['activities']),
        'compounds': compounds
    }

# Usage
doi = "10.1016/j.ejmech.2016.07.009"
data = get_chembl_data(doi)

if data:
    print(f"Found {data['compound_count']} compounds with {data['activity_count']} activities")

    # Save to JSON
    with open(f"chembl_{data['document_id']}.json", 'w') as f:
        json.dump(data, f, indent=2)
```

## Coverage Statistics

**~99,000 documents** in ChEMBL (as of 2025)
- Most are medicinal chemistry papers
- Heavily focused on drug discovery
- Strong coverage of SAR studies
- Includes patents and datasets

**What's typically included:**
- Papers with compound series and activity data
- SAR studies with multiple analogs
- Lead optimization papers
- Target validation studies with activity measurements

**What's typically NOT included:**
- Papers without activity data
- Purely mechanistic studies
- Papers without defined compounds
- Very recent papers (curation lag ~6-12 months)

## Limitations

1. **Curation lag:** New papers take months to be curated
2. **Selective:** Only papers with extractable SAR data
3. **No full text:** Abstract only, not full paper content
4. **Standardization:** Some data may be converted/normalized

## Benefits vs PDF Parsing

**Advantages of ChEMBL:**
- ✓ Structures already extracted and standardized
- ✓ Units normalized (all IC50s in nM, etc.)
- ✓ Machine-readable format
- ✓ Linked to assay protocols
- ✓ No OCR or table parsing errors
- ✓ Can filter by activity type, value ranges, targets

**When to still use PDF:**
- Full experimental details
- Synthesis procedures
- Papers not in ChEMBL
- Very recent papers
- Context and discussion

## Permissions

Add to `.claude/settings.local.json.template`:

```json
{
  "permissions": {
    "allow": [
      "Bash(curl*https://www.ebi.ac.uk/chembl/api/data/*)",
      "WebFetch(domain:www.ebi.ac.uk)"
    ]
  }
}
```

## Quick Reference

| Task | Endpoint |
|------|----------|
| Check if DOI in ChEMBL | `GET /document.json?doi=DOI` |
| Get paper activities | `GET /activity.json?document_chembl_id=ID` |
| Filter by activity type | `GET /activity.json?document_chembl_id=ID&standard_type=MIC` |
| Get molecules | `GET /molecule.json?document_chembl_id=ID` |
| Get assay details | `GET /assay.json?document_chembl_id=ID` |

## Future Integration Ideas

1. **Automatic ChEMBL check** during paper evaluation
2. **Extract SAR data** for papers in ChEMBL
3. **Compound similarity search** to find related papers
4. **Target-based search** to find papers testing against specific proteins
5. **Activity range filtering** to find papers with specific MIC/IC50 ranges

## Resources

- **API Docs:** https://chembl.gitbook.io/chembl-interface-documentation/
- **Live API Explorer:** https://www.ebi.ac.uk/chembl/api/data/docs
- **Python Client:** https://github.com/chembl/chembl_webresource_client
- **ChEMBL Interface:** https://www.ebi.ac.uk/chembl/
- **Database Paper:** https://doi.org/10.1093/nar/gkac1075 (NAR 2023)
