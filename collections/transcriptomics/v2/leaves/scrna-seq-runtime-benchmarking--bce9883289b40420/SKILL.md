---
name: scrna-seq-runtime-benchmarking
description: 'Use when comparing scalability of clustering or cluster-merging methods
  on scRNA-seq data: you need wall-clock runtime on a fixed, small, cell-type-labelled
  subset to contrast with AUROC performance on the full dataset.'
when_to_use_negative:
- The analysis goal is statistical significance of cluster differences rather than
  method benchmarking — run cc.tl.merge_clusters directly instead of timing comparisons.
- The dataset is already small enough that all methods run interactively; runtime
  contrast is only meaningful where the expensive method measurably scales poorly
  (e.g. 178.80 s for 500 cells implies prohibitive cost at 483152 cells).
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: cytocipher
  role: Method under benchmark; its cc.tl.merge_clusters pipeline (get_markers, code_enrich,
    merge) is timed against Sc-SHC
  repo: https://github.com/BradBalderson/Cytocipher/
- name: scanpy
  role: Standard preprocessing workflow producing the log-cpm normalised AnnData with
    Leiden labels used as benchmark input
  repo: https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html
- name: AnnData
  role: Data container format for the downsampled benchmark subset
- name: Jupyter
  role: Notebook environment in which the timing runs and diagnostics are executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad435
    title: <i>Cytocipher</i> determines significantly different populations of cells
      in single-cell RNA-seq data
  source_content_hash: sha256:9f95ca2e8a39f675e0be0584708b34be46527d91bbc58e1d2f52817735878560
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradBalderson/Cytocipher
  tools:
  - AnnData
  - Jupyter
  - cytocipher
  - scanpy
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# scRNA-seq-runtime-benchmarking

## Summary

Times competing single-cell clustering/merging methods on identical cell subsets (e.g. Sc-SHC vs Cytocipher's merge_clusters on a 500-cell, 12-cell-type Tabula Sapiens subset) to quantify runtime scaling differences alongside accuracy benchmarks.

## When to use

Use when comparing scalability of clustering or cluster-merging methods on scRNA-seq data: you need wall-clock runtime on a fixed, small, cell-type-labelled subset to contrast with AUROC performance on the full dataset.

## When NOT to use

- The analysis goal is statistical significance of cluster differences rather than method benchmarking — run cc.tl.merge_clusters directly instead of timing comparisons.
- The dataset is already small enough that all methods run interactively; runtime contrast is only meaningful where the expensive method measurably scales poorly (e.g. 178.80 s for 500 cells implies prohibitive cost at 483152 cells).

## Inputs

- AnnData object (.h5ad) with log-cpm normalised values and high-resolution Leiden cluster labels, downsampled to ~500 cells across 12 cell types
- Known cell-type labels for the subset (ground truth for paired AUROC checks)
- Competing method implementations: Sc-SHC and Cytocipher (cytocipher package)

## Outputs

- Per-method wall-clock runtime measurements (seconds) on the fixed subset
- auroc_benchmark_results.csv combining runtimes with AUROC values for all scoring variants and competitors
- cc.pl.sig_cluster_diagnostics plots as QC of enrichment-score distributions

## How to apply

Downsample the full dataset to a small, cell-type-labelled subset — in the Cytocipher paper, 500 cells spanning 12 cell types from the 483152-cell Tabula Sapiens Atlas — so the expensive method completes in feasible time. Apply each method to the identical AnnData input under equivalent settings: Cytocipher's cc.tl.merge_clusters(data, 'leiden') pipeline (marker identification via cc.tl.get_markers, then cc.tl.code_enrich) versus the competing method (Sc-SHC). Record wall-clock time per method and assemble alongside AUROC accuracy values into a results table (auroc_benchmark_results.csv). The reported contrast is Sc-SHC <3.5 s versus Cytocipher 178.80 s on the same 500-cell subset, so runtime and AUROC should be reported together to capture the accuracy-vs-speed trade-off.

## Related tools

- **cytocipher** (Method under benchmark; its cc.tl.merge_clusters pipeline (get_markers, code_enrich, merge) is timed against Sc-SHC) — https://github.com/BradBalderson/Cytocipher/
- **scanpy** (Standard preprocessing workflow producing the log-cpm normalised AnnData with Leiden labels used as benchmark input) — https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html
- **AnnData** (Data container format for the downsampled benchmark subset)
- **Jupyter** (Notebook environment in which the timing runs and diagnostics are executed)

## Examples

```
import time; t0=time.time(); cc.tl.get_markers(data,'leiden'); cc.tl.code_enrich(data,'leiden'); cc.tl.merge_clusters(data,'leiden'); runtime=time.time()-t0
```

## Evaluation signals

- Runtime contrast reproduces the reported Sc-SHC <3.5 s versus Cytocipher 178.80 s on the 500-cell, 12-cell-type Tabula Sapiens subset (order-of-magnitude gap expected).
- AUROC values on the same subset match reported references: 0.9930 (code-scoring), 0.9937 (coexpr-scoring), 0.9945 (Giotto PAGE), 0.9940 (Giotto (−)) at 7385-cell downsample; 0.9974/0.9976 at full 483152-cell scale.
- Both methods are timed on the identical AnnData input and identical cluster labels — any input mismatch invalidates the comparison.
- Results table (auroc_benchmark_results.csv) contains both runtime and AUROC columns for every method variant.

## Limitations

- Runtime benchmarking is restricted to a 500-cell, 12-cell-type subset because Cytocipher's merging takes 178.80 s even at that scale; full-scale (483152-cell) timing of both methods is not directly comparable.
- Cytocipher scales to >480000 cells with high test performance, but the runtime trade-off versus faster methods like Sc-SHC (<3.5 s) is an explicit accuracy-versus-speed trade-off.
- The article's discussion/limitations section content was unavailable (no changelog / discussion text extracted), so additional caveats cannot be traced.

## Evidence

- [methods] On a 500-cell, 12-cell-type subset, time Sc-SHC against Cytocipher's merging pipeline and verify the reported runtime contrast (<3.5 s vs 178.80 s).: [evidence span withheld — anchor status: not_found]
- [methods] Assemble all AUROC values and runtimes into auroc_benchmark_results.csv: [evidence span withheld — anchor status: not_found]
- [abstract] Cytocipher also scales to large datasets with high-test performance, as shown by application to the Tabula Sapiens Atlas representing >480 000 cells: "Cytocipher also scales to large datasets with high-test performance, as shown by application to the Tabula Sapiens Atlas representing >480 000 cells"
- [readme] An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow]: "An AnnData object, *data*, that has been processed similarly to the [scanpy standard workflow](https://scanpy-tutorials.readthedocs.io/en/latest/pbmc3k.html) to produce log-cpm normalised data with"
- [methods] Downsample to 7385 cells and repeat the merge under each scoring variant plus the competing methods Giotto PAGE and Giotto (−): [evidence span withheld — anchor status: not_found]
