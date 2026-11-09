---
name: harmony-batch-integration
description: Use when a held-out query dataset from a different technology or batch
  must be projected into a stable reference embedding built from the other datasets,
  and the mapping quality must be quantified against a de novo Harmony integration.
when_to_use_negative:
- The goal is de novo discovery of novel cell types in the query — Symphony does not
  replace de novo analyses of the query alone, and its mapping metrics are uninformative
  when a novel population resembles an existing reference population
- The query dataset lacks comprehensive cell-type coverage in the reference (Condition
  I), since label transfer and mapping fidelity degrade when reference coverage is
  incomplete
- The input is already an integrated embedding and no batch/technology confounder
  needs to be removed
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Harmony
  role: Performs the de novo integration of the three 10x PBMC technologies (K = 100,
    vars = 'donor') whose embedding serves both as the base for Symphony reference
    compression and as the gold-standard benchmark for mapping fidelity
  repo: github.com/immunogenomics/symphony
- name: Symphony
  role: Compresses the Harmony object into a portable reference (buildReference /
    buildReferenceFromHarmonyObj) and maps held-out queries (mapQuery) with 5-NN label
    transfer (knnPredict)
  repo: github.com/immunogenomics/symphony
- name: immunogenomics/LISI
  role: Computes technology LISI to quantify how well query and reference cells mix
    in the Symphony embedding versus de novo Harmony
  repo: github.com/immunogenomics/LISI
- name: immunogenomics/singlecellmethods
  role: Provides supporting single-cell methods (vargenes_vst, scaleDataWithStats,
    calcknncorr) used in reference building and benchmarking
  repo: github.com/immunogenomics/singlecellmethods
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:9c438116ee6d4f29d5ff955765c94f37db114a9faade5b6900ccf7246d16f1c0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Harmony
  - Symphony
  - dplyr
  - immunogenomics/LISI
  - immunogenomics/singlecellmethods
  - irlba
  - uwot
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# harmony-batch-integration

## Summary

Build a Harmony-integrated embedding across batches or technologies (here, three 10x PBMC datasets from 3'v1, 3'v2, and 5' chemistry), then compress it into a Symphony reference to map held-out query cells and benchmark the mapping against de novo Harmony integration using LISI, k-NN correlation, and 5-NN label-transfer accuracy.

## When to use

Use when a held-out query dataset from a different technology or batch must be projected into a stable reference embedding built from the other datasets, and the mapping quality must be quantified against a de novo Harmony integration.

## When NOT to use

- The goal is de novo discovery of novel cell types in the query — Symphony does not replace de novo analyses of the query alone, and its mapping metrics are uninformative when a novel population resembles an existing reference population
- The query dataset lacks comprehensive cell-type coverage in the reference (Condition I), since label transfer and mapping fidelity degrade when reference coverage is incomplete
- The input is already an integrated embedding and no batch/technology confounder needs to be removed

## Inputs

- pbmcs_exprs_small.rda expression matrix (10x PBMC counts across 3'v1, 3'v2, 5' technologies)
- pbmcs_meta_small.rda cell metadata with donor and technology labels
- Held-out query expression matrix for the single held-out technology
- Cell-type annotations for reference cells (for 5-NN label transfer)

## Outputs

- Symphony reference objects saved via saveRDS (one per leave-one-technology-out fold)
- Symphony-mapped query embedding with UMAP coordinates and transferred cell-type labels with confidence scores
- De novo Harmony-integrated embedding of all three datasets for comparison
- pbmc_mapping_benchmark_metrics.csv with technology LISI, k-NN-correlation fractions (k = 500, >0.4), and 5-NN accuracy/per-class F1 per fold

## How to apply

Load pbmcs_exprs_small and pbmcs_meta_small and define three leave-one-technology-out splits so each of 3'v1, 3'v2, and 5' is held out once as the query. For each split, select variable genes with vargenes_vst (groups by donor, topn = 1000), scale with scaleDataWithStats, run SVD-based PCA (d = 20), and run Harmony with K = 100 clusters and vars = 'donor' (setting return_object = TRUE is critical when building from a Harmony object), then compress via symphony::buildReference with do_normalize = FALSE and save with saveRDS. Map the held-out query with mapQuery (do_normalize = FALSE, do_umap = TRUE) and transfer cell-type labels with knnPredict using k = 5 neighbors and confidence = TRUE. Rationale: Symphony compresses the Harmony mixture-model parameters so query cells localize in the reference embedding in seconds, approximating what full de novo Harmony of all three datasets would produce.

## Related tools

- **Harmony** (Performs the de novo integration of the three 10x PBMC technologies (K = 100, vars = 'donor') whose embedding serves both as the base for Symphony reference compression and as the gold-standard benchmark for mapping fidelity) — github.com/immunogenomics/symphony
- **Symphony** (Compresses the Harmony object into a portable reference (buildReference / buildReferenceFromHarmonyObj) and maps held-out queries (mapQuery) with 5-NN label transfer (knnPredict)) — github.com/immunogenomics/symphony
- **immunogenomics/LISI** (Computes technology LISI to quantify how well query and reference cells mix in the Symphony embedding versus de novo Harmony) — github.com/immunogenomics/LISI
- **immunogenomics/singlecellmethods** (Provides supporting single-cell methods (vargenes_vst, scaleDataWithStats, calcknncorr) used in reference building and benchmarking) — github.com/immunogenomics/singlecellmethods

## Examples

```
reference = symphony::buildReference(exprs, meta, var_genes, 'donor', K = 100, d = 20, do_normalize = FALSE); query = mapQuery(query_exp, query_meta, reference, do_normalize = FALSE, do_umap = TRUE)
```

## Evaluation signals

- Technology LISI of the Symphony-mapped embedding closely matches the de novo Harmony value (reported mean 2.12 vs 2.15 across three folds)
- Majority of query cells exceed k-NN correlation 0.4 at k = 500 against the de novo reference embedding (reported 87.7% vs 50.2% for the PCA-only baseline)
- 5-NN label transfer (knnPredict, k = 5, confidence = TRUE) reaches ~97.5% accuracy with median per-class F1 ~0.97 against held-out labels
- Harmony was run with return_object = TRUE before compression — omitting this breaks buildReferenceFromHarmonyObj
- All three leave-one-technology-out folds compile into pbmc_mapping_benchmark_metrics.csv with LISI, k-NN-corr fraction, and accuracy/F1 columns

## Limitations

- Mapping metrics are less informative when a novel query population is very similar to an existing reference population, so failure to map may go undetected
- Equivalence with de novo Harmony integration depends on Condition I (comprehensive cell type coverage in the reference), which is not flexible
- Symphony does not replace de novo analysis of the query alone for novel cell type discovery
- Transfer of annotations from reference to query remains an open research area (automated classifiers such as scmap are noted as rigid alternatives)

## Evidence

- [abstract] We present Symphony ( https://github.com/immunogenomics/symphony ), an algorithm for building large-scale, integrated reference atlases: "We present Symphony ( https://github.com/immunogenomics/symphony ), an algorithm for building large-scale, integrated reference atlases"
- [methods] Run Harmony integration with return_object = TRUE: "Run Harmony integration. It is important to set `return_object = TRUE`."
- [methods] vargenes_vst grouped by donor with topn 1000: "var_genes = vargenes_vst(ref_exp_full, groups = as.character(ref_metadata[['donor']]), topn = 1000)"
- [discussion] Symphony compresses the reference to map query cells in seconds: "Symphony compresses the reference by extracting relevant reference-derived parameters from the mixture model to map query cells in seconds."
- [readme] LISI computes local inverse Simpson's index for batch mixing: "we provide an algorithm for computing a Local Inverse Simpson’s
Index (LISI)."
