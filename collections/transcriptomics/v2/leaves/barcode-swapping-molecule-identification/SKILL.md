---
name: barcode-swapping-molecule-identification
description: Use when multiplexed 10x Genomics droplet libraries were sequenced on
  patterned flow-cell Illumina machines (HiSeq 4000/HiSeq X/NovaSeq) and swapped molecules
  must be excluded from per-sample count matrices.
when_to_use_negative:
- Data generated on non-patterned flow-cell instruments (e.g. HiSeq 2500) where swapping
  is minimal — the paper used HiSeq 2500 sequencing as a negative control
- Plate-based (Smart-seq2) scRNA-seq data, where the paper instead estimates swapped-read
  fractions by linear regression rather than molecule exclusion
- When only aggregate swapped-read fractions are needed rather than molecule-level
  decontamination
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: DropletUtils
  role: R/Bioconductor package implementing molecule-level removal of barcode-swapped
    pseudo-cells and loading of 10x molecule information files
  repo: https://github.com/MarioniLab/DropletUtils
- name: R
  role: Language used for the swapping analyses and downstream scran/Rtsne processing
- name: 10x Genomics
  role: Droplet protocol whose molecule information files provide the UMI/cell-barcode/gene
    keys used to identify swapped molecules
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:dc237b294f1cd04c724e0480cebc9168c08000f4d764eeaedc170ace5cdede31
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarioniLab/DropletUtils
  tools:
  - 10x Genomics
  - DropletUtils
  - R
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# barcode-swapping-molecule-identification

## Summary

Identifies and excludes individual barcode-swapped molecules in multiplexed 10x Genomics droplet scRNA-seq data by grouping molecules with identical UMI, cell barcode, and aligned gene combinations across samples, and reassigning or removing them based on a min.frac read-fraction threshold.

## When to use

Use when multiplexed 10x Genomics droplet libraries were sequenced on patterned flow-cell Illumina machines (HiSeq 4000/HiSeq X/NovaSeq) and swapped molecules must be excluded from per-sample count matrices.

## When NOT to use

- Data generated on non-patterned flow-cell instruments (e.g. HiSeq 2500) where swapping is minimal — the paper used HiSeq 2500 sequencing as a negative control
- Plate-based (Smart-seq2) scRNA-seq data, where the paper instead estimates swapped-read fractions by linear regression rather than molecule exclusion
- When only aggregate swapped-read fractions are needed rather than molecule-level decontamination

## Inputs

- 10x Genomics molecule information files (UMI, cell barcode, aligned gene) for each multiplexed droplet sample
- Per-sample BAM/count data from multiplexed droplet scRNA-seq libraries sequenced on HiSeq 4000 or HiSeq 2500
- min.frac threshold (default 0.9; paper used 0.8)

## Outputs

- Cleaned per-sample sparse count matrices with swapped molecules reassigned or removed (e.g. cleaned_count_matrices.rds)
- List of excluded molecules whose reads could not be confidently attributed to a sample

## How to apply

Load the 10x molecule information files (UMI, cell barcode, gene) for each multiplexed droplet sample. Group molecules across samples by identical UMI/cell-barcode/aligned-gene keys to find candidate swapping events; for each ambiguous combination, compute the fraction of reads contributed by each sample. If a single sample holds at least the min.frac threshold (default 0.9; the paper used 0.8, and roughly 2.5% of HiSeq 4000 reads were mislabelled), reassign the molecule to that sample-of-origin and remove its count from all other samples; molecules with no sample meeting min.frac are removed from every sample. This molecule-level exclusion is preferred over removing all shared cell barcodes, which incorrectly discards entire cell libraries. Cleaned per-sample sparse count matrices (e.g. saved to cleaned_count_matrices.rds) can be produced alongside cell calling with emptyDrops at FDR 0.01 and a minimum cell size of 1000 UMIs.

## Related tools

- **DropletUtils** (R/Bioconductor package implementing molecule-level removal of barcode-swapped pseudo-cells and loading of 10x molecule information files) — https://github.com/MarioniLab/DropletUtils
- **R** (Language used for the swapping analyses and downstream scran/Rtsne processing)
- **10x Genomics** (Droplet protocol whose molecule information files provide the UMI/cell-barcode/gene keys used to identify swapped molecules)

## Examples

```
library(DropletUtils); clean <- swappedDrops(mol.info.files = c("sample1.molecule_info.h5", "sample2.molecule_info.h5"), min.frac = 0.9)
```

## Evaluation signals

- Reassignment rate: only molecules where one sample holds at least min.frac (0.9 default, 0.8 in paper) of reads are reassigned; no molecule is counted in more than one sample after cleaning
- Compare swapped-read fraction against the expected ~2.5% mislabelling rate on HiSeq 4000
- Molecule-level exclusion should retain far more cell libraries than cell exclusion, which removes all barcodes shared between samples
- Sanity check on HiSeq 2500-sequenced libraries, where swapping should be negligible (control in the paper)
- Validate that all cells pass emptyDrops with FDR < 0.01 and minimum cell size of 1000 UMIs before/after cleaning as expected

## Limitations

- Molecules with no sample meeting min.frac are discarded entirely from every sample, so genuinely ambiguous molecules are lost
- Molecule exclusion method details are deferred to Supplementary Note 6, which is unavailable in the provided material ('No changelog found.')
- Cell exclusion (removing all shared barcodes) incorrectly discards cell libraries; molecule exclusion avoids this but only addresses swapping, not other artefacts
- Swapping is instrument-specific: approximately 2.5% of reads mislabelled on HiSeq 4000, lower than previous reports, and minimal on HiSeq 2500

## Evidence

- [abstract] Algorithm identifies swapped molecules and reassigns by majority read fraction: "we have developed an algorithm to exclude individual molecules that have swapped between samples in 10x Genomics experiments"
- [abstract] Approximate swapped-read fraction on HiSeq 4000: "approximately 2.5% of reads were mislabelled between samples on the HiSeq 4000, which is lower than previous reports"
- [discussion] Molecule exclusion key definition: "removing molecules that share the same combination of UMI, cell barcode, and aligned gene"
- [discussion] Cell calling parameters used alongside exclusion: "calling cells using emptyDrops18 using an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs"
- [discussion] Cell exclusion discards whole libraries, motivating molecule exclusion: "We applied cell exclusion by removing all barcodes that were shared between samples, and counted the fraction of cell libraries that were incorrectly discarded."
- [readme] DropletUtils provides the swapping-removal functionality: "This includes data loading from count matrices or molecule information files, identification of cells from empty droplets, removal of barcode-swapped pseudo-cells"
