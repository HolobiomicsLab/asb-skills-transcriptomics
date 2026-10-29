---
name: generative-model-training-single-cell
description: Use when you have a reference scRNA-seq dataset (h5ad/AnnData) and need
  unbiased low-dimensional cell embeddings robust to batch effect, as input for Cell
  BLAST querying, benchmarking, or annotation transfer.
when_to_use_negative:
- The reference dataset is very small (fewer than 1000 cells) — the over-parameterized
  neural network model may suffer from under-training; ~3000 cells is generally sufficient
- The goal is integrating datasets at mega-scale with highly skewed cell-type composition,
  where batch over-correction may occur
- You only need a generic dimension-reduction result (e.g. plain PCA visualization)
  and will not use the model for cell-to-cell similarity querying or annotation transfer
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Cell BLAST
  role: Framework providing the DIRECTi training API (Cell_BLAST.directi), data loading
    (Cell_BLAST.data), and BLAST querying built on the fitted model
  repo: github.com/gao-lab/Cell_BLAST
- name: DIRECTi
  role: The deep generative model (neural network-based) fitted on the reference dataset
    to learn low-dimensional cell embeddings
  repo: github.com/gao-lab/Cell_BLAST
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:9f2abf59af75b3fe9236fe92cf3637b89febb846fc77b7d1eb9c9fb48f732184
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cell BLAST
  - Cell_BLAST.blast
  - Cell_BLAST.blast.BLAST
  - Cell_BLAST.data
  - Cell_BLAST.directi
  - Cell_BLAST.latent
  - DCA
  - DIRECTi
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# generative-model-training-single-cell

## Summary

Train the DIRECTi deep generative model (Cell BLAST's neural network-based variational model) on a scRNA-seq reference dataset to learn low-dimensional, batch-corrected cell embeddings that support downstream cell-querying and annotation.

## When to use

Use when you have a reference scRNA-seq dataset (h5ad/AnnData) and need unbiased low-dimensional cell embeddings robust to batch effect, as input for Cell BLAST querying, benchmarking, or annotation transfer.

## When NOT to use

- The reference dataset is very small (fewer than 1000 cells) — the over-parameterized neural network model may suffer from under-training; ~3000 cells is generally sufficient
- The goal is integrating datasets at mega-scale with highly skewed cell-type composition, where batch over-correction may occur
- You only need a generic dimension-reduction result (e.g. plain PCA visualization) and will not use the model for cell-to-cell similarity querying or annotation transfer

## Inputs

- Reference scRNA-seq dataset in h5ad (AnnData) format (convertible from legacy h5 via cb.data.h5_to_h5ad or the h5_to_h5ad.py migration script)
- Optionally a query scRNA-seq dataset (h5ad) to embed with the fitted model

## Outputs

- Fitted DIRECTi deep generative model object (from Cell_BLAST.directi.fit_DIRECTi)
- Low-dimensional posterior cell embeddings for reference (and query) cells
- A Cell_BLAST.blast.BLAST object built from the fitted model for cell querying

## How to apply

Install Cell BLAST via pip (`pip install Cell-BLAST`) inside a Python 3.9 conda environment, and verify `import Cell_BLAST as cb`. Load the reference scRNA-seq data in h5ad (AnnData) format with `Cell_BLAST.data` (migrating legacy h5 files with `cb.data.h5_to_h5ad` if needed). Fit the DIRECTi deep generative model on the reference dataset using `Cell_BLAST.directi.fit_DIRECTi`; unlike the canonical variational autoencoder used by scVI, DIRECTi's customized posterior explicitly models multilevel batch effect and uncertainty in cell-to-cell similarity estimation. Use the learned low-dimensional posterior embeddings as the representation for downstream steps (e.g. `Cell_BLAST.blast.BLAST` querying with posterior-distribution distance, or MAP benchmarking against DCA, PCA, tSNE). The rationale is that existing metrics like approximate cosine distance or LSH Hamming distance do not faithfully reflect cell-to-cell similarity under nonbiological confounding variation, so a model-specific embedding trained on the reference is required.

## Related tools

- **Cell BLAST** (Framework providing the DIRECTi training API (Cell_BLAST.directi), data loading (Cell_BLAST.data), and BLAST querying built on the fitted model) — github.com/gao-lab/Cell_BLAST
- **DIRECTi** (The deep generative model (neural network-based) fitted on the reference dataset to learn low-dimensional cell embeddings) — github.com/gao-lab/Cell_BLAST

## Examples

```
conda create -n cb python=3.9 && conda activate cb && pip install Cell-BLAST; import Cell_BLAST as cb; model = cb.directi.fit_DIRECTi(ref_adata)
```

## Evaluation signals

- The fitted model importably produces posterior embeddings via Cell_BLAST.latent / the fitted DIRECTi object from Cell_BLAST.directi.fit_DIRECTi
- Package import check passes: `import Cell_BLAST as cb; print(cb.__version__)` in the conda environment
- In the dimension-reduction benchmark, MAP (K = 1% of total cells, computed via Cell_BLAST.blast.BLAST with posterior-distribution distance) ranks Cell BLAST/DIRECTi with DCA, PCA, and tSNE among the best methods, matching Supplementary Table 1/2 and Supplementary Fig. 3
- A BLAST object built from the fitted model yields biologically consistent annotation transfer (e.g. 0.873 MBA in the ionocyte case study)

## Limitations

- The model is over-parameterized and requires a reasonable number of samples to train properly; ~3000 cells is generally sufficient while datasets with fewer than 1000 cells may suffer from under-training
- Integrating datasets at mega-scale may over-correct batches with highly skewed cell-type composition
- Only pip installation is currently supported (per README); editable dev install uses `flit install -s`

## Evidence

- [methods] directi generative model: "a deep generative model (currently named **DIRECTi**)"
- [intro] batch-effect handling rationale: "we introduce Cell BLAST, a cell-querying tool employing a customized neural network-based generative model that effectively handles batch effect"
- [discussion] posterior model advantage: "By explicitly modeling multilevel batch effect as well as uncertainty in cell-to-cell similarity estimation, Cell BLAST is an accurate and robust querying algorithm"
- [readme] install command: "`pip install Cell-BLAST`"
- [discussion] small-dataset caveat: [evidence span withheld — anchor status: not_found]
