---
name: rnaseq-splice-aware-variant-calling
description: Use when the goal is to call splicing-derived (SplAdder/ScanExitron/Arriba)
  plus SNV/indel variants from raw DNA/RNA sequencing reads as inputs to downstream
  neoepitope prediction.
when_to_use_negative:
- Input is already a set of called variants (e.g. a VCF); use the downstream epitope-extraction/prioritization
  step instead of re-calling
- Only SNV/indel-derived neoantigens are of interest and no RNA-seq data exist — splicing
  and fusion callers (SplAdder, ScanExitron, Arriba) cannot contribute
- Reads are long-read (Nanopore/PacBio) — the pipeline's fastp/STAR/BWA short-read
  toolchain does not apply
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: SplAdder
  role: Detection of alternative splicing events from processed RNA-seq alignments
  repo: https://github.com/ylab-hi/ScanNeo2/
- name: ScanExitron
  role: Identification of exitron-splicing events with read-support (ao) and exon-exclusion-rate
    (pso) cutoffs
  repo: https://github.com/ylab-hi/ScanNeo2/
- name: Arriba
  role: Gene fusion detection on STAR-realigned RNA-seq reads
  repo: https://github.com/suhrig/arriba
- name: STAR
  role: Chimeric-mode RNA-seq alignment preceding fusion detection
  repo: https://github.com/alexdobin/STAR
- name: BWA
  role: Realignment of RNA-seq reads and direct alignment of DNA-seq reads against
    the reference genome
  repo: https://github.com/ylab-hi/ScanNeo2/
- name: GATK Mutect2/haplotypecaller
  role: Detection of short indels and SNVs
- name: transIndel
  role: Detection of long indel variants
- name: fastp
  role: Pre-processing of sequencing reads (quality filtering, adapter removal, minlen/wsize
    trimming)
  repo: https://github.com/OpenGene/fastp
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad659
    title: 'ScanNeo2: a comprehensive workflow for neoantigen detection and immunogenicity
      prediction from diverse genomic and transcriptomic alterations'
  source_content_hash: sha256:f0fb65a7902238148237faeff6bf059506553c805f6e85ac878bc3a580fdc27b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ylab-hi/ScanNeo2
  tools:
  - Arriba
  - BWA
  - GATK Mutect2
  - GATK Mutect2/haplotypecaller
  - HLA-HD
  - STAR
  - ScanExitron
  - ScanNeo2
  - SplAdder
  - fastp
  - netMHCpan
  - transIndel
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5789473684210527
n_claims: 19
---

# rnaseq-splice-aware-variant-calling

## Summary

Detects non-canonical and canonical variants from RNA-seq (and supporting DNA-seq) data as the variant-calling core of the ScanNeo2 neoantigen workflow, covering alternative splicing, exitron splicing, gene fusions, SNVs, and short/long indels.

## When to use

Use when the goal is to call splicing-derived (SplAdder/ScanExitron/Arriba) plus SNV/indel variants from raw DNA/RNA sequencing reads as inputs to downstream neoepitope prediction.

## When NOT to use

- Input is already a set of called variants (e.g. a VCF); use the downstream epitope-extraction/prioritization step instead of re-calling
- Only SNV/indel-derived neoantigens are of interest and no RNA-seq data exist — splicing and fusion callers (SplAdder, ScanExitron, Arriba) cannot contribute
- Reads are long-read (Nanopore/PacBio) — the pipeline's fastp/STAR/BWA short-read toolchain does not apply

## Inputs

- Raw paired-end DNA-seq and/or RNA-seq FASTQ files (Illumina)
- Sample sheet at config/samples.tsv
- Reference genome from a chosen ENSEMBL release
- config/config.yml with threads, mapq 30, basequal 20, and caller parameters

## Outputs

- Per-group merged VCFs under results/<sample>/variants/
- Alternative-splicing, exitron, and fusion event calls feeding epitope extraction
- Per-variant-type <variant_type>_variant_effects.tsv and <variant_type>_<mhc_class>_neoepitopes.txt tables in results/<sample>/prioritization/

## How to apply

Pre-process reads with fastp (minlen 10, 3'-end sliding-window trimming with wsize 3), then align RNA-seq with STAR in chimeric mode (chimSegmentMin 20, chimScoreMin 10) and realign with BWA; align DNA-seq directly with BWA. Call alternative splicing events per group with SplAdder (confidence 3, iterations 5), exitrons with ScanExitron (ao 3, pso 0.05), and gene fusions with Arriba (suppreads 2). Detect short indels and SNVs with GATK Mutect2/haplotypecaller and long indels with transIndel, merging per-group VCFs under results/<sample>/variants/. The rationale is that neoantigens arise from diverse sources beyond SNVs, so each variant class needs a dedicated caller with tuned read-support thresholds.

## Related tools

- **SplAdder** (Detection of alternative splicing events from processed RNA-seq alignments) — https://github.com/ylab-hi/ScanNeo2/
- **ScanExitron** (Identification of exitron-splicing events with read-support (ao) and exon-exclusion-rate (pso) cutoffs) — https://github.com/ylab-hi/ScanNeo2/
- **Arriba** (Gene fusion detection on STAR-realigned RNA-seq reads) — https://github.com/suhrig/arriba
- **STAR** (Chimeric-mode RNA-seq alignment preceding fusion detection) — https://github.com/alexdobin/STAR
- **BWA** (Realignment of RNA-seq reads and direct alignment of DNA-seq reads against the reference genome) — https://github.com/ylab-hi/ScanNeo2/
- **GATK Mutect2/haplotypecaller** (Detection of short indels and SNVs)
- **transIndel** (Detection of long indel variants)
- **fastp** (Pre-processing of sequencing reads (quality filtering, adapter removal, minlen/wsize trimming)) — https://github.com/OpenGene/fastp

## Examples

```
git clone --recurse-submodules https://github.com/ylab-hi/ScanNeo2 && snakemake --use-conda results/<sample>/variants
```

## Evaluation signals

- Merged per-group VCF files exist under results/<sample>/variants/ for each enabled variant type
- Per-variant-type tables <variant_type>_variant_effects.tsv and <variant_type>_<mhc_class>_neoepitopes.txt are written to results/<sample>/prioritization/
- Exitron calls meet the configured minimum supporting reads (ao 3) and exon-exclusion cutoff (pso 0.05)
- STAR ran in chimeric mode with chimSegmentMin 20 and chimScoreMin 10 so fusion candidates are available to Arriba
- On the TESLA benchmark, called variants ultimately yield 9152–96819 putative HLA-binding peptides per patient, indicating no variant source was silently dropped

## Limitations

- Frameshift-derived transcripts are only considered if they escape nonsense-mediated decay
- RNA-seq reads are filtered for mapping quality, duplicates, and unmapped reads, so low-quality alignments reduce sensitivity
- ScanNeo2 requires on average ~22.5% more CPU time than nextNEOpi, a cost of its multi-source variant calling

## Evidence

- [results] subjected to SplAdder (Kahles et al. 2016) for the detection of alternative splicing events, and ScanExitron for identifying exitron-splicing events: "subjected to SplAdder (Kahles et al. 2016) for the detection of alternative splicing events, and ScanExitron for identifying exitron-splicing events"
- [results] the aligned reads are first realigned using STAR (Dobin et al. 2013), and then subjected to Arriba: "the aligned reads are first realigned using STAR (Dobin et al. 2013), and then subjected to Arriba"
- [results] gatk ... is utilized for detection of short indels, as well as SNVs ... combined with transIndel (Yang et al. 2018) to detect long variants: [evidence span withheld — anchor status: not_found]
- [results] the sequencing reads can be pre-processed, which includes quality filtering or removal of adapter sequences: "the sequencing reads can be pre-processed, which includes quality filtering or removal of adapter sequences"
- [results] In the case of a frameshift, only the transcripts are further considered that escape nonsense-mediated decay: "In the case of a frameshift, only the transcripts are further considered that escape nonsense-mediated decay"
