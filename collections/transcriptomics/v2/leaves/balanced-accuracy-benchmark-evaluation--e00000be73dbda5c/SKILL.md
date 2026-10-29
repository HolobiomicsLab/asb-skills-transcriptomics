---
name: balanced-accuracy-benchmark-evaluation
description: Use when comparing cell-querying or label-transfer methods on scRNA-seq
  datasets where some query cell types are absent from the reference, so both sensitivity
  (positive queries) and specificity (negative queries) must be measured.
when_to_use_negative:
- Evaluation where all query cell types are present in the reference — without negative
  queries the specificity (negative-query MBA) component of balanced accuracy cannot
  be computed
- Tasks requiring single-cell-level performance metrics rather than the per-dataset-group
  balanced accuracy aggregation used here
- Benchmarking methods on non-transcriptomic data (e.g., ATAC-seq or spatial data)
  for which the DIRECTi/scmap/CellFishing.jl cell-querying setup does not apply
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cell BLAST
  role: Query-based cell typing method under evaluation; builds DIRECTi-based indexes
    and transfers cell types via posterior-distance hit search
  repo: https://github.com/gao-lab/Cell_BLAST
- name: DIRECTi
  role: Deep generative model fit on each reference dataset to obtain unbiased cell
    embeddings used by Cell BLAST
  repo: https://github.com/gao-lab/Cell_BLAST
- name: scmap
  role: Baseline method performing cell-type transfer via correlation-based mapping
- name: CellFishing.jl
  role: Baseline method performing cell-type transfer via LSH Hamming distance with
    a cutoff of 120
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:380def644205a155d20e47974b5cc439a9651b0e78bcb1198050d638e24b557b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gao-lab/Cell_BLAST
  tools:
  - Cell BLAST
  - CellFishing.jl
  - DIRECTi
  - scmap
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# balanced-accuracy-benchmark-evaluation

## Summary

Benchmark protocol for query-based cell typing that scores each method by balanced accuracy, combining correct cell-type assignment on positive queries with correct rejection of negative queries (cell types absent from the reference), as used to compare Cell BLAST against scmap and CellFishing.jl.

## When to use

Use when comparing cell-querying or label-transfer methods on scRNA-seq datasets where some query cell types are absent from the reference, so both sensitivity (positive queries) and specificity (negative queries) must be measured.

## When NOT to use

- Evaluation where all query cell types are present in the reference — without negative queries the specificity (negative-query MBA) component of balanced accuracy cannot be computed
- Tasks requiring single-cell-level performance metrics rather than the per-dataset-group balanced accuracy aggregation used here
- Benchmarking methods on non-transcriptomic data (e.g., ATAC-seq or spatial data) for which the DIRECTi/scmap/CellFishing.jl cell-querying setup does not apply

## Inputs

- Reference scRNA-seq datasets for each of four benchmark groups
- Query scRNA-seq datasets per group, split into positive queries (cell types present in the reference) and negative queries (cell types absent from the reference)
- Cell-type annotations for reference and query cells
- Method configuration (e.g., CellFishing.jl Hamming distance cutoff of 120)

## Outputs

- Per-query-set balanced accuracy scores for each method (Cell BLAST, scmap, CellFishing.jl)
- Mean balanced accuracy aggregated across dataset groups, as plotted in Fig. 2b-c and Supplementary Figs. 8-9
- benchmark_balanced_accuracy.csv containing the per-method mean balanced accuracy table

## How to apply

Assemble four benchmark dataset groups, each containing reference scRNA-seq data plus query sets split into positive queries (cell types present in the reference) and negative queries (cell types absent from the reference). Fit a DIRECTi generative model on each reference and build a Cell BLAST index via Cell_BLAST.directi and Cell_BLAST.blast, which estimate cell-to-cell similarity as the distance between posterior distributions; run scmap via correlation-based mapping and CellFishing.jl via LSH Hamming distance with a cutoff of 120. For each query set and method, compute balanced accuracy, combining correct cell-type assignment on positive queries with correct rejection of negative queries — the negative-query term is what captures specificity. Aggregate per-method balanced accuracy across the four dataset groups into a mean balanced accuracy, which is the headline metric reported in Fig. 2b-c and Supplementary Figs. 8-9. Write the per-method mean balanced accuracy table to benchmark_balanced_accuracy.csv for inspection and comparison.

## Related tools

- **Cell BLAST** (Query-based cell typing method under evaluation; builds DIRECTi-based indexes and transfers cell types via posterior-distance hit search) — https://github.com/gao-lab/Cell_BLAST
- **DIRECTi** (Deep generative model fit on each reference dataset to obtain unbiased cell embeddings used by Cell BLAST) — https://github.com/gao-lab/Cell_BLAST
- **scmap** (Baseline method performing cell-type transfer via correlation-based mapping)
- **CellFishing.jl** (Baseline method performing cell-type transfer via LSH Hamming distance with a cutoff of 120)

## Examples

```
import Cell_BLAST as cb; model = cb.directi.fit(ref_exp); ref = cb.blast(model, ref_exp); result = ref.query(query_exp)
```

## Evaluation signals

- Each dataset group includes both positive queries (cell types present in the reference) and negative queries (cell types absent), and per-method scores combine correct assignment on positives with correct rejection on negatives
- Per-method mean balanced accuracy across all four dataset groups reproduces the ranking in Fig. 2b-c, with Cell BLAST achieving the highest overall mean balanced accuracy
- Negative-query mean balanced accuracy (specificity) is markedly higher for Cell BLAST than for scmap and CellFishing.jl, consistent with the reported significantly superior specificity
- The aggregation script writes a per-method mean balanced accuracy table to benchmark_balanced_accuracy.csv with one row per method
- Results are reproducible from the repository's 'rep' branch as instructed in the README

## Limitations

- Integrating datasets at mega-scale may cause over-correcting of batches with highly skewed cell-type composition
- Cell BLAST is over-parameterized (like all neural network-based models) and requires a reasonable number of samples to train properly
- A cell number of 3000 is generally sufficient, but datasets with fewer than 1000 cells may suffer from under-training issues

## Evidence

- [results] Negative queries measure specificity: [evidence span withheld — anchor status: not_found]
- [abstract] Posterior-distance similarity metric: "we present Cell BLAST, an accurate and robust cell-querying method built on a neural network-based generative model and a customized cell-to-cell similarity metric."
- [readme] Install and import the package: "import Cell_BLAST as cb
print(cb.__version__)"
- [intro] Benchmark method for comparison: "We evaluate the query-based cell typing performance of Cell BLAST with extensive benchmark experiments"
