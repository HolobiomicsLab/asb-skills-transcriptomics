---
name: single-cell-variant-allele-frequency-inference
description: Use when you have per-cell mutation VAF and coverage matrices from scRNA-seq
  variant calling that suffer from low coverage and allelic bias, and the goal is
  inferring corrected genotypes/VAFs or cell lineages.
when_to_use_negative:
- Input is bulk sequencing data without per-cell mutation/coverage matrices — the
  Bayesian model operates on single-cell variant calling where expressional drop-outs
  occur
- You only have transcriptome-level clustering goals with no genetic markers (variants/VAFs)
  — SClineager performs genetics-based, not expression-based, lineage tracing
- Input is unprocessed 10x fastq data — reads must first be split per cell (scSplitter)
  and variant-called (QBRC Somatic Pipeline) before SClineager can be applied
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: SClineager
  role: Core R package whose Bayesian hierarchical model (sclineager_internal/run_sclineager
    MCMC sampling) infers corrected genotypes/VAFs and covariance from mutations and
    coverage inputs
  repo: https://github.com/inmybrain/SClineager
- name: Rcpp
  role: Dependency (>= 1.0.2) used by SClineager for the underlying sampling implementation
- name: MCMCpack
  role: Dependency (ver 1.4.5) providing MCMC sampling routines used in the Bayesian
    inference
- name: gplots
  role: Dependency (ver 3.0.3) used to generate heatmaps for raw and fitted VAFs
- name: devtools
  role: Used to install SClineager from GitHub with the subdir argument specified
  repo: https://github.com/inmybrain/SClineager
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.celrep.2020.108589
    title: Overcoming Expressional Drop-outs in Lineage Reconstruction from Single-Cell
      RNA-Sequencing Data
  source_content_hash: sha256:4f8cc053133d460ee1f20b04161886ea59558b3fc722e625ea4c5e23dd36f0e6
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
  - MCMCpack
  - R
  - Rcpp
  - SClineager
  - devtools
  - gplots
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# single-cell-variant-allele-frequency-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies the SClineager Bayesian hierarchical model (R package, MCMC sampling via sclineager_internal/run_sclineager) to infer true cell genotypes and variant allele frequencies (VAFs) from scRNA-seq-derived variant calling, explicitly modeling both expressional drop-outs: low coverage and allelic bias. This enables genetics-based lineage tracing of single cells from existing single-cell sequencing data.

## When to use

Use when you have per-cell mutation VAF and coverage matrices from scRNA-seq variant calling that suffer from low coverage and allelic bias, and the goal is inferring corrected genotypes/VAFs or cell lineages.

## When NOT to use

- Input is bulk sequencing data without per-cell mutation/coverage matrices — the Bayesian model operates on single-cell variant calling where expressional drop-outs occur
- You only have transcriptome-level clustering goals with no genetic markers (variants/VAFs) — SClineager performs genetics-based, not expression-based, lineage tracing
- Input is unprocessed 10x fastq data — reads must first be split per cell (scSplitter) and variant-called (QBRC Somatic Pipeline) before SClineager can be applied

## Inputs

- mutations matrix (raw variants x cells VAF matrix, e.g. res_data$mutations_mat_obs)
- coverage matrix of the same cells and variants (e.g. res_data$coverage_mat_obs)
- per-cell mutation folders with coverage.txt and germline_mutations_hg38.txt from single-cell variant calling
- toy_data.RData example dataset shipped with the SClineager package

## Outputs

- genotype_mat: inferred VAF matrix (corrected cell genotypes)
- genotype_mat_orig: raw VAF matrix
- sigma: inferred covariance matrix of variants
- genotype_mat_all and sigma_all: sampled parameters at each MCMC iteration
- heatmaps of raw vs fitted VAFs

## How to apply

First produce variant inputs: split 10x Genomics fastqs per cell with scSplitter, call variants with the QBRC Somatic Pipeline (tumor-only mode), and preprocess per-sample mutation folders with read_sclineager using coverage_cutoff=3, coverage_percentage=0.2, cell_percentage=0.2, artefact_percentage=0.03; optionally inspect data with explore_sclineager. Run the Bayesian MCMC per sample independently via run_sclineager (or directly sclineager_internal with mutations_mat and coverage_mat inputs, e.g. max_iter=100, vaf_offset=0.01, dfreedom set to the number of variants or 100, psi as a diagonal covariance, and optionally skip_common=TRUE with categories like chrM and synonymous SNV masked and HLA-A/B/C masked as mask_genes). The rationale is that related cells borrow information so monoallelic expression or sampling artifacts do not masquerade as genotype differences. Extract genotype_mat (inferred VAFs) and sigma (inferred covariance matrix) from the fit and visualize raw vs fitted VAFs as heatmaps for quality control.

## Related tools

- **SClineager** (Core R package whose Bayesian hierarchical model (sclineager_internal/run_sclineager MCMC sampling) infers corrected genotypes/VAFs and covariance from mutations and coverage inputs) — https://github.com/inmybrain/SClineager
- **Rcpp** (Dependency (>= 1.0.2) used by SClineager for the underlying sampling implementation)
- **MCMCpack** (Dependency (ver 1.4.5) providing MCMC sampling routines used in the Bayesian inference)
- **gplots** (Dependency (ver 3.0.3) used to generate heatmaps for raw and fitted VAFs)
- **devtools** (Used to install SClineager from GitHub with the subdir argument specified) — https://github.com/inmybrain/SClineager

## Examples

```
load("./data/toy_data.RData"); fit_scl <- sclineager_internal(res_data$mutations_mat_obs, res_data$coverage_mat_obs, max_iter=100, vaf_offset=0.01, save=FALSE)
```

## Evaluation signals

- The fit object contains genotype_mat and sigma; genotype_mat has the same cells x variants shape as the raw mutations matrix, and sigma is a covariance matrix matching the number of variants
- Heatmaps of raw vs fitted VAFs show fitted values pulled toward internally consistent genotype states (drop-outs corrected) rather than following the sparse raw VAF pattern
- Bayesian sampling on the package toy dataset completes in about 5-6 seconds; substantially longer runs suggest mis-set parameters or oversized inputs
- Exploratory filtering parameters (coverage_cutoff=3, coverage_percentage=0.2, cell_percentage=0.2, artefact_percentage=0.03) produce cleaned.RData with mutations_mat, coverage_mat, runinfo, and annotation fields before inference
- Results are reproducible per sample: run_sclineager must be executed independently for each sample folder and returns results.RData with the documented attributes

## Limitations

- The method is designed for scRNA-seq variant calling, which suffers low coverage on many positions in many cells; very sparse coverage can still limit inference
- Allelic bias from true monoallelic expression or sampling bias is modeled, but masked genes (e.g. HLA-A, HLA-B, HLA-C) and common artifact categories (chrM, synonymous SNV, UTR, substitution classes with skip_common) should be excluded to avoid artifacts
- Analysis has to be performed for each sample independently; the variant calls feeding SClineager come from an external pipeline (QBRC Somatic Pipeline), so errors propagate from upstream calling
- Preprocessing with read_sclineager takes about 30 seconds per batch of mutation files, and the pipeline's variant-calling repository (QBRC-Somatic-Pipeline) was noted as temporarily closed

## Evidence

- [intro] Two inherent drop-out issues motivate the model: [evidence span withheld — anchor status: not_found]
