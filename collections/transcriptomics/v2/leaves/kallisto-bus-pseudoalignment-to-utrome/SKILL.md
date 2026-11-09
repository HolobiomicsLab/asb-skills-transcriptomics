---
name: kallisto-bus-pseudoalignment-to-utrome
description: Use when raw FASTQ/BAM files from 10x Genomics 3'-end tag scRNA-seq libraries
  must be quantified at 3'UTR isoform resolution against the UTRome annotation rather
  than the whole transcriptome.
when_to_use_negative:
- Data were generated with full-length or non-3'-end-tag protocols (scUTRquant is
  compatible with most 3'-end tag scRNA-seq libraries only)
- Goal is de novo cleavage-site discovery from 10x data — most 10x reads do not span
  mRNA 3' end cleavage sites, preventing exact de novo mapping; use dedicated 3' end
  sequencing (e.g., MWS) workflows instead
- Only gene-level counts are needed and 3'UTR isoform resolution is irrelevant — a
  standard transcriptome index suffices
edam_operation: http://edamontology.org/operation_0294
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: kallisto
  role: Pseudoaligns scRNA-seq reads (`kallisto bus`) against the prebuilt UTRome
    kallisto index, using the `tech` argument for the 10x chemistry
- name: bustools
  role: Corrects cell barcodes against the technology whitelist (`bustools correct`)
- name: scUTRquant
  role: Snakemake pipeline that orchestrates kallisto bus pseudoalignment against
    the UTRome index, barcode correction, min_umis cell filtering, and SingleCellExperiment
    output
  repo: https://github.com/Mayrlab/scUTRquant
- name: Snakemake
  role: Workflow engine driving the scUTRquant pipeline in Conda or Singularity mode
  repo: https://github.com/Mayrlab/scUTRquant
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:41e6b7655ba545bbb3ac716d607d018b9d92682d9ea86ba42b4e1e9b70999ac5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/scUTRquant
  tools:
  - Conda
  - Snakemake
  - bustools
  - kallisto
  - kallisto-bustools
  - scUTRquant
  edam_operation: http://edamontology.org/operation_0294
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# kallisto-bus-pseudoalignment-to-utrome

## Summary

Pseudoalign 3'-end tag scRNA-seq reads (e.g., 10x Genomics FASTQs) with `kallisto bus` against a prebuilt UTRome kallisto index (utrome_hg38_v1 or utrome_mm10_v1) inside the scUTRquant Snakemake pipeline, then correct cell barcodes with `bustools correct` to produce 3'UTR isoform count matrices.

## When to use

Use when raw FASTQ/BAM files from 10x Genomics 3'-end tag scRNA-seq libraries must be quantified at 3'UTR isoform resolution against the UTRome annotation rather than the whole transcriptome.

## When NOT to use

- Data were generated with full-length or non-3'-end-tag protocols (scUTRquant is compatible with most 3'-end tag scRNA-seq libraries only)
- Goal is de novo cleavage-site discovery from 10x data — most 10x reads do not span mRNA 3' end cleavage sites, preventing exact de novo mapping; use dedicated 3' end sequencing (e.g., MWS) workflows instead
- Only gene-level counts are needed and 3'UTR isoform resolution is irrelevant — a standard transcriptome index suffices

## Inputs

- Raw scRNA-seq FASTQ files (R1/R2) or BAM inputs from 10x Genomics 3'-end tag libraries (sample_sheet.csv)
- Prebuilt UTRome kallisto index (utrome_hg38_v1 or utrome_mm10_v1) from Mayrlab/scUTRquant-inputs
- Barcode whitelist file for the 10x chemistry
- scUTRquant config.yaml (target, tech, strand, min_umis)

## Outputs

- Sparse 3'UTR isoform count matrices as SingleCellExperiment objects (*.txs.Rds and *.genes.Rds per target)
- Gene-level and isoform-level expression per cell, suitable for downstream scUTRboot testing and cell-type aggregation

## How to apply

Configure the scUTRquant Snakemake configfile (config.yaml) and sample_sheet.csv with the FASTQ/BAM inputs, setting `target` to utrome_hg38_v1 or utrome_mm10_v1 and `tech` to the matching 10x chemistry. Run the pipeline in Conda or Singularity mode so that `kallisto bus` pseudoaligns reads against the prebuilt UTRome kallisto index with `--fr-stranded` strandness for 10X 3'-end libraries. Correct cell barcodes with `bustools correct` using the technology-specific barcode whitelist. After pseudoalignment, apply cell-level filtering with `min_umis` and produce sparse 3'UTR isoform count matrices stored as SingleCellExperiment objects (*.txs.Rds and *.genes.Rds per target). The truncated-transcript UTRome targets are what allow multi-UTR genes to be distinguished and classified downstream (e.g., 10% isoform-fraction criterion, multi-UTR vs IPA gene labeling).

## Related tools

- **kallisto** (Pseudoaligns scRNA-seq reads (`kallisto bus`) against the prebuilt UTRome kallisto index, using the `tech` argument for the 10x chemistry)
- **bustools** (Corrects cell barcodes against the technology whitelist (`bustools correct`))
- **scUTRquant** (Snakemake pipeline that orchestrates kallisto bus pseudoalignment against the UTRome index, barcode correction, min_umis cell filtering, and SingleCellExperiment output) — https://github.com/Mayrlab/scUTRquant
- **Snakemake** (Workflow engine driving the scUTRquant pipeline in Conda or Singularity mode) — https://github.com/Mayrlab/scUTRquant

## Examples

```
kallisto bus -i utrome_hg38_v1.kidx -o kb_out -x 10xv3 --fr-stranded sample_R1.fastq.gz sample_R2.fastq.gz
```

## Evaluation signals

- Output directory contains both *.txs.Rds and *.genes.Rds SingleCellExperiment files for the configured target (utrome_hg38_v1 or utrome_mm10_v1)
- Downstream gene classification reproduces published per-category counts (human: 8129/16185 multi-UTR and 4113 IPA; mouse: 6766/16195 multi-UTR and 1869 IPA) matching Supplementary Data 3-4
- Cell barcode counts are consistent with the whitelist used in `bustools correct`; cells below the `min_umis` threshold are excluded
- ISO counts show stranded behavior consistent with `--fr-stranded` for 10X 3'-end libraries
- Pseudoalignment succeeds against the truncated-transcript UTRome index, resolving multiple 3'UTR isoforms for multi-UTR genes (as validated by the kallisto-overlap benchmark on simulated multi-UTR reads)

## Limitations

- Most reads from 10x Genomics data do not span mRNA 3′ end cleavage sites, which prevents exact de novo mapping of mRNA 3′ ends — quantification is limited to the curated UTRome annotation
- Requires 3'-end tag scRNA-seq libraries; the workflow is not designed for full-length protocols
- Conda should be upgraded to at least v23.11 (libmamba solver) for reliable environment installation during pipeline runs

## Evidence

- [other] kallisto bus pseudoaligns reads against the prebuilt UTRome kallisto index and bustools corrects cell barcodes against the whitelist: [evidence span withheld — anchor status: not_found]
- [discussion] kallisto-bustools workflow compatibility with 3'-end tag libraries: "By building on the kallisto-bustools workflow, we provide a fast and direct means of quantification that is compatible with most 3′-end tag scRNA-seq libraries"
- [methods] kallisto bus tech argument: "`tech`: argument to `kallisto bus` indicating the scRNA-seq technology"
- [methods] bustools correct whitelist: "`bx_whitelist`: file of valid barcodes used in `bustools correct`"
- [readme] scUTRquant purpose: "A bioinformatics pipeline for single-cell 3' UTR isoform quantification."
