---
name: mixture-model-soft-clustering
description: Use when compressing a Harmony-integrated reference atlas into a portable
  Symphony reference or when mapping query cells into that reference's embedding via
  soft cluster assignment.
when_to_use_negative:
- The goal is discovering novel cell types absent from the reference — Symphony's
  mapping metrics may be uninformative when a novel population closely resembles an
  existing reference population, and de novo analysis of the query alone is needed
- The reference lacks comprehensive cell type coverage (Condition I), where soft-cluster-based
  mapping is less flexible than de novo Harmony integration
- The analysis requires de novo integration of all datasets together rather than reference-based
  mapping (e.g. BBKNN, Seurat anchors, or Harmony on the full combined data)
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Symphony
  role: Implements the mixture-model compression (buildReferenceFromHarmonyObj) and
    soft-cluster query mapping (mapQuery, knnPredict)
  repo: https://github.com/immunogenomics/symphony
- name: Harmony
  role: Provides the integrated reference embedding and mixture model whose soft clusters
    and centroids are compressed into the Symphony reference
- name: irlba
  role: Runs PCA by SVD (nv = 20) on scaled reference expression, saving gene loadings
    used to project query cells
- name: singlecellmethods
  role: Provides vargenes_vst and scaleDataWithStats used to prepare reference expression
    before PCA and compression
  repo: https://github.com/immunogenomics/singlecellmethods
- name: symphonypy
  role: Python port of Symphony reference mapping for users outside R
  repo: https://github.com/potulabe/symphonypy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:c02314f0cfaaca6796154ad485e984f5d8f27151c845f07da5190fb91ade9bad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/symphony
  tools:
  - Harmony
  - Symphony
  - irlba
  - singlecellmethods
  - symphonypy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Mixture-model soft clustering

## Summary

Symphony compresses a Harmony-integrated single-cell reference by fitting a mixture model with soft cluster assignments and centroids, then reuses the stored mixture components to softly assign query cells to reference clusters during mapping. This soft-clustering step is what makes million-cell references mappable in seconds without access to raw per-cell data.

## When to use

Use when compressing a Harmony-integrated reference atlas into a portable Symphony reference or when mapping query cells into that reference's embedding via soft cluster assignment.

## When NOT to use

- The goal is discovering novel cell types absent from the reference — Symphony's mapping metrics may be uninformative when a novel population closely resembles an existing reference population, and de novo analysis of the query alone is needed
- The reference lacks comprehensive cell type coverage (Condition I), where soft-cluster-based mapping is less flexible than de novo Harmony integration
- The analysis requires de novo integration of all datasets together rather than reference-based mapping (e.g. BBKNN, Seurat anchors, or Harmony on the full combined data)

## Inputs

- Harmony object from integrated reference PCA (return_object = TRUE), e.g. from 10x 3'v1 and 3'v2 PBMC donors
- Reference PCA gene loadings (s$u from irlba SVD, nv = 20)
- Variable gene means and standard deviations from vargenes_vst (top 1000 genes per donor group)
- Query expression matrix (e.g. 10x 5' PBMC donor, pbmcs_exprs_small)
- Query metadata (pbmcs_meta_small) with batch/celltype variables

## Outputs

- Symphony reference object storing soft cluster assignments R, centroids C, N_r, Y_cos, mu, sigma, and loadings U
- Query embedding Z placing query cells in the reference low-dimensional space
- Query pre-correction PC projection Zq_pca
- Soft cluster assignment probabilities of query cells over reference clusters
- Predicted query annotations via knnPredict

## How to apply

After running Harmony on the reference PCA embedding (return_object = TRUE, theta = 2, nclust = 100, max_iter = 20), extract the mixture model components: soft cluster assignments R, harmonized soft cluster centroids C, cell counts N_r, the harmonized embedding Y_cos, variable-gene means/sds (mu, sigma), and PCA gene loadings (U) via buildReferenceFromHarmonyObj. During query mapping with mapQuery, scale the query expression with the stored reference gene statistics, project query cells through the gene loadings into the pre-Harmony PC space (Zq_pca), compute soft-cluster assignments of query cells against the stored centroids, and apply the mixture-of-experts batch correction to place query cells in the reference embedding. Soft assignments (rather than hard labels) preserve uncertainty and enable the regression of query batch effects using stored mixture model components. Validate by comparing reimplemented R, Zq_pca, and query Z against the released Symphony package outputs (seed = 0) on identical inputs.

## Related tools

- **Symphony** (Implements the mixture-model compression (buildReferenceFromHarmonyObj) and soft-cluster query mapping (mapQuery, knnPredict)) — https://github.com/immunogenomics/symphony
- **Harmony** (Provides the integrated reference embedding and mixture model whose soft clusters and centroids are compressed into the Symphony reference)
- **irlba** (Runs PCA by SVD (nv = 20) on scaled reference expression, saving gene loadings used to project query cells)
- **singlecellmethods** (Provides vargenes_vst and scaleDataWithStats used to prepare reference expression before PCA and compression) — https://github.com/immunogenomics/singlecellmethods
- **symphonypy** (Python port of Symphony reference mapping for users outside R) — https://github.com/potulabe/symphonypy

## Examples

```
reference = buildReferenceFromHarmonyObj(harmony_obj, meta_data, vars = c('donor'), verbose = TRUE); query = mapQuery(query_exp, query_meta, reference, vars = c('donor')); query = knnPredict(query, reference, 'cell_type', k = 5)
```

## Evaluation signals

- Reimplemented reference Z_corr and query Z match the released Symphony package outputs (reference$Z_corr and query Z) within numerical tolerance on identical inputs (seed = 0)
- Recomputed Zq_pca matches the package's pre-correction PC projection of the query
- Soft cluster assignment matrix R reproduces the package's per-cell cluster probabilities over reference clusters
- Query cells localize within the reference embedding coherently with transferred annotations (knnPredict cell-type labels consistent with known query metadata)
- Compression is lossy but faithful: the stored components (R, C, N_r, Y_cos, mu, sigma, U) alone suffice to map a new query without accessing raw reference cells

## Limitations

- Identification of novel cell types that failed to map remains unsolved; mapping metrics may be less informative when a novel population is very similar to an existing reference population
- Condition I (comprehensive cell type coverage in the reference) is required and limits equivalence with de novo Harmony integration
- Symphony does not supplant de novo analyses of the query alone for novel cell type discovery

## Evidence

- [discussion] Symphony compresses the reference by extracting relevant reference-derived parameters from the mixture model to map query cells in seconds.: "Symphony compresses the reference by extracting relevant reference-derived parameters from the mixture model to map query cells in seconds."
- [intro] We present Symphony, a novel algorithm to compress a large, integrated reference and map query cells to a precise location in the reference embedding within seconds.: "We present Symphony, a novel algorithm to compress a large, integrated reference and map query cells to a precise location in the reference embedding within seconds."
- [intro] Here, we define reference mapping to mean placing query cells within the same embedding as integrated reference cells without requiring access to the raw data on all individual reference cells.: "Here, we define reference mapping to mean placing query cells within the same embedding as integrated reference cells without requiring access to the raw data on all individual reference cells."
- [methods] Scale data using calculated gene means and standard deviations: "Scale data using calculated gene means and standard deviations"
- [methods] Run Harmony integration. It is important to set `return_object = TRUE`.: "Run Harmony integration. It is important to set `return_object = TRUE`."
- [methods] # Compress a Harmony object into a Symphony reference: "# Compress a Harmony object into a Symphony reference"
