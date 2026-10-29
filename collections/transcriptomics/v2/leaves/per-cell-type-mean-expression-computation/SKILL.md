---
name: per-cell-type-mean-expression-computation
description: Use when you have a labeled scRNA-seq reference matrix (cells × genes
  with cell-type annotations) and need cell-type-level mean expression vectors to
  feed the E-test ΔS gene-selection statistic or a multinomial MLE classifier.
when_to_use_negative:
- The expression matrix has no cell-type labels — the E-test ΔS statistic requires
  annotated groups to compute per-type means.
- Data are raw UMI counts or raw TPM that have not been log-normalized; the multinomial
  parameter estimation with Laplace smoothing assumes normalized values.
- The goal is unsupervised dimensionality reduction (e.g. HVG detection without labels)
  rather than supervised gene selection.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SciBet
  role: R package whose SelectGene(expr, k, r) function consumes the per-cell-type
    mean expression vectors to compute ΔS and select the top 500 informative genes
  repo: github.com/PaulingLiu/scibet
- name: E-test
  role: 'Gene-selection statistic computed from the per-cell-type mean vectors: ΔS_i
    = n × (ln(AM_i) − ln(GM_i)) over the n per-type means'
  repo: github.com/PaulingLiu/scibet
- name: scikit-learn (sklearn)
  role: Provides model_selection.train_test_split for the stratified 7:3 reference/test
    split that defines the labeled training matrix input
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:687e177b8843a40fe2731b2a7f8ffc130e378dbcffdba7ca0d0938722bdfcbd4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - E-test
  - SciBet
  - scikit-learn (sklearn)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# per-cell-type-mean-expression-computation

## Summary

Computes the per-cell-type mean expression vector for every gene in a labeled single-cell training matrix (e.g., the 70% reference split of GSE104276); these means are the foundation of SciBet's E-test ΔS statistic and its multinomial likelihood classification.

## When to use

Use when you have a labeled scRNA-seq reference matrix (cells × genes with cell-type annotations) and need cell-type-level mean expression vectors to feed the E-test ΔS gene-selection statistic or a multinomial MLE classifier.

## When NOT to use

- The expression matrix has no cell-type labels — the E-test ΔS statistic requires annotated groups to compute per-type means.
- Data are raw UMI counts or raw TPM that have not been log-normalized; the multinomial parameter estimation with Laplace smoothing assumes normalized values.
- The goal is unsupervised dimensionality reduction (e.g. HVG detection without labels) rather than supervised gene selection.

## Inputs

- Labeled single-cell training expression matrix (cells × genes), e.g. the 70% reference split of GSE104276
- Per-cell cell type annotation vector/metadata aligned to the matrix rows
- Pre-normalized expression values (UMI: size-factor 10,000 + add 1 + log; full-length: TPM + pseudocount 1 + log)

## Outputs

- Per-gene set of n cell-type mean expression values (n = number of cell types)
- Ranked informative gene list with per-gene ΔS values (informative_genes.csv; 500 genes by default)
- Fitted SciBet model summary (cell-type mean expression vectors over informative genes)

## How to apply

Load the labeled training expression matrix — normalized per SciBet conventions (UMI data: size-factor 10,000, add one, log normalization; full-length reads: TPM plus pseudo-value one, log-normalized) — with cell type annotations. For each gene, group cells by their annotated type and compute the arithmetic mean expression across all cells of each type, yielding n cell-type means per gene (n = number of cell types). These mean vectors are the input to the E-test statistic ΔS_i = n × (ln(AM_i) − ln(GM_i)), where AM_i and GM_i are the arithmetic and geometric means of the n per-type means; genes with top ΔS are selected as informative genes (500 by default via SelectGene(expr, k, r)). The same mean vectors underlie classification, since each cell type is represented by its simple mean expression vector and the test cell is assigned to the type with minimal Kullback–Leibler divergence. The SciBet R package (devtools::install_github("PaulingLiu/scibet")) wraps this computation in SelectGene/Learn functions.

## Related tools

- **SciBet** (R package whose SelectGene(expr, k, r) function consumes the per-cell-type mean expression vectors to compute ΔS and select the top 500 informative genes) — github.com/PaulingLiu/scibet
- **E-test** (Gene-selection statistic computed from the per-cell-type mean vectors: ΔS_i = n × (ln(AM_i) − ln(GM_i)) over the n per-type means) — github.com/PaulingLiu/scibet
- **scikit-learn (sklearn)** (Provides model_selection.train_test_split for the stratified 7:3 reference/test split that defines the labeled training matrix input)

## Examples

```
if (!requireNamespace("devtools")) install.packages("devtools"); devtools::install_github("PaulingLiu/scibet"); genes <- SelectGene(expr, k=500, r=0.2)
```

## Evaluation signals

- informative_genes.csv exists and contains exactly 500 gene rows (SciBet's default k=500).
- Each output row has a gene identifier and a non-negative ΔS value (ΔS ≥ 0 since AM ≥ GM).
- The number of per-type means per gene equals the number of distinct annotated cell types in the training split.
- For each gene, arithmetic mean AM_i of the cell-type means is ≥ geometric mean GM_i, so ln(AM_i) − ln(GM_i) ≥ 0.
- Genes retained in the top-500 set are reproducibly recovered across repeated stratified 7:3 splits (procedure repeated 50× in the paper).

## Limitations

- The statistic assumes the reference covers all relevant cell types; cell types absent from the reference are handled separately via the null-dataset confidence score, not by the mean vectors.
- No seed or randomization parameters were reported for the 50× repeated splits, so exact reproducibility of the selected gene set is not guaranteed.
- ΔS significance is only assessed via a random permutation test of cell group labels, whose approximation strategy is relegated to Supplementary Note 2.4 and not fully described in the main text.

## Evidence

- [discussion] Both E-test and SciBet utilizes the fundamental concept that each cell type is represented by the simple mean expression vector: "Both E-test and SciBet utilizes the fundamental concept that each cell type is represented by the simple mean expression vector"
- [discussion] Top ΔS genes selected as informative genes: "we selected genes with top ΔS as informative genes (500 by default)"
- [discussion] Normalization convention for UMI data: "normalizing the UMI count of each cell with size-factor 10,000, adding one and then log normalization"
- [readme] Package installation for reproducing the computation: "devtools::install_github("PaulingLiu/scibet")"
- [discussion] Reference split construction from GSE104276: "For dataset GSE104276, which comprised cells from human prefrontal cortex, we randomly sampled 70% of the cells as reference"
