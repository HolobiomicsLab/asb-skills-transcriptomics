---
name: arabidopsis-transcriptome-index-usage
description: Use when raw 10x v3 scRNA-seq reads from Arabidopsis thaliana (e.g. SRA
  runs like SRR12046119/SRR12046120) need pseudoalignment against the TAIR10 cDNA+introns
  kallisto index to produce spliced/unspliced count matrices.
when_to_use_negative:
- Data are not from Arabidopsis thaliana — the cDNA_introns_10xv3.idx index and TAIR10
  annotation are species-specific, and downstream COPILOT annotation only supports
  Arabidopsis root.
- Input is already a spliced/unspliced gene-by-cell count matrix with column and row
  names — run COPILOT directly rather than re-aligning reads.
- Base-level genome alignments (BAM files) are required — kallisto is a pseudoaligner,
  so the output is count matrices rather than alignment files.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scKB
  role: Wrapper script that orchestrates kallisto pseudoalignment of the cDNA reads
    against cDNA_introns_10xv3.idx and bustools barcode counting
- name: kallisto
  role: Pseudoalignment engine aligning the cDNA reads to the TAIR10-based Arabidopsis
    transcriptome index
- name: bustools
  role: Barcode/UMI counting step that builds the raw spliced and unspliced gene-by-cell
    matrices
- name: COPILOT
  role: Downstream QC consumer — its sample.name must match the scKB output directory
    containing the spliced and unspliced matrices
  repo: https://github.com/Hsu-Che-Wei/COPILOT
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xpro.2022.101729
    title: Protocol for fast scRNA-seq raw data processing using scKB and non-arbitrary
      quality control with COPILOT
  source_content_hash: sha256:88c49a841987d3cbf5f37671cf1bf786b5f626d1fa315fc44ed17983ca87067f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ohlerlab/COPILOT
  repo_url: https://github.com/Hsu-Che-Wei/COPILOT
  tools:
  - COPILOT
  - bustools
  - kallisto
  - scKB
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# arabidopsis-transcriptome-index-usage

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs scKB, a wrapper script of kallisto and bustools, to pseudoalign Arabidopsis thaliana 10x Genomics v3 scRNA-seq reads against the TAIR10-based cDNA_introns_10xv3.idx index, yielding raw spliced and unspliced gene-by-cell UMI count matrices that feed COPILOT quality control.

## When to use

Use when raw 10x v3 scRNA-seq reads from Arabidopsis thaliana (e.g. SRA runs like SRR12046119/SRR12046120) need pseudoalignment against the TAIR10 cDNA+introns kallisto index to produce spliced/unspliced count matrices.

## When NOT to use

- Data are not from Arabidopsis thaliana — the cDNA_introns_10xv3.idx index and TAIR10 annotation are species-specific, and downstream COPILOT annotation only supports Arabidopsis root.
- Input is already a spliced/unspliced gene-by-cell count matrix with column and row names — run COPILOT directly rather than re-aligning reads.
- Base-level genome alignments (BAM files) are required — kallisto is a pseudoaligner, so the output is count matrices rather than alignment files.

## Inputs

- Raw 10x Genomics v3 scRNA-seq SRA runs dumped to FASTQ (e.g. SRR12046119 and SRR12046120 from GSM4626009, Arabidopsis col0, 386 million reads)
- FASTQs rearranged into the scKB-compatible layout: barcode/UMI reads separated from cDNA reads, in a directory named per the COPILOT sample.name convention
- Kallisto index cDNA_introns_10xv3.idx (TAIR10-based Arabidopsis transcriptome including introns)

## Outputs

- Gene-by-cell spliced UMI count matrix with row (gene) and column (cell barcode) names
- Gene-by-cell unspliced UMI count matrix with row (gene) and column (cell barcode) names
- Raw quality-filterable count matrices formatted as direct COPILOT input

## How to apply

Download the SRA runs (e.g. SRR12046119 and SRR12046120 from GSM4626009 for the 386-million-read col0 sample) and dump the reads to FASTQ. Convert the FASTQs into the scKB-compatible layout — barcode/UMI reads separated from cDNA reads — and place them in a directory whose name matches the COPILOT 'sample.name' convention. Run the scKB wrapper, which drives kallisto pseudoalignment of the cDNA reads against the cDNA_introns_10xv3.idx index (TAIR10-based Arabidopsis transcriptome with introns, built for 10x v3 chemistry) followed by bustools barcode counting. Export the gene-by-cell spliced and unspliced matrices with proper row (gene) and column (cell barcode) names, because COPILOT's spliced.mtx/unspliced.mtx arguments require matrices with column and row names and its documentation recommends the raw scKB output as COPILOT input. The rationale is speed: kallisto pseudoalignment makes scKB about ~30 times faster than CellRanger 3.1 in processing time, so the full col0 dataset completes in roughly 22 minutes instead of hours.

## Related tools

- **scKB** (Wrapper script that orchestrates kallisto pseudoalignment of the cDNA reads against cDNA_introns_10xv3.idx and bustools barcode counting)
- **kallisto** (Pseudoalignment engine aligning the cDNA reads to the TAIR10-based Arabidopsis transcriptome index)
- **bustools** (Barcode/UMI counting step that builds the raw spliced and unspliced gene-by-cell matrices)
- **COPILOT** (Downstream QC consumer — its sample.name must match the scKB output directory containing the spliced and unspliced matrices) — https://github.com/Hsu-Che-Wei/COPILOT

## Examples

```
copilot(sample.name="col0", mt.pattern="ATMG", cp.pattern="ATCG", transcriptome.name="TAIR10")
```

## Evaluation signals

- Run on the full col0 dataset (SRR12046119 + SRR12046120, 386 million reads) completes in roughly the reported ~22 minutes, consistent with scKB being ~30x faster than CellRanger 3.1
- Both spliced and unspliced gene-by-cell matrices are produced as quality-filterable count matrices carrying column (cell barcode) and row (gene) names
- Output directory name equals the COPILOT sample.name so copilot() consumes the raw scKB matrices without renaming
- Pseudoalignment used the cDNA_introns_10xv3.idx index and matrices carry TAIR10 gene identifiers consistent with transcriptome.name='TAIR10'
- Matrices load into COPILOT's spliced.mtx/unspliced.mtx arguments, which explicitly require matrices with column and row names

## Limitations

- The index and workflow are Arabidopsis thaliana/TAIR10-specific; other species require their own transcriptome index, and a customized genome may need conversion into a BSgenome object
- kallisto pseudoaligns rather than base-level aligns, so the output is count matrices — no genome BAM alignments are produced
- scKB can fail to run properly during the 'align raw reads to genome via scKB' step (article troubleshooting Problem 3)
- Downstream COPILOT annotation only supports cell type and developmental stage annotation for the root of Arabidopsis thaliana

## Evidence
