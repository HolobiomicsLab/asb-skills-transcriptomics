---
name: umap-embedding-visualization-of-integrated-data
description: Use when you have an integrated or batch-adjusted single-cell expression
  matrix (scRNA-seq or CyTOF) and need to visually verify that datasets are mixed
  within shared cell types and that cell-type-specific populations are preserved.
when_to_use_negative:
- The expression data are raw/unintegrated — the UMAP is used here specifically to
  evaluate post-integration dataset mixing, though a raw-data UMAP can serve as the
  pre-integration comparison.
- Only an embedding output is available and you need interpretable downstream features
  such as cell–cell interactions or pathway information — the article notes a low-dimensional
  representation 'does not naturally lend itself to the development of interpretable
  features'.
- The goal is quantitative batch-removal benchmarking against other methods rather
  than qualitative visual inspection of one integration.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: scMerge2
  role: Produces the adjusted expression matrix whose single-cell-level factors are
    the input to the UMAP embedding
  repo: https://github.com/SydneyBioX/scMerge
- name: Seurat
  role: Computes the joint UMAP embedding on the scMerge2-adjusted expression matrix
- name: CyTOF
  role: Data modality of the COMBAT/Geanon inputs; marker densities are overlaid as
    complementary validation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:7d10e2bea3e75f9c7c309a1f3550d78669e861fe699d5dd86523635c3a00b879
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - CyTOF
  - Seurat
  - scMerge2
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# umap-embedding-visualization-of-integrated-data

## Summary

Compute a joint UMAP embedding on scMerge2-adjusted single-cell expression matrices (e.g. the 11M+ cell COMBAT and Geanon CyTOF datasets) and render it colored by dataset and unified cell type to visually confirm batch/dataset-effect removal while retaining biologically distinct populations.

## When to use

Use when you have an integrated or batch-adjusted single-cell expression matrix (scRNA-seq or CyTOF) and need to visually verify that datasets are mixed within shared cell types and that cell-type-specific populations are preserved.

## When NOT to use

- The expression data are raw/unintegrated — the UMAP is used here specifically to evaluate post-integration dataset mixing, though a raw-data UMAP can serve as the pre-integration comparison.
- Only an embedding output is available and you need interpretable downstream features such as cell–cell interactions or pathway information — the article notes a low-dimensional representation 'does not naturally lend itself to the development of interpretable features'.
- The goal is quantitative batch-removal benchmarking against other methods rather than qualitative visual inspection of one integration.

## Inputs

- scMerge2-adjusted single-cell expression matrix (post propagation of adjustment factors to cell level)
- Cell-level metadata: source dataset label, condition (healthy control / COVID-19 / sepsis), sample ID, and harmonized cell-type annotations (unified to 18 cell types)
- Marker/panel gene list for per-marker density overlays (CyTOF markers)

## Outputs

- Joint UMAP embedding colored by dataset (e.g. COMBAT vs Geanon)
- Joint UMAP embedding colored by unified cell type (18 types)
- Per-marker expression density distribution plots comparing datasets post-integration
- Composite validation figure (qualitative match to FigS7_v1.png)

## How to apply

After propagating scMerge2 dataset-effect adjustment factors back to the single-cell level, compute a joint UMAP embedding (e.g. via Seurat) on the adjusted expression matrix. Render the UMAP twice: colored by source dataset (e.g. COMBAT vs Geanon) to check mixing, and colored by unified cell type (18 harmonized types) to check that cell-type structure survives integration. Complement the UMAP by overlaying per-marker expression density distributions across datasets to confirm that marker distributions now overlap. Validation is qualitative: datasets should be mixed within shared cell types, and a biologically expected distinct population — here the Granulocyte (Neutrophil/Eosinophil) cluster unique to the Geanon dataset — must remain clearly separable rather than being falsely merged away.

## Related tools

- **scMerge2** (Produces the adjusted expression matrix whose single-cell-level factors are the input to the UMAP embedding) — https://github.com/SydneyBioX/scMerge
- **Seurat** (Computes the joint UMAP embedding on the scMerge2-adjusted expression matrix)
- **CyTOF** (Data modality of the COMBAT/Geanon inputs; marker densities are overlaid as complementary validation)

## Examples

```
library(Seurat); integrated <- CreateSeuratObject(scMerge2_adjusted); integrated <- RunUMAP(integrated, dims = 1:30); DimPlot(integrated, group.by = c("dataset", "cell_type"))
```

## Evaluation signals

- Datasets (COMBAT and Geanon) appear mixed within shared cell types in the post-integration UMAP, unlike in the raw-data UMAP
- A clearly separable Granulocyte (Neutrophil/Eosinophil) cluster unique to Geanon is retained in the integrated UMAP
- Per-marker expression density curves overlap across datasets after integration
- All cells are colored/assigned to one of the 18 harmonized unified cell types
- Composite figure qualitatively matches FigS7_v1.png

## Limitations

- Validation is qualitative (visual comparison against a reference figure), not a quantitative mixing metric
- A distinct dataset-unique population (Geanon Granulocytes) can be over-merged or falsely interpreted as residual batch effect; it must be checked against biology
- Embedding-only outputs limit downstream interpretability (e.g. cell–cell interactions, pathway features) per the article's discussion

## Evidence

- [other] UMAP colored by dataset and cell type: [evidence span withheld — anchor status: not_found]
- [abstract] scMerge2 removes dataset variability in CyTOF: "we demonstrate that scMerge2 can remove dataset variability in CyTOF, imaging mass cytometry and CITE-seq experiments, demonstrating its applicability to a broad spectrum of single-cell profiling"
- [readme] scMerge2 vignette reference: [evidence span withheld — anchor status: external_doc]
- [discussion] embedding interpretability limitation: "an embedding output by itself increases interpretability challenges since a low-dimensional representation does not naturally lend itself to the development of interpretable features"
