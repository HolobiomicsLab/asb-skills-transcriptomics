---
name: cross-validation-performance-metric-computation
description: Use when a curated gene set or biomarker panel must be validated computationally
  by measuring normal-vs-cancer classification performance (Accuracy, AUC, F1) under
  cross-validation against competing gene signatures.
when_to_use_negative:
- Input is a raw fused network that has not yet been community-extracted with the
  Gephi modularity method — run the Pipeline.ipynb steps and Gephi extraction first
- The goal is knowledge-based (functional, PubMed, or DrugBank actionability) evaluation
  of IC genes rather than computational classification validation — use 'Knowledge-based
  evaluation of the results.ipynb' instead
- The task involves survival association analysis (Cox proportional-hazards model)
  rather than normal/cancer discrimination
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: Python
  role: 'Implements extraction_classification.py: LASSO selection of ICL genes, Random
    Forest training with 5-fold stratified CV, and computation of Accuracy/AUC/F1
    into auc_acc_f1.csv'
  repo: https://github.com/DEIB-GECO/GeneNetFusion
- name: Jupyter Notebook
  role: Pipeline.ipynb runs the pipeline up to Gephi community extraction; extraction_classification.py
    produces the performance tables
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
- name: Gephi
  role: 'Upstream prerequisite: extracts gene communities from the fused networks
    with the modularity method before classification'
  repo: https://gephi.org
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:9bd8b219585426e04e330fbad635037e998e34083949fc83ac3d8abd531cb170
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# cross-validation-performance-metric-computation

## Summary

Computational validation of extracted gene sets (IC genes) by training a Random Forest classifier with 5-fold stratified cross-validation on TCGA expression features and reporting Accuracy, AUC and F1 to compare LASSO-selected ICL genes against baseline gene sets (AllL, RandL, DE genes) for normal/cancer discrimination.

## When to use

Use when a curated gene set or biomarker panel must be validated computationally by measuring normal-vs-cancer classification performance (Accuracy, AUC, F1) under cross-validation against competing gene signatures.

## When NOT to use

- Input is a raw fused network that has not yet been community-extracted with the Gephi modularity method — run the Pipeline.ipynb steps and Gephi extraction first
- The goal is knowledge-based (functional, PubMed, or DrugBank actionability) evaluation of IC genes rather than computational classification validation — use 'Knowledge-based evaluation of the results.ipynb' instead
- The task involves survival association analysis (Cox proportional-hazards model) rather than normal/cancer discrimination

## Inputs

- Fused co-expression networks per cancer type in Gephi-compatible format (Graphs folder)
- TCGA gene expression matrices for normal and cancer samples (Datasets_created folder)
- Extracted gene communities saved as 'IC_'+str(tumor)+'.csv' (Extracted folder)
- Baseline gene sets: AllL, RandL, and differentially expressed genes (DESeq2)

## Outputs

- auc_acc_f1.csv performance table with Accuracy, AUC and F1 per cancer type and gene set (Auc_acc_f1 folder)
- Boxplots comparing classification performances across gene sets (Boxplots folder)
- Performance values mirroring the classification-performance table (Fig. 3)

## How to apply

Load the fused co-expression networks from the repository's Graphs folder (Gephi-compatible format) and TCGA gene expression matrices from Datasets_created, then extract gene communities with the modularity method in Gephi and save them as 'IC_'+str(tumor)+'.csv' in the Extracted folder. Within each IC gene set, run LASSO regression to select a sparse subset of representative genes (ICL genes) with non-zero coefficients. Train a Random Forest classifier with 5-fold stratified cross-validation on the selected gene expression features to discriminate cancer versus normal samples for KIRC, LIHC and PRAD. Compute Accuracy, AUC (area under the ROC curve) and F1 score per cancer type and gene set, and write the values to auc_acc_f1.csv mirroring the repo's Auc_acc_f1 folder. Compare the ICL gene set against the AllL, RandL and differentially expressed (DESeq2) gene baselines; the rationale is that outperforming these baselines provides computational evidence of the IC genes' relevance.

## Related tools

- **Python** (Implements extraction_classification.py: LASSO selection of ICL genes, Random Forest training with 5-fold stratified CV, and computation of Accuracy/AUC/F1 into auc_acc_f1.csv) — https://github.com/DEIB-GECO/GeneNetFusion
- **Jupyter Notebook** (Pipeline.ipynb runs the pipeline up to Gephi community extraction; extraction_classification.py produces the performance tables) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
- **Gephi** (Upstream prerequisite: extracts gene communities from the fused networks with the modularity method before classification) — https://gephi.org

## Examples

```
python extraction_classification.py
```

## Evaluation signals

- auc_acc_f1.csv exists and contains Accuracy, AUC and F1 fields with all values in the 0–1 range
- Values in the table match the reported classification-performance comparison (Fig. 3) per cancer type and gene set
- LASSO-selected ICL genes outperform AllL and RandL baselines across cancer types, with the single exception of the PRAD AUC of RandL
- Classification is performed with 5-fold stratified cross-validation on expression features of the non-zero LASSO coefficient genes
- Boxplots in the Boxplots folder reproduce the performance comparisons across gene sets

## Limitations

- The PRAD AUC of RandL shows very wide variance and is the single case where ICL does not outperform a baseline
- Results cover three cancer types only (KIRC, LIHC and PRAD TCGA data sets)
- Performance depends on the upstream Gephi modularity community extraction, which is a manual step between Pipeline.ipynb and classification
- A Random Forest classifier is used; conclusions are relative to this classifier and to the specific baseline gene sets (AllL, RandL, DE genes)

## Evidence

- [abstract] computational validation of IC gene classification performance: "we provide a computational validation of their relevance for each cancer type by comparing the performance of normal/cancer classification"
- [readme] Auc_acc_f1 folder contents: "that contains the tables with the performances values after having executed [extraction_classification.py]"
- [results] TCGA cancer types used: "We apply our computational approach on the TCGA gene expression data for KIRC, LIHC, and PRAD cancer types to build their gene co-expression networks"
