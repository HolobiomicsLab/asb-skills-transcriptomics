---
name: single-cell-rna-seq-count-matrix-loading
description: Use when starting an scRNAseq analysis from raw benchmark count data
  (Zhengmix, SCMixology) and a SingleCellExperiment object with a standardized feature
  set is needed before dimension reduction with corral or glmPCA.
when_to_use_negative:
- The input is already a reduced-dimension embedding or a logcounts matrix — CA in
  corral operates on raw counts via the Pearson chi-squared residual matrix, avoiding
  log-transformation
- The task requires complex-tissue datasets with independently validated cell identities
  — the article notes ground-truth benchmarks are limited to simple pseudo-cell mixtures
  or pools of distinct cell types
- The analysis is multi-table batch integration rather than single-table loading —
  corralm, not single-table corral, is the appropriate entry point
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: corral
  role: Consumes the loaded SingleCellExperiment directly for CA dimension reduction;
    installed via BiocManager or from its GitHub repo
  repo: github.com/laurenhsu1/corral
- name: Bioconductor
  role: Framework providing the SingleCellExperiment class that corral interfaces
    with directly
- name: R
  role: Language environment in which the count data is loaded and prepared
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:9bbf19a0cbb291778b6b2d4ba095714c3e077c98761b1a2273c80bb30efe5897
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - corral
  - glmPCA
  - gridExtra
  - made4
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# single-cell-rna-seq-count-matrix-loading

## Summary

Load benchmark single-cell RNA-seq count datasets (e.g., Zhengmix4eq, Zhengmix8eq, SCMixology) as SingleCellExperiment objects in R/Bioconductor and optionally down-select them to a standardized feature count (e.g., 1500 features) so they can be passed directly to corral or glmPCA.

## When to use

Use when starting an scRNAseq analysis from raw benchmark count data (Zhengmix, SCMixology) and a SingleCellExperiment object with a standardized feature set is needed before dimension reduction with corral or glmPCA.

## When NOT to use

- The input is already a reduced-dimension embedding or a logcounts matrix — CA in corral operates on raw counts via the Pearson chi-squared residual matrix, avoiding log-transformation
- The task requires complex-tissue datasets with independently validated cell identities — the article notes ground-truth benchmarks are limited to simple pseudo-cell mixtures or pools of distinct cell types
- The analysis is multi-table batch integration rather than single-table loading — corralm, not single-table corral, is the appropriate entry point

## Inputs

- scRNA-seq count matrix from benchmark datasets such as Zhengmix4eq, Zhengmix8eq, Zhengmix, or SCMixology
- Original-publication cell type labels for each dataset
- R/Bioconductor environment with SingleCellExperiment and corral installed

## Outputs

- SingleCellExperiment object containing the count assay and cell labels
- Feature-downselected SingleCellExperiment (e.g., 1500 features) ready for corral or glmPCA
- Per-dataset prepared input matrices for downstream timing or biplot analyses (e.g., runtime_comparison.csv inputs)

## How to apply

Import the count matrix into R/Bioconductor as a SingleCellExperiment, since corral 'interfaces directly with Bioconductor classes (including SingleCellExperiment)'. Standardize dimensionality by down-selecting each dataset to 1500 features when benchmarking (e.g., the runtime comparison of CA vs glmPCA used ten benchmark datasets down-selected to 1500 features). Keep original-publication cell labels attached so downstream ARI-based clustering evaluation and biplot interpretation remain possible. Install corral with BiocManager (dev version via devtools::install_github('laurenhsu1/corral')) and pass the SCE directly to the corral function, which computes the embedding by SVD of the Pearson chi-squared residual matrix without log-transformation of counts.

## Related tools

- **corral** (Consumes the loaded SingleCellExperiment directly for CA dimension reduction; installed via BiocManager or from its GitHub repo) — github.com/laurenhsu1/corral
- **Bioconductor** (Framework providing the SingleCellExperiment class that corral interfaces with directly)
- **R** (Language environment in which the count data is loaded and prepared)

## Examples

```
BiocManager::install("laurenhsu1/corral", dependencies = TRUE); sce <- SingleCellExperiment(assays=list(counts=cnt_matrix)); sce <- corral(sce)
```

## Evaluation signals

- The loaded object is a valid SingleCellExperiment with the count assay and original-publication cell labels intact
- Feature down-selection yields exactly 1500 features per dataset when standardizing for benchmarking
- Downstream corral runs succeed on the object without a log-transformation step being applied to counts
- Reproduced timings on the prepared matrices match the paper's reported behavior (CA under a minute; glmPCA over an hour for ~22,000-cell, 1500-feature inputs)
- Biplot validation: high-weight labeled features on Zhengmix8eq include GNLY, TYROBP, LYZ, S100A8, CD74, and HLA-DRA

## Limitations

- No accession IDs or links for the SCMixology and Zhengmix benchmarking datasets are provided in the discussion, so acquiring the raw data may require external sources
- Ground-truth labels come from original dataset publications and were not independently validated
- No tool or package versions are reported for corral or its dependencies, so exact reproducibility of the loading/preparation environment is not guaranteed

## Evidence

- [intro] We implemented standard and adapted CA for scRNAseq in corral, an R/Bioconductor package that interfaces directly with Bioconductor classes (including SingleCellExperiment): "We implemented standard and adapted CA for scRNAseq in corral, an R/Bioconductor package that interfaces directly with Bioconductor classes (including SingleCellExperiment)"
- [intro] results in purified PBMCs (Zhengmix4eq benchmarking dataset): "results in purified PBMCs (Zhengmix4eq benchmarking dataset)"
- [readme] Or it can be installed with `BiocManager::install("laurenhsu1/corral", dependencies = TRUE)`: "Or it can be installed with `BiocManager::install("laurenhsu1/corral", dependencies = TRUE)`"
- [abstract] CA is based on decomposition of a chi-squared residual matrix, avoiding distortive log-transformation: "CA is based on decomposition of a chi-squared residual matrix, avoiding distortive log-transformation"
- [discussion] Except for SCMixology and Zhengmix (both comprising well-defined cell clusters and by design simpler than data from complex tissues): "Except for SCMixology and Zhengmix (both comprising well-defined cell clusters and by design simpler than data from complex tissues)"
