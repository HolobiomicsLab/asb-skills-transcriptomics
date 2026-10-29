---
name: gaussian-mixture-ambient-noise-modeling
description: Use when normalizing droplet-based protein (ADT) counts from CITE-seq/ASAP-seq/TEA-seq
  where ambient unbound-antibody noise and cell-level technical noise must be removed
  before clustering or differential expression.
when_to_use_negative:
- Dataset has no empty droplets and no isotype controls — the caveats for such data
  are deferred to the dsb Supplementary Note and require the alternative no_empty_drops
  workflow
- Proteins have very low raw signal (maximum UMI count <5 across all cells, e.g. CD34
  in two datasets) — these should be removed before normalization rather than modeled
tools:
- name: dsb
  role: Implements the mixture-based denoising via DSBNormalizeProtein()
  repo: https://cran.r-project.org/package=dsb
- name: mclust
  role: Fits the per-cell 2-component Gaussian mixture by expectation-maximization
    to estimate background mean µ1
- name: limma
  role: Provides removeBatchEffect() for robust and efficient matrix decomposition
    when regressing out the technical component
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:51d00fa801d1d2508ed2d228eb9f82f612c7441a4ebf30c07a3c0dea59edcf8e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Seurat
  - dsb
  - limma
  - mclust
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# Gaussian mixture ambient noise modeling

## Summary

Model per-cell ADT noise in droplet-based protein data by fitting a 2-component Gaussian mixture (via mclust EM) to ambient-corrected counts, then building a technical noise matrix from fitted background means plus isotype controls whose PC1 is regressed out to produce denoised, scaled protein values (dsb step II).

## When to use

Use when normalizing droplet-based protein (ADT) counts from CITE-seq/ASAP-seq/TEA-seq where ambient unbound-antibody noise and cell-level technical noise must be removed before clustering or differential expression.

## When NOT to use

- Dataset has no empty droplets and no isotype controls — the caveats for such data are deferred to the dsb Supplementary Note and require the alternative no_empty_drops workflow
- Proteins have very low raw signal (maximum UMI count <5 across all cells, e.g. CD34 in two datasets) — these should be removed before normalization rather than modeled

## Inputs

- Cell-containing droplet ADT count matrix (e.g., dsb::cells_citeseq_mtx)
- Empty-droplet background ADT count matrix (e.g., dsb::empty_drop_citeseq_mtx)
- Isotype control antibody measurements (e.g., rows 67:70)
- Filtered raw UMI matrices from Cell Ranger for defining cells vs. background drops

## Outputs

- Denoised, scaled ADT matrix (dsb residuals) suitable for Seurat/SingleCellExperiment/AnnData
- Per-cell fitted Gaussian mixture background mean µ1 values
- PC1 technical component (noise variable matrix scores)

## How to apply

After step I (log-transform counts with log(cell + 10), subtract per-antibody empty-droplet background mean and divide by background SD), fit a 2-component Gaussian mixture with EM (mclust) to each cell's transformed counts and extract the background mean µ1. Build a noise variable matrix combining all fitted µ1 values with the isotype control antibodies (rows 67:70 in the PBMC dataset) and compute PC1 as the dsb technical component. Regress this technical component out of the ambient-corrected values and return residuals; optionally clip outliers with quantile.clipping=TRUE and quantile.clip=c(0.001, 0.9995). The rationale is that isotype controls and the background protein population average share variance that tracks droplet/cell-specific noise, enabling its removal via limma::removeBatchEffect-style decomposition.

## Related tools

- **dsb** (Implements the mixture-based denoising via DSBNormalizeProtein()) — https://cran.r-project.org/package=dsb
- **mclust** (Fits the per-cell 2-component Gaussian mixture by expectation-maximization to estimate background mean µ1)
- **limma** (Provides removeBatchEffect() for robust and efficient matrix decomposition when regressing out the technical component)

## Examples

```
dsb_rescaled = DSBNormalizeProtein(cell_protein_matrix = cells_citeseq_mtx, empty_drop_matrix = empty_drop_citeseq_mtx, denoise.counts = TRUE, use.isotype.control = TRUE, isotype.control = c(67:70), quantile.clipping = TRUE, quantile.clip = c(0.001, 0.9995))
```

## Evaluation signals

- Each cell's mixture fit should use exactly k=2 components, with the lower component identified as the background subpopulation
- Isotype control values and fitted background means should exhibit significant positive correlations across single cells before PC1 construction
- Downstream Gap-Statistic from k-medoids clustering (k=1..20) should be higher for dsb-normalized data than CLR-normalized data
- Clipped output values must respect the quantile.clip = c(0.001, 0.9995) bounds
- Low-signal proteins (max raw UMI <5 across all cells) should have been removed prior to normalization

## Limitations

- Defining a reasonable subset of barcodes as background (empty) droplets still requires care; dsb normalized values are sensitive to this definition
- In experiments without isotype controls, guidance on the technical component is deferred to the CRAN documentation and Supplementary Note
- Software version numbers for dsb/mclust/limma are not stated in the methods

## Evidence

- [methods] Per-cell 2-component mixture fit via EM: "dsb fits a Gaussian mixture with 2 mixing components to model the ADT levels in each cell"
- [discussion] mclust implements the mixture model: "dsb fits a Gaussian mixture model through the expectation-maximization algorithm implemented with the mclust51 R package"
- [discussion] Noise matrix construction and PC1 technical component: "A noise variable matrix is then constructed by combining all the fitted µ1 values with the isotype control values for all cells."
- [discussion] Residuals after regressing out the technical component: "the dsb technical component for each cell is then regressed out of the ambient-noise-corrected values “Y” from part 1; the values returned by dsb are the residuals"
- [abstract] Shared variance rationale for noise correction: "isotype control antibodies and the background protein population average in each cell exhibit significant correlations across single cells"
