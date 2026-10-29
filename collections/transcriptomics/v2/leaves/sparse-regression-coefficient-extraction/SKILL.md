---
name: sparse-regression-coefficient-extraction
description: Use when you already have extracted gene communities (e.g. 'IC_'+tumor+'.csv'
  files for KIRC, LIHC, PRAD) and need a sparse, smaller gene signature per community
  for a classification task.
when_to_use_negative:
- The goal is the differential expression baseline (DESeq2 DE genes), not sparse selection
  from pre-extracted co-expression communities
- No gene communities have been extracted yet — the LASSO step operates on IC communities
  produced upstream via Gephi modularity, not on the full expression matrix
- The task is gene set enrichment, PubMed literature, or DrugBank actionability evaluation
  of IC genes, which does not require LASSO re-selection
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3474
tools:
- name: Python
  role: Language in which the pipeline scripts (preprocessing.py, creation_matrices.py,
    fusion_networks.py, extraction_classification.py) implement the LASSO-based ICL
    gene selection from IC communities
  repo: https://github.com/DEIB-GECO/GeneNetFusion
- name: Jupyter Notebook
  role: Pipeline.ipynb contains the complete pipeline, including the classification
    step that consumes the ICL-selected genes; run until 'After extraction of communities
    with Gephi' then continue with the community-derived IC files
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
- name: Gephi
  role: Upstream producer of the IC gene communities ('IC_'+str(tumor)+'.csv') that
    serve as the input to LASSO selection
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:6cb34c8ec10555b6ea5b7d9dce5d159bf79fadacd91deec4d62f6d2f45c63e9c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DEIB-GECO/GeneNetFusion
  tools:
  - Gephi
  - Jupyter Notebook
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# sparse-regression-coefficient-extraction

## Summary

Selects representative ICL genes from each per-cancer-type IC gene community by fitting a LASSO regression on the community's TCGA RNA-Seq/miRNA-Seq expression matrix and keeping genes with non-zero L1-shrunk coefficients, for downstream normal/cancer classification with Random Forest.

## When to use

Use when you already have extracted gene communities (e.g. 'IC_'+tumor+'.csv' files for KIRC, LIHC, PRAD) and need a sparse, smaller gene signature per community for a classification task.

## When NOT to use

- The goal is the differential expression baseline (DESeq2 DE genes), not sparse selection from pre-extracted co-expression communities
- No gene communities have been extracted yet — the LASSO step operates on IC communities produced upstream via Gephi modularity, not on the full expression matrix
- The task is gene set enrichment, PubMed literature, or DrugBank actionability evaluation of IC genes, which does not require LASSO re-selection

## Inputs

- IC gene community files ('IC_'+str(tumor)+'.csv' for KIRC, LIHC, PRAD) from the Extracted folder
- TCGA RNA-Seq and miRNA-Seq expression matrices (FPKM values for normal and cancer samples)
- Normal/cancer condition labels for TCGA samples

## Outputs

- Per-cancer-type ICL gene sets (gene symbols with non-zero LASSO coefficients)
- LASSO coefficient value for each selected ICL gene
- Reduced gene features for downstream Random Forest normal/cancer classification

## How to apply

Load each IC gene community file ('IC_'+str(tumor)+'.csv') from the Extracted folder of the GeneNetFusion repository, then subset the TCGA RNA-Seq and miRNA-Seq expression matrices (FPKM for normal and cancer samples of KIRC, LIHC, and PRAD) to those gene symbols and attach normal/cancer condition labels. Fit a LASSO regression on the community expression matrix; the L1 regularization constraint shrinks irrelevant gene coefficients to exactly zero, acting as an embedded feature-selection filter on top of the modularity-based community extraction done in Gephi. Retain only genes with non-zero regression coefficients as the ICL gene set, recording each gene symbol together with its LASSO coefficient. Save one ICL selected-gene record per cancer type so the reduced signatures can be fed to the Random Forest classifier used to validate the communities via normal/cancer classification performance comparison.

## Related tools

- **Python** (Language in which the pipeline scripts (preprocessing.py, creation_matrices.py, fusion_networks.py, extraction_classification.py) implement the LASSO-based ICL gene selection from IC communities) — https://github.com/DEIB-GECO/GeneNetFusion
- **Jupyter Notebook** (Pipeline.ipynb contains the complete pipeline, including the classification step that consumes the ICL-selected genes; run until 'After extraction of communities with Gephi' then continue with the community-derived IC files) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
- **Gephi** (Upstream producer of the IC gene communities ('IC_'+str(tumor)+'.csv') that serve as the input to LASSO selection)

## Examples

```
pip install -r requirements.txt; jupyter notebook Pipeline.ipynb  # run through classification after saving 'IC_'+str(tumor)+'.csv' from Gephi
```

## Evaluation signals

- The LASSO selection script runs without error on each cancer type's IC community file
- Every per-community ICL output file exists in the Extracted folder and contains both gene symbol and coefficient fields
- Every selected ICL gene has a strictly non-zero LASSO coefficient
- Each ICL gene set is a strict subset of its parent IC community gene symbols
- Downstream Random Forest normal/cancer classification produces performance tables (AUC, accuracy, F1) as in the Auc_acc_f1 folder

## Limitations

- The Gephi community-extraction step is manual: the notebook must be run only up to 'After extraction of communities with Gephi', and communities must be saved by hand in Gephi as 'IC_'+str(tumor)+'.csv' before LASSO selection can proceed
- No LASSO regularization-strength (alpha) or tuning procedure is specified in the article or README, so coefficient-shrinkage aggressiveness is not documented
- LASSO-selected ICL genes are only one branch of evaluation; the full IC gene sets are additionally assessed via enrichment analysis, PubMed, DrugBank, and survival association, which LASSO selection does not address

## Evidence

- [readme] Execute the jupyter notebook [Pipeline.ipynb](https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb) until the part 'After extraction of communities with Gephi'.: "Execute the jupyter notebook [Pipeline.ipynb](https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb) until the part 'After extraction of communities with Gephi'."
- [readme] one folder [Extracted] that contains the data regarding all the genes extracted with the pipeline: "one folder [Extracted](https://github.com/DEIB-GECO/GeneNetFusion/tree/master/Extracted) that contains the data regarding all the genes extracted with the pipeline,"
- [readme] save the genes in the folder [Extracted] with the name 'IC_'+str(tumor)+'.csv': "and save the genes in the folder [Extracted](https://github.com/DEIB-GECO/GeneNetFusion/tree/master/Extracted) with the name 'IC_'+str(tumor)+'.csv'."
- [abstract] we provide a computational validation of their relevance for each cancer type by comparing the performance of normal/cancer classification: "we provide a computational validation of their relevance for each cancer type by comparing the performance of normal/cancer classification"
