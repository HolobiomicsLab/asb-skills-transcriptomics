---
name: biomarker-gene-list-generation
description: Use when a large network-derived gene community (IC genes) must be reduced
  to a compact, classification-ready biomarker list and you have per-community gene
  files plus normal/cancer expression matrices available.
when_to_use_negative:
- 'When communities have not yet been extracted: this skill expects ''IC_''+str(tumor)+''.csv''
  files produced after Gephi modularity community detection on the fused co-expression
  network'
- 'When the goal is differential expression-based biomarker identification: the paper
  treats DE genes as a baseline gene signature to compare against, not as the output
  of this LASSO-on-communities step'
- 'When the gene set is already a small, classification-ready signature: re-applying
  L1 shrinkage would re-reduce an already compact feature list'
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: Python
  role: Implements the LASSO fitting and ICL gene/coefficient extraction within extraction_classification.py
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/extraction_classification.py
- name: Jupyter Notebook
  role: Pipeline.ipynb runs the complete pipeline up to saving the IC community gene
    files that feed the LASSO selection step
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
- name: Random Forest classifier
  role: Consumes the ICL selected-gene lists as features for normal/cancer classification
    and produces AUC/accuracy/F1 performance tables
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:02e4e26987cab99b10cd92d162db9f063ea9a0bdfc33f92296f04498fa5ad7e0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter Notebook
  - Python
  - Random Forest classifier
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# biomarker-gene-list-generation

## Summary

Selects a compact representative biomarker list (ICL genes) from each extracted gene community (IC genes) by fitting LASSO (L1-regularized) regression on the community's TCGA expression profiles and keeping only genes with non-zero coefficients, per cancer type, for downstream Random Forest normal/cancer classification.

## When to use

Use when a large network-derived gene community (IC genes) must be reduced to a compact, classification-ready biomarker list and you have per-community gene files plus normal/cancer expression matrices available.

## When NOT to use

- When communities have not yet been extracted: this skill expects 'IC_'+str(tumor)+'.csv' files produced after Gephi modularity community detection on the fused co-expression network
- When the goal is differential expression-based biomarker identification: the paper treats DE genes as a baseline gene signature to compare against, not as the output of this LASSO-on-communities step
- When the gene set is already a small, classification-ready signature: re-applying L1 shrinkage would re-reduce an already compact feature list

## Inputs

- Per-cancer-type IC gene community files 'IC_'+str(tumor)+'.csv' (KIRC, LIHC, PRAD) from the Extracted folder
- TCGA RNA-Seq and miRNA-Seq expression matrices (FPKM values) for normal and cancer samples, labeled with condition

## Outputs

- Per-cancer-type ICL gene lists: selected gene symbols paired with their LASSO regression coefficients
- Reduced per-cancer feature sets consumed by the Random Forest normal/cancer classifier (performance values in the Auc_acc_f1 and Boxplots folders)

## How to apply

Load the per-cancer-type IC gene community files ('IC_'+str(tumor)+'.csv' for KIRC, LIHC, PRAD) from the Extracted folder of the GeneNetFusion repository. Subset the TCGA RNA-Seq and miRNA-Seq expression matrices (FPKM values for normal and cancer samples) to the gene symbols of each community and attach the normal/cancer condition labels. Fit a LASSO regression model on each community's expression matrix, so the L1 constraint on the sum of absolute coefficients shrinks coefficients of non-informative genes to exactly zero. Extract the genes with non-zero coefficients as the ICL gene set, recording each selected gene symbol together with its LASSO coefficient. Save one ICL selected-gene record per cancer type; these lists then serve as reduced feature sets for the Random Forest normal/cancer classifier whose AUC/accuracy/F1 performances are compared against differential-expression and other gene signatures.

## Related tools

- **Python** (Implements the LASSO fitting and ICL gene/coefficient extraction within extraction_classification.py) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/extraction_classification.py
- **Jupyter Notebook** (Pipeline.ipynb runs the complete pipeline up to saving the IC community gene files that feed the LASSO selection step) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
- **Random Forest classifier** (Consumes the ICL selected-gene lists as features for normal/cancer classification and produces AUC/accuracy/F1 performance tables)

## Examples

```
pip install -r requirements.txt && jupyter notebook Pipeline.ipynb
```

## Evaluation signals

- The LASSO selection script runs without error and every per-community ICL output file exists (one per cancer type)
- Each ICL output file contains both gene symbol and LASSO coefficient fields
- Each ICL gene set is a strict subset of its parent IC community, since L1 regularization shrinks non-selected genes' coefficients to zero
- ICL sets are smaller than the IC communities they were derived from, reflecting the intended sparsification
- Downstream Random Forest normal/cancer classification executes and produces performance tables in the Auc_acc_f1 folder

## Limitations

- Demonstrated only on TCGA KIRC, LIHC, and PRAD data; the public repository README ships LIHC data and results only
- Selection optimizes normal/cancer classification relevance; it does not by itself guarantee survival association, DrugBank actionability, or PubMed literature support, which are evaluated separately
- Depends on upstream choices (Gephi modularity community extraction, FPKM expression matrices, zFPKM > −3.0 preprocessing); changes upstream propagate to the ICL lists
- No LASSO penalty strength or solver settings are documented in the article entities or README

## Evidence

- [other] The pipeline selects representative ICL genes by applying LASSO regression on the IC genes, using their expression profiles as features: [evidence span withheld — anchor status: not_found]
- [readme] save the genes in the folder [Extracted](https://github.com/DEIB-GECO/GeneNetFusion/tree/master/Extracted) with the name 'IC_'+str(tumor)+'.csv': "save the genes in the folder [Extracted](https://github.com/DEIB-GECO/GeneNetFusion/tree/master/Extracted) with the name 'IC_'+str(tumor)+'.csv'"
- [readme] one folder [Extracted](https://github.com/DEIB-GECO/GeneNetFusion/tree/master/Extracted) that contains the data regarding all the genes extracted with the pipeline: "one folder [Extracted](https://github.com/DEIB-GECO/GeneNetFusion/tree/master/Extracted) that contains the data regarding all the genes extracted with the pipeline"
- [abstract] comparing the performance of normal/cancer classification for our identified gene sets and other gene signatures, including the typically-used differentially expressed genes: "comparing the performance of normal/cancer classification for our identified gene sets and other gene signatures, including the typically-used differentially expressed genes"
- [results] We apply our computational approach on the TCGA gene expression data for KIRC, LIHC, and PRAD cancer types to build their gene co-expression networks: "We apply our computational approach on the TCGA gene expression data for KIRC, LIHC, and PRAD cancer types to build their gene co-expression networks"
