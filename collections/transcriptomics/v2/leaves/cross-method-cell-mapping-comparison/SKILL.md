---
name: cross-method-cell-mapping-comparison
description: Use when comparing how cell-querying/mapping time of methods like Cell
  BLAST, scmap, and CellFishing.jl scales with reference size, from 1,000 to 1,000,000
  cells.
when_to_use_negative:
- The goal is annotation accuracy or cell-type transfer rather than querying-time
  scalability benchmarking
- Reference datasets are small (<1,000 cells), where the scaling regime being tested
  cannot be probed and models may under-train
- Only one mapping method is in scope — the skill requires timing at least the three
  methods at identical reference sizes for a fair comparison
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: Cell_BLAST
  role: Trains the DIRECTi generative model on each subsampled mouse brain reference
    and performs timed cell querying via Cell_BLAST.blast.BLAST against the Marques
    query cells
  repo: github.com/gao-lab/Cell_BLAST
- name: DIRECTi
  role: Deep generative model trained on each reference subsample to provide Cell
    BLAST's unbiased cell embedding before querying
  repo: github.com/gao-lab/Cell_BLAST
- name: scmap
  role: Baseline cell-mapping method timed on the same reference subsamples for the
    scalability comparison
- name: CellFishing.jl
  role: Baseline cell-mapping method timed on the same reference subsamples for the
    scalability comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:90aee8c0a167be87258c7e29a90fbc72de473a5af2049eecb094acdc34fe7c8d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cell BLAST
  - CellFishing.jl
  - Cell_BLAST
  - DIRECTi
  - scmap
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cross-method-cell-mapping-comparison

## Summary

Benchmark the wall-clock querying time of cell-mapping methods (Cell BLAST/DIRECTi, scmap, CellFishing.jl) as reference size grows, using subsampled references of the 1.3M mouse brain scRNA-seq dataset to reproduce the Fig. 2d scalability comparison.

## When to use

Use when comparing how cell-querying/mapping time of methods like Cell BLAST, scmap, and CellFishing.jl scales with reference size, from 1,000 to 1,000,000 cells.

## When NOT to use

- The goal is annotation accuracy or cell-type transfer rather than querying-time scalability benchmarking
- Reference datasets are small (<1,000 cells), where the scaling regime being tested cannot be probed and models may under-train
- Only one mapping method is in scope — the skill requires timing at least the three methods at identical reference sizes for a fair comparison

## Inputs

- 1.3M mouse brain scRNA-seq reference dataset (legacy h5 or h5ad/AnnData)
- Marques query dataset (h5ad/AnnData)
- Series of reference subsample sizes (1,000 to 1,000,000 cells)

## Outputs

- CSV table of method x reference size x querying time
- Plot of querying time versus reference size for Cell BLAST, scmap, and CellFishing.jl (Fig. 2d-style comparison curves)

## How to apply

Load the 1.3M mouse brain scRNA-seq reference and a query dataset (e.g., Marques), converting legacy h5 inputs to h5ad/AnnData with cb.data.h5_to_h5ad if needed. Subsample the reference to sizes spanning 1,000 to 1,000,000 cells. At each size, train the DIRECTi generative model on the reference and query with Cell_BLAST.blast.BLAST, timing wall-clock querying; repeat identically with scmap and CellFishing.jl. Aggregate timings into a method x reference size x querying-time CSV, then plot querying time versus reference size to reproduce the Fig. 2d comparison curves. Timing methodology must be identical across methods so the growth curves are comparable.

## Related tools

- **Cell_BLAST** (Trains the DIRECTi generative model on each subsampled mouse brain reference and performs timed cell querying via Cell_BLAST.blast.BLAST against the Marques query cells) — github.com/gao-lab/Cell_BLAST
- **DIRECTi** (Deep generative model trained on each reference subsample to provide Cell BLAST's unbiased cell embedding before querying) — github.com/gao-lab/Cell_BLAST
- **scmap** (Baseline cell-mapping method timed on the same reference subsamples for the scalability comparison)
- **CellFishing.jl** (Baseline cell-mapping method timed on the same reference subsamples for the scalability comparison)

## Examples

```
pip install Cell-BLAST && python -c "import Cell_BLAST as cb; print(cb.__version__)"
```

## Evaluation signals

- Timing rows exist for all three methods at every reference size from 1,000 to 1,000,000 cells (no missing cells in the method x size grid)
- CSV follows a method x reference size x querying-time schema with numeric wall-clock timings
- Plotted curves qualitatively match Fig. 2d: scmap's querying time rises dramatically after 10,000 reference cells, while Cell BLAST and CellFishing.jl scale well with increasing reference size
- All legacy h5 inputs were successfully converted to h5ad/AnnData (cb.data.h5_to_h5ad) before querying
- Same reference subsamples and query dataset are used for each method so timing differences reflect the methods, not the data

## Limitations

- scmap's querying time rises dramatically after 10,000 reference cells, so it cannot be fairly timed at the largest reference sizes without extreme runtimes
- The Cell BLAST model is over-parameterized and requires a reasonable number of samples to train properly; datasets with fewer than 1000 cells may suffer from under-training issues
- Legacy h5 inputs must first be converted to h5ad (h5_to_h5ad.py) before Cell BLAST can consume them

## Evidence

- [readme] Cell BLAST is a cell querying tool for single-cell transcriptomics data.: "Cell BLAST is a cell querying tool for single-cell transcriptomics data."
- [readme] To reproduce results, please check out the rep branch.: [evidence span withheld — anchor status: external_doc]
- [methods] cb.data.h5_to_h5ad(inputfilename, outputfilename): "cb.data.h5_to_h5ad(inputfilename, outputfilename)"
- [discussion] Cell BLAST is an accurate and robust querying algorithm for heterogeneous single-cell transcriptome datasets.: "Cell BLAST is an accurate and robust querying algorithm for heterogeneous single-cell transcriptome datasets."
- [discussion] a cell number of 3000 is generally sufficient for good performance, while small-sized datasets with fewer than 1000 cells may suffer from under-training issues.: "a cell number of 3000 is generally sufficient for good performance, while small-sized datasets with fewer than 1000 cells may suffer from under-training issues."
