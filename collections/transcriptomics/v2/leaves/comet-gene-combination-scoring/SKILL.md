---
name: comet-gene-combination-scoring
description: Use when clustered scRNAseq cells need a minimal, high-specificity marker
  panel scored from an SCA latent-space (frequency) matrix, and COMETsc statistics
  with TP/TN rates must be reproduced or compared.
when_to_use_negative:
- Cells have not yet been clustered — COMET scores gene combinations per cluster and
  requires an upstream clustering analysis of scRNAseq data
- Input is a raw UMI count table rather than the SCA latent-space (frequency) matrix
  encoding TF/functional features
- The goal is to quantify miRNAs alongside mRNAs — it is currently not possible to
  quantify miRNAs together with mRNAs at single-cell level, so TF-miRNA circuits cannot
  be directly measured
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: COMET (cometsc)
  role: Scores candidate gene combinations from the SCA latent-space frequency matrix
    to identify the optimal marker panel per cluster
- name: rCASC
  role: Framework hosting the cometsc function and the SCA module used to produce
    the latent-space input; installed via install_github("kendomaniac/rCASC")
  repo: https://github.com/kendomaniac/rCASC
- name: SCA (Sparsely-Connected Autoencoder)
  role: Generates the TF-based encoded-space frequency matrix that COMET is applied
    to
  repo: https://github.com/kendomaniac/SCAtutorial
- name: R
  role: Environment in which the rCASC/cometsc workflow is executed
  repo: https://cran.r-project.org/
- name: Docker
  role: Container runtime required to run the rCASC SCA/cometsc modules reproducibly
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:1bf8531ded5c60776dd5129ebb83911166d20b966b0ca88c54d9b199d8b0038e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kendomaniac/rCASC
  tools:
  - COMET (cometsc)
  - Docker
  - R
  - SCA (Sparsely-Connected Autoencoder)
  - rCASC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# comet-gene-combination-scoring

## Summary

This skill reproduces COMET (rCASC cometsc) gene-combination marker-panel discovery on the TF-based SCA latent-space frequency matrix of a clustered single-cell dataset, ranking candidate gene panels (≤4 genes, including negated features) per cluster by the COMETsc statistic and validating against published TP/TN benchmarks (e.g., Table 1 of the SCA paper).

## When to use

Use when clustered scRNAseq cells need a minimal, high-specificity marker panel scored from an SCA latent-space (frequency) matrix, and COMETsc statistics with TP/TN rates must be reproduced or compared.

## When NOT to use

- Cells have not yet been clustered — COMET scores gene combinations per cluster and requires an upstream clustering analysis of scRNAseq data
- Input is a raw UMI count table rather than the SCA latent-space (frequency) matrix encoding TF/functional features
- The goal is to quantify miRNAs alongside mRNAs — it is currently not possible to quantify miRNAs together with mRNAs at single-cell level, so TF-miRNA circuits cannot be directly measured

## Inputs

- TF-based SCA latent-space frequency matrix for the setA blood-cell dataset
- Clustering output assigning single cells to clusters (e.g., clusters 1 and 2)
- Cluster labels / cell annotations for TP/TN evaluation

## Outputs

- Top-ranked gene marker panel per cluster, including negated features (cluster 1: PAX5, NFAT5, RFXANK, CHD4; cluster 2: CEBPA, KHSRP negation, CEBPB, CREBBP)
- COMETsc statistic per top panel (1.45E-49 for cluster 1; 1.19E-46 for cluster 2)
- True-positive (TP) and true-negative (TN) rates per panel (cluster 1: TP 0.589, TN 0.997)
- comet_setA_panels_table1.csv results table

## How to apply

Load the TF-based SCA latent-space frequency matrix together with the clustering output that assigns cells to the target clusters (e.g., clusters 1 and 2 of the setA blood-cell dataset). Run the rCASC cometsc function (COMET) separately on each cluster, restricting feature combinations to at most 4 genes. Rank candidate gene combinations by the COMETsc statistic, recording the top panel per cluster including negated features such as KHSRP negation in cluster 2 or POU2F2 negation in cluster 5. Compute the true-positive (TP) and true-negative (TN) rates for each top panel. Accept the run only when the reproduced panels and COMETsc/TP/TN values match the reference entries (cluster 1: PAX5, NFAT5, RFXANK, CHD4 with COMETsc 1.45E-49, TP 0.589, TN 0.997; cluster 2: CEBPA, KHSRP negation, CEBPB, CREBBP with COMETsc 1.19E-46), and write results to comet_setA_panels_table1.csv.

## Related tools

- **COMET (cometsc)** (Scores candidate gene combinations from the SCA latent-space frequency matrix to identify the optimal marker panel per cluster)
- **rCASC** (Framework hosting the cometsc function and the SCA module used to produce the latent-space input; installed via install_github("kendomaniac/rCASC")) — https://github.com/kendomaniac/rCASC
- **SCA (Sparsely-Connected Autoencoder)** (Generates the TF-based encoded-space frequency matrix that COMET is applied to) — https://github.com/kendomaniac/SCAtutorial
- **R** (Environment in which the rCASC/cometsc workflow is executed) — https://cran.r-project.org/
- **Docker** (Container runtime required to run the rCASC SCA/cometsc modules reproducibly)

## Examples

```
install.packages("devtools"); library(devtools); install_github("kendomaniac/rCASC")
```

## Evaluation signals

- Reproduced top panel for cluster 1 matches exactly the four genes PAX5, NFAT5, RFXANK, CHD4
- Reproduced COMETsc statistics match Table 1: 1.45E-49 (cluster 1) and 1.19E-46 (cluster 2)
- Cluster 1 TP rate = 0.589 and TN rate = 0.997 as in Table 1
- Cluster 2 top panel includes the KHSRP negation feature alongside CEBPA, CEBPB, and CREBBP
- Results are written to comet_setA_panels_table1.csv in the expected schema (panel genes, COMETsc, TP, TN per cluster)

## Limitations

- Transcription factors and kinases are encoded by lowly expressed genes and can be notably affected by dropout in scRNAseq data, which is very noisy
- Feature combinations are restricted to at most 4 genes, so larger panels are never explored by this workflow
- COMET results depend on the upstream clustering and SCA encoding; a different clustering or latent space (e.g., non-TF features) changes which combinations rank best

## Evidence

- [results] Task card finding on the cluster 1 marker panel: [evidence span withheld — anchor status: not_found]
- [discussion] Top ranked TFs from SCA encoded space for setA clusters: "top ranked transcription factors, derived from SCA encoded space for cluster 1 and 2 of the blood-cell based dataset (setA), which were shown to be key elements for B cells and monocytes."
- [discussion] Negated feature refining a cluster signature: "the SSCA signature for cluster 5 (naïve T cell) refined the definition of the cluster, since POU2F2 negation fits well with naïve T cell."
- [readme] SCA analysis as rCASC extension: [evidence span withheld — anchor status: not_found]
- [readme] Docker-embedded reproducible tooling: [evidence span withheld — anchor status: not_found]
