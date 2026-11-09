---
name: deep-generative-embedding-training
description: Use when scRNA-seq datasets (h5ad/AnnData) must be embedded into a low-dimensional
  space that removes batch effect while preserving cell type structure, e.g. as the
  embedding step before BLAST querying or batch-correction benchmarks.
when_to_use_negative:
- Datasets with fewer than ~1000 cells, which the paper warns may suffer from under-training
  issues for neural network-based models
- Mega-scale dataset integration with highly skewed cell-type composition, where the
  authors note over-correcting of batches may occur
- Input data is already a corrected/embedded representation and the goal is only similarity
  search rather than learning an unbiased embedding
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cell BLAST
  role: Package providing the DIRECTi deep generative model used to train unbiased
    cell embeddings that model multilevel batch effect
  repo: github.com/gao-lab/Cell_BLAST
- name: DIRECTi
  role: The deep generative model itself; trained on the scRNA-seq dataset to produce
    low-dimensional cell embeddings
- name: Cell_BLAST
  role: Python import namespace (import Cell_BLAST as cb) used for installation checks
    and h5_to_h5ad data migration before training
  repo: github.com/gao-lab/Cell_BLAST
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:70bb407230d70e2c6132e244a947ed6bf00a6ad4f87d97db6627b82ba855111a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cell BLAST
  - Cell_BLAST
  - DIRECTi
  - scVI
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# deep-generative-embedding-training

## Summary

Train a DIRECTi deep generative model (Cell BLAST's neural network-based generative model) on scRNA-seq data to obtain unbiased low-dimensional cell embeddings that explicitly model multilevel batch effect, enabling downstream cell querying, dataset mixing, and label transfer.

## When to use

Use when scRNA-seq datasets (h5ad/AnnData) must be embedded into a low-dimensional space that removes batch effect while preserving cell type structure, e.g. as the embedding step before BLAST querying or batch-correction benchmarks.

## When NOT to use

- Datasets with fewer than ~1000 cells, which the paper warns may suffer from under-training issues for neural network-based models
- Mega-scale dataset integration with highly skewed cell-type composition, where the authors note over-correcting of batches may occur
- Input data is already a corrected/embedded representation and the goal is only similarity search rather than learning an unbiased embedding

## Inputs

- scRNA-seq datasets as h5ad (AnnData) matrices with gene expression, cell type labels, and batch labels (optionally migrated from legacy .h5 via cb.data.h5_to_h5ad)
- Batch covariate annotations (e.g. dataset/run-level batch variables) to model multilevel batch effect

## Outputs

- Low-dimensional cell embeddings from the trained DIRECTi model (per-cell latent representation)
- Trained DIRECTi/Cell BLAST model usable for posterior-distribution-based cell-to-cell similarity querying (BLAST)

## How to apply

Install Cell BLAST from PyPI (`pip install Cell-BLAST`) in a Python 3.9 conda/virtualenv and verify the import (`import Cell_BLAST as cb`). Convert legacy data if needed with `cb.data.h5_to_h5ad()` or the h5_to_h5ad.py migration script, then fit the DIRECTi deep generative model on the input dataset to learn low-dimensional cell embeddings. DIRECTi differs from canonical variational autoencoders (scVI) by not enforcing a Gaussian diagonal-covariance variational posterior, and it models multilevel batch effect (both cross-dataset and within-dataset batch variation) plus uncertainty in cell-to-cell similarity estimation. Ensure a reasonable number of training cells — the paper reports ~3000 cells is generally sufficient, while datasets with fewer than 1000 cells risk under-training — and use the learned embedding as input to posterior-distance-based cell querying or as the corrected representation compared against ComBat, MNN, CCA, Harmony, and scVI. The paper's reproduction scripts live in the `rep` branch of gao-lab/Cell_BLAST.

## Related tools

- **Cell BLAST** (Package providing the DIRECTi deep generative model used to train unbiased cell embeddings that model multilevel batch effect) — github.com/gao-lab/Cell_BLAST
- **DIRECTi** (The deep generative model itself; trained on the scRNA-seq dataset to produce low-dimensional cell embeddings)
- **Cell_BLAST** (Python import namespace (import Cell_BLAST as cb) used for installation checks and h5_to_h5ad data migration before training) — github.com/gao-lab/Cell_BLAST

## Examples

```
conda create -n cb python=3.9 && conda activate cb && pip install Cell-BLAST; python -c "import Cell_BLAST as cb; cb.data.h5_to_h5ad('input.h5','output.h5ad')"
```

## Evaluation signals

- The package imports successfully in the Python interpreter (`import Cell_BLAST as cb; print(cb.__version__)`) as the README instructs
- Trained embeddings should mix batches (measurable via Seurat alignment score on corrected embeddings) while preserving cell-type resolution (mean average precision from posterior-distance similarity), with Cell BLAST comparable to the Fig. 2a ranking
- In benchmark reproduction, the results table (batch_benchmark_results.csv) contains entries for all eight methods across dataset combinations and the within-dataset multilevel batch condition
- Training set size check: ~3000 cells is generally sufficient; flag datasets under 1000 cells as at risk of under-training
- Legacy .h5 inputs converted to h5ad successfully via cb.data.h5_to_h5ad or h5_to_h5ad.py before training

## Limitations

- Cell BLAST, like all neural network-based models, is over-parameterized and requires a reasonable number of samples to train properly; datasets with fewer than 1000 cells may suffer from under-training issues (3000 cells generally sufficient)
- Integrating datasets at mega-scale may incur over-correcting of batches with highly skewed cell-type composition

## Evidence

- [abstract] DIRECTi deep generative model produces unbiased cell embeddings: "Cell BLAST, an accurate and robust cell-querying method built on a neural network-based generative model and a customized cell-to-cell similarity metric"
- [methods] DIRECTi named in methods: "a deep generative model (currently named **DIRECTi**)"
- [methods] Installation via pip: "pip install Cell-BLAST"
- [methods] h5 to h5ad conversion step: "cb.data.h5_to_h5ad(inputfilename, outputfilename)"
- [discussion] Multilevel batch effect modeling rationale: "By explicitly modeling multilevel batch effect as well as uncertainty in cell-to-cell similarity estimation, Cell BLAST is an accurate and robust querying algorithm"
- [discussion] Under-training limitation: [evidence span withheld — anchor status: not_found]
- [readme] README conda setup: [evidence span withheld — anchor status: external_doc]
