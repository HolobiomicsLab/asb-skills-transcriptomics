---
name: arithmetic-geometric-mean-dispersion-statistic
description: Use when you have a labeled scRNA-seq training matrix with cell type
  annotations and need a fast, supervised feature-selection statistic to rank/select
  cell type-specific genes before training a classifier like SciBet.
when_to_use_negative:
- The training matrix has no cell type labels — ΔS is a supervised statistic defined
  over per-cell-type mean expression vectors and cannot be computed without annotations
- The input is already a reduced/selected feature set or a ranked gene list rather
  than a per-gene expression matrix requiring feature selection
- You need unsupervised marker discovery for unannotated data; SciBet itself positions
  supervised annotation with known labels as the alternative to laborious unsupervised
  approaches
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: SciBet
  role: Hosts the E-test feature-selection step; SelectGene(expr, k, r) implements
    ΔS ranking and top-k gene selection
  repo: https://github.com/PaulingLiu/scibet
- name: E-test
  role: Computes the ΔS statistic from per-cell-type mean expression vectors; SciBet
    relies on the same cell-type mean-vector concept
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:cafed0d2715f5d52883c8f41eed5d0cc2ddd53889536bfbd6bbaed1544bb547e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/scibet
  tools:
  - E-test
  - SciBet
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# arithmetic-geometric-mean-dispersion-statistic

## Summary

The E-test ΔS statistic from SciBet quantifies per-gene cell-type specificity as the entropy difference between the arithmetic and geometric means of per-cell-type mean expression, ΔS_i = n·(ln AM_i − ln GM_i); genes with larger ΔS are kept as informative features for the multinomial classifier.

## When to use

Use when you have a labeled scRNA-seq training matrix with cell type annotations and need a fast, supervised feature-selection statistic to rank/select cell type-specific genes before training a classifier like SciBet.

## When NOT to use

- The training matrix has no cell type labels — ΔS is a supervised statistic defined over per-cell-type mean expression vectors and cannot be computed without annotations
- The input is already a reduced/selected feature set or a ranked gene list rather than a per-gene expression matrix requiring feature selection
- You need unsupervised marker discovery for unannotated data; SciBet itself positions supervised annotation with known labels as the alternative to laborious unsupervised approaches

## Inputs

- Labeled single-cell expression matrix (e.g. TPM + log-normalized full-length data, or Seurat v3-style size-factor-10,000 log-normalized UMI data) with per-cell cell type annotations
- Selection parameter k (number of informative genes; SciBet default 500)

## Outputs

- informative_genes.csv: ranked gene list (descending ΔS), exactly k = 500 rows, each with a gene identifier and a non-negative ΔS value
- Optional per-gene permutation-test significance for ΔS

## How to apply

Load the labeled training expression matrix (e.g. the 70% reference split of GSE104276, normalized per SciBet's log-normalization rules) with cell type labels. Compute the per-cell-type mean expression vector for every gene, giving n cell-type means per gene. For each gene compute ΔS_i = n·(ln AM_i − ln GM_i), where AM_i is the arithmetic mean and GM_i the geometric mean of the n cell-type means. Rank genes in descending ΔS order and keep the top k = 500 (SciBet's default, via SelectGene(expr, k, r)); optionally assess per-gene significance with a random permutation test of cell group labels. The rationale is that the ΔS entropy gap captures how strongly a gene's cell-type-mean pattern deviates from a homogeneous (single-mean) null, so large-ΔS genes are discriminative for supervised annotation.

## Related tools

- **SciBet** (Hosts the E-test feature-selection step; SelectGene(expr, k, r) implements ΔS ranking and top-k gene selection) — https://github.com/PaulingLiu/scibet
- **E-test** (Computes the ΔS statistic from per-cell-type mean expression vectors; SciBet relies on the same cell-type mean-vector concept)

## Examples

```
devtools::install_github("PaulingLiu/scibet"); genes <- SelectGene(expr, k = 500)
```

## Evaluation signals

- informative_genes.csv exists and contains exactly 500 gene rows (matching SciBet's default k)
- Every row has a gene identifier and a non-negative ΔS value
- Genes are ordered by descending ΔS (top rows show the largest entropy gap)
- Per-gene ΔS values are consistent with hand-computed n·(ln AM − ln GM) on a small validation subset
- Selected genes, fed into downstream SciBet classification (e.g. 7:3 stratified hold-out repeated 50× on GSE104276), reproduce expected prediction accuracy with low false positive rate

## Limitations

- ΔS magnitude depends on the number of cell types n, so values are not directly comparable across references with different cell-type counts
- ΔS significance assessment by random permutation test relies on an approximation strategy relegated to Supplementary Note 2.4, and no seed/randomization parameters are reported for the permutation/splitting procedures
- The statistic presupposes each cell type is represented by a simple mean expression vector — heterogeneity within a labeled type is collapsed into one mean

## Evidence

- [discussion] Top-k selection default: "we selected genes with top ΔS as informative genes (500 by default)"
- [discussion] Cell-type mean-vector concept: "Both E-test and SciBet utilizes the fundamental concept that each cell type is represented by the simple mean expression vector"
- [discussion] Permutation-test significance: "The significance of ΔS for each gene can be obtained by the random permutation test"
- [methods] SelectGene API: "- `SelectGene(expr, k, r)`"
- [readme] Installation command: "devtools::install_github("PaulingLiu/scibet")"
