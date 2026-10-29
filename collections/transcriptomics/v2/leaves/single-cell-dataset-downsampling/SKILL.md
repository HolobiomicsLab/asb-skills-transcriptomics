---
name: single-cell-dataset-downsampling
description: Use when benchmarking the scalability of a reference-mapping pipeline
  (e.g. Symphony/Harmony) and you need matched downsampled expression/metadata subsets
  at multiple cell counts as controlled inputs.
when_to_use_negative:
- When the analysis goal is discovery of novel cell types in the query — downsampling
  to fit a reference benchmark discards cells and Symphony itself does not replace
  de novo analyses of the query alone
- When evaluating mapping accuracy rather than runtime — downsampled subsets may no
  longer cover all reference cell types, violating Symphony's Condition I requiring
  comprehensive cell type coverage
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Symphony
  role: Builds the compressed reference from each downsampled subset (buildReference
    / buildReferenceFromHarmonyObj) and maps queries (mapQuery), the runtime being
    the benchmarked output of each downsampled pair
  repo: https://github.com/immunogenomics/symphony
- name: Harmony
  role: Provides the de novo integration baseline run on combined reference-plus-query
    data at each downsampled size, plus integration inside reference building (return_object
    = TRUE)
  repo: https://github.com/immunogenomics/harmony
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:fbe2687b6f175077ac94aeb3fa027988e86afd01ba6705d9ced479fad470635a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/symphony
  tools:
  - Harmony
  - Symphony
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-dataset-downsampling

## Summary

Systematically subset a single-cell expression matrix and its cell metadata to a ladder of cell counts (e.g. 20K–500K reference cells; 1K–100K query cells) so that runtime and scalability of reference-building vs. query-mapping pipelines can be measured on comparable inputs.

## When to use

Use when benchmarking the scalability of a reference-mapping pipeline (e.g. Symphony/Harmony) and you need matched downsampled expression/metadata subsets at multiple cell counts as controlled inputs.

## When NOT to use

- When the analysis goal is discovery of novel cell types in the query — downsampling to fit a reference benchmark discards cells and Symphony itself does not replace de novo analyses of the query alone
- When evaluating mapping accuracy rather than runtime — downsampled subsets may no longer cover all reference cell types, violating Symphony's Condition I requiring comprehensive cell type coverage

## Inputs

- Reference expression matrix (genes x cells), e.g. CITE-seq memory T-cell counts, log(CP10k)-normalized
- Cell metadata dataframe (cells x attributes, e.g. donor labels) aligned to the expression columns
- Target subset sizes (reference sizes such as 20K/100K/500K; query sizes such as 1K/10K/100K cells)

## Outputs

- Downsampled expression matrices at each target cell count
- Downsampled metadata dataframes matching each expression subset
- A size-stratified set of (reference, query) pairs ready for build/mapping timing runs, ultimately assembled into benchmark_timings.csv

## How to apply

Load the paired expression matrix (genes x cells) and cell metadata dataframe — e.g. the CITE-seq memory T-cell matrices used in the Symphony benchmark — and keep them cell-index-aligned while subsetting. Draw random cell subsets spanning the planned size ladder: reference sizes from 20K to 500K cells and query sizes of 1K, 10K, and 100K cells, so build-time and mapping-time scaling can be compared against a de novo Harmony baseline at each scale. Feed each downsampled pair into the downstream pipeline unchanged (vargenes_vst variable gene selection, scaleDataWithStats, PCA via SVD, Harmony integration with return_object = TRUE, then buildReferenceFromHarmonyObj, and mapQuery for query timing), recording wall-clock time per step. Rationale: a fixed-size query mapped onto an increasingly large compressed reference isolates the cost of Symphony's reference compression, whereas de novo integration cost grows with total cell count (e.g. ~4806 s vs. ~0.99 s at the 500K/10K scale).

## Related tools

- **Symphony** (Builds the compressed reference from each downsampled subset (buildReference / buildReferenceFromHarmonyObj) and maps queries (mapQuery), the runtime being the benchmarked output of each downsampled pair) — https://github.com/immunogenomics/symphony
- **Harmony** (Provides the de novo integration baseline run on combined reference-plus-query data at each downsampled size, plus integration inside reference building (return_object = TRUE)) — https://github.com/immunogenomics/harmony

## Examples

```
reference = symphony::buildReference(ref_exp_downsampled, ref_metadata_downsampled, vars = c('donor'), K = 100, verbose = TRUE, do_normalize = FALSE, vargenes_method = 'vst', topn = 2000, d = 20)
```

## Evaluation signals

- Downsampled metadata row count equals the expression matrix column count at every target size (cells remain index-aligned)
- Subset sizes match the planned ladder exactly (e.g. 20K–500K reference; 1K/10K/100K query) before any timing run
- Recorded build time grows with reference size and reaches ~5163 s at 500K cells; Symphony mapQuery completes in ~0.99 s for a 10K-cell query at that scale
- De novo Harmony runtime on combined cells (~4806 s at comparable scale) far exceeds Symphony query-mapping time, confirming the expected scaling asymmetry
- All recorded timings assemble into benchmark_timings.csv without missing entries for any (reference size, query size) combination

## Limitations

- Downsampled benchmarks inherit Symphony's Condition I limitation: if a subset loses cell-type coverage, results are less flexible than de novo Harmony integration
- Runtime figures are hardware- and implementation-dependent; the ~5163 s build / ~0.99 s map and ~17.7 h build / ~11.0 s map figures are reference points, not guarantees
- Subsampling for timing does not address accuracy; Symphony may map poorly when the query contains populations absent from the reference, and novel-population detection remains unsolved

## Evidence

- [other] Downsampling ladder rationale: [evidence span withheld — anchor status: not_found]
- [other] Expected timing figures: [evidence span withheld — anchor status: not_found]
- [readme] Pipeline applied to each downsampled subset: [evidence span withheld — anchor status: external_doc]
- [discussion] Why mapping is fast on large references: "Symphony compresses the reference by extracting relevant reference-derived parameters from the mixture model to map query cells in seconds."
