---
name: background-droplet-standardization
description: Use when raw CITE-seq/ASAP-seq/TEA-seq ADT count matrices are available
  together with raw (unfiltered) barcode matrices containing empty droplets, and ambient
  protein-specific noise must be removed prior to cell-level normalization.
when_to_use_negative:
- No empty droplets are available (raw/unfiltered matrix missing) — the dsb vignette
  'no_empty_drops' workflow or an alternative background definition must be used instead
- The input is already normalized protein data (e.g., CLR-normalized or dsb step-II
  output) rather than raw ADT UMI counts
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: dsb
  role: R package providing the empty-droplet matrices and DSBNormalizeProtein() that
    implements step I (log(x+10) transform, background mean/variance standardization)
  repo: https://cran.r-project.org/package=dsb
- name: R
  role: Runtime environment in which the dsb package and matrix standardization are
    executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:339839e683a913ec32801fd628b35f2e107ce67df971b8f666d74f91dc5eb8f2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - dsb
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# background-droplet-standardization

## Summary

DSB step I: standardize raw ADT (protein) counts in droplet-based single-cell data by log-transforming counts with log(x+10) and then rescaling each protein per cell using the mean (µ_N) and standard deviation (σ_N) of that protein measured in empty (background) droplets. This removes protein-specific ambient noise originating from unbound antibodies encapsulated during droplet generation before any cell-level denoising.

## When to use

Use when raw CITE-seq/ASAP-seq/TEA-seq ADT count matrices are available together with raw (unfiltered) barcode matrices containing empty droplets, and ambient protein-specific noise must be removed prior to cell-level normalization.

## When NOT to use

- No empty droplets are available (raw/unfiltered matrix missing) — the dsb vignette 'no_empty_drops' workflow or an alternative background definition must be used instead
- The input is already normalized protein data (e.g., CLR-normalized or dsb step-II output) rather than raw ADT UMI counts

## Inputs

- Cell-containing ADT/protein count matrix (e.g., filtered Cell Ranger barcodes, dsb::cells_citeseq_mtx)
- Background/empty-droplet ADT count matrix (barcodes from the raw Cell Ranger output after removing cells, dsb::empty_drop_citeseq_mtx)

## Outputs

- Ambient-noise-corrected, rescaled ADT matrix (log-transformed values standardized by background mean µ_N and standard deviation σ_N)
- Per-protein background statistics µ_N and σ_N estimated from empty droplets

## How to apply

Load the cell protein matrix (e.g., dsb::cells_citeseq_mtx) and a background droplet matrix (e.g., dsb::empty_drop_citeseq_mtx derived from the raw Cell Ranger output after removing cell barcodes). Apply the log transformation dlog = log(cell + 10) to both matrices. For each protein, subtract the empty-droplet mean µ_N from the cell values and divide by the empty-droplet standard deviation σ_N; this can be done directly or via DSBNormalizeProtein with denoise.counts = FALSE to keep only step I. The output is an ambient-noise-corrected matrix 'Y' that serves as input to step II (Gaussian mixture fit of µ1, isotype-based technical component via PC1, and residual regression).

## Related tools

- **dsb** (R package providing the empty-droplet matrices and DSBNormalizeProtein() that implements step I (log(x+10) transform, background mean/variance standardization)) — https://cran.r-project.org/package=dsb
- **R** (Runtime environment in which the dsb package and matrix standardization are executed)

## Examples

```
dsb_rescaled = DSBNormalizeProtein(cell_protein_matrix = cells_citeseq_mtx, empty_drop_matrix = empty_drop_citeseq_mtx, denoise.counts = FALSE)
```

## Evaluation signals

- Per-protein µ_N and σ_N computed from the background droplet matrix must be finite and non-degenerate (σ_N > 0) for every retained protein
- Proteins with maximum raw UMI count < 5 across all cells should be removed prior to normalization (e.g., CD34 in two datasets of the paper)
- Step-I output values should be centered near 0 per protein and on a standardized scale comparable across proteins
- Downstream, fitting a k=2 Gaussian mixture (mclust, G=2) to each cell's step-I values should yield a background mean µ1 that correlates with isotype controls (r ≈ 0.59 vs mean isotype), confirming ambient correction did not erase cell-intrinsic technical noise
- Setting denoise.counts = FALSE in DSBNormalizeProtein should return values matching a manual log(x+10) + background-standardization implementation

## Limitations

- dsb normalized values are sensitive to how background droplets are defined; 'defining a reasonable subset of barcodes as background droplets still requires care'
- In batch correction settings, 'in general this could be dependent on the extent of uniformity among the batches'
- Caveats for using dsb on datasets without isotype control antibody measurements are deferred to the Supplementary Note

## Evidence

- [discussion] Step I: estimate ambient background noise per antibody from empty droplets and rescale cell counts: "In step I, protein counts in empty droplets are used to estimate the expected ambient background noise for each antibody"
- [methods] log transformation of ADT counts: "# log transformation
dlog = log(cell + 10)"
- [methods] standardization by background mean and standard deviation: "Subtract background mean $\mu_{N}$ and divide by the background standard deviation $\sigma_{N}$"
- [discussion] definition of cells vs background droplets: "Cells were defined as barcodes in the filtered Cell Ranger output, and background drops were defined as after removing the cells from the from the raw Cell Ranger output"
- [readme] README tool entry point: [evidence span withheld — anchor status: external_doc]
