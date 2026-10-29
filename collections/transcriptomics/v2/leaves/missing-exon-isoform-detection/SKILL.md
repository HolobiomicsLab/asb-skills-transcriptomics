---
name: missing-exon-isoform-detection
description: Use when comparing isoform call sets from StringTie2 and VIsoQLR on long-read
  RNA-seq (e.g., SIRV or minigene assays) and you need to identify missed-exon isoforms
  and distinguish genuine boundary extensions from tool artifacts.
when_to_use_negative:
- Input is a single tool's isoform set with no second call set to compare against
  — boundary-extension and missed-exon behaviors are defined relative to the VIsoQLR
  consensus.
- Data are short-read or Sanger/capillary electrophoresis based — the article states
  these traditional techniques have drawbacks across the full spectrum of splicing
  events and this skill is built on long-read sequencing isoform calls.
- The goal is de novo transcriptome-wide discovery (e.g., with epi2me-labs wf-transcriptomes/bambu)
  rather than focused one-gene fine-tuning of splicing-assay isoforms.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: VIsoQLR
  role: Produces the consensus isoform coordinates used as the reference against which
    StringTie2 missed-exon and boundary-extension behaviors are measured
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: StringTie2
  role: Isoform caller whose call set is screened for missing central exons and outer-boundary
    extensions relative to VIsoQLR
- name: VIsoQLR_benchmark
  role: Repository with the isoform comparison scripts (isoform_comparison.R) used
    to compare VIsoQLR, FLAIR and StringTie2 calls
  repo: https://github.com/TBLabFJD/VIsoQLR_benchmark
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:cae4c82f4cb82d7ab8167818ac50e7a6225d815b1cf01c37424bd1015a92e8f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - StringTie2
  - VIsoQLR
  - VIsoQLR_benchmark
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# missing-exon-isoform-detection

## Summary

Systematic cross-tool comparison of StringTie2- and VIsoQLR-called isoforms on long-read sequencing data to flag isoforms with missing central exons that StringTie2 fails to call (or calls with extra exons), and to annotate StringTie2 outer-boundary extensions as artifacts by cross-checking against shared BAM alignments.

## When to use

Use when comparing isoform call sets from StringTie2 and VIsoQLR on long-read RNA-seq (e.g., SIRV or minigene assays) and you need to identify missed-exon isoforms and distinguish genuine boundary extensions from tool artifacts.

## When NOT to use

- Input is a single tool's isoform set with no second call set to compare against — boundary-extension and missed-exon behaviors are defined relative to the VIsoQLR consensus.
- Data are short-read or Sanger/capillary electrophoresis based — the article states these traditional techniques have drawbacks across the full spectrum of splicing events and this skill is built on long-read sequencing isoform calls.
- The goal is de novo transcriptome-wide discovery (e.g., with epi2me-labs wf-transcriptomes/bambu) rather than focused one-gene fine-tuning of splicing-assay isoforms.

## Inputs

- VIsoQLR isoform coordinates table (from VIsoQLR analysis of BAM/GFF3/BED6-aligned long reads)
- StringTie2 isoform coordinates table (from StringTie2 run with default parameters)
- BAM files (plus BAI index) shared by both tools, e.g., GMAP-mapped long reads

## Outputs

- stringtie2_behavior_table.csv with matched isoform IDs, per-boundary extension lengths in bp, artifact status, and missed/extra-exon annotations

## How to apply

Load the VIsoQLR isoform coordinates table (task_001 output) and the StringTie2 isoform coordinates table (task_002 output), then match isoforms between the two tools and compute the outer-boundary extension of each StringTie2 isoform relative to the VIsoQLR consensus exon coordinates, in bases per boundary. Screen the StringTie2 call set for isoforms containing missing central exons, listing those StringTie2 fails to call and those it calls with extra exons — in the PAX6 minigene comparison, StringTie2 missed Iso4 and Iso9, which carry missed exons 5 and 6, respectively. Flag boundary extensions as artifacts by cross-checking that they are not present in the BAM alignments shared by both tools (e.g., a canonical isoform 816 bp in StringTie2 vs 776 bp in VIsoQLR). Assemble matched isoform IDs, extension lengths, and missed/extra-exon annotations into a single table written as stringtie2_behavior_table.csv. The rationale comes from the SIRV/PAX6 benchmark showing StringTie2 boundary extension is an artifact absent from the BAM files and that it does not call isoforms lacking certain exons.

## Related tools

- **VIsoQLR** (Produces the consensus isoform coordinates used as the reference against which StringTie2 missed-exon and boundary-extension behaviors are measured) — https://github.com/TBLabFJD/VIsoQLR
- **StringTie2** (Isoform caller whose call set is screened for missing central exons and outer-boundary extensions relative to VIsoQLR)
- **VIsoQLR_benchmark** (Repository with the isoform comparison scripts (isoform_comparison.R) used to compare VIsoQLR, FLAIR and StringTie2 calls) — https://github.com/TBLabFJD/VIsoQLR_benchmark

## Examples

```
Rscript isoform_comparison.R
```

## Evaluation signals

- Every boundary-extension entry in stringtie2_behavior_table.csv is annotated with its BAM-file artifact status.
- All isoforms exhibiting missing-exon behavior appear in the table with the corresponding StringTie2 behavior (not called vs. called with extra exons).
- Extension lengths are recorded per boundary in bases (e.g., canonical isoform 816 bp in StringTie2 vs 776 bp in VIsoQLR, i.e., 40 bp total across both ends).
- Flagged boundary extensions must be absent from the BAM alignments shared by both tools, matching the article's finding that the extension is an artifact.
- Missed StringTie2 isoforms should match isoforms detected by VIsoQLR and corroborated independently (e.g., present in the PAX6 minigene electropherogram).

## Limitations

- VIsoQLR isoforms below the default 2% read threshold may be missing from the automatic analysis, requiring manual parameter tuning before comparison.
- Small isoform abundances (e.g., Iso5 at 4% and Iso8 at 1.7%) may be overrepresented by LRS and hard to distinguish from noise in electropherograms.
- No software version numbers are reported for VIsoQLR, StringTie2, or FLAIR, and no accession identifiers are given for the underlying LRS datasets, complicating exact reproduction.

## Evidence

- [discussion] StringTie2 boundary extension is an artifact: "This extension (also seen in the SIRVs analysis) seems to be an artifact as it is not present in BAM files used by StringTie2 and VIsoQLR"
- [discussion] StringTie2 missed-exon behavior: "it extends the boundaries of some isoforms, and it does not call isoforms without certain exons"
- [discussion] StringTie2 missed isoforms have missed exons: "Its missed isoforms (Iso4 and Iso9), appearing in the electropherogram and detected by VIsoQLR, have missed exons 5 and 6, respectively."
- [discussion] VIsoQLR benchmark accuracy on SIRVs: "VIsoQLR detects with 99% accuracy of exon coordinates 72% of the known isoforms using SIRVs, outperforming the other tools."
- [readme] Mini-IsoQLR consensus breakpoint definition: [evidence span withheld — anchor status: external_doc]
