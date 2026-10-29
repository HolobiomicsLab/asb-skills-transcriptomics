---
name: negative-query-rejection-scoring
description: Use when benchmarking cell-type prediction methods on reference/query
  scRNA-seq dataset groups that include both positive queries and negative queries,
  so specificity (rejecting absent cell types) is scored alongside assignment accuracy.
when_to_use_negative:
- When all query cells belong to cell types present in the reference (no negative
  queries) — plain balanced accuracy on positives suffices and specificity cannot
  be measured
- When the goal is annotating a single new dataset rather than comparatively benchmarking
  cell-querying methods on known ground truth
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3366
tools:
- name: Cell BLAST
  role: Query-based cell typing via DIRECTi posterior-distance hit search; benchmarked
    for positive-query accuracy and negative-query rejection
  repo: github.com/gao-lab/Cell_BLAST
- name: scmap
  role: Comparison method performing correlation-based mapping of query cells to reference
    cells
- name: CellFishing.jl
  role: Comparison method using LSH Hamming distance with a cutoff of 120 for cell
    search
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:618b9261f5877fca551a38f3582ecf218a2f713c579f9ce5e6fa39a16df4d7f7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cell BLAST
  - CellFishing.jl
  - scmap
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# negative-query-rejection-scoring

## Summary

A benchmark-scoring skill from the Cell BLAST paper that evaluates query-based cell typing by combining correct cell-type assignment on positive queries with correct rejection of negative queries (cell types absent from the reference), aggregated as mean balanced accuracy (MBA).

## When to use

Use when benchmarking cell-type prediction methods on reference/query scRNA-seq dataset groups that include both positive queries and negative queries, so specificity (rejecting absent cell types) is scored alongside assignment accuracy.

## When NOT to use

- When all query cells belong to cell types present in the reference (no negative queries) — plain balanced accuracy on positives suffices and specificity cannot be measured
- When the goal is annotating a single new dataset rather than comparatively benchmarking cell-querying methods on known ground truth

## Inputs

- Reference scRNA-seq datasets for each of four benchmark groups
- Query scRNA-seq datasets containing both positive queries (cell types present in reference) and negative queries (cell types absent from reference)
- Cell-type predictions/hit lists from each method (Cell BLAST, scmap, CellFishing.jl)

## Outputs

- Per-query-set balanced accuracy scores per method
- Mean balanced accuracy (MBA) per method across dataset groups (Fig. 2b-c, Supplementary Figs. 8-9)
- Per-method mean balanced accuracy table written to benchmark_balanced_accuracy.csv

## How to apply

For each benchmark group, load reference and query scRNA-seq datasets deliberately including both positive queries (cell types present in the reference) and negative queries (cell types absent from the reference). Run each competing method — Cell BLAST via posterior-distance hit search on a DIRECTi model built with Cell_BLAST.directi and Cell_BLAST.blast, scmap via correlation-based mapping, and CellFishing.jl via LSH Hamming distance with a cutoff of 120 — on every query set. Score each query set with balanced accuracy so that correctly assigning positive-query cells and correctly rejecting negative-query cells contribute equally, rather than using plain accuracy which lets a method score well by assigning everything to some reference type. Aggregate per-method balanced accuracy across the four dataset groups into the mean balanced accuracy (MBA) reported in Fig. 2b-c and Supplementary Figs. 8-9, and write the per-method table (e.g. benchmark_balanced_accuracy.csv). Correct application is judged by whether negative-query MBA is reported separately, exposing specificity differences such as Cell BLAST's superior negative-query MBA versus scmap and CellFishing.jl.

## Related tools

- **Cell BLAST** (Query-based cell typing via DIRECTi posterior-distance hit search; benchmarked for positive-query accuracy and negative-query rejection) — github.com/gao-lab/Cell_BLAST
- **scmap** (Comparison method performing correlation-based mapping of query cells to reference cells)
- **CellFishing.jl** (Comparison method using LSH Hamming distance with a cutoff of 120 for cell search)

## Examples

```
conda create -n cb python=3.9 && conda activate cb && pip install Cell-BLAST; import Cell_BLAST as cb; model = cb.directi.fit_DIRECTi(ref); result = cb.blast.blast(query, model)
```

## Evaluation signals

- Each benchmark group's query sets include both positive and negative queries; negative queries must yield rejections, not forced assignments
- Balanced accuracy is computed per query set, giving equal weight to positive-query assignment and negative-query rejection
- Per-method MBA values should reproduce the ranking in Fig. 2b-c, with Cell BLAST achieving the highest overall mean balanced accuracy
- Negative-query MBA should be reported separately, showing Cell BLAST's significantly superior specificity over scmap and CellFishing.jl
- Final per-method table written to benchmark_balanced_accuracy.csv without missing method/group entries

## Limitations

- Integrating datasets at mega-scale may incur over-correcting of batches with highly skewed cell-type composition
- Cell BLAST's model is over-parameterized (like all neural network-based models) and requires a reasonable number of samples to train properly
- Datasets with fewer than 1000 cells may suffer from under-training issues; ~3000 cells is generally sufficient

## Evidence

- [results] highest overall mean balanced accuracy and superior specificity: [evidence span withheld — anchor status: not_found]
- [results] CellFishing.jl LSH cutoff 120: [evidence span withheld — anchor status: not_found]
- [abstract] DIRECTi model-based cell search: "we present Cell BLAST, an accurate and robust cell-querying method built on a neural network-based generative model and a customized cell-to-cell similarity metric."
- [readme] install Cell BLAST via pip: "Install Cell BLAST from PyPI by running:

   `pip install Cell-BLAST`"
- [discussion] under-training on small datasets: "a cell number of 3000 is generally sufficient for good performance, while small-sized datasets with fewer than 1000 cells may suffer from under-training issues."
