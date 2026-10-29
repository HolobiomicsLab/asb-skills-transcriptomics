---
name: multiomic-coembedded-data-handling
description: Use when single-nucleus RNA-seq and ATAC-seq from the same tissue (e.g.
  human heart post-myocardial infarction) are stored in separate Seurat objects and
  must be co-embedded, batch-corrected, and cell-paired before trajectory or GRN analysis.
when_to_use_negative:
- Data are already natively paired 10x Multiome with matched barcodes per cell — PairCells-based
  cross-modality matching is unnecessary for such objects
- Only one modality is available (e.g. snRNA-seq alone), so no co-embedding or enhancer-to-gene
  linking across modalities is possible
- Input is bulk RNA-seq/ATAC-seq rather than single-cell/nucleus data, where Seurat/ArchR
  cell-level co-embedding does not apply
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scMEGA
  role: Provides CoembedData, PairCells, CreatePairedObject, and AddTrajectory functions
    that implement the co-embedding, cell-pairing, and trajectory-ordering steps on
    the Seurat objects
  repo: https://github.com/CostaLab/scMEGA
- name: Seurat
  role: Supplies the CCA method used to project snRNA-seq and snATAC-seq data into
    a common low-dimensional space; scMEGA's CoembedData wraps these functions
  repo: https://github.com/satijalab/seurat
- name: Harmony
  role: Batch correction of the co-embedded object across patient, region, and tech
    covariates, generating a new UMAP embedding where modalities are well mixed
- name: ArchR
  role: Source of the approach used by scMEGA's AddTrajectory for pseudotime trajectory
    inference from SCARA5+ fibroblasts to myofibroblasts
- name: Nebulosa
  role: Kernel-density visualization of sparse marker features (e.g. SCARA5, COL1A1,
    POSTN, FN1) used to identify fibroblast-lineage clusters in the co-embedded object
  repo: https://github.com/powellgenomicslab/Nebulosa
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad003
    title: 'scMEGA: single-cell multi-omic enhancer-based gene regulatory network
      inference'
  source_content_hash: sha256:a0b43d156d754bf270f2495d980babdca593bfad2a8574fa107188ddc3be90ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/scMEGA
  tools:
  - ArchR
  - Harmony
  - Nebulosa
  - Seurat
  - scMEGA
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# multiomic-coembedded-data-handling

## Summary

Handles the integration stage of the scMEGA workflow: co-embedding unpaired snRNA-seq and snATAC-seq cells into a common low-dimensional space with Seurat's CCA, batch-correcting with Harmony, sub-clustering, and pairing cells across modalities so enhancer-based GRN inference can proceed on a paired Seurat object.

## When to use

Use when single-nucleus RNA-seq and ATAC-seq from the same tissue (e.g. human heart post-myocardial infarction) are stored in separate Seurat objects and must be co-embedded, batch-corrected, and cell-paired before trajectory or GRN analysis.

## When NOT to use

- Data are already natively paired 10x Multiome with matched barcodes per cell — PairCells-based cross-modality matching is unnecessary for such objects
- Only one modality is available (e.g. snRNA-seq alone), so no co-embedding or enhancer-to-gene linking across modalities is possible
- Input is bulk RNA-seq/ATAC-seq rather than single-cell/nucleus data, where Seurat/ArchR cell-level co-embedding does not apply

## Inputs

- Separate snRNA-seq and snATAC-seq Seurat objects from the same tissue (e.g. human heart after myocardial infarction), stored as .rds files such as snRNA.rds
- Metadata covariates to batch-correct on (patient, region, tech)
- Predefined marker knowledge for cluster annotation (e.g. SCARA5, COL1A1, POSTN, FN1)

## Outputs

- Batch-corrected co-embedded Seurat object (obj.coembed) with a harmony reduction and new UMAP embedding
- Sub-clustered co-embed object with annotated cell populations (coembed.sub / coembed.sub2)
- Cell-pairing data frame (df.pair) matching each snATAC-seq cell to an snRNA-seq cell
- Paired Seurat object (obj.pair) with pseudotime trajectory assignments (coembed.sub.rds)

## How to apply

Load the separate snRNA-seq and snATAC-seq Seurat objects and run scMEGA's CoembedData, which wraps Seurat's CCA-based projection into a common low-dimensional space. Apply Harmony batch correction with RunHarmony specifying group.by.vars = c("patient", "region", "tech") to mix cells from different patients, regions, and modalities, then re-compute the UMAP on the corrected embedding. Sub-cluster the co-embedded object with FindClusters (resolution = 1 in the tutorial) and identify fibroblast-lineage populations from marker expression (SCARA5-high progenitors; COL1A1/POSTN/FN1-high terminal myofibroblasts), visualizing markers with Nebulosa kernel-density plots. Match cells between modalities with PairCells on the harmony reduction — for each cell in snATAC-seq, find its closest snRNA-seq cell — and assemble the result with CreatePairedObject into obj.pair. Sub-cluster the paired object, then order clusters biologically (e.g. trajectory = c(0, 2, 1), SCARA5+ progenitor to myofibroblast) with scMEGA's AddTrajectory (ArchR-based approach), keeping only cells assigned to the trajectory. The paired, sub-clustered object (coembed.sub.rds) is the input that downstream SelectGenes/AddMotifs/RunChromVAR GRN steps consume.

## Related tools

- **scMEGA** (Provides CoembedData, PairCells, CreatePairedObject, and AddTrajectory functions that implement the co-embedding, cell-pairing, and trajectory-ordering steps on the Seurat objects) — https://github.com/CostaLab/scMEGA
- **Seurat** (Supplies the CCA method used to project snRNA-seq and snATAC-seq data into a common low-dimensional space; scMEGA's CoembedData wraps these functions) — https://github.com/satijalab/seurat
- **Harmony** (Batch correction of the co-embedded object across patient, region, and tech covariates, generating a new UMAP embedding where modalities are well mixed)
- **ArchR** (Source of the approach used by scMEGA's AddTrajectory for pseudotime trajectory inference from SCARA5+ fibroblasts to myofibroblasts)
- **Nebulosa** (Kernel-density visualization of sparse marker features (e.g. SCARA5, COL1A1, POSTN, FN1) used to identify fibroblast-lineage clusters in the co-embedded object) — https://github.com/powellgenomicslab/Nebulosa

## Examples

```
obj.coembed <- RunHarmony(obj.coembed, group.by.vars = c("patient", "region", "tech"))
```

## Evaluation signals

- After RunHarmony, UMAP inspection shows cells from different patients, regions, and modalities are well mixed
- PairCells output (df.pair) assigns exactly one matched snRNA-seq cell per snATAC-seq cell and CreatePairedObject yields a non-empty obj.pair containing both modalities
- Marker dot plot of top 10 markers per cluster resolves expected identities (e.g. cluster 1 classified as myofibroblasts; SCARA5-high and COL1A1/POSTN/FN1-high sub-populations present)
- AddTrajectory retains only cells assigned to the trajectory and the cluster order follows the biological direction from SCARA5+ progenitor to terminal myofibroblast state
- Downstream SelectGenes on the paired object returns a non-empty gene table in which every retained gene is linkable to at least one ATAC peak

## Limitations

- Software/package versions are not specified in the tutorials (only a sessionInfo() call is shown), so exact reproducibility of the co-embedding may vary across Seurat v5/Signac builds
- Trajectory ordering (e.g. trajectory = c(0, 2, 1)) is user-defined and depends on prior biological annotation of clusters; wrong ordering produces a meaningless pseudotime
- Cell pairing quality depends on the Harmony-corrected co-embedding; poor batch correction across patient/region/tech covariates degrades the ATAC-to-RNA matches

## Evidence

- [methods] we need to project the data into a common low-dimensional space. This is done by using the CCA method from [Seurat]: "we need to project the data into a common low-dimensional space. This is done by using the CCA method from [Seurat]"
- [methods] We can match the cells between these two modalities. In other words, for each cell in, for example, snATAC-seq, we will find a cell from snRNA-seq data: "We can match the cells between these two modalities. In other words, for each cell in, for example, snATAC-seq, we will find a cell from snRNA-seq data"
- [methods] after batch-correction, cells from different patients, regions, and modalities are well mixed.: "after batch-correction, cells from different patients, regions, and modalities are well mixed."
- [methods] We next perform sub-clustering to identify different populations in our multi-omic fibroblast data.: "We next perform sub-clustering to identify different populations in our multi-omic fibroblast data."
- [readme] We provided the following tutorials to show how to use scMEGA to build GRN by using single-cell multiomics/multimodal data:: "We provided the following tutorials to show how to use scMEGA to build GRN by using single-cell multiomics/multimodal data:"
