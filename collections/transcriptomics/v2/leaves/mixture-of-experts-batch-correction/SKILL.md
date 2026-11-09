---
name: mixture-of-experts-batch-correction
description: Use when mapping a query scRNA-seq dataset (e.g. 10x 5' PBMCs) into a
  pre-built Symphony/Harmony reference embedding (built from 10x 3'v1/3'v2 donors)
  and query batch effects must be corrected using stored reference parameters only.
when_to_use_negative:
- You need novel cell type discovery — Symphony does not replace de novo analyses
  of the query alone, and mapping metrics are less informative when a novel population
  closely resembles an existing reference population
- The reference lacks comprehensive cell type coverage for the query context (Condition
  I), limiting equivalence with de novo Harmony integration
- Reference and query datasets use different feature sets with no defined optimal
  selection (e.g. scATAC-seq peaks), where mapping is not immediately clear how to
  optimally perform
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Symphony
  role: Provides buildReferenceFromHarmonyObj (reference compression, Algorithm 1)
    and mapQuery (query projection with mixture-of-experts batch correction, Algorithm
    2)
  repo: https://github.com/immunogenomics/symphony
- name: Harmony
  role: Integrates the reference PCA embedding (theta = 2, nclust = 100, max_iter
    = 20, return_object = TRUE) whose mixture model components are compressed into
    the Symphony reference
- name: irlba
  role: Runs PCA by SVD (nv = 20) on scaled reference expression, saving gene loadings
    used to project the query into PC space
- name: singlecellmethods
  role: Supplies vargenes_vst and scaleDataWithStats for variable gene selection and
    gene scaling with reference means/sds
  repo: https://github.com/immunogenomics/singlecellmethods
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:48c040cebb524cc6c2599d592a1a5e7a62b7d38aac1fc9b381d4297cc1fc4af3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/symphony
  tools:
  - Harmony
  - Symphony
  - irlba
  - singlecellmethods
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# mixture-of-experts-batch-correction

## Summary

Symphony's query-mapping step that softly assigns query cells to reference mixture-model components and regresses out query batch effects using stored reference parameters, placing cells in the reference's harmonized low-dimensional embedding. It enables batch-corrected reference mapping without re-running integration on the full reference.

## When to use

Use when mapping a query scRNA-seq dataset (e.g. 10x 5' PBMCs) into a pre-built Symphony/Harmony reference embedding (built from 10x 3'v1/3'v2 donors) and query batch effects must be corrected using stored reference parameters only.

## When NOT to use

- You need novel cell type discovery — Symphony does not replace de novo analyses of the query alone, and mapping metrics are less informative when a novel population closely resembles an existing reference population
- The reference lacks comprehensive cell type coverage for the query context (Condition I), limiting equivalence with de novo Harmony integration
- Reference and query datasets use different feature sets with no defined optimal selection (e.g. scATAC-seq peaks), where mapping is not immediately clear how to optimally perform

## Inputs

- Query gene expression matrix (e.g. pbmcs_exprs_small subset to the 10x 5' donor)
- Pre-compressed Symphony reference object containing mixture model components, centroids C, cache (N_r, Y_cos), gene means/sds (mu, sigma), and PCA loadings U
- Reference gene means and standard deviations from vargenes_vst (top 1000 variable genes per donor group)
- Query metadata (e.g. pbmcs_meta_small)

## Outputs

- Query embedding Z placing query cells in the harmonized reference embedding
- Query PCA projection Zq_pca into pre-Harmony PC space
- Soft-cluster assignments R of query cells to reference centroids
- Transferred cell type annotations via knnPredict (optional downstream step)

## How to apply

First compress the reference (Algorithm 1, buildReferenceFromHarmonyObj) by extracting the soft cluster assignments R, harmonized soft cluster centroids C, the reference-dependent cache (N_r, Y_cos), variable-gene means/sds (mu, sigma), and PCA gene loadings (U) from the Harmony object (run with theta = 2, nclust = 100, max_iter = 20, return_object = TRUE). Then during mapping (Algorithm 2, mapQuery), scale the query gene expression with the stored reference gene means/standard deviations via scaleDataWithStats, project it through the loadings into the pre-Harmony PC space (Zq_pca), compute soft-cluster assignments of query cells to the reference centroids, and apply the mixture-of-experts correction using the stored mixture model components to regress out query batch effects, yielding the query embedding Z. Validate by comparing against the released Symphony package's mapQuery output (reference$Z_corr and query Z) with seed = 0 on identical inputs and confirming numerical agreement within tolerance.

## Related tools

- **Symphony** (Provides buildReferenceFromHarmonyObj (reference compression, Algorithm 1) and mapQuery (query projection with mixture-of-experts batch correction, Algorithm 2)) — https://github.com/immunogenomics/symphony
- **Harmony** (Integrates the reference PCA embedding (theta = 2, nclust = 100, max_iter = 20, return_object = TRUE) whose mixture model components are compressed into the Symphony reference)
- **irlba** (Runs PCA by SVD (nv = 20) on scaled reference expression, saving gene loadings used to project the query into PC space)
- **singlecellmethods** (Supplies vargenes_vst and scaleDataWithStats for variable gene selection and gene scaling with reference means/sds) — https://github.com/immunogenomics/singlecellmethods

## Examples

```
query = mapQuery(query_exp, query_meta, reference, vars = 'donor')  # then knnPredict(query, referencemodel, ref_metadata, k = 50)
```

## Evaluation signals

- The reimplemented query embedding Z matches the released Symphony package's mapQuery output within numerical tolerance on identical inputs (seed = 0)
- Reimplemented reference Z_corr and soft cluster assignments R agree with reference$Z_corr and the package's outputs on the same cells
- Query projection Zq_pca agrees numerically with the package's Zq_pca when scaling with the same reference gene statistics and loadings
- Query cells localize within the reference embedding such that reference-defined annotations transfer (e.g. via knnPredict) with expected cell type composition for the donor

## Limitations

- Symphony's mapping metrics may be less informative when a novel query population is very similar to an existing reference population
- Symphony does not replace de novo analyses of the query alone for users interested in novel cell type discovery
- Condition I (comprehensive cell type coverage in the reference) is not flexible and limits equivalence with de novo Harmony integration

## Evidence

- [discussion] Symphony compresses the reference by extracting relevant reference-derived parameters from the mixture model to map query cells in seconds.: "Symphony compresses the reference by extracting relevant reference-derived parameters from the mixture model to map query cells in seconds."
- [abstract] Symphony localizes query cells within a stable low-dimensional reference embedding, facilitating reproducible downstream transfer of reference-defined annotations: "Symphony localizes query cells within a stable low-dimensional reference embedding, facilitating reproducible downstream transfer of reference-defined annotations to the query"
- [methods] Run Harmony integration. It is important to set `return_object = TRUE`.: "Run Harmony integration. It is important to set `return_object = TRUE`."
- [methods] # Compress a Harmony object into a Symphony reference: "# Compress a Harmony object into a Symphony reference"
- [readme] We present Symphony ( https://github.com/immunogenomics/symphony ), an algorithm for building large-scale, integrated reference atlases: "Efficient and precise single-cell reference atlas mapping with Symphony"
