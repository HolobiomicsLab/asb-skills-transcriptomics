---
name: hdf5-single-cell-matrix-export
description: Use when running scTE on 10x-style or plate-based scRNA-seq BAM files
  and you need the resulting TE count matrix in anndata-compatible .h5ad format for
  cell-type-level TE aggregation and marker-TE identification.
when_to_use_negative:
- You need only bulk (non-single-cell) TE counts from a BAM without cell-level resolution
  — a simpler counter such as te_count from te_counter suffices
- The downstream pipeline requires plain-text CSV output rather than HDF5/anndata
  format
- Input is single-cell ATAC-seq — use the scTEATAC/scTEATAC_build pipeline instead
  of the scRNA-seq scTE workflow
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scTE
  role: Quantifies TE expression per cell from BAM input and writes the TE-by-cell
    matrix as out.h5ad when --hdf5 True is set
  repo: https://github.com/JiekaiLab/scTE
- name: scTE_build
  role: Builds the genome index (e.g., mm10.exclusive.idx) from TE BED and gene GTF
    annotations that scTE requires as -x input
  repo: https://github.com/JiekaiLab/scTE
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-21808-x
    title: Identifying transposable element expression dynamics and heterogeneity
      during development at the single-cell level with a processing pipeline scTE
  source_content_hash: sha256:3a9e3e21e1b6e709f95600825d175f030d8443fde6eaafc0dce0f75141caa548
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiekaiLab/scTE
  tools:
  - Cell Ranger
  - STARsolo
  - scTE
  - scTE_build
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# hdf5-single-cell-matrix-export

## Summary

Emit scTE's transposable-element-by-cell count matrix as an HDF5 (.h5ad) file instead of CSV by passing --hdf5 True, enabling direct anndata-based loading and downstream aggregation of TE families (e.g., IAP/RLTR45, MER46C, L1ME3D, ETnERV3-int) per cell type.

## When to use

Use when running scTE on 10x-style or plate-based scRNA-seq BAM files and you need the resulting TE count matrix in anndata-compatible .h5ad format for cell-type-level TE aggregation and marker-TE identification.

## When NOT to use

- You need only bulk (non-single-cell) TE counts from a BAM without cell-level resolution — a simpler counter such as te_count from te_counter suffices
- The downstream pipeline requires plain-text CSV output rather than HDF5/anndata format
- Input is single-cell ATAC-seq — use the scTEATAC/scTEATAC_build pipeline instead of the scRNA-seq scTE workflow

## Inputs

- Unfiltered BAM/SAM alignment file with cell barcode/UMI in CB:Z/UB:Z (Cell Ranger) or CR:Z/UR:Z (STARsolo) tags
- Pre-built scTE genome index (e.g., mm10.exclusive.idx from scTE_build with -te and -gene options)

## Outputs

- TE-by-cell count matrix in .h5ad (HDF5/anndata) format (out.h5ad)
- Optionally the same matrix as CSV when --hdf5 is omitted

## How to apply

Run scTE on the unfiltered BAM/SAM alignment file with --hdf5 True, specifying the cell barcode and UMI tags matching the aligner output (-CB CB -UMI UB for Cell Ranger BAMs, -CB CR -UMI UR for STARsolo BAMs; for Smart-seq2/C1 datasets without integrated tags, set -CB False -UMI False so cell identities come from BAM file names). scTE allocates reads first to gene exons and then to TEs using the pre-built index (e.g., mm10.exclusive.idx from scTE_build -te TEs.bed -gene Genes.gtf -o mm10), and with --hdf5 True writes out.h5ad rather than CSV. The rationale is that the h5ad output preserves the TE-by-cell sparse matrix in a scanpy/anndata-readable container, which supports downstream steps such as aggregating TE family counts (IAP/RLTR45, MER46C, RLTR20B3, LTRIS2, L1ME3D, ETnERV3-int) by gastrulation cell-type annotations or identifying ~130 cell-type-specific TEs across Tabula Muris organs. Memory is ~10 Gb per thread for mouse, so scale threads to available RAM.

## Related tools

- **scTE** (Quantifies TE expression per cell from BAM input and writes the TE-by-cell matrix as out.h5ad when --hdf5 True is set) — https://github.com/JiekaiLab/scTE
- **scTE_build** (Builds the genome index (e.g., mm10.exclusive.idx) from TE BED and gene GTF annotations that scTE requires as -x input) — https://github.com/JiekaiLab/scTE

## Examples

```
scTE -i inp.bam -o out -x mm10.exclusive.idx --hdf5 True -CB CB -UMI UB
```

## Evaluation signals

- Output file exists and loads as valid .h5ad via scanpy/anndata (h5py-readable, sparse TE-by-cell matrix).
- Barcode/UMI tags were honored: cells and UMIs are parsed from the BAM tag columns (CB/UB or CR/UR) rather than producing empty cell identities.
- Known cell-type/TE associations are recoverable from the h5ad matrix, e.g., IAP/RLTR45 in extraembryonic ectoderm, MER46C/RLTR20B3/LTRIS2 in Apoa2+ extraembryonic endoderm, L1ME3D in Tnnt2+ cardiomyocytes, and ~130 TE–cell-type-specific associations in Tabula Muris (L1 family members in oligodendrocytes/microglia, LTR58 in B cells, MER110 in type B pancreatic cells, RLTR46 in hepatocytes).
- Each expected marker TE shows cell-type-specific signal relative to other cell types in the time course after aggregating family counts by cell-type annotations.

## Limitations

- Requires the unfiltered BAM as input; filtered BAMs will drop cells/reads that scTE would otherwise recover.
- Memory footprint is substantial — roughly 10 Gb per thread for mouse — limiting thread scaling.
- For plate-based datasets without CB/UB tags, cell identity falls back to BAM file names (-CB False -UMI False), so barcodes cannot be error-corrected.

## Evidence

- [methods] hdf5 output command: "$ scTE -i inp.bam -o out -x mm10.exclusive.idx --hdf5 True -CB CR -UMI UR"
- [readme] unfiltered BAM recommendation: "scTE makes BAM/SAM file as input, highly recommend to use unfiltered alignment file as input."
- [methods] exclusive read allocation mode: "The scTE algorithm will allocate 
reads first to gene exons, and then to TEs by default."
