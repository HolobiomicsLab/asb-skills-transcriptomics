---
name: empty-droplet-noise-correlation-assessment
description: Use when raw and filtered Cell Ranger ADT matrices are available, empty
  droplets can be defined from the raw barcodes, and you must decide whether CLR normalization
  of protein data is confounded by ambient noise (e.g. misinterpreting MAIT-like clusters).
when_to_use_negative:
- Dataset has no raw matrix with empty droplets available — use dsb's 'no empty drops'
  workflow instead of an ambient-correlation assessment
- Data is mRNA-only or has no ADT/protein assay, since ambient antibody noise is undefined
- Proteins have already been filtered with maximum raw UMI count < 5 across all cells
  removed and only high-signal antibodies remain — still applicable, but do not run
  the diagnostic on removed low-signal proteins
tools:
- name: dsb
  role: Provides the denoised normalization (DSBNormalizeProtein with empty-drop background
    matrix) whose residual values are compared against ambient levels, and estimates
    ambient background in step I from empty droplets
  repo: https://cran.r-project.org/package=dsb
- name: Seurat
  role: Provides the CLR comparison normalization (NormalizeData on the ADT assay)
    and the WNN joint clustering (FindMultiModalNeighbors, dims.list = list(1:30,
    1:18)) used to contextualize the correlation finding
- name: mclust
  role: Fits the per-cell two-component Gaussian mixture model underlying dsb's background
    mean estimation, which removes the droplet-level noise the diagnostic tests for
- name: R
  role: Runtime environment; dsb is an R package and Pearson correlations with 98th-percentile
    ambient quantiles are computed in R
  repo: https://cran.r-project.org/package=dsb
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:00fda8c701ffe30f93c9cd0e2dbfb3088181a89a7e7e2b144f4c63ee11e2b777
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - dsb
  - mclust
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# empty-droplet-noise-correlation-assessment

## Summary

Diagnostic for ambient antibody noise in droplet ADT data (CITE-seq/TEA-seq/ASAP-seq): correlate per-protein median normalized values (CLR vs. dsb) with the 98th percentile of each protein's expression in empty droplets to quantify how much a normalization is driven by ambient, unbound antibody.

## When to use

Use when raw and filtered Cell Ranger ADT matrices are available, empty droplets can be defined from the raw barcodes, and you must decide whether CLR normalization of protein data is confounded by ambient noise (e.g. misinterpreting MAIT-like clusters).

## When NOT to use

- Dataset has no raw matrix with empty droplets available — use dsb's 'no empty drops' workflow instead of an ambient-correlation assessment
- Data is mRNA-only or has no ADT/protein assay, since ambient antibody noise is undefined
- Proteins have already been filtered with maximum raw UMI count < 5 across all cells removed and only high-signal antibodies remain — still applicable, but do not run the diagnostic on removed low-signal proteins

## Inputs

- Raw Cell Ranger ADT (protein UMI) matrix including empty droplets
- Filtered Cell Ranger ADT matrix (cell-containing barcodes)
- Optional: mRNA matrix for joint WNN clustering context (Seurat object)
- Isotype control antibody measurements (for dsb denoise.counts=TRUE)

## Outputs

- Per-protein 98th-percentile empty-droplet expression table
- Per-method Pearson correlation table (median CLR vs. ambient; median dsb vs. ambient, with r and p-values)
- Normalized ADT value tables (dsb and CLR) supporting downstream cluster inspection

## How to apply

Define cells as barcodes in the filtered Cell Ranger output and background drops as the remaining raw barcodes with protein reads. Normalize the ADT matrix two ways in parallel: dsb::DSBNormalizeProtein with denoise.counts=TRUE, use.isotype.control=TRUE and the empty-drop background matrix, and CLR (e.g. Seurat NormalizeData on the ADT assay). For each normalization, for each of the ~45 proteins, correlate the median normalized value across cells against the 98th percentile of that protein's counts in the empty droplets. Rationale: dsb's step I removes ambient background estimated from empty droplets, so residual dsb values should be largely independent of ambient levels, whereas CLR retains them. A high ambient correlation for one method (CLR r≈0.8) and near-zero for the other (dsb r≈0.13) demonstrates that ambient noise drove the first method's values.

## Related tools

- **dsb** (Provides the denoised normalization (DSBNormalizeProtein with empty-drop background matrix) whose residual values are compared against ambient levels, and estimates ambient background in step I from empty droplets) — https://cran.r-project.org/package=dsb
- **Seurat** (Provides the CLR comparison normalization (NormalizeData on the ADT assay) and the WNN joint clustering (FindMultiModalNeighbors, dims.list = list(1:30, 1:18)) used to contextualize the correlation finding)
- **mclust** (Fits the per-cell two-component Gaussian mixture model underlying dsb's background mean estimation, which removes the droplet-level noise the diagnostic tests for)
- **R** (Runtime environment; dsb is an R package and Pearson correlations with 98th-percentile ambient quantiles are computed in R) — https://cran.r-project.org/package=dsb

## Examples

```
dsb_rescaled <- DSBNormalizeProtein(cell_protein_matrix = cells_citeseq_mtx, empty_drop_matrix = empty_drop_citeseq_mtx, denoise.counts = TRUE, use.isotype.control = TRUE); r_clr <- cor(median_clr, sapply(rownames(cells_citeseq_mtx), function(p) quantile(empty_drop_citeseq_mtx[p, ], 0.98)))
```

## Evaluation signals

- Median CLR values correlated with the 98th percentile empty-droplet expression give r≈0.8 (p≈2.4e-7) in the TEA-seq case study
- Median dsb values correlated with the same ambient quantile give r≈0.13 (p≈0.39), i.e. near zero
- Under dsb, the cluster of interest (cluster 14) is the only TCR-Va7.2-positive cluster, whereas under CLR it is misread as conventional T cells
- GSEA of the dsb-identified cluster against a MAIT-cell signature yields NES≈2.64 with p≈1e-10, confirming the biological identity
- Pearson correlation is computed per protein across the full antibody panel (~45 proteins) for both methods before comparison

## Limitations

- dsb normalized values are sensitive to how background droplets are defined; 'defining a reasonable subset of barcodes as background droplets still requires care'
- Assessment details and caveats (robustness of noise estimation, empty droplet definition sensitivity, batch effects, isotype control caveats) are deferred to the Supplementary Note, not the main text
- Software version numbers for the tools are not stated in the paper, complicating exact reproduction

## Evidence

- [discussion] Ambient background estimated from empty droplets is the noise dsb step I corrects: "In step I, protein counts in empty droplets are used to estimate the expected ambient background noise for each antibody"
- [discussion] Cell/background definition for ambient estimates: "Cells were defined as barcodes in the filtered Cell Ranger output"
- [discussion] CLR and dsb normalized protein data feed the same joint clustering: "the same 30 mRNA principal components were combined with either dsb or CLR normalized protein data for joint clustering"
- [readme] Ambient levels in empty droplets closely match observed background in cells: [evidence span withheld — anchor status: external_doc]
- [methods] Concrete DSBNormalizeProtein invocation with empty-drop background: "dsb_rescaled = DSBNormalizeProtein(cell_protein_matrix = cells_citeseq_mtx"
