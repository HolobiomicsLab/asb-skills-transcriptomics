---
name: alternative-polyadenylation-gene-classification
description: Use when you have scUTRquant 3'UTR isoform count matrices (*.txs.Rds
  / *.genes.Rds) from 10x 3'-end scRNA-seq data and need to label genes as multi-UTR
  or IPA and tabulate per-category gene counts.
when_to_use_negative:
- Input is a gene-level count matrix without 3'UTR isoform resolution — multi-UTR/IPA
  calls require isoform-fraction quantification from scUTRquant
- Data are not 3'-end tag-based scRNA-seq libraries compatible with the kallisto-bustools
  workflow (most 10x reads do not span cleavage sites, so exact de novo CS mapping
  is not possible)
- The goal is per-cell-type differential 3'UTR length testing rather than static gene
  classification — use scUTRboot instead
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: scUTRquant
  role: Snakemake pipeline producing gene and 3'UTR isoform count matrices (SingleCellExperiment
    .txs.Rds/.genes.Rds) used as the basis for classification
  repo: https://github.com/Mayrlab/scUTRquant
- name: kallisto-bustools
  role: kallisto bus pseudoalignment against the UTRome index and bustools barcode
    correction underlying isoform quantification
  repo: https://github.com/Mayrlab/scUTRquant
- name: atlas-mm
  role: Mouse UTRome Atlas pipeline that tabulates tandem isoform usage per cell type
    at 5%/10% frequency and classifies multi-UTR genes
  repo: https://github.com/Mayrlab/atlas-mm
- name: atlas-hs
  role: Human UTRome Atlas pipeline analyzing Tabula Sapiens for the same cell-type
    isoform-usage tabulation and multi-UTR classification
  repo: https://github.com/Mayrlab/atlas-hs
- name: Snakemake
  role: Workflow manager executing the scUTRquant pipeline in Conda or Singularity
    mode
  repo: https://github.com/Mayrlab/scUTRquant
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:9c53497d57dbab3e9285ea7c65d6b616720339a59c7dc73c4adb1fdc0c6e1397
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/scUTRquant
  tools:
  - Conda
  - Snakemake
  - atlas-hs
  - atlas-mm
  - kallisto-bustools
  - scUTRquant
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# alternative-polyadenylation-gene-classification

## Summary

Classifies expressed protein-coding genes as multi-UTR or intronic polyadenylation (IPA) genes by aggregating 3'UTR isoform usage across cell types from scUTRquant SingleCellExperiment outputs and applying a ≥10% isoform-fraction criterion.

## When to use

Use when you have scUTRquant 3'UTR isoform count matrices (*.txs.Rds / *.genes.Rds) from 10x 3'-end scRNA-seq data and need to label genes as multi-UTR or IPA and tabulate per-category gene counts.

## When NOT to use

- Input is a gene-level count matrix without 3'UTR isoform resolution — multi-UTR/IPA calls require isoform-fraction quantification from scUTRquant
- Data are not 3'-end tag-based scRNA-seq libraries compatible with the kallisto-bustools workflow (most 10x reads do not span cleavage sites, so exact de novo CS mapping is not possible)
- The goal is per-cell-type differential 3'UTR length testing rather than static gene classification — use scUTRboot instead

## Inputs

- FASTQ/BAM inputs from 10x Genomics 3'-end scRNA-seq libraries (e.g., Tabula Sapiens, Tabula Muris)
- config.yaml Snakemake configfile with target (utrome_hg38_v1 / utrome_mm10_v1), tech, and strand settings
- sample_sheet.csv listing sample inputs
- Prebuilt UTRome kallisto index and barcode whitelist
- scUTRquant SingleCellExperiment objects (*.txs.Rds and *.genes.Rds) of sparse 3'UTR isoform count matrices

## Outputs

- gene_utr_classification.csv with per-gene multi-UTR/IPA labels and per-category gene counts
- Cell-type-aggregated 3'UTR isoform usage tables (cell types with >=200 cells)

## How to apply

First run scUTRquant (Snakemake, Conda or Singularity mode) so kallisto bus pseudoaligns reads against a prebuilt UTRome kallisto index (target utrome_hg38_v1 or utrome_mm10_v1) and bustools corrects cell barcodes against the whitelist, with strand set to --fr-stranded for 10X 3'-end libraries. Apply cell-level filtering with the min_umis threshold to produce sparse 3'UTR isoform count matrices as SingleCellExperiment objects. Aggregate 3'UTR isoform usage across cells belonging to each cell type, retaining only cell types with >=200 cells, then classify each gene by its isoform fraction using a >=10% isoform fraction cutoff, labeling genes as multi-UTR (genes with more than two tandem isoforms used at this frequency) or IPA genes. Tabulate per-gene classifications in gene_utr_classification.csv and validate the category counts against the reported values (human 8129/16185 multi-UTR and 4113 IPA; mouse 6766/16195 multi-UTR and 1869 IPA, matching Supplementary Data 3-4).

## Related tools

- **scUTRquant** (Snakemake pipeline producing gene and 3'UTR isoform count matrices (SingleCellExperiment .txs.Rds/.genes.Rds) used as the basis for classification) — https://github.com/Mayrlab/scUTRquant
- **kallisto-bustools** (kallisto bus pseudoalignment against the UTRome index and bustools barcode correction underlying isoform quantification) — https://github.com/Mayrlab/scUTRquant
- **atlas-mm** (Mouse UTRome Atlas pipeline that tabulates tandem isoform usage per cell type at 5%/10% frequency and classifies multi-UTR genes) — https://github.com/Mayrlab/atlas-mm
- **atlas-hs** (Human UTRome Atlas pipeline analyzing Tabula Sapiens for the same cell-type isoform-usage tabulation and multi-UTR classification) — https://github.com/Mayrlab/atlas-hs
- **Snakemake** (Workflow manager executing the scUTRquant pipeline in Conda or Singularity mode) — https://github.com/Mayrlab/scUTRquant

## Examples

```
snakemake --use-conda --configfile config.yaml --cores 8  # with target=utrome_mm10_v1 in config.yaml; classification from *.txs.Rds via isoform-fraction >=10% per cell type
```

## Evaluation signals

- Per-category gene counts in gene_utr_classification.csv must equal human 8129/16185 multi-UTR and 4113 IPA; mouse 6766/16195 multi-UTR and 1869 IPA (matching Supplementary Data 3-4)
- Isoform usage aggregation must retain only cell types with >=200 cells
- Genes must be labeled multi-UTR only when more than two tandem (non-intronic) isoforms are used at the >=10% frequency threshold across distinct cell types
- IPA isoforms must be distinguished from tandem (non-intronic) isoforms in the classification table
- All classified genes must be among expressed protein-coding transcripts (16,195 mouse and 16,185 human denominators must be reproduced)

## Limitations

- 10x Genomics reads mostly do not span mRNA 3' end cleavage sites, preventing exact de novo mapping of 3' ends; classification relies on the curated UTRome annotation
- Cell types with fewer than 200 cells are excluded from the aggregation, so gene classifications reflect only well-populated cell types
- Only protein-coding transcripts with validated 3' ends should be retained; transcripts tagged mRNA_end_NF should be excluded

## Evidence

- [readme] This pipeline merges multiple datasets to define an atlas of 3' UTRs. At its core is a tabulation of how many distinct cell types use a particular tandem (non-intronic) isoform at either 5% or 10% frequency. Genes that have more that two isoforms of this type are classified as multi-UTR genes.: [evidence span withheld — anchor status: external_doc]
- [methods] Workflow classification criterion from the source card: [evidence span withheld — anchor status: not_found]
- [results] Finding with mouse counts: [evidence span withheld — anchor status: not_found]
- [methods] Aggregation threshold from source card: [evidence span withheld — anchor status: not_found]
- [intro] 10x limitation on cleavage-site coverage: "most reads from 10x Genomics data do not span mRNA 3′ end cleavage sites (CS), which prevents exact de novo mapping of mRNA 3′ ends."
