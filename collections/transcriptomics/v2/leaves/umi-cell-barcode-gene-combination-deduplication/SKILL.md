---
name: umi-cell-barcode-gene-combination-deduplication
description: Use when droplet-based 10x Genomics libraries from multiple samples were
  sequenced together on patterned flow-cell instruments (HiSeq 4000, HiSeq X, NovaSeq)
  and swapped molecules sharing UMI, cell barcode, and gene across samples must be
  removed.
when_to_use_negative:
- Non-multiplexed samples sequenced on non-patterned instruments such as the HiSeq
  2500, where the control expectation is a negligible removed fraction (below 0.0005%)
- Plate-based scRNA-seq data, where the article instead applies cell exclusion by
  removing all barcodes shared between samples rather than molecule-level exclusion
- Data lacking per-molecule UMI/cell-barcode/gene assignments (e.g. only aggregated
  count matrices without molecule information)
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: DropletUtils
  role: Implements the molecule-exclusion algorithm removing molecules sharing the
    same UMI, cell barcode, and aligned gene combination, plus emptyDrops cell calling
    and removal of barcode-swapped pseudo-cells
  repo: https://github.com/MarioniLab/DropletUtils
- name: R
  role: Analysis environment used to run DropletUtils for molecule exclusion and emptyDrops
    cell calling
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:d64501c232b2a19b3ce4836077e5a84a5808e0922bbb37cb9ac6aeb0bcf36267
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarioniLab/DropletUtils
  tools:
  - DropletUtils
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# umi-cell-barcode-gene-combination-deduplication

## Summary

Excludes molecules that share the same combination of UMI, cell barcode, and aligned gene across multiplexed 10x Genomics samples, as implemented in DropletUtils, to remove artefactual barcode-swapped molecules from patterned flow-cell (HiSeq 4000) droplet scRNA-seq data.

## When to use

Use when droplet-based 10x Genomics libraries from multiple samples were sequenced together on patterned flow-cell instruments (HiSeq 4000, HiSeq X, NovaSeq) and swapped molecules sharing UMI, cell barcode, and gene across samples must be removed.

## When NOT to use

- Non-multiplexed samples sequenced on non-patterned instruments such as the HiSeq 2500, where the control expectation is a negligible removed fraction (below 0.0005%)
- Plate-based scRNA-seq data, where the article instead applies cell exclusion by removing all barcodes shared between samples rather than molecule-level exclusion
- Data lacking per-molecule UMI/cell-barcode/gene assignments (e.g. only aggregated count matrices without molecule information)

## Inputs

- 10x-style UMI count matrices or molecule information files from multiplexed droplet scRNA-seq samples (e.g. from ArrayExpress E-MTAB-6854)
- Reads mapped to mm10 genome with exonic read pairs counted per gene per sample
- emptyDrops cell calls (FDR 0.01, minimum 1000 UMIs) on uncorrected data as a baseline

## Outputs

- Molecule-excluded UMI count matrices per sample
- Corrected emptyDrops cell calls (FDR 0.01, minimum 1000 UMIs) per sample
- Number and fraction of molecules removed per sample (e.g. swapped_molecule_removal_results.csv)

## How to apply

Load the 10x droplet count matrices (e.g. mouse epithelial data from ArrayExpress E-MTAB-6854, mapped to mm10 with exonic read pairs counted per gene) into R using DropletUtils. Establish baseline cell calls with emptyDrops at an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs on the uncorrected data. Apply the molecule-exclusion algorithm to remove molecules sharing the same combination of UMI, cell barcode, and aligned gene across samples, assigning each ambiguous molecule to its majority (80%) sample of origin. Re-run emptyDrops (FDR 0.01, minimum 1000 UMIs) on the molecule-excluded matrices to obtain corrected cell calls. The rationale is that genuinely distinct molecules cannot share the same UMI, cell barcode, and gene in two samples simultaneously, so such collisions indicate barcode swapping on patterned flow-cell machines.

## Related tools

- **DropletUtils** (Implements the molecule-exclusion algorithm removing molecules sharing the same UMI, cell barcode, and aligned gene combination, plus emptyDrops cell calling and removal of barcode-swapped pseudo-cells) — https://github.com/MarioniLab/DropletUtils
- **R** (Analysis environment used to run DropletUtils for molecule exclusion and emptyDrops cell calling)

## Evaluation signals

- On non-multiplexed HiSeq 2500-sequenced control samples, the removed molecule fraction must be below 0.0005%
- On multiplexed HiSeq 4000 data, nearly all emptyDrops-called cells should be lost from artefactual samples B1 and B2 after exclusion
- Approximately 2.5% of cell libraries from other HiSeq 4000 samples should no longer be called as cells, consistent with the estimated swapped-read fraction
- Cell calling before and after exclusion must use identical parameters (emptyDrops, FDR threshold 0.01, minimum 1000 UMIs) so differences reflect only removed molecules
- The per-sample summary table (e.g. swapped_molecule_removal_results.csv) should record the number and fraction of molecules removed for each sample

## Limitations

- Assigning ambiguous molecules to their majority (80%) sample of origin is a heuristic; borderline molecules may be misassigned
- The method is specific to multiplexed 10x Genomics droplet experiments; it does not address swapping in plate-based datasets, which require cell- or barcode-level exclusion instead
- On non-multiplexed HiSeq 2500 data the method removes a negligible fraction (below 0.0005%), so it provides no benefit as a correction there and serves only as a negative control

## Evidence

- [discussion] Molecule exclusion definition: "removing molecules that share the same combination of UMI, cell barcode, and aligned gene"
- [discussion] emptyDrops parameters: "calling cells using emptyDrops18 using an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs"
- [discussion] Read mapping and counting: "Reads were mapped to the mm10 genome, and the number of read pairs mapping to the exonic region of each gene were counted."
- [abstract] Algorithm objective: "we have developed an algorithm to exclude individual molecules that have swapped between samples in 10x Genomics experiments"
- [readme] DropletUtils scope: "This includes data loading from count matrices or molecule information files, identification of cells from empty droplets, removal of barcode-swapped pseudo-cells"
