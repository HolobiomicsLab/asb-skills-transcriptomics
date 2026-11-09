---
name: motif-position-distribution-analysis
description: Use when you have high-confidence TSS coordinates (e.g., HOMER-called
  csRNA-seq peaks, EPDnew/CAGE, RAMPAGE annotations) and need to quantify whether
  core promoter elements show preferential positional enrichment around the TSS.
when_to_use_negative:
- Input sequences have no TSS-centered coordinate frame — ElemeNT positional stringency
  interpretation requires motifs scored relative to an annotated TSS
- The goal is general de novo motif discovery in arbitrary DNA rather than detection
  and positioning of known core promoter elements (TATA, dInr, PB, DPE)
- The TSS source is low-quality or unvalidated — the article positions this skill
  specifically for evaluating the quality of high-throughput TSS datasets, not for
  raw unfiltered peak lists
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_0749
tools:
- name: ElemeNT 2023
  role: Command-line/web tool that detects core promoter elements (TATA, dInr, PB,
    DPE) in FASTA promoter sequences and reports positions and scores relative to
    the TSS
  repo: https://github.com/OritAdato/ElemeNT
- name: Perl
  role: Implementation language of the ElemeNT/ElemeNT 2023 command-line version executed
    on promoter FASTA inputs
  repo: https://github.com/OritAdato/ElemeNT
- name: HOMER
  role: Used with findPeaks to call high-confidence TSS coordinates from csRNA-seq
    data prior to promoter extraction
- name: bedtools
  role: getfasta extracts ±100 bp core promoter sequences centered on called TSSs
    into FASTA for ElemeNT input
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae110
    title: 'ElemeNT 2023: an enhanced tool for detection and curation of core promoter
      elements'
  source_content_hash: sha256:edb14a3cfe6f5488e888032c9bbee75b8d4e0738144dd6cfc29bd031c42c395b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OritAdato/ElemeNT
  tools:
  - ElemeNT 2023
  - HOMER
  - Perl
  - bedtools
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# motif-position-distribution-analysis

## Summary

Analyzes the position and score distributions of core promoter motifs (TATA, dInr, PB, DPE) relative to annotated TSSs using ElemeNT 2023, enabling dataset-quality assessment and cross-species comparison of promoter element positioning.

## When to use

Use when you have high-confidence TSS coordinates (e.g., HOMER-called csRNA-seq peaks, EPDnew/CAGE, RAMPAGE annotations) and need to quantify whether core promoter elements show preferential positional enrichment around the TSS.

## When NOT to use

- Input sequences have no TSS-centered coordinate frame — ElemeNT positional stringency interpretation requires motifs scored relative to an annotated TSS
- The goal is general de novo motif discovery in arbitrary DNA rather than detection and positioning of known core promoter elements (TATA, dInr, PB, DPE)
- The TSS source is low-quality or unvalidated — the article positions this skill specifically for evaluating the quality of high-throughput TSS datasets, not for raw unfiltered peak lists

## Inputs

- High-confidence TSS coordinates (BED or annotation table) from csRNA-seq peaks called with HOMER findPeaks, EPDnew/CAGE promoters, or RAMPAGE datasets
- Reference genome FASTA for promoter sequence extraction
- ElemeNT CORE database records (predicted core promoter elements around EPDnew-annotated TSSs, 10 metazoan species)
- FASTA files of ±100 bp promoter windows centered on TSSs

## Outputs

- Per-dataset/per-species motif position distributions relative to the TSS (e.g., element_distributions.csv, element_position_stringency_comparison.csv)
- Per-species element score summary statistics (mean/median score per element type, core_element_distributions.csv)
- Cross-dataset positional stringency comparison of dInr, PB, DPE, and TATA motifs

## How to apply

Obtain TSS coordinates from a high-throughput TSS source (e.g., call peaks from csRNA-seq FASTQ with HOMER findPeaks, or download EPDnew/RAMPAGE annotations), then extract ±100 bp core promoter sequences centered on each TSS into FASTA with bedtools getfasta. Run the ElemeNT 2023 command-line implementation (Perl, github.com/OritAdato/ElemeNT) on each FASTA file to detect core promoter elements (TATA, dInr, PB, DPE). Parse ElemeNT output and express each detected element's position and score relative to its TSS, then aggregate per dataset or per species into position, fraction, and median-score distributions. The rationale, per the article, is that ElemeNT 2023 leverages 'preferential motif positioning' to evaluate the quality of high-throughput TSS datasets and to assess evolutionary conservation across species; write the aggregated distributions (e.g., element_distributions.csv or core_element_distributions.csv) for downstream comparison.

## Related tools

- **ElemeNT 2023** (Command-line/web tool that detects core promoter elements (TATA, dInr, PB, DPE) in FASTA promoter sequences and reports positions and scores relative to the TSS) — https://github.com/OritAdato/ElemeNT
- **Perl** (Implementation language of the ElemeNT/ElemeNT 2023 command-line version executed on promoter FASTA inputs) — https://github.com/OritAdato/ElemeNT
- **HOMER** (Used with findPeaks to call high-confidence TSS coordinates from csRNA-seq data prior to promoter extraction)
- **bedtools** (getfasta extracts ±100 bp core promoter sequences centered on called TSSs into FASTA for ElemeNT input)

## Examples

```
bedtools getfasta -fi genome.fa -bed tss_peaks.bed -fo promoters.fa  # extract ±100 bp promoters, then run the ElemeNT 2023 command-line (Perl) implementation on promoters.fa
```

## Evaluation signals

- Detected elements show preferential positional enrichment around the TSS, consistent with ElemeNT 2023's intended use for evaluating high-throughput TSS dataset quality
- Per-species distributions reproduce conserved TATA and dInr positioning across the 10 metazoan species (worm to human)
- dInr mean score is elevated in fly (D. melanogaster) and honeybee, and the PB motif is restricted to D. melanogaster with enrichment at +26 relative to the TSS
- Cross-dataset comparison shows dInr/PB/DPE positioning is most stringent in csRNA-seq nascent TSSs, while TATA is more strictly positioned in RAMPAGE data than csRNA-seq
- Output CSVs contain per-species/per-dataset position, fraction, and median-score distributions for each element type without missing TSS-relative coordinates

## Limitations

- The PB motif is restricted to D. melanogaster (enriched at +26), so PB-positioning comparisons are only meaningful in fly data
- Positional stringency conclusions depend on the TSS source: different high-throughput TSS datasets (EPDnew/CAGE, csRNA-seq, RAMPAGE) yield different motif spacing stringency, so cross-dataset comparisons must control for the TSS-capture method
- ElemeNT is focused on core promoter elements; it is not a general-purpose motif scanner, though it can also search user-defined motifs including sequence-specific DNA binding sites

## Evidence

- [readme] ElemeNT is an interactive tool (implemented in Perl) for rapid and convenient detection of core promoter elements within any given sequences.: "ElemeNT is an interactive tool (implemented in Perl) for rapid and convenient detection of core promoter elements within any given sequences."
- [abstract] ElemeNT 2023 capabilities for TSS dataset quality assessment via preferential motif positioning: "which provides novel capabilities for assessing evolutionary conservation and for readily evaluating the quality of high-throughput transcription start site (TSS) datasets, leveraging preferential"
- [abstract] CORE database expanded to 10 species: "ElemeNT's CORE database, which contains predicted core promoter elements around annotated TSSs, is now expanded to cover 10 species, ranging from worms to human"
- [abstract] Command-line availability and repo: "The source code and command line version of ElemeNT 2023 are available at https://github.com/OritAdato/ElemeNT."
