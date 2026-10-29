---
name: protein-based-clustering-umap-visualization
description: Use when you have dsb-normalized ADT/protein values (or want to validate
  a normalization against Figure 2-style results) and need to confirm the normalized
  data resolve interpretable protein-defined clusters and biaxial population structure.
when_to_use_negative:
- The protein matrix has not been background-corrected/denoised (e.g. raw UMI counts
  or CLR-normalized values), since clustering on noisy ADT data is exactly what dsb
  was shown to outperform
- No empty-droplet background is available and the dsb step I ambient correction cannot
  be performed, unless the no-empty-droplets dsb variant is deliberately used
- The goal is mRNA-only clustering where protein abundance carries no signal
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: Seurat
  role: Runs clustering (RunUMAP, joint WNN clustering combining 30 mRNA PCs with
    normalized protein values) on the dsb matrix
- name: dsb
  role: Produces the denoised, background-scaled protein matrix that serves as the
    clustering input; interfaces with Seurat via standard R matrix objects
  repo: https://cran.r-project.org/package=dsb
- name: mclust
  role: Fits the per-cell 2-component Gaussian mixture whose mu1 feeds the technical
    component removed before clustering
- name: limma
  role: removeBatchEffect() regresses the technical component out of ambient-corrected
    values, yielding the residuals used for clustering
- name: R
  role: Environment in which the whole pipeline (prcomp for PC1, clustering, UMAP)
    is executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:7dd1ccd0210c164f0a344cb48694dc40ffea51126f7d4bb1d74959d879c1fb84
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - dsb
  - limma
  - mclust
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# protein-based-clustering-umap-visualization

## Summary

Cluster cells on dsb-normalized protein (ADT) values and embed with UMAP to verify that denoised, background-corrected protein data unmask biologically meaningful cell populations, e.g. PBMC subsets visible in biaxial plots such as CD14 vs CD4.

## When to use

Use when you have dsb-normalized ADT/protein values (or want to validate a normalization against Figure 2-style results) and need to confirm the normalized data resolve interpretable protein-defined clusters and biaxial population structure.

## When NOT to use

- The protein matrix has not been background-corrected/denoised (e.g. raw UMI counts or CLR-normalized values), since clustering on noisy ADT data is exactly what dsb was shown to outperform
- No empty-droplet background is available and the dsb step I ambient correction cannot be performed, unless the no-empty-droplets dsb variant is deliberately used
- The goal is mRNA-only clustering where protein abundance carries no signal

## Inputs

- dsb-normalized protein expression matrix (residuals from DSBNormalizeProtein, cells x ADT)
- Optionally, mRNA principal components (30 PCs) for joint WNN clustering
- Per-cell fitted background means (cellwise_background_mean / mu1) from the dsb noise model
- ADT library size (lambda) per cell for correlation diagnostics

## Outputs

- Cluster assignments table per cell on dsb-normalized protein values
- UMAP coordinates file
- Biaxial plots (e.g. CD14 vs CD4) on dsb-normalized values
- CSV summary of Cor 1 (mu1 vs isotype controls) and Cor 2 (mu1 vs library size) for comparison with Fig. 2i

## How to apply

After normalizing ADT counts with DSBNormalizeProtein (log(cell+10) transform, background mean/SD subtraction from >50,000 empty droplets, mclust 2-component Gaussian mixture to extract mu1, and limma::removeBatchEffect regression of the technical component built from mu1 plus isotype controls via prcomp PC1), load the dsb matrix into Seurat and cluster cells on protein values only. Embed the clusters with RunUMAP to visualize population structure, and generate biaxial plots (e.g. CD14 vs CD4) on the dsb-normalized values as a direct analog to flow cytometry. Optionally combine 30 mRNA principal components with the normalized protein values for joint WNN clustering when both modalities are available. Correctness is judged by whether clusters correspond to known immune populations and by significant positive correlations of the fitted background mean mu1 (cellwise_background_mean) with isotype controls (Cor 1) and with protein library size lambda (Cor 2), replicating the paper's noise-model assumptions.

## Related tools

- **Seurat** (Runs clustering (RunUMAP, joint WNN clustering combining 30 mRNA PCs with normalized protein values) on the dsb matrix)
- **dsb** (Produces the denoised, background-scaled protein matrix that serves as the clustering input; interfaces with Seurat via standard R matrix objects) — https://cran.r-project.org/package=dsb
- **mclust** (Fits the per-cell 2-component Gaussian mixture whose mu1 feeds the technical component removed before clustering)
- **limma** (removeBatchEffect() regresses the technical component out of ambient-corrected values, yielding the residuals used for clustering)
- **R** (Environment in which the whole pipeline (prcomp for PC1, clustering, UMAP) is executed)

## Examples

```
dsb_rescaled <- DSBNormalizeProtein(cell_protein_matrix = dsb::cells_citeseq_mtx, raw_drop_matrix = dsb::empty_drop_citeseq_mtx); seurat_obj <- RunUMAP(seurat_obj, features = rownames(dsb_rescaled))
```

## Evaluation signals

- Protein-based clusters and UMAP reproduce the known population structure (e.g. PBMC subsets) shown in Fig. 2a-h of the paper
- Biaxial CD14 vs CD4 plots on dsb-normalized values show clean separation of monocyte and T-cell populations, analogous to flow cytometry
- Cor 1 (fitted background mean mu1 vs isotype control levels) is positive and statistically significant across single cells
- Cor 2 (background mean vs protein library size lambda) is positive and statistically significant, matching Fig. 2i
- The 2-component Gaussian mixture captures most cells (e.g. ~89% of cells at k=2) with convergent mclust fits

## Limitations

- dsb normalized values are sensitive to how background droplets are defined; defining a reasonable subset of barcodes requires care
- Caveats for using dsb on datasets without isotype control antibody measurements are deferred to the Supplementary Note; usage guidance for the technical component in such experiments is in the CRAN documentation
- Proteins with very low raw signal (maximum UMI count < 5 across all cells, e.g. CD34 in two datasets) must be removed prior to normalization or they can degrade the analysis

## Evidence

- [abstract] interfaces easily with existing single cell software platforms such as Seurat, Bioconductor, and Scanpy: "interfaces easily with existing single cell software platforms such as Seurat, Bioconductor, and Scanpy"
- [abstract] Compared to existing normalization methods, our approach improves downstream analyses by better unmasking biologically meaningful cell populations: "Compared to existing normalization methods, our approach improves downstream analyses by better unmasking biologically meaningful cell populations"
- [discussion] A noise variable matrix is then constructed by combining all the fitted µ1 values with the isotype control values for all cells. dsb then calculates principal component 1: "A noise variable matrix is then constructed by combining all the fitted µ1 values with the isotype control values for all cells. dsb then calculates principal component 1"
- [discussion] the same 30 mRNA principal components were combined with either dsb or CLR normalized protein data for joint clustering: "the same 30 mRNA principal components were combined with either dsb or CLR normalized protein data for joint clustering"
- [readme] The functions in this package return standard R matrix objects that can be added to any data container like a `SingleCellExperiment`, `Seurat`,: [evidence span withheld — anchor status: external_doc]
