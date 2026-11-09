---
name: multi-species-pancreatic-islet-annotation-analysis
description: Use when a multi-donor or cross-species scRNA-seq query (e.g. inDrop
  human+mouse islets) must be mapped onto an existing human pancreatic islet reference
  and its cell types annotated via reference-label transfer.
when_to_use_negative:
- The goal is discovering novel query cell types absent from the reference — Symphony
  requires comprehensive cell type coverage in the reference and does not replace
  de novo analyses of the query alone
- The query contains a novel population very similar to an existing reference population,
  where mapping metrics may be less informative
- Reference and query are scATAC-seq peak matrices analyzed with different peak sets,
  where optimal feature selection for mapping is undefined
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Symphony
  role: Builds the compressed reference from the harmonized object and maps the multi-species
    query into the reference embedding with annotation transfer
  repo: https://github.com/immunogenomics/symphony
- name: Harmony
  role: Integrates reference donors (vars_use = donor, nclust = 100, max_iter = 20,
    return_object = TRUE) prior to reference compression
- name: Seurat
  role: Anchor-based integration baseline benchmarked against Symphony on LISI and
    F1
  repo: https://github.com/satijalab/seurat
- name: scArches
  role: Autoencoder-based reference-mapping baseline benchmarked against Symphony
    on LISI and F1
  repo: https://github.com/theislab/scarches
- name: LISI
  role: Computes donor LISI on the joint embedding to quantify integration of reference
    and query
  repo: https://github.com/immunogenomics/LISI
- name: singlecellmethods
  role: Provides vargenes_vst, scaleDataWithStats, and irlba-based PCA used in reference
    preprocessing
  repo: https://github.com/immunogenomics/singlecellmethods
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:38ef68f1c1c8e772217a1f9b5024f46ad6edc7e8567f3cfcb94e0708316ac88d
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# multi-species-pancreatic-islet-annotation-analysis

## Summary

Reproduces the Symphony-based mapping of a multi-donor, multi-species (human+mouse) inDrop pancreatic islet query onto a human islet reference, benchmarking integration (donor LISI) and cell-type transfer accuracy (F1) against Seurat and scArches.

## When to use

Use when a multi-donor or cross-species scRNA-seq query (e.g. inDrop human+mouse islets) must be mapped onto an existing human pancreatic islet reference and its cell types annotated via reference-label transfer.

## When NOT to use

- The goal is discovering novel query cell types absent from the reference — Symphony requires comprehensive cell type coverage in the reference and does not replace de novo analyses of the query alone
- The query contains a novel population very similar to an existing reference population, where mapping metrics may be less informative
- Reference and query are scATAC-seq peak matrices analyzed with different peak sets, where optimal feature selection for mapping is undefined

## Inputs

- Pancreatic islet reference expression matrices from four technologies, normalized as log(CP10k+1) with donor metadata
- 10,455-cell inDrop human+mouse query expression matrix with donor and species metadata
- Reference cell-type annotations for knnPredict label transfer
- True query cell-type labels for F1 evaluation

## Outputs

- Symphony reference object (.rds) plus uwot UMAP model for query projection
- Query cells mapped into the reference harmonized embedding with predicted cell-type labels and per-cell confidence scores
- Donor LISI on the joint embedding and human/mouse F1 scores, comparable to Seurat and scArches benchmarks (Fig. 4, Supplementary Tables 8-10)

## How to apply

Normalize all pancreatic islet reference matrices consistently across the four technologies with log(CP10k+1), then select variable genes per donor group using vargenes_vst with topn = 1000 per group. Scale the variable-gene expression with scaleDataWithStats using stored gene means and standard deviations, run PCA via irlba SVD with d = 20 PCs, and integrate donors with Harmony (vars_use = donor, nclust = 100, max_iter = 20, return_object = TRUE) before compressing into a Symphony reference with buildReferenceFromHarmonyObj, saving the reference .rds and uwot UMAP model. Map the 10,455-cell inDrop human+mouse query with mapQuery (do_normalize matching the reference convention, do_umap = TRUE) and transfer labels with knnPredict (k = 5, confidence = TRUE). Benchmark against Seurat anchor-based integration and scArches using donor LISI on the joint embedding and F1 of predicted vs. true labels, expecting donor LISI ≈ 2.72, human F1 ≈ 0.96, and mouse F1 ≈ 0.95 with Symphony outperforming both baselines.

## Related tools

- **Symphony** (Builds the compressed reference from the harmonized object and maps the multi-species query into the reference embedding with annotation transfer) — https://github.com/immunogenomics/symphony
- **Harmony** (Integrates reference donors (vars_use = donor, nclust = 100, max_iter = 20, return_object = TRUE) prior to reference compression)
- **Seurat** (Anchor-based integration baseline benchmarked against Symphony on LISI and F1) — https://github.com/satijalab/seurat
- **scArches** (Autoencoder-based reference-mapping baseline benchmarked against Symphony on LISI and F1) — https://github.com/theislab/scarches
- **LISI** (Computes donor LISI on the joint embedding to quantify integration of reference and query) — https://github.com/immunogenomics/LISI
- **singlecellmethods** (Provides vargenes_vst, scaleDataWithStats, and irlba-based PCA used in reference preprocessing) — https://github.com/immunogenomics/singlecellmethods

## Examples

```
reference = symphony::buildReference(ref_exp, ref_metadata, var_genes, harmony_obj, vars_use = 'donor', do_umap = TRUE); query = mapQuery(query_exp, query_meta, reference, do_normalize = TRUE, do_umap = TRUE); query = knnPredict(query, reference, ref_celltype_col, k = 5, confidence = TRUE)
```

## Evaluation signals

- Donor LISI on the joint Symphony embedding approximates 2.72
- Human cell-type F1 of knnPredict labels (k = 5) approximates 0.96
- Mouse cell-type F1 approximates 0.95
- Symphony LISI and F1 exceed Seurat anchor-based and scArches baselines as reported in Fig. 4 and Supplementary Tables 8-10
- Symphony reference compression returns an object that maps the 10,455-cell query within seconds while localizing cells in the reference embedding

## Limitations

- Condition I — comprehensive cell type coverage in the reference — is required and makes Symphony less flexible than de novo Harmony integration
- Symphony does not replace de novo analyses of the query alone for novel cell type discovery; identification of novel cell types that fail to map is an unsolved direction
- Mapping metrics may be less informative when a novel population closely resembles an existing reference population

## Evidence

- [full_text] Symphony obtained higher levels of integration than Seurat and scArches, both between reference and query and among donors within the query.: [evidence span withheld — anchor status: not_found]
- [methods] knnPredict with k=5 and confidence scores: "query = knnPredict(query,       # query object"
- [methods] Variable gene selection per donor group: "var_genes = vargenes_vst(ref_exp_full, groups = as.character(ref_metadata[['donor']]), topn = 1000)"
- [methods] Harmony return_object requirement: "Run Harmony integration. It is important to set `return_object = TRUE`."
- [abstract] Symphony abstract scope: "Symphony localizes query cells within a stable low-dimensional reference embedding, facilitating reproducible downstream transfer of reference-defined annotations to the query"
- [readme] Symphony repository: "Efficient and precise single-cell reference atlas mapping with Symphony

[Kang et al. (Nat Comm,
2021)](https://www.nature.com/articles/s41467-021-25957-x)"
