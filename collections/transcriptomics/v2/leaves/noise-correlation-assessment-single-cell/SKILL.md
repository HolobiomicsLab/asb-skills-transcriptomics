---
name: noise-correlation-assessment-single-cell
description: 'Use when reproducing or validating the dsb noise model on a droplet
  CITE-seq dataset: you have raw+filtered Cell Ranger ADT matrices, empty droplets,
  and want to confirm the µ1–isotype (Cor 1) and µ1–library-size (Cor 2) correlations
  across single cells.'
when_to_use_negative:
- The dataset has no isotype control antibodies — the article defers the caveats for
  dsb without isotype measurements to the Supplementary Note, so Cor 1 cannot be assessed
- ADT data are already normalized (e.g. CLR-normalized) — µ1 must be fitted on the
  dsb log-transformed, background-standardized values, not pre-normalized matrices
- Only filtered ADT counts are available with no raw matrix — empty droplets needed
  to estimate the ambient background and define µ1 cannot be obtained
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: dsb
  role: DSBNormalizeProtein performs the log(cell+10) transformation, empty-droplet
    µ_N/σ_N standardization, and per-cell k=2 mclust mixture fit yielding µ1 used
    in the correlations
  repo: https://cran.r-project.org/package=dsb
- name: mclust
  role: EM-based 2-component Gaussian mixture fitted to each cell's transformed ADT
    counts to extract the background subpopulation mean µ1
- name: limma
  role: removeBatchEffect() regresses out the PC1 technical component when the full
    denoising branch is run alongside the correlation check
- name: R
  role: prcomp() computes PC1 of the noise matrix (µ1 + isotype controls); correlations
    and significance computed in R
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:c1ea58210e5e706870be592a6af5789881ebe2752aa335cbd3cd1a739c98471f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - dsb
  - limma
  - mclust
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# noise-correlation-assessment-single-cell

## Summary

Validates the two noise-model assumptions underlying dsb ADT normalization by fitting a k=2 Gaussian mixture per cell to extract the background mean µ1, then computing cross-single-cell correlations of µ1 with isotype controls (Cor 1) and with protein library size λ (Cor 2), replicating Fig. 2i of Mulè, Martins & Tsang (2022).

## When to use

Use when reproducing or validating the dsb noise model on a droplet CITE-seq dataset: you have raw+filtered Cell Ranger ADT matrices, empty droplets, and want to confirm the µ1–isotype (Cor 1) and µ1–library-size (Cor 2) correlations across single cells.

## When NOT to use

- The dataset has no isotype control antibodies — the article defers the caveats for dsb without isotype measurements to the Supplementary Note, so Cor 1 cannot be assessed
- ADT data are already normalized (e.g. CLR-normalized) — µ1 must be fitted on the dsb log-transformed, background-standardized values, not pre-normalized matrices
- Only filtered ADT counts are available with no raw matrix — empty droplets needed to estimate the ambient background and define µ1 cannot be obtained

## Inputs

- Raw Cell Ranger ADT (Antibody Capture) UMI count matrix (for empty/background droplets)
- Filtered Cell Ranger ADT UMI count matrix (cell-containing droplets)
- Barcode rank / EmptyDrops classification to identify >50,000 empty droplets
- Isotype control antibody measurements included in the ADT panel
- Per-cell protein library size (λ, ADT UMI counts per cell)

## Outputs

- Per-cell fitted background mean µ1 (cellwise_background_mean) from the k=2 mclust Gaussian mixture
- Correlation summary (Cor 1: µ1 vs isotype; Cor 2: µ1 vs protein library size) with significance
- Optional supporting artifacts: dsb-normalized matrix, cluster assignments, UMAP coordinates, CD14 vs CD4 biaxial plots

## How to apply

Define cells as barcodes in the filtered Cell Ranger output and background droplets as the remaining raw-output barcodes with ADT reads (e.g. >50,000 empty droplets identified from the EmptyDrops/Cell Ranger barcode rank). Run DSBNormalizeProtein with dsb defaults: log(cell + 10) transformation, subtraction of the empty-droplet background mean µ_N and SD σ_N per antibody, then a 2-component Gaussian mixture model (mclust, EM) per cell to extract the fitted background mean µ1. Optionally run the full denoising branch (noise matrix of µ1 + isotype controls, PC1 via prcomp, limma::removeBatchEffect) so clustering outputs can also be checked. Extract the per-cell fitted background mean (cellwise_background_mean) and compute Pearson correlations across single cells: Cor 1 = µ1 vs isotype control levels, Cor 2 = µ1 vs protein library size λ. Both correlations must be positive and statistically significant for the noise-model assumptions to be considered replicated; additionally verify that the k=2 mixture assigns the majority of cells to the lower (background) component.

## Related tools

- **dsb** (DSBNormalizeProtein performs the log(cell+10) transformation, empty-droplet µ_N/σ_N standardization, and per-cell k=2 mclust mixture fit yielding µ1 used in the correlations) — https://cran.r-project.org/package=dsb
- **mclust** (EM-based 2-component Gaussian mixture fitted to each cell's transformed ADT counts to extract the background subpopulation mean µ1)
- **limma** (removeBatchEffect() regresses out the PC1 technical component when the full denoising branch is run alongside the correlation check)
- **R** (prcomp() computes PC1 of the noise matrix (µ1 + isotype controls); correlations and significance computed in R)

## Examples

```
dsb_rescaled = DSBNormalizeProtein(cell_protein_matrix = cells_citeseq_mtx, empty_drop_matrix = empty_drop_citeseq_mtx, denoise.counts = TRUE); cor(dsb_metadata$cellwise_background_mean, isotype_values, method = 'pearson')
```

## Evaluation signals

- Cor 1 (µ1 vs isotype controls) is positive and statistically significant across single cells
- Cor 2 (µ1 vs protein library size λ) is positive and statistically significant across single cells
- The k=2 Gaussian mixture places the majority of cells in the lower (background) component (e.g. 89% of cells in the external 10X NextGem replicate)
- Clustering/UMAP and CD14 vs CD4 biaxial plots on dsb-normalized values reproduce the population structure of Fig. 2a-h
- Background droplets are defined from the raw-minus-filtered barcodes with ADT reads, anchored on >50,000 empty droplets from the barcode rank

## Limitations

- dsb normalized values are sensitive to how background droplets are defined; defining a reasonable subset of barcodes as background requires care
- Caveats for using dsb on datasets without isotype control antibody measurements are deferred to the Supplementary Note
- Assessment details (robustness of noise estimation, empty-droplet definition sensitivity) are in the Supplementary Note, not the main text; software version numbers are not stated

## Evidence

- [abstract] Isotype/background-mean shared variance used to correct noise: "isotype control antibodies and the background protein population average in each cell exhibit significant correlations across single cells, we thus use their shared variance to correct for"
- [discussion] Gaussian mixture construction of µ1: "A noise variable matrix is then constructed by combining all the fitted µ1 values with the isotype control values for all cells. dsb then calculates principal component 1"
- [discussion] mclust EM fit with k=2: "dsb fits a Gaussian mixture model through the expectation-maximization algorithm implemented with the mclust51 R package to the transformed count of each cell from step 1 with k = 2 mixture components"
- [discussion] Empty-droplet definition from raw minus filtered barcodes: "Cells were defined as barcodes in the filtered Cell Ranger output, and background drops were defined as after removing the cells from the from the raw Cell Ranger output"
- [methods] Ambient background estimated from empty droplets: "protein counts in empty droplets are used to estimate the expected ambient background noise for each antibody"
- [readme] README on empty droplets as noise proxy: [evidence span withheld — anchor status: external_doc]
