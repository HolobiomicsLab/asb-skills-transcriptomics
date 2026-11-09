---
name: de-novo-harmony-integration-baseline
description: 'Use when benchmarking Symphony (or similar reference-mapping) against
  de novo integration: you have a large expression matrix plus cell metadata and need
  per-size build, query-mapping, and de novo Harmony wall-clock timings.'
when_to_use_negative:
- The goal is discovering novel query cell types rather than benchmarking runtime
  — de novo integration over the reference-plus-query set cannot map populations absent
  from the reference, and Condition I (comprehensive cell type coverage) limits equivalence
  with de novo Harmony integration
- The query contains a novel population very similar to an existing reference population,
  where mapping metrics may be less informative and the baseline comparison is not
  meaningful
- Input is scATAC-seq reference/query data with different peak sets, where the optimal
  feature selection for reference mapping is not defined
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Symphony
  role: Subject of the benchmark; builds compressed integrated references and maps
    query cells in seconds, whose timings are compared to the Harmony baseline
  repo: https://github.com/immunogenomics/symphony
- name: Harmony
  role: De novo integration baseline run on combined reference-plus-query cells at
    each size, and the integration backend for Symphony reference building (must set
    return_object = TRUE)
  repo: https://github.com/immunogenomics/harmony
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:b61f84aa474c806180d73b8e728c6d66b0d129d73d66b67c6cf6a2528ae288f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/symphony
  tools:
  - Harmony
  - Symphony
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# de-novo-harmony-integration-baseline

## Summary

Establishes de novo Harmony integration of reference-plus-query cells as the runtime baseline against which Symphony's compressed reference building and seconds-scale query mapping are benchmarked, using the CITE-seq memory T-cell atlas and a 1.39M-cell COVID-19 reference.

## When to use

Use when benchmarking Symphony (or similar reference-mapping) against de novo integration: you have a large expression matrix plus cell metadata and need per-size build, query-mapping, and de novo Harmony wall-clock timings.

## When NOT to use

- The goal is discovering novel query cell types rather than benchmarking runtime — de novo integration over the reference-plus-query set cannot map populations absent from the reference, and Condition I (comprehensive cell type coverage) limits equivalence with de novo Harmony integration
- The query contains a novel population very similar to an existing reference population, where mapping metrics may be less informative and the baseline comparison is not meaningful
- Input is scATAC-seq reference/query data with different peak sets, where the optimal feature selection for reference mapping is not defined

## Inputs

- CITE-seq memory T-cell gene expression matrix (genes x cells) and cell metadata dataframe (e.g. donor column)
- Downsampled reference expression subsets spanning 20K to 500K cells
- Downsampled query expression subsets of 1K, 10K, and 100K cells
- 1.39M-cell COVID-19 reference dataset (expression plus metadata)

## Outputs

- Symphony reference objects per downsampled size (Harmony object compressed via buildReferenceFromHarmonyObj)
- Harmonized query embeddings (query$Z) with per-query mapping times
- Summary table of build, mapping, and de novo Harmony baseline timings saved as benchmark_timings.csv

## How to apply

Downsample the CITE-seq memory T-cell expression and metadata matrices to reference sizes spanning 20K–500K cells and query sizes of 1K, 10K, and 100K cells. For each reference size, run the Symphony build pipeline (vargenes_vst variable gene selection, scaleDataWithStats, PCA via SVD, then Harmony integration with return_object = TRUE) and record build wall-clock time, expecting ~5163 s (86.1 min) for the 500K-cell reference. For each query, call mapQuery against the built reference and record mapping time (expected ~0.99 s for a 10K-cell query). As the baseline, rerun Harmony de novo on the combined reference-plus-query data at each size and record the runtime (expected ~4806 s / 80.1 min at the comparable scale), then repeat build and mapping timing on the 1.39M-cell COVID-19 reference (~17.7 h build, ~11.0 s per query) and assemble all timings into benchmark_timings.csv.

## Related tools

- **Symphony** (Subject of the benchmark; builds compressed integrated references and maps query cells in seconds, whose timings are compared to the Harmony baseline) — https://github.com/immunogenomics/symphony
- **Harmony** (De novo integration baseline run on combined reference-plus-query cells at each size, and the integration backend for Symphony reference building (must set return_object = TRUE)) — https://github.com/immunogenomics/harmony

## Examples

```
harmObj <- harmony::HarmonyMatrix(t(Z_pca_ref), ref_metadata, theta=2, vars_use=c('donor'), nclust=100, max.iter.harmony=10, return_object=TRUE, do_pca=FALSE)
```

## Evaluation signals

- 500K-cell Symphony reference build completes in ~5163 s (86.1 min) wall-clock
- Mapping a 10K-cell query onto the built reference completes in ~0.99 s, and per-query mapping stays sub-second to ~11 s even on million-cell references
- De novo Harmony integration on comparable reference-plus-query data takes ~4806 s (80.1 min), confirming the baseline scales with total cell count
- 1.39M-cell COVID-19 reference shows ~17.7 h build time and ~11.0 s per-query mapping time
- All recorded build, mapping, and baseline timings assemble into benchmark_timings.csv with consistent per-size entries

## Limitations

- Condition I (comprehensive cell type coverage in the reference) is less flexible than de novo integration and limits equivalence with de novo Harmony integration
- Symphony mapping metrics may be less informative when a novel query population is very similar to an existing reference population, and Symphony does not replace de novo analyses of the query alone for novel cell type discovery
- Symphony mapping accelerates inference on a prebuilt reference, but the reference itself must still be built once via Harmony integration, which scales with total cell count

## Evidence

- [discussion] Symphony compresses the reference by extracting relevant reference-derived parameters from the mixture model to map query cells in seconds.: "Symphony compresses the reference by extracting relevant reference-derived parameters from the mixture model to map query cells in seconds."
- [intro] We present Symphony, a novel algorithm to compress a large, integrated reference and map query cells to a precise location in the reference embedding within seconds.: "We present Symphony, a novel algorithm to compress a large, integrated reference and map query cells to a precise location in the reference embedding within seconds."
- [methods] Run Harmony integration. It is important to set `return_object = TRUE`.: "Run Harmony integration. It is important to set `return_object = TRUE`."
- [readme] Run Harmony to integrate the reference cells: [evidence span withheld — anchor status: external_doc]
