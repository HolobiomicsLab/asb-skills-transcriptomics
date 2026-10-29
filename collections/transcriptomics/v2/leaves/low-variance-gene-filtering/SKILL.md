---
name: low-variance-gene-filtering
description: Use when preparing gene-by-sample expression matrices for differential
  network inference and the feature space is large relative to sample size — filter
  by per-gene variance (here a cutoff of 0.4) before running BoostDiff or similar
  tree-based methods.
when_to_use_negative:
- The dataset has already been preprocessed and feature-selected (e.g. TCGA or COVID-19
  matrices from the tutorial/preprocessing scripts) — re-filtering could remove disease-relevant
  genes.
- The downstream tool does not require a reduced feature space and sample sizes are
  ample (>= 30 per condition), making aggressive variance pruning unnecessary or harmful.
- Genes of low variance are themselves the object of study (e.g. housekeeping-gene
  control analysis) rather than input to differential network inference.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: BoostDiff
  role: Downstream consumer of the variance-filtered matrices; filtering reduces the
    gene space prior to building the AdaBoost differential-tree ensemble
  repo: https://github.com/scibiome/boostdiff_inference
- name: dplyr
  role: R package used to aggregate replicates and assemble the tab-separated Gene-by-sample
    matrices that are then variance-filtered
- name: pySMOTE
  role: Applied after variance filtering to oversample the stress condition to 48
    samples per class
  repo: https://github.com/akazs/pySMOTE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbae034
    title: Inference of differential gene regulatory networks using boosted differential
      trees
  source_content_hash: sha256:b0bfba61101f55057e669dce000bf8a1882a3d725c213ed74a322c469af39931
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scibiome/boostdiff_inference
  tools:
  - BoostDiff
  - GEOquery
  - Python
  - R
  - dplyr
  - pySMOTE
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Low-variance gene filtering

## Summary

A preprocessing step that removes genes below a variance cutoff (e.g. 0.4 in the GSE27219 B. subtilis stress dataset) before differential network inference, shrinking the feature space so BoostDiff models focus on genes whose expression actually varies across conditions.

## When to use

Use when preparing gene-by-sample expression matrices for differential network inference and the feature space is large relative to sample size — filter by per-gene variance (here a cutoff of 0.4) before running BoostDiff or similar tree-based methods.

## When NOT to use

- The dataset has already been preprocessed and feature-selected (e.g. TCGA or COVID-19 matrices from the tutorial/preprocessing scripts) — re-filtering could remove disease-relevant genes.
- The downstream tool does not require a reduced feature space and sample sizes are ample (>= 30 per condition), making aggressive variance pruning unnecessary or harmful.
- Genes of low variance are themselves the object of study (e.g. housekeeping-gene control analysis) rather than input to differential network inference.

## Inputs

- Tab-separated disease/stress expression matrix with 'Gene' as first column (rows = genes, columns = samples), e.g. exprs_salt.txt
- Tab-separated control expression matrix with the same gene set, e.g. exprs_smm.txt
- Variance cutoff (0.4 in the GSE27219 B. subtilis workflow)

## Outputs

- Filtered stress and control expression matrices restricted to genes exceeding the variance cutoff, preserving matched gene sets across the two conditions

## How to apply

After downloading and aggregating replicates into tab-separated stress and control matrices with 'Gene' as the first column, compute a variance measure for each gene across samples and retain only genes passing the cutoff (0.4 for the B. subtilis salt/heat/ethanol contexts). This filtering happens upstream of SMOTE oversampling to 48 samples per condition, so both disease and control matrices must be filtered on the same gene set to satisfy BoostDiff's requirement that the two inputs share identical features. Rationale: boosting ensembles of 50 differential trees over n_features=2500 regulator genes can overfit when many genes are near-invariant, so variance prefiltering concentrates the search on genes with enough signal to contribute differential variance improvement (DVI) splits.

## Related tools

- **BoostDiff** (Downstream consumer of the variance-filtered matrices; filtering reduces the gene space prior to building the AdaBoost differential-tree ensemble) — https://github.com/scibiome/boostdiff_inference
- **dplyr** (R package used to aggregate replicates and assemble the tab-separated Gene-by-sample matrices that are then variance-filtered)
- **pySMOTE** (Applied after variance filtering to oversample the stress condition to 48 samples per class) — https://github.com/akazs/pySMOTE

## Examples

```
df_new <- df_expr %>% group_by(hgnc) %>% summarise_all("mean"); write.table(df_new, file = "df_crohns.txt", quote = FALSE, sep = "\t", row.names = FALSE)  # then keep genes with variance > 0.4 before BoostDiff
```

## Evaluation signals

- The filtered disease and control matrices contain exactly the same set of gene rows, with 'Gene' as the first column name.
- Every retained gene has variance above the 0.4 cutoff; a spot-check of removed genes should show near-constant expression across samples.
- After filtering, the dataset still supports BoostDiff's minimum of 15 samples per condition and the n_features=2500 regulator pool.
- Downstream validation: the filtered network's top-ten hubs (e.g. csbD, ohrB, yuzA, yycD for salt/heat) should be SigB regulon members, indicating the filter did not remove biologically relevant variance genes.

## Limitations

- A fixed cutoff of 0.4 is dataset- and scale-dependent; different normalization schemes (BoostDiff defaults to 'unit_variance' normalization) change gene variances and would require re-tuning the threshold.
- Filtering before SMOTE means synthetic oversampled samples inherit the variance profile of the retained gene set; genes discarded as low-variance cannot be recovered downstream.
- Real transcriptomics datasets like the Crohn's disease one are large enough to need a high-performance computing cluster, so over-aggressive filtering may be tempting but risks discarding true differential signals.

## Evidence

- [other] Filter genes by the variance cutoff of 0.4 and oversample each stress condition with SMOTE (akazs/pySMOTE) to 48 samples per condition to balance the two classes.: [evidence span withheld — anchor status: not_found]
- [readme] Note that the two inputs should have the same set of features, where the first column should be named "Gene".: "Note that the two inputs should have the same set of features, where the first column should be named "Gene"."
- [readme] For real transcriptomics datasets, we recommend to set a relatively low number of base learners, e.g. set `n_estimators=50`, so that 50 adaptively boosted differential trees will be built to avoid overfitting.: "we recommend to set a relatively low number of base learners, e.g. set `n_estimators=50`, so that 50 adaptively boosted differential trees will be built to avoid overfitting"
- [methods] you might need to map the genes to the desired genes IDs, filter, and/or preprocess them to select specific samples to analyze for your study: "you might need to map the genes to the desired genes IDs, filter, and/or preprocess them to select specific samples to analyze for your study"
