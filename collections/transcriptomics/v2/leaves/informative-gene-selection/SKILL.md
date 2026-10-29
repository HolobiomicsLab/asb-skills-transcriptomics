---
name: informative-gene-selection
description: Use when preparing a normalized scRNA-seq reference matrix for supervised
  cell-type classification (e.g., SciBet Learn/Bet) and a compact marker gene set
  is needed to discriminate close subtypes or rare populations.
when_to_use_negative:
- Input is raw UMI counts that have not yet been normalized (TPM+log or Seurat v3-style
  size-factor 10,000 normalization), since ΔS ranking assumes log-scale expression.
- The goal is unsupervised clustering or marker discovery on unlabeled cells — E-test
  ΔS requires pre-annotated cell group labels.
- Genes are already fixed by the experimental design (e.g., a targeted panel or a
  HomoloGene one-to-one orthologue gene space); selection would be redundant or invalid.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SciBet
  role: Provides SelectGene (E-test ΔS ranking), Entropy, and SEM functions for selecting
    the top 500 informative genes before model training with Learn.
  repo: github.com/PaulingLiu/scibet
- name: Python
  role: Drives the 50-repeat stratified 7:3 splitting so that gene selection by ΔS
    is repeated within each cross-validation split.
- name: scikit-learn (sklearn)
  role: model_selection.train_test_split generates the stratified 7:3 splits inside
    which per-split informative-gene selection is performed.
provenance:
  source_task_ids:
  - task_006
  - task_008
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:4e18e348e3cfee7e85e739922abd8ee47c7587103d92cf9748fd5cfe8e9a716f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HomoloGene
  - Python
  - SciBet
  - Tabula Muris
  - imbalanced-learn
  - scikit-learn (sklearn)
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# informative-gene-selection

## Summary

Ranks genes by the ΔS entropy statistic (E-test) and retains the top 500 informative genes per cell-type reference, providing the compact feature space on which SciBet's multinomial maximum-likelihood classifier operates.

## When to use

Use when preparing a normalized scRNA-seq reference matrix for supervised cell-type classification (e.g., SciBet Learn/Bet) and a compact marker gene set is needed to discriminate close subtypes or rare populations.

## When NOT to use

- Input is raw UMI counts that have not yet been normalized (TPM+log or Seurat v3-style size-factor 10,000 normalization), since ΔS ranking assumes log-scale expression.
- The goal is unsupervised clustering or marker discovery on unlabeled cells — E-test ΔS requires pre-annotated cell group labels.
- Genes are already fixed by the experimental design (e.g., a targeted panel or a HomoloGene one-to-one orthologue gene space); selection would be redundant or invalid.

## Inputs

- Normalized (log-scale) annotated scRNA-seq expression matrix with cell-type labels (dataframe: cells × genes)
- Number of top genes to retain k (default 500)
- Optional window/low parameters for Entropy(expr, window, low)

## Outputs

- Ranked list of informative genes by ΔS statistic (top 500 by default)
- Gene set passed to Learn() to build the multinomial reference model
- Optional per-gene significance scores from the random permutation test

## How to apply

After normalizing expression (TPM with pseudo-count 1 plus log-normalization for full-length data, or size-factor 10,000, add-one, log normalization for UMI data), apply the E-test by computing each gene's ΔS statistic, which measures entropy reduction between cells grouped by label versus ungrouped. Rank genes by ΔS and keep the top 500 by default (SelectGene(expr, k, r) in the SciBet R package). Genes are selected per training split — in the 50× repeated stratified 7:3 train/test benchmark, the top 500 informative genes are re-selected on every split, so selection must be embedded inside the cross-validation loop to avoid leakage. The significance of each gene's ΔS can be assessed by a random permutation test of cell group labels, and the SEM(ent_res) helper ranks E-test results for exploratory gene scoring. The rationale is that each cell type is represented by its mean expression vector over these informative genes, and classification reduces to assigning the label whose mean vector has minimal Kullback–Leibler divergence to the query cell.

## Related tools

- **SciBet** (Provides SelectGene (E-test ΔS ranking), Entropy, and SEM functions for selecting the top 500 informative genes before model training with Learn.) — github.com/PaulingLiu/scibet
- **Python** (Drives the 50-repeat stratified 7:3 splitting so that gene selection by ΔS is repeated within each cross-validation split.)
- **scikit-learn (sklearn)** (model_selection.train_test_split generates the stratified 7:3 splits inside which per-split informative-gene selection is performed.)

## Examples

```
if (!requireNamespace("devtools", quietly=TRUE)) install.packages("devtools"); devtools::install_github("PaulingLiu/scibet")
```

## Evaluation signals

- The selected gene set contains exactly k=500 genes ranked by ΔS by default, matching the paper's stated default.
- Gene selection is re-executed for each of the 50 training/test splits, not fixed across splits.
- Downstream confusion matrices discriminate close subtypes (CD4 memory vs CD4 regulatory T cells in the 2,500-cells-per-type PBMC design) and rare types (delta cells, 3.8% of pancreas cells), indicating the feature space retains discriminative signal.
- Permutation-test significance of ΔS can be computed for selected genes without errors, and SEM ranking returns a coherent gene ordering.
- Benchmark accuracy/balanced accuracy over the 14 datasets remains in [0,1] with SciBet highest on the majority of datasets, consistent with Fig. 2c/2d.

## Limitations

- No seed or randomization parameters are reported for the 50× repeated splits or permutation tests, so exact reproduction of a specific gene set may vary run to run.
- The permutation-test approximation strategy for ΔS significance is relegated to Supplementary Note 2.4 and is not fully described in the main text.
- ΔS assumes cell-type identity is captured by the mean expression vector; it may not capture probabilistic or trajectory-based identity signals.

## Evidence

- [methods] Methods — SelectGene signature: "SelectGene(expr, k, r)"
- [discussion] ΔS statistic and default gene count: "we selected genes with top ΔS as informative genes (500 by default)"
- [discussion] Permutation test for ΔS significance: "The significance of ΔS for each gene can be obtained by the random permutation test"
- [discussion] KL divergence rationale for mean-expression feature space: "we annotate the test cell to such the cell type that its mean expression vector of all marker genes has the minimal Kullback–Leibler divergence to the unknown cell y."
- [readme] README installation of the SciBet package providing SelectGene: "devtools::install_github("PaulingLiu/scibet")"
