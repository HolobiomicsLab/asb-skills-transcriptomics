---
name: trajectory-gene-expression-variation-analysis
description: Use when a paired snRNA-seq/snATAC-seq co-embedded object has an inferred
  pseudotime trajectory (e.g. SCARA5+ progenitors to myofibroblasts) and you need
  candidate genes as input for enhancer-based GRN inference.
when_to_use_negative:
- When the goal is unbiased variable-feature selection on static expression data (no
  pseudotime trajectory has been inferred) — standard highly-variable-gene selection,
  not trajectory variation ranking, is the right tool.
- 'When single-modal (RNA-only) data are available: scMEGA''s candidate genes must
  be linkable to at least one ATAC peak, so ATAC-seq peaks are required.'
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: scMEGA
  role: Provides AddTrajectory (ArchR-based pseudotime fitting) and SelectGenes (smoothed
    per-gene expression along pseudotime, gene–peak linking, variation ranking and
    top-10% cutoff)
  repo: github.com/CostaLab/scMEGA
- name: ArchR
  role: Underlying trajectory inference approach used by AddTrajectory to order predefined
    clusters along pseudotime
- name: Seurat
  role: Holds the co-embedded paired-object data structure and clustering/reductions
    consumed by AddTrajectory and SelectGenes
  repo: github.com/satijalab/seurat (branch seurat5)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad003
    title: 'scMEGA: single-cell multi-omic enhancer-based gene regulatory network
      inference'
  source_content_hash: sha256:e26edfb9c875e6f499b8c569f4c9767b4b5e227247dba6d1c482416aae6fa686
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ArchR
  - Seurat
  - scMEGA
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# trajectory-gene-expression-variation-analysis

## Summary

Selects trajectory-relevant candidate genes for GRN inference by computing each gene's expression variation along an inferred pseudotime trajectory and retaining the top 10% most variable genes (cutoff user-adjustable) that can be linked to at least one ATAC peak.

## When to use

Use when a paired snRNA-seq/snATAC-seq co-embedded object has an inferred pseudotime trajectory (e.g. SCARA5+ progenitors to myofibroblasts) and you need candidate genes as input for enhancer-based GRN inference.

## When NOT to use

- When the goal is unbiased variable-feature selection on static expression data (no pseudotime trajectory has been inferred) — standard highly-variable-gene selection, not trajectory variation ranking, is the right tool.
- When single-modal (RNA-only) data are available: scMEGA's candidate genes must be linkable to at least one ATAC peak, so ATAC-seq peaks are required.

## Inputs

- Co-embedded, sub-clustered Seurat object (.rds) of paired snRNA-seq/snATAC-seq cells (e.g. coembed.sub.rds)
- Predefined cluster ordering defining the trajectory (e.g. trajectory = c(0, 2, 1))
- A dimensionality reduction (e.g. harmony/UMAP or diffusion-map embedding) used by AddTrajectory
- Peak-to-gene linkage parameters (var.cutoff.gene, cor.cutoff, fdr.cutoff)

## Outputs

- Table file of the top-10% most variable (trajectory-relevant) genes along pseudotime
- Gene set in which every retained gene is linkable to at least one ATAC peak

## How to apply

Load the co-embedded, sub-clustered Seurat object (e.g. coembed.sub.rds) containing paired snRNA-seq/snATAC-seq cells of the lineage of interest. Fit the supervised pseudotime trajectory with scMEGA's AddTrajectory (ArchR-based approach) by ordering predefined clusters along the diffusion-map/UMAP reduction — in the myofibroblast case study, clusters ordered c(0, 2, 1) from the SCARA5+ progenitor state toward terminal COL1A1/POSTN/FN1-high myofibroblasts — keeping only cells assigned to the trajectory. Compute smoothed per-gene expression values along pseudotime and link genes to peaks with SelectGenes (e.g. labelTop1 = 0, labelTop2 = 0, with var.cutoff.gene, cor.cutoff, and fdr.cutoff parameters), so only genes varying along pseudotime and associated with at least one peak are retained. Rank the retained genes by their variation along pseudotime and write the top-10% most variable gene set to a table file; this user-adjustable cutoff defines the trajectory-relevant candidate genes fed into downstream GRN inference.

## Related tools

- **scMEGA** (Provides AddTrajectory (ArchR-based pseudotime fitting) and SelectGenes (smoothed per-gene expression along pseudotime, gene–peak linking, variation ranking and top-10% cutoff)) — github.com/CostaLab/scMEGA
- **ArchR** (Underlying trajectory inference approach used by AddTrajectory to order predefined clusters along pseudotime)
- **Seurat** (Holds the co-embedded paired-object data structure and clustering/reductions consumed by AddTrajectory and SelectGenes) — github.com/satijalab/seurat (branch seurat5)

## Examples

```
obj.pair <- AddTrajectory(object = obj.pair, trajectory = c(0, 2, 1), reduction = "harmony"); genes <- SelectGenes(obj.pair, labelTop1 = 0, labelTop2 = 0)
```

## Evaluation signals

- The output gene table is non-empty.
- Every retained gene can be associated with at least one ATAC peak (the tutorial states a gene is only a potential target when linkable to at least one peak).
- The number of retained genes equals the expected fraction set by the variation cutoff (default top 10% of genes ranked by variation along pseudotime).
- Retained genes show smooth, monotonic or graded expression change when plotted along the inferred pseudotime.
- All input cells assigned to the trajectory correspond to the ordered clusters (e.g. SCARA5+ progenitor to terminal myofibroblast states).

## Limitations

- The top-10% cutoff is user-adjustable, so the candidate gene set size (and downstream GRN) depends on this arbitrary choice.
- Genes that vary along pseudotime but cannot be linked to any peak are discarded, potentially removing genuine trajectory-relevant regulators.
- Package versions used in the tutorials are not specified (the tutorial only provides sessionInfo()).

## Evidence

- [methods] Pseudo-time trajectory inference: "we infer a pseudo-time trajectory from SCARA5+ fibroblasts to myofibroblast using the approach from [ArchR]"
- [methods] AddTrajectory call: "obj.pair <- AddTrajectory(object = obj.pair, trajectory = c(2, 1, 0, 3),"
- [abstract] GRN pipeline scope: "enables an end-to-end analysis of multi-omics data for gene regulatory network inference including modalities integration, trajectory analysis, enhancer-to-promoter association, network analysis and"
- [methods] Terminal myofibroblast markers: "some cells highly express COL1A1, POSTN, and FN1 and they could be terminally differentiated myofibroblasts"
- [readme] Tutorials listing: "We provided the following tutorials to show how to use scMEGA to build GRN by using single-cell multiomics/multimodal data"
