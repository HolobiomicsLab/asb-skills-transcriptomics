---
name: pairwise-condition-cell-population-comparison
description: Use when two processed, cell-type-annotated single-cell objects from
  distinct conditions are available and the goal is to identify which cell populations
  are enriched or depleted in one condition versus the other, e.
when_to_use_negative:
- Only a single condition/object is available with no second sample set to compare
  against — DAseq pairwise comparison requires two object pairs
- The input objects lack cell-type annotations or a UMAP embedding; run the upstream
  clustering and annotation workflow first
- The goal is differential gene expression (DE) rather than differential abundance
  of populations — use identifyDifferentialGenes directly instead
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SingCellaR
  role: R package providing the processed objects, the identifyDifferentialGenes filtering
    conventions (min.log2FC, min.expFraction), UMAP embedding, and plot_cells_annotation-style
    labeling used for the pairwise DAseq comparison
  repo: github.com/supatt-lab/SingCellaR
- name: R
  role: Analysis environment in which the two SingCellaR objects are loaded, the DAseq
    comparison is run, and the DA UMAP figure is generated and saved
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:efa29bb266625eaa0056d4a4ebe66caf3918be3e45b5845ea500e6cce3b16940
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SingCellaR
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# pairwise-condition-cell-population-comparison

## Summary

A skill for assessing differential abundance (DA) of annotated cell subpopulations between two biological conditions (e.g., early fetal liver vs. fetal liver HSPCs) using DAseq-style analysis inside SingCellaR, projecting per-cell DA scores onto a UMAP, and rendering a condition-enrichment figure.

## When to use

Use when two processed, cell-type-annotated single-cell objects from distinct conditions are available and the goal is to identify which cell populations are enriched or depleted in one condition versus the other, e.g. erythroid progenitors enriched in FL versus megakaryocytic progenitors enriched in eFL.

## When NOT to use

- Only a single condition/object is available with no second sample set to compare against — DAseq pairwise comparison requires two object pairs
- The input objects lack cell-type annotations or a UMAP embedding; run the upstream clustering and annotation workflow first
- The goal is differential gene expression (DE) rather than differential abundance of populations — use identifyDifferentialGenes directly instead

## Inputs

- Two processed SingCellaR R objects from distinct conditions (e.g., eFL and FL HSPC objects) with clustering and cell-type annotations completed
- Precomputed UMAP embedding of the integrated dataset
- Cell-type annotation labels for subpopulations (e.g., erythroid, megakaryocytic, lymphoid, myeloid progenitors)

## Outputs

- Per-cell DAseq differential abundance scores mapped onto the UMAP
- Condition-wise enrichment assignments for each cell subpopulation (DA log-fold-change scores)
- Differential abundance UMAP figure with cell-type annotation labels, e.g. eFL_vs_FL.pdf

## How to apply

Load the two processed SingCellaR objects (e.g., eFL and FL HSPC objects produced by the upstream clustering and annotation workflow) in R. Run the pairwise DAseq comparison via the run_DAseq_comparison wrapper on the object pair, which computes DA log-fold-change scores per subpopulation using the same min.log2FC and min.expFraction filtering conventions used in SingCellaR's identifyDifferentialGenes family. Project the resulting per-cell DA scores onto the existing UMAP embedding of the integrated dataset. Draw a differential-abundance UMAP that labels cells by DA score and cell-type annotation (plot_cells_annotation-style labeling) and save the figure (e.g., eFL_vs_FL.pdf). Interpret positive scores condition-wise: in the eFL vs FL comparison, erythroid progenitors score positive in FL, megakaryocytic progenitors score positive in eFL, and lymphoid/myeloid progenitors are predominantly from BM samples.

## Related tools

- **SingCellaR** (R package providing the processed objects, the identifyDifferentialGenes filtering conventions (min.log2FC, min.expFraction), UMAP embedding, and plot_cells_annotation-style labeling used for the pairwise DAseq comparison) — github.com/supatt-lab/SingCellaR
- **R** (Analysis environment in which the two SingCellaR objects are loaded, the DAseq comparison is run, and the DA UMAP figure is generated and saved)

## Examples

```
run_DAseq_comparison(objectA = eFL_HSPC, objectB = FL_HSPC, min.log2FC = 0.25, min.expFraction = 0.1); plot_cells_annotation(Human_HSPC, annotation.label = "DA_score"); ggsave("eFL_vs_FL.pdf")
```

## Evaluation signals

- The output figure (e.g., eFL_vs_FL.pdf) exists and renders the DA UMAP
- Qualitative match to the reference figure: erythroid populations score positive in FL while megakaryocytic populations score positive in eFL
- DA scores are successfully projected onto the existing UMAP embedding (cells colored by DA score across the integrated dataset)
- Subpopulation DA log-fold-change scores respect the min.log2FC and min.expFraction filtering conventions, so only sufficiently abundant/changed populations receive significant DA calls
- Cell-type labels are drawn on the DA UMAP (plot_cells_annotation-style) enabling direct visual cross-check of DA scores against annotations

## Limitations

- SingCellaR lacks 'automatic object transformation' to interact with other existing packages, such as Seurat, so the DAseq comparison must stay within SingCellaR objects
- UMAP plot rotations can differ from published figures due to software version and seed number differences
- As a qualitative visualization, the DA UMAP result is a visual/qualitative match rather than a strict numeric reproduction

## Evidence

- [methods] identifyDifferentialGenes with min.log2FC and min.expFraction parameters as the DAseq filtering convention: "`identifyDifferentialGenes(method, objectA, objectB, cellsA, cellsB, write.to.file, min.log2FC, min.expFraction)`"
- [intro] SingCellaR as the integrative analysis tool for single-cell RNA-seq data: "SingCellaR an integrative analysis tool for analysing single-cell RNA sequencing data"
- [discussion] Lack of interoperability with other R packages as a limitation: "SingCellaR still lacks 'automatic object transformation' to interact with other existing packages, such as Seurat."
- [discussion] UMAP rotation differences due to software versions and seed settings: "This is caused by different software versions for generating plots and the seed number setting."
- [readme] SingCellaR tutorial and example datasets hosted for download: "The tutorial of how to use the software can be found at https://supatt-lab.github.io/SingCellaR.Doc/"
