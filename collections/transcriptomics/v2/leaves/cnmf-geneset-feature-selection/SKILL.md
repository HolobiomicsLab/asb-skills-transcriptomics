---
name: cnmf-geneset-feature-selection
description: Use when building a ProjectSVR reference model or projecting query cells
  and you must define gene-set features — e.g. choosing cNMF programs (K selection
  such as K=70 for mTCA) or top-25 marker sets per cell type.
when_to_use_negative:
- The analysis goal is an integration-dependent, PCA-anchor workflow (Seurat/scArches/Symphony-style)
  rather than ProjectSVR's gene-set-score feature space
- Cell populations are discrete, well-separated types — the article routes these through
  overexpression marker testing rather than cNMF continuous-state factorization
- Raw or rank-expression data are unavailable and AUCell scoring cannot be computed
  on the query
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3474
tools:
- name: cNMF
  role: Derives gene sets for continuous cell states; invoked through ProjectSVR's
    RunCNMF wrapper (installed via reticulate)
  repo: https://github.com/dylkot/cNMF
- name: ProjectSVR
  role: Provides RunCNMF for cNMF feature selection and ComputeModuleScore(method='AUCell')
    to score the selected gene sets
  repo: https://github.com/JarningGau/ProjectSVR
- name: AUCell
  role: Computes 0-1 gene activity scores from rank expression for the selected gene
    sets; robust to normalization
- name: Seurat
  role: Holds reference/query objects and supplies marker-test statistics used to
    build discrete-type gene sets
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf586
    title: 'ProjectSVR: mapping single-cell RNA-seq data to reference atlases by supported
      vector regression'
  source_content_hash: sha256:38462faf44da58dd8aabd508c3461723f3904c2cbca63641db9c1a4a9281edda
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/jarninggau/ProjectSVR
  repo_url: https://github.com/dylkot/cNMF
  tools:
  - AUCell
  - ProjectSVR
  - Seurat
  - cNMF
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# cnmf-geneset-feature-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Defines the gene-set feature space ProjectSVR maps onto reference embeddings: cell-type programs come from overexpression marker testing (discrete types) or cNMF factorization (continuous states, e.g. K=70 in mTCA), then AUCell converts sets into 0-1 activity scores that serve as SVR inputs.

## When to use

Use when building a ProjectSVR reference model or projecting query cells and you must define gene-set features — e.g. choosing cNMF programs (K selection such as K=70 for mTCA) or top-25 marker sets per cell type.

## When NOT to use

- The analysis goal is an integration-dependent, PCA-anchor workflow (Seurat/scArches/Symphony-style) rather than ProjectSVR's gene-set-score feature space
- Cell populations are discrete, well-separated types — the article routes these through overexpression marker testing rather than cNMF continuous-state factorization
- Raw or rank-expression data are unavailable and AUCell scoring cannot be computed on the query

## Inputs

- Reference atlas count matrix as a Seurat object (e.g. mTCA.seurat.slim.qs)
- Query dataset Seurat object (e.g. query_Ythdc2-KO.seurat.slim.qs)
- cNMF usage/spectra outputs at chosen K (e.g. K70 programs for mTCA)
- Marker-test results table (p_val_adj, avg_log2FC per gene per cell type)

## Outputs

- Cell type/state-specific gene set list (per program or per annotated type)
- Cells x gene-sets AUCell score matrix (values in 0-1) for reference and query cells

## How to apply

Derive cell type-specific gene sets from overexpression testing for discrete types (filter markers by p_val_adj < 1e-6 and avg_log2FC > log2(1.5), removing ribosomal '^mt-' and mitochondrial genes) or from cNMF for continuous states, using the ProjectSVR RunCNMF wrapper of the python cnmf package installed via reticulate. Choose the cNMF rank K (e.g. K=70 for the mTCA atlas with cNMF-derived K70 gene sets). Compute the gene-set score matrix for reference and query cells with ProjectSVR::ComputeModuleScore using method='AUCell', which yields 0-1 rank-based activity scores robust to normalization. Treat gene set size as a hyperparameter: results were consistent across settings, with the top 25 genes used by default. The resulting scores matrix is the feature table handed to ProjectSVR::FitEnsembleSVM to learn the UMAP_1/2 embedding.

## Related tools

- **cNMF** (Derives gene sets for continuous cell states; invoked through ProjectSVR's RunCNMF wrapper (installed via reticulate)) — https://github.com/dylkot/cNMF
- **ProjectSVR** (Provides RunCNMF for cNMF feature selection and ComputeModuleScore(method='AUCell') to score the selected gene sets) — https://github.com/JarningGau/ProjectSVR
- **AUCell** (Computes 0-1 gene activity scores from rank expression for the selected gene sets; robust to normalization)
- **Seurat** (Holds reference/query objects and supplies marker-test statistics used to build discrete-type gene sets) — https://github.com/satijalab/seurat

## Examples

```
ProjectSVR::ComputeModuleScore(seu, gene.sets = gene.sets, method = "AUCell")
```

## Evaluation signals

- Marker sets for discrete types pass p_val_adj < 1e-6 and avg_log2FC > log2(1.5), with ribosomal and mitochondrial ('^mt-') genes removed
- AUCell scores for all gene sets fall within 0-1 and are rank-based (insensitive to count normalization)
- Gene set size sensitivity check: mapping results remain consistent across sizes, with top 25 genes as the default setting
- cNMF K selection documented (e.g. K=70 programs for mTCA) and each program yields a distinct score column
- Downstream check: ensemble SVR trained on these scores recovers held-out reference UMAP_1/2 coordinates at accuracy comparable to state-of-the-art mappers

## Limitations

- Gene set size is a hyperparameter; the paper reports consistency across settings but defaults to the top 25 genes
- Gene-set-based projections can be misleading for query cells containing novel states absent from the reference; compare unsupervised clustering to mapping results
- Performance declines when projecting across platforms (e.g. Smart-seq2 to 10x) due to differing UMI distributions and sequencing depth

## Evidence

- [discussion] Cell type-specific gene sets were derived either from overexpression testing (for discrete types) or cNMF (for continuous states): "Cell type-specific gene sets were derived either from overexpression testing (for discrete types) or cNMF (for continuous states)"
- [discussion] we used AUCell to compute gene activity scores (0–1) based on rank expression, robust to normalization and compatible with datasets lacking raw counts: "we used AUCell to compute gene activity scores (0–1) based on rank expression, robust to normalization and compatible with datasets lacking raw counts"
- [discussion] Although gene set size is a hyperparameter, results were consistent across different settings; we used the top 25 genes by default.: "Although gene set size is a hyperparameter, results were consistent across different settings; we used the top 25 genes by default."
- [methods] methods marker filter: "yy <- subset(xx, p_val_adj < 1e-6 & avg_log2FC > log2(1.5))"
- [readme] README RunCNMF wrapper: [evidence span withheld — anchor status: external_doc]
