---
name: single-cell-differential-abundance-analysis
description: Use when two annotated single-cell RNA-seq datasets processed in SingCellaR
  (e.g., eFL and FL HSPC objects with existing UMAP embeddings and cell-type labels)
  must be compared for cell-subpopulation abundance shifts, and per-cell DA scores
  plus a DA UMAP figure are the desired outputs.
when_to_use_negative:
- The datasets have not been clustered, annotated, and embedded (no UMAP) in SingCellaR
  — run the upstream clustering/annotation workflow first.
- The goal is differential gene expression rather than differential cell-subpopulation
  abundance — use identifyDifferentialGenes directly instead.
- More than two conditions must be compared simultaneously — this skill covers a pairwise
  two-object comparison (e.g., eFL vs FL).
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SingCellaR
  role: R package providing run_DAseq_comparison for pairwise differential abundance
    testing, DA score projection onto UMAP, and plot_cells_annotation-style labeling
  repo: https://github.com/supatt-lab/SingCellaR
- name: R
  role: Runtime environment in which the SingCellaR DAseq workflow is executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:1482d92bb8aeabd555e6e40196c347cd7554b0f893ef9a763e4e565dd9b45144
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/supatt-lab/SingCellaR
  tools:
  - R
  - SingCellaR
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# single-cell-differential-abundance-analysis

## Summary

Performs pairwise differential abundance (DAseq) testing between two processed SingCellaR objects (e.g., early fetal liver vs fetal liver HSPCs), computes DA log2FC scores per subpopulation, projects them onto the UMAP embedding, and renders a labeled DA UMAP figure (eFL_vs_FL.pdf).

## When to use

Use when two annotated single-cell RNA-seq datasets processed in SingCellaR (e.g., eFL and FL HSPC objects with existing UMAP embeddings and cell-type labels) must be compared for cell-subpopulation abundance shifts, and per-cell DA scores plus a DA UMAP figure are the desired outputs.

## When NOT to use

- The datasets have not been clustered, annotated, and embedded (no UMAP) in SingCellaR — run the upstream clustering/annotation workflow first.
- The goal is differential gene expression rather than differential cell-subpopulation abundance — use identifyDifferentialGenes directly instead.
- More than two conditions must be compared simultaneously — this skill covers a pairwise two-object comparison (e.g., eFL vs FL).

## Inputs

- Two processed SingCellaR objects (e.g., eFL and FL HSPC datasets) with clustering, cell-type annotation, and UMAP embedding
- Cell-type annotation labels for each subpopulation

## Outputs

- Per-cell and per-subpopulation DAseq log2FC abundance scores
- DA-score-projected UMAP embedding
- Differential abundance UMAP figure with cell-type labels (eFL_vs_FL.pdf)

## How to apply

Load the two processed SingCellaR objects (eFL and FL) produced by the upstream clustering and annotation workflow. Call the run_DAseq_comparison wrapper on the object pair to run the pairwise differential abundance (DAseq) analysis, computing DA log-fold-change scores for cell subpopulations using min.log2FC and min.expFraction filtering conventions consistent with SingCellaR's identifyDifferentialGenes family. Project the per-cell DA scores onto the existing UMAP embedding of the integrated dataset. Draw the differential abundance UMAP with cell-type annotation labeling (plot_cells_annotation-style), so positive scores mark populations enriched in one condition — e.g., erythroid progenitors positive in FL and megakaryocytic progenitors positive in eFL. Save the figure as eFL_vs_FL.pdf and verify it renders and qualitatively matches Figure 7B.

## Related tools

- **SingCellaR** (R package providing run_DAseq_comparison for pairwise differential abundance testing, DA score projection onto UMAP, and plot_cells_annotation-style labeling) — https://github.com/supatt-lab/SingCellaR
- **R** (Runtime environment in which the SingCellaR DAseq workflow is executed)

## Examples

```
run_DAseq_comparison(objectA = eFL_HSPC, objectB = FL_HSPC)
```

## Evaluation signals

- eFL_vs_FL.pdf exists and renders the differential abundance UMAP.
- Erythroid progenitors score positive (enriched) in FL and megakaryocytic progenitors score positive (enriched) in eFL, qualitatively matching Figure 7B.
- DA log2FC scores are computed with min.log2FC and min.expFraction filters applied consistently with the identifyDifferentialGenes conventions.
- Per-cell DA scores successfully map onto the pre-existing UMAP embedding of the integrated dataset.
- Lymphoid and myeloid progenitors predominantly trace back to BM samples in the comparison output.

## Limitations

- SingCellaR lacks 'automatic object transformation' to interact with other existing packages, such as Seurat, so objects must stay in native SingCellaR format for the comparison.
- Plots may differ in rotation between runs/users due to different software versions and seed number settings.
- The comparison is pairwise; the finding applies to the eFL-vs-FL HSPC setting demonstrated in the protocol.

## Evidence

- [other] Call the run_DAseq_comparison wrapper on the object pair to perform the pairwise differential abundance (DAseq) analysis, computing DA log-fold-change scores: [evidence span withheld — anchor status: not_found]
- [methods] identifyDifferentialGenes parameters: "`identifyDifferentialGenes(method, objectA, objectB, cellsA, cellsB, write.to.file, min.log2FC, min.expFraction)`"
- [discussion] SingCellaR interoperability limitation: "SingCellaR still lacks 'automatic object transformation' to interact with other existing packages, such as Seurat."
- [discussion] UMAP rotation variability: "This is caused by different software versions for generating plots and the seed number setting."
- [readme] README tutorial pointer: "The tutorial of how to use the software can be found at https://supatt-lab.github.io/SingCellaR.Doc/"
