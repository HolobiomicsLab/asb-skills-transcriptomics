---
name: snakemake-workflow-configuration-and-execution
description: Use when the goal is to run neoantigen prediction end-to-end from raw
  FASTQ (WES and/or RNA-seq) and produce per-sample predicted neoepitope output tables
  under results/<sample>/prioritization/.
when_to_use_negative:
- Input is already a list of variants or VCFs and only the IEDB/netMHCpan epitope
  prediction step is needed — the full raw-data workflow is unnecessary
- Reads are long reads (PacBio/Nanopore); fastp and the short-read alignment stack
  are designed for Illumina short reads (NovaSeq/MGI)
- No conda/mamba or Snakemake ≥9.0.0 environment is available, since installation
  depends on environment.yml and the Snakemake engine
tools:
- name: Snakemake
  role: Workflow engine whose DAG rules define and execute the entire ScanNeo2 pipeline;
    requires snakemake ≥9.0.0 per the repository badge
- name: mamba
  role: Package manager used to create the workflow environment from environment.yml
  repo: https://github.com/mamba-org/mamba
- name: ScanNeo2
  role: The Snakemake workflow being configured and executed; cloned with --recurse-submodules
  repo: https://github.com/ylab-hi/ScanNeo2/
- name: fastp
  role: 'First rule in the DAG: quality filtering/adapter trimming of raw FASTQ (minlen,
    wsize sliding window)'
  repo: https://github.com/OpenGene/fastp
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad659
    title: 'ScanNeo2: a comprehensive workflow for neoantigen detection and immunogenicity
      prediction from diverse genomic and transcriptomic alterations'
  source_content_hash: sha256:ab2894e6d823af6ed8dd5f7c43e064e0dea06bec3a20e42a2c9884c60a6e7c4f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mamba-org/mamba
  tools:
  - Arriba
  - BWA
  - GATK Mutect2
  - HLA-HD
  - STAR
  - ScanExitron
  - ScanNeo2
  - Snakemake
  - SplAdder
  - fastp
  - mamba
  - netMHCpan
  - transIndel
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# snakemake-workflow-configuration-and-execution

## Summary

Configure and execute the ScanNeo2 Snakemake workflow (requires snakemake ≥9.0.0) so that raw WES/RNA-seq FASTQ data is processed end-to-end — from fastp pre-processing through HLA typing, variant calling, and IEDB epitope prediction — into per-sample neoepitope tables.

## When to use

Use when the goal is to run neoantigen prediction end-to-end from raw FASTQ (WES and/or RNA-seq) and produce per-sample predicted neoepitope output tables under results/<sample>/prioritization/.

## When NOT to use

- Input is already a list of variants or VCFs and only the IEDB/netMHCpan epitope prediction step is needed — the full raw-data workflow is unnecessary
- Reads are long reads (PacBio/Nanopore); fastp and the short-read alignment stack are designed for Illumina short reads (NovaSeq/MGI)
- No conda/mamba or Snakemake ≥9.0.0 environment is available, since installation depends on environment.yml and the Snakemake engine

## Inputs

- Raw paired-end sequencing FASTQ files (DNA-seq/WES and RNA-seq) for each sample
- Sample sheet at config/samples.tsv listing samples and read groups
- config/config.yml specifying reference genome, ENSEMBL release, threads, mapq 30, basequal 20
- environment.yml for building the conda/mamba execution environment

## Outputs

- results/<sample>/prioritization/<mhc_class>_neoepitopes_all.txt (combined per-sample neoepitope table)
- results/<sample>/prioritization/<variant_type>_variant_effects.tsv and <variant_type>_<mhc_class>_neoepitopes.txt per variant type
- results/<sample>/variants/ per-group merged VCFs (SNVs, indels, splicing, exitrons, fusions)
- mhc-I.tsv and mhc-II.tsv HLA genotyping results

## How to apply

Clone the ScanNeo2 repository with --recurse-submodules (submodules pull in tools like ScanExitron and Arriba) and create the environment from environment.yml using conda/mamba. Edit config/config.yml to set the reference genome and ENSEMBL `release`, thread count, and quality thresholds mapq 30 and basequal 20, and point it at the sample sheet config/samples.tsv. The workflow then chains fastp pre-processing (minlen, 3'-end sliding-window wsize trimming), BWA/STAR (chimeric mode, chimSegmentMin 20, chimScoreMin 10) alignment, HLA genotyping (yara+OptiType for class I, HLA-HD for class II), per-group variant calling (SplAdder confidence 3, ScanExitron ao 3/pso 0.05, Arriba suppreads 2, GATK Mutect2/transIndel), and netMHCpan/netMHCIIpan/IEDB prediction for MHC-I lengths 8,9,10,11 and MHC-II lengths 13,14,15. Snakemake's DAG resolves rule dependencies and parallelization via the configured threads, writing results under results/<sample>/variants/ and results/<sample>/prioritization/. The rationale is that a single declarative Snakefile guarantees the multi-source variant integration (SNVs, indels, alternative splicing, exitrons, fusions) is executed reproducibly rather than as ad-hoc scripts.

## Related tools

- **Snakemake** (Workflow engine whose DAG rules define and execute the entire ScanNeo2 pipeline; requires snakemake ≥9.0.0 per the repository badge)
- **mamba** (Package manager used to create the workflow environment from environment.yml) — https://github.com/mamba-org/mamba
- **ScanNeo2** (The Snakemake workflow being configured and executed; cloned with --recurse-submodules) — https://github.com/ylab-hi/ScanNeo2/
- **fastp** (First rule in the DAG: quality filtering/adapter trimming of raw FASTQ (minlen, wsize sliding window)) — https://github.com/OpenGene/fastp

## Examples

```
git clone --recurse-submodules https://github.com/ylab-hi/ScanNeo2 && cd ScanNeo2 && mamba env create -f environment.yml
```

## Evaluation signals

- results/<sample>/prioritization/<mhc_class>_neoepitopes_all.txt exists for every sample in config/samples.tsv and contains the documented output columns
- Per-variant-type files (<variant_type>_variant_effects.tsv, <variant_type>_<mhc_class>_neoepitopes.txt) were generated for SNVs, indels, alternative splicing, exitrons, and fusions
- mhc-I.tsv and mhc-II.tsv were produced from HLA genotyping (OptiType/HLA-HD) before prediction rules ran
- Snakemake completed the full DAG without rule failures, honoring config.yml values (mapq 30, basequal 20, ENSEMBL release)

## Limitations

- The paper text does not pin a specific Snakemake version (the repository badge requires ≥9.0.0, so older Snakemake installs may fail)
- Reference-dependent: the workflow uses the ENSEMBL genome globally, and HLA-HD requires its own reference data (freq_data, HLA_gene.split.txt, dictionary); IEDB TSVs were downloaded on a specific date (03-05-24)
- ScanNeo2 required on average ~22.5% more CPU time than nextNEOpi in the benchmark, reflecting the cost of integrating multiple variant sources
- No software license is mentioned in the paper text (the GitHub badge indicates MIT)

## Evidence

- [methods] Clone command documented in the methods for installing the workflow: "git clone --recurse-submodules https://github.com/ylab-hi/ScanNeo2"
- [discussion] Rationale for using Snakemake as the workflow engine: "It has been implemented in snakemake to ensure ease of installation, usage as well as high portability."
- [results] Availability and documentation of the workflow: "ScanNeo2 is freely available at https://github.com/ylab-hi/ScanNeo2/ and is accompanied by instruction and application data."
- [methods] Configuration parameter for the reference genome release: "The `release` option corresponds to the ENSEMBL release version"
