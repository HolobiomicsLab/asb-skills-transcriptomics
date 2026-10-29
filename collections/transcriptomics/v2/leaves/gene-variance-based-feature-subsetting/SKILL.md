---
name: gene-variance-based-feature-subsetting
description: Use when evaluating a pathway-activity classifier's robustness to missing
  data or cross-platform gene annotation gaps, by testing whether balanced accuracy
  stays high (>90%) with far fewer genes than the full annotated set.
when_to_use_negative:
- The goal is unsupervised grouping of cancer patients — DeepCC is specifically developed
  for classification but not clustering of cancer patients
- The input is already a functional spectra (pathway activity) feature table; subsetting
  genes applies only at the gene-expression level before getFunctionalSpectra
- A single sample without matched reference expression profile — single-sample classification
  requires a reference profile (e.g., refExp = "COADREAD") and does not exercise this
  subsampling robustness analysis
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3525
tools:
- name: DeepCC
  role: Provides getFunctionalSpectra to compute functional spectra from each gene
    subset, train_DeepCC_model (keras ANN) for classifier training, and cross_validation
    for metric evaluation
  repo: https://github.com/CityUHK-CompBio/DeepCC
- name: msigdbr
  role: Supplies the MSigDB gene set database (get_msigdbr()) used to compute functional
    spectra from the subsetted gene lists
- name: keras
  role: Backend framework for the DeepCC ANN classifier; must be installed in R before
    training models on each subsample
  repo: https://keras.rstudio.com/
- name: R
  role: Implementation language; DeepCC is installed from GitHub via devtools/remotes
  repo: https://github.com/CityUHK-CompBio/DeepCC
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:2d73bf51fbffc8fed57e72becb4fa0460c00a59b97a3dfcd33edfd0c7cf7629e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CityUHK-CompBio/DeepCC
  tools:
  - DeepCC
  - R
  - keras
  - msigdbr
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3525
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gene-variance-based-feature-subsetting

## Summary

A robustness simulation in which functional spectra for DeepCC are recomputed from random subsets of the most variable genes (e.g., 1000 up to 20,531 genes from TCGA CRC data, n=456) to quantify how classification performance degrades when gene annotation is incomplete across platforms.

## When to use

Use when evaluating a pathway-activity classifier's robustness to missing data or cross-platform gene annotation gaps, by testing whether balanced accuracy stays high (>90%) with far fewer genes than the full annotated set.

## When NOT to use

- The goal is unsupervised grouping of cancer patients — DeepCC is specifically developed for classification but not clustering of cancer patients
- The input is already a functional spectra (pathway activity) feature table; subsetting genes applies only at the gene-expression level before getFunctionalSpectra
- A single sample without matched reference expression profile — single-sample classification requires a reference profile (e.g., refExp = "COADREAD") and does not exercise this subsampling robustness analysis

## Inputs

- TCGA colorectal cancer (COADREAD) gene expression matrix (n=456 samples, rows = patient samples, columns = Entrez gene identifiers)
- Character vector of subtype training labels (NA allowed; DeepCC ignores them automatically)
- Gene set collection: MSigDB v7 (22,596 gene sets) via default get_gene_sets or MSigDBr from msigdbr::get_msigdbr()

## Outputs

- Per-subset-size performance table (e.g., gene_subsampling_performance.csv) with accuracy, balanced accuracy, mean sensitivity, and mean specificity at each gene count
- Performance-versus-gene-number robustness curve (reproduces Fig. 1c of the DeepCC paper)

## How to apply

Load the TCGA colorectal cancer expression matrix (n=456 samples) with subtype labels and rank genes by variance to define the most-variable gene pool. Iteratively draw random subsets of these genes at sizes from 1000 up to 20,531 genes; for each subset, compute functional spectra with getFunctionalSpectra using the default MSigDB v7 collection (22,596 gene sets, or MSigDBr from msigdbr's get_msigdbr()). Train a keras-based DeepCC model with train_DeepCC_model on each subset's functional spectra and evaluate with cross_validation, recording accuracy, balanced accuracy, mean sensitivity, and mean specificity. Compile the metrics (e.g., gene_subsampling_performance.csv) into a performance-versus-gene-number trend reproducing Fig. 1c. The rationale is that pathway-level functional spectra act as an aggregation layer, so classification should degrade only gradually as the gene subset shrinks, demonstrating platform independence.

## Related tools

- **DeepCC** (Provides getFunctionalSpectra to compute functional spectra from each gene subset, train_DeepCC_model (keras ANN) for classifier training, and cross_validation for metric evaluation) — https://github.com/CityUHK-CompBio/DeepCC
- **msigdbr** (Supplies the MSigDB gene set database (get_msigdbr()) used to compute functional spectra from the subsetted gene lists)
- **keras** (Backend framework for the DeepCC ANN classifier; must be installed in R before training models on each subsample) — https://keras.rstudio.com/
- **R** (Implementation language; DeepCC is installed from GitHub via devtools/remotes) — https://github.com/CityUHK-CompBio/DeepCC

## Examples

```
library(DeepCC); library(keras); fs_sub <- getFunctionalSpectra(eps[, top_var_genes]); deepcc_model <- train_DeepCC_model(fs_sub, labels); cv <- cross_validattion(deepcc_model, fs_sub, labels)
```

## Evaluation signals

- Balanced accuracy remains >90% when functional spectra are computed from only 2000 of the most variable TCGA CRC genes
- Accuracy and balanced accuracy degrade only gradually as the random gene subset shrinks toward 1000 genes, matching the paper's Fig. 1c trend
- Mean sensitivity and mean specificity are recorded for every subset size and remain consistently high across the range 1000–20,531 genes
- Each subset's functional spectra use the same gene set collection (MSigDB v7, 22,596 gene sets) so only the gene pool varies between runs
- Cross-validation (cross_validation) is applied per subset size, so metrics are computed out-of-fold rather than on training data

## Limitations

- DeepCC depends on comprehensive gene set prior knowledge to avoid potential bias; major characteristic signaling pathways or biological processes for each subtype should be included in the gene set database
- Subsampling tests robustness to missing annotation, but performance will ultimately degrade as the subset shrinks; the curve should be inspected rather than assumed flat
- Results are specific to the TCGA CRC cohort (n=456) and MSigDB v7; other cancer types or gene set versions may show different robustness profiles

## Evidence

- [abstract] Simulation analysis based on random subsampling of genes demonstrated the robustness of DeepCC to missing data: "Simulation analysis based on random subsampling of genes demonstrated the robustness of DeepCC to missing data"
- [results] DeepCC maintained balanced accuracy >90% with only 2000 genes: [evidence span withheld — anchor status: not_found]
- [methods] Default MSigDB v7 gene set collection: "By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra."
- [methods] MSigDBr attained from R package msigdbr: "MSigDBr attained from R package `msigdbr` with function get_msigdbr() in DeepCC"
- [readme] DeepCC input format and default gene sets: "a data.frame containing gene expression profiles (each row represents one patient sample and the column names should be Entrez identifiers of genes)"
