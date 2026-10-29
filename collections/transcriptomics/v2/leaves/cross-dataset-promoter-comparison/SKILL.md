---
name: cross-dataset-promoter-comparison
description: Use when two or more transcription start site (TSS) datasets for the
  same organism must be benchmarked against each other, and a motif-positioning-based
  quality comparison of nascent vs. annotated TSSs is desired.
when_to_use_negative:
- The input is a single sequence set with no second TSS source to compare against
  — ElemeNT alone suffices for element detection without the cross-dataset stringency
  analysis.
- The goal is generic motif search for arbitrary user-defined DNA binding sites rather
  than benchmarking TSS dataset quality via core promoter element positioning.
- No TSS coordinates are available to center promoter windows on, so element positions
  cannot be interpreted relative to the TSS.
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_3511
- http://edamontology.org/topic_0749
tools:
- name: ElemeNT 2023
  role: Command-line (Perl) detector of core promoter elements (dInr, PB, DPE, TATA)
    run on FASTA promoter windows from each TSS dataset to obtain position distributions
  repo: github.com/OritAdato/ElemeNT
- name: ElemeNT
  role: Underlying element navigation tool whose CORE database of predicted elements
    around annotated TSSs covers 10 species and underpins the comparative analysis
  repo: github.com/OritAdato/ElemeNT
- name: Perl
  role: Implementation language of the ElemeNT command-line version used in the workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae110
    title: 'ElemeNT 2023: an enhanced tool for detection and curation of core promoter
      elements'
  source_content_hash: sha256:fe15e2e69d6d3b690973ed18ee865d31c41890abd0712be3dfd5b8285715f64e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ElemeNT
  - ElemeNT 2023
  - Perl
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_3511
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# cross-dataset-promoter-comparison

## Summary

Evaluates the quality of high-throughput TSS datasets by running ElemeNT 2023 on promoter sequence windows from multiple TSS sources (e.g., EPDnew/CAGE, csRNA-seq GSE221852, RAMPAGE GSE89299 in D. melanogaster) and comparing the positional stringency of core promoter elements (dInr, PB, DPE, TATA) relative to the TSS across datasets.

## When to use

Use when two or more transcription start site (TSS) datasets for the same organism must be benchmarked against each other, and a motif-positioning-based quality comparison of nascent vs. annotated TSSs is desired.

## When NOT to use

- The input is a single sequence set with no second TSS source to compare against — ElemeNT alone suffices for element detection without the cross-dataset stringency analysis.
- The goal is generic motif search for arbitrary user-defined DNA binding sites rather than benchmarking TSS dataset quality via core promoter element positioning.
- No TSS coordinates are available to center promoter windows on, so element positions cannot be interpreted relative to the TSS.

## Inputs

- D. melanogaster (or other organism) TSS annotations from EPDnew, csRNA-seq (GSE221852), RAMPAGE (GSE89299), or comparable high-throughput TSS datasets
- Promoter sequence windows centered on each TSS, formatted as FASTA

## Outputs

- Per-dataset core promoter element calls from ElemeNT 2023 (dInr, PB, DPE, TATA positions relative to the TSS)
- Cross-dataset motif position stringency comparison written to element_position_stringency_comparison.csv

## How to apply

Retrieve TSS annotations from each source (e.g., EPDnew CAGE-based promoters, the csRNA-seq dataset GSE221852, and the RAMPAGE dataset GSE89299), extract promoter sequence windows centered on each TSS, and format them as FASTA for ElemeNT. Run the ElemeNT 2023 command-line version (Perl, github.com/OritAdato/ElemeNT) on each sequence set to detect core promoter elements, then parse the output to compute position distributions of dInr, PB, DPE, and TATA motifs relative to the TSS per dataset. The rationale, per the article, is that ElemeNT 2023 leverages preferential motif positioning to evaluate high-throughput TSS dataset quality; more sharply peaked motif position distributions indicate a more stringently positioned, higher-quality TSS set. Compare the spatial stringency across sources, writing the result to element_position_stringency_comparison.csv, and test whether dInr/PB/DPE positioning is most stringent in the csRNA-seq nascent TSS set while RAMPAGE-derived TATA is more strictly positioned than csRNA-seq TATA.

## Related tools

- **ElemeNT 2023** (Command-line (Perl) detector of core promoter elements (dInr, PB, DPE, TATA) run on FASTA promoter windows from each TSS dataset to obtain position distributions) — github.com/OritAdato/ElemeNT
- **ElemeNT** (Underlying element navigation tool whose CORE database of predicted elements around annotated TSSs covers 10 species and underpins the comparative analysis) — github.com/OritAdato/ElemeNT
- **Perl** (Implementation language of the ElemeNT command-line version used in the workflow)

## Evaluation signals

- The comparison CSV (element_position_stringency_comparison.csv) exists and contains per-dataset position distributions for dInr, PB, DPE, and TATA.
- dInr, PB, and DPE motif positions show the tightest (most stringent) distributions in the csRNA-seq (GSE221852) nascent TSS set relative to EPDnew and RAMPAGE.
- TATA box positioning is more strictly constrained in the RAMPAGE (GSE89299) set than in the csRNA-seq set, reproducing the reported finding.
- ElemeNT output was produced for all three sequence sets with comparable numbers of called elements, indicating consistent FASTA window formatting across datasets.

## Limitations

- The evaluation leverages preferential motif positioning and is therefore focused on core promoter elements; it is not a general-purpose TSS quality metric for organisms or datasets lacking such positioning.
- Findings are demonstrated on D. melanogaster datasets (EPDnew/CAGE, GSE221852, GSE89299); stringency rankings of specific elements (e.g., TATA in RAMPAGE) may not transfer to other organisms or TSS-capture methods.

## Evidence

- [abstract] ElemeNT 2023 provides novel capabilities for assessing evolutionary conservation and for evaluating the quality of high-throughput TSS datasets leveraging preferential motif positioning: [evidence span withheld — anchor status: not_found]
- [intro] ElemeNT is an interactive tool (implemented in Perl) for rapid and convenient detection of core promoter elements within any given sequences.: "ElemeNT is an interactive tool (implemented in Perl) for rapid and convenient detection of core promoter elements within any given sequences."
- [abstract] The source code and command line version of ElemeNT 2023 are available at https://github.com/OritAdato/ElemeNT.: "The source code and command line version of ElemeNT 2023 are available at https://github.com/OritAdato/ElemeNT."
- [readme] ElemeNT/ElemeNT2023/src/Manuscript - the source code used for the analysis included in the manuscript: "ElemeNT/ElemeNT2023/src/Manuscript - the source code used for the analysis included in the manuscript"
