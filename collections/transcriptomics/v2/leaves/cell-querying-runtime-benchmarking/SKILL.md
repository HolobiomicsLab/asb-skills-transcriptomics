---
name: cell-querying-runtime-benchmarking
description: Use when reproducing or extending runtime scalability benchmarks of cell-querying
  methods (Cell BLAST, scmap, CellFishing.jl) against reference datasets spanning
  1,000 to 1,000,000 cells.
when_to_use_negative:
- When the goal is cell-type annotation accuracy rather than runtime scalability of
  the querying step
- When the reference dataset has fewer than 1,000 cells — the article notes small-sized
  datasets may suffer from under-training issues of the DIRECTi model
- When comparing methods on tasks other than cell querying (e.g. scVI-style integration,
  BLAST2CO ontology curation), where runtime scaling of the querying step is not the
  question
tools:
- name: Cell BLAST
  role: Cell-querying method under benchmark; queries Marques cells via Cell_BLAST.blast.BLAST
    at each reference size
  repo: github.com/gao-lab/Cell_BLAST
- name: DIRECTi
  role: Neural network-based generative model trained on each subsampled reference
    before Cell BLAST querying
  repo: github.com/gao-lab/Cell_BLAST
- name: scmap
  role: Baseline cell-querying method whose querying time is recorded at each reference
    size
- name: CellFishing.jl
  role: Baseline cell-querying method whose querying time is recorded at each reference
    size
- name: Cell_BLAST
  role: Python package providing data conversion (cb.data.h5_to_h5ad) and the BLAST
    querying API used for timing
  repo: github.com/gao-lab/Cell_BLAST
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:7e650ad3248003faa708ad073a48f5d37c3221cc1f6b0647c1cd594a2b26ab56
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cell BLAST
  - CellFishing.jl
  - Cell_BLAST
  - DIRECTi
  - scmap
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cell-querying-runtime-benchmarking

## Summary

Benchmark how cell-querying runtimes (Cell BLAST via DIRECTi, scmap, CellFishing.jl) scale as the scRNA-seq reference grows from 1,000 to 1,000,000 cells, producing a method x reference-size timing table and Fig. 2d-style scaling curves for the 1.3M mouse brain reference.

## When to use

Use when reproducing or extending runtime scalability benchmarks of cell-querying methods (Cell BLAST, scmap, CellFishing.jl) against reference datasets spanning 1,000 to 1,000,000 cells.

## When NOT to use

- When the goal is cell-type annotation accuracy rather than runtime scalability of the querying step
- When the reference dataset has fewer than 1,000 cells — the article notes small-sized datasets may suffer from under-training issues of the DIRECTi model
- When comparing methods on tasks other than cell querying (e.g. scVI-style integration, BLAST2CO ontology curation), where runtime scaling of the querying step is not the question

## Inputs

- 1.3M mouse brain scRNA-seq reference dataset (legacy h5, convertible to h5ad/AnnData)
- Marques query dataset (scRNA-seq)
- Reference-size series for subsampling (1,000 to 1,000,000 cells)
- Querying implementations: Cell BLAST (Cell_BLAST.blast.BLAST + DIRECTi), scmap, CellFishing.jl

## Outputs

- CSV table of method x reference size x wall-clock querying time
- Plot of querying time versus reference size for Cell BLAST, scmap, and CellFishing.jl (Fig. 2d-style comparison curves)

## How to apply

Load the 1.3M mouse brain scRNA-seq reference and the Marques query dataset, converting legacy h5 inputs to h5ad/AnnData with cb.data.h5_to_h5ad where needed. Subsample the mouse brain reference to a series of sizes spanning 1,000 to 1,000,000 cells. At each reference size, train the DIRECTi generative model on the reference cells, query the Marques cells with Cell_BLAST.blast.BLAST, and record wall-clock querying time; repeat the identical querying procedure with scmap and CellFishing.jl. Aggregate timings into a CSV table of method x reference size x querying time, then plot querying time versus reference size to reproduce the Fig. 2d comparison curves. Judging correctness: timings must exist for all three methods at every reference size, and the curves must qualitatively match the reported ordering and trends (scmap degrades beyond ~10,000 reference cells; Cell BLAST and CellFishing.jl scale well).

## Related tools

- **Cell BLAST** (Cell-querying method under benchmark; queries Marques cells via Cell_BLAST.blast.BLAST at each reference size) — github.com/gao-lab/Cell_BLAST
- **DIRECTi** (Neural network-based generative model trained on each subsampled reference before Cell BLAST querying) — github.com/gao-lab/Cell_BLAST
- **scmap** (Baseline cell-querying method whose querying time is recorded at each reference size)
- **CellFishing.jl** (Baseline cell-querying method whose querying time is recorded at each reference size)
- **Cell_BLAST** (Python package providing data conversion (cb.data.h5_to_h5ad) and the BLAST querying API used for timing) — github.com/gao-lab/Cell_BLAST

## Examples

```
pip install Cell-BLAST; import Cell_BLAST as cb; print(cb.__version__)
```

## Evaluation signals

- Timing rows exist for all three methods (Cell BLAST, scmap, CellFishing.jl) at every reference size from 1,000 to 1,000,000 cells
- Plotted curves qualitatively match the method ordering and trends reported in Fig. 2d of the article
- scmap's querying time rises dramatically after 10,000 reference cells while Cell BLAST and CellFishing.jl scale well with increasing reference size
- Timing table is a complete method x reference size x querying-time grid saved as CSV
- Reference data successfully converted from legacy h5 to h5ad/AnnData (cb.data.h5_to_h5ad) before benchmarking

## Limitations

- scmap does not scale to large references: its querying time rises dramatically after 10,000 reference cells
- The DIRECTi model underlying Cell BLAST is over-parameterized and requires a reasonable number of samples to train properly; datasets with fewer than 1000 cells may suffer from under-training issues
- Results to reproduce live on the rep branch of the Cell_BLAST repository, so dev/main branches may not carry the benchmark code

## Evidence

- [other] scmap's querying time rises dramatically after 10,000 reference cells, while Cell BLAST and CellFishing.jl scale well with increasing reference size.: [evidence span withheld — anchor status: not_found]
- [readme] To reproduce results, please check out the `rep` branch.: [evidence span withheld — anchor status: external_doc]
- [methods] cb.data.h5_to_h5ad(inputfilename, outputfilename): "cb.data.h5_to_h5ad(inputfilename, outputfilename)"
- [discussion] a cell number of 3000 is generally sufficient for good performance, while small-sized datasets with fewer than 1000 cells may suffer from under-training issues.: "a cell number of 3000 is generally sufficient for good performance, while small-sized datasets with fewer than 1000 cells may suffer from under-training issues."
- [other] The following packages are required to build the documentation:: "The following packages are required to build the documentation:"
