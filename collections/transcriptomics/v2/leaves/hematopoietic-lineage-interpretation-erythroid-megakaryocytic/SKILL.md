---
name: hematopoietic-lineage-interpretation-erythroid-megakaryocytic
description: Use when two processed SingCellaR single-cell HSPC objects (e.g., eFL
  and FL) must be compared for differential abundance of annotated subpopulations
  such as erythroid, megakaryocytic, lymphoid, and myeloid progenitors, with results
  visualized on a shared UMAP.
when_to_use_negative:
- Input data are raw gene expression matrices that have not yet passed SingCellaR
  quality control (filter_cells_and_genes), normalization (normalize_UMIs), clustering,
  and cell-type annotation
- Only one single-cell dataset is available — pairwise DAseq comparison requires two
  objects (objectA and objectB)
- The goal is gene-level rather than cell-population-level differential analysis —
  use identifyDifferentialGenes directly instead
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SingCellaR
  role: R package providing run_DAseq_comparison, identifyDifferentialGenes, UMAP
    projection, and plot_cells_annotation-style labeling used for the pairwise differential
    abundance analysis of HSPC subpopulations
  repo: https://github.com/supatt-lab/SingCellaR
- name: R
  role: Execution environment for the SingCellaR-based DAseq workflow and figure generation
    (eFL_vs_FL.pdf)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:0edc8c8a8ed86db621a5c4e45aa78ffd5dedafdd3ca1317570b89323f17bb4ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/supatt-lab/SingCellaR
  tools:
  - R
  - SingCellaR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# hematopoietic-lineage-interpretation-erythroid-megakaryocytic

## Summary

A skill for interpreting differential cell-population abundance between hematopoietic stem and progenitor cell (HSPC) datasets (e.g., early fetal liver vs fetal liver) using SingCellaR's pairwise DAseq comparison, yielding per-cell DA log-fold-change scores projected onto UMAP.

## When to use

Use when two processed SingCellaR single-cell HSPC objects (e.g., eFL and FL) must be compared for differential abundance of annotated subpopulations such as erythroid, megakaryocytic, lymphoid, and myeloid progenitors, with results visualized on a shared UMAP.

## When NOT to use

- Input data are raw gene expression matrices that have not yet passed SingCellaR quality control (filter_cells_and_genes), normalization (normalize_UMIs), clustering, and cell-type annotation
- Only one single-cell dataset is available — pairwise DAseq comparison requires two objects (objectA and objectB)
- The goal is gene-level rather than cell-population-level differential analysis — use identifyDifferentialGenes directly instead

## Inputs

- Two processed SingCellaR objects (e.g., eFL and FL HSPC datasets) with UMAP embeddings and cell-type annotations from upstream clustering/annotation
- Cell-type annotation labels for subpopulations (erythroid, megakaryocytic, lymphoid, myeloid progenitors)
- DAseq parameters (min.log2FC, min.expFraction) consistent with SingCellaR's identifyDifferentialGenes conventions

## Outputs

- Per-cell DAseq differential abundance scores (DA log-fold-change) for each subpopulation
- Differential abundance UMAP figure labeled by DA score and cell-type annotation, saved as eFL_vs_FL.pdf
- Lineage-enrichment interpretation (e.g., erythroid enriched in FL, megakaryocytic enriched in eFL)

## How to apply

Load the two processed SingCellaR objects (eFL and FL) produced by the upstream clustering and annotation workflow in R. Call the run_DAseq_comparison wrapper on the object pair to perform pairwise differential abundance (DAseq) analysis, computing DA log-fold-change scores for cell subpopulations using the min.log2FC and min.expFraction filtering conventions of the identifyDifferentialGenes family in SingCellaR. Project the per-cell DA scores onto the existing UMAP embedding of the integrated dataset. Draw the differential abundance UMAP labeling cells by DA score and cell-type annotation (plot_cells_annotation-style labeling), so erythroid populations score positive in FL and megakaryocytic populations score positive in eFL. Save the figure as eFL_vs_FL.pdf and interpret lineage shifts: erythroid progenitors enriched in FL, megakaryocytic progenitors mainly from eFL, and lymphoid/myeloid progenitors predominantly from BM samples.

## Related tools

- **SingCellaR** (R package providing run_DAseq_comparison, identifyDifferentialGenes, UMAP projection, and plot_cells_annotation-style labeling used for the pairwise differential abundance analysis of HSPC subpopulations) — https://github.com/supatt-lab/SingCellaR
- **R** (Execution environment for the SingCellaR-based DAseq workflow and figure generation (eFL_vs_FL.pdf))

## Examples

```
run_DAseq_comparison(eFL_object, FL_object, min.log2FC = 0.25, min.expFraction = 0.1)
```

## Evaluation signals

- Output file eFL_vs_FL.pdf exists and renders the DA UMAP
- Erythroid progenitors score positive (enriched) in the FL condition on the DA UMAP
- Megakaryocytic progenitors score positive (enriched) in the eFL condition on the DA UMAP
- Lymphoid and myeloid progenitors map predominantly to BM samples rather than eFL or FL
- DA score computation used min.log2FC and min.expFraction filtering consistent with SingCellaR's identifyDifferentialGenes family

## Limitations

- SingCellaR still lacks 'automatic object transformation' to interact with other existing packages, such as Seurat, limiting portability of the workflow
- Plot rotations and scores can differ between runs due to different software versions and seed number settings
- The interpretation is qualitative: validation relies on the output figure qualitatively matching the expected lineage-enrichment pattern rather than a numeric threshold

## Evidence

- [methods] identifyDifferentialGenes(method, objectA, objectB, cellsA, cellsB, write.to.file, min.log2FC, min.expFraction): "`identifyDifferentialGenes(method, objectA, objectB, cellsA, cellsB, write.to.file, min.log2FC, min.expFraction)`"
- [readme] SingCellaR integrative analysis description: "an integrative analysis tool for analysing single-cell RNA sequencing data"
- [other] DAseq comparison finding: [evidence span withheld — anchor status: not_found]
- [discussion] SingCellaR interoperability limitation: "SingCellaR still lacks 'automatic object transformation' to interact with other existing packages, such as Seurat."
