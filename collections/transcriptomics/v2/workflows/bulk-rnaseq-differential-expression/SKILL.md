---
name: bulk-rnaseq-differential-expression-workflow
description: 'Use when paired-end bulk RNA-seq reads must be adapter-trimmed, pseudoaligned
  to an external reference transcriptome, aggregated into raw gene counts, tested
  for differential expression, and interpreted with preranked pathway enrichment.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 4
  member_skills:
  - rna-seq-adapter-quality-trimming
  - wgs-read-quality-trimming
  - bulk-rnaseq-read-qc-trimming
  - fastq-adapter-and-quality-trimming
  - kallisto-reference-assembly-pseudoalignment
  - kallisto-pseudoalignment-quantification
  - transcriptome-reference-index-preparation
  - kallisto-bus-pseudoalignment-to-utrome
  - rna-seq-differential-expression-analysis--864ee06faecb526b
  - differential-gene-expression-analysis--40640222da726826
  - deg-method-consensus-combination
  - multi-method-deg-consensus-voting
  - gsea-prerank-pathway-enrichment-analysis
  - pathway-enrichment-estimation
  - gene-set-enrichment-analysis--1f9beeabb1328d9f
  - ssgsea-pathway-enrichment-scoring
  member_tools:
  - CCIVR
  - DESeq2
  - RSEM
  - STAR
  - Trim Galore!
  - ExpressAnalyst
  - ExpressAnalystR
  - Kallisto
  - Seq2Fun
  - Seq2Fun 2.0
  - Seq2Fun 2.0.2
  - limma
  - DEgenes Hunter
  - ExpHunter Suite
  - NOISeq
  - R
  - VEGA
  - gseapy
  - scanpy
  coverage_gaps: []
  derived_from_workflows: []
  bound_by: perspicacite-semantic
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/transcriptomics/v2
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# Bulk RNA-seq Quantification, Differential Expression, and Pathway Enrichment

## Summary

Trim reads, quantify and aggregate gene counts with Kallisto, derive consensus differential-expression calls and rankings, and run pathway enrichment.


## When to use

Use when paired-end bulk RNA-seq reads must be adapter-trimmed, pseudoaligned to an external reference transcriptome, aggregated into raw gene counts, tested for differential expression, and interpreted with preranked pathway enrichment.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — trim_bulk_reads

**Goal:** remove adapters and low-quality sequence from bulk RNA-seq reads

**EDAM operation:** http://edamontology.org/operation_3192

**Inputs:** paired-fastq/raw-bulk-rnaseq · **Outputs:** paired-fastq/trimmed-bulk-rnaseq

**Candidate leaf skills:** `rna-seq-adapter-quality-trimming` (primary), `wgs-read-quality-trimming`, `bulk-rnaseq-read-qc-trimming`, `fastq-adapter-and-quality-trimming`

**Tools (primary):** CCIVR, DESeq2, RSEM, STAR, Trim Galore!

**Other candidate tools:** BWA, BWA 0.7.17, GATK, Mutect2, SAMtools, TSNAD v2.0, Trimmomatic, Trimmomatic 0.39, BiocManager, R, Salmon, clusterProfiler, tximport, FastQC, ScanNeo2, Snakemake, fastp

**Grounding:** 4 KB(s); DOIs: 10.1016/j.csbj.2021.08.016, 10.1038/s41598-022-19782-5, 10.1080/2162402x.2020.1737369, 10.1093/bioinformatics/btad659

### Stage 2 — quantify_raw_gene_counts

**Goal:** pseudoalign reads and aggregate transcript estimates into raw gene counts

**EDAM operation:** http://edamontology.org/operation_3223

**Inputs:** paired-fastq/trimmed-bulk-rnaseq, fasta/reference-transcriptome, tsv/transcript-to-gene-map · **Outputs:** count-table/raw-gene-counts

**Candidate leaf skills:** `kallisto-reference-assembly-pseudoalignment` (primary), `kallisto-pseudoalignment-quantification`, `transcriptome-reference-index-preparation`, `kallisto-bus-pseudoalignment-to-utrome`

**Tools (primary):** ExpressAnalyst, ExpressAnalystR, Kallisto, Seq2Fun, Seq2Fun 2.0, Seq2Fun 2.0.2, limma

**Other candidate tools:** CanIsoNet, Conda, Snakemake, bustools, kallisto-bustools, scUTRquant

**Grounding:** 3 KB(s); DOIs: 10.1038/s41467-023-38785-y, 10.1038/s41467-024-48254-9, 10.1093/bioadv/vbad050

### Stage 3 — test_differential_expression

**Goal:** derive consensus differential-expression calls with per-gene logFC estimates

**EDAM operation:** http://edamontology.org/operation_3223

**Inputs:** count-table/raw-gene-counts, tsv/sample-design · **Outputs:** differential-expression-table/consensus-with-logfc

**Candidate leaf skills:** `rna-seq-differential-expression-analysis--864ee06faecb526b` (primary), `differential-gene-expression-analysis--40640222da726826`, `deg-method-consensus-combination`, `multi-method-deg-consensus-voting`

**Tools (primary):** DESeq2, DEgenes Hunter, ExpHunter Suite, NOISeq, R, limma

**Other candidate tools:** ExpHunterSuiteExamples, ExpHunter Suite (DEgenes Hunter pipeline), ExpHunterSuiteExamples scripts, Bioconductor

**Grounding:** 1 KB(s); DOIs: 10.1038/s41598-021-94343-w

### Stage 4 — test_pathway_enrichment

**Goal:** rank genes by their consensus logFC and identify pathways enriched at the extremes of that ranking

**EDAM operation:** http://edamontology.org/operation_2436

**Inputs:** differential-expression-table/consensus-with-logfc, gmt/pathway-gene-sets · **Outputs:** enrichment-table/preranked-pathways

**Candidate leaf skills:** `gsea-prerank-pathway-enrichment-analysis` (primary), `pathway-enrichment-estimation`, `gene-set-enrichment-analysis--1f9beeabb1328d9f`, `ssgsea-pathway-enrichment-scoring`

**Tools (primary):** VEGA, gseapy, scanpy

**Other candidate tools:** R, Snakemake, enrichplot, ggraph, pareg, tidyverse, condiments, fgsea, ggplot2, slingshot, tradeSeq, Docker, GSFM, Jupyter Notebook, Python 3.9.7, R 4.1.1

**Grounding:** 4 KB(s); DOIs: 10.1016/j.apsb.2024.08.017, 10.1038/s41467-021-26017-0, 10.1038/s41467-024-44823-0, 10.1093/bioinformatics/btad522

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
