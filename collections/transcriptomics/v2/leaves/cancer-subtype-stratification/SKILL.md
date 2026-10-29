---
name: cancer-subtype-stratification
description: Use when you have a bulk tumor cohort expression matrix (e.g., TCGA)
  and want unsupervised patient stratification into subpopulations without pre-selected
  gene subsets, or when studying intra-cohort heterogeneity beyond single-cell applications.
when_to_use_negative:
- Input is single-cell RNA-seq data from heterogeneous cell populations — use SIMLR's
  standard single-cell workflow rather than the patient-stratification framing
- You need supervised subtype classification using known labels — SIMLR is unsupervised
  and performs de novo discovery, not label-driven prediction
- You need to rank genes/features for the learned similarities — use SIMLR_Feature_Ranking
  instead of the stratification workflow
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
tools:
- name: SIMLR
  role: Learns the multi-kernel patient-to-patient similarity matrix from the cohort
    expression data for clustering and 2-D visualization of the tumor stratification
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: R
  role: Implementation environment in which SIMLR is installed from GitHub/Bioconductor
    and the TCGA expression matrix is processed
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: SIMLR_Large_Scale
  role: Large-scale variant of SIMLR available for handling large cohort datasets
  repo: https://github.com/BatzoglouLabSU/SIMLR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/pmic.201700232
    title: 'SIMLR: A Tool for Large‐Scale Genomic Analyses by Multi‐Kernel Learning'
  source_content_hash: sha256:04d26761578bebbcb55eaf946ab2f0bc7b6af47cdf8b0640c842fb281894dd8c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/BatzoglouLabSU/SIMLR
  tools:
  - R
  - SIMLR
  - SIMLR_Large_Scale
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cancer-subtype-stratification

## Summary

Apply SIMLR's multi-kernel similarity learning to bulk cancer cohort expression data (e.g., TCGA diffuse lower-grade glioma) to learn a patient-to-patient similarity matrix, estimate the number of patient populations, and stratify tumors by clustering and 2-D visualization of the learned structure.

## When to use

Use when you have a bulk tumor cohort expression matrix (e.g., TCGA) and want unsupervised patient stratification into subpopulations without pre-selected gene subsets, or when studying intra-cohort heterogeneity beyond single-cell applications.

## When NOT to use

- Input is single-cell RNA-seq data from heterogeneous cell populations — use SIMLR's standard single-cell workflow rather than the patient-stratification framing
- You need supervised subtype classification using known labels — SIMLR is unsupervised and performs de novo discovery, not label-driven prediction
- You need to rank genes/features for the learned similarities — use SIMLR_Feature_Ranking instead of the stratification workflow

## Inputs

- Bulk cancer cohort gene expression matrix (e.g., TCGA diffuse lower-grade glioma expression matrix, reference [12]) loaded into R as the input data object

## Outputs

- Multi-kernel patient-to-patient similarity matrix
- Estimated number of patient populations (from SIMLR_Estimate_Number_of_Clusters.R)
- Patient cluster assignments (tumor stratification)
- 2-D visualization of the learned patient similarity structure (e.g., simlr_tcga_lgg_2d.png)
- Results table combining cluster assignments, cluster count, and similarity matrix (e.g., simlr_tcga_lgg_results.csv)

## How to apply

Load the cohort expression matrix (e.g., the TCGA diffuse lower-grade glioma data, reference [12]) into R as the input data object, and install SIMLR from GitHub with install_github('BatzoglouLabSU/SIMLR', ref='master'), compiling the external C code with 'R CMD SHLIB -c projsplx_R.c'. Run SIMLR on the expression matrix to learn the multi-kernel patient-to-patient similarity matrix, relying on graph diffusion to strengthen weak similarities arising from noise or dropout-like events. Estimate the number of patient populations using the SIMLR_Estimate_Number_of_Clusters.R script, then cluster patients on the learned similarity and produce a 2-D visualization of the similarity structure to interpret the stratification. Write the cluster assignments, estimated number of clusters, and similarity matrix to a results file (e.g., simlr_tcga_lgg_results.csv) and the 2-D plot to an image (e.g., simlr_tcga_lgg_2d.png). Validate by checking that the estimated number of populations and the resulting stratification match the complementary example reported in the Supplementary Materials.

## Related tools

- **SIMLR** (Learns the multi-kernel patient-to-patient similarity matrix from the cohort expression data for clustering and 2-D visualization of the tumor stratification) — https://github.com/BatzoglouLabSU/SIMLR
- **R** (Implementation environment in which SIMLR is installed from GitHub/Bioconductor and the TCGA expression matrix is processed) — https://github.com/BatzoglouLabSU/SIMLR
- **SIMLR_Large_Scale** (Large-scale variant of SIMLR available for handling large cohort datasets) — https://github.com/BatzoglouLabSU/SIMLR

## Examples

```
library("devtools"); install_github("BatzoglouLabSU/SIMLR", ref = 'master'); library("SIMLR")
```

## Evaluation signals

- The estimated number of patient populations matches the complementary example reported in the Supplementary Materials (output_matches_reference against the SI-reported stratification)
- The resulting tumor stratification is consistent with the SI-reported stratification for the diffuse lower-grade glioma data
- The learned patient-to-patient similarity matrix is produced and strengthened by graph diffusion (weak similarities from noise are improved)
- A 2-D visualization of the similarity structure is generated and is interpretable with respect to the patient clusters
- Cluster assignments, estimated cluster count, and similarity matrix are written to simlr_tcga_lgg_results.csv and the plot to simlr_tcga_lgg_2d.png without errors

## Limitations

- Example datasets in the repository are provided purely as examples after pre-processing and should not be used in place of the ones provided in the respective publications
- SIMLR is unsupervised and does not pre-select gene subsets of known function, so stratification is de novo and must be validated against external references
- If issues arise compiling the external C code (projsplx_R.c), try removing the pre-compiled files projsplx_R.o and projsplx_R.so

## Evidence

- [supplementary] SIMLR applied to diffuse lower-grade glioma stratification: [evidence span withheld — anchor status: not_found]
- [methods] Multi-kernel similarity learning rationale: "it learns a distance metric that best fits the structure of the data via combining multiple kernels"
- [methods] Graph diffusion strengthens weak similarities: "by employing graph diffusion, which improves weak similarity measures that are likely to result from noise or dropout events"
- [readme] Unsupervised de novo discovery: "in contrast to some previous analyses that pre-select gene subsets of known function, *SIMLR* is unsupervised, thus allowing de novo discovery from the data"
- [readme] Installation and C compilation: "install_github("BatzoglouLabSU/SIMLR", ref = 'master')"
