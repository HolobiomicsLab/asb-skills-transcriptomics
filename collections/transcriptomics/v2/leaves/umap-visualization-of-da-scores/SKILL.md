---
name: umap-visualization-of-da-scores
description: Use when two processed single-cell datasets (e.g. early fetal liver eFL
  and fetal liver FL HSPC objects) have been integrated and annotated, and you need
  to identify and visualize which cell subpopulations are differentially abundant
  between them via DAseq scores on UMAP.
when_to_use_negative:
- When only a single dataset is available — DAseq comparison requires a pairwise objectA/objectB
  comparison of cell sets
- When the SingCellaR objects lack an integrated UMAP embedding or cell-type annotations,
  since DA scores must be projected onto the existing UMAP and interpreted per annotated
  subpopulation
- When the goal is differential gene expression rather than differential abundance
  of cell populations — use identifyDifferentialGenes directly instead
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SingCellaR
  role: R analysis package providing run_DAseq_comparison, UMAP embedding, and plot_cells_annotation-style
    labeling for the DA visualization
  repo: https://github.com/supatt-lab/SingCellaR
- name: R
  role: Runtime environment in which the SingCellaR DAseq comparison and plotting
    are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:ef9596868d48b7c3791fc2d85fb76de0eeb45434deb415664d34e732329fc952
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/supatt-lab/SingCellaR
  tools:
  - R
  - SingCellaR
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# umap-visualization-of-da-scores

## Summary

Projects per-cell differential abundance (DAseq) log-fold-change scores onto a pre-computed UMAP embedding of integrated SingCellaR single-cell RNA-seq objects, and draws the DA UMAP with cell-type annotation labels so enriched subpopulations (e.g. erythroid in FL vs megakaryocytic in eFL HSPCs) are visually resolved.

## When to use

Use when two processed single-cell datasets (e.g. early fetal liver eFL and fetal liver FL HSPC objects) have been integrated and annotated, and you need to identify and visualize which cell subpopulations are differentially abundant between them via DAseq scores on UMAP.

## When NOT to use

- When only a single dataset is available — DAseq comparison requires a pairwise objectA/objectB comparison of cell sets
- When the SingCellaR objects lack an integrated UMAP embedding or cell-type annotations, since DA scores must be projected onto the existing UMAP and interpreted per annotated subpopulation
- When the goal is differential gene expression rather than differential abundance of cell populations — use identifyDifferentialGenes directly instead

## Inputs

- Two processed SingCellaR R objects with UMAP embeddings and cell-type annotations (e.g. eFL and FL HSPC datasets)
- Cell-type annotation labels (identifyDifferentialGenes-style annotated subpopulations)
- DAseq comparison parameters: min.log2FC and min.expFraction cutoffs

## Outputs

- Per-cell DA log-fold-change scores mapped onto the UMAP embedding
- Differential abundance UMAP figure labeled by DA score and cell-type annotation (e.g. eFL_vs_FL.pdf)

## How to apply

Load the two processed SingCellaR objects from the upstream clustering and annotation workflow, then call the run_DAseq_comparison wrapper on the object pair to compute DA log-fold-change scores per subpopulation, filtering genes with the min.log2FC and min.expFraction conventions of the identifyDifferentialGenes family. Project the resulting per-cell DA scores onto the existing UMAP embedding of the integrated dataset rather than recomputing an embedding, so scores map onto the same coordinate system used for cluster annotation. Draw the differential abundance UMAP labeling cells by DA score and cell-type annotation (plot_cells_annotation-style labeling). Rationale: DAseq comparison is a pairwise test between objectA and objectB cell sets, and overlaying scores on the integrated UMAP makes enrichment interpretable per annotated lineage (erythroid positive in FL, megakaryocytic positive in eFL, lymphoid/myeloid progenitors predominantly from BM samples).

## Related tools

- **SingCellaR** (R analysis package providing run_DAseq_comparison, UMAP embedding, and plot_cells_annotation-style labeling for the DA visualization) — https://github.com/supatt-lab/SingCellaR
- **R** (Runtime environment in which the SingCellaR DAseq comparison and plotting are executed)

## Examples

```
identifyDifferentialGenes(method, objectA, objectB, cellsA, cellsB, write.to.file, min.log2FC, min.expFraction)
```

## Evaluation signals

- The output figure (e.g. eFL_vs_FL.pdf) exists and renders the DA UMAP
- Qualitative match to Figure 7B: erythroid cells score positive in FL while megakaryocytic cells score positive in eFL
- DA scores are mapped onto the existing integrated UMAP coordinates, not a newly computed embedding
- Per-subpopulation DA log-fold-changes pass the min.log2FC and min.expFraction filtering conventions used in identifyDifferentialGenes
- Lymphoid and myeloid progenitors predominantly associate with BM samples rather than eFL or FL, consistent with the reported comparison

## Limitations

- SingCellaR lacks 'automatic object transformation' to interact with other existing packages, such as Seurat, limiting reuse of embeddings or plots outside SingCellaR
- UMAP/FDG plot rotations can differ from the protocol due to different software versions and seed number settings
- The skill is demonstrated on human hematopoietic stem and progenitor cells; adaptations to other systems require curated signature gene sets for annotation

## Evidence

- [methods] identifyDifferentialGenes(method, objectA, objectB, cellsA, cellsB, write.to.file, min.log2FC, min.expFraction): "identifyDifferentialGenes(method, objectA, objectB, cellsA, cellsB, write.to.file, min.log2FC, min.expFraction)"
- [discussion] SingCellaR lacks interoperability with other packages: "SingCellaR still lacks 'automatic object transformation' to interact with other existing packages, such as Seurat."
- [other] DAseq workflow on eFL vs FL objects: [evidence span withheld — anchor status: not_found]
- [readme] SingCellaR is an integrative analysis tool for single-cell RNA-seq data: "SingCellaR an integrative analysis tool for analysing single-cell RNA sequencing data"
