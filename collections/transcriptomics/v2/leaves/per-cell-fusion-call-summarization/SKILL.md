---
name: per-cell-fusion-call-summarization
description: Use when a scRNA-seq dataset (e.g., K562 Smart-seq2, SRP291312) has been
  run through fusion detectors per cell and calls must be aggregated per cell per
  tool for benchmarking.
when_to_use_negative:
- Input is bulk RNA-seq data with no per-cell structure, where a single-sample call
  summary suffices
- The analysis aims to discover novel fusion events rather than summarize targeted
  (anchored-gene) fusion calls across cells
- Calls involve fusion genes that do not include the user-provided target gene, which
  Anchored-fusion cannot output
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: Anchored-fusion
  role: Primary targeted fusion detector run per cell with --file_anchored_cds set
    to the BCR transcript FASTA and HVLD filter enabled; its per-cell calls are the
    benchmark headline
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: STAR-Fusion v1.12.0
  role: Competing de novo fusion detector run on the same per-cell FASTQs for call
    comparison
  repo: https://github.com/STAR-Fusion
- name: FusionInspector v2.8.0
  role: Competing fusion detector applied per cell to the K562 dataset
  repo: https://github.com/FusionInspector/FusionInspector
- name: FusionCatcher v1.30
  role: Competing fusion detector applied per cell to the K562 dataset
  repo: https://github.com/ndaniel/fusioncatcher
- name: Arriba v2.4.0
  role: Competing fusion detector applied per cell to the K562 dataset
  repo: https://github.com/suhrig/arriba
- name: scFusion
  role: Competing scRNA-seq fusion detector applied per cell to the K562 dataset
  repo: https://github.com/XiDsLab/scFusion
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:a55e3e64b0fe72b6336436d4bcd014c0dae0c743910f6d8c3a406dc526d8f297
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ShenLab-Genomics/Anchored-Fusion
  repo_url: https://github.com/ShenLab-Genomics/Anchored-Fusion
  tools:
  - Anchored-fusion
  - Arriba
  - Arriba v2.4.0
  - BLAT
  - BWA-MEM
  - FusionCatcher
  - FusionCatcher v1.30
  - FusionInspector
  - FusionInspector v2.8.0
  - STAR-Fusion
  - STAR-Fusion v1.12.0
  - scFusion
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# per-cell-fusion-call-summarization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tabulate targeted fusion-gene (e.g., BCR-ABL1) positive/negative calls per single cell across multiple detection tools, producing per-cell call matrices and evidence-level stratifications used to benchmark Anchored-fusion against competing methods on scRNA-seq data.

## When to use

Use when a scRNA-seq dataset (e.g., K562 Smart-seq2, SRP291312) has been run through fusion detectors per cell and calls must be aggregated per cell per tool for benchmarking.

## When NOT to use

- Input is bulk RNA-seq data with no per-cell structure, where a single-sample call summary suffices
- The analysis aims to discover novel fusion events rather than summarize targeted (anchored-gene) fusion calls across cells
- Calls involve fusion genes that do not include the user-provided target gene, which Anchored-fusion cannot output

## Inputs

- Per-cell paired-end scRNA-seq FASTQ files (350-cell K562 Smart-seq2, SRA SRP291312)
- Fusion detection outputs per cell from Anchored-fusion, STAR-Fusion v1.12.0, FusionInspector v2.8.0, FusionCatcher v1.30, Arriba v2.4.0, and scFusion
- Cell-level supporting-evidence annotations (evidence-bearing vs. very-low-evidence categories)

## Outputs

- Per-cell per-tool BCR-ABL1 positive/negative call table (350 cells x 6 tools)
- Stratified summary of BCR-ABL1-positive counts by evidence level per tool
- Benchmark comparison showing Anchored-fusion's 35/350 positive calls vs. competitor counts

## How to apply

After running each tool (Anchored_Fusion.py, STAR-Fusion v1.12.0, FusionInspector v2.8.0, FusionCatcher v1.30, Arriba v2.4.0, scFusion) on every cell's paired-end FASTQs, parse each tool's output to extract whether the target fusion (BCR-ABL1) is called in each cell. Tabulate a positive/negative call per cell per tool, then stratify cells by supporting-evidence level (e.g., 42 evidence-bearing cells and a 15-cell very-low-evidence category). Validate the summary against expected benchmarks: Anchored-fusion should recover 35 BCR-ABL1-positive cells out of 350, corresponding to 83.3% recovery of the 42 evidence-bearing cells and detection in two-thirds (10/15) of very-low-evidence cells; compare other tools' counts to confirm they report lower numbers.

## Related tools

- **Anchored-fusion** (Primary targeted fusion detector run per cell with --file_anchored_cds set to the BCR transcript FASTA and HVLD filter enabled; its per-cell calls are the benchmark headline) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **STAR-Fusion v1.12.0** (Competing de novo fusion detector run on the same per-cell FASTQs for call comparison) — https://github.com/STAR-Fusion
- **FusionInspector v2.8.0** (Competing fusion detector applied per cell to the K562 dataset) — https://github.com/FusionInspector/FusionInspector
- **FusionCatcher v1.30** (Competing fusion detector applied per cell to the K562 dataset) — https://github.com/ndaniel/fusioncatcher
- **Arriba v2.4.0** (Competing fusion detector applied per cell to the K562 dataset) — https://github.com/suhrig/arriba
- **scFusion** (Competing scRNA-seq fusion detector applied per cell to the K562 dataset) — https://github.com/XiDsLab/scFusion

## Examples

```
python Anchored_Fusion.py --file_anchored_cds=BCR_transcript.fasta --fastq1=cell_001_1.fastq.gz --fastq2=cell_001_2.fastq.gz --out_folder=output_dir --file_ref_seq=hg38.fa --file_ref_ann=hg38.gtf
```

## Evaluation signals

- Anchored-fusion detects exactly 35 BCR-ABL1-positive cells out of 350 K562 Smart-seq2 cells
- Per-cell calls cover 83.3% recovery of the 42 evidence-bearing cells
- Detection in 10 of 15 (two-thirds) very-low-evidence cells
- Every tool's per-cell call count is populated for all 350 cells (no missing cells in the table)
- Other competing tools report lower BCR-ABL1-positive cell counts than Anchored-fusion

## Limitations

- Anchored-fusion only detects fusions that include the user-specified target gene; other fusions are absent from outputs and thus from the summary
- Anchored-fusion requires paired-end RNA-seq data and cannot be applied to single-end scRNA-seq libraries
- If the per-cell dataset is too small to retrain the HVLD filter, default model parameters from GitHub must be used, which may cause deviations in per-cell calls

## Evidence

- [results] Anchored-fusion detection of K562 single cells: [evidence span withheld — anchor status: not_found]
- [results] Tabulate per-cell calls across tools: [evidence span withheld — anchor status: not_found]
