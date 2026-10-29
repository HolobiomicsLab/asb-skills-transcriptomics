---
name: cross-method-benchmarking-of-reference-mapping-algorithms
description: Use when you must compare reference-mapping methods (Symphony vs Seurat
  vs scArches) on the same query/reference pair, quantifying integration via LISI
  and annotation transfer via F1 against true labels.
when_to_use_negative:
- The query may contain novel cell types absent from the reference — the article states
  Symphony does not replace de novo analyses for novel cell type discovery and its
  mapping metrics are less informative when a novel population is very similar to
  an existing reference population
- Reference and query come from scATAC-seq with different peak sets — no optimal feature
  selection approach is defined since peaks are not standardized across analyses
- The goal is de novo integration of all datasets together rather than mapping a query
  onto a fixed, pre-built reference
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Symphony
  role: 'Primary method under evaluation: builds the compressed reference and maps
    the query via mapQuery'
  repo: github.com/immunogenomics/symphony
- name: Harmony
  role: Integrates reference donors (vars_use = donor, nclust = 100, max_iter = 20)
    before compression into the Symphony reference
  repo: github.com/immunogenomics/symphony
- name: Seurat
  role: Benchmark comparator using Seurat anchor-based integration on the same query-reference
    pair
  repo: github.com/satijalab/seurat
- name: scArches
  role: Benchmark comparator using an autoencoder-based framework (scANVI/trVAE) to
    map the query
  repo: github.com/theislab/scarches
- name: LISI
  role: Computes Local Inverse Simpson's Index to quantify donor mixing/integration
    in the joint embedding
  repo: github.com/immunogenomics/LISI
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-25957-x
    title: Efficient and precise single-cell reference atlas mapping with Symphony
  source_content_hash: sha256:333376767a647cd355079f1c3b5e41a142c026af52dbd360b97f2e95975c28d3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Harmony
  - LISI
  - Seurat
  - Symphony
  - scArches
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-method-benchmarking-of-reference-mapping-algorithms

## Summary

A benchmarking protocol that maps a multi-donor, multi-species query (10,455-cell inDrop human+mouse pancreas cells) onto a compressed human pancreatic islet reference with Symphony, then benchmarks integration and annotation-transfer accuracy against Seurat anchor-based integration and scArches.

## When to use

Use when you must compare reference-mapping methods (Symphony vs Seurat vs scArches) on the same query/reference pair, quantifying integration via LISI and annotation transfer via F1 against true labels.

## When NOT to use

- The query may contain novel cell types absent from the reference — the article states Symphony does not replace de novo analyses for novel cell type discovery and its mapping metrics are less informative when a novel population is very similar to an existing reference population
- Reference and query come from scATAC-seq with different peak sets — no optimal feature selection approach is defined since peaks are not standardized across analyses
- The goal is de novo integration of all datasets together rather than mapping a query onto a fixed, pre-built reference

## Inputs

- Pancreatic islet reference expression matrices from four technologies with donor metadata
- 10,455-cell inDrop query containing human and mouse pancreas cells with true cell-type labels
- Reference cell-type annotations for knnPredict training

## Outputs

- Symphony reference .rds object and uwot UMAP model
- Query mapped into the harmonized reference embedding with predicted cell-type labels and per-cell confidence scores
- Per-method comparison table of donor LISI and F1 (human and mouse) across Symphony, Seurat, and scArches

## How to apply

Build a shared reference first: normalize pancreatic islet expression across all four technologies as log(CP10k+1), select variable genes per donor group with vargenes_vst (topn = 1000 per group), scale with scaleDataWithStats using stored gene means/SDs, run PCA via irlba SVD (d = 20 PCs), integrate donors with Harmony (vars_use = donor, nclust = 100, max_iter = 20, return_object = TRUE), and compress with buildReferenceFromHarmonyObj, saving the reference .rds and uwot UMAP model. Map the 10,455-cell inDrop human+mouse query with mapQuery (do_normalize matching the reference convention, do_umap = TRUE), then transfer labels with knnPredict (k = 5, confidence = TRUE). Benchmark the identical query against Seurat anchor-based integration and scArches (autoencoder-based). Compute donor LISI on the joint embedding and F1 of predicted vs true labels separately for human and mouse cells, then compare all three methods.

## Related tools

- **Symphony** (Primary method under evaluation: builds the compressed reference and maps the query via mapQuery) — github.com/immunogenomics/symphony
- **Harmony** (Integrates reference donors (vars_use = donor, nclust = 100, max_iter = 20) before compression into the Symphony reference) — github.com/immunogenomics/symphony
- **Seurat** (Benchmark comparator using Seurat anchor-based integration on the same query-reference pair) — github.com/satijalab/seurat
- **scArches** (Benchmark comparator using an autoencoder-based framework (scANVI/trVAE) to map the query) — github.com/theislab/scarches
- **LISI** (Computes Local Inverse Simpson's Index to quantify donor mixing/integration in the joint embedding) — github.com/immunogenomics/LISI

## Examples

```
vargenes = vargenes_vst(ref_exp_full, groups = as.character(ref_metadata[['donor']]), topn = 1000)
```

## Evaluation signals

- Donor LISI on the joint embedding for the Symphony mapping should approximate 2.72
- Cell-type F1 for human query cells should be approximately 0.96 (predicted vs true labels)
- Cell-type F1 for mouse query cells should be approximately 0.95
- Symphony should obtain higher integration than Seurat and scArches, both between reference and query and among donors within the query, as reported in Fig. 4 and Supplementary Tables 8-10
- knnPredict with k = 5 and confidence = TRUE must return both predicted labels and per-cell confidence scores for every query cell

## Limitations

- Identification of novel cell types that failed to map is an unsolved future direction for mapping algorithms
- Symphony mapping metrics may be less informative when a novel population is very similar to an existing reference population
- Symphony does not replace de novo analyses of the query alone for users interested in novel cell type discovery
- Condition I (comprehensive cell type coverage in the reference) is not flexible and limits equivalence with de novo Harmony integration

## Evidence

- [results] On the pancreatic islet mapping experiment, Symphony obtained higher levels of integration than Seurat and scArches, both between reference and query and among donors within the query.: [evidence span withheld — anchor status: not_found]
- [methods] vargenes_vst with topn = 1000 per donor group: "vargenes = vargenes_vst(ref_exp_full, groups = as.character(ref_metadata[['donor']]), topn = 1000)"
- [methods] Harmony integration requires return_object = TRUE: "Run Harmony integration. It is important to set `return_object = TRUE`."
- [discussion] scArches comparator description: "the scArches28 algorithm uses an autoencoder-based framework to map to references built with autoencoder-based integration algorithms32,33."
- [intro] Seurat comparator description: "Reference mapping approaches introduced so far include Seurat30, which is compatible with Seurat anchor-based integration18"
- [readme] LISI definition: [evidence span withheld — anchor status: external_doc]
