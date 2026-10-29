---
name: clr-protein-count-normalization
description: Use when normalizing droplet-based protein (ADT) counts from CITE-seq/TEA-seq
  data and you need a standard baseline normalization against which background-corrected
  methods like dsb can be compared, or when empty-droplet background matrices are
  not being modeled.
when_to_use_negative:
- 'When ambient, unbound-antibody background noise must be removed: CLR does not use
  empty-droplet background and retains correlation with ambient ADT levels (r≈0.8
  in TEA-seq), whereas dsb corrects for it'
- When isotype-control-based cell-to-cell technical denoising is needed; CLR performs
  no per-cell noise modeling (no Gaussian mixture / technical component regression
  as in dsb)
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: Performs CLR normalization via NormalizeData on the ADT assay and downstream
    WNN multimodal integration and clustering
  repo: https://github.com/satijalab/seurat-data
- name: dsb
  role: Parallel comparator method whose ambient- and cell-level denoising CLR is
    benchmarked against
  repo: https://cran.r-project.org/package=dsb
- name: R
  role: Runtime environment for the CLR normalization and comparison workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:564ea1fc6e4da1e152cb8ea277d6641cad7103a43a3f5bae6999d595352ab61a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/satijalab/seurat-data
  tools:
  - R
  - Seurat
  - dsb
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# clr-protein-count-normalization

## Summary

Centered log-ratio (CLR) normalization of the ADT/protein count matrix, e.g. via Seurat's NormalizeData on the ADT assay, used here as the baseline comparator to dsb normalization in CITE-seq/TEA-seq workflows.

## When to use

Use when normalizing droplet-based protein (ADT) counts from CITE-seq/TEA-seq data and you need a standard baseline normalization against which background-corrected methods like dsb can be compared, or when empty-droplet background matrices are not being modeled.

## When NOT to use

- When ambient, unbound-antibody background noise must be removed: CLR does not use empty-droplet background and retains correlation with ambient ADT levels (r≈0.8 in TEA-seq), whereas dsb corrects for it
- When isotype-control-based cell-to-cell technical denoising is needed; CLR performs no per-cell noise modeling (no Gaussian mixture / technical component regression as in dsb)

## Inputs

- Filtered Cell Ranger ADT (protein) UMI count matrix (cells x proteins, e.g. 45-protein TEA-seq panel)
- Seurat object with an ADT assay containing raw protein counts
- Standard scRNA-seq QC-filtered cell barcode list (mRNA content, mitochondrial proportion, protein library size)

## Outputs

- CLR-normalized ADT expression matrix per cell
- WNN-integrated joint mRNA+ADT clustering under CLR normalization
- Per-protein correlation table of median CLR values vs 98th-percentile empty-droplet expression

## How to apply

Apply CLR to the filtered cell-by-protein ADT count matrix (e.g. the 45-protein TEA-seq panel) after standard scRNA-seq QC on mRNA content, mitochondrial proportion, and protein library size; in Seurat this is done by running NormalizeData on the ADT assay. Use the CLR-normalized values in parallel with dsb-normalized values to compute 30 mRNA principal components and run Seurat FindMultiModalNeighbors WNN integration (dims.list = list(1:30, 1:18)) followed by FindClusters (algorithm=3, resolution=2). The rationale for including CLR as a comparator is that CLR values remain correlated with ambient noise: in TEA-seq data, median CLR values (but not dsb values) correlated with the 98th percentile empty-droplet expression (r=0.8, p=2.4e-7 vs r=0.13, p=0.39), which can misattribute ambient antibody signal to biological populations such as the TCR-Va7.2+ MAIT cells in cluster 14.

## Related tools

- **Seurat** (Performs CLR normalization via NormalizeData on the ADT assay and downstream WNN multimodal integration and clustering) — https://github.com/satijalab/seurat-data
- **dsb** (Parallel comparator method whose ambient- and cell-level denoising CLR is benchmarked against) — https://cran.r-project.org/package=dsb
- **R** (Runtime environment for the CLR normalization and comparison workflow)

## Examples

```
adt_norm <- NormalizeData(seurat_obj, assay = 'ADT', normalization.method = 'CLR', margin = 2)
```

## Evaluation signals

- Correlate per-protein median CLR values across cells with the 98th percentile of that protein's expression in empty droplets; expect high correlation (r≈0.8, p≈2.4e-7), confirming ambient noise drives CLR values
- Joint mRNA+ADT clustering under CLR should fail to isolate the TCR-Va7.2+ MAIT population: cluster 14 appears as conventional T cells rather than a distinct Va7.2+ cluster
- Verify CLR values are only applied to cells passing QC (mRNA content, mitochondrial proportion, protein library size) and cover the full 45-protein panel
- Compare CLR vs dsb correlation tables: CLR r≈0.8 vs dsb r≈0.13 (p=0.39) is the expected differential signature

## Limitations

- CLR values correlate strongly with ambient unbound-antibody levels in empty droplets, causing ambient noise to be misinterpreted as biological signal (e.g. apparent TCR-Va7.2 positivity in non-MAIT clusters)
- Without background correction, cluster-level protein medians under CLR can misidentify cell populations (TEA-seq cluster 14 MAIT cells misread as conventional T cells)

## Evidence

- [discussion] Multimodal WNN clustering with CLR baseline: "the same 30 mRNA principal components were combined with either dsb or CLR normalized protein data for joint clustering"
- [abstract] Ambient noise source motivates CLR comparator: "protein-specific noise originates from unbound antibodies encapsulated during droplet generation"
- [abstract] dsb unmasks populations CLR obscures: "Compared to existing normalization methods, our approach improves downstream analyses by better unmasking biologically meaningful cell populations"
- [discussion] Background droplet definition: "Cells were defined as barcodes in the filtered Cell Ranger output, and background drops were defined as after removing the cells from the from the raw Cell Ranger output"
- [readme] dsb package overview: "dsb: Normalize and denoise antibody-derived-tag data from CITE-seq, ASAP-seq, TEA-seq and related assays."
