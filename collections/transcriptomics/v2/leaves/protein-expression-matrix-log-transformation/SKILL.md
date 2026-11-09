---
name: protein-expression-matrix-log-transformation
description: Use when you have raw cell and empty-droplet ADT UMI matrices (e.g.,
  Cell Ranger filtered/raw barcodes from CITE-seq, ASAP-seq, TEA-seq) and need the
  log(pseudocount=10) transform required before dsb step I ambient standardization.
when_to_use_negative:
- The ADT matrix is already dsb-normalized, CLR-normalized, or otherwise log/scaled
  transformed — re-transforming would double-apply the pseudocount
- The input is mRNA/RNA UMI data rather than antibody-derived-tag protein counts;
  dsb normalization is designed for ADT noise structure (ambient antibody + cell-intrinsic
  technical noise)
- No background/empty droplets exist and you need a plug-in transform without considering
  dsb's alternative no-empty-drops workflow
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: dsb
  role: Provides example cell and empty-drop CITE-seq matrices (cells_citeseq_mtx,
    empty_drop_citeseq_mtx) and DSBNormalizeProtein() that consumes the log-transformed
    values
  repo: https://cran.r-project.org/package=dsb
- name: R
  role: Environment in which the log(cell + 10) transform and the subsequent prcomp()/limma
    steps are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:5e82a4c43ec36985dc471f4129a4c438183fae92ef4632421676f8f461329cca
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - dsb
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# protein-expression-matrix-log-transformation

## Summary

Log-transforms raw ADT/protein UMI count matrices from droplet-based CITE-seq data with pseudocount 10, the prerequisite step for dsb ambient-noise standardization, which subsequently rescales cells by per-protein empty-droplet mean (mu_N) and SD (sigma_N).

## When to use

Use when you have raw cell and empty-droplet ADT UMI matrices (e.g., Cell Ranger filtered/raw barcodes from CITE-seq, ASAP-seq, TEA-seq) and need the log(pseudocount=10) transform required before dsb step I ambient standardization.

## When NOT to use

- The ADT matrix is already dsb-normalized, CLR-normalized, or otherwise log/scaled transformed — re-transforming would double-apply the pseudocount
- The input is mRNA/RNA UMI data rather than antibody-derived-tag protein counts; dsb normalization is designed for ADT noise structure (ambient antibody + cell-intrinsic technical noise)
- No background/empty droplets exist and you need a plug-in transform without considering dsb's alternative no-empty-drops workflow

## Inputs

- Raw ADT/protein UMI count matrix for cells (filtered Cell Ranger barcodes)
- Raw ADT/protein UMI count matrix for empty/background droplets (raw barcodes minus cells)
- Optional isotype control antibody rows (e.g., Mouse IgG2bk, MouseIgG1kappa, MouseIgG2akappa, RatIgG2bk)

## Outputs

- Log-transformed cell ADT matrix, dlog = log(cell + 10)
- Log-transformed empty-droplet ADT matrix for estimating per-protein background mu_N and sigma_N
- Downstream: ambient-corrected, then denoised ADT matrix from DSBNormalizeProtein()

## How to apply

Load the cell protein matrix and empty-droplet (background) protein matrix, e.g., dsb::cells_citeseq_mtx and dsb::empty_drop_citeseq_mtx, and apply the log transform with pseudocount 10 to both: dlog = log(cell + 10). The pseudocount 10 is dsb's default to stabilize low ADT counts before background subtraction. The log-transformed empty-droplet matrix supplies the per-protein ambient mean mu_N and SD sigma_N used to standardize each cell's log counts as (x - mu_N) / sigma_N in dsb step I. Cells are defined as barcodes in the filtered Cell Ranger output; background drops are the remaining raw barcodes with protein reads. Downstream (step II), the ambient-corrected values feed a 2-component Gaussian mixture per cell (mclust) to extract background mean mu1, which combined with isotype controls yields the PC1 technical component regressed out via limma::removeBatchEffect.

## Related tools

- **dsb** (Provides example cell and empty-drop CITE-seq matrices (cells_citeseq_mtx, empty_drop_citeseq_mtx) and DSBNormalizeProtein() that consumes the log-transformed values) — https://cran.r-project.org/package=dsb
- **R** (Environment in which the log(cell + 10) transform and the subsequent prcomp()/limma steps are executed)

## Examples

```
dlog = log(cell + 10); neglog = log(neg + 10)
```

## Evaluation signals

- Transformed values equal log(raw count + 10) — spot-check a few entries against the raw UMI matrix
- Step-I standardized values reproduce (x - mu_N) / sigma_N where mu_N and sigma_N come from log-transformed empty droplets
- Full pipeline reproduction check: all.equal(denoised_adt, denoised_adt_3) matches DSBNormalizeProtein() output
- Final denoised matrix rows retain the original antibody (ADT) identifiers and columns retain cell barcodes

## Limitations

- dsb normalized values are sensitive to how background droplets are defined; defining a reasonable subset of barcodes as background requires care
- Handling datasets without isotype control antibody measurements (needed for the step-II technical component) requires the Supplementary Note / CRAN documentation caveats
- A max raw UMI count < 5 across all cells is a signal-removal threshold (e.g., CD34 removed from two datasets) that may affect which proteins are carried through

## Evidence

- [methods] log transform with pseudocount 10 and step-I standardization: "# log transformation
dlog = log(cell + 10)"
- [methods] background mean and SD standardization: "Subtract background mean $\mu_{N}$ and divide by the background standard deviation $\sigma_{N}$"
- [discussion] step I ambient noise estimation from empty droplets: "In step I, protein counts in empty droplets are used to estimate the expected ambient background noise for each antibody"
- [readme] dsb available on CRAN for installing and reproducing the transform: "The dsb R package is available on [**CRAN: latest dsb
release**](https://CRAN.R-project.org/package=dsb)  
To install in R use `install.packages('dsb')`"
