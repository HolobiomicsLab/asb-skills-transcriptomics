---
name: transcription-factor-panel-verification
description: Use when a TF-based SCA latent-space frequency matrix plus cluster assignments
  (e.g., setA clusters 1 and 2) are available and the goal is to extract and verify
  an optimal gene marker panel (≤4 genes, negations allowed) per cluster with COMET.
when_to_use_negative:
- Input is a raw UMI counts table (e.g., from cellrangerCount or indropCounts) rather
  than an SCA latent-space frequency matrix — run SCA first.
- Marker combinations larger than 4 genes are required — the workflow restricts feature
  combinations to at most 4 genes.
- The biological features of interest are miRNAs, since it is currently not possible
  to quantify miRNAs together with mRNAs at single-cell level, so TF-miRNA circuits
  cannot be directly measured.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: COMET (cometsc)
  role: Ranks gene/negated-feature combinations by the COMETsc statistic to define
    the optimal marker panel per cluster
- name: rCASC
  role: Framework providing the cometsc function and the SCA modules executed in Docker
    containers
  repo: https://github.com/kendomaniac/rCASC
- name: SCA (Sparsely-Connected Autoencoder)
  role: Produces the latent-space frequency matrix from TF targets whose top-ranked
    features feed the COMET panel search
  repo: https://github.com/kendomaniac/SCAtutorial
- name: Docker
  role: Encapsulates the rCASC/SCA workflow tools (docker.io/repbioinfo) for reproducible
    execution
- name: R
  role: Environment in which rCASC, devtools, and cometsc are installed and run
  repo: https://github.com/kendomaniac/rCASC
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:70274b87bcd1c41ffd939b6824f180de567f131bde9df5959b4c693d6fc93a29
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# transcription-factor-panel-verification

## Summary

Reproduces cluster-specific gene marker panels by running COMET (rCASC cometsc) on the frequency matrix of the SCA latent space built from TF targets, and validates the top panels against published COMETsc/TP/TN statistics (e.g., Table 1 of the SCA paper).

## When to use

Use when a TF-based SCA latent-space frequency matrix plus cluster assignments (e.g., setA clusters 1 and 2) are available and the goal is to extract and verify an optimal gene marker panel (≤4 genes, negations allowed) per cluster with COMET.

## When NOT to use

- Input is a raw UMI counts table (e.g., from cellrangerCount or indropCounts) rather than an SCA latent-space frequency matrix — run SCA first.
- Marker combinations larger than 4 genes are required — the workflow restricts feature combinations to at most 4 genes.
- The biological features of interest are miRNAs, since it is currently not possible to quantify miRNAs together with mRNAs at single-cell level, so TF-miRNA circuits cannot be directly measured.

## Inputs

- TF-based SCA latent-space frequency matrix for dataset setA
- Clustering output assigning cells to clusters (e.g., clusters 1 and 2 of setA)
- Reference table of expected marker panels and statistics (Table 1: COMETsc, TP, TN values)

## Outputs

- Top marker panel per cluster (cluster 1: PAX5, NFAT5, RFXANK, CHD4; cluster 2: CEBPA, KHSRP negation, CEBPB, CREBBP)
- COMETsc statistic, TP rate, and TN rate per top panel
- comet_setA_panels_table1.csv with reproduced panels and statistics

## How to apply

Load the TF-based SCA latent-space frequency matrix for the dataset (e.g., blood-cell setA) together with the clustering output assigning cells to the target clusters. Run the rCASC cometsc function (COMET) separately on each cluster, restricting feature combinations to at most 4 genes, and allow negated features (e.g., KHSRP negation in cluster 2, POU2F2 negation for naïve T cells). Rank candidate gene combinations by the COMETsc statistic and record the top panel per cluster, then compute true-positive (TP) and true-negative (TN) rates for each top panel. Accept the run only when the reproduced panels and their COMETsc/TP/TN values match the published reference (cluster 1: PAX5, NFAT5, RFXANK, CHD4 with COMETsc 1.45E-49, TP 0.589, TN 0.997; cluster 2: CEBPA, KHSRP negation, CEBPB, CREBBP with COMETsc 1.19E-46), and write results to comet_setA_panels_table1.csv.

## Related tools

- **COMET (cometsc)** (Ranks gene/negated-feature combinations by the COMETsc statistic to define the optimal marker panel per cluster)
- **rCASC** (Framework providing the cometsc function and the SCA modules executed in Docker containers) — https://github.com/kendomaniac/rCASC
- **SCA (Sparsely-Connected Autoencoder)** (Produces the latent-space frequency matrix from TF targets whose top-ranked features feed the COMET panel search) — https://github.com/kendomaniac/SCAtutorial
- **Docker** (Encapsulates the rCASC/SCA workflow tools (docker.io/repbioinfo) for reproducible execution)
- **R** (Environment in which rCASC, devtools, and cometsc are installed and run) — https://github.com/kendomaniac/rCASC

## Examples

```
library(rCASC); cometsc(cluster=cluster_file, scratch_folder="/data/scratch", "run.comet.sc", cluster_folder="/data/setA", n комбинаций=4)
```

## Evaluation signals

- Reproduced cluster 1 top panel equals PAX5, NFAT5, RFXANK, CHD4 with COMETsc 1.45E-49, TP 0.589, TN 0.997
- Reproduced cluster 2 top panel equals CEBPA, KHSRP negation, CEBPB, CREBBP with COMETsc 1.19E-46
- All candidate feature combinations restricted to at most 4 genes, with negated features correctly encoded (e.g., KHSRP negation in cluster 2)
- Results written to comet_setA_panels_table1.csv with panel members and statistics matching Table 1 entries
- COMET run separately per cluster using the TF-based SCA frequency matrix, not the raw count matrix

## Limitations

- Transcription factors and kinases are lowly expressed genes and can be notably affected by dropout, which perturbs the underlying frequency matrix.
- scRNAseq is very noisy and produces more zeros than expected, with greater bias for poorly expressed genes.
- Panels are cluster-specific: negated features (e.g., POU2F2 negation) refine cluster definitions, so panels are not transferable across cluster identities without re-running COMET.

## Evidence

- [results] COMET analysis of the TF-based SCA latent-space frequency matrix defined the optimal marker panel for cluster 1 (B cells) as four genes: PAX5, NFAT5, RFXANK, and CHD4.: [evidence span withheld — anchor status: not_found]
- [discussion] Top ranked transcription factors from SCA encoded space for clusters 1 and 2 of setA were key elements for B cells and monocytes: "top ranked transcription factors, derived from SCA encoded space for cluster 1 and 2 of the blood-cell based dataset (setA), which were shown to be key elements for B cells and monocytes."
- [discussion] SSCA signature for cluster 5 refined via POU2F2 negation: "the SSCA signature for cluster 5 (naïve T cell) refined the definition of the cluster, since POU2F2 negation fits well with naïve T cell."
- [abstract] SCA is implemented as a module in rCASC and supported by a GUI: "SCA analysis is implemented as module in rCASC framework and it is supported by a GUI"
- [readme] SCA tools embedded in Docker containers for reproducibility: [evidence span withheld — anchor status: not_found]
