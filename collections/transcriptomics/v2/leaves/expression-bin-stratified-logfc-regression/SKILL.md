---
name: expression-bin-stratified-logfc-regression
description: Use when you need to quantify how accurately normalization/DE methods
  estimate effect sizes (logFC) on simulated single-cell data with known ground truth,
  especially to detect expression-level-dependent bias.
when_to_use_negative:
- You lack ground-truth effect sizes — the regression against truth requires simulated
  data with known per-gene logFC, not real datasets.
- Evaluating p-value calibration or FDR sensitivity/specificity rather than effect-size
  (logFC) accuracy — this skill targets bias and variance of logFC, not significance
  calling.
- Data are full-length (non-UMI) scRNA-seq — the authors only focused on UMI-based
  datasets and note count distributions may differ on full-length platforms.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_2269
tools:
- name: Normalisr
  role: 'Reference pipeline: QC (qc_reads, qc_outlier), normalization (lcpm, normcov,
    compute_var, normvar), DE (normalisr.de), and parallel execution (normalisr.parallel)
    on simulated datasets'
  repo: https://github.com/lingfeiwang/normalisr
- name: SymSim
  role: Simulator fit to empirical Perturb-seq count distributions, generating two-group
    DE datasets with ground-truth logFC
- name: Splatter
  role: Alternative simulator fit to the same empirical count distributions for robustness
    across simulation models
- name: edgeR
  role: Bulk DE method compared for logFC bias/variance on the simulated datasets
- name: sctransform
  role: Single-cell normalization method compared in the benchmark
- name: bayNorm
  role: Single-cell Bayesian normalization method compared in the benchmark
- name: Sanity
  role: Single-cell normalization method compared in the benchmark
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:9f9328a5533c2ba7fe4be61a96bd561ce24b4a22ad25815ff78b039b392ac271
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lingfeiwang/normalisr
  tools:
  - Normalisr
  - Sanity
  - Splatter
  - SymSim
  - bayNorm
  - edgeR
  - sctransform
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# expression-bin-stratified-logfc-regression

## Summary

Benchmark accuracy of differential-expression effect-size estimation by binning genes by expression level and regressing each method's estimated logFC against simulation ground-truth logFC, recording regression coefficient (bias) and R2 (variance). Used to show Normalisr obtains among the lowest logFC bias and variance across expression levels versus edgeR and single-cell alternatives.

## When to use

Use when you need to quantify how accurately normalization/DE methods estimate effect sizes (logFC) on simulated single-cell data with known ground truth, especially to detect expression-level-dependent bias.

## When NOT to use

- You lack ground-truth effect sizes — the regression against truth requires simulated data with known per-gene logFC, not real datasets.
- Evaluating p-value calibration or FDR sensitivity/specificity rather than effect-size (logFC) accuracy — this skill targets bias and variance of logFC, not significance calling.
- Data are full-length (non-UMI) scRNA-seq — the authors only focused on UMI-based datasets and note count distributions may differ on full-length platforms.

## Inputs

- Perturb-seq UMI read count matrix from GEO (optionally MARS-seq melanoma dataset) as reference for simulation parameter fitting
- SymSim and Splatter simulated two-group count datasets with stored per-gene ground-truth logFC (minor-group sizes 519–3927 cells)
- Normalized expression matrices and per-gene estimated logFC from normalisr.de, edgeR, sctransform, bayNorm, and Sanity

## Outputs

- Summary table of regression coefficient (bias) and R2 per method per expression bin per minor-group size
- Scatter/line plots of bias and R2 versus expression level reproducing Fig. 3a-b and S5

## How to apply

Simulate two-group DE datasets with SymSim and Splatter, with parameters fit to the QC-filtered Perturb-seq UMI count matrix (cells ≥500 reads and ≥100 detected genes; genes detected in ≥50 cells and ≥2% of cells), spanning minor-group sizes of 519–3927 cells, and store the ground-truth logFC per gene per replicate. Run each method — normalisr.norm (lcpm, normcov, compute_var, normvar) followed by normalisr.de, plus edgeR, sctransform, bayNorm, and Sanity — on every replicate, using normalisr.parallel for scale. Bin genes by expression level and, within each bin, regress estimated logFC against stored ground-truth logFC for every method; the regression coefficient quantifies bias and R2 quantifies residual variance. Aggregate per-bin bias and R2 across methods and group sizes into a summary table and generate the scatter/line plots reproducing Fig. 3a–b and S5. Rationale: bias estimated this way isolates expression-dependent normalization distortions that aggregate metrics hide.

## Related tools

- **Normalisr** (Reference pipeline: QC (qc_reads, qc_outlier), normalization (lcpm, normcov, compute_var, normvar), DE (normalisr.de), and parallel execution (normalisr.parallel) on simulated datasets) — https://github.com/lingfeiwang/normalisr
- **SymSim** (Simulator fit to empirical Perturb-seq count distributions, generating two-group DE datasets with ground-truth logFC)
- **Splatter** (Alternative simulator fit to the same empirical count distributions for robustness across simulation models)
- **edgeR** (Bulk DE method compared for logFC bias/variance on the simulated datasets)
- **sctransform** (Single-cell normalization method compared in the benchmark)
- **bayNorm** (Single-cell Bayesian normalization method compared in the benchmark)
- **Sanity** (Single-cell normalization method compared in the benchmark)

## Examples

```
normalisr de --input sim_counts.mtx --group_labels groups.txt --output logfc.tsv && normalisr norm --input raw_counts.mtx --mode lcpm --output norm_counts.mtx
```

## Evaluation signals

- Output table contains exactly one bias (regression coefficient) and R2 entry per method per expression bin per minor-group size.
- Plotted bias/R2 relationships across bins match the patterns reported in Fig. 3a-b and S5 of the paper.
- Normalisr's per-bin regression coefficient should lie near 1 (low bias) with high R2 across all expression levels, consistent with the reported finding.
- Simulated data reproduce the empirical mean/variance/library-size structure of the QC-filtered Perturb-seq counts.
- Results are stable across both SymSim and Splatter simulation models and across minor-group sizes 519–3927.

## Limitations

- Conclusions depend on simulation fidelity: bias/variance estimates vary with how well SymSim/Splatter match the real dataset's count distributions.
- ScImpute and ZINB-WaVE were excluded from comparison because they could not accommodate the data size with excessive memory and/or time requirements; scVI was excluded because it does not recommend P value-based FDR estimation.
- The two-step regression score reflects effect-size estimation only; it does not assess p-value calibration or sensitivity/specificity of significance calls.

## Evidence

- [results] Normalisr consistently obtained one of the lowest biases and variances in logFC estimation across all expression levels, despite variation in the simulation models and their similarity to the real dataset.: [evidence span withheld — anchor status: not_found]
- [intro] Single-cell DE benchmarking context: "A comprehensive benchmarking found that single-cell-specific attempts in DE could not outperform existing bulk methods such as edgeR8."
- [discussion] QC thresholds grounding simulation parameter fitting: "we restricted our analyses to cells with at least 500 reads and 100 genes with non-zero reads"
- [intro] Existing normalization methods not designed for effect-size accuracy: "existing normalization methods were designed primarily for clustering. Sensitivity, specificity, and effect size estimation in association testing are left mostly uncharted."
- [methods] Normalisr function surface used for normalization and DE steps: "lcpm
	normcov
	scaling_factor
	compute_var
	normvar"
