---
name: neoantigen-prioritization-output-interpolation
description: Use when starting from raw WES/DNA-seq and RNA-seq FASTQ reads and you
  need per-sample predicted neoantigen output tables spanning SNVs, indels, alternative
  splicing, exitrons, and gene fusions.
when_to_use_negative:
- Input is already variant calls (VCFs) with no raw FASTQ — the workflow is designed
  for neoantigen prediction from raw sequencing data
- Only SNV/indel-derived neoantigens from a single alteration class are needed — a
  lighter SNV-only tool may suffice, whereas ScanNeo2 targets the multi-source setting
  (canonical/exitrion splicing, fusions, SNVs, indels)
- Data are long reads (Nanopore/PacBio) — the pipeline's fastp/BWA/STAR stack is built
  for short-read Illumina data
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: ScanNeo2
  role: Snakemake workflow orchestrating the full raw-reads-to-neoepitope-tables run
  repo: https://github.com/ylab-hi/ScanNeo2/
- name: fastp
  role: 'Pre-processing: quality filtering, adapter removal, minlen 10, 3''-end sliding-window
    trimming (wsize 3)'
  repo: https://github.com/OpenGene/fastp
- name: STAR
  role: Chimeric RNA-seq alignment (chimSegmentMin 20, chimScoreMin 10) preceding
    BWA realignment for fusion detection
  repo: https://github.com/alexdobin/STAR
- name: Arriba
  role: Gene fusion detection from STAR-realigned reads
  repo: https://github.com/suhrig/arriba
- name: ScanExitron
  role: Exitron-splicing event calling (ao 3, pso 0.05)
  repo: https://github.com/ylab-hi/ScanExitron
- name: SplAdder
  role: Alternative splicing event detection (confidence 3, iterations 5)
- name: Optitype
  role: HLA class I genotyping from yara-filtered HLA reads
- name: HLA-HD
  role: HLA class II genotyping
- name: IEDB
  role: T-cell epitope and immunogenicity prediction for MHC class I and II
- name: netMHCpan
  role: MHC-I binding affinity prediction for peptide lengths 8,9,10,11
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad659
    title: 'ScanNeo2: a comprehensive workflow for neoantigen detection and immunogenicity
      prediction from diverse genomic and transcriptomic alterations'
  source_content_hash: sha256:ec10fe13baa53eba02d54f518272457be3d93872047e980effdf3ad1acec9309
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ylab-hi/ScanNeo2
  tools:
  - Arriba
  - BWA
  - GATK Mutect2
  - HLA-HD
  - IEDB
  - Optitype
  - STAR
  - ScanExitron
  - ScanNeo2
  - SplAdder
  - fastp
  - netMHCpan
  - transIndel
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 20
---

# neoantigen prioritization output interpolation

## Summary

Running the ScanNeo2 Snakemake workflow end-to-end from raw FASTQ to generate per-sample predicted neoepitope tables (per-variant-type neoepitope files plus the combined <mhc_class>_neoepitopes_all.txt) under results/<sample>/prioritization/, validating that the pipeline reproduces the paper's high-throughput neoantigen prediction outputs.

## When to use

Use when starting from raw WES/DNA-seq and RNA-seq FASTQ reads and you need per-sample predicted neoantigen output tables spanning SNVs, indels, alternative splicing, exitrons, and gene fusions.

## When NOT to use

- Input is already variant calls (VCFs) with no raw FASTQ — the workflow is designed for neoantigen prediction from raw sequencing data
- Only SNV/indel-derived neoantigens from a single alteration class are needed — a lighter SNV-only tool may suffice, whereas ScanNeo2 targets the multi-source setting (canonical/exitrion splicing, fusions, SNVs, indels)
- Data are long reads (Nanopore/PacBio) — the pipeline's fastp/BWA/STAR stack is built for short-read Illumina data

## Inputs

- Raw paired-end WES/DNA-seq FASTQ files
- Raw RNA-seq FASTQ files
- Sample sheet (config/samples.tsv)
- Workflow configuration (config/config.yml with ENSEMBL release, mapq 30, basequal 20)
- Reference genome (globally ENSEMBL)

## Outputs

- results/<sample>/prioritization/<variant_type>_<mhc_class>_neoepitopes.txt (per variant type: SNV, indel, altsplicing, exitronsplicing, genefusion)
- results/<sample>/prioritization/<mhc_class>_neoepitopes_all.txt (combined per-class neoepitope table)
- results/<sample>/prioritization/<variant_type>_variant_effects.tsv
- results/<sample>/variants/ merged per-group VCFs
- mhc-I.tsv and mhc-II.tsv HLA genotyping tables

## How to apply

Clone ScanNeo2 with --recurse-submodules and create the conda/mamba environment from environment.yml, then configure config/config.yml (reference genome, ENSEMBL release, threads, mapq 30, basequal 20) pointing to the sample sheet at config/samples.tsv. Pre-process reads with fastp (minlen 10, 3'-end sliding-window trimming with wsize 3), align DNA-seq with BWA and RNA-seq with STAR in chimeric mode (chimSegmentMin 20, chimScoreMin 10) followed by BWA realignment. Type HLA alleles per sample — class I via yara filtering plus OptiType, class II via HLA-HD — writing mhc-I.tsv and mhc-II.tsv. Call variants per group: SplAdder (confidence 3, iterations 5) for alternative splicing, ScanExitron (ao 3, pso 0.05) for exitrons, Arriba for gene fusions, and GATK Mutect2/transIndel for SNVs, short and long indels, merging per-group VCFs under results/<sample>/variants/. Finally predict binding and immunogenicity via IEDB (netMHCpan/netMHCIIpan for MHC-I lengths 8,9,10,11 and MHC-II lengths 13,14,15) and write the per-variant-type <variant_type>_variant_effects.tsv and <variant_type>_<mhc_class>_neoepitopes.txt tables plus the combined <mhc_class>_neoepitopes_all.txt in results/<sample>/prioritization/. Note that MHC-II peptide length spans 13–25 aa in the peptide extraction stage, and stringent prioritization (immunogenicity score ≥0.5, TPM ≥2, ranking score ≥1000, VAF ≥0.02) can be applied downstream.

## Related tools

- **ScanNeo2** (Snakemake workflow orchestrating the full raw-reads-to-neoepitope-tables run) — https://github.com/ylab-hi/ScanNeo2/
- **fastp** (Pre-processing: quality filtering, adapter removal, minlen 10, 3'-end sliding-window trimming (wsize 3)) — https://github.com/OpenGene/fastp
- **STAR** (Chimeric RNA-seq alignment (chimSegmentMin 20, chimScoreMin 10) preceding BWA realignment for fusion detection) — https://github.com/alexdobin/STAR
- **Arriba** (Gene fusion detection from STAR-realigned reads) — https://github.com/suhrig/arriba
- **ScanExitron** (Exitron-splicing event calling (ao 3, pso 0.05)) — https://github.com/ylab-hi/ScanExitron
- **SplAdder** (Alternative splicing event detection (confidence 3, iterations 5))
- **Optitype** (HLA class I genotyping from yara-filtered HLA reads)
- **HLA-HD** (HLA class II genotyping)
- **IEDB** (T-cell epitope and immunogenicity prediction for MHC class I and II)
- **netMHCpan** (MHC-I binding affinity prediction for peptide lengths 8,9,10,11)

## Examples

```
git clone --recurse-submodules https://github.com/ylab-hi/ScanNeo2
```

## Evaluation signals

- results/<sample>/prioritization/<mhc_class>_neoepitopes_all.txt exists and contains detected neoepitopes with the documented columns
- All per-variant-type tables (<variant_type>_<mhc_class>_neoepitopes.txt and <variant_type>_variant_effects.tsv) are present for SNV, indel, altsplicing, exitronsplicing, and genefusion inputs
- mhc-I.tsv and mhc-II.tsv HLA genotyping files are produced for each sample
- MHC-I predicted peptide lengths fall in 8–11 aa and MHC-II extraction spans 13–25 aa
- The integration test dataset under .tests/integration reproduces without rule failures

## Limitations

- ScanNeo2 requires on average ~22.5% more CPU time than nextNEOpi due to its broader multi-source variant detection
- With default settings the pipeline captures many non-immunogenic peptides (476 alongside the 35/37 validated in TESLA); stringent filtering (immunogenicity score ≥0.5, TPM ≥2, ranking score ≥1000, VAF ≥0.02) trades recall (34/37) for precision (391 non-immunogenic)
- No specific Snakemake version is pinned beyond the ≥9.0.0 badge; netMHCpan/IEDB licensing may restrict reproducibility

## Evidence

- [results] the Immune Epitope Database and Analysis Resource (IEDB): "the Immune Epitope Database and Analysis Resource (IEDB) (Dhanda et al. 2019) is used to predict the T-cell epitope for MHC class I and II as well as the immunogenicity"
- [results] peptide lengths: "peptides of length 8–11 and 13–25 amino acids, corresponding to MHC class I and MHC class II peptides, respectively"
- [results] stringent parameters: "we applied ScanNeo2 with more stringent parameters (immunogenicity score ≥0.5, TPM ≥2, ranking score ≥1000, and VAF ≥0.02)"
- [methods] integration test data: "Can be found at `.tests/integration`"
- [abstract] availability: "ScanNeo2 is freely available at https://github.com/ylab-hi/ScanNeo2/ and is accompanied by instruction and application data."
