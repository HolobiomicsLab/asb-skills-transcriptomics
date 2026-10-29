---
name: gene-ranking-and-top-k-selection
description: Use when a labeled single-cell training matrix with cell type annotations
  needs supervised feature selection before building SciBet (or similar multinomial)
  classifiers.
when_to_use_negative:
- Input is unlabeled data with no cell type annotations — ΔS requires per-cell-type
  mean expression vectors from a supervised reference
- You need a full model or predictions rather than a feature ranking — use Learn/Bet
  or ExportModel instead of SelectGene alone
- Input is already a reduced feature table of preselected marker genes; re-running
  ΔS ranking would discard prior curation
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SciBet
  role: Provides SelectGene(expr, k, r) and Entropy(expr, window, low) functions implementing
    the E-test ΔS ranking and top-k selection (k=500 default)
  repo: https://github.com/PaulingLiu/scibet
- name: E-test
  role: The entropy-based statistic ΔS = n*(ln(AM) − ln(GM)) computed over per-cell-type
    mean expression vectors to score gene informativeness
  repo: https://github.com/PaulingLiu/scibet
- name: scikit-learn (sklearn)
  role: model_selection.train_test_split supplies the stratified 7:3 labeled training
    split on which ΔS is computed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:ba24a5dc87e7d4cfd29e47c77f9b3db95db100e846ccd7ea9c130647d87c6bec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/scibet
  tools:
  - E-test
  - SciBet
  - scikit-learn (sklearn)
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# gene-ranking-and-top-k-selection

## Summary

Implements SciBet's E-test feature selection: compute the entropy-based ΔS statistic per gene over cell-type mean expression vectors, then rank genes by ΔS and keep the top 500 as informative genes for the multinomial classifier.

## When to use

Use when a labeled single-cell training matrix with cell type annotations needs supervised feature selection before building SciBet (or similar multinomial) classifiers.

## When NOT to use

- Input is unlabeled data with no cell type annotations — ΔS requires per-cell-type mean expression vectors from a supervised reference
- You need a full model or predictions rather than a feature ranking — use Learn/Bet or ExportModel instead of SelectGene alone
- Input is already a reduced feature table of preselected marker genes; re-running ΔS ranking would discard prior curation

## Inputs

- Labeled single-cell training expression matrix (e.g. 70% reference split of GSE104276) with cell type annotations
- Top-k count k (SciBet default: 500)
- Optional window and low parameters for Entropy(expr, window, low)

## Outputs

- informative_genes.csv: ranked list of exactly 500 gene identifiers with per-gene non-negative ΔS values
- Informative gene set passed to Learn(expr, geneset, k, a) for model training

## How to apply

Load the labeled expression matrix (e.g. the 70% stratified reference split of GSE104276) with cell type annotations and compute each cell type's mean expression vector per gene, giving n cell-type means per gene. For each gene i, calculate the E-test statistic ΔS_i = n * (ln(AM_i) − ln(GM_i)), where AM_i and GM_i are the arithmetic and geometric means of the n cell-type means; ΔS measures deviation of observed mean expression from the null hypothesis of equal expression across types. Rank genes in descending ΔS and select the top 500 (SciBet's default, via SelectGene(expr, k, r) with k=500) as informative genes. The rationale is that each cell type is represented by its simple mean expression vector, and larger ΔS marks more cell type-specific genes for downstream multinomial MLE classification. Write the ranked list with per-gene ΔS values to informative_genes.csv. Gene significance can additionally be assessed by random permutation of cell group labels.

## Related tools

- **SciBet** (Provides SelectGene(expr, k, r) and Entropy(expr, window, low) functions implementing the E-test ΔS ranking and top-k selection (k=500 default)) — https://github.com/PaulingLiu/scibet
- **E-test** (The entropy-based statistic ΔS = n*(ln(AM) − ln(GM)) computed over per-cell-type mean expression vectors to score gene informativeness) — https://github.com/PaulingLiu/scibet
- **scikit-learn (sklearn)** (model_selection.train_test_split supplies the stratified 7:3 labeled training split on which ΔS is computed)

## Examples

```
devtools::install_github("PaulingLiu/scibet"); genes <- SelectGene(expr, k = 500)
```

## Evaluation signals

- informative_genes.csv exists and contains exactly 500 gene rows
- Each row has a gene identifier and a non-negative ΔS value
- Genes are sorted in descending ΔS order
- Selected gene count matches the k argument passed to SelectGene (k=500 by default)
- Permuting cell group labels should drive ΔS values toward the null, per the article's permutation-test significance assessment

## Limitations

- Significance of ΔS requires a random permutation test whose approximation strategy is only in Supplementary Note 2.4, not described in the main text
- SciBet's default of 500 informative genes is a fixed choice; no seed or randomization parameters are reported for repeated splits underlying the ranking context
- Gene selection presumes normalization appropriate to the data (TPM + log-normalization for full-length reads; Seurat v3 UMI size-factor 10,000 + log normalization)

## Evidence

- [discussion] we selected genes with top ΔS as informative genes (500 by default): "we selected genes with top ΔS as informative genes (500 by default)"
- [discussion] E-test and SciBet use mean expression vectors: "Both E-test and SciBet utilizes the fundamental concept that each cell type is represented by the simple mean expression vector"
- [discussion] Permutation test for ΔS significance: "The significance of ΔS for each gene can be obtained by the random permutation test"
- [methods] SelectGene filter step in methods: "- `SelectGene(expr, k, r)`"
- [readme] Repository installation via devtools: "devtools::install_github("PaulingLiu/scibet")"
