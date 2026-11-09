---
name: harmony-multidonor-integration-of-scrnaseq-batches
description: Use when building an integrated single-cell reference atlas from expression
  matrices spanning multiple donors or technologies (e.g., pancreatic islet donors
  across four protocols), where donor/batch covariates must be removed before downstream
  reference mapping.
when_to_use_negative:
- The analysis goal is novel cell-type discovery in the query alone — Harmony-based
  reference mapping assumes comprehensive cell-type coverage in the reference (Condition
  I) and does not replace de novo analyses of the query
- The reference consists of a single donor/technology with no batch covariate to integrate
  — Harmony correction over donor is unnecessary
- Query data should not be merged into the reference embedding itself; this skill
  integrates the reference, while query mapping is a separate mapQuery step
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Harmony
  role: Performs the donor-aware integration of the reference PCA embedding (vars_use
    = donor, nclust = 100, max_iter = 20, return_object = TRUE) whose harmonized object
    is compressed into the Symphony reference
  repo: https://github.com/immunogenomics/symphony
- name: Symphony
  role: Compresses the harmonized Harmony object into a portable reference via buildReferenceFromHarmonyObj
    and later maps queries into the integrated embedding
  repo: https://github.com/immunogenomics/symphony
- name: singlecellmethods
  role: Provides the preprocessing utilities vargenes_vst, scaleDataWithStats, and
    irlba-based SVD used to prepare the reference PCs before Harmony
  repo: https://github.com/immunogenomics/singlecellmethods
- name: LISI
  role: Computes donor Local Inverse Simpson's Index on the joint embedding to quantify
    how well donors were mixed by Harmony integration
  repo: https://github.com/immunogenomics/LISI
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:858891b0fb0b52476013dfd59792957e51d4c37383e1451c22b2d540e833d3a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/symphony
  tools:
  - Harmony
  - LISI
  - Seurat
  - Symphony
  - scArches
  - singlecellmethods
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# harmony-multidonor-integration-of-scrnaseq-batches

## Summary

Integrate multi-donor or multi-technology scRNA-seq reference data with Harmony's corrected low-dimensional embedding (PCA + Harmony with donor as the batch covariate) prior to compressing it into a Symphony reference. Harmony explicitly models complex study design so donor effects are removed while preserving biological cell-type structure.

## When to use

Use when building an integrated single-cell reference atlas from expression matrices spanning multiple donors or technologies (e.g., pancreatic islet donors across four protocols), where donor/batch covariates must be removed before downstream reference mapping.

## When NOT to use

- The analysis goal is novel cell-type discovery in the query alone — Harmony-based reference mapping assumes comprehensive cell-type coverage in the reference (Condition I) and does not replace de novo analyses of the query
- The reference consists of a single donor/technology with no batch covariate to integrate — Harmony correction over donor is unnecessary
- Query data should not be merged into the reference embedding itself; this skill integrates the reference, while query mapping is a separate mapQuery step

## Inputs

- Normalized reference scRNA-seq expression matrix (log(CP10k+1)), genes x cells, spanning multiple donors/technologies (e.g., pancreatic islet reference across four technologies)
- Reference cell metadata with a donor column used as the Harmony batch covariate
- Variable genes and their per-group means/standard deviations from vargenes_vst (topn = 1000 per donor group)

## Outputs

- Harmonized low-dimensional embedding (Harmony object returned via return_object = TRUE) with donor effects regressed out
- Compressed Symphony reference object (.rds) produced by buildReferenceFromHarmonyObj, plus a saved uwot model for UMAP projection
- Gene loadings from the SVD (s$u) retained for query projection

## How to apply

First normalize the reference expression matrices consistently across technologies (log(CP10k+1)) and select variable genes per donor group with vargenes_vst (topn = 1000 per group). Scale the variable-gene expression with scaleDataWithStats using stored gene means and standard deviations, then run PCA via irlba SVD with d = 20 PCs. Run Harmony on the PC embedding with vars_use = donor, nclust = 100, max_iter = 20, and critically set return_object = TRUE so the full Harmony object is retained. The harmonized embedding is then passed to buildReferenceFromHarmonyObj to compress it into a portable Symphony reference; donor mixing is later assessed with donor LISI on the joint embedding. The rationale is that Harmony explicitly models complex study design, removing donor/technology confounds while keeping cell-type structure intact for annotation transfer.

## Related tools

- **Harmony** (Performs the donor-aware integration of the reference PCA embedding (vars_use = donor, nclust = 100, max_iter = 20, return_object = TRUE) whose harmonized object is compressed into the Symphony reference) — https://github.com/immunogenomics/symphony
- **Symphony** (Compresses the harmonized Harmony object into a portable reference via buildReferenceFromHarmonyObj and later maps queries into the integrated embedding) — https://github.com/immunogenomics/symphony
- **singlecellmethods** (Provides the preprocessing utilities vargenes_vst, scaleDataWithStats, and irlba-based SVD used to prepare the reference PCs before Harmony) — https://github.com/immunogenomics/singlecellmethods
- **LISI** (Computes donor Local Inverse Simpson's Index on the joint embedding to quantify how well donors were mixed by Harmony integration) — https://github.com/immunogenomics/LISI

## Examples

```
harmony_obj <- harmony::HarmonyMatrix(pca_embedding, meta_data, vars_use = 'donor', nclust = 100, max_iter = 20, return_object = TRUE); reference <- buildReferenceFromHarmonyObj(harmony_obj, meta_data, vargenes_means_sds, loadings)
```

## Evaluation signals

- Donor LISI computed on the joint reference+query embedding should approximate 2.72 for the pancreatic islet experiment, indicating donor mixing
- Harmony must be run with return_object = TRUE; a default call that discards the object cannot be passed to buildReferenceFromHarmonyObj
- Downstream Symphony mapping of the multi-donor inDrop query should achieve human cell-type F1 ≈ 0.96 and mouse F1 ≈ 0.95, outperforming Seurat anchor-based integration and scArches
- Integration should hold both between reference and query and among donors within the query (as reported in Fig. 4 and Supplementary Tables 8–10)

## Limitations

- Condition I, which requires comprehensive cell type coverage in the reference, is not flexible and limits equivalence with de novo Harmony integration
- Reference compression means mapping works only when the query's cell types are represented; identification of novel cell types that fail to map remains an unsolved challenge
- Harmony integration is de novo for the reference: the donor covariate must be known and supplied in the metadata for vars_use

## Evidence

- [methods] Run Harmony integration. It is important to set `return_object = TRUE`.: "Run Harmony integration. It is important to set `return_object = TRUE`."
- [methods] Compress a Harmony object into a Symphony reference: "# Compress a Harmony object into a Symphony reference"
- [intro] Our group previously developed Harmony, a fast, accurate, and well-reviewed method that is able to explicitly model complex study design: "Our group previously developed Harmony15, a fast, accurate, and well-reviewed method20 that is able to explicitly model complex study design"
- [methods] Run PCA (using SVD), save gene loadings (`s$u`): "Run PCA (using SVD), save gene loadings (`s$u`)"
- [discussion] Symphony compresses the reference by extracting relevant reference-derived parameters from the mixture model to map query cells in seconds.: "Symphony compresses the reference by extracting relevant reference-derived parameters from the mixture model to map query cells in seconds."
