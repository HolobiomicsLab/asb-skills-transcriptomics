---
name: immune-cell-state-classification-interpretation
description: Use when annotated or unannotated single-cell expression data (Seurat
  .rds, scanpy .h5ad, or 10x cellranger output) needs immune cell type/state labels,
  or when you need per-gene importance scores explaining which genes drive a cell-state
  classification.
when_to_use_negative:
- Input expression matrix is normalized rather than raw counts — pollock requires
  unnormalized counts in .X (scanpy) or @assays$RNA@counts (Seurat)
- Explaining very large numbers of cells at once — explain mode has excessive runtimes,
  so cells should be downsampled first
- The tissue or cell states of interest are not covered by available pretrained modules
  (scRNAseq_brca, cesc, hnscc, melanoma, mmy, pdac, generalized, brca_panimmune; snRNAseq
  and snATACseq variants) and no annotated training data exists to train a custom
  module
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Pollock
  role: Deep learning (VAE + linear classification head) tool run in train, predict,
    or explain mode to classify immune cell states and generate per-gene feature attribution
    scores
  repo: https://github.com/ding-lab/pollock
- name: captum
  role: Feature attribution library underpinning explain mode's per-gene interpretability
    scores
- name: scanpy
  role: Reads and preprocesses 10x data (read_10x_mtx), copies raw counts, and serves
    as the .h5ad input format for pollock from_scanpy runs
- name: PyTorch
  role: Deep learning backend installed in the pollock conda environment
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbac028
    title: 'Pollock: fishing for cell states'
  source_content_hash: sha256:09e981c09a9f6c951b00cbb28518a0abbdd79f2eb572ce1b8abffa2030fb4b35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ding-lab/pollock
  tools:
  - Pollock
  - PyTorch
  - captum
  - scanpy
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# immune-cell-state-classification-interpretation

## Summary

Classify immune cell states in scRNA-seq data with a pretrained Pollock VAE module and interpret the predictions via per-gene feature attribution scores to identify discriminatively weighted genes (DWGs) driving each state's classification.

## When to use

Use when annotated or unannotated single-cell expression data (Seurat .rds, scanpy .h5ad, or 10x cellranger output) needs immune cell type/state labels, or when you need per-gene importance scores explaining which genes drive a cell-state classification.

## When NOT to use

- Input expression matrix is normalized rather than raw counts — pollock requires unnormalized counts in .X (scanpy) or @assays$RNA@counts (Seurat)
- Explaining very large numbers of cells at once — explain mode has excessive runtimes, so cells should be downsampled first
- The tissue or cell states of interest are not covered by available pretrained modules (scRNAseq_brca, cesc, hnscc, melanoma, mmy, pdac, generalized, brca_panimmune; snRNAseq and snATACseq variants) and no annotated training data exists to train a custom module

## Inputs

- Pretrained Pollock module directory (e.g. scRNAseq_brca_panimmune from pretrained_models.tar.gz, Zenodo record 5895221)
- Annotated raw-count Seurat .rds or scanpy .h5ad object with cell type/state labels in metadata (.obs) column
- Background raw-count .rds or .h5ad object for explain-mode sampling
- 10x cellranger counts (mtx directory at outs/raw_feature_bc_matrix or .h5) for predict mode

## Outputs

- Predicted cell type/state labels and probabilities added to the input object metadata (or a tab-separated .txt file with --txt-output)
- VAE embedding UMAP coordinates incorporated into the predicted object
- Per-gene feature attribution/importance scores per predicted cell state
- Rank-ordered top DWGs per state (e.g. top 20) and downstream GO enrichment results (e.g. dwg_go_enrichment_per_state.tsv)

## How to apply

Download a pretrained module (e.g. scRNAseq_brca_panimmune, trained on an immune cell state breast cancer dataset) and matching annotated scRNA-seq data from Zenodo record 5895221 (pretrained_models.tar.gz, benchmarking_datasets.tar.gz). Run prediction or explanation with the pollock CLI: the input object must contain raw (unnormalized) counts — for scanpy, .X must be raw counts; for Seurat, @assays$RNA@counts. In explain mode, supply the module via --module-filepath, the annotated raw-count object via --explain-filepath, a background object via --background-filepath, and the label column via --predicted-key; set --background-sample-size 100 (default, sufficient in most cases) — ~100 explained cells with 100 background cells runs in ~15 minutes. Rank genes by feature attribution score per predicted state to obtain top discriminative genes (DWGs), then optionally run GO Biological Process over-representation (e.g. ToppFun) on the top 20 per state. Pollock is a VAE with a linear classification head; captum powers the feature attribution, and VAE embeddings are UMAP-transformed into the output object by default (disable with --no-umap for speed).

## Related tools

- **Pollock** (Deep learning (VAE + linear classification head) tool run in train, predict, or explain mode to classify immune cell states and generate per-gene feature attribution scores) — https://github.com/ding-lab/pollock
- **captum** (Feature attribution library underpinning explain mode's per-gene interpretability scores)
- **scanpy** (Reads and preprocesses 10x data (read_10x_mtx), copies raw counts, and serves as the .h5ad input format for pollock from_scanpy runs)
- **PyTorch** (Deep learning backend installed in the pollock conda environment)

## Examples

```
pollock explain from_scanpy --module-filepath <module_dir> --explain-filepath <annotated.h5ad> --background-filepath <background.h5ad> --background-sample-size 100 --predicted-key cell_type --output-prefix output
```

## Evaluation signals

- During prediction, log output reports gene overlap with the model (e.g. '12679 genes overlap with model after filtering'; missing-gene counts should be checked)
- Prediction log confirms cell count processed (e.g. 'starting prediction of 2638 cells') matches the input object
- Pretrained scRNAseq_brca_panimmune module predicts the expected immune state classes, including Treg, CD8 T cell, and NK
- Feature attribution results are concordant with expected biology: cell-cycle pathways for proliferating CD8 T cells, regulation of leukocyte activation for Tregs, cell activation/stress response for exhausted CD8 T cells, cell killing for NK cells
- Runtime check for explain mode: ~100 explained cells with background-sample-size 100 completes in roughly 15 minutes

## Limitations

- Explain mode has excessive runtimes for very large numbers of cells; downsampling is recommended
- Genes missing from the input dataset after filtering cannot contribute to classification (e.g. 4859 genes missing in the PBMC demo)
- 10x mode drops cells below --min-genes-per-cell (default 10)
- Some cell states require the disease/state-specific module (scRNAseq_brca_panimmune) rather than the generalized module

## Evidence

- [intro] Pollock is a deep learning method for single cell classification: "Pollock is a deep learning method for single cell classification."
- [methods] VAE architecture: "At it's core, pollock is a variational autoencoder (VAE) with a linear classification head."
- [methods] pretrained models on Zenodo: "The models and training datasets can be found on Zenodo at https://zenodo.org/record/5895221 in pretrained_models.tar.gz and benchmarking_datasets.tar.gz"
- [readme] raw counts requirement: "scanpy expression matrix (.X attribute in the anndata object) must be raw expression counts (i.e. not normalized)"
- [readme] background sampling default: "Number of cells to sample as background samples. The default of 100 cells is sufficient in most use cases."
- [readme] explain runtime guidance: "running ~100 cells with a background sample size of ~100 cells results in a runtime of approximately 15 minutes"
- [readme] available immune-state module: "scRNAseq_brca_panimmune (trained with specific immune cell state breast cancer dataset)"
- [abstract] interpretability scores: "reports interpretability scores that identify the genes that drive cell type classifications"
