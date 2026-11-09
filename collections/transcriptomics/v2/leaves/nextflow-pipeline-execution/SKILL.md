---
name: nextflow-pipeline-execution
description: Use when raw amplicon FASTQ data (population-level bulk or single-cell
  barcoding) must be processed through a reproducible, containerized Nextflow pipeline
  such as DaneVass/bartab, and execution profile/mode selection is needed.
when_to_use_negative:
- Data is not raw FASTQ — e.g. already a feature/barcode count matrix or a BAM needing
  no barcode re-quantification
- The analysis goal is downstream statistical analysis or visualization of barcode
  counts (bartools in R) rather than pre-processing/execution of the pipeline itself
- Barcode design is incompatible with the pipeline's parameter space (e.g. evolving
  barcode methods, which BARtab/bartools currently do not officially support)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Nextflow
  role: Workflow engine used to execute the BARtab pipeline reproducibly across environments
  repo: https://github.com/DaneVass/BARtab
- name: BARtab
  role: Target Nextflow pipeline tabulating synthetic barcode counts from NGS data
  repo: https://github.com/DaneVass/BARtab
- name: fastp
  role: Quality filtering module within the pipeline (FILTER_READS, minqual 20, pctqual
    80)
  repo: https://github.com/OpenGene/fastp
- name: cutadapt
  role: Constant-region trimming module (CUTADAPT_READS)
- name: MultiQC
  role: Aggregates per-step QC metrics into a single report for validation
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_006
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:d1980017944fa12f378954badf7aa27e91e4f42aaca461d4216eb16a39d8c347
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DaneVass/BARtab
  tools:
  - BARtab
  - FLASh
  - FastQC
  - MultiQC
  - Nextflow
  - R
  - bartools
  - bowtie1
  - cutadapt
  - fastp
  - pycashier
  - samtools
  - starcode
  - timemachine
  - umi-tools
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.85
n_claims: 20
---

# nextflow-pipeline-execution

## Summary

Executes portable, scalable Nextflow workflows such as BARtab on NGS barcode data, selecting the correct run mode (paired-bulk, single-bulk reference-free, single-cell) and conda/docker/singularity plus slurm/lsf profiles to generate barcode counts tables from FASTQ inputs.

## When to use

Use when raw amplicon FASTQ data (population-level bulk or single-cell barcoding) must be processed through a reproducible, containerized Nextflow pipeline such as DaneVass/bartab, and execution profile/mode selection is needed.

## When NOT to use

- Data is not raw FASTQ — e.g. already a feature/barcode count matrix or a BAM needing no barcode re-quantification
- The analysis goal is downstream statistical analysis or visualization of barcode counts (bartools in R) rather than pre-processing/execution of the pipeline itself
- Barcode design is incompatible with the pipeline's parameter space (e.g. evolving barcode methods, which BARtab/bartools currently do not officially support)

## Inputs

- Paired-end or single-end amplicon FASTQ files of synthetic lineage barcodes (e.g. 22-sample Goyal dataset or FM0-2 FateMap single-cell data)
- Cell barcode whitelist TSV files (<sample_id>_whitelist.tsv, '-1' suffixes removed) for single-cell mode
- Barcode reference library FASTA (e.g. SPLINTR_mCHERRY_V2_barcode_reference_library.fasta) for reference-based modes
- Nextflow pipeline repository (DaneVass/bartab) and a chosen execution profile (conda/docker/singularity + slurm/lsf)

## Outputs

- Per-sample starcode consensus counts files (starcode/<sample_id>_starcode.tsv) and a combined barcode-by-sample counts table (counts/all_counts_combined.tsv) in bulk mode
- Single-cell outputs: counts/<sample>_mapped_barcodes.tsv, <sample>.counts.tsv (barcode, cell barcode, deduplicated UMI count), and <sample>_cell_barcode_annotation.tsv
- MultiQC report summarizing fastqc, flash, fastp, cutadapt, bowtie QC metrics
- QC plots such as counts/qc/<sample>_barcodes_per_cell.pdf

## How to apply

Select the pipeline mode matching the input: --mode paired-bulk for paired-end FASTQ (merging with FLASh at mergeoverlap = 10), single-bulk reference-free mode for population-level data (FASTQC, fastp with minqual 20 and pctqual 80, cutadapt trimming with constantmismatches 0.1 and min_readlength 20), or --mode single-cell --input_type fastq for single-cell data with whitelist_indir cell-barcode whitelists. Resolve dependencies and compute environments via profiles: choose conda, docker, or singularity (e.g. docker image henriettaholze/bartab:v1.4) and slurm or lsf schedulers as appropriate. Validate the installation and parameters first with `nextflow run DaneVass/bartab --help`, then run a deposited test dataset (test_SE, test_PE, test_SE_ref_free, test_sc, test_sc_bam, test_sc_saw_fastq) before the real data. Monitor pipeline outputs per module — merged reads, fastp-filtered reads, cutadapt-trimmed reads, starcode consensus counts, and the MultiQC report — to confirm each processing stage completed.

## Related tools

- **Nextflow** (Workflow engine used to execute the BARtab pipeline reproducibly across environments) — https://github.com/DaneVass/BARtab
- **BARtab** (Target Nextflow pipeline tabulating synthetic barcode counts from NGS data) — https://github.com/DaneVass/BARtab
- **fastp** (Quality filtering module within the pipeline (FILTER_READS, minqual 20, pctqual 80)) — https://github.com/OpenGene/fastp
- **cutadapt** (Constant-region trimming module (CUTADAPT_READS))
- **MultiQC** (Aggregates per-step QC metrics into a single report for validation)

## Examples

```
nextflow run DaneVass/bartab -profile test_PE,docker,slurm --mode paired-bulk
```

## Evaluation signals

- A successful `nextflow run DaneVass/bartab --help` call resolves parameters and confirms the pipeline is installed and reachable
- Test profiles (test_SE, test_PE, test_sc, etc.) complete and reproduce the deposited exemplar outputs, e.g. barcode-by-sample counts matching test_PE run output
- Expected artifacts exist per mode: starcode/<sample_id>_starcode.tsv and counts/all_counts_combined.tsv in bulk; test_sc.counts.tsv with the three required columns in single-cell mode
- MultiQC report shows non-trivial metrics at every stage (fastqc, flash, fastp, cutadapt, bowtie), confirming each module executed rather than passing through empty data
- Runtime/memory should be in the range reported for comparable inputs (e.g. BARtab 35 min / 6.83 GB on the 22-sample bulk dataset)

## Limitations

- Benchmarking against similar tools was difficult due to the lack of a gold-standard dataset with known barcode abundance and differing parameter flexibility between tools
- BARtab and bartools currently do not officially support evolving barcode methods
- Resource and runtime profiles vary strongly by mode: bulk population-level runs are far cheaper (35 min / 6.83 GB) than competitor workflows, but scheduler/container profiles (slurm/lsf, conda/docker/singularity) must be available on the target system

## Evidence

- [abstract] BARtab, a portable and scalable Nextflow pipeline: "we present BARtab, a portable and scalable Nextflow pipeline"
- [methods] Run test datasets via profiles: "Run any of the test datasets using `nextflow run DaneVass/bartab -profile <test_SE,test_PE,test_SE_ref_free,test_sc,test_sc_bam,test_sc_saw_fastq>,<conda,docker,singularity>,<slurm,lsf>`"
- [methods] Docker image for BARtab: "docker pull henriettaholze/bartab:v1.4"
- [methods] Help invocation: "nextflow run DaneVass/bartab --help"
- [methods] Per-mode module routing: "If running in mode paired-bulk, forward and reverse reads are merged using [FLASh](http://ccb.jhu.edu/software/FLASH/)."
- [methods] Software check first: "Software check is always performed as first module."
