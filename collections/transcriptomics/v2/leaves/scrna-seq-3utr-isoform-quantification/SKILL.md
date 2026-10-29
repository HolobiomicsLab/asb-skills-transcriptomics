---
name: scrna-seq-3utr-isoform-quantification
description: Use when raw 3'-end tag scRNA-seq data (FASTQ/BAM from 10x Genomics or
  Microwell-seq chemistry) must be converted into sparse gene-level and 3'UTR isoform
  count matrices for APA/UTR-length analysis.
when_to_use_negative:
- Data are full-length transcripts that span mRNA 3' end cleavage sites and support
  exact de novo cleavage-site mapping rather than UTRome-based pseudoalignment
- Only gene-level expression is needed — standard gene-count quantification without
  isoform resolution is simpler and sufficient
- The library is a bulk RNA-seq or non-3'-end-tag protocol incompatible with the kallisto-bustools
  3'-end tag workflow
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scUTRquant
  role: Snakemake pipeline that runs kallisto bus pseudoalignment against UTRome truncated-transcript
    targets, bustools barcode correction, min_umis cell filtering, and emits SingleCellExperiment
    3'UTR isoform matrices
  repo: https://github.com/Mayrlab/scUTRquant
- name: kallisto-bustools
  role: 'Underlying workflow: kallisto bus pseudoaligns reads to the UTRome index;
    bustools corrects cell barcodes against the whitelist'
- name: Snakemake
  role: Workflow manager; Conda installs the needed software within the pipeline
- name: Conda
  role: Environment management for pipeline execution (Conda or Singularity mode);
    Miniforge/Mamba recommended with libmamba solver
  repo: https://github.com/conda-forge/miniforge#miniforge
- name: txcutr-db
  role: Snakemake pipeline generating the truncated-transcript target files used as
    the UTRome indices
  repo: https://github.com/Mayrlab/txcutr-db
- name: scUTRquant-inputs
  role: Repository of input sample sheets for datasets analyzed in the manuscript
  repo: https://github.com/Mayrlab/scUTRquant-inputs
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:31800f2df4e3181b59891d3716afe547667841b730b03a17af9a2732a9aa8443
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/scUTRquant
  tools:
  - Conda
  - Snakemake
  - kallisto-bustools
  - scUTRboot
  - scUTRquant
  - scUTRquant-inputs
  - txcutr-db
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# scrna-seq-3utr-isoform-quantification

## Summary

Quantify gene and 3'UTR isoform expression simultaneously from 3'-end tag scRNA-seq data (e.g., 10x Genomics, Microwell-seq) by pseudoaligning reads with kallisto bus against a prebuilt UTRome index of truncated transcript (txcutr) targets, via the Snakemake-based scUTRquant pipeline.

## When to use

Use when raw 3'-end tag scRNA-seq data (FASTQ/BAM from 10x Genomics or Microwell-seq chemistry) must be converted into sparse gene-level and 3'UTR isoform count matrices for APA/UTR-length analysis.

## When NOT to use

- Data are full-length transcripts that span mRNA 3' end cleavage sites and support exact de novo cleavage-site mapping rather than UTRome-based pseudoalignment
- Only gene-level expression is needed — standard gene-count quantification without isoform resolution is simpler and sufficient
- The library is a bulk RNA-seq or non-3'-end-tag protocol incompatible with the kallisto-bustools 3'-end tag workflow

## Inputs

- Raw scRNA-seq FASTQ or BAM files from 3'-end tag protocols (10x Genomics or Microwell-seq)
- sample_sheet.csv listing input libraries
- config.yaml (target, tech, strand, min_umis settings)
- Prebuilt UTRome kallisto index (e.g., utrome_hg38_v1, utrome_mm10_v1) and 10x barcode whitelist

## Outputs

- SingleCellExperiment objects with sparse 3'UTR isoform count matrices (*.txs.Rds)
- Gene-level count matrices (*.genes.Rds)
- Gene UTR classification tables (e.g., gene_utr_classification.csv)
- scverse AnnData objects (experimental)

## How to apply

Assemble a Snakemake config.yaml and sample_sheet.csv listing FASTQ/BAM inputs, setting `target` to a UTRome index (e.g., utrome_hg38_v1 or utrome_mm10_v1), `tech` to the matching 10x chemistry, and `strand` to --fr-stranded for 10X 3'-end libraries. Run the scUTRquant pipeline in Conda/Mamba or Singularity mode so kallisto bus pseudoaligns reads against the prebuilt UTRome kallisto index and bustools corrects cell barcodes against the provided whitelist. Apply cell-level filtering with the `min_umis` threshold (optionally `exclude_unannotated_cells`) and emit SingleCellExperiment objects of sparse 3'UTR isoform counts (*.txs.Rds and *.genes.Rds per target). For downstream isoform-usage analyses, pseudobulk counts across cells per condition or cell type (retaining groups with >=200 cells) and classify genes with a >=10% isoform-fraction cutoff as multi-UTR or IPA. The truncated-transcript target design is what allows reads that do not span cleavage sites to be assigned to the correct 3'UTR isoform.

## Related tools

- **scUTRquant** (Snakemake pipeline that runs kallisto bus pseudoalignment against UTRome truncated-transcript targets, bustools barcode correction, min_umis cell filtering, and emits SingleCellExperiment 3'UTR isoform matrices) — https://github.com/Mayrlab/scUTRquant
- **kallisto-bustools** (Underlying workflow: kallisto bus pseudoaligns reads to the UTRome index; bustools corrects cell barcodes against the whitelist)
- **Snakemake** (Workflow manager; Conda installs the needed software within the pipeline)
- **Conda** (Environment management for pipeline execution (Conda or Singularity mode); Miniforge/Mamba recommended with libmamba solver) — https://github.com/conda-forge/miniforge#miniforge
- **txcutr-db** (Snakemake pipeline generating the truncated-transcript target files used as the UTRome indices) — https://github.com/Mayrlab/txcutr-db
- **scUTRquant-inputs** (Repository of input sample sheets for datasets analyzed in the manuscript) — https://github.com/Mayrlab/scUTRquant-inputs

## Examples

```
git clone git@github.com:Mayrlab/scUTRquant.git
```

## Evaluation signals

- Per-sample *.genes.Rds and *.txs.Rds SingleCellExperiment files exist for the specified target and contain sparse count matrices
- Cell-level QC applied: cells below the configured `min_umis` threshold are excluded from outputs
- Downstream gene classification reproduces reported counts when run on the same datasets (human: 8129/16185 multi-UTR, 4113 IPA; mouse: 6766/16195 multi-UTR, 1869 IPA at the >=10% isoform-fraction cutoff)
- Pseudobulk aggregation retains only cell types with >=200 cells before isoform-fraction classification
- Barcodes restricted to the 10x whitelist after bustools correct, matching the configured `tech` chemistry

## Limitations

- Most reads from 10x Genomics data do not span mRNA 3' end cleavage sites, so exact de novo CS mapping is not possible from such data — quantification relies on the prebuilt UTRome truncated-transcript targets
- Primary cleavage-site catalog was built from Microwell-seq data (Human Cell Landscape and Mouse Cell Atlas); isoforms absent from the UTRome cannot be quantified
- The manuscript recommends filtering targets to protein-coding transcripts with validated 3' ends and excluding mRNA_end_NF-tagged transcripts
- AnnData output is explicitly labeled experimental; SingleCellExperiment is the primary output format

## Evidence

- [readme] A bioinformatics pipeline for single-cell 3' UTR isoform quantification.: "A bioinformatics pipeline for single-cell 3' UTR isoform quantification."
- [methods] kallisto bus and bustools in the workflow: "`tech`: argument to `kallisto bus` indicating the scRNA-seq technology"
- [methods] min_umis cell filter: "`min_umis`: minimum number of UMIs per cell; cells below this threshold are excluded"
- [abstract] simultaneous gene and 3'UTR quantification scope: "simultaneous quantification of gene and 3'UTR isoform expression from single-cell RNA sequencing (scRNA-seq) data"
- [intro] rationale for truncated targets given 10x read limitation: "most reads from 10x Genomics data do not span mRNA 3′ end cleavage sites (CS), which prevents exact de novo mapping of mRNA 3′ ends."
- [methods] target recommendations for transcript filtering: "We recommend filtering transcripts to only protein-coding transcripts with validated 3' ends"
