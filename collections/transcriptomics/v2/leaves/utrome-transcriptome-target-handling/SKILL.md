---
name: utrome-transcriptome-target-handling
description: Use when running scUTRquant on 10x Genomics or Microwell-seq-style 3'-end
  scRNA-seq FASTQs and a UTRome/CS-annotated target (e.g. utrome_mm10_v2) must be
  chosen, built, or configured for gene and 3'UTR isoform quantification.
when_to_use_negative:
- The analysis goal is standard whole-transcript gene quantification with full-length
  transcriptomes (Cell Ranger) and no 3'UTR isoform resolution is needed — a standard
  GENCODE reference suffices
- Input data is full-length or 5'-end scRNA-seq that does not cover transcript 3'
  ends, since truncated-UTR targets are designed for 3'-end tag-based libraries
- A custom cleavage-site catalog is required and cannot come from GENCODE-validated
  3' ends or the MWS CS atlas — txcutr-db/UTRome targets would not reflect the desired
  CS set
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: txcutr-db
  role: Snakemake pipeline that generates the truncated-transcript target files used
    by scUTRquant
  repo: https://github.com/Mayrlab/txcutr-db
- name: scUTRquant
  role: Pipeline that consumes the target via the config.yaml 'target' key to quantify
    gene and 3'UTR isoform expression
  repo: https://github.com/Mayrlab/scUTRquant
- name: mca-utrome
  role: Produces the Mouse UTRome (UTRome target) from Mouse Cell Atlas cleavage-site
    calls
  repo: https://github.com/Mayrlab/mca-utrome
- name: kallisto-bustools
  role: Pseudoaligns reads (kallisto bus) against the truncated target and corrects
    barcodes (bustools correct) to produce BUS/MTX counts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:a939c10ac6ac43f4901c42712ac44095ff8e6e9457b68e3eb48aab7693f8800e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/txcutr-db
  tools:
  - Conda
  - kallisto-bustools
  - mca-utrome
  - scUTRquant
  - txcutr-db
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# utrome-transcriptome-target-handling

## Summary

Prepares and selects truncated-UTR transcriptome targets (e.g. utrome_mm10_v2) as the pseudoalignment reference for scUTRquant, so that kallisto|bustools can quantify 3'UTR isoforms and genes simultaneously from scRNA-seq data. Targets are built by truncating protein-coding transcripts with validated 3' ends from GENCODE plus MWS-derived cleavage sites.

## When to use

Use when running scUTRquant on 10x Genomics or Microwell-seq-style 3'-end scRNA-seq FASTQs and a UTRome/CS-annotated target (e.g. utrome_mm10_v2) must be chosen, built, or configured for gene and 3'UTR isoform quantification.

## When NOT to use

- The analysis goal is standard whole-transcript gene quantification with full-length transcriptomes (Cell Ranger) and no 3'UTR isoform resolution is needed — a standard GENCODE reference suffices
- Input data is full-length or 5'-end scRNA-seq that does not cover transcript 3' ends, since truncated-UTR targets are designed for 3'-end tag-based libraries
- A custom cleavage-site catalog is required and cannot come from GENCODE-validated 3' ends or the MWS CS atlas — txcutr-db/UTRome targets would not reflect the desired CS set

## Inputs

- GENCODE v39 / vM21 GTF and transcript FASTA annotations (protein-coding transcripts with verified 3' ends)
- MWS-derived cleavage site (CS) catalog from Human Cell Landscape / Mouse Cell Atlas processing
- Pre-built target files from the Mayrlab/scUTRquant-inputs repository (e.g. utrome_mm10_v2)
- scUTRquant dataset config.yaml with keys dataset_name, output_type ['txs','genes'], and target

## Outputs

- Truncated-transcript target files (UTRome FASTA/index, e.g. utrome_mm10_v2) consumable by kallisto bus
- Transcript-level (txs) and gene-level (genes) UMI count matrices as SingleCellExperiment objects under data/sce/<target>/
- A kallisto transcriptome index compatible with scUTRquant's BUS workflow

## How to apply

Build or obtain a truncated-transcript target via the txcutr-db Snakemake pipeline, which derives targets from protein-coding transcripts with verified 3' ends in GENCODE v39 (human) or vM21 (mouse), excluding transcripts tagged mRNA_end_NF; the Mouse UTRome output of the mca-utrome pipeline is one such target. Reference target files are distributed through the scUTRquant-inputs repository so a pre-built target like 'utrome_mm10_v2' can be used directly. In the dataset config.yaml, set the 'target' key (e.g. utrome_mm10_v2) alongside output_type ['txs','genes'] so kallisto bus pseudoaligns reads to truncated UTR transcripts and bustools generates both transcript- and gene-level counts. The rationale is that truncated targets separate 3'UTR isoforms that full-length transcriptomes collapse, while still yielding gene counts comparable to Cell Ranger.

## Related tools

- **txcutr-db** (Snakemake pipeline that generates the truncated-transcript target files used by scUTRquant) — https://github.com/Mayrlab/txcutr-db
- **scUTRquant** (Pipeline that consumes the target via the config.yaml 'target' key to quantify gene and 3'UTR isoform expression) — https://github.com/Mayrlab/scUTRquant
- **mca-utrome** (Produces the Mouse UTRome (UTRome target) from Mouse Cell Atlas cleavage-site calls) — https://github.com/Mayrlab/mca-utrome
- **kallisto-bustools** (Pseudoaligns reads (kallisto bus) against the truncated target and corrects barcodes (bustools correct) to produce BUS/MTX counts)

## Examples

```
snakemake --use-conda --configfile examples/<dataset>/config.yaml  # config.yaml sets target: utrome_mm10_v2 and output_type: ['txs','genes']
```

## Evaluation signals

- config.yaml resolves the 'target' key (e.g. utrome_mm10_v2) and output_type is ['txs','genes'], producing SCE objects under data/sce/utrome_mm10_v2/
- The target's source transcripts are protein-coding with verified 3' ends and exclude mRNA_end_NF-tagged transcripts
- Gene-level counts from the truncated target agree with Cell Ranger: per-cell Spearman rho > 0.99 and per-gene Spearman rho ~0.92 on the 10x demo datasets
- The BUS/MTX intermediates and HTML QC reports are generated for every sample run against the target
- Target files are reproducibly regenerated by the txcutr-db Snakemake pipeline (CI snakemake-validate passes)

## Limitations

- Most reads from 10x Genomics data do not span mRNA 3' end cleavage sites, preventing exact de novo mapping of CS; 10x libraries require '--fr-stranded' strand setting in scUTRquant
- Targets should be filtered to protein-coding transcripts with validated 3' ends; unvalidated ends (mRNA_end_NF) are excluded, limiting coverage to well-annotated genes
- CS annotations extend GENCODE by ~40% but still reflect major/minor cleavage site usage — most genes have only one or two major 3' ends, so rare isoforms may be under-resolved

## Evidence

- [methods] protein-coding filter: "We recommend filtering transcripts to only protein-coding transcripts with verified 3' ends"
- [methods] gencode source: "protein coding transcripts that have verified 3' ends in the GENCODE v39 and vM21 annotations"
- [readme] mouse utrome source: [evidence span withheld — anchor status: not_found]
- [readme] target generation: [evidence span withheld — anchor status: external_doc]
- [abstract] quantification purpose: "We incorporate the CS annotations into a computational pipeline, called scUTRquant, for rapid, accurate, and simultaneous quantification of gene and 3'UTR isoform expression"
