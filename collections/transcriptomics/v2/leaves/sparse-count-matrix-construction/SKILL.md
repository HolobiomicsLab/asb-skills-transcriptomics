---
name: sparse-count-matrix-construction
description: Use when processing 10x molecule information files from libraries multiplexed
  on patterned flow-cell instruments (HiSeq 4000/X/NovaSeq) and you need barcode-swap-corrected
  per-sample sparse gene-by-cell count matrices.
when_to_use_negative:
- Data come from non-patterned-flowcell instruments (e.g. HiSeq 2500), where the paper's
  control showed barcode swapping is not the dominant artefact and cell exclusion
  of shared barcodes would unnecessarily discard cell libraries
- Input is a plate-based scRNA-seq count matrix (e.g. Smart-seq2 plates); the molecule-exclusion
  approach is specific to 10x droplet molecule information files
- Input is already a cleaned or count-matrix feature table rather than raw molecule-level
  information, since the key (UMI, cell barcode, aligned gene) grouping requires molecule-level
  records
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DropletUtils
  role: R package implementing swappedDrops (molecule-exclusion) and emptyDrops (cell
    calling) on 10x molecule information files to produce cleaned sparse count matrices
  repo: https://github.com/MarioniLab/DropletUtils
- name: R
  role: Analysis language in which the sparse matrices are returned and saved (e.g.
    to cleaned_count_matrices.rds)
- name: 10x Genomics
  role: Source of the molecule information files (UMI, cell barcode, gene) consumed
    by the algorithm
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:12a1bb747e6da1d6acdfc0fe230278dec101aee1b0947176384a9174acc0d017
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarioniLab/DropletUtils
  tools:
  - 10x Genomics
  - DropletUtils
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# sparse-count-matrix-construction

## Summary

Constructs cleaned per-sample sparse count matrices for multiplexed 10x Genomics droplet scRNA-seq by assigning each molecule (UMI / cell barcode / aligned gene combination) to its sample-of-origin and excluding swapped molecules, as implemented in DropletUtils' swappedDrops.

## When to use

Use when processing 10x molecule information files from libraries multiplexed on patterned flow-cell instruments (HiSeq 4000/X/NovaSeq) and you need barcode-swap-corrected per-sample sparse gene-by-cell count matrices.

## When NOT to use

- Data come from non-patterned-flowcell instruments (e.g. HiSeq 2500), where the paper's control showed barcode swapping is not the dominant artefact and cell exclusion of shared barcodes would unnecessarily discard cell libraries
- Input is a plate-based scRNA-seq count matrix (e.g. Smart-seq2 plates); the molecule-exclusion approach is specific to 10x droplet molecule information files
- Input is already a cleaned or count-matrix feature table rather than raw molecule-level information, since the key (UMI, cell barcode, aligned gene) grouping requires molecule-level records

## Inputs

- Per-sample 10x molecule information files (UMI, cell barcode, aligned gene per molecule) from multiplexed droplet libraries
- min.frac threshold parameter (default 0.9; 0.8 used in the paper)
- emptyDrops parameters: FDR threshold 0.01, minimum cell size 1000 UMIs

## Outputs

- Cleaned per-sample sparse count matrices (gene x cell)
- cleaned_count_matrices.rds file with assigned/reassigned molecule counts

## How to apply

Load the 10x molecule information (UMI, cell barcode, aligned gene) for each multiplexed droplet sample. Group molecules across samples sharing the identical UMI / cell-barcode / aligned-gene key to flag candidate swapping events. For each ambiguous combination, compute the fraction of reads contributed by each sample; if a single sample holds at least min.frac (DropletUtils default 0.9; the paper used 0.8), reassign the molecule to that sample-of-origin, otherwise remove it from all samples. Call cells first with emptyDrops (FDR 0.01, minimum 1000 UMIs) so downstream matrices retain only cell-associated barcodes. Return and save the cleaned per-sample sparse count matrices (e.g. cleaned_count_matrices.rds) for subsequent scran/Rtsne analyses.

## Related tools

- **DropletUtils** (R package implementing swappedDrops (molecule-exclusion) and emptyDrops (cell calling) on 10x molecule information files to produce cleaned sparse count matrices) — https://github.com/MarioniLab/DropletUtils
- **R** (Analysis language in which the sparse matrices are returned and saved (e.g. to cleaned_count_matrices.rds))
- **10x Genomics** (Source of the molecule information files (UMI, cell barcode, gene) consumed by the algorithm)

## Examples

```
library(DropletUtils); cleaned <- swappedDrops(mol.info.files, min.frac=0.8); saveRDS(cleaned$cleaned, "cleaned_count_matrices.rds")
```

## Evaluation signals

- Every molecule key (UMI, cell barcode, aligned gene) appearing in multiple samples is either reassigned to a single sample holding >= min.frac of its reads or removed from all samples — no ambiguous molecule persists in any output matrix
- Reads lost by the molecule-exclusion step approximate the expected swapped fraction (~2.5% of reads on HiSeq 4000), not the large fraction lost by cell exclusion of shared barcodes
- Each output matrix is a sparse gene x cell matrix with cell barcodes restricted to emptyDrops-called cells (FDR < 0.01, >= 1000 UMIs)
- Same libraries sequenced on HiSeq 2500 yield a swapping fraction near zero, serving as a negative control on the pipeline
- Per-sample matrix dimensions and per-cell totals can be reconciled against the molecule information file after reassignment/exclusion

## Limitations

- Molecules whose reads cannot be confidently attributed to any sample meeting min.frac are discarded entirely, so real signal in heavily swapped combinations is lost
- The paper reports ~2.5% of reads mislabelled on the HiSeq 4000 — the correction only applies to patterned flow-cell instruments and HiSeq 2500 controls showed minimal swapping
- Molecule exclusion method details were deferred to Supplementary Note 6, and the electronic supplementary material is unavailable ('No changelog found.'), limiting reproducibility of parameter choices beyond the stated min.frac values

## Evidence

- [discussion] Molecule exclusion: removal of molecules sharing the same UMI, cell barcode, and aligned gene combination: [evidence span withheld — anchor status: not_found]
- [discussion] emptyDrops cell calling parameters: "calling cells using emptyDrops18 using an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs"
- [abstract] Algorithm goal: "an algorithm to exclude individual molecules that have swapped between samples in 10x Genomics experiments"
- [readme] DropletUtils capabilities: "This includes data loading from count matrices or molecule information files, identification of cells from empty droplets, removal of barcode-swapped pseudo-cells, and downsampling of the count"
- [discussion] Read mapping and counting step: "Reads were mapped to the mm10 genome, and the number of read pairs mapping to the exonic region of each gene were counted."
