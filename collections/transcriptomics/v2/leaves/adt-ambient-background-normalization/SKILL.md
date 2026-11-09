---
name: adt-ambient-background-normalization
description: Use when raw ADT/protein count matrices from droplet-based single cell
  assays (CITE-seq, TEA-seq, ASAP-seq) show ambient antibody noise that masks biologically
  meaningful populations — e.g., empty-droplet signal correlating with CLR-normalized
  protein values.
when_to_use_negative:
- When no empty droplets are available and no model-based background can be estimated
  — use the dsb 'no empty drops' vignette variant (ModelNegativeADTnorm) instead,
  not the empty-drop background workflow
- When the protein panel contains proteins with very low signal (maximum raw UMI count
  <5 across all cells) — these must be removed prior to normalization (e.g., CD34
  was removed from two datasets)
- When the analysis goal is mRNA-only scRNA-seq normalization — dsb targets ADT/protein
  data specifically
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: dsb
  role: Core normalization function DSBNormalizeProtein() performing ambient background
    subtraction and technical-component regression on ADT counts
  repo: https://cran.r-project.org/package=dsb
- name: mclust
  role: Fits the 2-component Gaussian mixture model (EM) per cell to extract the background
    protein subpopulation mean µ1
- name: limma
  role: removeBatchEffect() used for robust and efficient matrix decomposition of
    the noise variable matrix
- name: R
  role: Implementation language; prcomp() loading vectors multiplied by the noise
    matrix
- name: Seurat
  role: CLR normalization comparator and downstream WNN integration (FindMultiModalNeighbors,
    dims.list = list(1:30, 1:18)) and clustering of dsb vs CLR protein values with
    30 mRNA PCs
  repo: https://github.com/satijalab/seurat-data
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:3bc68b8186182405f100e0c452046bbb94021a26271cbb5af951d4025d74489b
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
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# adt-ambient-background-normalization

## Summary

Normalize and denoise droplet-based ADT (protein) data from CITE-seq/TEA-seq/ASAP-seq using the dsb method, which removes ambient unbound-antibody background estimated from empty droplets plus cell-to-cell technical noise via isotype controls and a per-cell background mean.

## When to use

Use when raw ADT/protein count matrices from droplet-based single cell assays (CITE-seq, TEA-seq, ASAP-seq) show ambient antibody noise that masks biologically meaningful populations — e.g., empty-droplet signal correlating with CLR-normalized protein values.

## When NOT to use

- When no empty droplets are available and no model-based background can be estimated — use the dsb 'no empty drops' vignette variant (ModelNegativeADTnorm) instead, not the empty-drop background workflow
- When the protein panel contains proteins with very low signal (maximum raw UMI count <5 across all cells) — these must be removed prior to normalization (e.g., CD34 was removed from two datasets)
- When the analysis goal is mRNA-only scRNA-seq normalization — dsb targets ADT/protein data specifically

## Inputs

- Raw Cell Ranger mRNA + ADT UMI count matrices (10X mtx format)
- Filtered Cell Ranger mRNA + ADT matrices (cell-containing barcodes)
- Empty-droplet ADT count matrix (raw barcodes minus filtered cells with protein reads)
- Isotype control antibody measurements (optional but recommended for denoise.counts=TRUE)

## Outputs

- dsb-normalized ADT matrix (residuals after ambient and technical noise removal)
- CLR-normalized ADT matrix for comparison
- Per-protein correlation table: median dsb/CLR values vs 98th percentile empty-droplet expression (Pearson r and p)
- Cluster-level protein expression table (e.g., TCR-Va7.2 positivity per cluster after WNN clustering)

## How to apply

First, define cells as barcodes in the filtered Cell Ranger output and background drops as the remaining raw barcodes with protein reads, then apply standard scRNA-seq QC on mRNA content, mitochondrial read proportion, and protein library size; remove proteins with maximum raw UMI count <5 across all cells. Run dsb::DSBNormalizeProtein with denoise.counts=TRUE and use.isotype.control=TRUE, supplying the empty-droplet background matrix: step I estimates each antibody's ambient background from empty droplets (log transformation log(cell+10), standardization by background mean and SD), and step II fits a 2-component Gaussian mixture per cell with mclust to extract the background mean µ1, combines fitted µ1 values with isotype controls into a noise matrix, computes PC1 as the technical component (via limma::removeBatchEffect and prcomin), and regresses it out, returning residuals. In parallel, compute CLR normalization (e.g., Seurat NormalizeData on the ADT assay) as a comparator. Validate by correlating per-protein median normalized values with the 98th percentile of that protein's expression in empty droplets: CLR values correlate strongly with ambient noise (r≈0.8, p≈2.4e-7) while dsb values do not (r≈0.13, p≈0.39), and confirm that dsb unmasks populations like TCR-Va7.2+ MAIT cells (cluster 14, GSEA NES≈2.64, p≈1e-10) that CLR misclassifies.

## Related tools

- **dsb** (Core normalization function DSBNormalizeProtein() performing ambient background subtraction and technical-component regression on ADT counts) — https://cran.r-project.org/package=dsb
- **mclust** (Fits the 2-component Gaussian mixture model (EM) per cell to extract the background protein subpopulation mean µ1)
- **limma** (removeBatchEffect() used for robust and efficient matrix decomposition of the noise variable matrix)
- **R** (Implementation language; prcomp() loading vectors multiplied by the noise matrix)
- **Seurat** (CLR normalization comparator and downstream WNN integration (FindMultiModalNeighbors, dims.list = list(1:30, 1:18)) and clustering of dsb vs CLR protein values with 30 mRNA PCs) — https://github.com/satijalab/seurat-data

## Examples

```
dsb_rescaled = DSBNormalizeProtein(cell_protein_matrix = cells_citeseq_mtx, empty_drop_matrix = empty_drop_citeseq_mtx, denoise.counts = TRUE, use.isotype.control = TRUE)
```

## Evaluation signals

- Pearson correlation of per-protein median CLR values with the 98th percentile empty-droplet expression should be high (r≈0.8, p≈2.4e-7), while the same correlation for dsb values should be near zero (r≈0.13, p≈0.39)
- In the TEA-seq case study, cluster 14 should be the only TCR-Va7.2-positive cluster under dsb normalization
- GSEA of cluster 14 transcripts against a MAIT-cell signature should yield NES≈2.64 with p≈1e-10
- Proteins with maximum raw UMI count <5 across all cells should be flagged and removed (e.g., CD34) prior to normalization
- dsb output should be standard R matrices insertable into SingleCellExperiment, Seurat, or AnnData containers

## Limitations

- dsb normalized values are sensitive to how background droplets are defined; defining a reasonable subset of barcodes as background still requires care
- Caveats for using dsb on datasets without isotype control antibody measurements are deferred to the Supplementary Note and CRAN documentation
- Software version numbers are not stated in the article; batch normalization behavior depends on the extent of batch uniformity

## Evidence

- [discussion] Step I: estimate ambient background noise per antibody from empty droplets and rescale cell counts: "In step I, protein counts in empty droplets are used to estimate the expected ambient background noise for each antibody"
- [discussion] Gaussian mixture model per cell via mclust: "dsb fits a Gaussian mixture model through the expectation-maximization algorithm implemented with the mclust51 R package"
- [discussion] Noise matrix and technical component via limma: "dsb uses a function from the limma52 package removeBatchEffect() for robust and efficient matrix decomposition"
- [discussion] Cell vs background drop definition: "Cells were defined as barcodes in the filtered Cell Ranger output, and background drops were defined as after removing the cells from the from the raw Cell Ranger output"
- [discussion] WNN joint clustering with dsb or CLR protein values: "the same 30 mRNA principal components were combined with either dsb or CLR normalized protein data for joint clustering"
- [discussion] Low-signal protein removal filter: "Proteins with very low raw data signal (a maximum UMI count <5 across all cells) were removed prior to normalization, resulting in removal of the CD34 protein from two datasets."
- [discussion] Background droplet definition sensitivity: "defining a reasonable subset of barcodes as background droplets still requires care"
- [readme] dsb is a single function call interfacing with Seurat: [evidence span withheld — anchor status: external_doc]
