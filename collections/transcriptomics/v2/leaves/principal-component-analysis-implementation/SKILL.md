---
name: principal-component-analysis-implementation
description: Use when a normalized, log-transformed, gene-centered scRNA-Seq matrix
  must be decomposed into principal components to identify dominant directions of
  unwanted variation (e.g. cell-cycle effect) prior to removal or clustering.
when_to_use_negative:
- The goal is to remove general hidden confounders (e.g. library-quality or batch
  effects) rather than the cell-cycle effect — scLVM-style latent-factor removal is
  the wrong target here since ccRemover is designed exclusively for the cell cycle
- The input matrix is not log-transformed and gene-centered, or no cell-cycle gene
  annotation (if_cc indicator) is available to define control vs cell-cycle gene sets
- Cell-cycle activity is known to be very low and the standard cutoff would miss it
  — the article uses a smaller cutoff C_delta = 2 for glioblastoma data instead of
  defaulting to C = 3
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3523
tools:
- name: ccRemover
  role: 'R package that wraps the iterative PCA-plus-bootstrap loop: PCA on XB, projection
    of XA, bootstrap significance testing, and subtraction of significant cell-cycle
    directions'
  repo: https://cran.r-project.org/package=ccRemover
- name: bootstrap_diff
  role: Generates nboot = 200 bootstrap replicates of the loading-strength difference
    per component and supports the bootstrap-based t-statistic used to call significant
    components
- name: get_diff
  role: Computes the difference in loading strength between cell-cycle and control
    genes for each principal component prior to bootstrap testing
- name: gene_indexer
  role: Maps gene names to cell-cycle annotation indices (species = "mouse" or "human")
    to build the if_cc indicator before PCA
- name: scLVM
  role: Comparison latent-variable method applied in the benchmark; unlike the PCA-based
    ccRemover test it does not inspect whether a leading factor represents the cell
    cycle
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/srep33892
    title: Identifying and removing the cell-cycle effect from single-cell RNA-Sequencing
      data
  source_content_hash: sha256:02d1b06733efea0a0db37d4de08de67b00a6788b364c2b2f0232cd400b296ab9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - bootstrap_diff
  - ccRemover
  - gene_indexer
  - get_diff
  - scLVM
  - stats
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# principal-component-analysis-implementation

## Summary

Implements PCA on a log-transformed, gene-by-gene centered single-cell expression matrix to expose leading components of systematic variation, as in ccRemover where PCA on control genes XB yields loadings v1..v_ntop used to test for cell-cycle effects.

## When to use

Use when a normalized, log-transformed, gene-centered scRNA-Seq matrix must be decomposed into principal components to identify dominant directions of unwanted variation (e.g. cell-cycle effect) prior to removal or clustering.

## When NOT to use

- The goal is to remove general hidden confounders (e.g. library-quality or batch effects) rather than the cell-cycle effect — scLVM-style latent-factor removal is the wrong target here since ccRemover is designed exclusively for the cell cycle
- The input matrix is not log-transformed and gene-centered, or no cell-cycle gene annotation (if_cc indicator) is available to define control vs cell-cycle gene sets
- Cell-cycle activity is known to be very low and the standard cutoff would miss it — the article uses a smaller cutoff C_delta = 2 for glioblastoma data instead of defaulting to C = 3

## Inputs

- Normalized, log-transformed, gene-by-gene centered gene-by-cell expression matrix
- if_cc indicator vector labeling cell-cycle genes, splitting genes into cell-cycle set XA and control set XB
- ntop number of leading components to test (default 10)

## Outputs

- PCA loadings v1,...,v_ntop and component scores beta_j for control genes XB
- Projected component scores of the cell-cycle gene matrix XA
- Per-component differences in loading strength between cell-cycle and control genes, fed to bootstrap_diff for t-statistic computation

## How to apply

Log-transform the expression data and center it gene-by-gene, then split genes into the cell-cycle set XA (via the if_cc indicator) and the control set XB. Perform PCA on the control-gene matrix XB to obtain loadings v1,...,v_ntop with ntop = 10 components tested per iteration, and compute component scores beta_j = XB v_j; also project the cell-cycle gene matrix XA onto each loading. Rank components by the difference in loading strength between cell-cycle and control genes, and pass those differences to a two-class bootstrap (bootstrap_diff, nboot = 200 resamples) to compute a bootstrap-based t-statistic; declare components significant when t exceeds the cutoff C = 3 (roughly p = 0.01). Rationale: PCA on control genes captures dominant variation whose loading strength should be lower for non-cell-cycle genes, so an excess in XA signals a cell-cycle direction.

## Related tools

- **ccRemover** (R package that wraps the iterative PCA-plus-bootstrap loop: PCA on XB, projection of XA, bootstrap significance testing, and subtraction of significant cell-cycle directions) — https://cran.r-project.org/package=ccRemover
- **bootstrap_diff** (Generates nboot = 200 bootstrap replicates of the loading-strength difference per component and supports the bootstrap-based t-statistic used to call significant components)
- **get_diff** (Computes the difference in loading strength between cell-cycle and control genes for each principal component prior to bootstrap testing)
- **gene_indexer** (Maps gene names to cell-cycle annotation indices (species = "mouse" or "human") to build the if_cc indicator before PCA)
- **scLVM** (Comparison latent-variable method applied in the benchmark; unlike the PCA-based ccRemover test it does not inspect whether a leading factor represents the cell cycle)

## Examples

```
gene_cycle_gene_indices <- gene_indexer(gene_names, species = "mouse", name_type = "symbols"); xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE)
```

## Evaluation signals

- Per-iteration component calls: on simulated data ccRemover must identify the 2nd and 3rd principal components as cell-cycle effects; on T helper data it must flag the 1st PC on iteration 1 and detect no further effects after removal
- Bootstrap t-statistics exceed the cutoff C = 3 for components declared significant (roughly p = 0.01), with the loop terminating when a full pass finds no significant component or after max_it = 4 iterations
- Corrected matrix preserves cell-type structure: after removal, cells cluster by cell type with cell-cycle stages mixed (not vice versa, which indicates scLVM-style failure)
- Downstream sanity check: 2-means clustering plus t-test at FDR 0.01 should yield a TH2 signature odds ratio > 1 on corrected T helper data (odds ratio < 1 on original data indicates clustering is not physiologically meaningful)
- Output xhat matrix has gene means added back and is written to xhat.csv with the same gene-by-cell dimensions as the input

## Limitations

- ccRemover may miss the cell-cycle effect if it influences too few control genes (below ~8% affected), since such a signal may not be captured by the leading principal components
- The method relies on annotation databases (Cyclebase, org.Mm.eg.db, org.Hs.eg.db) that are incomplete and inaccurate, and requires a known set of cell-cycle genes
- ccRemover removes only the cell-cycle effect; other confounding effects may dominate after removal and are not corrected
- Removal of leading components risks deleting biological signal of interest if the tested components are not truly cell-cycle related, as seen when scLVM removed the cell-type effect on simulated data

## Evidence

- [methods] Perform a PCA on the data matrix of control genes XB.: "The ccRemover algorithm follows these steps: 1. Perform a PCA on the data matrix of control genes XB."
- [methods] declare components with t > cutoff (default C = 3) significant cell-cycle effects: [evidence span withheld — anchor status: not_found]
- [methods] We recommend transforming the data to a log scale and centering it on a gene-by-gene basis.: "We recommend transforming the data to a log scale and centering it on a gene-by-gene basis."
- [results] ccRemover is able to correctly identify the second and third principal components as cell-cycle effects and removes them: "ccRemover is able to correctly identify the second and third principal components as cell-cycle effects and removes them"
- [methods] xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE): "xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE)"
- [methods] Take a random sample with replacement of |A| rows from XA and another |B| rows from XB: [evidence span withheld — anchor status: not_found]
