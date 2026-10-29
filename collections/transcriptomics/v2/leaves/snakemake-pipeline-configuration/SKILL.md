---
name: snakemake-pipeline-configuration
description: Use when raw 10x 3'-end FASTQ/BAM scRNA-seq inputs must be quantified
  for 3'UTR isoforms via the scUTRquant Snakemake pipeline, requiring a config.yaml
  with target, tech, strand, whitelist, and min_umis settings plus a sample_sheet.csv.
when_to_use_negative:
- The input is already a gene-by-cell UMI count matrix (e.g. a Cell Ranger filtered
  HDF5), not raw FASTQ/BAM reads needing pseudoalignment
- The data are full-length or 5'-end scRNA-seq libraries rather than 3'-end tag-based
  libraries, since scUTRquant is built for 3'-end tag data
- De novo mapping of mRNA 3' end cleavage sites is needed — most 10x reads do not
  span cleavage sites, and cleavage-site calling is handled by separate UTRome pipelines
  (hcl-utrome, mca-utrome)
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scUTRquant
  role: Snakemake pipeline being configured; orchestrates kallisto bus pseudoalignment
    against a UTRome index and bustools barcode correction to quantify gene and 3'UTR
    isoform expression
  repo: https://github.com/Mayrlab/scUTRquant
- name: Snakemake
  role: Workflow engine invoked with --use-conda and a configfile; requires version
    >= 6.0
- name: Conda
  role: Installs pipeline software environments; Conda >= v23.11 with the libmamba
    solver recommended (installable via Miniforge)
  repo: https://github.com/conda-forge/miniforge
- name: kallisto-bustools
  role: 'Underlying quantification engine: kallisto bus pseudoaligns reads to the
    UTRome index; bustools correct validates cell barcodes against the whitelist'
- name: scUTRquant-inputs
  role: Provides input sample sheets and scripts for downloading raw data used to
    run published datasets through the pipeline
  repo: https://github.com/Mayrlab/scUTRquant-inputs
- name: scUTRquant-demo
  role: Demonstrates full end-to-end pipeline runs on GitHub-hosted runners, including
    Mambaforge/Snakemake installation and resource download
  repo: https://github.com/mfansler/scUTRquant-demo
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:6e8f45bdfe9ac95f4eaf6c808ca4e0c51665ab32c920b0de377a1b57a55da4af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/scUTRquant
  tools:
  - Conda
  - Snakemake
  - kallisto-bustools
  - scUTRquant
  - scUTRquant-demo
  - scUTRquant-inputs
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# snakemake-pipeline-configuration

## Summary

Configure and run the scUTRquant Snakemake pipeline (Conda or Singularity mode) by assembling a config.yaml and sample_sheet.csv so that kallisto bus pseudoaligns 10x 3'-end reads against a prebuilt UTRome index and bustools corrects barcodes, producing SingleCellExperiment count matrices.

## When to use

Use when raw 10x 3'-end FASTQ/BAM scRNA-seq inputs must be quantified for 3'UTR isoforms via the scUTRquant Snakemake pipeline, requiring a config.yaml with target, tech, strand, whitelist, and min_umis settings plus a sample_sheet.csv.

## When NOT to use

- The input is already a gene-by-cell UMI count matrix (e.g. a Cell Ranger filtered HDF5), not raw FASTQ/BAM reads needing pseudoalignment
- The data are full-length or 5'-end scRNA-seq libraries rather than 3'-end tag-based libraries, since scUTRquant is built for 3'-end tag data
- De novo mapping of mRNA 3' end cleavage sites is needed — most 10x reads do not span cleavage sites, and cleavage-site calling is handled by separate UTRome pipelines (hcl-utrome, mca-utrome)

## Inputs

- Raw 10x Genomics 3'-end scRNA-seq FASTQ files (R1/R2) or BAM files
- sample_sheet.csv with sample_id, file_type, and semicolon-separated read file lists per sample
- config.yaml specifying dataset_name, sample_file, output_type ['txs','genes'], target (utrome_hg38_v1/v2, utrome_mm10_v1/v2), tech ('10xv2'/'10xv3'), strand ('--fr-stranded'), bx_whitelist, and min_umis
- Barcode whitelist file for bustools correct
- Prebuilt UTRome kallisto index (from Mayrlab/scUTRquant-inputs resources)

## Outputs

- SingleCellExperiment objects with sparse 3'UTR isoform count matrices (*.txs.Rds and *.genes.Rds per target)
- BUS/MTX intermediate files
- HTML QC reports per sample
- Gene-level count matrices suitable for downstream Spearman correlation or isoform-fraction classification analyses

## How to apply

Assemble a Snakemake config.yaml and sample_sheet.csv listing sample_id, file_type, and semicolon-separated R1/R2 FASTQ (or BAM) files per sample. Set `target` to utrome_hg38_v1/v2 or utrome_mm10_v1/v2 to select the prebuilt UTRome kallisto index, set `tech` (e.g. '10xv2' or '10xv3') to match the 10x chemistry, set strand to '--fr-stranded' (required for all 10X 3'-end libraries), provide a `bx_whitelist` for bustools correct, set `output_type` to ['txs','genes'], and set a `min_umis` threshold for cell filtering. Execute with 'snakemake --use-conda --configfile examples/<dataset>/config.yaml' using Snakemake >= 6.0 and Conda with the libmamba solver (Conda >= v23.11 recommended; Miniforge or the prebuilt Docker image docker.io/mfansler/scutr-quant are supported alternatives). The pipeline pseudoaligns reads with kallisto bus, corrects cell barcodes, applies min_umis cell filtering, and emits per-target sparse 3'UTR isoform count matrices as SingleCellExperiment objects (*.txs.Rds and *.genes.Rds) plus BUS/MTX intermediates and HTML QC reports. Input sample sheets for published datasets are available in the scUTRquant-inputs repository, and end-to-end runnable examples are in mfansler/scUTRquant-demo.

## Related tools

- **scUTRquant** (Snakemake pipeline being configured; orchestrates kallisto bus pseudoalignment against a UTRome index and bustools barcode correction to quantify gene and 3'UTR isoform expression) — https://github.com/Mayrlab/scUTRquant
- **Snakemake** (Workflow engine invoked with --use-conda and a configfile; requires version >= 6.0)
- **Conda** (Installs pipeline software environments; Conda >= v23.11 with the libmamba solver recommended (installable via Miniforge)) — https://github.com/conda-forge/miniforge
- **kallisto-bustools** (Underlying quantification engine: kallisto bus pseudoaligns reads to the UTRome index; bustools correct validates cell barcodes against the whitelist)
- **scUTRquant-inputs** (Provides input sample sheets and scripts for downloading raw data used to run published datasets through the pipeline) — https://github.com/Mayrlab/scUTRquant-inputs
- **scUTRquant-demo** (Demonstrates full end-to-end pipeline runs on GitHub-hosted runners, including Mambaforge/Snakemake installation and resource download) — https://github.com/mfansler/scUTRquant-demo

## Examples

```
snakemake --use-conda --configfile examples/<dataset>/config.yaml
```

## Evaluation signals

- Pipeline completes without errors for all samples listed in sample_sheet.csv and emits both *.txs.Rds and *.genes.Rds SingleCellExperiment objects under the target folder (e.g. data/sce/utrome_mm10_v2/)
- HTML QC reports are generated per sample and retained BUS/MTX intermediates match the configured output_type ['txs','genes']
- Downstream validation: gene-level counts correlate with Cell Ranger 3.0.0 at Spearman rho > 0.99 per cell and ~0.92 per gene on the 10x demo datasets
- Cell-level filtering respects min_umis: cells below the threshold are absent from the final output
- Classification downstream of the configured run reproduces reported gene counts (mouse: 6766/16195 multi-UTR, 1869 IPA; human: 8129/16185 multi-UTR, 4113 IPA) matching Supplementary Data 3-4

## Limitations

- The strand setting '--fr-stranded' is required for all 10X 3'-end libraries; omitting it misquantifies reads
- `tech` must match the actual 10x chemistry ('10xv2' vs '10xv3') or pseudoalignment/barcode correction will be misconfigured
- Most 10x Genomics reads do not span mRNA 3' end cleavage sites, so the pipeline cannot perform exact de novo mapping of 3' ends — it quantifies against prebuilt UTRome targets
- Snakemake >= 6.0 and a recent Conda (>= v23.11 with libmamba solver) are recommended; older Conda installs may be slow or fail environment solves

## Evidence

- [methods] Config keys for scUTRquant: "`tech`: argument to `kallisto bus` indicating the scRNA-seq technology"
- [methods] Barcode whitelist key: "`bx_whitelist`: file of valid barcodes used in `bustools correct`"
- [methods] min_umis filter definition: "`min_umis`: minimum number of UMIs per cell; cells below this threshold are excluded"
- [methods] Conda version recommendation: "we strongly recommend that Conda be upgraded to at least v23.11 which uses the faster `libmamba` solver"
- [readme] Demo repository runs examples end to end: [evidence span withheld — anchor status: external_doc]
- [readme] Sample sheets provided for published data: "This repository provides input sample sheets for running data through 
[the scUTRquant pipeline](https://github.com/Mayrlab/scUTRquant)
as well as scripts for downloading the raw data."
