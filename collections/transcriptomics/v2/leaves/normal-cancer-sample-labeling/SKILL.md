---
name: normal-cancer-sample-labeling
description: Use when per-cancer-type gene communities (IC genes) must be reduced
  to representative genes and samples carry normal/cancer condition labels needed
  for classification-based validation.
when_to_use_negative:
- Input is already a filtered feature table of selected biomarker genes with no need
  for community-level LASSO shrinkage
- The goal is to construct or filter the co-expression networks themselves (disparity
  filter, 99th-percentile Euclidean thresholding, permutation-based Pearson filtering)
  rather than select representative genes from already-extracted communities
- Cancer types other than the TCGA KIRC, LIHC, and PRAD communities shipped in the
  repository, since the IC community files are specific to those three tumor analyses
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Python
  role: Language of the pipeline scripts (extraction_classification.py) that subset
    the FPKM matrices, attach normal/cancer labels, and fit the LASSO model per community
  repo: https://github.com/DEIB-GECO/GeneNetFusion
- name: Jupyter Notebook
  role: Pipeline.ipynb contains the complete pipeline through which the IC community
    files are produced and the extraction/classification step is executed
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
- name: Gephi
  role: 'Upstream step: extracts the relevant gene communities via the modularity
    method and saves them as the ''IC_''+str(tumor)+''.csv'' files consumed by this
    skill'
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:20db04653244b2c90d5abac4000ed2e888357bc648a8f854081a04fbdade7447
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DEIB-GECO/GeneNetFusion
  tools:
  - Gephi
  - Jupyter Notebook
  - Python
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# normal-cancer-sample-labeling

## Summary

Labels TCGA RNA-Seq/miRNA-Seq FPKM samples as normal or cancer and, within each extracted IC gene community ('IC_'+str(tumor)+'.csv'), applies LASSO L1 regression to shrink irrelevant gene coefficients to zero, yielding a smaller ICL gene set per cancer type (KIRC, LIHC, PRAD) for downstream Random Forest normal/cancer classification.

## When to use

Use when per-cancer-type gene communities (IC genes) must be reduced to representative genes and samples carry normal/cancer condition labels needed for classification-based validation.

## When NOT to use

- Input is already a filtered feature table of selected biomarker genes with no need for community-level LASSO shrinkage
- The goal is to construct or filter the co-expression networks themselves (disparity filter, 99th-percentile Euclidean thresholding, permutation-based Pearson filtering) rather than select representative genes from already-extracted communities
- Cancer types other than the TCGA KIRC, LIHC, and PRAD communities shipped in the repository, since the IC community files are specific to those three tumor analyses

## Inputs

- Per-cancer-type IC gene community files 'IC_KIRC.csv', 'IC_LIHC.csv', 'IC_PRAD.csv' from the Extracted folder of the GeneNetFusion repository
- TCGA RNA-Seq expression matrices (FPKM) for normal and cancer samples
- TCGA miRNA-Seq expression matrices (FPKM) for normal and cancer samples
- Normal/cancer condition labels for each TCGA sample

## Outputs

- Per-cancer-type ICL selected-gene records containing gene symbol and LASSO regression coefficient
- Labeled gene-by-sample expression matrices restricted to community genes
- Classification performance values (AUC, accuracy, F1) stored in the Auc_acc_f1 folder after running extraction_classification.py

## How to apply

Load each per-tumor community file ('IC_'+str(tumor)+'.csv' for KIRC, LIHC, PRAD) from the GeneNetFusion Extracted folder, then subset the TCGA RNA-Seq and miRNA-Seq expression matrices (FPKM values for normal and cancer samples) to the gene symbols in that community, attaching the normal/cancer condition labels to each sample. Fit a LASSO regression on the community expression matrix with the genes as features, relying on the L1 constraint on the sum of absolute coefficients to shrink coefficients of irrelevant genes to exactly zero. Collect the genes with non-zero coefficients as the ICL gene set, recording each selected gene symbol together with its LASSO coefficient. Feed the labeled, community-restricted feature matrices and ICL gene sets into the Random Forest classifier used for the normal/cancer classification comparison against DE-gene signatures.

## Related tools

- **Python** (Language of the pipeline scripts (extraction_classification.py) that subset the FPKM matrices, attach normal/cancer labels, and fit the LASSO model per community) — https://github.com/DEIB-GECO/GeneNetFusion
- **Jupyter Notebook** (Pipeline.ipynb contains the complete pipeline through which the IC community files are produced and the extraction/classification step is executed) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
- **Gephi** (Upstream step: extracts the relevant gene communities via the modularity method and saves them as the 'IC_'+str(tumor)+'.csv' files consumed by this skill)

## Examples

```
pip install -r requirements.txt && jupyter nbconvert --to notebook --execute Pipeline.ipynb
```

## Evaluation signals

- The LASSO selection script runs without error for all three tumor types (KIRC, LIHC, PRAD)
- Every per-community ICL output file exists and contains both gene symbol and coefficient fields
- Only genes with non-zero LASSO coefficients appear in each ICL gene set (irrelevant genes shrunk to zero)
- Every gene symbol in an ICL output is a member of the corresponding IC community file from the Extracted folder
- Downstream Random Forest classification produces performance tables in the Auc_acc_f1 folder after running extraction_classification.py

## Limitations

- The skill depends on communities already extracted with the Gephi modularity method; if the IC community files are missing or renamed, labeling and LASSO selection cannot proceed
- LASSO coefficients depend on the FPKM-normalized TCGA expression matrices and normal/cancer label assignment; mislabeled or unlabeled samples will corrupt both selection and classification
- Validation of the selected genes is computational (classification comparison vs DE genes and other signatures); biological relevance requires the separate knowledge-based evaluation notebook

## Evidence

- [abstract] comparing the performance of normal/cancer classification for our identified gene sets and other gene signatures, including the typically-used differentially expressed genes: "comparing the performance of normal/cancer classification for our identified gene sets and other gene signatures, including the typically-used differentially expressed genes"
- [intro] extraction_classification.py, python files which contain the main steps of the process: "extraction_classification.py, python files which contain the main steps of the process"
- [readme] The jupyter notebook [Pipeline.ipynb](https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb) contains the complete pipeline: "The jupyter notebook [Pipeline.ipynb](https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb) contains the complete pipeline"
- [readme] that contains the tables with the performances values after having executed extraction_classification.py: "that contains the tables with the performances values after having executed extraction_classification.py"
