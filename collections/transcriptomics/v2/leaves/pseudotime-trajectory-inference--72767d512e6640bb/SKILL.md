---
name: pseudotime-trajectory-inference
description: Use when a single-cell multi-omic dataset has been co-embedded, batch-corrected
  (RunHarmony) and sub-clustered, and the goal is to order clusters along a differentiation
  continuum (e.g. progenitors to myofibroblasts) for GRN inference.
when_to_use_negative:
- Cells do not lie on a differentiation/transition continuum (e.g. static, unrelated
  populations) — the user-supplied cluster ordering would be arbitrary and the trajectory
  meaningless.
- Modalities have not been co-embedded and batch-corrected (CoembedData + RunHarmony),
  so snRNA and snATAC cells cannot be jointly ordered or paired with PairCells.
- No meaningful sub-clusters with significant group differences (CompareCellProp)
  exist to serve as trajectory nodes.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3172
tools:
- name: scMEGA
  role: provides AddTrajectory (ArchR-based supervised pseudotime), PairCells, CreatePairedObject,
    SelectGenes and SelectTFs that consume/produce the trajectory
  repo: github.com/CostaLab/scMEGA
- name: ArchR
  role: source of the trajectory-inference approach re-implemented by scMEGA's AddTrajectory
- name: Seurat
  role: supplies co-embedding (CCA), FindClusters sub-clustering and dimension reductions
    (harmony/diffusion-map) that the trajectory is built on
  repo: github.com/satijalab/seurat
- name: destiny
  role: diffusion-map embedding used to order cells before trajectory fitting
provenance:
  source_task_ids:
  - task_007
  - task_005
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad003
    title: 'scMEGA: single-cell multi-omic enhancer-based gene regulatory network
      inference'
  source_content_hash: sha256:593d412f5e6e4b8008a0b9a338ebc344a896d37369e8762e5ba470e6f4ada268
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ArchR
  - BSgenome.Hsapiens.UCSC.hg38
  - JASPAR2020
  - Nebulosa
  - R
  - Seurat
  - Signac
  - TFBSTools
  - destiny
  - devtools
  - ggraph
  - igraph
  - optmatch
  - scMEGA
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 20
---

# pseudotime-trajectory-inference

## Summary

Fits a supervised pseudotime trajectory over a co-embedded snRNA/snATAC Seurat object using scMEGA's ArchR-based AddTrajectory, ordering predefined cell clusters from a progenitor state (SCARA5+ fibroblasts) to a terminal state (POSTN/COL1A1+ myofibroblasts) so that trajectory-dependent genes and TFs can be selected downstream.

## When to use

Use when a single-cell multi-omic dataset has been co-embedded, batch-corrected (RunHarmony) and sub-clustered, and the goal is to order clusters along a differentiation continuum (e.g. progenitors to myofibroblasts) for GRN inference.

## When NOT to use

- Cells do not lie on a differentiation/transition continuum (e.g. static, unrelated populations) — the user-supplied cluster ordering would be arbitrary and the trajectory meaningless.
- Modalities have not been co-embedded and batch-corrected (CoembedData + RunHarmony), so snRNA and snATAC cells cannot be jointly ordered or paired with PairCells.
- No meaningful sub-clusters with significant group differences (CompareCellProp) exist to serve as trajectory nodes.

## Inputs

- co-embedded, batch-corrected (RunHarmony) and sub-clustered Seurat object of paired snRNA-seq/snATAC-seq fibroblast cells (coembed.sub.rds)
- harmony reduction with diffusion-map embedding (RunDiffusionMap dims 1:3)
- predefined cluster order reflecting the expected progenitor-to-terminal transition

## Outputs

- Seurat/paired object annotated with pseudotime trajectory values (coembed.trajectory.rds)
- trajectory-relevant gene table: top 10% most variable genes along pseudotime, each linked to at least one peak
- trajectory-correlated TF table (df.cor with tf and time_point columns) and TF activity dynamics heatmap

## How to apply

First run a diffusion map on the harmony embedding (RunDiffusionMap) of the sub-clustered co-embed object, then pair snATAC and snRNA cells with PairCells (pair.mode='greedy') and assemble a multimodal object with CreatePairedObject. Call scMEGA's AddTrajectory (the ArchR approach) supplying an explicit cluster order along the diffusion-map/UMAP reduction — e.g. trajectory = c(0, 2, 1) from SCARA5+ progenitors to myofibroblasts, computed on diffusion-map dims 1:3 — so only cells on the path are retained. The cluster order is user-supervised: verify biologically via marker expression (SCARA5-high progenitors; COL1A1/POSTN/FN1-high terminal myofibroblasts) and CompareCellProp significance across patient_group before ordering. The resulting pseudotime feeds SelectGenes (top 10% most variable genes along pseudotime, each linkable to at least one ATAC peak) and SelectTFs (chromVAR activity correlated with TF expression, cor.cutoff ~0.1), which are prerequisites for GetGRN assembly.

## Related tools

- **scMEGA** (provides AddTrajectory (ArchR-based supervised pseudotime), PairCells, CreatePairedObject, SelectGenes and SelectTFs that consume/produce the trajectory) — github.com/CostaLab/scMEGA
- **ArchR** (source of the trajectory-inference approach re-implemented by scMEGA's AddTrajectory)
- **Seurat** (supplies co-embedding (CCA), FindClusters sub-clustering and dimension reductions (harmony/diffusion-map) that the trajectory is built on) — github.com/satijalab/seurat
- **destiny** (diffusion-map embedding used to order cells before trajectory fitting)

## Examples

```
obj.pair <- AddTrajectory(object = obj.pair, trajectory = c(2, 1, 0, 3))
```

## Evaluation signals

- AddTrajectory returns a non-empty object retaining only cells assigned to the supplied cluster path (e.g. c(0, 2, 1)); clusters off the path are dropped.
- Cluster ordering matches marker biology: trajectory starts in SCARA5-high progenitors and ends in COL1A1/POSTN/FN1-high myofibroblasts (dot-plot annotation of cluster 1 as myofibroblasts).
- SelectGenes output gene table is non-empty and every retained gene is associated with at least one ATAC peak.
- SelectTFs (return.heatmap = TRUE) yields df.cor with only TFs whose activity-expression correlation exceeds the cutoff and a heatmap showing TF activity dynamics along pseudotime.
- Downstream GRNHeatmap resolves two regulatory modules matching the SCARA5+ progenitor and POSTN/COL1A1+ myofibroblast sub-clusters.

## Limitations

- Trajectory is supervised: the user must supply the cluster order (trajectory = c(...)); an incorrect biological ordering propagates into gene/TF selection and the final GRN.
- Cell pairing between modalities uses greedy matching (PairCells), so pairs may be imperfect in regions of low cell-type overlap.
- Software/package versions are not specified in the tutorials (only sessionInfo() is referenced), so reproducibility across R/Bioconductor versions is not guaranteed.

## Evidence

- [methods] we infer a pseudo-time trajectory from SCARA5+ fibroblasts to myofibroblast using the approach from [ArchR]: "we infer a pseudo-time trajectory from SCARA5+ fibroblasts to myofibroblast using the approach from [ArchR]"
- [methods] obj.pair <- AddTrajectory(object = obj.pair, 
                          trajectory = c(2, 1, 0, 3),: "obj.pair <- AddTrajectory(object = obj.pair, 
                          trajectory = c(2, 1, 0, 3),"
- [methods] We next perform sub-clustering to identify different populations in our multi-omic fibroblast data.: "We next perform sub-clustering to identify different populations in our multi-omic fibroblast data."
- [readme] We provided the following tutorials to show how to use scMEGA to build GRN by using single-cell multiomics/multimodal data: "We provided the following tutorials to show how to use scMEGA to build GRN by using single-cell multiomics/multimodal data"
