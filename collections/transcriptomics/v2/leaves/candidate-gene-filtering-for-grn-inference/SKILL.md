---
name: candidate-gene-filtering-for-grn-inference
description: Use when building a GRN from paired snRNA-seq/snATAC-seq multiomic data
  with scMEGA and you need to restrict network inference to genes varying along a
  pseudotime trajectory that have at least one linked peak.
when_to_use_negative:
- Cells or clusters have not yet been assigned to a trajectory — variation-along-pseudotime
  filtering is undefined without AddTrajectory ordering
- Genes cannot be linked to any ATAC peak (e.g. no enhancer-to-promoter association
  step run) — such genes are not potential GRN targets in scMEGA
- Analysis targets a static, non-trajectory GRN (e.g. clustering-only differential
  expression) where pseudotime-variable selection is not the intended criterion
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scMEGA
  role: Provides AddTrajectory and SelectGenes used to fit pseudotime and filter top
    trajectory-variable, peak-linked candidate genes
  repo: https://github.com/CostaLab/scMEGA
- name: ArchR
  role: Trajectory inference approach underlying scMEGA's AddTrajectory pseudotime
    fitting
- name: Seurat
  role: Seurat object and sub-clustering/co-embedding utilities providing the input
    data structure for filtering
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad003
    title: 'scMEGA: single-cell multi-omic enhancer-based gene regulatory network
      inference'
  source_content_hash: sha256:453e869d8464b63ee9e2577f5797f4f21a26243b61fca527fd546f549c5ac3fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/scMEGA
  tools:
  - ArchR
  - Seurat
  - scMEGA
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# candidate-gene-filtering-for-grn-inference

## Summary

Selects trajectory-relevant candidate genes for enhancer-based GRN inference in scMEGA by computing per-gene expression variation along the inferred pseudotime and retaining only genes whose expression varies along the trajectory and that can be linked to at least one ATAC peak (top ~10% most variable, cutoffs user-adjustable).

## When to use

Use when building a GRN from paired snRNA-seq/snATAC-seq multiomic data with scMEGA and you need to restrict network inference to genes varying along a pseudotime trajectory that have at least one linked peak.

## When NOT to use

- Cells or clusters have not yet been assigned to a trajectory — variation-along-pseudotime filtering is undefined without AddTrajectory ordering
- Genes cannot be linked to any ATAC peak (e.g. no enhancer-to-promoter association step run) — such genes are not potential GRN targets in scMEGA
- Analysis targets a static, non-trajectory GRN (e.g. clustering-only differential expression) where pseudotime-variable selection is not the intended criterion

## Inputs

- Paired Seurat/ArchR object (e.g. coembed.sub.rds) containing snRNA-seq and snATAC-seq cells with a fitted trajectory (AddTrajectory output)
- Predefined cluster ordering defining the pseudotime (e.g. SCARA5+ progenitor cluster to terminal myofibroblast cluster)
- Gene-to-peak links (enhancer-to-promoter associations) computed from the snATAC-seq peak set

## Outputs

- Table file of trajectory-relevant candidate genes ranked by variation along pseudotime (top ~10% most variable)
- Filtered gene set in which every retained gene is associated with at least one ATAC peak, used as GRN inference input

## How to apply

After sub-clustering the co-embedded Seurat object and pairing cells across modalities (PairCells/CreatePairedObject), fit a supervised pseudotime trajectory with scMEGA's AddTrajectory (ArchR-based), ordering predefined clusters from the progenitor state to the terminal state (e.g. trajectory = c(2, 1, 0, 3) in the myofibroblast case). Compute smoothed per-gene expression along pseudotime, then call SelectGenes with labelTop1 = 0 and labelTop2 = 0 and tune var.cutoff.gene, cor.cutoff, and fdr.cutoff so that only genes varying along pseudotime and associated with at least one peak are retained. Rank genes by their variation along pseudotime and keep the top ~10% most variable (user-adjustable cutoff) as the trajectory-relevant candidate gene set feeding GRN inference. The rationale is that a gene is only a potential GRN target when its expression varies across the trajectory and its regulatory signal can be attributed to an accessible enhancer peak.

## Related tools

- **scMEGA** (Provides AddTrajectory and SelectGenes used to fit pseudotime and filter top trajectory-variable, peak-linked candidate genes) — https://github.com/CostaLab/scMEGA
- **ArchR** (Trajectory inference approach underlying scMEGA's AddTrajectory pseudotime fitting)
- **Seurat** (Seurat object and sub-clustering/co-embedding utilities providing the input data structure for filtering) — https://github.com/satijalab/seurat

## Examples

```
genes <- SelectGenes(object = obj.pair, reduction = "harmony", labelTop1 = 0, labelTop2 = 0, var.cutoff.gene = 0.9)
```

## Evaluation signals

- Output gene table is non-empty
- Every retained gene is associated with at least one ATAC peak (tutorial acceptance rule: a gene is only a potential target when linkable to a peak)
- Retained genes show measurable expression variation along the fitted pseudotime (top ~10% by variation given var.cutoff.gene/cor.cutoff/fdr.cutoff settings)
- Set labelTop1 = 0 and labelTop2 = 0 in SelectGenes so filtering is driven by pseudotime variation and peak linkage rather than top-label heuristics

## Limitations

- Software/package versions are not specified in the tutorials (only sessionInfo() output referenced)
- The top-10% cutoff and the var.cutoff.gene/cor.cutoff/fdr.cutoff parameters are user-adjustable, so results depend on chosen thresholds
- Genes not linkable to any peak are excluded even if highly variable along pseudotime

## Evidence

- [methods] Pseudo-time trajectory inference: "we infer a pseudo-time trajectory from SCARA5+ fibroblasts to myofibroblast using the approach from [ArchR]"
- [methods] Trajectory ordering in AddTrajectory: "obj.pair <- AddTrajectory(object = obj.pair, 
                          trajectory = c(2, 1, 0, 3),"
- [readme] Tutorial scope of scMEGA GRN construction: "We provided the following tutorials to show how to use scMEGA to build GRN by using single-cell multiomics/multimodal data"
