---
name: reference-table-result-reproduction
description: Use when reproducing or validating published gene-marker panels derived
  from COMET on an SCA latent-space frequency matrix, where a reference table of expected
  panels and COMETsc/TP/TN statistics is available for comparison.
when_to_use_negative:
- Input is a raw UMI counts table or FASTQ rather than an SCA-derived latent-space
  frequency matrix — run the SCA encoding and clustering first
- The goal is de novo marker discovery without a published reference table to validate
  against — this skill is specifically a reference-result reproduction
- Clusters other than those in the reference table, or combinations restricted to
  more than 4 genes, since the reference panels and statistics were defined under
  the ≤4-gene constraint
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: COMET (cometsc)
  role: Ranks candidate gene combinations by the COMETsc statistic to define the optimal
    ≤4-gene marker panel per cluster
- name: rCASC
  role: Framework embedding cometsc and the SCA modules used to generate the latent-space
    frequency matrix
  repo: github.com/kendomaniac/rCASC
- name: SCA (Sparsely-Connected Autoencoder)
  role: Produces the TF-based encoded-space frequency matrix for setA clusters 1 and
    2 that COMET consumes
  repo: github.com/kendomaniac/SCAtutorial
- name: Docker
  role: Runs the rCASC/SCA workflow containers (docker.io/repbioinfo) for reproducibility
- name: R
  role: Environment in which rCASC, devtools-based installation, and the COMET workflow
    are executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41540-020-00162-6
    title: Sparsely-connected autoencoder (SCA) for single cell RNAseq data mining
  source_content_hash: sha256:8edc1eaafcc5784b2fe3eab1e7d7325e6b6229f2260bd46490501f0908d7b7cb
schema_version: 0.2.0
metadata:
  license_tier: open
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
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# reference-table-result-reproduction

## Summary

Reproduces a published marker-panel result by re-running COMET (rCASC cometsc) on the TF-based SCA latent-space frequency matrix for setA clusters 1 and 2 and validating the top panels and statistics against the paper's Table 1 reference values (COMETsc 1.45E-49 / 1.19E-46, TP 0.589, TN 0.997).

## When to use

Use when reproducing or validating published gene-marker panels derived from COMET on an SCA latent-space frequency matrix, where a reference table of expected panels and COMETsc/TP/TN statistics is available for comparison.

## When NOT to use

- Input is a raw UMI counts table or FASTQ rather than an SCA-derived latent-space frequency matrix — run the SCA encoding and clustering first
- The goal is de novo marker discovery without a published reference table to validate against — this skill is specifically a reference-result reproduction
- Clusters other than those in the reference table, or combinations restricted to more than 4 genes, since the reference panels and statistics were defined under the ≤4-gene constraint

## Inputs

- TF-based SCA latent-space frequency matrix for the setA blood-cell dataset
- Clustering output assigning cells of setA to clusters 1 and 2
- Reference Table 1 entries: expected top panels with COMETsc, TP, and TN values

## Outputs

- Top gene-marker panel per cluster (cluster 1: PAX5, NFAT5, RFXANK, CHD4; cluster 2: CEBPA, KHSRP negation, CEBPB, CREBBP)
- COMETsc statistics per top panel (1.45E-49 for cluster 1, 1.19E-46 for cluster 2)
- True-positive and true-negative rates per panel (cluster 1: TP 0.589, TN 0.997)
- comet_setA_panels_table1.csv

## How to apply

Load the TF-based SCA latent-space frequency matrix for the blood-cell dataset setA together with the clustering output assigning cells to clusters 1 and 2. Run the rCASC cometsc (COMET) function separately on each cluster, restricting candidate feature combinations to at most 4 genes; expect negated features in top panels (e.g. KHSRP negation in cluster 2, POU2F2 negation in cluster 5 of the SSCA signature). Rank combinations by the COMETsc statistic and compute true-positive (TP) and true-negative (TN) rates for each top panel. The reproduction is accepted only when panels and statistics match the Table 1 entries: cluster 1 (B cells) = PAX5, NFAT5, RFXANK, CHD4 with COMETsc 1.45E-49, TP 0.589, TN 0.997; cluster 2 (monocytes) = CEBPA, KHSRP negation, CEBPB, CREBBP with COMETsc 1.19E-46. Write the reproduced panels and statistics to comet_setA_panels_table1.csv.

## Related tools

- **COMET (cometsc)** (Ranks candidate gene combinations by the COMETsc statistic to define the optimal ≤4-gene marker panel per cluster)
- **rCASC** (Framework embedding cometsc and the SCA modules used to generate the latent-space frequency matrix) — github.com/kendomaniac/rCASC
- **SCA (Sparsely-Connected Autoencoder)** (Produces the TF-based encoded-space frequency matrix for setA clusters 1 and 2 that COMET consumes) — github.com/kendomaniac/SCAtutorial
- **Docker** (Runs the rCASC/SCA workflow containers (docker.io/repbioinfo) for reproducibility)
- **R** (Environment in which rCASC, devtools-based installation, and the COMET workflow are executed)

## Examples

```
install.packages("devtools"); library(devtools); install_github("kendomaniac/rCASC"); # then run rCASC::cometsc() on the TF-SCA frequency matrix per cluster and write comet_setA_panels_table1.csv
```

## Evaluation signals

- Reproduced cluster 1 top panel equals {PAX5, NFAT5, RFXANK, CHD4} with COMETsc 1.45E-49
- Reproduced cluster 2 top panel equals {CEBPA, KHSRP negation, CEBPB, CREBBP} with COMETsc 1.19E-46
- Cluster 1 true-positive rate is 0.589 and true-negative rate is 0.997
- Feature combinations restricted to at most 4 genes in every cometsc run
- All reproduced panels and statistics are written to comet_setA_panels_table1.csv

## Limitations

- Transcription factors are lowly expressed genes and can be notably affected by dropout in scRNA-seq, which may perturb the frequency matrix and COMETsc rankings
- scRNAseq is very noisy and produces more zeros than expected, with greater bias for poorly expressed genes
- The reference values are dataset- and clustering-specific to setA clusters 1 and 2; they will not transfer to other datasets, cluster definitions, or larger gene-combination sizes

## Evidence

- [discussion] top ranked transcription factors, derived from SCA encoded space for cluster 1 and 2 of the blood-cell based dataset (setA), which were shown to be key elements for B cells and monocytes.: "top ranked transcription factors, derived from SCA encoded space for cluster 1 and 2 of the blood-cell based dataset (setA), which were shown to be key elements for B cells and monocytes."
- [abstract] SCA analysis is implemented as module in rCASC framework and it is supported by a GUI: "SCA analysis is implemented as module in rCASC framework and it is supported by a GUI"
- [discussion] The SSCA signature for cluster 5 (naïve T cell) refined the cluster definition, since POU2F2 negation fits well with naïve T cell: "the SSCA signature for cluster 5 (naïve T cell) refined the definition of the cluster, since POU2F2 negation fits well with naïve T cell."
- [readme] SCA analysis was added as extention of rCASC.: [evidence span withheld — anchor status: not_found]
