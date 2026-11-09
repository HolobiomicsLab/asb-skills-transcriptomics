---
name: lasso-gene-feature-selection
description: Use when a gene community or candidate gene set (e.g. IC genes from Gephi
  community extraction) must be reduced to a sparse, discriminative signature prior
  to supervised classification on expression data.
when_to_use_negative:
- Input is already a reduced feature table or pre-selected gene signature — LASSO
  would redundantly re-shrink an already sparse set
- The goal is unsupervised community detection or network fusion (handled earlier
  in Pipeline.ipynb by Gephi modularity / SNF), not supervised feature selection
- Sample labels (normal vs cancer) are unavailable, since LASSO here is a supervised
  model requiring condition labels
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: Python
  role: Implements the LASSO fitting and extraction of non-zero-coefficient genes
    within extraction_classification.py
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/extraction_classification.py
- name: Jupyter Notebook
  role: Runs the pipeline steps up to and including gene selection and classification
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:9fad3d527ebcd73f0ed22d8d0a60ce7f79c684544644f6c195da4a5f2946e8c5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Gephi
  - Jupyter Notebook
  - Python
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# lasso-gene-feature-selection

## Summary

Selects representative genes (ICL genes) from each fused-network gene community by fitting LASSO (L1-regularized) regression on TCGA expression profiles, shrinking irrelevant gene coefficients to zero so only non-zero-coefficient genes are retained for downstream classification.

## When to use

Use when a gene community or candidate gene set (e.g. IC genes from Gephi community extraction) must be reduced to a sparse, discriminative signature prior to supervised classification on expression data.

## When NOT to use

- Input is already a reduced feature table or pre-selected gene signature — LASSO would redundantly re-shrink an already sparse set
- The goal is unsupervised community detection or network fusion (handled earlier in Pipeline.ipynb by Gephi modularity / SNF), not supervised feature selection
- Sample labels (normal vs cancer) are unavailable, since LASSO here is a supervised model requiring condition labels

## Inputs

- Per-cancer-type IC gene community files ('IC_'+str(tumor)+'.csv') from the GeneNetFusion Extracted folder
- TCGA RNA-Seq and miRNA-Seq expression matrices (FPKM, normal and cancer samples) for KIRC, LIHC, PRAD from Datasets_created
- Normal/cancer condition labels for the TCGA samples

## Outputs

- Per-community ICL gene sets: gene symbols with non-zero LASSO coefficients, one file per cancer type
- LASSO coefficient values for each selected gene

## How to apply

Load the per-cancer-type IC gene community files ('IC_'+str(tumor)+'.csv' for KIRC, LIHC, PRAD) from the repository's Extracted folder, then subset the TCGA RNA-Seq and miRNA-Seq expression matrices (FPKM values for normal and cancer samples) to those gene symbols and attach normal/cancer condition labels. Fit a LASSO regression on each community's expression matrix; the L1 penalty on the sum of absolute coefficients shrinks coefficients of irrelevant genes to exactly zero, acting as embedded feature selection. Extract the genes with non-zero coefficients as the ICL gene set, recording each gene symbol together with its LASSO coefficient. Save one ICL record per cancer type for downstream Random Forest normal/cancer classification (the step implemented in the repo's extraction_classification.py, whose AUC/accuracy/F1 tables land in Auc_acc_f1).

## Related tools

- **Python** (Implements the LASSO fitting and extraction of non-zero-coefficient genes within extraction_classification.py) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/extraction_classification.py
- **Jupyter Notebook** (Runs the pipeline steps up to and including gene selection and classification) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb

## Examples

```
from sklearn.linear_model import Lasso; lasso = Lasso(alpha=0.01); lasso.fit(X_ic, y_label); icl_genes = [g for g, c in zip(ic_genes, lasso.coef_) if c != 0]
```

## Evaluation signals

- The LASSO selection script runs without error and every per-community ICL output file exists
- Each ICL output file contains both gene symbol and coefficient fields
- Only genes with strictly non-zero LASSO coefficients are retained (zero-coefficient genes are excluded)
- Downstream Random Forest 5-fold stratified CV on the ICL genes yields Accuracy, AUC, and F1 values in the 0–1 range written to auc_acc_f1.csv
- ICL genes outperform the AllL and RandL baselines in normal/cancer classification across cancer types, except the PRAD AUC of RandL (which shows very wide variance)

## Limitations

- One ICL gene set is produced per cancer type; the selection is community-specific and does not generalize across cancer types
- Reported performance has exceptions and variance issues: RandL beats ICL on the PRAD AUC with very wide variance, so ICL superiority is not universal
- The README notes only LIHC data, scripts and results are fully contained in the repository, so KIRC/PRAD communities must come from the Results folder gene symbol lists

## Evidence

- [methods] LASSO selection rationale: [evidence span withheld — anchor status: not_found]
- [methods] Input handling: [evidence span withheld — anchor status: not_found]
- [methods] Output definition: [evidence span withheld — anchor status: not_found]
- [readme] README naming convention: "Use [Gephi](https://gephi.org) in order to extract the relevant communities and save the genes in the folder [Extracted](https://github.com/DEIB-GECO/GeneNetFusion/tree/master/Extracted) with the"
- [results] Validation finding: [evidence span withheld — anchor status: not_found]
