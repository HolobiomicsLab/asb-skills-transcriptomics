---
name: query-latency-curve-visualization
description: Use when comparing the querying-time scalability of cell-querying tools
  across increasing reference sizes, e.g. validating that a method scales to 1,000,000-cell
  references before deployment on large scRNA-seq databases.
when_to_use_negative:
- The goal is accuracy or cell-type annotation quality rather than querying-time scalability
  benchmarking
- Reference datasets contain fewer than ~1,000 cells, where neural network-based models
  like DIRECTi may be under-trained and timings are not meaningful for scalability
  testing
- The comparison targets batch correction or integration quality instead of computational
  efficiency
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cell BLAST
  role: Querying method under test; DIRECTi model is trained on each subsampled reference
    and Cell_BLAST.blast.BLAST performs the timed cell queries
  repo: https://github.com/gao-lab/Cell_BLAST
- name: scmap
  role: Comparison method whose querying time is recorded at each reference size;
    known to rise dramatically after 10,000 reference cells
- name: CellFishing.jl
  role: Comparison method whose querying time is recorded at each reference size;
    scales well like Cell BLAST
- name: DIRECTi
  role: Deep generative model trained on each subsampled mouse brain reference prior
    to Cell BLAST queries
  repo: https://github.com/gao-lab/Cell_BLAST
- name: Cell_BLAST
  role: Python package providing cb.data.h5_to_h5ad for legacy h5-to-h5ad conversion
    and Cell_BLAST.blast.BLAST for querying
  repo: https://github.com/gao-lab/Cell_BLAST
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:df56cf956e782f726591b48017a5994a0d1189a4d65e460ffb176bc720ec59fb
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# query-latency-curve-visualization

## Summary

Benchmarks and visualizes how cell-querying wall-clock times (Cell BLAST, scmap, CellFishing.jl) scale as the reference dataset grows from 1,000 to 1,000,000 cells subsampled from the 1.3M mouse brain scRNA-seq dataset, reproducing the Fig. 2d scalability comparison.

## When to use

Use when comparing the querying-time scalability of cell-querying tools across increasing reference sizes, e.g. validating that a method scales to 1,000,000-cell references before deployment on large scRNA-seq databases.

## When NOT to use

- The goal is accuracy or cell-type annotation quality rather than querying-time scalability benchmarking
- Reference datasets contain fewer than ~1,000 cells, where neural network-based models like DIRECTi may be under-trained and timings are not meaningful for scalability testing
- The comparison targets batch correction or integration quality instead of computational efficiency

## Inputs

- 1.3M mouse brain scRNA-seq reference dataset (h5 legacy format convertible to h5ad/AnnData)
- Marques query dataset
- Series of reference subsample sizes from 1,000 to 1,000,000 cells

## Outputs

- CSV table of method x reference size x querying time
- Plot of querying time versus reference size comparing Cell BLAST, scmap, and CellFishing.jl (Fig. 2d reproduction)

## How to apply

Load the 1.3M mouse brain scRNA-seq reference (converting legacy h5 to h5ad/AnnData via cb.data.h5_to_h5ad if needed) and the Marques query dataset. Subsample the reference to sizes spanning 1,000 to 1,000,000 cells; at each size, train the DIRECTi generative model on reference cells and query Marques cells with Cell_BLAST.blast.BLAST while recording wall-clock querying time. Repeat the same procedure at each reference size with scmap and CellFishing.jl. Aggregate timings into a method x reference-size x querying-time CSV table, then plot querying time versus reference size for all three methods to reproduce the Fig. 2d comparison curves. Cell BLAST scales well because it is built on a customized neural network-based generative model (DIRECTi), whereas scmap's querying time rises dramatically beyond 10,000 reference cells.

## Related tools

- **Cell BLAST** (Querying method under test; DIRECTi model is trained on each subsampled reference and Cell_BLAST.blast.BLAST performs the timed cell queries) — https://github.com/gao-lab/Cell_BLAST
- **scmap** (Comparison method whose querying time is recorded at each reference size; known to rise dramatically after 10,000 reference cells)
- **CellFishing.jl** (Comparison method whose querying time is recorded at each reference size; scales well like Cell BLAST)
- **DIRECTi** (Deep generative model trained on each subsampled mouse brain reference prior to Cell BLAST queries) — https://github.com/gao-lab/Cell_BLAST
- **Cell_BLAST** (Python package providing cb.data.h5_to_h5ad for legacy h5-to-h5ad conversion and Cell_BLAST.blast.BLAST for querying) — https://github.com/gao-lab/Cell_BLAST

## Examples

```
pip install Cell-BLAST; python h5_to_h5ad.py -i mouse_brain_1.3M.h5 -o mouse_brain_1.3M.h5ad; import Cell_BLAST as cb
```

## Evaluation signals

- Timing rows exist for all three methods (Cell BLAST, scmap, CellFishing.jl) at every reference size from 1,000 to 1,000,000 cells in the CSV table
- Plotted curves qualitatively match the method ordering and trends reported in Fig. 2d of the article
- scmap's querying time rises sharply after 10,000 reference cells while Cell BLAST and CellFishing.jl scale well with increasing reference size
- Wall-clock timings are recorded consistently per method per reference size, producing a complete method x size x time table

## Limitations

- The Cell BLAST model is over-parameterized (like all neural network-based models) and requires a reasonable number of samples to train properly; small datasets with fewer than 1000 cells may suffer from under-training issues
- Legacy h5 inputs must be converted to h5ad/AnnData before use with the current Cell_BLAST package

## Evidence

- [results] scmap's querying time rises dramatically after 10,000 reference cells, while Cell BLAST and CellFishing.jl scale well with increasing reference size.: [evidence span withheld — anchor status: not_found]
- [methods] Convert legacy h5 data to h5ad format: "cb.data.h5_to_h5ad(inputfilename, outputfilename)"
- [abstract] Cell BLAST is a cell querying tool using a neural network-based generative model: "Herein, we present Cell BLAST, an accurate and robust cell-querying method built on a neural network-based generative model and a customized cell-to-cell similarity metric."
- [readme] Reproduce results via the rep branch: [evidence span withheld — anchor status: external_doc]
- [discussion] Cell BLAST models batch effect, making it robust for heterogeneous datasets: "By explicitly modeling multilevel batch effect as well as uncertainty in cell-to-cell similarity estimation, Cell BLAST is an accurate and robust querying algorithm"
