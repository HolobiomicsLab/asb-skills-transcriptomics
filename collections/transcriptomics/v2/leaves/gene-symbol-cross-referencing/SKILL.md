---
name: gene-symbol-cross-referencing
description: Use when you have gene symbol lists from a network/biomarker pipeline
  and need to annotate each gene with drug-actionability status (e.g., DrugBank active
  targets) and literature association to identify novel candidate biomarkers.
when_to_use_negative:
- Input is raw gene expression data rather than an already-extracted gene symbol list
  — pre-processing and network extraction (Pipeline.ipynb) must run first
- The knowledge base of interest is not gene-symbol-addressable (e.g., variant- or
  region-level resources) — symbol matching will miss or mismatch entries
- You need functional enrichment rather than actionability/literature annotation —
  use g:Profiler GO:BP/KEGG analysis instead
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3407
tools:
- name: Python
  role: Implements the symbol-matching script that joins IC gene lists against DrugBank
    active-target entries and PubMed results
  repo: https://github.com/DEIB-GECO/GeneNetFusion
- name: Jupyter Notebook
  role: Hosts the 'Knowledge-based evaluation of the results.ipynb' notebook performing
    the DrugBank and PubMed evaluation steps
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Knowledge-based%20evaluation%20of%20the%20results.ipynb
- name: DrugBank
  role: Source of pharmacologically active target annotations used to assign the 'actionable'
    flag
- name: PubMed
  role: Source of automatic literature-association retrieval used to label genes as
    novel candidate biomarkers when unassociated with the cancer type
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:fe7fb72e30adfd9bda36b29cdda40592f7036754c2e7db69aa7c0caacc4aa5cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DEIB-GECO/GeneNetFusion
  tools:
  - DrugBank
  - Jupyter Notebook
  - PubMed
  - Python
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# gene-symbol-cross-referencing

## Summary

Cross-reference IC gene symbol lists extracted from fused co-expression networks (KIRC, LIHC, PRAD) against external knowledge bases — DrugBank pharmacologically active targets and PubMed literature — to flag drug actionability and novelty of candidate cancer biomarkers.

## When to use

Use when you have gene symbol lists from a network/biomarker pipeline and need to annotate each gene with drug-actionability status (e.g., DrugBank active targets) and literature association to identify novel candidate biomarkers.

## When NOT to use

- Input is raw gene expression data rather than an already-extracted gene symbol list — pre-processing and network extraction (Pipeline.ipynb) must run first
- The knowledge base of interest is not gene-symbol-addressable (e.g., variant- or region-level resources) — symbol matching will miss or mismatch entries
- You need functional enrichment rather than actionability/literature annotation — use g:Profiler GO:BP/KEGG analysis instead

## Inputs

- IC gene symbol CSV lists per cancer type ('IC_'+str(tumor)+'.csv' from the Extracted folder, for KIRC, LIHC, PRAD)
- DB_pharmacologically_active.csv from DrugBank (Supplementary data folder)
- PubMed evaluation results from 'Knowledge-based evaluation of the results.ipynb' (Results folder)

## Outputs

- candidate_biomarkers_actionability.csv with columns: gene symbol, cancer type, 'actionable' flag, PubMed-association status
- Per-cancer counts of actionable genes (e.g., 13 actionable genes for LIHC, 24 for PRAD, 115 actionable hits for PRAD overall)

## How to apply

Load the IC gene symbol lists from the repository's Extracted folder (files named 'IC_'+str(tumor)+'.csv' for KIRC, LIHC, PRAD). Load the Supplementary data file DB_pharmacologically_active.csv downloaded from DrugBank and match gene symbols against its pharmacologically active target entries to assign an 'actionable' flag per gene (e.g., MT-CYB, NDUFV3, PARP3, TOP1MT for KIRC). Retrieve PubMed evaluation results from the 'Knowledge-based evaluation of the results.ipynb' notebook and mark IC genes lacking any literature association with the corresponding cancer type as novel candidate biomarkers. Compile a single per-cancer table with gene symbol, cancer type, 'actionable' flag, and PubMed-association status, saved as candidate_biomarkers_actionability.csv.

## Related tools

- **Python** (Implements the symbol-matching script that joins IC gene lists against DrugBank active-target entries and PubMed results) — https://github.com/DEIB-GECO/GeneNetFusion
- **Jupyter Notebook** (Hosts the 'Knowledge-based evaluation of the results.ipynb' notebook performing the DrugBank and PubMed evaluation steps) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Knowledge-based%20evaluation%20of%20the%20results.ipynb
- **DrugBank** (Source of pharmacologically active target annotations used to assign the 'actionable' flag)
- **PubMed** (Source of automatic literature-association retrieval used to label genes as novel candidate biomarkers when unassociated with the cancer type)

## Examples

```
Execute the jupyter notebook 'Knowledge-based evaluation of the results.ipynb' after loading 'IC_'+str(tumor)+'.csv' and Supplementary data/DB_pharmacologically_active.csv to produce candidate_biomarkers_actionability.csv
```

## Evaluation signals

- Output file candidate_biomarkers_actionability.csv contains an 'actionable' field and the named candidate genes (substring check, e.g., MT-CYB and TOP1MT for KIRC)
- Per-cancer candidate row counts match reported values: 13 actionable genes for LIHC and 24 for PRAD
- KIRC and LIHC actionable IC gene sets show significant enrichment with p-values 8.10 × 10−4 and 2.23 × 10−3 respectively; PRAD is not significantly enriched but yields 115 actionable hits
- PRAD actionable hits include the 4 genes targeted by drugs used for PRAD treatment: ACPP, MAP1A, VKORC1, VKORC1L1
- Genes flagged as novel biomarkers have no PubMed association with their cancer type in the retrieved literature results

## Limitations

- PRAD actionable IC genes are not significantly enriched, so actionability alone is not sufficient evidence of relevance for all cancer types
- No accession IDs or knowledge-base version details (DrugBank, PubMed) are recorded for the datasets used, affecting reproducibility of the match
- Novelty assessment depends on the automatic PubMed retrieval, so genes may be mislabeled novel if literature search terms miss relevant publications

## Evidence

- [readme] DrugBank actionability and PubMed novelty workflow: [evidence span withheld — anchor status: not_found]
- [discussion] Actionable KIRC genes in DrugBank: "for KIRC disease, MT-CYB, NDUFV3, PARP3, and TOP1MT gene products are labeled as actionable in DrugBank"
- [discussion] Novel biomarker definition via PubMed: "IC genes that may be novel cancer biomarkers are those not associated with the cancer type in PubMed"
- [results] PubMed retrieval procedure: "From PubMed (https://www.ncbi.nlm.nih.gov/pubmed/), we automatically retrieve all scientific publications"
- [other] Enrichment p-values for actionable genes: [evidence span withheld — anchor status: not_found]
