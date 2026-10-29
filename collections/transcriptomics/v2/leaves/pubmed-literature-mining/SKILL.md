---
name: pubmed-literature-mining
description: Use when you have a list of candidate genes (e.g., IC genes from fused
  co-expression networks) and need to determine which are literature-associated with
  a specific cancer type versus which are novel candidate biomarkers.
when_to_use_negative:
- The gene set is not derived from the fused co-expression network pipeline (no IC
  gene lists in the expected 'IC_'+tumor+'.csv' format), so counts cannot be validated
  against reported values.
- The task requires functional enrichment or classification validation of biomarkers
  rather than literature/actionability annotation.
- Candidate genes already have curated literature-association annotations, making
  automated PubMed association mining redundant.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0219
- http://edamontology.org/topic_0203
tools:
- name: Python
  role: Scripting language for loading IC gene CSVs, matching symbols against DrugBank
    target entries, and compiling the candidate biomarker table
  repo: https://github.com/DEIB-GECO/GeneNetFusion
- name: Jupyter Notebook
  role: Hosts the 'Knowledge-based evaluation of the results.ipynb' notebook that
    performs the systematic and statistical PubMed evaluation of IC genes
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Knowledge-based%20evaluation%20of%20the%20results.ipynb
- name: PubMed
  role: Source of scientific publications automatically retrieved to test literature
    association of each IC gene with its cancer type
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:61eb6e0d89a325d7781de6294f838ec2caa514d1a01264e2399f546624035d71
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DEIB-GECO/GeneNetFusion
  tools:
  - Jupyter Notebook
  - PubMed
  - Python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0219
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# pubmed-literature-mining

## Summary

Systematic literature evaluation of pipeline-extracted IC genes via automated PubMed queries, distinguishing literature-associated genes from novel candidate biomarkers and pairing the results with DrugBank actionability flags per cancer type (KIRC, LIHC, PRAD).

## When to use

Use when you have a list of candidate genes (e.g., IC genes from fused co-expression networks) and need to determine which are literature-associated with a specific cancer type versus which are novel candidate biomarkers.

## When NOT to use

- The gene set is not derived from the fused co-expression network pipeline (no IC gene lists in the expected 'IC_'+tumor+'.csv' format), so counts cannot be validated against reported values.
- The task requires functional enrichment or classification validation of biomarkers rather than literature/actionability annotation.
- Candidate genes already have curated literature-association annotations, making automated PubMed association mining redundant.

## Inputs

- IC gene symbol lists per cancer type (Extracted/IC_KIRC.csv, IC_LIHC.csv, IC_PRAD.csv)
- DrugBank pharmacologically active target annotations (Supplementary data/DB_pharmacologically_active.csv)
- PubMed publication database (https://www.ncbi.nlm.nih.gov/pubmed/)

## Outputs

- candidate_biomarkers_actionability.csv with columns: gene symbol, cancer type, actionable flag, PubMed-association status
- Per-cancer counts of actionable candidate genes (13 for LIHC, 24 for PRAD)
- Enrichment p-values of actionable IC genes per cancer type (KIRC 8.10 × 10−4, LIHC 2.23 × 10−3)

## How to apply

Load the IC gene symbol lists for each tumor type from the repository's Extracted folder using the naming convention 'IC_'+str(tumor)+'.csv'. In the 'Knowledge-based evaluation of the results.ipynb' notebook, automatically retrieve all PubMed scientific publications mentioning each gene symbol together with its cancer type (KIRC, LIHC, or PRAD); IC genes with no PubMed association to the cancer type are flagged as novel candidate biomarkers. In parallel, cross-reference gene symbols against the Supplementary data file DB_pharmacologically_active.csv (DrugBank pharmacologically active targets) to assign an 'actionable' flag per gene (e.g., MT-CYB, NDUFV3, PARP3 and TOP1MT for KIRC). Compile a single per-cancer candidate biomarker table with gene symbol, cancer type, 'actionable' flag and PubMed-association status, saved as candidate_biomarkers_actionability.csv. The rationale is that literature support plus drug actionability computationally validates the relevance of extracted gene sets, complementing functional enrichment analysis.

## Related tools

- **Python** (Scripting language for loading IC gene CSVs, matching symbols against DrugBank target entries, and compiling the candidate biomarker table) — https://github.com/DEIB-GECO/GeneNetFusion
- **Jupyter Notebook** (Hosts the 'Knowledge-based evaluation of the results.ipynb' notebook that performs the systematic and statistical PubMed evaluation of IC genes) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Knowledge-based%20evaluation%20of%20the%20results.ipynb
- **PubMed** (Source of scientific publications automatically retrieved to test literature association of each IC gene with its cancer type)

## Examples

```
pip install -r requirements.txt; jupyter notebook "Knowledge-based evaluation of the results.ipynb"
```

## Evaluation signals

- Output file candidate_biomarkers_actionability.csv exists and contains an 'actionable' field for each gene row.
- Substring check confirms named candidate genes are present, e.g., MT-CYB and TOP1MT for KIRC.
- Per-cancer actionable candidate row counts match reported values: 13 for LIHC and 24 for PRAD.
- PRAD table contains 115 actionable hits, with ACPP, MAP1A, VKORC1, and VKORC1L1 flagged as targets of drugs used for PRAD treatment.
- Recomputed enrichment p-values for actionable IC genes reproduce KIRC 8.10 × 10−4 and LIHC 2.23 × 10−3 (PRAD not significantly enriched).

## Limitations

- No accession IDs or knowledge-base version details (DrugBank, PubMed snapshots) are recorded, so results may not be exactly reproducible against current releases.
- PRAD actionable IC genes are not significantly enriched (p-value not significant), so actionability alone is a weaker validation signal for that cancer type.
- Novel-biomarker status depends entirely on absence of PubMed co-occurrence; genes missing from PubMed indexing or described under different nomenclature may be misclassified as novel.

## Evidence

- [results] From PubMed (https://www.ncbi.nlm.nih.gov/pubmed/), we automatically retrieve all scientific publications: "From PubMed (https://www.ncbi.nlm.nih.gov/pubmed/), we automatically retrieve all scientific publications"
- [discussion] IC genes that may be novel cancer biomarkers are those not associated with the cancer type in PubMed: "IC genes that may be novel cancer biomarkers are those not associated with the cancer type in PubMed"
- [intro] we assess their relevance from the perspective of the gene set enrichment analysis, the literature and their drug actionability: "we assess their relevance from the perspective of the gene set enrichment analysis, the literature and their drug actionability"
- [readme] contains systematic and statistic evaluation of the extracted genes (here called IC genes): "contains systematic and statistic evaluation of the extracted genes (here called IC genes)"
- [readme] the IC gene symbols for LIHC, KIRC and PRAD, and the PubMed evaluation results for LIHC: "containing the gene symbols of the LIHC fused network, the IC gene symbols for LIHC, KIRC and PRAD, and the PubMed evaluation results for LIHC"
