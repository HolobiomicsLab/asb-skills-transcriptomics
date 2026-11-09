---
name: single-cell-reference-atlas-building
description: Use when constructing a reusable integrated reference from large single-cell
  RNA-seq (or CITE-seq) expression matrices so that future query datasets can be mapped
  and annotated without re-running de novo integration.
when_to_use_negative:
- The goal is novel cell type discovery in the query alone — Symphony does not replace
  de novo analyses of the query, and mapping metrics may be uninformative when a novel
  population resembles an existing reference population
- The reference lacks comprehensive cell type coverage (Condition I) — equivalence
  with de novo Harmony integration is not guaranteed and mapping is less flexible
- Only a one-off integration of reference plus query is needed — de novo Harmony integration
  on all cells (e.g. ~4806 s at comparable scale) may be simpler than building a reusable
  reference
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Symphony
  role: 'Core algorithm: compresses the integrated Harmony object into a portable
    reference (buildReferenceFromHarmonyObj / buildReference) and later maps query
    cells in seconds'
  repo: https://github.com/immunogenomics/symphony
- name: Harmony
  role: Performs the batch/donor integration step on the reference PCA embedding (HarmonyMatrix
    with vars_use such as 'donor', nclust = 100, return_object = TRUE) before compression
  repo: https://github.com/immunogenomics/harmony
- name: singlecellmethods
  role: Companion methods package from immunogenomics used for variable gene selection
    and scaling utilities in the reference-building pipeline
  repo: https://github.com/immunogenomics/singlecellmethods
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:cc9ef0ae191ef9aba7a80271353519b5dac594257b59121cac462cbb626cc5b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/symphony
  tools:
  - Harmony
  - Symphony
  - singlecellmethods
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# single-cell-reference-atlas-building

## Summary

Build a large-scale, compressed, integrated single-cell reference atlas using Symphony: variable gene selection, scaling, PCA via SVD, Harmony integration, and reference compression into a portable object that supports mapping query cells in seconds.

## When to use

Use when constructing a reusable integrated reference from large single-cell RNA-seq (or CITE-seq) expression matrices so that future query datasets can be mapped and annotated without re-running de novo integration.

## When NOT to use

- The goal is novel cell type discovery in the query alone — Symphony does not replace de novo analyses of the query, and mapping metrics may be uninformative when a novel population resembles an existing reference population
- The reference lacks comprehensive cell type coverage (Condition I) — equivalence with de novo Harmony integration is not guaranteed and mapping is less flexible
- Only a one-off integration of reference plus query is needed — de novo Harmony integration on all cells (e.g. ~4806 s at comparable scale) may be simpler than building a reusable reference

## Inputs

- Reference gene-by-cell expression matrix (e.g. CITE-seq memory T-cell or PBMC 10x 3'v1/3'v2 data, downsampled 20K–500K cells or up to 1.39M)
- Reference cell metadata dataframe (cells x attributes, e.g. donor column used for integration)
- Optionally a precomputed Harmony object with PCA embedding (Z_pca_ref) and gene loadings

## Outputs

- Compressed Symphony reference object (.rds) containing harmonized embedding, gene means/std devs, and loadings
- Saved uwot UMAP model file for projecting query cells
- Benchmark timings (e.g. benchmark_timings.csv) recording reference build time, query mapping time, and de novo Harmony baseline

## How to apply

Load the gene-by-cell expression matrix (e.g. CITE-seq memory T-cell data) and cell metadata, then run the Symphony reference-building pipeline: select variable genes per group (e.g. vargenes_vst with topn per donor), scale genes with computed means and standard deviations (scaleDataWithStats), run PCA via SVD saving gene loadings, run Harmony integration with return_object = TRUE to obtain the full Harmony object, and finally compress the Harmony object into a Symphony reference with buildReferenceFromHarmonyObj (or use the one-call symphony::buildReference wrapper, e.g. K = 100 soft clusters, d = 20 PCs, theta = 2, vargenes_method = 'vst'). Optionally run UMAP and save the uwot model path so query cells can be projected into the reference embedding. The compression extracts reference-derived parameters from the Harmony mixture model so queries map in seconds rather than requiring de novo reintegration; benchmark wall-clock build times (e.g. ~5163 s for 500K cells, ~17.7 h for 1.39M cells) against de novo Harmony on the same data.

## Related tools

- **Symphony** (Core algorithm: compresses the integrated Harmony object into a portable reference (buildReferenceFromHarmonyObj / buildReference) and later maps query cells in seconds) — https://github.com/immunogenomics/symphony
- **Harmony** (Performs the batch/donor integration step on the reference PCA embedding (HarmonyMatrix with vars_use such as 'donor', nclust = 100, return_object = TRUE) before compression) — https://github.com/immunogenomics/harmony
- **singlecellmethods** (Companion methods package from immunogenomics used for variable gene selection and scaling utilities in the reference-building pipeline) — https://github.com/immunogenomics/singlecellmethods

## Examples

```
reference = symphony::buildReference(ref_exp, ref_metadata, vars = c('donor'), K = 100, vargenes_method = 'vst', vargenes_groups = 'donor', topn = 2000, theta = 2, d = 20, do_umap = TRUE, save_uwot_path = 'uwot_model_1')
```

## Evaluation signals

- Query mapping against the built reference completes in seconds (sub-second for a 10K-cell query, ~0.99 s; ~11.0 s against a 1.39M-cell reference), confirming the compression succeeded
- Reference build timing matches expected scaling: ~5163 s (86.1 min) for a 500,000-cell reference and ~17.7 h for the 1.39M-cell COVID-19 reference
- Harmony integration was run with return_object = TRUE so the full Harmony object (not just corrected embeddings) is available for buildReferenceFromHarmonyObj
- vargenes_means_sds table contains the required columns c('symbol', 'mean', 'stddev')
- Downstream mapping of a held-out query (e.g. 10x 5' PBMC) transfers reference-defined annotations consistent with de novo Harmony integration at comparable scale (~4806 s baseline)

## Limitations

- Condition I (comprehensive cell type coverage in the reference) is required for equivalence with de novo Harmony integration and is less flexible
- Mapping metrics may be less informative when a novel population is very similar to an existing reference population; identifying novel cell types that fail to map is an unsolved direction
- Multimodal integration (e.g. CCA-based CITE-seq strategy) is an active research area and may not be optimal for all applications; for scATAC-seq, optimal peak feature selection across differently analyzed reference and query is unclear
- Installation of R dependencies (systemfonts, textshaping, ragg required by ggrastr) can fail on missing freetype/harfbuzz system libraries and needs workarounds

## Evidence

- [discussion] Symphony compresses the reference by extracting relevant reference-derived parameters from the mixture model to map query cells in seconds.: "Symphony compresses the reference by extracting relevant reference-derived parameters from the mixture model to map query cells in seconds."
- [methods] Run Harmony integration with return_object = TRUE: "Run Harmony integration. It is important to set `return_object = TRUE`."
- [methods] Compress a Harmony object into a Symphony reference: "# Compress a Harmony object into a Symphony reference"
- [readme] vargenes_means_sds requires column names: [evidence span withheld — anchor status: external_doc]
- [abstract] an algorithm for building large-scale, integrated reference atlases in a convenient, portable format: "an algorithm for building large-scale, integrated reference atlases in a convenient, portable format"
