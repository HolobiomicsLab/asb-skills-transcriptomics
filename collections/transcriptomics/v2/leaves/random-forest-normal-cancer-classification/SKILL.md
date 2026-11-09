---
name: random-forest-normal-cancer-classification
description: Use when you need computational validation that gene communities extracted
  from fused networks are relevant, by testing whether they classify TCGA normal/cancer
  samples better than DE or random gene sets.
when_to_use_negative:
- The goal is biological/functional characterization of the gene communities (use
  the Knowledge-based evaluation notebook with g:Profiler, DrugBank, PubMed instead
  of classification)
- The networks have not yet been built and communities extracted — this classification
  step requires 'IC_'+str(tumor)+'.csv' files produced after running Pipeline.ipynb
  up to 'After extraction of communities with Gephi'
- The goal is de novo biomarker discovery from expression data alone; this skill only
  validates already-extracted IC gene sets against baseline signatures
tools:
- name: Python
  role: 'Implements extraction_classification.py: LASSO gene selection, Random Forest
    training with 5-fold stratified CV, and Accuracy/AUC/F1 computation'
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/extraction_classification.py
- name: Jupyter Notebook
  role: Pipeline.ipynb contains the complete pipeline up to the point where Gephi-extracted
    communities feed into classification
  repo: https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
- name: Gephi
  role: 'Upstream: modularity method extracts gene communities from fused networks
    that become the IC gene features for classification'
- name: GMQL
  role: Retrieves the TCGA gene expression data used as classification features from
    the Datasets_created folder
  repo: http://gmql.eu
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41540-021-00175-9
    title: Computational analysis of fused co-expression networks for the identification
      of candidate cancer gene biomarkers
  source_content_hash: sha256:ac70488c9521af3b19bfefbf03b9f77278046e996e99ad08e1443bf2a35d6ae9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GMQL
  - Gephi
  - Jupyter Notebook
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# random-forest-normal-cancer-classification

## Summary

Computationally validates extracted IC gene communities from fused co-expression networks by using LASSO-selected ICL genes as features in a Random Forest classifier (5-fold stratified cross-validation) to discriminate normal vs cancer TCGA samples, benchmarked against AllL, RandL, and DE gene signatures.

## When to use

Use when you need computational validation that gene communities extracted from fused networks are relevant, by testing whether they classify TCGA normal/cancer samples better than DE or random gene sets.

## When NOT to use

- The goal is biological/functional characterization of the gene communities (use the Knowledge-based evaluation notebook with g:Profiler, DrugBank, PubMed instead of classification)
- The networks have not yet been built and communities extracted — this classification step requires 'IC_'+str(tumor)+'.csv' files produced after running Pipeline.ipynb up to 'After extraction of communities with Gephi'
- The goal is de novo biomarker discovery from expression data alone; this skill only validates already-extracted IC gene sets against baseline signatures

## Inputs

- IC gene community files 'IC_'+str(tumor)+'.csv' from the Extracted folder (genes extracted from Gephi modularity communities of 99th-percentile disparity-filter-pruned fused networks)
- TCGA gene expression matrices from the Datasets_created folder (retrieved via GMQL/PyGMQL, zFPKM > -3.0 preprocessed)
- Baseline gene sets: AllL, RandL, and DE genes (DESeq2-identified differentially expressed genes)
- extraction_classification.py script

## Outputs

- auc_acc_f1.csv performance table in the Auc_acc_f1 folder with Accuracy, AUC, and F1 values per cancer type and gene set
- Boxplots in the Boxplots folder comparing classification performance across gene sets (mirrors Fig. 3 of the article)

## How to apply

Within each IC gene set (saved as 'IC_'+str(tumor)+'.csv' in the Extracted folder after modularity community detection in Gephi), run LASSO regression to select a sparse subset of representative ICL genes with non-zero coefficients. Train a Random Forest classifier on the TCGA gene expression values of these selected genes with 5-fold stratified cross-validation to discriminate cancer versus normal samples. Repeat for baseline gene sets (AllL, RandL, DE genes) per cancer type (KIRC, LIHC, PRAD). Compute Accuracy, AUC, and F1 per cancer type and gene set via extraction_classification.py, writing results to the Auc_acc_f1 folder (auc_acc_f1.csv) and visualizing comparisons as boxplots. Correct application is judged by LASSO-selected ICL genes outperforming AllL and RandL baselines across cancer types (the only exception being the PRAD AUC of RandL, which shows very wide variance).

## Related tools

- **Python** (Implements extraction_classification.py: LASSO gene selection, Random Forest training with 5-fold stratified CV, and Accuracy/AUC/F1 computation) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/extraction_classification.py
- **Jupyter Notebook** (Pipeline.ipynb contains the complete pipeline up to the point where Gephi-extracted communities feed into classification) — https://github.com/DEIB-GECO/GeneNetFusion/blob/master/Pipeline.ipynb
- **Gephi** (Upstream: modularity method extracts gene communities from fused networks that become the IC gene features for classification)
- **GMQL** (Retrieves the TCGA gene expression data used as classification features from the Datasets_created folder) — http://gmql.eu

## Examples

```
python extraction_classification.py
```

## Evaluation signals

- auc_acc_f1.csv exists in the Auc_acc_f1 folder after running extraction_classification.py and contains Accuracy, AUC, and F1 fields with values in the 0-1 range
- LASSO-selected ICL genes from fused networks outperform AllL and RandL baselines across cancer types, with the single exception of the PRAD AUC of RandL (very wide variance)
- Performance values match the classification-performance comparison shown in the article (Fig. 3 boxplots)
- Per-fold evaluation uses 5-fold stratified cross-validation, so scores are means over folds rather than single-split values
- ICL gene sets are sparse: only genes with non-zero LASSO coefficients are used as classifier features

## Limitations

- The repository README states it contains 'all data, scripts and results related to the LIHC tumor analysis', so results for KIRC and PRAD may need to be regenerated or obtained from the paper
- The PRAD AUC of RandL shows very wide variance, so that single comparison is not a reliable discriminator
- Random Forest CV performance depends on the upstream Gephi modularity community extraction, which is a manual step outside the notebook pipeline

## Evidence

- [abstract] computational validation via normal/cancer classification: "we provide a computational validation of their relevance for each cancer type by comparing the performance of normal/cancer classification"
- [abstract] classification compared against DE genes: "comparing the performance of normal/cancer classification for our identified gene sets and other gene signatures, including the typically-used differentially expressed genes"
- [readme] Auc_acc_f1 folder contents: "one folder Auc_acc_f1 that contains the tables with the performances values after having executed extraction_classification.py"
- [readme] script roles: "extraction_classification.py, python files which contain the main steps of the process"
- [methods] Gephi community extraction upstream of classification: "gene communities in the fused network are extracted using the modularity method implemented in Gephi39"
- [results] ICL outperform baselines finding: [evidence span withheld — anchor status: not_found]
