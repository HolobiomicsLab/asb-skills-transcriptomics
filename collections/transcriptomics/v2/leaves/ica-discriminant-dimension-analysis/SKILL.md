---
name: ica-discriminant-dimension-analysis
description: Use when comparing two projected query conditions (e.g., knockout vs
  WT) on a shared reference atlas and you need to find which latent dimensions/gene
  programs shift, beyond changes in cell-state composition.
when_to_use_negative:
- The query has not yet been projected onto a reference atlas — run make.projection/Run.ProjecTILs
  first, since discriminant dimensions require query and control in a shared embedding
  space
- The goal is only to quantify cell-state composition differences between conditions
  — plot.statepred.composition alone suffices without ICA dimension ranking
- The input is raw unprojected counts without a reference map; ProjecTILs performs
  cell-state quantification in the reference PCA latent space, and UMAP should be
  used for visual exploration only
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: ProjecTILs
  role: Provides find.discriminant.dimensions and find.discriminant.genes to rank
    ICA dimensions where the query deviates from a control, plus Run.ProjecTILs for
    reference projection
  repo: https://github.com/carmonalab/ProjecTILs
- name: Seurat
  role: Data structure (Seurat objects) and downstream analysis; ProjecTILs is integrated
    with Seurat
- name: R
  role: Implementation language; ProjecTILs is an R package run via R help and RStudio
    workflows
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-23324-4
    title: Interpretation of T cell states from single-cell transcriptomics data using
      reference atlases
  source_content_hash: sha256:6351ab526fe379748da847053e14819eafe4c9fe4d0526c8c76a8483c507cdcc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/ProjecTILs
  tools:
  - ProjecTILs
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# ICA discriminant dimension analysis

## Summary

Uses the ProjecTILs function find.discriminant.dimensions with ICA as the reduction to rank low-dimensional components where a projected query population (e.g., Regnase-1-null Tex CD8+ TILs) deviates most from a control (WT), then inspects the top components' driving genes for altered gene programs.

## When to use

Use when comparing two projected query conditions (e.g., knockout vs WT) on a shared reference atlas and you need to find which latent dimensions/gene programs shift, beyond changes in cell-state composition.

## When NOT to use

- The query has not yet been projected onto a reference atlas — run make.projection/Run.ProjecTILs first, since discriminant dimensions require query and control in a shared embedding space
- The goal is only to quantify cell-state composition differences between conditions — plot.statepred.composition alone suffices without ICA dimension ranking
- The input is raw unprojected counts without a reference map; ProjecTILs performs cell-state quantification in the reference PCA latent space, and UMAP should be used for visual exploration only

## Inputs

- ProjecTILs-projected Seurat objects for two conditions (e.g., Regnase-1-null and WT CD8+ TIL expression matrices loaded with read.sc.query)
- Tumour-infiltrating lymphocyte reference atlas loaded with load.reference.map
- Predicted cell-state labels (e.g., Tex, Tpex) from Run.ProjecTILs

## Outputs

- Ranked discriminant-dimension table (ICA components ordered by deviation, with KS test statistics and p-values)
- Top discriminant dimension identity (e.g., ICA 25) with its driving genes (e.g., Lag3, Klrc1)
- Direction of regulation of the top component in the perturbed condition (down-regulated in Regnase-1-null Tex cells)
- Saved projected objects, composition plot, and ranked ICA table as output files

## How to apply

After projecting both conditions onto the reference atlas with Run.ProjecTILs (which embeds the query in the reference PCA/UMAP space and predicts cell-state labels via make.projection and cellstate.predict), first verify the expected cell-state frequency changes with plot.statepred.composition (here ~3-fold Tpex and ~1.8-fold Tex enrichment in Regnase-1-null). Then restrict the analysis to the matched cell state (Tex) and call find.discriminant.dimensions with ICA as the reduction to rank dimensions by deviation between conditions; in the Regnase-1-null vs WT Tex comparison the top component was ICA 25 (KS statistic = 0.612, p = 0). Follow up with find.discriminant.genes to identify the genes driving the top dimension (here checkpoint molecules Lag3 and Klrc1, down-regulated in Regnase-1-null Tex cells), which reveals the gene program altered beyond abundance shifts.

## Related tools

- **ProjecTILs** (Provides find.discriminant.dimensions and find.discriminant.genes to rank ICA dimensions where the query deviates from a control, plus Run.ProjecTILs for reference projection) — https://github.com/carmonalab/ProjecTILs
- **Seurat** (Data structure (Seurat objects) and downstream analysis; ProjecTILs is integrated with Seurat)
- **R** (Implementation language; ProjecTILs is an R package run via R help and RStudio workflows)

## Examples

```
library(ProjecTILs); disc <- find.discriminant.dimensions(projected.KO, control=projected.WT, reduction="ICA")
```

## Evaluation signals

- The top-ranked discriminant dimension is ICA 25 with KS test statistic ~0.612 and p = 0 in the Regnase-1-null vs WT Tex comparison
- Cell-state composition check passes: Tpex fold-change ~3 and Tex fold-change ~1.8 in the Regnase-1-null condition
- The top component's driving genes include checkpoint molecules Lag3 and Klrc1
- ICA 25 is down-regulated in Regnase-1-null Tex cells relative to WT
- Projected query objects are embedded in the reference PCA/UMAP space with predicted cell-state labels (Run.ProjecTILs output schema)

## Limitations

- UMAP embeddings are for visual exploration only; cell-cell distances and densities in UMAP are highly distorted, so discriminant quantification is done in the PCA/ICA latent space
- Reference atlases do not encompass the full diversity of T cell transcriptional states, so truly novel programs may be only partially captured by reference dimensions
- ProjecTILs expects raw counts in the query Seurat object (internally log-normalized), or normalized data in the 'data' layer with skip.normalize=TRUE

## Evidence

- [methods] find.discriminant.dimensions searches PCA or ICA dimensions where the query set deviates the most from a control set or from the reference map: "Searches PCA or ICA dimensions where the query set deviates the most from a control set or from the reference map"
- [methods] Run.ProjecTILs returns the query embedded in reference space with predicted labels: "It returns the query embedded in the PCA and UMAP space of the reference, and predict cell state labels for the query cells"
- [readme] README warns UMAP embeddings are for visual exploration only: [evidence span withheld — anchor status: external_doc]
- [readme] README describes load.reference.map default mouse TIL reference: [evidence span withheld — anchor status: external_doc]
- [abstract] Article abstract states ProjecTILs identifies gene programs altered between conditions: "ProjecTILs accurately predicts the effects of cell perturbations and identifies gene programs that are altered in different conditions and tissues"
