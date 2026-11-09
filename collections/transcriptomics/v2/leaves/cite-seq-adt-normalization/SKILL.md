---
name: cite-seq-adt-normalization
description: Use when raw ADT/protein UMI counts from droplet-based single-cell assays
  (CITE-seq, ASAP-seq, TEA-seq) must be normalized and denoised before clustering,
  or when comparing dsb against CLR normalization for protein-based analyses.
when_to_use_negative:
- The dataset has no empty droplets and no isotype controls and the caveats for such
  cases (deferred to the dsb Supplementary Note/vignettes) have not been considered
  — use the dedicated no-empty-drops vignette workflow instead of defaulting to the
  standard recipe
- The goal is mRNA/counts normalization (e.g., scRNA-seq log-normalization) rather
  than ADT/protein counts from droplet-based assays
- Proteins have near-zero signal — those with maximum raw UMI count < 5 across all
  cells should be removed prior to normalization rather than normalized
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: dsb
  role: Core normalization/denoising method via a single call to DSBNormalizeProtein();
    also provides cells_citeseq_mtx and empty_drop_citeseq_mtx example matrices and
    ModelNegativeADTnorm
  repo: https://cran.r-project.org/package=dsb
- name: Seurat
  role: CLR baseline normalization (NormalizeData, margin=2) and joint mRNA-protein
    WNN clustering via FindMultiModalNeighbors and FindClusters
- name: mclust
  role: Fits the 2-component Gaussian mixture model per cell (EM algorithm) to extract
    background protein means used in dsb step II
- name: limma
  role: Provides removeBatchEffect() for the robust matrix decomposition used to compute
    the dsb technical component
- name: R
  role: Implementation language; prcomp() loading vectors are multiplied by the noise
    matrix
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:4ac59a3b63bc3b463a369c82f3d7e4083fec9448c6e0b06bab8e5c7302b507ac
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - dsb
  - limma
  - mclust
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# cite-seq-adt-normalization

## Summary

Normalize and denoise antibody-derived tag (ADT) counts from CITE-seq and related assays with dsb, which removes ambient unbound-antibody noise estimated from empty droplets and cell-to-cell technical noise captured by isotype controls and background protein means, outperforming CLR in downstream WNN clustering.

## When to use

Use when raw ADT/protein UMI counts from droplet-based single-cell assays (CITE-seq, ASAP-seq, TEA-seq) must be normalized and denoised before clustering, or when comparing dsb against CLR normalization for protein-based analyses.

## When NOT to use

- The dataset has no empty droplets and no isotype controls and the caveats for such cases (deferred to the dsb Supplementary Note/vignettes) have not been considered — use the dedicated no-empty-drops vignette workflow instead of defaulting to the standard recipe
- The goal is mRNA/counts normalization (e.g., scRNA-seq log-normalization) rather than ADT/protein counts from droplet-based assays
- Proteins have near-zero signal — those with maximum raw UMI count < 5 across all cells should be removed prior to normalization rather than normalized

## Inputs

- Cell-containing droplet ADT count matrix (e.g., dsb::cells_citeseq_mtx or filtered Cell Ranger output)
- Empty-droplet background ADT count matrix (e.g., dsb::empty_drop_citeseq_mtx or remaining raw Cell Ranger barcodes with protein reads)
- Isotype control antibody measurements (required for the default technical-denoising step)
- Optional: mRNA count matrix for joint WNN clustering (e.g., 30 mRNA principal components)

## Outputs

- dsb-normalized (denoised and scaled) ADT matrix of residuals, insertable into Seurat, SingleCellExperiment, or AnnData
- Per-protein median normalized-value tables for dsb and CLR comparisons
- Correlation statistics of cluster-median normalized values vs. empty-droplet 98th percentile levels
- Downstream artifacts: WNN clusters (FindMultiModalNeighbors + FindClusters, algorithm=3), Gap-Statistic tables, lineage-marker fold changes, and ROC-test differential expression gene tables

## How to apply

Define cells as barcodes in the filtered Cell Ranger output and background droplets as the remaining raw-output barcodes with protein reads; optionally remove proteins with maximum raw UMI count < 5 across all cells (e.g., CD34 was removed in two datasets). Run DSBNormalizeProtein in two conceptual steps: (I) log-transform counts (log(cell + 10)), estimate per-antibody ambient noise from empty droplets, and standardize by the background mean mu_N and standard deviation sigma_N; (II) fit a 2-component Gaussian mixture model per cell (mclust, EM) to extract the background mean mu1, combine the fitted mu1 values with isotype controls into a noise matrix, compute PC1 as the technical component (prcomp / limma::removeBatchEffect decomposition), and regress it out, returning residuals. Use parameters such as denoise.counts=TRUE, use.isotype.control=TRUE, quantile.clipping=TRUE with quantile.clip=c(0.001, 0.9995), and isotype controls at specified rows (e.g., 67:70). In parallel, CLR-normalize (Seurat NormalizeData, normalization.method='CLR', margin=2) for comparison; validate by checking that dsb removes the correlation between cluster-median CLR values and the 98th percentile of empty-droplet expression (CLR: R = 0.67, p = 3.1e-10 for CD14+ CD86-high monocyte cluster 3; dsb: no such correlation) and yields higher Gap-Statistics for k-medoids protein clustering.

## Related tools

- **dsb** (Core normalization/denoising method via a single call to DSBNormalizeProtein(); also provides cells_citeseq_mtx and empty_drop_citeseq_mtx example matrices and ModelNegativeADTnorm) — https://cran.r-project.org/package=dsb
- **Seurat** (CLR baseline normalization (NormalizeData, margin=2) and joint mRNA-protein WNN clustering via FindMultiModalNeighbors and FindClusters)
- **mclust** (Fits the 2-component Gaussian mixture model per cell (EM algorithm) to extract background protein means used in dsb step II)
- **limma** (Provides removeBatchEffect() for the robust matrix decomposition used to compute the dsb technical component)
- **R** (Implementation language; prcomp() loading vectors are multiplied by the noise matrix)

## Examples

```
dsb_rescaled <- DSBNormalizeProtein(cell_protein_matrix = cells_citeseq_mtx, denoise.counts = TRUE, use.isotype.control = TRUE, isotype.control = rownames(cells_citeseq_mtx)[67:70], quantile.clipping = TRUE, quantile.clip = c(0.001, 0.9995))
```

## Evaluation signals

- Under CLR, median normalized values of the dsb-defined CD14+ CD86-high monocyte cluster 3 correlate with the 98th percentile of empty-droplet protein levels (expected R = 0.67, p = 3.1e-10); under dsb this correlation should be absent — a successful run removes this ambient-noise signature.
- Gap-Statistic from k-medoids clustering on protein data alone (k = 1..20) should be consistently higher for dsb than CLR normalized matrices (Supplementary Fig. 3f,g), and lineage-marker fold changes (e.g., CD56 for NK cells, CD8/CD4 for T cells) should be sharper under dsb.
- Cluster 3 under dsb-WNN should be identifiable as CD14+ monocytes with high CD86, with differentially expressed transcripts (ROC-test) enriched for inflammatory and activation genes consistent with the CD86-high phenotype.
- Sanity-check the output matrix schema: dsb returns standard R matrices insertable into SingleCellExperiment, Seurat, or AnnData containers, with one row per antibody after any low-signal protein removal (e.g., CD34 dropped where max UMI < 5).
- Verify that denoising parameters were honored: denoise.counts=TRUE, use.isotype.control=TRUE with the specified isotype rows, and quantile clipping at c(0.001, 0.9995) if clipping is enabled.

## Limitations

- dsb normalized values are sensitive to how background droplets are defined; defining a reasonable subset of barcodes as background requires care.
- Batch normalization behavior depends on the extent of batch uniformity among batches.
- Caveats for datasets without isotype control antibody measurements, and guidance on the technical component in such experiments, are deferred to the Supplementary Note and CRAN documentation rather than the main text.
- Software version numbers for the tools used are not stated in the paper; antibody lists are only available in supplementary Data 1.

## Evidence

- [discussion] In step I, protein counts in empty droplets are used to estimate the expected ambient background noise for each antibody: "In step I, protein counts in empty droplets are used to estimate the expected ambient background noise for each antibody"
- [discussion] dsb fits a Gaussian mixture model via mclust to extract background mean: "dsb fits a Gaussian mixture model through the expectation-maximization algorithm implemented with the mclust51 R package to the transformed count of each cell from step 1 with k = 2 mixture components"
- [discussion] Proteins with very low raw data signal removed prior to normalization: "Proteins with very low raw data signal (a maximum UMI count <5 across all cells) were removed prior to normalization, resulting in removal of the CD34 protein from two datasets."
- [discussion] Cells defined from filtered Cell Ranger output; background from raw: "Cells were defined as barcodes in the filtered Cell Ranger output, and background drops were defined as after removing the cells from the from the raw Cell Ranger output"
- [methods] Methods code: log transformation of ADT counts: "# log transformation
dlog = log(cell + 10)"
- [discussion] The dsb method is implemented via a single function call: "The dsb method is implemented via the R package “dsb [https://cran.r-project.org/package=dsb]” through a single function call to DSBNormalizeProtein()"
- [readme] README: dsb normalizes and denoises ADT data; install from CRAN: "dsb: Normalize and denoise antibody-derived-tag data from CITE-seq, ASAP-seq, TEA-seq and related assays."
