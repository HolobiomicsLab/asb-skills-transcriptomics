---
name: single-cell-reference-mapping
description: Use when a query scRNA-seq dataset (e.g., a new 10x technology or donor)
  must be placed into an existing integrated reference embedding for label transfer,
  without re-running de novo integration on all raw cells.
when_to_use_negative:
- When the goal is novel cell type discovery — Symphony does not replace de novo analyses
  of the query alone
- When the query contains populations very similar to an existing reference population
  — mapping metrics may be less informative about novelty
- When the reference lacks comprehensive cell type coverage (Condition I), which limits
  equivalence with de novo Harmony integration
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Symphony
  role: Builds the compressed reference (buildReference/buildReferenceFromHarmonyObj)
    and maps queries (mapQuery) with knnPredict label transfer
  repo: https://github.com/immunogenomics/symphony
- name: Harmony
  role: Performs the de novo integration of reference donors that Symphony compresses;
    must be run with return_object = TRUE
- name: irlba
  role: SVD-based PCA (nv = 20) of scaled variable-gene expression, saving gene loadings
    s$u
- name: immunogenomics/LISI
  role: Computes technology/donor LISI to quantify integration of query and reference
    cells (e.g., 2.12 vs 2.15 for de novo Harmony)
  repo: https://github.com/immunogenomics/LISI
- name: singlecellmethods
  role: Utility functions such as vargenes_vst, scaleDataWithStats, and calcknncorr
    used in the reference-building workflow
  repo: https://github.com/immunogenomics/singlecellmethods
- name: symphonypy
  role: Python port of the Symphony R package for users mapping queries in Python
  repo: https://github.com/potulabe/symphonypy
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_005
  - task_007
  - task_006
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:c9f4221c76d529c04620ba043e04feceac2168e978752c7b3daf97b3e7dac0f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/immunogenomics/symphony
  tools:
  - Harmony
  - Seurat
  - Symphony
  - dplyr
  - immunogenomics/LISI
  - irlba
  - scArches
  - singlecellmethods
  - symphonypy
  - uwot
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6818181818181818
n_claims: 22
---

# single-cell-reference-mapping

## Summary

Build a compressed, integrated single-cell reference atlas with Symphony (variable-gene selection, PCA, Harmony, reference compression) and map held-out query cells into the stable harmonized embedding to transfer annotations within seconds.

## When to use

Use when a query scRNA-seq dataset (e.g., a new 10x technology or donor) must be placed into an existing integrated reference embedding for label transfer, without re-running de novo integration on all raw cells.

## When NOT to use

- When the goal is novel cell type discovery — Symphony does not replace de novo analyses of the query alone
- When the query contains populations very similar to an existing reference population — mapping metrics may be less informative about novelty
- When the reference lacks comprehensive cell type coverage (Condition I), which limits equivalence with de novo Harmony integration

## Inputs

- Reference scRNA-seq expression matrix (e.g., pbmcs_exprs_small.rda) and cell metadata (pbmcs_meta_small.rda) with donor/batch variables
- Held-out query expression matrix normalized as log(CP10k+1) with matching gene names
- Reference cell type labels for training the annotation-transfer classifier
- Optionally, a pre-computed Harmony object (return_object = TRUE) for buildReferenceFromHarmonyObj

## Outputs

- Compressed Symphony reference object (.rds) storing soft cluster assignments R, harmonized centroids C, cache (N_r, Y_cos), gene means/sds, and PCA loadings U
- Query object with harmonized embedding Z, Zq_pca coordinates, soft cluster assignments, UMAP coordinates, predicted labels and confidence scores
- Benchmark metrics (technology LISI, k-NN-correlation fractions, accuracy/F1) e.g., pbmc_mapping_benchmark_metrics.csv or f1_benchmark_results.csv

## How to apply

Normalize reference expression (log(CP10k+1)), select variable genes with vargenes_vst (topn = 1000 per donor group), scale with scaleDataWithStats using stored gene means/standard deviations, and run PCA by SVD with irlba (nv = 20), saving the gene loadings. Integrate with Harmony over the donor variable (nclust = 100, max_iter = 20, return_object = TRUE) and compress into a Symphony reference via symphony::buildReference (or buildReferenceFromHarmonyObj), optionally saving a uwot UMAP model. Normalize the query to match the reference convention, then project it with mapQuery (do_normalize = FALSE if pre-normalized, do_umap = TRUE), which scales the query with reference gene statistics, projects through the PCA loadings, softly assigns cells to reference centroids, and applies the mixture-of-experts batch correction. Transfer annotations with knnPredict (k = 5 with confidence = TRUE, or k = 30 for finer fetal liver-style transfer). Assess fidelity with technology/donor LISI (immunogenomics/LISI), per-cell k-NN correlation (calcknncorr, k = 500, threshold 0.4), and accuracy/per-class F1 against held-out labels.

## Related tools

- **Symphony** (Builds the compressed reference (buildReference/buildReferenceFromHarmonyObj) and maps queries (mapQuery) with knnPredict label transfer) — https://github.com/immunogenomics/symphony
- **Harmony** (Performs the de novo integration of reference donors that Symphony compresses; must be run with return_object = TRUE)
- **irlba** (SVD-based PCA (nv = 20) of scaled variable-gene expression, saving gene loadings s$u)
- **immunogenomics/LISI** (Computes technology/donor LISI to quantify integration of query and reference cells (e.g., 2.12 vs 2.15 for de novo Harmony)) — https://github.com/immunogenomics/LISI
- **singlecellmethods** (Utility functions such as vargenes_vst, scaleDataWithStats, and calcknncorr used in the reference-building workflow) — https://github.com/immunogenomics/singlecellmethods
- **symphonypy** (Python port of the Symphony R package for users mapping queries in Python) — https://github.com/potulabe/symphonypy

## Examples

```
reference = symphony::buildReference(exprs_selected, ref_metadata, vars = c('donor'), K = 100, verbose = TRUE, do_umap = TRUE, do_normalize = FALSE); query = mapQuery(query_exp, query_meta, reference, do_normalize = FALSE, do_umap = TRUE)
```

## Evaluation signals

- Technology LISI of the Symphony-mapped embedding closely matches de novo Harmony (reported ~2.12 vs 2.15) in leave-one-technology-out PBMC experiments
- Majority of query cells exceed k-NN correlation 0.4 against the de novo reference embedding at k = 500 (reported 87.7% vs 50.2% PCA-only baseline)
- 5-NN annotation transfer (knnPredict, k = 5) reaches ~97.5% accuracy with median per-class F1 ~0.97 on cross-technology PBMC mapping
- Reimplemented Algorithm 1/2 (compression + projection) reproduces reference Z_corr and query Z, Zq_pca, and R within numerical tolerance of the released package (seed = 0)
- Per-cell and per-cluster Mahalanobis mapping metrics detect ablated novel cell types with AUC up to 0.997/1.0 when query states are very different from the reference

## Limitations

- Identification of novel cell types that have failed to map remains an important future direction; metrics can be less informative when the novel population closely resembles an existing reference population
- Condition I (comprehensive cell type coverage in the reference) is less flexible and limits equivalence with de novo Harmony integration
- Annotation transfer from reference to query remains an open area of research; Symphony does not supplant de novo query-only analyses for discovery

## Evidence

- [abstract] We present Symphony ( https://github.com/immunogenomics/symphony ), an algorithm for building large-scale, integrated reference atlases: "We present Symphony ( https://github.com/immunogenomics/symphony ), an algorithm for building large-scale, integrated reference atlases"
- [discussion] Symphony compresses the reference by extracting relevant reference-derived parameters from the mixture model to map query cells in seconds.: "Symphony compresses the reference by extracting relevant reference-derived parameters from the mixture model to map query cells in seconds."
- [methods] vargenes_vst with topn = 1000 per donor group: "vargenes_vst(ref_exp_full, groups = as.character(ref_metadata[['donor']]), topn = 1000)"
- [methods] Run Harmony integration. It is important to set `return_object = TRUE`.: "Run Harmony integration. It is important to set `return_object = TRUE`."
- [methods] query = knnPredict(query,       # query object: "query = knnPredict(query,       # query object"
