---
name: tss-dataset-quality-assessment
description: Use when you have a set of TSS annotations (e.g. from EPDnew/CAGE, csRNA-seq
  GSE221852, or RAMPAGE GSE89299) and want to judge dataset quality or compare TSS
  sources by the stringency of core promoter element positioning.
when_to_use_negative:
- Input is a raw sequencing dataset (FASTQ/BAM) without called TSS peaks — TSS peaks
  must first be called (e.g. with HOMER) before element positioning can be evaluated
- Sequences are not centered on a TSS (arbitrary genomic regions) — preferential-position-based
  quality assessment is only meaningful relative to an annotated TSS
- Searching a user-defined, non-core-promoter motif — ElemeNT 2023 supports this,
  but the positional-quality logic here applies to core promoter elements
edam_operation: http://edamontology.org/operation_0432
edam_topics:
- http://edamontology.org/topic_0749
tools:
- name: ElemeNT 2023
  role: Command-line (Perl) element detector run on TSS-centered promoter FASTA windows
    to identify core promoter elements and their positions/scores relative to the
    TSS
  repo: github.com/OritAdato/ElemeNT
- name: ElemeNT
  role: Provides the CORE database of predicted core promoter elements around annotated
    TSSs across 10 species, usable as reference for TSS-dataset evaluation
  repo: github.com/OritAdato/ElemeNT
- name: HOMER
  role: Used to call transcription start site peaks from csRNA-seq data before promoter
    sequences are extracted and scored
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae110
    title: 'ElemeNT 2023: an enhanced tool for detection and curation of core promoter
      elements'
  source_content_hash: sha256:f8a6bb064faaeceac796e1ad5c727d18a855bc83185fc7b71705d54e0d911b2a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ElemeNT
  - ElemeNT 2023
  - HOMER
  - Perl
  edam_operation: http://edamontology.org/operation_0432
  edam_topics:
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# tss-dataset-quality-assessment

## Summary

Assess the quality of high-throughput TSS datasets (CAGE, csRNA-seq, RAMPAGE) by running ElemeNT 2023 on promoter sequences centered on each TSS and checking whether core promoter elements (dInr, DPE, PB, TATA, Motif 1, dTCT) appear at their preferential positions relative to the TSS; strict positional preference indicates high-quality TSS input data.

## When to use

Use when you have a set of TSS annotations (e.g. from EPDnew/CAGE, csRNA-seq GSE221852, or RAMPAGE GSE89299) and want to judge dataset quality or compare TSS sources by the stringency of core promoter element positioning.

## When NOT to use

- Input is a raw sequencing dataset (FASTQ/BAM) without called TSS peaks — TSS peaks must first be called (e.g. with HOMER) before element positioning can be evaluated
- Sequences are not centered on a TSS (arbitrary genomic regions) — preferential-position-based quality assessment is only meaningful relative to an annotated TSS
- Searching a user-defined, non-core-promoter motif — ElemeNT 2023 supports this, but the positional-quality logic here applies to core promoter elements

## Inputs

- TSS annotations/peak coordinates from a high-throughput TSS dataset (e.g. EPDnew CAGE-based promoters, GEO csRNA-seq GSE221852, GEO RAMPAGE GSE89299)
- Reference genome assembly for the organism (e.g. D. melanogaster) to extract promoter windows
- Promoter sequences in FASTA format, ±100 bp centered on each annotated TSS

## Outputs

- ElemeNT 2023 element detection output per sequence (TATA, dInr, PB, DPE, GAGA, Motif 1, dTCT with positions and PWM scores)
- Positional distribution of each element relative to the TSS (fraction of transcripts per position, median PWM score per position)
- Cross-dataset element position stringency comparison table (e.g. element_position_stringency_comparison.csv)

## How to apply

Retrieve TSS annotations from the dataset of interest and extract ±100 bp genomic sequence windows centered on each TSS from the relevant genome assembly, formatted as FASTA. Run the ElemeNT 2023 command-line version (Perl, github.com/OritAdato/ElemeNT) to detect TATA, dInr, PB, DPE, GAGA, Motif 1 and dTCT. For each element, compute the fraction of transcripts containing it at each position relative to the TSS and the median PWM score at each position. Quality is judged by whether element-specific frequency peaks and median PWM score maxima coincide with reported preferential positions (dInr at −2, DPE at +28, Motif 1 at −5, dTCT at −2, PB at +26). To compare multiple TSS sources, compute the positional distributions per dataset (e.g. EPDnew/CAGE vs csRNA-seq vs RAMPAGE) and compare spatial stringency of dInr, PB, DPE and TATA positioning, writing the cross-dataset comparison to a table such as element_position_stringency_comparison.csv.

## Related tools

- **ElemeNT 2023** (Command-line (Perl) element detector run on TSS-centered promoter FASTA windows to identify core promoter elements and their positions/scores relative to the TSS) — github.com/OritAdato/ElemeNT
- **ElemeNT** (Provides the CORE database of predicted core promoter elements around annotated TSSs across 10 species, usable as reference for TSS-dataset evaluation) — github.com/OritAdato/ElemeNT
- **HOMER** (Used to call transcription start site peaks from csRNA-seq data before promoter sequences are extracted and scored)

## Examples

```
perl ElemeNT2023/src/ElemeNT/ElemeNT.pl --input drosophila_csRNAseq_TSS_pm100bp.fasta --output element_results.txt
```

## Evaluation signals

- Element-specific positional frequency peaks coincide with reported positions: dInr at −2, DPE at +28, Motif 1 at −5, dTCT at −2, PB at +26 relative to the TSS
- Median PWM score maxima for each element coincide with the same preferential positions (elevated scores at expected positions)
- In cross-dataset comparison, dInr/PB/DPE positional stringency is highest in csRNA-seq nascent TSSs (GSE221852), and TATA is more strictly positioned in RAMPAGE (GSE89299) than in csRNA-seq data
- Output table (element_position_stringency_comparison.csv) contains per-element position distributions for all compared TSS sources

## Limitations

- Positional-stringency patterns are organism- and dataset-specific; reported preferential positions (e.g. dInr −2, DPE +28) were established for D. melanogaster and may differ in other species
- Quality conclusions depend on upstream TSS peak calling; a poor peak-calling step (e.g. HOMER parameters) will propagate into misleading element-position profiles
- ElemeNT focuses on core promoter elements; searching arbitrary user-defined motifs is supported but is not the basis of this quality-assessment logic

## Evidence

- [abstract] quality assessment rationale: "which provides novel capabilities for assessing evolutionary conservation and for readily evaluating the quality of high-throughput transcription start site (TSS) datasets, leveraging preferential"
- [abstract] CORE database scope: "ElemeNT's CORE database, which contains predicted core promoter elements around annotated TSSs, is now expanded to cover 10 species"
- [intro] tool implementation: "ElemeNT is an interactive tool (implemented in Perl) for rapid and convenient detection of core promoter elements within any given sequences."
- [readme] command-line availability: "The source code and command line version of ElemeNT 2023 are available at https://github.com/OritAdato/ElemeNT."
