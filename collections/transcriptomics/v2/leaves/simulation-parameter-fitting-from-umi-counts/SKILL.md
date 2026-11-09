---
name: simulation-parameter-fitting-from-umi-counts
description: Use when benchmarking normalization or differential-expression methods
  on scRNA-seq requires simulated datasets with known ground-truth logFC whose mean/variance/library-size
  structure matches real UMI count data.
when_to_use_negative:
- Analyzing real (non-simulated) scRNA-seq data without ground-truth logFC — there
  is no reference against which to compute regression bias and R2.
- Full-length (non-UMI) scRNA-seq platforms — the article only focused on UMI-based
  datasets and notes full-length platforms may need revalidation of nonlinear covariates.
- The goal is biological discovery or differential expression calling itself rather
  than quantifying method accuracy against a known effect-size ground truth.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Normalisr
  role: Provides the reference QC pipeline (qc_reads, qc_outlier), normalization functions
    (lcpm, normcov, compute_var, normvar), and DE calling (normalisr.de) applied to
    every simulated replicate, plus normalisr.parallel for scalable execution
  repo: github.com/lingfeiwang/normalisr
- name: SymSim
  role: Simulation model whose parameters are fitted to the empirical count distributions
    of QC-filtered Perturb-seq data to generate two-group DE datasets with ground-truth
    logFC
- name: Splatter
  role: Alternative simulation model fitted to the same empirical count distributions,
    providing variation in simulation models and their similarity to the real dataset
- name: edgeR
  role: Bulk DE method run on every simulated replicate as a competing method whose
    logFC estimates are compared for bias and variance
- name: sctransform
  role: Single-cell normalization method run as a competing normalization/DE approach
    on simulated replicates
- name: bayNorm
  role: Single-cell Bayesian normalization method compared on simulated replicates
- name: Sanity
  role: Single-cell normalization method compared on simulated replicates
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:172ae5c016f514440150e36b04321647c19cbd9611d415a8d86a8d603296dce7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Normalisr
  - Sanity
  - Splatter
  - SymSim
  - bayNorm
  - edgeR
  - sctransform
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# simulation-parameter-fitting-from-umi-counts

## Summary

Fits SymSim and Splatter simulation parameters to the empirical count distributions of real UMI scRNA-seq data (e.g., QC-filtered Perturb-seq) to generate two-group differential-expression datasets with known ground-truth logFC, enabling bias and variance (R²) benchmarking of normalization/DE methods.

## When to use

Use when benchmarking normalization or differential-expression methods on scRNA-seq requires simulated datasets with known ground-truth logFC whose mean/variance/library-size structure matches real UMI count data.

## When NOT to use

- Analyzing real (non-simulated) scRNA-seq data without ground-truth logFC — there is no reference against which to compute regression bias and R2.
- Full-length (non-UMI) scRNA-seq platforms — the article only focused on UMI-based datasets and notes full-length platforms may need revalidation of nonlinear covariates.
- The goal is biological discovery or differential expression calling itself rather than quantifying method accuracy against a known effect-size ground truth.

## Inputs

- Perturb-seq UMI read count matrix from GEO (e.g., GSE120861), as sparse mtx (Cell Ranger output) or tsv
- MARS-seq human melanoma scRNA-seq count matrix (optional second dataset for parameter fitting)
- SymSim and Splatter simulation software with parameters fitted to the empirical count distributions
- Competing DE/normalization methods: edgeR, sctransform, bayNorm, Sanity

## Outputs

- Fitted SymSim and Splatter parameter sets matching the real dataset's mean/variance/library-size structure
- Simulated two-group DE count matrices (minor-group sizes 519–3927 cells) with stored ground-truth logFC per gene per replicate
- Estimated logFC per gene per method per replicate from normalisr.de and competing methods
- Summary table of per-bin bias (regression coefficient) and R2 (variance) per method per expression bin per minor-group size
- Scatter/line plots reproducing Fig. 3a-b and S5

## How to apply

First QC the Perturb-seq UMI read count matrix downloaded from GEO using Normalisr's reference pipeline (normalisr.qc with qc_reads and qc_outlier): keep cells with at least 500 reads and 100 detected genes, and genes with non-zero reads in at least 50 cells and at least 2% of all cells. Then fit SymSim and Splatter parameters to the empirical count distributions of the QC-filtered data (optionally repeating on the MARS-seq melanoma dataset) so simulated data match the real mean/variance/library-size structure. Simulate two-group DE datasets at minor-group sizes spanning 519–3927 cells, storing the ground-truth logFC per gene for every replicate. Normalize each replicate with normalisr.norm (lcpm, normcov, compute_var, normvar) and run competing DE methods (edgeR, sctransform, bayNorm, Sanity) on every replicate, calling DE via normalisr.de to obtain estimated logFCs. Finally bin genes by expression level and regress estimated logFC against ground-truth logFC within each bin per method, recording the regression coefficient (bias) and R2 (variance), then aggregate into a summary table and reproduce the Fig. 3a-b and S5 scatter/line plots.

## Related tools

- **Normalisr** (Provides the reference QC pipeline (qc_reads, qc_outlier), normalization functions (lcpm, normcov, compute_var, normvar), and DE calling (normalisr.de) applied to every simulated replicate, plus normalisr.parallel for scalable execution) — github.com/lingfeiwang/normalisr
- **SymSim** (Simulation model whose parameters are fitted to the empirical count distributions of QC-filtered Perturb-seq data to generate two-group DE datasets with ground-truth logFC)
- **Splatter** (Alternative simulation model fitted to the same empirical count distributions, providing variation in simulation models and their similarity to the real dataset)
- **edgeR** (Bulk DE method run on every simulated replicate as a competing method whose logFC estimates are compared for bias and variance)
- **sctransform** (Single-cell normalization method run as a competing normalization/DE approach on simulated replicates)
- **bayNorm** (Single-cell Bayesian normalization method compared on simulated replicates)
- **Sanity** (Single-cell normalization method compared on simulated replicates)

## Examples

```
pip install normalisr; python3 -m normalisr de -h
```

## Evaluation signals

- The output table contains a bias (regression coefficient) and R2 entry per method per expression bin per minor-group size (519–3927 cells).
- Plotted bias/R2 relationships match the patterns reported in Fig. 3a-b and S5 of the paper.
- Simulated datasets reproduce the empirical mean/variance/library-size structure of the QC-filtered Perturb-seq (and optionally MARS-seq) counts.
- Ground-truth logFC is stored per gene for every simulation replicate before any method is run.
- Normalisr should consistently show one of the lowest biases and variances in logFC estimation across all expression levels, despite variation between SymSim and Splatter simulation models.

## Limitations

- Only UMI-based datasets were analyzed; full-length platforms may produce a different read-count distribution and would need revalidation of the fitted parameters and nonlinear covariates.
- Simulation fidelity varies — SymSim and Splatter differ in their similarity to the real dataset, so benchmark conclusions depend on both simulation models being included.
- Some competing methods could not be included: ScImpute and ZINB-WaVE could not accommodate the data size with excessive memory and/or time requirements, and ScVI does not recommend P value-based FDR estimation.

## Evidence

- [discussion] QC thresholds: "we restricted our analyses to cells with at least 500 reads and 100 genes with non-zero reads"
- [discussion] Gene filtering thresholds: "genes with non-zero reads both in at least 50 cells and in at least 2% of all cells for 10× technologies, or in at least 500 cells"
- [intro] UMI-based normalization: "The normalization step estimates the pre-measurement mRNA frequencies from the scRNA-seq read counts (e.g., unique molecular identifiers (UMIs))"
- [readme] Input file formats: [evidence span withheld — anchor status: not_found]
- [other] Parameter fitting step: [evidence span withheld — anchor status: not_found]
- [intro] Benchmarking motivation: "A comprehensive benchmarking found that single-cell-specific attempts in DE could not outperform existing bulk methods such as edgeR"
