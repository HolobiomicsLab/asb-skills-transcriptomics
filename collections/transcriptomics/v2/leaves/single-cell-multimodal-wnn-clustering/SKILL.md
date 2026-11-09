---
name: single-cell-multimodal-wnn-clustering
description: Use when a droplet-based single-cell dataset contains matched mRNA and
  antibody-derived tag (ADT) measurements and you need a joint cell clustering whose
  dependence on protein normalization (e.g., dsb vs. CLR) must be evaluated.
when_to_use_negative:
- The dataset is transcript-only scRNA-seq with no ADT/protein modality — WNN multimodal
  integration is unnecessary; plain RNA PCA/clustering suffices.
- 'You have not yet normalized the protein data: comparing unnormalized ADT counts
  within WNN conflates ambient and cell-to-cell noise rather than testing normalization
  effects.'
- Isotype control antibodies and empty droplets are unavailable and no fallback normalization
  strategy has been chosen — dsb's technical component and background estimation require
  careful redefinition (see Supplementary Note).
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3940
- http://edamontology.org/topic_2229
tools:
- name: Seurat
  role: Performs CLR ADT normalization, joint WNN neighbor graph via FindMultiModalNeighbors,
    and Leiden clustering via FindClusters (algorithm = 3)
  repo: https://github.com/satijalab/seurat-data
- name: dsb
  role: Provides the ambient-noise-corrected, denoised protein values (DSBNormalizeProtein)
    used as the alternative protein modality in WNN
  repo: https://github.com/niaid/dsb
- name: mclust
  role: Fits the 2-component Gaussian mixture model per cell used inside dsb denoising
    that precedes WNN clustering
- name: limma
  role: removeBatchEffect() matrix decomposition used within dsb to compute the technical
    component removed before clustering
- name: R
  role: Execution environment; prcomp() supplies the mRNA/loading PCs combined with
    normalized protein in WNN
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:590a820435486ef8f5f0789ae820a2c5c03878cc843bcebcc355b8aff976c986
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/satijalab/seurat-data
  tools:
  - R
  - Seurat
  - dsb
  - limma
  - mclust
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3940
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# single-cell-multimodal-wnn-clustering

## Summary

Jointly cluster CITE-seq cells by combining mRNA and normalized protein (ADT) data with Seurat weighted nearest neighbors (WNN), as used in the dsb paper to compare dsb- versus CLR-normalized protein inputs within the same RNA space.

## When to use

Use when a droplet-based single-cell dataset contains matched mRNA and antibody-derived tag (ADT) measurements and you need a joint cell clustering whose dependence on protein normalization (e.g., dsb vs. CLR) must be evaluated.

## When NOT to use

- The dataset is transcript-only scRNA-seq with no ADT/protein modality — WNN multimodal integration is unnecessary; plain RNA PCA/clustering suffices.
- You have not yet normalized the protein data: comparing unnormalized ADT counts within WNN conflates ambient and cell-to-cell noise rather than testing normalization effects.
- Isotype control antibodies and empty droplets are unavailable and no fallback normalization strategy has been chosen — dsb's technical component and background estimation require careful redefinition (see Supplementary Note).

## Inputs

- Filtered and raw Cell Ranger output (UMI matrices) for RNA and ADT counts from a CITE-seq experiment
- Isotype control antibody identities for dsb denoising
- Healthy donor PBMC CITE-seq data (Kotliarov et al.) as the reference dataset in the paper

## Outputs

- WNN joint clustering assignments (Seurat FindClusters, algorithm = 3) for each protein normalization (dsb and CLR)
- Per-protein median dsb and CLR normalized values per cluster
- Pearson correlation of cluster-3 median CLR values vs. the 98th percentile of empty-droplet protein levels (expected R = 0.67, p = 3.1e-10; no such correlation under dsb)
- Differential expression gene table for cluster 3 (ROC-test), e.g., inflammatory and activation genes consistent with the CD86-high phenotype

## How to apply

Define cells as barcodes present in the filtered Cell Ranger output and background droplets as the remaining raw-output barcodes with protein reads. Normalize ADT counts two ways in parallel: dsb::DSBNormalizeProtein (denoise.counts=TRUE, use.isotype.control=TRUE with specified isotype controls) and Seurat CLR (NormalizeData with normalization.method='CLR'). For each protein normalization, run Seurat's FindMultiModalNeighbors combining the same 30 mRNA principal components with the normalized protein values (dims.list = list(1:30, ...)), then cluster with FindClusters using algorithm = 3 (Leiden). Identify matching cell populations across the two runs by marker expression (e.g., cluster 3 as CD14+ CD86-high monocytes) and compare per-protein median normalized values between normalizations to assess residual ambient noise. The rationale is that holding the RNA space (30 PCs) fixed isolates the effect of protein normalization on joint clustering and downstream correlations with empty-droplet background.

## Related tools

- **Seurat** (Performs CLR ADT normalization, joint WNN neighbor graph via FindMultiModalNeighbors, and Leiden clustering via FindClusters (algorithm = 3)) — https://github.com/satijalab/seurat-data
- **dsb** (Provides the ambient-noise-corrected, denoised protein values (DSBNormalizeProtein) used as the alternative protein modality in WNN) — https://github.com/niaid/dsb
- **mclust** (Fits the 2-component Gaussian mixture model per cell used inside dsb denoising that precedes WNN clustering)
- **limma** (removeBatchEffect() matrix decomposition used within dsb to compute the technical component removed before clustering)
- **R** (Execution environment; prcomp() supplies the mRNA/loading PCs combined with normalized protein in WNN)

## Examples

```
dsb_rescaled = DSBNormalizeProtein(cell_protein_matrix = cells_citeseq_mtx, empty_drop_matrix = empty_drop_citeseq_mtx, denoise.counts = TRUE, use.isotype.control = TRUE, isotype.control = isotype.controls)
```

## Evaluation signals

- Cluster 3 under dsb-normalized input is identifiable as CD14+ monocytes with high CD86.
- Pearson correlation of cluster-3 median CLR values against the 98th percentile of empty-droplet protein levels reproduces R = 0.67, p = 3.1e-10.
- The same correlation computed with dsb values shows no significant association (dsb accounted for ambient noise via empty droplets).
- Both WNN runs use the identical 30 mRNA principal components, so differences in clustering trace to protein normalization only.
- DE genes for cluster 3 (ROC-test) include inflammatory and activation genes consistent with the CD86-high monocyte phenotype.

## Limitations

- dsb normalized values are sensitive to how background droplets are defined; 'defining a reasonable subset of barcodes as background droplets still requires care'.
- Caveats for datasets without isotype control antibody measurements are deferred to the Supplementary Note and dsb documentation.
- Software version numbers for dsb/Seurat are not stated in the paper, so exact reproducibility may vary across versions.

## Evidence

- [discussion] Multimodal WNN clustering combining 30 mRNA principal components with normalized protein values: "the same 30 mRNA principal components were combined with either dsb or CLR normalized protein data for joint clustering"
- [discussion] Cell and background droplet definition: "Cells were defined as barcodes in the filtered Cell Ranger output, and background drops were defined as after removing the cells from the from the raw Cell Ranger output"
- [discussion] dsb denoising internals: "dsb fits a Gaussian mixture model through the expectation-maximization algorithm implemented with the mclust51 R package"
- [discussion] dsb availability and single-function interface: "The dsb method is implemented via the R package “dsb [https://cran.r-project.org/package=dsb]” through a single function call to DSBNormalizeProtein()"
- [readme] Package outputs integrate with Seurat: [evidence span withheld — anchor status: external_doc]
