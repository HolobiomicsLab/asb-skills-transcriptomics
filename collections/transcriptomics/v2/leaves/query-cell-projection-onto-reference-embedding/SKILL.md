---
name: query-cell-projection-onto-reference-embedding
description: Use when a normalized query dataset (e.g., log(CP10k+1)) must be placed
  into an existing integrated reference embedding — e.g., a 10x 5' PBMC query against
  3'v1/3'v2 donors, or annotation/trajectory localization against a saved Symphony
  reference.
when_to_use_negative:
- The goal is novel cell-type discovery — Symphony does not replace de novo analyses
  of the query alone, and its metrics are uninformative for novel populations similar
  to existing reference types
- The reference does not comprehensively cover the query's cell types (Condition I),
  so query cells may map spuriously
- Reference and query were analyzed with different peak sets (scATAC-seq), where the
  optimal feature selection for mapping is undefined
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Symphony
  role: Provides mapQuery for projecting query cells into the compressed reference
    embedding and knnPredict for annotation transfer
  repo: https://github.com/immunogenomics/symphony
- name: Harmony
  role: Produces the integrated reference embedding (return_object = TRUE, theta =
    2, nclust = 100) whose harmonized centroids anchor query mapping
- name: irlba
  role: Runs PCA by SVD (nv = 20) on scaled variable genes, saving the gene loadings
    (s$u) used to project query cells
- name: symphonypy
  role: Python port of the Symphony R package for equivalent query mapping in Python
    workflows
  repo: https://github.com/potulabe/symphonypy
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:19164a193f00173410d52818a04e0e2f6be0e0ed1762e0320414d873bf99e872
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/symphony
  tools:
  - Harmony
  - Symphony
  - irlba
  - symphonypy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# query-cell-projection-onto-reference-embedding

## Summary

Project query single-cell expression data into a precomputed Symphony reference embedding by scaling query genes with stored reference means/standard deviations, projecting through stored PCA loadings, softly assigning cells to harmonized soft-cluster centroids, and applying mixture-model batch correction — enabling annotation transfer in seconds without re-running integration.

## When to use

Use when a normalized query dataset (e.g., log(CP10k+1)) must be placed into an existing integrated reference embedding — e.g., a 10x 5' PBMC query against 3'v1/3'v2 donors, or annotation/trajectory localization against a saved Symphony reference.

## When NOT to use

- The goal is novel cell-type discovery — Symphony does not replace de novo analyses of the query alone, and its metrics are uninformative for novel populations similar to existing reference types
- The reference does not comprehensively cover the query's cell types (Condition I), so query cells may map spuriously
- Reference and query were analyzed with different peak sets (scATAC-seq), where the optimal feature selection for mapping is undefined

## Inputs

- Query expression matrix normalized as log(CP10k+1), e.g., the 10x 5' PBMC query from pbmcs_exprs_small
- Query cell metadata (e.g., pbmcs_meta_small) with donor/batch labels
- Compressed Symphony reference object storing mu, sigma, PCA loadings U, soft-cluster centroids C, assignments R, and cache (N_r, Y_cos)
- vargenes_means_sds table (variable-gene means/standard deviations, top 1000 genes per donor group via vargenes_vst)

## Outputs

- Query embedding Z in the harmonized reference low-dimensional space
- Zq_pca, the query projection in pre-Harmony PC space
- Soft cluster assignments R of query cells to reference centroids
- Reference UMAP coordinates for query cells (do_umap = TRUE)
- knnPredict cell-type annotations with per-cell confidence scores (e.g., median F1 0.83, accuracy 85.0% at k = 30)
- Query mapping results table (predictions, confidences, UMAP coordinates, e.g., query_mapping_results.csv)

## How to apply

Build or load a compressed Symphony reference whose stored components include soft cluster assignments R, harmonized soft-cluster centroids C, variable-gene means and standard deviations (mu, sigma), PCA gene loadings U, and the reference-dependent cache (N_r, Y_cos). Scale the query expression with the stored reference gene statistics (scaleDataWithStats), project the scaled query through the loadings into the pre-Harmony PC space (Zq_pca), softly assign query cells to reference centroids, and apply the mixture-of-experts batch correction to yield the query embedding Z (mapQuery, Algorithm 2). Optionally enable do_umap = TRUE to place query cells in reference UMAP coordinates. For annotation transfer, follow with knnPredict (k = 30, confidence = TRUE); for trajectory localization, average the coordinates of each query cell's 10 nearest reference neighbors on a reference-defined trajectory such as fetal liver FDG.

## Related tools

- **Symphony** (Provides mapQuery for projecting query cells into the compressed reference embedding and knnPredict for annotation transfer) — https://github.com/immunogenomics/symphony
- **Harmony** (Produces the integrated reference embedding (return_object = TRUE, theta = 2, nclust = 100) whose harmonized centroids anchor query mapping)
- **irlba** (Runs PCA by SVD (nv = 20) on scaled variable genes, saving the gene loadings (s$u) used to project query cells)
- **symphonypy** (Python port of the Symphony R package for equivalent query mapping in Python workflows) — https://github.com/potulabe/symphonypy

## Examples

```
query = mapQuery(query_exp, query_metadata, reference, vargenes_means_sds, do_umap = TRUE); query = knnPredict(query, reference, k = 30, confidence = TRUE)
```

## Evaluation signals

- Reimplemented query embedding Z matches the released Symphony mapQuery output (reference$Z_corr and query Z) within numerical tolerance on identical inputs (seed = 0)
- Zq_pca and soft cluster assignments R reproduce buildReferenceFromHarmonyObj/mapQuery outputs on the same cells
- Annotation transfer reaches the reported median cell type F1 of 0.83 and accuracy of 85.0% at k = 30 against reference-derived true labels
- MEM-lineage query cells recapitulate the fetal liver FDG trajectory with monotonic PPBP, HBB, and KIT marker gradients
- Reference compression outputs (R, C, N_r, Y_cos, mu, sigma, U) fully reconstruct buildReferenceFromHarmonyObj's stored components

## Limitations

- Identification of novel cell types that have failed to map is an unsolved future direction; mapping metrics may be less informative when a novel population is very similar to an existing reference population
- Condition I (comprehensive cell type coverage in the reference) limits flexibility and equivalence with de novo Harmony integration
- Symphony does not entirely supplant de novo analyses for users interested in novel cell type discovery

## Evidence

- [discussion] Symphony compresses the reference by extracting relevant reference-derived parameters from the mixture model to map query cells in seconds.: "Symphony compresses the reference by extracting relevant reference-derived parameters from the mixture model to map query cells in seconds."
- [methods] Scale data using calculated gene means and standard deviations: "Scale data using calculated gene means and standard deviations"
- [methods] Run PCA (using SVD), save gene loadings (`s$u`): "Run PCA (using SVD), save gene loadings (`s$u`)"
- [abstract] Symphony localizes query cells within a stable low-dimensional reference embedding, facilitating reproducible downstream transfer of reference-defined annotations to the query: "Symphony localizes query cells within a stable low-dimensional reference embedding, facilitating reproducible downstream transfer of reference-defined annotations to the query"
- [methods] Run Symphony's mapQuery() to project query cells into the harmonized reference embedding and into reference UMAP coordinates (do_umap = TRUE).: [evidence span withheld — anchor status: not_found]
