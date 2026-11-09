---
name: single-cell-rnaseq-simulation
description: Use when benchmarking cell-cycle correction or latent-factor removal
  methods on synthetic scRNA-Seq data, or when testing how sparse an effect (fraction
  of control genes influenced) must be before a PCA-based detector like ccRemover
  misses it.
when_to_use_negative:
- The goal is to validate a biological finding — simulated matrices carry a planted
  effect and cannot substitute for real scRNA-Seq data
- The method under test does not rely on annotated cell-cycle genes or PCA structure
  (e.g. annotation-based enrichment tests without a component-detection step)
- The cell-cycle effect influences essentially none of the control genes — below ~8%
  affected the effect is not captured by leading principal components and any detection
  comparison will be uninformative for robustness claims
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ccRemover
  role: 'The correction algorithm run on each simulated dataset: PCA on control genes,
    bootstrap significance testing with cutoff CΔ = 3, iterative removal of detected
    cell-cycle components'
  repo: https://cran.r-project.org/package=ccRemover
- name: scLVM
  role: Benchmark correction method applied to simulated data with K = 1 and scree-plot-chosen
    K to expose removal of the wrong (cell-type) effect
- name: stats
  role: Used for tabulating detection success across the sweep of affected-control-gene
    proportions
provenance:
  source_task_ids:
  - task_007
  - task_001
  - task_004
  source_papers:
  - doi: 10.1038/srep33892
    title: Identifying and removing the cell-cycle effect from single-cell RNA-Sequencing
      data
  source_content_hash: sha256:5f9179b2275300dabc89c64f5e2d08dfffd92f8b3b0d9eb8dcc12da5a4792115
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Biomart
  - ccRemover
  - get_diff
  - org.Hs.eg.db
  - scLVM
  - stats
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# single-cell-rnaseq-simulation

## Summary

Generate simulated scRNA-Seq expression matrices with a planted cell-cycle effect (e.g. 50 cells x 2,000 genes with 400 cell-cycle genes) so that detection/removal methods like ccRemover can be benchmarked, including sweeps over how many control genes the effect touches.

## When to use

Use when benchmarking cell-cycle correction or latent-factor removal methods on synthetic scRNA-Seq data, or when testing how sparse an effect (fraction of control genes influenced) must be before a PCA-based detector like ccRemover misses it.

## When NOT to use

- The goal is to validate a biological finding — simulated matrices carry a planted effect and cannot substitute for real scRNA-Seq data
- The method under test does not rely on annotated cell-cycle genes or PCA structure (e.g. annotation-based enrichment tests without a component-detection step)
- The cell-cycle effect influences essentially none of the control genes — below ~8% affected the effect is not captured by leading principal components and any detection comparison will be uninformative for robustness claims

## Inputs

- Simulation design parameters: number of cells (e.g. 50), number of genes (e.g. 2,000), number of cell-cycle genes (e.g. 400)
- Proportion of control genes influenced by the cell-cycle effect (swept variable in Supplementary Section S2)
- Ground-truth cell type and cell-cycle stage labels per cell for downstream clustering checks

## Outputs

- Simulated gene-by-cell expression matrix X with known cell-cycle component
- if_cc indicator vector marking the 400 cell-cycle genes
- Per-replicate detection/removal outcome table (proportion of affected control genes vs. whether a significant Δj component was found and removed)

## How to apply

Simulate an expression matrix X with the baseline design of 50 cells and 2,000 genes, of which 400 are assigned as cell-cycle genes (if_cc indicator), planting the cell-cycle effect into a chosen proportion of the control genes (Supplementary Section S2 sweep). For each simulation replicate, run the correction being tested — for ccRemover this means PCA on the control-gene matrix XB, projection of cell-cycle genes XA, bootstrap significance testing (nboot = 100 for the sparsity sweep, cutoff CΔ = 3, ntop = 10 components, max_it = 4) — and record whether a significant Δj > 0 component is detected and removed. Tabulate detection/removal success versus the proportion of affected control genes to locate the reliability threshold (~8% of control genes affected). Also simulate scLVM-benchmark scenarios where the leading factors encode cell-cycle stage, cell type, or both, to expose methods that remove the wrong effect.

## Related tools

- **ccRemover** (The correction algorithm run on each simulated dataset: PCA on control genes, bootstrap significance testing with cutoff CΔ = 3, iterative removal of detected cell-cycle components) — https://cran.r-project.org/package=ccRemover
- **scLVM** (Benchmark correction method applied to simulated data with K = 1 and scree-plot-chosen K to expose removal of the wrong (cell-type) effect)
- **stats** (Used for tabulating detection success across the sweep of affected-control-gene proportions)

## Examples

```
R: gene_cycle_gene_indices <- gene_indexer(gene_names, species="mouse", name_type="symbols"); xhat <- ccRemover(dat, cutoff=3, max_it=4, nboot=200, ntop=10, bar=FALSE)
```

## Evaluation signals

- On baseline simulated data (50 cells, 2,000 genes, 400 cell-cycle genes), ccRemover identifies the 2nd and 3rd principal components as cell-cycle effects and removes them, leaving cells clustered by cell type with cell-cycle stages mixed
- The sparsity sweep reproduces the reported threshold: ccRemover reliably detects and removes the effect once at least ~8% of control genes are affected, and misses it below that level
- Corrected matrix contains no significant Δj > 0 component on a final pass (algorithm terminates within max_it = 4 iterations)
- scLVM failure modes are reproducible: with K = 1 it removes the cell-type effect instead of the cell-cycle effect; with K = 3 it removes both, leaving only noise
- Gene means removed during centering are added back so the corrected matrix xhat remains on the original expression scale

## Limitations

- If the cell-cycle effect influences only a few control genes, it may not be captured by the principal components and hence missed for detection (threshold ~8% of control genes affected)
- Simulations rely on the assumed generative model; conclusions may not transfer to datasets where cell-cycle activity is very low (e.g. glioblastoma data, where the paper instead prescribes cutoff CΔ = 2)
- Version numbers and full supplementary simulation details (Sections S2–S3) are not fully specified in the main text, limiting exact reproducibility

## Evidence

- [results] We simulate data matrix X that contains measurements for 50 cells and 2,000 genes, of these genes 400 are assigned as cell-cycle genes: "We simulate data matrix X that contains measurements for 50 cells and 2,000 genes, of these genes 400 are assigned as cell-cycle genes"
- [results] ccRemover correctly identifies the second and third principal components: "ccRemover is able to correctly identify the second and third principal components as cell-cycle effects and removes them"
- [discussion] if the cell-cycle effect only influences a few of the control genes, it may not be captured by the principal components: "if the cell-cycle effect only influences a few of the control genes, it may not be captured by the principal components and hence missed for detection"
- [methods] xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE): "xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE)"
- [results] scLVM has failed completely here by mistakenly removing the cell-type effect instead of the cell-cycle effect: "scLVM has failed completely here by mistakenly removing the cell-type effect instead of the cell-cycle effect"
