---
name: exitron-splicing-event-detection
description: Use when RNA-seq alignments are available and the goal is to capture
  neoantigen sources beyond SNVs/indels, specifically exitron-splicing events that
  most neoantigen-prediction tools overlook.
when_to_use_negative:
- The variant of interest is a long indel or gene fusion — use the transIndel or Arriba
  modules instead, not the exitron module
- Only DNA-seq data is available with no RNA-seq alignment, since exitron splicing
  is called exclusively from RNA-seq
- The goal is canonical alternative splicing (exon skipping etc.) rather than exitrons
  — the SplAdder module covers that source
edam_operation: http://edamontology.org/operation_0231
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ScanExitron
  role: Detects exitron-splicing events from the RNA-seq alignment using ao/pso/strand
    thresholds
  repo: https://github.com/ylab-hi/ScanExitron
- name: ScanNeo2
  role: Snakemake workflow that wraps ScanExitron, converts its output to VCF, annotates
    GRP/SRC keys, and merges per-group calls into exitrons.vcf.gz
  repo: https://github.com/ylab-hi/ScanNeo2/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad659
    title: 'ScanNeo2: a comprehensive workflow for neoantigen detection and immunogenicity
      prediction from diverse genomic and transcriptomic alterations'
  source_content_hash: sha256:5994d4e75f9bfb40288936949ae86d9d83047028b3998a3d147eda639ab8d04f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ylab-hi/ScanExitron
  tools:
  - Arriba
  - GATK Mutect2
  - ScanExitron
  - ScanNeo2
  - SplAdder
  - transIndel
  edam_operation: http://edamontology.org/operation_0231
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# exitron-splicing-event-detection

## Summary

Detects exitron-splicing events from RNA-seq alignments using ScanExitron within ScanNeo2's variant-calling stage, converting exitron calls into VCF records annotated with GRP and SRC=exitron INFO keys for downstream neoantigen extraction.

## When to use

Use when RNA-seq alignments are available and the goal is to capture neoantigen sources beyond SNVs/indels, specifically exitron-splicing events that most neoantigen-prediction tools overlook.

## When NOT to use

- The variant of interest is a long indel or gene fusion — use the transIndel or Arriba modules instead, not the exitron module
- Only DNA-seq data is available with no RNA-seq alignment, since exitron splicing is called exclusively from RNA-seq
- The goal is canonical alternative splicing (exon skipping etc.) rather than exitrons — the SplAdder module covers that source

## Inputs

- Aligned RNA-seq BAM from the ScanNeo2 alignment stage
- config/config.yml with the exitronsplicing module activation flag and ao/pso/strand parameters
- Sample sheet grouping samples into groups

## Outputs

- <group>.exitron intermediate calls per group
- Per-group exitron VCFs with GRP and SRC=exitron INFO keys
- Merged exitrons.vcf.gz under results/<sample>/variants/

## How to apply

Gate the module on its `activate` flag in config/config.yml and run ScanExitron on the aligned RNA-seq BAM with ao=3 (minimum number of reads supporting the exitron), pso=0.05 (minimum cutoff for the exon-exclusion rate), and strand=1. Convert the resulting <group>.exitron output to VCF format, then augment each record with GRP (group) and SRC=exitron INFO keys. Sort the per-group VCFs and merge them into exitrons.vcf.gz under results/<sample>/variants/ alongside the other variant-type VCFs (altsplicing, long.indels, somatic SNVs/short indels, fusions). Exitron events complement canonical alternative splicing (SplAdder) because existing tools 'often overlook other sources of neoantigens like gene fusions or alternative' splicing.

## Related tools

- **ScanExitron** (Detects exitron-splicing events from the RNA-seq alignment using ao/pso/strand thresholds) — https://github.com/ylab-hi/ScanExitron
- **ScanNeo2** (Snakemake workflow that wraps ScanExitron, converts its output to VCF, annotates GRP/SRC keys, and merges per-group calls into exitrons.vcf.gz) — https://github.com/ylab-hi/ScanNeo2/

## Evaluation signals

- exitrons.vcf.gz exists under results/<sample>/variants/ when the exitron module's `activate` flag is set
- Every record in exitrons.vcf.gz carries GRP (group) and SRC=exitron INFO keys
- Exitron calls respect the configured thresholds: at least ao=3 supporting reads and exon-exclusion rate ≥ pso=0.05
- Per-group <group>.exitron intermediates exist and each converted VCF is coordinate-sorted before merging

## Limitations

- Exitron detection requires RNA-seq; exitrons invisible in the transcriptome or below the ao=3 / pso=0.05 cutoffs are missed
- READMEs do not state a license or pinned tool version for ScanExitron; version drift may change output format
- This module covers only exitron splicing — other neoantigen sources (canonical splicing, fusions, indels/SNVs) need the other ScanNeo2 modules

## Evidence

- [methods] Exitron events are called using [ScanExitron](https://github.com/ylab-hi/ScanExitron): "Exitron events are called using [ScanExitron](https://github.com/ylab-hi/ScanExitron)"
- [methods] the minimum number of reads that support the exitron: "The former describes the minimum number of reads that support the exitron"
- [methods] the minimum cutoff for the exon-exclusion rate: "the latter the minimum cutoff for the exon-exclusion rate"
- [results] and ScanExitron for identifying exitron-splicing events: "subjected to SplAdder (Kahles et al. 2016) for the detection of alternative splicing events, and ScanExitron for identifying exitron-splicing events"
- [readme] A computational workflow for exitron splicing identification from RNA-Seq data.: [evidence span withheld — anchor status: external_doc]
