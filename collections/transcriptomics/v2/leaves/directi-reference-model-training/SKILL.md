---
name: directi-reference-model-training
description: Use when a raw or subsampled scRNA-seq reference dataset (AnnData/h5ad,
  or legacy h5) must be embedded with DIRECTi before building a Cell BLAST BLAST index
  for cell querying.
when_to_use_negative:
- Reference datasets with fewer than ~1000 cells, which may suffer from under-training
  of the over-parameterized neural network model
- Mega-scale integration where highly skewed cell-type composition risks over-correcting
  batches
- When you only need clustering/annotation of a single dataset without querying against
  a reference — Cell BLAST's similarity metric is designed for cross-dataset cell
  querying
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cell BLAST
  role: Framework whose DIRECTi generative model is trained on the reference to enable
    posterior-distance-based cell querying
  repo: https://github.com/gao-lab/Cell_BLAST
- name: DIRECTi
  role: The neural network-based deep generative model fitted on reference cells;
    handles batch effect and defines the cell-to-cell similarity metric
  repo: https://github.com/gao-lab/Cell_BLAST
- name: Cell_BLAST
  role: Python package providing Cell_BLAST.directi for model fitting, Cell_BLAST.blast.BLAST
    for index building/querying, and cb.data.h5_to_h5ad for format migration
  repo: https://github.com/gao-lab/Cell_BLAST
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:cf5839e4ee618bda49008612458304843269504622507fded48f50056110c6c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gao-lab/Cell_BLAST
  tools:
  - Cell BLAST
  - CellFishing.jl
  - Cell_BLAST
  - DIRECTi
  - scmap
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# directi-reference-model-training

## Summary

Train a DIRECTi deep generative model on a scRNA-seq reference dataset to produce unbiased cell embeddings that power Cell BLAST's posterior-distance-based cell querying and downstream annotation transfer.

## When to use

Use when a raw or subsampled scRNA-seq reference dataset (AnnData/h5ad, or legacy h5) must be embedded with DIRECTi before building a Cell BLAST BLAST index for cell querying.

## When NOT to use

- Reference datasets with fewer than ~1000 cells, which may suffer from under-training of the over-parameterized neural network model
- Mega-scale integration where highly skewed cell-type composition risks over-correcting batches
- When you only need clustering/annotation of a single dataset without querying against a reference — Cell BLAST's similarity metric is designed for cross-dataset cell querying

## Inputs

- scRNA-seq reference dataset (AnnData/h5ad; legacy h5 accepted after cb.data.h5_to_h5ad conversion)
- Optional batch covariates in the reference for multilevel batch-effect modeling

## Outputs

- Trained DIRECTi generative model over the reference cells
- Cell BLAST BLAST index built from the model (via Cell_BLAST.blast.BLAST) ready for querying

## How to apply

Install Cell BLAST via pip inside a dedicated conda environment (python=3.9) and import it as Cell_BLAST. Convert any legacy h5 references to AnnData first with cb.data.h5_to_h5ad (or the h5_to_h5ad.py migration script) before training. Fit the DIRECTi generative model on the reference cells with Cell_BLAST.directi; the model explicitly handles multilevel batch effect and yields posterior distributions per cell, which is the basis for Cell BLAST's cell-to-cell similarity metric (distance between posteriors) used by Cell_BLAST.blast.BLAST. The trained model is then passed to the BLAST index builder for query-based cell-type transfer (e.g., benchmarking against scmap's correlation-based mapping or CellFishing.jl's LSH Hamming distance). Judgment of correct application: query-based cell typing should reach competitive mean balanced accuracy, including high negative-query MBA (specificity), and querying time should scale well up to ~1M-cell references.

## Related tools

- **Cell BLAST** (Framework whose DIRECTi generative model is trained on the reference to enable posterior-distance-based cell querying) — https://github.com/gao-lab/Cell_BLAST
- **DIRECTi** (The neural network-based deep generative model fitted on reference cells; handles batch effect and defines the cell-to-cell similarity metric) — https://github.com/gao-lab/Cell_BLAST
- **Cell_BLAST** (Python package providing Cell_BLAST.directi for model fitting, Cell_BLAST.blast.BLAST for index building/querying, and cb.data.h5_to_h5ad for format migration) — https://github.com/gao-lab/Cell_BLAST

## Examples

```
conda create -n cb python=3.9 && conda activate cb && pip install Cell-BLAST; import Cell_BLAST as cb; model = cb.directi.fit_DIRECTi(ref_data)
```

## Evaluation signals

- Query-based cell typing achieves high mean balanced accuracy across benchmark dataset groups, including correct rejection of negative queries (high negative-query MBA/specificity)
- import Cell_BLAST as cb; print(cb.__version__) succeeds in the environment, confirming correct installation
- The trained model plus BLAST index reproduces scaling behavior where querying time grows acceptably with reference size (up to ~1,000,000 cells subsampled from the 1.3M mouse brain dataset)
- Reference inputs load as AnnData; any legacy h5 files were converted without error via cb.data.h5_to_h5ad

## Limitations

- The model is over-parameterized (like all neural network-based models) and requires a reasonable number of samples to train properly
- A cell number of 3000 is generally sufficient for good performance; datasets with fewer than 1000 cells may suffer from under-training issues
- Only pip installation is supported for the Python package

## Evidence

- [methods] a deep generative model (currently named DIRECTi): "a deep generative model (currently named **DIRECTi**)"
- [discussion] Model over-parameterization requires sufficient samples for training: "the Cell BLAST model, like all neural network-based models, is over-parameterized, and requires a reasonable number of samples to train properly."
- [discussion] Guidance on minimum cell numbers for training: "a cell number of 3000 is generally sufficient for good performance, while small-sized datasets with fewer than 1000 cells may suffer from under-training issues."
- [readme] Package install and import check: "pip install Cell-BLAST"
- [methods] Legacy data conversion step before training: "cb.data.h5_to_h5ad(inputfilename, outputfilename)"
- [discussion] Rationale for the generative-model-based similarity metric: "By explicitly modeling multilevel batch effect as well as uncertainty in cell-to-cell similarity estimation, Cell BLAST is an accurate and robust querying algorithm"
