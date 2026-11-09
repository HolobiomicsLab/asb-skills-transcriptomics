---
name: single-nucleus-multiomic-data-integration
description: 'Use when you have single-nucleus RNA and ATAC assays from the same tissue
  (e.g. human heart post-myocardial infarction Seurat objects: snRNA.rds, snATAC.rds,
  gene.activity.rds) and need a joint embedding for clustering, trajectory, or GRN
  inference.'
when_to_use_negative:
- Data are already paired 10X Multiome measurements with matched cell barcodes — a
  dedicated multimodal workflow (e.g. the pbmc_10x_multiome tutorial) avoids greedy
  pairing
- Only one modality is available (RNA-only or ATAC-only); CCA-based co-embedding requires
  both assays plus a gene activity matrix
- Goal is differential expression or peak calling within a single modality, not cross-modality
  integration or GRN construction
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3934
tools:
- name: scMEGA
  role: Provides CoembedData, PairCells, CreatePairedObject wrappers around Seurat/ArchR
    functions for integration and cell pairing
  repo: https://github.com/CostaLab/scMEGA
- name: Seurat
  role: CCA-based co-embedding of RNA and ATAC modalities; FindClusters and UMAP on
    the joint embedding
  repo: https://github.com/satijalab/seurat (branch seurat5)
- name: Signac
  role: Handles the single-cell chromatin (ATAC) assay objects within Seurat v5
  repo: https://github.com/stuart-lab/signac (branch seurat5)
- name: harmony
  role: Batch correction of the co-embedded object across patient, region, and tech
    variables
- name: ArchR
  role: Approach used for pseudo-time trajectory inference on the integrated object
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad003
    title: 'scMEGA: single-cell multi-omic enhancer-based gene regulatory network
      inference'
  source_content_hash: sha256:3f388ee236fb918dc13562d9fbc0283b950898d34fd6f09fc62b3a61030b858d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/scMEGA
  tools:
  - ArchR
  - JASPAR2020
  - Nebulosa
  - R
  - Seurat
  - Signac
  - TFBSTools
  - destiny
  - devtools
  - ggraph
  - harmony
  - igraph
  - optmatch
  - scMEGA
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3934
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# single-nucleus-multiomic-data-integration

## Summary

Integrates matched or unpaired snRNA-seq and snATAC-seq data by co-embedding the two modalities in a shared low-dimensional space (Seurat CCA on the gene activity matrix), batch-correcting with Harmony, and pairing cells across modalities so downstream trajectory and GRN inference operates on a joint multi-omic object.

## When to use

Use when you have single-nucleus RNA and ATAC assays from the same tissue (e.g. human heart post-myocardial infarction Seurat objects: snRNA.rds, snATAC.rds, gene.activity.rds) and need a joint embedding for clustering, trajectory, or GRN inference.

## When NOT to use

- Data are already paired 10X Multiome measurements with matched cell barcodes — a dedicated multimodal workflow (e.g. the pbmc_10x_multiome tutorial) avoids greedy pairing
- Only one modality is available (RNA-only or ATAC-only); CCA-based co-embedding requires both assays plus a gene activity matrix
- Goal is differential expression or peak calling within a single modality, not cross-modality integration or GRN construction

## Inputs

- Seurat objects: snRNA.rds (nucleus gene expression), snATAC.rds (peaks), gene.activity.rds (gene activity matrix)
- Batch/covariate metadata columns (e.g. patient, region, tech, patient_group)
- BSgenome.Hsapiens.UCSC.hg38 genome (for subsequent motif/TF steps)

## Outputs

- Co-embedded and Harmony-corrected Seurat object with 'harmony' reduction and UMAP on dims 1:30
- Cluster assignments at resolution 1 and sub-clusters at resolution 0.2 for retained populations
- Paired multimodal Seurat object linking snATAC and snRNA cells (PairCells/CreatePairedObject)
- Diffusion-map embedding for trajectory inference

## How to apply

Convert both RNA and ATAC assays to Seurat v5 Assay5, then use scMEGA's CoembedData wrapper, which runs Seurat's CCA on the gene activity matrix to project snRNA-seq and snATAC-seq cells into a common low-dimensional space. Batch-correct the co-embedded object with RunHarmony (grouping by variables such as patient, region, and tech) and recompute a UMAP on harmony dims 1:30, confirming cells from different patients, regions, and modalities are well mixed. Perform two-round sub-clustering: FindClusters at resolution 1, retain clusters showing significant group differences via CompareCellProp across a biological covariate (e.g. patient_group myogenic/ischemic/fibrotic), then re-cluster retained cells at resolution 0.2. For downstream GRN work, match snATAC-seq cells to snRNA-seq cells with PairCells on the harmony reduction (pair.mode='greedy'), assemble a multimodal object with CreatePairedObject, and reduce dimensions with RunDiffusionMap on the harmony embedding to support pseudotime trajectory inference (AddTrajectory).

## Related tools

- **scMEGA** (Provides CoembedData, PairCells, CreatePairedObject wrappers around Seurat/ArchR functions for integration and cell pairing) — https://github.com/CostaLab/scMEGA
- **Seurat** (CCA-based co-embedding of RNA and ATAC modalities; FindClusters and UMAP on the joint embedding) — https://github.com/satijalab/seurat (branch seurat5)
- **Signac** (Handles the single-cell chromatin (ATAC) assay objects within Seurat v5) — https://github.com/stuart-lab/signac (branch seurat5)
- **harmony** (Batch correction of the co-embedded object across patient, region, and tech variables)
- **ArchR** (Approach used for pseudo-time trajectory inference on the integrated object)

## Examples

```
obj.coembed <- RunHarmony(obj.coembed, group.by.vars = c("patient", "region", "tech"))
```

## Evaluation signals

- Post-Harmony UMAP shows cells from different patients, regions, and modalities well mixed rather than separating by batch
- Co-embedding is computed on the gene activity matrix via CCA; verify the harmony reduction exists and UMAP used dims 1:30
- Sub-clustering: resolution 1 clusters with significant differences by CompareCellProp across patient_group are retained, then re-clustered at resolution 0.2
- PairCells with pair.mode='greedy' on the harmony reduction produces a df.pair table that CreatePairedObject converts into a valid paired object with both assays
- Marker sanity check on sub-clusters: SCARA5-high cells mark progenitors and COL1A1/POSTN/FN1-high cells mark terminally differentiated myofibroblasts

## Limitations

- Software/package versions are not pinned in the tutorials (only sessionInfo() is suggested), so reproducibility across R/Bioconductor versions is not guaranteed
- For unpaired nuclei, cell pairing between modalities is greedy (PairCells pair.mode='greedy'), so RNA-ATAC cell assignments are approximate, not molecule-level links
- Batch correction requires explicit covariates (patient, region, tech); omission leads to modality- or donor-driven separation in the embedding

## Evidence

- [methods] Co-embedding via CCA: "we need to project the data into a common low-dimensional space. This is done by using the CCA method from [Seurat]"
- [methods] Harmony batch correction and mixing check: "after batch-correction, cells from different patients, regions, and modalities are well mixed."
- [methods] CoembedData wrapper: "we have wrapped several functions from Seurat into a single function _CoembedData_."
- [methods] Cell pairing between modalities: "We can match the cells between these two modalities. In other words, for each cell in, for example, snATAC-seq, we will find a cell from snRNA-seq data"
- [methods] Sub-clustering step: "We next perform sub-clustering to identify different populations in our multi-omic fibroblast data."
- [readme] Install instructions (Seurat v5/Signac branches): "# First install Seurate v5
devtools::install_github("satijalab/seurat", "seurat5", quiet = TRUE)"
