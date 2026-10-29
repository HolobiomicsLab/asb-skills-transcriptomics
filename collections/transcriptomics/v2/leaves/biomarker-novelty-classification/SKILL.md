---
name: biomarker-novelty-classification
description: Use when IC gene symbol lists from the fused co-expression pipeline must
  be triaged into drug-actionable versus literature-novel candidate biomarkers per
  cancer type (KIRC, LIHC, PRAD).
when_to_use_negative:
- The fused co-expression networks have not yet been built and IC gene communities
  not yet extracted with Gephi (run Pipeline.ipynb first — this skill consumes IC_*.csv
  lists, not raw TCGA expression or network files)
- The goal is functional/pathway characterization (g:Profiler GO:BP and KEGG enrichment)
  rather than drug actionability and literature-novelty classification
- Genes of interest are differentially expressed (DESeq2) baseline biomarkers rather
  than IC genes from the fused networks
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0208
tools:
- name: Jupyter Notebook
  role: Hosts 'Knowledge-based evaluation of the results.ipynb', which performs the
    systematic and statistic evaluation of IC genes, the DrugBank actionability matching,
    and the PubMed novelty assessment
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Knowledge-based%20evaluation%20of%20the%20results.ipynb
- name: Python
  role: Implements the gene-symbol matching against DB_pharmacologically_active.csv
    and compilation of candidate_biomarkers_actionability.csv
  repo: https://github.com/DEIB-GECO/GeneNetFusion
- name: DrugBank
  role: Source of pharmacologically active target annotations used to flag IC genes
    as drug actionable
- name: PubMed
  role: Automatically retrieved scientific publications used to determine whether
    an IC gene is already literature-associated with its cancer type
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:4e957d35b730ee61dd4ea5a2e3682a0dd4d41167584101bd3357688fce4600c1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DrugBank
  - Jupyter Notebook
  - PubMed
  - Python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0208
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# biomarker-novelty-classification

## Summary

Cross-references IC genes extracted from fused KIRC/LIHC/PRAD co-expression networks against DrugBank pharmacologically-active-target annotations and PubMed literature to label each gene as drug actionable and/or a novel candidate biomarker for its cancer type.

## When to use

Use when IC gene symbol lists from the fused co-expression pipeline must be triaged into drug-actionable versus literature-novel candidate biomarkers per cancer type (KIRC, LIHC, PRAD).

## When NOT to use

- The fused co-expression networks have not yet been built and IC gene communities not yet extracted with Gephi (run Pipeline.ipynb first — this skill consumes IC_*.csv lists, not raw TCGA expression or network files)
- The goal is functional/pathway characterization (g:Profiler GO:BP and KEGG enrichment) rather than drug actionability and literature-novelty classification
- Genes of interest are differentially expressed (DESeq2) baseline biomarkers rather than IC genes from the fused networks

## Inputs

- IC gene symbol lists per cancer type: 'IC_KIRC.csv', 'IC_LIHC.csv', 'IC_PRAD.csv' from the Extracted folder
- DB_pharmacologically_active.csv from the Supplementary data folder (DrugBank pharmacologically active target entries)
- PubMed evaluation results for the IC genes from the 'Knowledge-based evaluation of the results.ipynb' notebook (Results folder)

## Outputs

- candidate_biomarkers_actionability.csv with per-gene columns: gene symbol, cancer type, 'actionable' flag, PubMed-association status
- Novel candidate biomarker list per cancer type (genes not associated with the cancer type in PubMed)
- Per-cancer actionable-gene counts and enrichment p-values (e.g. 13 actionable for LIHC, 24 for PRAD, 115 PRAD actionable hits)

## How to apply

Load the IC gene symbol lists ('IC_'+str(tumor)+'.csv') for KIRC, LIHC and PRAD from the repository's Extracted folder, plus the PubMed evaluation results from 'Knowledge-based evaluation of the results.ipynb'. Match each gene symbol against the Supplementary data file DB_pharmacologically_active.csv (downloaded from DrugBank, https://www.drugbank.ca/releases/latest) to assign an 'actionable' flag — e.g. MT-CYB, NDUFV3, PARP3 and TOP1MT for KIRC. Mark IC genes with no PubMed literature association to the corresponding cancer type as novel candidate biomarkers. Compile a single per-cancer table (gene symbol, cancer type, 'actionable' flag, PubMed-association status) saved as candidate_biomarkers_actionability.csv, and optionally test enrichment of actionable IC genes per cancer (reported significant for KIRC p=8.10e-4 and LIHC p=2.23e-3, not significant for PRAD despite 115 actionable hits).

## Related tools

- **Jupyter Notebook** (Hosts 'Knowledge-based evaluation of the results.ipynb', which performs the systematic and statistic evaluation of IC genes, the DrugBank actionability matching, and the PubMed novelty assessment) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Knowledge-based%20evaluation%20of%20the%20results.ipynb
- **Python** (Implements the gene-symbol matching against DB_pharmacologically_active.csv and compilation of candidate_biomarkers_actionability.csv) — https://github.com/DEIB-GECO/GeneNetFusion
- **DrugBank** (Source of pharmacologically active target annotations used to flag IC genes as drug actionable)
- **PubMed** (Automatically retrieved scientific publications used to determine whether an IC gene is already literature-associated with its cancer type)

## Examples

```
pip install -r requirements.txt; jupyter notebook "Knowledge-based evaluation of the results.ipynb"
```

## Evaluation signals

- Output candidate_biomarkers_actionability.csv contains a named 'actionable' field per gene and the required per-cancer candidate rows
- Substring check passes for the reported KIRC actionable genes, e.g. MT-CYB and TOP1MT appear in the KIRC candidates
- Per-cancer actionable candidate row counts match the reported values: 13 actionable genes for LIHC and 24 for PRAD
- PRAD actionable hits total 115, of which exactly 4 (ACPP, MAP1A, VKORC1, VKORC1L1) are targets of drugs used for PRAD treatment
- Enrichment of actionable IC genes reproduces reported significance: p=8.10e-4 for KIRC and p=2.23e-3 for LIHC, with PRAD not significant

## Limitations

- DrugBank actionability and PubMed novelty depend on the knowledge-base version and retrieval date at execution time; the article provides no accession IDs or versions for DrugBank, BioGRID or PubMed
- Enrichment of actionable IC genes is not significant for all cancer types — PRAD is not significantly enriched, so actionability alone does not confirm relevance there
- The public repository ships complete data/scripts only for the LIHC tumor analysis; KIRC and PRAD IC gene lists come from the Results folder without the full per-tumor pipeline data

## Evidence

- [discussion] KIRC actionable genes in DrugBank: "for KIRC disease, MT-CYB, NDUFV3, PARP3, and TOP1MT gene products are labeled as actionable in DrugBank"
- [discussion] Novelty criterion: "IC genes that may be novel cancer biomarkers are those not associated with the cancer type in PubMed"
- [readme] DrugBank evaluation input: "downloaded from DrugBank to be used in the DrugBank evaluation part of [Knowledge-based evaluation of the results.ipynb]"
- [readme] Notebook purpose: "contains systematic and statistic evaluation of the extracted genes (here called IC genes)"
- [intro] Relevance assessment dimensions: "we assess their relevance from the perspective of the gene set enrichment analysis, the literature and their drug actionability"
