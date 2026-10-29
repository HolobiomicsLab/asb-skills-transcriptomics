---
name: single-cell-query-mapping-benchmarking
description: Use when evaluating whether a reference-mapping workflow (Symphony/Harmony)
  scales to large atlases and need wall-clock timings of reference building, query
  mapping, and de novo integration baselines.
when_to_use_negative:
- The analysis goal is novel cell-type discovery in the query — Symphony does not
  replace de novo analyses of the query alone
- The reference lacks comprehensive cell type coverage, so mapping is not equivalent
  to de novo Harmony integration
- Datasets are scATAC-seq with different peak sets across reference and query, where
  optimal feature selection for mapping is undefined
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Symphony
  role: Builds compressed reference (buildReferenceFromHarmonyObj / buildReference)
    and maps query cells in seconds via mapQuery; timings are the benchmark subject
  repo: https://github.com/immunogenomics/symphony
- name: Harmony
  role: Performs reference integration (HarmonyMatrix with return_object=TRUE) and
    provides the de novo integration baseline runtime
  repo: https://github.com/immunogenomics/harmony
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:295801b9d60cd3fb0d4434a4155d1f5319a68d67a947c3cba4c372bed8b6953e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/symphony
  tools:
  - Harmony
  - Symphony
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# single-cell-query-mapping-benchmarking

## Summary

Benchmark the scalability of Symphony reference mapping against de novo Harmony integration by timing reference builds (20K-500K CITE-seq memory T cells; 1.39M-cell COVID-19 atlas) and query mapping (1K/10K/100K cells) to quantify the ~5000x speedup from reference compression.

## When to use

Use when evaluating whether a reference-mapping workflow (Symphony/Harmony) scales to large atlases and need wall-clock timings of reference building, query mapping, and de novo integration baselines.

## When NOT to use

- The analysis goal is novel cell-type discovery in the query — Symphony does not replace de novo analyses of the query alone
- The reference lacks comprehensive cell type coverage, so mapping is not equivalent to de novo Harmony integration
- Datasets are scATAC-seq with different peak sets across reference and query, where optimal feature selection for mapping is undefined

## Inputs

- CITE-seq memory T-cell expression matrix (genes x cells) and cell metadata dataframe
- Downsampled reference subsets (20K to 500K cells) and query subsets (1K, 10K, 100K cells)
- 1.39M-cell COVID-19 reference dataset (expression + metadata)
- Reference metadata with batch/donor variables for Harmony integration (e.g. 'donor')

## Outputs

- benchmark_timings.csv with reference build times, per-query mapping times, and de novo Harmony baseline times at each size
- Symphony reference objects (from buildReferenceFromHarmonyObj / symphony::buildReference) per reference size
- Query mapping results containing harmonized query embedding (query$Z)

## How to apply

Downsample the CITE-seq memory T-cell expression and metadata matrices to reference sizes spanning 20K-500K cells and query sizes of 1K, 10K, and 100K cells. For each reference size, run the full Symphony build pipeline: vargenes_vst variable gene selection, scaleDataWithStats scaling, PCA via SVD (saving gene loadings s$u), HarmonyMatrix with return_object=TRUE, then compress with buildReferenceFromHarmonyObj while recording wall-clock build time (expected ~5163 s for the 500K-cell reference). Map each query subset with mapQuery and record per-query time (expected ~0.99 s for a 10K-cell query). As a baseline, rerun de novo Harmony integration on the combined reference-plus-query data at each size (expected ~4806 s at comparable scale) and repeat on the 1.39M-cell COVID-19 reference (~17.7 h build, ~11.0 s mapping). Assemble all timings into a summary table saved as benchmark_timings.csv.

## Related tools

- **Symphony** (Builds compressed reference (buildReferenceFromHarmonyObj / buildReference) and maps query cells in seconds via mapQuery; timings are the benchmark subject) — https://github.com/immunogenomics/symphony
- **Harmony** (Performs reference integration (HarmonyMatrix with return_object=TRUE) and provides the de novo integration baseline runtime) — https://github.com/immunogenomics/harmony

## Examples

```
reference = symphony::buildReference(ref_exp, ref_metadata, vars = c('donor'), vargenes_method = 'vst', topn = 2000, d = 20); query = mapQuery(query_exp, query_metadata, reference)
```

## Evaluation signals

- Symphony query mapping completes in seconds (sub-second ~0.99 s for a 10K-cell query; ~11.0 s against the 1.39M-cell reference) regardless of reference size
- Reference build time for the 500K-cell CITE-seq reference is ~5163 s (86.1 min) and de novo Harmony integration at comparable scale is ~4806 s (80.1 min)
- 1.39M-cell COVID-19 reference build takes ~17.7 h with per-query mapping ~11.0 s
- Harmony is called with return_object = TRUE and do_pca = FALSE so the full object can be compressed by buildReferenceFromHarmonyObj
- vargenes_means_sds table has columns c('symbol', 'mean', 'stddev') before scaling and compression

## Limitations

- Symphony mapping metrics may be less informative when a novel population is very similar to an existing reference population
- Identification of novel cell types that have failed to map is an unsolved future direction for mapping algorithms
- Condition I (comprehensive cell type coverage in the reference) is less flexible and limits equivalence with de novo Harmony integration
- Timings depend on Harmony parameters (theta, nclust, max.iter.harmony) and the number of variable genes/PCs chosen during reference building

## Evidence

- [discussion] Symphony compresses the reference by extracting relevant reference-derived parameters from the mixture model to map query cells in seconds.: "Symphony compresses the reference by extracting relevant reference-derived parameters from the mixture model to map query cells in seconds."
- [methods] Run Harmony integration. It is important to set `return_object = TRUE`.: "Run Harmony integration. It is important to set `return_object = TRUE`."
- [readme] This function performs all steps of the reference building pipeline including variable gene selection, scaling, PCA, Harmony, and Symphony compression.: [evidence span withheld — anchor status: external_doc]
- [intro] Fast mapping of query cells against a large, stable reference is a well-recognized open problem27 and active area of research28–30.: "Fast mapping of query cells against a large, stable reference is a well-recognized open problem27 and active area of research28–30."
