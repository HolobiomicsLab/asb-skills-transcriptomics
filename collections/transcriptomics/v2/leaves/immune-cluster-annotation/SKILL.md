---
name: immune-cluster-annotation
description: Use when annotating clusters from Seurat WNN joint mRNA+ADT analysis
  of PBMC CITE-seq data by matching protein markers (CD14, CD86) to immune cell identities
  and confirming phenotypes with differential gene expression.
when_to_use_negative:
- Data are mRNA-only scRNA-seq with no ADT/protein modality, so clusters cannot be
  anchored by canonical surface-protein markers
- Protein normalization has not yet been performed — dsb requires cells and background
  droplets defined from Cell Ranger filtered/raw outputs before WNN clustering and
  annotation
- Datasets lacking isotype control antibody measurements without consulting the dsb
  Supplementary Note guidance on defining the technical component
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: Seurat
  role: Performs WNN multimodal clustering (FindMultiModalNeighbors) with 30 mRNA
    PCs plus normalized ADT values, FindClusters (algorithm = 3), CLR normalization,
    and differential expression testing
  repo: https://github.com/satijalab/seurat-data
- name: dsb
  role: Denoises and normalizes ADT counts via DSBNormalizeProtein (denoise.counts=TRUE,
    use.isotype.control=TRUE) so cluster annotation reflects biology rather than ambient
    antibody noise
  repo: https://github.com/niaid/dsb
- name: mclust
  role: Gaussian mixture model EM implementation underlying dsb's per-cell background
    mean estimation used in the normalized values feeding the annotated clusters
- name: limma
  role: Provides removeBatchEffect() for the robust matrix decomposition used by dsb
    in producing denoised protein values
- name: R
  role: Runtime environment; prcomp() computes the technical noise component and correlation/DE
    analyses are executed in R
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:835658d578ce6d666adaa69f7dd3621d5ee87e226120ce43d205bded8bdb2932
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# immune-cluster-annotation

## Summary

Annotate immune cell populations from joint mRNA-protein (CITE-seq) clustering by identifying clusters via canonical marker proteins (e.g., CD14+ monocytes with high CD86) and validating their biological identity with differential transcript expression, as demonstrated for the CD86-high monocyte cluster 3 in the dsb Nature Communications study.

## When to use

Use when annotating clusters from Seurat WNN joint mRNA+ADT analysis of PBMC CITE-seq data by matching protein markers (CD14, CD86) to immune cell identities and confirming phenotypes with differential gene expression.

## When NOT to use

- Data are mRNA-only scRNA-seq with no ADT/protein modality, so clusters cannot be anchored by canonical surface-protein markers
- Protein normalization has not yet been performed — dsb requires cells and background droplets defined from Cell Ranger filtered/raw outputs before WNN clustering and annotation
- Datasets lacking isotype control antibody measurements without consulting the dsb Supplementary Note guidance on defining the technical component

## Inputs

- Filtered Cell Ranger PBMC CITE-seq UMI matrices (RNA + ADT) with raw-output barcodes retained to define empty droplets
- dsb-normalized ADT matrix (DSBNormalizeProtein with denoise.counts=TRUE, use.isotype.control=TRUE)
- CLR-normalized ADT matrix (Seurat NormalizeData, normalization.method='CLR')
- Seurat object with WNN neighbors from 30 mRNA PCs plus normalized protein values
- Empty-droplet (background) ADT count matrix for ambient-noise QC

## Outputs

- Per-protein median dsb and CLR normalized values for the annotated cluster (e.g., cluster 3 tables)
- Pearson correlation statistic of cluster-3 median CLR values vs. 98th percentile of empty-droplet protein levels (R = 0.67, p = 3.1e-10)
- Differentially expressed gene table for the annotated cluster (ROC-test), confirming inflammatory/activation gene signatures

## How to apply

Run Seurat WNN (FindMultiModalNeighbors) combining 30 mRNA principal components with normalized protein values (dsb or CLR), then cluster with FindClusters (algorithm = 3). Examine per-cluster median normalized protein values to identify populations by canonical markers — the CD14+ cluster with high CD86 defines the CD86-high monocyte population (cluster 3 in the dsb analysis). Confirm the annotation by running ROC-test differential expression for the cluster and checking that recovered transcripts match the expected biology (inflammatory and activation genes consistent with the CD86-high phenotype). Compute per-protein median normalized values (both dsb and CLR inputs) and, for quality control of ambient noise, correlate the cluster-3 median CLR values against the 98th percentile of empty-droplet protein levels (expected R = 0.67, p = 3.1e-10 under CLR; absent under dsb) to verify the annotation is not driven by ambient antibody noise.

## Related tools

- **Seurat** (Performs WNN multimodal clustering (FindMultiModalNeighbors) with 30 mRNA PCs plus normalized ADT values, FindClusters (algorithm = 3), CLR normalization, and differential expression testing) — https://github.com/satijalab/seurat-data
- **dsb** (Denoises and normalizes ADT counts via DSBNormalizeProtein (denoise.counts=TRUE, use.isotype.control=TRUE) so cluster annotation reflects biology rather than ambient antibody noise) — https://github.com/niaid/dsb
- **mclust** (Gaussian mixture model EM implementation underlying dsb's per-cell background mean estimation used in the normalized values feeding the annotated clusters)
- **limma** (Provides removeBatchEffect() for the robust matrix decomposition used by dsb in producing denoised protein values)
- **R** (Runtime environment; prcomp() computes the technical noise component and correlation/DE analyses are executed in R)

## Examples

```
dsb_rescaled = DSBNormalizeProtein(cell_protein_matrix = cells_citeseq_mtx, empty_drop_matrix = empty_drop_citeseq_mtx, denoise.counts = TRUE, use.isotype.control = TRUE, isotype.controls = isotype.controls)
```

## Evaluation signals

- Cluster 3 identified as the CD14+ monocyte population with high CD86 median protein values under dsb normalization
- Pearson correlation of cluster-3 median CLR values vs. 98th percentile empty-droplet protein levels reproduces R = 0.67 (p = 3.1e-10), and this correlation is absent for dsb values
- ROC-test differential expression for cluster 3 recovers inflammatory and activation genes consistent with the CD86-high monocyte phenotype
- WNN graph built from exactly 30 mRNA principal components combined with normalized protein values, clustered with algorithm = 3
- Normalized median-value tables, correlation statistic, and DE gene table are all written to output files

## Limitations

- Ambient antibody noise remains present in CLR-normalized ADT values and can contaminate cluster-level protein summaries; dsb normalization via empty droplets is needed to remove it
- dsb normalized values are sensitive to how background droplets are defined; defining a reasonable subset of barcodes as background requires care
- Caveats for using dsb on datasets without isotype control antibody measurements are deferred to the Supplementary Note

## Evidence

- [discussion] Median CLR values (but not dsb values) correlated with the 98th percentile of expression in empty droplets across proteins (R = 0.67, p = 3.1e−10): "the same 30 mRNA principal components were combined with either dsb or CLR normalized protein data for joint clustering"
- [discussion] dsb fits a Gaussian mixture model through the expectation-maximization algorithm implemented with the mclust R package: "dsb fits a Gaussian mixture model through the expectation-maximization algorithm implemented with the mclust51 R package"
- [discussion] dsb uses a function from the limma package removeBatchEffect() for robust and efficient matrix decomposition: "dsb uses a function from the limma52 package removeBatchEffect() for robust and efficient matrix decomposition"
- [methods] the DSBNormalizeProtein function call with cells and empty droplet matrices: "dsb_rescaled = DSBNormalizeProtein(cell_protein_matrix = cells_citeseq_mtx"
- [readme] The dsb R package is available on CRAN and installs via install.packages, with functions returning standard R matrix objects usable with Seurat: [evidence span withheld — anchor status: external_doc]
