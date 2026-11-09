---
name: cell-cycle-effect-removal
description: Use when a scRNA-Seq expression matrix (log-scaled, gene-by-gene centered)
  shows within-cell-type heterogeneity driven by annotated cell-cycle genes that obscures
  clustering of cell types or substructure, and a known cell-cycle gene set can be
  retrieved.
when_to_use_negative:
- When the confounder to remove is not the cell cycle — ccRemover does not remove
  batch effects or other technical effects, as it is designed for removing the cell-cycle
  effect exclusively
- When the cell-cycle effect influences only a small fraction of control genes (below
  ~8%), since it may not be captured by the leading principal components and will
  be missed
- When the analysis goal requires retaining cell-cycle state information (e.g., studying
  cell-cycle itself rather than removing it as a bias)
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ccRemover
  role: 'Primary algorithm: iteratively PCAs the control genes, projects onto cell-cycle
    genes, bootstrap-tests Δj against cutoff CΔ, and removes significant cell-cycle
    components'
  repo: cran/ccRemover
- name: gene_indexer
  role: Maps gene names to cell-cycle gene indices for mouse or human, producing the
    if_cc TRUE/FALSE indicator vector
  repo: cran/ccRemover
- name: scLVM
  role: Baseline latent-factor correction applied via its Python script (default K
    = 1 and scree-plot K) for comparison of clustering and enrichment outcomes
- name: Biomart
  role: Source of lists of genes annotated to the cell cycle used to build the if_cc
    indicator
- name: org.Mm.eg.db
  role: Mouse cell-cycle gene annotation database for identifying cell-cycle genes
- name: org.Hs.eg.db
  role: Human cell-cycle gene annotation database for identifying cell-cycle genes
provenance:
  source_task_ids:
  - task_003
  - task_007
  - task_004
  source_papers:
  - doi: 10.1038/srep33892
    title: Identifying and removing the cell-cycle effect from single-cell RNA-Sequencing
      data
  source_content_hash: sha256:6905a4de276e6bbaf2d078fa0c3f861a0254cf6aec355c4ef1a97228b43965c0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cran/ccRemover
  tools:
  - Biomart
  - Python
  - ccRemover
  - gene_indexer
  - org.Hs.eg.db
  - org.Mm.eg.db
  - scLVM
  - stats
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# cell-cycle-effect-removal

## Summary

Identifies and removes the cell-cycle effect from single-cell RNA-Seq data using ccRemover, a formal bootstrap-tested PCA-based procedure, so that downstream clustering detects true biological substructure rather than cell-cycle-driven heterogeneity.

## When to use

Use when a scRNA-Seq expression matrix (log-scaled, gene-by-gene centered) shows within-cell-type heterogeneity driven by annotated cell-cycle genes that obscures clustering of cell types or substructure, and a known cell-cycle gene set can be retrieved.

## When NOT to use

- When the confounder to remove is not the cell cycle — ccRemover does not remove batch effects or other technical effects, as it is designed for removing the cell-cycle effect exclusively
- When the cell-cycle effect influences only a small fraction of control genes (below ~8%), since it may not be captured by the leading principal components and will be missed
- When the analysis goal requires retaining cell-cycle state information (e.g., studying cell-cycle itself rather than removing it as a bias)

## Inputs

- scRNA-Seq gene-by-cell expression matrix (e.g., t.cell_data 81 cells x 14,147 genes with genes as rows)
- Vector of gene names (symbols or Ensembl IDs) matching the matrix rows
- Species identifier ('mouse' or 'human') for cell-cycle gene annotation lookup
- Known cell-cycle gene set retrieved from Biomart / org.Mm.eg.db / org.Hs.eg.db

## Outputs

- Cell-cycle-corrected expression matrix xhat (gene means re-added) preserving non-cell-cycle biological signal
- Per-iteration bootstrap test statistics (e.g., t.load.boot) identifying which principal components are cell-cycle effects
- Downstream comparison artifacts: clustering results, DE gene lists, and enrichment statistics (odds ratios, hypergeometric p-values, CIs)

## How to apply

Log-transform the raw counts (add 1 and take natural log) and center the matrix gene-by-gene with rowMeans, storing gene means for re-adding after correction. Identify cell-cycle genes from annotation databases (Biomart, org.Mm.eg.db / org.Hs.eg.db) using gene_indexer(gene_names, species, name_type), build the if_cc indicator, and assemble dat <- list(x = centered_matrix, if_cc = if_cc); all non-annotated genes serve as control genes. Run ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10): each iteration performs PCA on control genes, projects onto cell-cycle genes, and bootstrap-tests each leading component's contribution Δj against cutoff CΔ = 3, removing only components formally flagged as cell-cycle. Lower the cutoff to CΔ = 2 when cell-cycle activity is known to be very low (e.g., glioblastoma data), and compare against scLVM (default K = 1 and the scree-plot K) plus the uncorrected original data to confirm the biological signal of interest is preserved.

## Related tools

- **ccRemover** (Primary algorithm: iteratively PCAs the control genes, projects onto cell-cycle genes, bootstrap-tests Δj against cutoff CΔ, and removes significant cell-cycle components) — cran/ccRemover
- **gene_indexer** (Maps gene names to cell-cycle gene indices for mouse or human, producing the if_cc TRUE/FALSE indicator vector) — cran/ccRemover
- **scLVM** (Baseline latent-factor correction applied via its Python script (default K = 1 and scree-plot K) for comparison of clustering and enrichment outcomes)
- **Biomart** (Source of lists of genes annotated to the cell cycle used to build the if_cc indicator)
- **org.Mm.eg.db** (Mouse cell-cycle gene annotation database for identifying cell-cycle genes)
- **org.Hs.eg.db** (Human cell-cycle gene annotation database for identifying cell-cycle genes)

## Examples

```
if_cc <- gene_indexer(gene_names, species='mouse', name_type='symbols'); dat <- list(x=x_cen, if_cc=if_cc); xhat <- ccRemover(dat, cutoff=3, max_it=4, nboot=200, ntop=10, bar=FALSE)
```

## Evaluation signals

- On the T-helper data (cutoff=3, nboot=200), PC1 is flagged as a cell-cycle effect on the first iteration with t.load.boot = 7.13, and no further effects are detected on iteration 2
- On the T-helper dataset, TH2 signature enrichment after ccRemover should reach odds ratio 3.439 (95% CI lower bound 2.297, hypergeometric p = 2.373e-9) versus scLVM's 2.382, while the original data baseline has OR < 1
- On the glioblastoma data with cutoff CΔ = 2, tumor-of-origin hierarchical clustering accuracy should fall within ~±2 percentage points of 92.32% (ccRemover), 90.00% (scLVM), and 87.44% (original), with ccRemover exceeding original
- On simulated data (50 cells, 2,000 genes, 400 cell-cycle genes), ccRemover should flag the cell-cycle-associated principal components and cluster cells by cell type with cell-cycle stages mixed, not remove the cell-type effect as scLVM K=1 does

## Limitations

- Detection requires the cell-cycle effect to influence enough control genes: below ~8% of control genes affected, the effect may not be captured by the leading principal components and is missed
- Annotation databases are always incomplete and inaccurate, so the known cell-cycle gene set is imperfect
- ccRemover removes only the cell-cycle effect; other confounding effects that may dominate after its removal are left in the data
- Choice of cutoff CΔ must be adapted: default CΔ = 3, reduced to CΔ = 2 when cell-cycle activity is known to be very low (e.g., glioblastoma); no software versions are reported in the paper

## Evidence

- [methods] ccRemover call with default parameters: "xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE)"
- [methods] pre-processing recommendation: "We recommend transforming the data to a log scale and centering it on a gene-by-gene basis."
- [methods] reduced cutoff for low cell-cycle activity: "except for the glioblastoma data, where it is known that the cell-cycle activity is at a very low level47. We use a smaller cutoff value CΔ = 2"
- [discussion] sparsity limitation: "if the cell-cycle effect only influences a few of the control genes, it may not be captured by the principal components and hence missed for detection"
- [discussion] scope limitation versus other confounders: "ccRemover does not remove these effects as it is designed for removing the cell-cycle effect exclusively."
- [results] scLVM failure mode motivating the method: "scLVM has failed completely here by mistakenly removing the cell-type effect instead of the cell-cycle effect"
