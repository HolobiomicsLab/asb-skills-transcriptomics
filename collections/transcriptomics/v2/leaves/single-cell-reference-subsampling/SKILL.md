---
name: single-cell-reference-subsampling
description: Use when evaluating whether a cell-querying/annotation method remains
  tractable as the reference dataset grows to hundreds of thousands of cells, by timing
  queries across a range of subsampled reference sizes.
when_to_use_negative:
- The question is about annotation accuracy rather than computational scalability
  — accuracy benchmarking is evaluated separately in the article
- 'The reference dataset is small (< 1,000 cells): the article notes small-sized datasets
  with fewer than 1000 cells may suffer from under-training issues of the over-parameterized
  DIRECTi model'
- Reference cells are already embedded/searched via an index and no comparative timing
  across methods is needed
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cell BLAST
  role: Cell-querying method whose querying time is benchmarked across subsampled
    reference sizes, using the DIRECTi generative model
  repo: github.com/gao-lab/Cell_BLAST
- name: Cell_BLAST
  role: Python package used to convert h5 data (cb.data.h5_to_h5ad), train DIRECTi,
    and run Cell_BLAST.blast.BLAST queries with timing
  repo: github.com/gao-lab/Cell_BLAST
- name: DIRECTi
  role: Deep generative model trained on each subsampled reference before Cell BLAST
    querying
- name: scmap
  role: Baseline querying method timed at each reference size; its querying time rises
    dramatically after 10,000 reference cells
- name: CellFishing.jl
  role: Baseline querying method timed at each reference size; scales well with increasing
    reference size
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:3a5c4a1fb21c420c84e595f67b9bda88bbfc2ef749484fcfdd228e76971fa121
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cell BLAST
  - CellFishing.jl
  - Cell_BLAST
  - DIRECTi
  - scmap
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# single-cell-reference-subsampling

## Summary

Benchmark how cell-querying methods (Cell BLAST/DIRECTi, scmap, CellFishing.jl) scale by subsampling a 1.3M mouse brain scRNA-seq reference from 1,000 to 1,000,000 cells and recording wall-clock querying time against the Marques query dataset, reproducing the Fig. 2d scalability comparison.

## When to use

Use when evaluating whether a cell-querying/annotation method remains tractable as the reference dataset grows to hundreds of thousands of cells, by timing queries across a range of subsampled reference sizes.

## When NOT to use

- The question is about annotation accuracy rather than computational scalability — accuracy benchmarking is evaluated separately in the article
- The reference dataset is small (< 1,000 cells): the article notes small-sized datasets with fewer than 1000 cells may suffer from under-training issues of the over-parameterized DIRECTi model
- Reference cells are already embedded/searched via an index and no comparative timing across methods is needed

## Inputs

- 1.3M mouse brain scRNA-seq reference dataset (h5 or h5ad/AnnData)
- Marques query dataset
- Series of subsampled reference sizes from 1,000 to 1,000,000 cells

## Outputs

- CSV table of querying time by method and reference size
- Figure plotting querying time versus reference size for Cell BLAST, scmap, and CellFishing.jl (Fig. 2d reproduction)

## How to apply

Load the 1.3M mouse brain scRNA-seq reference and the Marques query dataset, converting legacy h5 inputs to h5ad/AnnData with cb.data.h5_to_h5ad where needed. Subsample the reference to a series of sizes spanning 1,000 to 1,000,000 cells; for Cell BLAST, at each size train the DIRECTi deep generative model on the reference cells and query Marques cells via Cell_BLAST.blast.BLAST while recording wall-clock querying time. Repeat the identical querying procedure with scmap and CellFishing.jl at every reference size so timings are directly comparable across methods. Aggregate timings into a method x reference size x querying time table (CSV) and plot querying time versus reference size to reproduce the Fig. 2d curves. Note that scmap's querying time rises sharply after 10,000 reference cells, while Cell BLAST and CellFishing.jl scale well, so the expected ordering of curves is the key qualitative check. Check out the `rep` branch of gao-lab/Cell_BLAST for reproduction scripts.

## Related tools

- **Cell BLAST** (Cell-querying method whose querying time is benchmarked across subsampled reference sizes, using the DIRECTi generative model) — github.com/gao-lab/Cell_BLAST
- **Cell_BLAST** (Python package used to convert h5 data (cb.data.h5_to_h5ad), train DIRECTi, and run Cell_BLAST.blast.BLAST queries with timing) — github.com/gao-lab/Cell_BLAST
- **DIRECTi** (Deep generative model trained on each subsampled reference before Cell BLAST querying)
- **scmap** (Baseline querying method timed at each reference size; its querying time rises dramatically after 10,000 reference cells)
- **CellFishing.jl** (Baseline querying method timed at each reference size; scales well with increasing reference size)

## Examples

```
import Cell_BLAST as cb; cb.data.h5_to_h5ad(inputfilename, outputfilename)
```

## Evaluation signals

- Timing rows exist for all three methods (Cell BLAST, scmap, CellFishing.jl) at every reference size from 1,000 to 1,000,000 cells
- Plotted querying-time curves qualitatively match the method ordering and trends of Fig. 2d
- scmap timing shows a dramatic rise after 10,000 reference cells while Cell BLAST and CellFishing.jl scale well
- Reference subsamples correctly span the 1,000 to 1,000,000 cell range derived from the 1.3M mouse brain dataset
- All legacy h5 inputs are successfully converted to h5ad/AnnData before querying

## Limitations

- Integrating datasets at mega-scale may cause over-correcting of batches with highly skewed cell-type composition
- The Cell BLAST model is over-parameterized and requires a reasonable number of samples; small datasets with fewer than 1000 cells may suffer from under-training issues
- Timings are wall-clock and specific to this hardware/software setup, so absolute values may not transfer across systems

## Evidence

- [results] scmap's querying time rises dramatically after 10,000 reference cells, while Cell BLAST and CellFishing.jl scale well with increasing reference size.: [evidence span withheld — anchor status: not_found]
- [methods] Legacy h5 to h5ad conversion command: "cb.data.h5_to_h5ad(inputfilename, outputfilename)"
- [discussion] Cell BLAST accuracy and robustness claim grounding the choice of Cell BLAST as benchmarked method: "Cell BLAST is an accurate and robust querying algorithm for heterogeneous single-cell transcriptome datasets."
- [discussion] Under-training limitation for small reference sizes: [evidence span withheld — anchor status: not_found]
- [readme] Repository instructions for reproducing paper results: [evidence span withheld — anchor status: external_doc]
