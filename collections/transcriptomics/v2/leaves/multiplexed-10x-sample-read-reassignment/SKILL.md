---
name: multiplexed-10x-sample-read-reassignment
description: Use when 10x Genomics droplet libraries from multiple samples were sequenced
  together on a patterned-flowcell machine (HiSeq 4000, HiSeq X, NovaSeq) and barcode
  swapping between samples must be corrected at the molecule level.
when_to_use_negative:
- Libraries sequenced on non-patterned-flowcell instruments such as the HiSeq 2500,
  where swapping is negligible and the control experiment showed low mislabelling
- Single-sample (non-multiplexed) 10x experiments, where no cross-sample UMI/barcode/gene
  sharing can occur
- Plate-based scRNA-seq data, where the paper instead estimates swapping fractions
  via linear regression on impossible barcode combinations
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DropletUtils
  role: R/Bioconductor package implementing the molecule-exclusion (swappedDrops)
    algorithm on 10x molecule information files to remove barcode-swapped pseudo-cells
  repo: https://github.com/MarioniLab/DropletUtils
- name: R
  role: Language in which the swappedDrops algorithm is implemented and the cleaned
    per-sample sparse matrices are generated and saved
- name: 10x Genomics
  role: Droplet scRNA-seq platform whose molecule information files provide the UMI
    / cell barcode / gene records used to detect swapped molecules
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:c6912c307f3984802a8983ed9c8f6e443fa3c32a3f2ceae3b8e09877da9fc471
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarioniLab/DropletUtils
  tools:
  - 10x Genomics
  - DropletUtils
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# multiplexed-10x-sample-read-reassignment

## Summary

A molecule-exclusion algorithm (swappedDrops in DropletUtils) that identifies barcode-swapped molecules shared between multiplexed 10x Genomics droplet samples — same UMI, cell barcode, and aligned gene — and reassigns or removes them using a tunable min.frac majority-read threshold.

## When to use

Use when 10x Genomics droplet libraries from multiple samples were sequenced together on a patterned-flowcell machine (HiSeq 4000, HiSeq X, NovaSeq) and barcode swapping between samples must be corrected at the molecule level.

## When NOT to use

- Libraries sequenced on non-patterned-flowcell instruments such as the HiSeq 2500, where swapping is negligible and the control experiment showed low mislabelling
- Single-sample (non-multiplexed) 10x experiments, where no cross-sample UMI/barcode/gene sharing can occur
- Plate-based scRNA-seq data, where the paper instead estimates swapping fractions via linear regression on impossible barcode combinations

## Inputs

- 10x Genomics molecule information files (per-sample UMI, cell barcode, aligned gene records) for multiplexed droplet samples
- min.frac threshold parameter (default 0.9; paper used 0.8)

## Outputs

- Cleaned per-sample sparse count matrices with swapped molecules reassigned or removed (saved as cleaned_count_matrices.rds)

## How to apply

Load the 10x molecule information files (UMI, cell barcode, aligned gene per read) for each multiplexed sample sequenced on the HiSeq 4000/HiSeq 2500, then group molecules across samples by identical UMI / cell-barcode / aligned-gene combination keys to find candidate swapping events. For each ambiguous combination, compute the fraction of reads contributed by each sample; if a single sample holds at least min.frac (default 0.9; the paper's analysis used 0.8), reassign the molecule to that sample-of-origin and remove its count from all other samples. Molecules whose reads cannot be confidently attributed (no sample meets min.frac) are removed from every sample. This molecule-level strategy is preferred over cell exclusion (removing all barcodes shared between samples), which can incorrectly discard entire cell libraries. The rationale is that patterned flow-cell Illumina machines such as the HiSeq 4000 mislabel ~2.5% of reads between samples, which can otherwise generate complex but artefactual cell libraries.

## Related tools

- **DropletUtils** (R/Bioconductor package implementing the molecule-exclusion (swappedDrops) algorithm on 10x molecule information files to remove barcode-swapped pseudo-cells) — https://github.com/MarioniLab/DropletUtils
- **R** (Language in which the swappedDrops algorithm is implemented and the cleaned per-sample sparse matrices are generated and saved)
- **10x Genomics** (Droplet scRNA-seq platform whose molecule information files provide the UMI / cell barcode / gene records used to detect swapped molecules)

## Examples

```
library(DropletUtils); out <- swappedDrops(mol.info.files, min.frac=0.9); saveRDS(out$cleaned, "cleaned_count_matrices.rds")
```

## Evaluation signals

- Cleaned count matrices for each sample are produced and saved (e.g. cleaned_count_matrices.rds) with swapped molecule counts removed from non-origin samples
- Every molecule flagged as swapped shares an identical UMI, cell barcode, and aligned gene combination across at least two multiplexed samples
- Reassigned molecules satisfy the min.frac rule: one sample holds at least the threshold fraction of reads (0.9 default, 0.8 in the paper) for that combination
- Molecules with no sample meeting min.frac are absent from all output matrices
- Applied as a control, the same procedure on HiSeq 2500-sequenced libraries should yield a much lower swapped-read fraction than the ~2.5% observed on the HiSeq 4000

## Limitations

- If no sample meets min.frac for a combination, the molecule is discarded entirely, potentially losing usable reads
- Cell-level exclusion (removing all shared barcodes) can incorrectly discard whole cell libraries; molecule-level exclusion avoids this but requires molecule information files rather than plain count matrices
- The swapped fraction is instrument-dependent: ~2.5% of reads mislabelled on the HiSeq 4000, lower than previous reports, so the correction matters mainly for patterned flow-cell machines
- Key details of the molecule exclusion method are deferred to Supplementary Note 6, which is absent from the available document ('No changelog found.')

## Evidence

- [abstract] Algorithm targets molecule-level exclusion in 10x data: "we have developed an algorithm to exclude individual molecules that have swapped between samples in 10x Genomics experiments"
- [discussion] Molecule exclusion defined by shared UMI/barcode/gene: "removing molecules that share the same combination of UMI, cell barcode, and aligned gene"
- [discussion] Cell exclusion can incorrectly discard libraries: "We applied cell exclusion by removing all barcodes that were shared between samples, and counted the fraction of cell libraries that were incorrectly discarded."
- [readme] DropletUtils implements barcode-swap removal: "This includes data loading from count matrices or molecule information files, identification of cells from empty droplets, removal of barcode-swapped pseudo-cells"
- [abstract] ~2.5% mislabelling on HiSeq 4000: "approximately 2.5% of reads were mislabelled between samples on the HiSeq 4000"
