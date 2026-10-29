---
name: single-cell-genotype-matrix-processing
description: Use when a detected variant/VAF matrix from scRNA-seq variant calling
  contains germline-annotated SNPs (e.g., ESP6500 frequency annotations) that must
  be removed before lineage reconstruction with SClineager.
when_to_use_negative:
- The variant matrix has no population-frequency annotations available, so common
  SNPs cannot be identified against ESP6500
- You are running the pre-modeling read/coverage filters in read_sclineager (coverage_cutoff,
  coverage_percentage, cell_percentage, artefact_percentage) rather than removing
  germline common SNPs before MCMC estimation
- The data are raw FASTQ files needing scSplitter demultiplexing or QBRC Somatic Pipeline
  variant calling — filtering operates downstream on the called variant matrix
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: SClineager
  role: R package whose run_sclineager/sclineager_internal workflow exposes the skip_common
    option that drops variants with ESP6500 frequency > 0.01 before Bayesian sampling
  repo: https://github.com/inmybrain/SClineager
- name: R
  role: Environment in which the toy_data.RData matrix is loaded, the ESP6500 frequency
    filter is applied to variant columns, and filtered CSV outputs are written
- name: devtools
  role: Used to install SClineager from GitHub with devtools::install_github("inmybrain/SClineager",
    subdir = "SClineager")
  repo: https://github.com/r-lib/remotes
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.celrep.2020.108589
    title: Overcoming Expressional Drop-outs in Lineage Reconstruction from Single-Cell
      RNA-Sequencing Data
  source_content_hash: sha256:4b8e210617979d9a109e26019640876ed0962e841a0e970cf6702c08b516251b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/inmybrain/SClineager
  repo_url: https://github.com/inmybrain/SClineager
  tools:
  - R
  - SClineager
  - devtools
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# single-cell-genotype-matrix-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pre-filters a single-cell variant/VAF genotype matrix by dropping variants annotated as common germline SNPs — specifically those with an ESP6500 population frequency > 0.01 — so that SClineager's Bayesian lineage model estimates from variants more likely to be somatically acquired.

## When to use

Use when a detected variant/VAF matrix from scRNA-seq variant calling contains germline-annotated SNPs (e.g., ESP6500 frequency annotations) that must be removed before lineage reconstruction with SClineager.

## When NOT to use

- The variant matrix has no population-frequency annotations available, so common SNPs cannot be identified against ESP6500
- You are running the pre-modeling read/coverage filters in read_sclineager (coverage_cutoff, coverage_percentage, cell_percentage, artefact_percentage) rather than removing germline common SNPs before MCMC estimation
- The data are raw FASTQ files needing scSplitter demultiplexing or QBRC Somatic Pipeline variant calling — filtering operates downstream on the called variant matrix

## Inputs

- Variant/VAF matrix of all simulated or sequenced cells (e.g., res_data$mutations_mat_obs from toy_data.RData, or mutations_mat from cleaned.RData)
- Coverage matrix of the same cells and variants (coverage_mat)
- Variant annotations including ESP6500 population frequency

## Outputs

- Filtered variant/VAF matrix with common-SNP columns removed and all cell rows preserved in original order (e.g., filtered_mutations_mat.csv)
- Table of removed common SNPs with their ESP6500 frequencies (e.g., removed_common_snps.csv)

## How to apply

Load the variant matrix (e.g., from toy_data.RData in the SClineager repository data folder, res_data$mutations_mat_obs, or the mutations_mat/coverage_mat produced by read_sclineager) and cross-reference variant identifiers against ESP6500 population-frequency annotations. Implement the skip_common filter with the common-SNP cutoff at ESP6500 frequency > 0.01, dropping every variant above this threshold as a presumed common germline SNP; in the SClineager workflow this is passed as skip_common <- T to run_sclineager, which then runs the MCMC/Bayesian sampling on the retained variants. Retain all cells in their original row order while subsetting variant columns, preserving coverage information for the same cells and variants. The rationale is that single-cell variant calling suffers from low coverage and allelic bias, so removing likely-germline common SNPs focuses the Bayesian hierarchical model on putatively somatic markers. Analysis via run_sclineager must be performed for each sample independently, with prior parameters (e.g., dfreedom <- 100, control list a=0.5, b=0.5, c=1, d=20, e=0.4) set alongside the filter.

## Related tools

- **SClineager** (R package whose run_sclineager/sclineager_internal workflow exposes the skip_common option that drops variants with ESP6500 frequency > 0.01 before Bayesian sampling) — https://github.com/inmybrain/SClineager
- **R** (Environment in which the toy_data.RData matrix is loaded, the ESP6500 frequency filter is applied to variant columns, and filtered CSV outputs are written)
- **devtools** (Used to install SClineager from GitHub with devtools::install_github("inmybrain/SClineager", subdir = "SClineager")) — https://github.com/r-lib/remotes

## Examples

```
skip_common <- T
```

## Evaluation signals

- Every variant column retained in the filtered matrix has ESP6500 population frequency <= 0.01
- The filtered matrix preserves the original number of cell rows (no cells dropped) in their original order
- removed_common_snps.csv accounts exactly for the difference in variant columns between the raw and filtered matrices
- SClineager model estimation (sclineager_internal MCMC) runs successfully on the filtered matrix and returns genotype_mat, genotype_mat_orig, sigma, and runinfo attributes
- The coverage matrix for downstream analysis contains the same cells and the same retained variants as the filtered VAF matrix

## Limitations

- skip_common is an optional pre-model filter; if disabled (skip_common <- F) common germline SNPs remain in the matrix and may confound lineage inference
- The filter relies on the quality of ESP6500 annotations; variants lacking frequency annotations are not classified by this rule
- SClineager's run_sclineager analysis must be performed for each sample independently, so the filter must be applied per sample

## Evidence
