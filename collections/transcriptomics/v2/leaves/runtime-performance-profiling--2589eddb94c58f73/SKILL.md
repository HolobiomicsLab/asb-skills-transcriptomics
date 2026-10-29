---
name: runtime-performance-profiling
description: Use when quantifying how mapping or integration runtime scales with cell
  count, e.g. benchmarking Symphony reference builds (20K-1.39M cells) and query mappings
  (1K-100K cells) against de novo Harmony integration.
when_to_use_negative:
- The goal is novel cell type discovery in the query — Symphony mapping assumes the
  reference comprehensively covers query cell types and does not replace de novo analyses
- The query batch variables or study design change at mapping time such that reference
  compression parameters no longer apply, requiring full de novo integration instead
- The benchmark targets accuracy or annotation transfer quality rather than runtime
  scaling; use integration-quality metrics (e.g. LISI) instead
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Symphony
  role: Builds compressed references via buildReferenceFromHarmonyObj / buildReference
    and maps query cells in seconds via mapQuery; the subject of the runtime benchmark
  repo: https://github.com/immunogenomics/symphony
- name: Harmony
  role: Performs the reference integration step (HarmonyMatrix with return_object
    = TRUE) and serves as the de novo integration baseline whose runtime scales with
    total cell count
  repo: https://github.com/immunogenomics/harmony
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:7eaa24bcf33e63445c7a3c5d8381b4f5db8cf5852c108c28a5c4e74034d709ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/symphony
  tools:
  - Harmony
  - Symphony
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# runtime-performance-profiling

## Summary

Profiles wall-clock runtimes of Symphony reference building, query mapping, and de novo Harmony integration as reference/query sizes grow, demonstrating that Symphony maps query cells in seconds against million-cell references built once (~5000x faster than rerunning integration).

## When to use

Use when quantifying how mapping or integration runtime scales with cell count, e.g. benchmarking Symphony reference builds (20K-1.39M cells) and query mappings (1K-100K cells) against de novo Harmony integration.

## When NOT to use

- The goal is novel cell type discovery in the query — Symphony mapping assumes the reference comprehensively covers query cell types and does not replace de novo analyses
- The query batch variables or study design change at mapping time such that reference compression parameters no longer apply, requiring full de novo integration instead
- The benchmark targets accuracy or annotation transfer quality rather than runtime scaling; use integration-quality metrics (e.g. LISI) instead

## Inputs

- CITE-seq memory T-cell expression matrix (genes x cells) and metadata (cells x attributes), downsampled to 20K-500K reference cells and 1K/10K/100K query cells
- 1.39M-cell COVID-19 reference dataset (expression and metadata)
- Variable gene means/standard deviations table (columns: symbol, mean, stddev)

## Outputs

- benchmark_timings.csv — summary table of reference build times, per-query mapping times, and de novo Harmony baseline runtimes at each scale
- Per-scale timing measurements (e.g. ~5163 s build / 0.99 s mapping at 500K+10K; ~17.7 h build / 11.0 s mapping at 1.39M cells)

## How to apply

Load the CITE-seq memory T-cell expression and metadata matrices and downsample to reference sizes spanning 20K to 500K cells and query sizes of 1K, 10K, and 100K cells. For each reference size, build a Symphony reference: select variable genes (vargenes_vst), scale with scaleDataWithStats, run PCA via SVD, run Harmony integration with return_object = TRUE, then compress the Harmony object with buildReferenceFromHarmonyObj, recording wall-clock build time (expected ~5163 s for the 500K-cell reference). Map each query subset with mapQuery and record per-query mapping time (expected ~0.99 s for a 10K-cell query). As a baseline, rerun de novo Harmony integration on the combined reference-plus-query data at each size and record its runtime (expected ~4806 s at the comparable scale). Repeat on the 1.39M-cell COVID-19 reference (expected ~17.7 h build, ~11.0 s per query mapping), then assemble all timings into benchmark_timings.csv.

## Related tools

- **Symphony** (Builds compressed references via buildReferenceFromHarmonyObj / buildReference and maps query cells in seconds via mapQuery; the subject of the runtime benchmark) — https://github.com/immunogenomics/symphony
- **Harmony** (Performs the reference integration step (HarmonyMatrix with return_object = TRUE) and serves as the de novo integration baseline whose runtime scales with total cell count) — https://github.com/immunogenomics/harmony

## Examples

```
reference = symphony::buildReference(ref_exp, ref_metadata, vars = c('donor'), K = 100, vargenes_method = 'vst', d = 20, do_umap = FALSE); t0 <- Sys.time(); query = symphony::mapQuery(query_exp, query_metadata, reference, do_normalize = FALSE); as.numeric(Sys.time() - t0)
```

## Evaluation signals

- 500K-cell reference build time is ~5163 s (86.1 min); deviations by more than an order of magnitude suggest pipeline misconfiguration
- Mapping a 10K-cell query onto the 500K-cell reference completes in ~0.99 s; mapping should remain sub-second to ~11 s even for million-cell references
- De novo Harmony integration on combined reference-plus-query data takes ~4806 s (80.1 min) at the comparable scale, confirming the ~5000x mapping speedup
- 1.39M-cell COVID-19 reference requires ~17.7 h to build but only ~11.0 s per query mapping
- benchmark_timings.csv contains rows for every reference size (20K-500K, 1.39M) and query size (1K, 10K, 100K) with recorded wall-clock seconds

## Limitations

- Harmony reference building itself is expensive at large scale (~5163 s at 500K cells, ~17.7 h at 1.39M cells); the seconds-scale speedup applies only to query mapping, not to building the reference
- Symphony mapping metrics may be less informative when a novel query population closely resembles an existing reference population
- Condition I (comprehensive cell type coverage in the reference) is required and limits equivalence with de novo Harmony integration

## Evidence

- [discussion] Symphony compresses the reference by extracting relevant reference-derived parameters from the mixture model to map query cells in seconds.: "Symphony compresses the reference by extracting relevant reference-derived parameters from the mixture model to map query cells in seconds."
- [methods] Run Harmony integration. It is important to set `return_object = TRUE`.: "Run Harmony integration. It is important to set `return_object = TRUE`."
- [methods] Compress a Harmony object into a Symphony reference: "# Compress a Harmony object into a Symphony reference"
- [intro] An ideal reference mapping algorithm must remove confounding signals from complex study design, scale to large datasets, map with high accuracy, and enable inference of diverse query cell annotations: "First, similar to de novo integration algorithms, it must be able to remove confounding signals due to complex study design in both the reference and query."
- [abstract] Symphony enables efficient query mapping of single cells onto large reference atlases within seconds: "enables efficient query mapping within seconds"
