---
name: tss-coordinate-sequence-extraction
description: Use when you have a set of TSS coordinates (e.g. EPDnew, GSE221852 csRNA-seq,
  GSE89299 RAMPAGE) and need promoter windows around each TSS as FASTA for motif-position
  analysis.
when_to_use_negative:
- Input sequences are already provided as FASTA windows aligned on the TSS — re-extraction
  adds no value
- The goal is de novo motif discovery in arbitrary sequences not tied to TSS coordinates
edam_operation: http://edamontology.org/operation_0380
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0749
tools:
- name: ElemeNT 2023
  role: Downstream consumer of the extracted FASTA windows; detects core promoter
    elements relative to the TSS
  repo: github.com/OritAdato/ElemeNT
- name: ElemeNT
  role: Interactive Perl tool for rapid detection of core promoter elements within
    given sequences
  repo: github.com/OritAdato/ElemeNT
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae110
    title: 'ElemeNT 2023: an enhanced tool for detection and curation of core promoter
      elements'
  source_content_hash: sha256:a66ddcc1e7b33d00c71a6c7221fe702f6823845bfd589ce9f9f1fadf72969e30
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ElemeNT
  - ElemeNT 2023
  - Perl
  edam_operation: http://edamontology.org/operation_0380
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# tss-coordinate-sequence-extraction

## Summary

Extracts promoter sequence windows centered on each annotated TSS coordinate and formats them as FASTA input for ElemeNT core promoter element detection, enabling cross-dataset TSS source comparisons such as EPDnew/CAGE vs csRNA-seq vs RAMPAGE.

## When to use

Use when you have a set of TSS coordinates (e.g. EPDnew, GSE221852 csRNA-seq, GSE89299 RAMPAGE) and need promoter windows around each TSS as FASTA for motif-position analysis.

## When NOT to use

- Input sequences are already provided as FASTA windows aligned on the TSS — re-extraction adds no value
- The goal is de novo motif discovery in arbitrary sequences not tied to TSS coordinates

## Inputs

- TSS coordinate annotations from EPDnew (CAGE-based promoters)
- TSS coordinates from high-throughput TSS datasets (e.g. GEO csRNA-seq GSE221852, RAMPAGE GSE89299)
- Reference genome sequence (e.g. D. melanogaster) for extracting windows

## Outputs

- FASTA file of promoter sequence windows centered on each TSS, one per dataset, ready as ElemeNT input
- Fixed TSS-offset convention within each window for downstream motif-position computation

## How to apply

For each TSS source (EPDnew CAGE-based promoters, csRNA-seq GSE221852, RAMPAGE GSE89299), retrieve the TSS annotations and extract a genomic sequence window centered on each TSS from the reference genome (D. melanogaster in the example). Write each window as a FASTA record so the TSS sits at a known, fixed offset within the sequence — this offset is what makes downstream ElemeNT motif positions interpretable relative to the TSS. Feed the resulting FASTA sets to the ElemeNT 2023 command-line version (Perl, github.com/OritAdato/ElemeNT), which can be installed from its installPackage zip. Keep window extraction identical across datasets being compared, since the analysis goal is comparing the positional stringency of motifs such as dInr, PB, DPE and TATA relative to the TSS.

## Related tools

- **ElemeNT 2023** (Downstream consumer of the extracted FASTA windows; detects core promoter elements relative to the TSS) — github.com/OritAdato/ElemeNT
- **ElemeNT** (Interactive Perl tool for rapid detection of core promoter elements within given sequences) — github.com/OritAdato/ElemeNT

## Examples

```
perl ElemeNT2023/src/ElemeNT/elemeNT.pl --input dm_epdnew_promoters.fasta
```

## Evaluation signals

- Each output FASTA record corresponds 1:1 to an input TSS annotation (no dropped or duplicated TSSs).
- The TSS sits at the same fixed offset in every window, so ElemeNT-reported motif positions are directly comparable across datasets.
- Windows are extracted identically (same span) for all three sources (EPDnew, csRNA-seq, RAMPAGE) before ElemeNT runs.
- Downstream ElemeNT output parses cleanly and yields per-dataset position distributions for dInr, PB, DPE, and TATA relative to the TSS.
- Reproduced comparison reproduces the expected qualitative pattern: dInr/PB/DPE positioning most stringent in csRNA-seq, TATA more strictly positioned in RAMPAGE than csRNA-seq.

## Limitations

- Quality of element-position conclusions depends on TSS annotation accuracy of each source; heterogeneous TSS definitions across datasets can confound the comparison.
- ElemeNT is focused on core promoter elements, so this extraction/consumption pipeline is not intended for arbitrary user-defined motif searches beyond that scope (though ElemeNT 2023 can search user-defined motifs).

## Evidence

- [other] Extract promoter sequence windows centered on each TSS and format them as FASTA input for ElemeNT.: [evidence span withheld — anchor status: not_found]
- [intro] ElemeNT is an interactive tool (implemented in Perl) for rapid and convenient detection of core promoter elements within any given sequences.: "ElemeNT is an interactive tool (implemented in Perl) for rapid and convenient detection of core promoter elements within any given sequences."
- [abstract] The source code and command line version of ElemeNT 2023 are available at https://github.com/OritAdato/ElemeNT.: "The source code and command line version of ElemeNT 2023 are available at https://github.com/OritAdato/ElemeNT."
- [other] The positional preference of the dInr, PB, and DPE motifs was most stringent in csRNA-seq-captured nascent TSSs.: [evidence span withheld — anchor status: not_found]
- [abstract] ElemeNT 2023 ... provides novel capabilities for assessing evolutionary conservation and for evaluating the quality of high-throughput TSS datasets leveraging preferential motif positioning.: "which provides novel capabilities for assessing evolutionary conservation and for readily evaluating the quality of high-throughput transcription start site (TSS) datasets, leveraging preferential"
