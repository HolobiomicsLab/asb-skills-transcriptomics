---
name: e-test-gene-feature-selection
description: Use when building a supervised single-cell classifier from labeled scRNA-seq
  data and you need a small, cell-type-discriminative gene set before fitting the
  multinomial model.
when_to_use_negative:
- The expression matrix has no cell type labels — ΔS is computed from per-cell-type
  means, so unlabeled data cannot be scored
- Downstream annotation does not use a multinomial/likelihood model over a compact
  marker set — genes with high ΔS are tuned to SciBet's classifier, not generic clustering
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: SciBet
  role: Provides SelectGene/Entropy functions implementing the E-test ΔS ranking and
    top-k gene selection
  repo: https://github.com/PaulingLiu/scibet
- name: E-test
  role: 'The statistic itself: ΔS = n·(ln(AM) − ln(GM)) over per-cell-type mean expression
    vectors'
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:609dd946ae84497fa3a7561436903e8f49c164c9ceb044b7f756e3e61ca3f35d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/scibet
  tools:
  - E-test
  - SciBet
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# e-test-gene-feature-selection

## Summary

SciBet's E-test selects cell type-specific genes by ranking each gene with the entropy-difference statistic ΔS = n·(ln(AM) − ln(GM)) computed over the n cell-type mean expression vectors, retaining the top k (default 500) informative genes for multinomial classifier training.

## When to use

Use when building a supervised single-cell classifier from labeled scRNA-seq data and you need a small, cell-type-discriminative gene set before fitting the multinomial model.

## When NOT to use

- The expression matrix has no cell type labels — ΔS is computed from per-cell-type means, so unlabeled data cannot be scored
- Downstream annotation does not use a multinomial/likelihood model over a compact marker set — genes with high ΔS are tuned to SciBet's classifier, not generic clustering

## Inputs

- Labeled single-cell training expression matrix (log-normalized counts, e.g. 70% stratified split of GSE104276) with cell type annotations
- Number of informative genes to keep (k, default 500)

## Outputs

- Ranked list of the top 500 informative genes with per-gene ΔS values (e.g. informative_genes.csv)
- Gene set passed to Learn for fitting multinomial cell-type parameters (p_ij vectors with Laplace smoothing)

## How to apply

Start from the labeled training expression matrix (e.g. the 70% reference split of GSE104276, log-normalized: TPM + pseudo-value one for full-length data, or Seurat v3 size-factor-10,000 + one + log for UMI data). For each gene, compute the per-cell-type mean expression vector, then the ΔS statistic: ΔS_i = n·(ln(AM_i) − ln(GM_i)), where AM_i and GM_i are the arithmetic and geometric means of the n cell-type means — a large ΔS means the gene's mean expression deviates strongly from a null hypothesis of equal expression across types. Rank genes in descending ΔS and retain the top 500 (SciBet's default, via SelectGene(expr, k, r) with k=500); significance of each ΔS can additionally be assessed with a random permutation test of cell group labels. The selected gene set and the cell-type parameter vectors learned from it (p_ij estimated with across-gene normalization and Laplace smoothing) together define the multinomial model used for maximum-likelihood annotation. Write the ranked list with per-gene ΔS values to a file such as informative_genes.csv for traceability.

## Related tools

- **SciBet** (Provides SelectGene/Entropy functions implementing the E-test ΔS ranking and top-k gene selection) — https://github.com/PaulingLiu/scibet
- **E-test** (The statistic itself: ΔS = n·(ln(AM) − ln(GM)) over per-cell-type mean expression vectors)

## Examples

```
devtools::install_github("PaulingLiu/scibet"); geneset <- SelectGene(expr, k = 500, r = NULL)
```

## Evaluation signals

- informative_genes.csv exists, contains exactly 500 rows (default k), and each row has a gene identifier and a non-negative ΔS value
- Genes are ranked in strictly descending ΔS order
- AM_i ≥ GM_i for every gene, so ΔS_i = n·(ln(AM_i) − ln(GM_i)) is never negative
- Downstream validation: fitted classifier assigns held-out test cells to reference cell types with high concordance, consistent with SciBet's reported high prediction accuracy
- Optional: per-gene ΔS significance confirmed by random permutation test of cell group labels

## Limitations

- Only gene ranking significance is provided; permutation-test approximation strategy is relegated to Supplementary Note 2.4 and not described in the main text
- The permutation-based significance assessment can be slow (article notes an approximation strategy to accelerate it)
- Gene set is data-dependent: it reflects the cell types present in the training split, so types absent from the reference cannot be selected for

## Evidence

- [discussion] ΔS definition and selection rule: [evidence span withheld — anchor status: not_found]
- [discussion] Top-k default: "we selected genes with top ΔS as informative genes (500 by default)"
- [discussion] Permutation significance: "The significance of ΔS for each gene can be obtained by the random permutation test"
- [discussion] Mean-vector basis of E-test: "Both E-test and SciBet utilizes the fundamental concept that each cell type is represented by the simple mean expression vector"
- [readme] Installation of the implementing package: "devtools::install_github("PaulingLiu/scibet")"
