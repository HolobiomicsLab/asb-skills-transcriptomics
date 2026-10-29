---
name: isoform-structure-comparison-across-tools
description: Use when two long-read isoform callers have been run on the same alignments
  and you must reconcile their call sets — e.g., quantifying outer-boundary extensions
  or missing central exons between VIsoQLR and StringTie2 isoform coordinates.
when_to_use_negative:
- When only one caller has been run — there is no second call set to reconcile, so
  the comparison step is undefined.
- When the goal is de novo transcriptome-wide discovery rather than fine-tuning isoform
  calls for a single gene/locus from long-read data.
- When comparing against a reference annotation rather than another caller's output
  — importing known transcripts into VIsoQLR as references is the appropriate workflow
  instead.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: VIsoQLR
  role: Provides the reference isoform coordinates and consensus exon calls against
    which the other caller's isoform boundaries are compared
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: StringTie2
  role: Comparison caller whose isoform boundary extensions and missed-exon behaviors
    are being characterized
- name: VIsoQLR_benchmark
  role: Contains the isoform comparison scripts (isoform_comparison.R) used to compare
    VIsoQLR, FLAIR and StringTie2 isoform calls
  repo: https://github.com/TBLabFJD/VIsoQLR_benchmark
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:c79415a2f31fb452ed16caf636272b6bed1dff40c3b167181bc46e23a2436589
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - StringTie2
  - VIsoQLR
  - VIsoQLR_benchmark
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# Isoform structure comparison across tools

## Summary

Cross-tool comparison of isoform structures called from long-read sequencing data, matching isoforms between callers (e.g., VIsoQLR vs StringTie2) to quantify boundary extensions, missed exons, and artifact status against the shared BAM alignments.

## When to use

Use when two long-read isoform callers have been run on the same alignments and you must reconcile their call sets — e.g., quantifying outer-boundary extensions or missing central exons between VIsoQLR and StringTie2 isoform coordinates.

## When NOT to use

- When only one caller has been run — there is no second call set to reconcile, so the comparison step is undefined.
- When the goal is de novo transcriptome-wide discovery rather than fine-tuning isoform calls for a single gene/locus from long-read data.
- When comparing against a reference annotation rather than another caller's output — importing known transcripts into VIsoQLR as references is the appropriate workflow instead.

## Inputs

- VIsoQLR isoform coordinates table (consensus exon coordinates per isoform)
- StringTie2 isoform coordinates/GTF output
- Shared BAM alignments (+ BAI index) used by both tools

## Outputs

- stringtie2_behavior_table.csv with matched isoform IDs, per-boundary extension lengths in bp, artifact status, and missed/extra-exon annotations

## How to apply

Load the isoform coordinates tables from each caller (e.g., VIsoQLR output from a prior run and the StringTie2 GTF/coordinates table). Match isoforms between tools by exon-chain similarity and compute, per boundary, the extension in bases of each StringTie2 isoform relative to the VIsoQLR consensus exon coordinates (e.g., canonical PAX6 isoform 816 bp in StringTie2 vs 776 bp in VIsoQLR). Screen the StringTie2 call set for isoforms with missing central exons, recording which are not called at all and which are called with extra exons. Cross-check every boundary extension against the BAM alignments shared by both tools to classify it as an artifact; assemble matched isoform IDs, extension lengths, and missed/extra-exon annotations into a single CSV table. Rationale: the VIsoQLR benchmark showed StringTie2 systematically extends isoform boundaries and fails to call isoforms lacking certain exons (Iso4 and Iso9 in the PAX6 minigene, with missed exons 5 and 6), so caller disagreement must be adjudicated against the raw alignments rather than trusted blindly.

## Related tools

- **VIsoQLR** (Provides the reference isoform coordinates and consensus exon calls against which the other caller's isoform boundaries are compared) — https://github.com/TBLabFJD/VIsoQLR
- **StringTie2** (Comparison caller whose isoform boundary extensions and missed-exon behaviors are being characterized)
- **VIsoQLR_benchmark** (Contains the isoform comparison scripts (isoform_comparison.R) used to compare VIsoQLR, FLAIR and StringTie2 isoform calls) — https://github.com/TBLabFJD/VIsoQLR_benchmark

## Examples

```
Rscript isoform_comparison.R
```

## Evaluation signals

- Every boundary-extension entry in the table must be annotated with its BAM-file artifact status (extension verified as absent from the shared BAM alignments before flagging as artifact).
- All isoforms exhibiting missing-exon behavior must appear in the table with the corresponding StringTie2 behavior (not called vs. called with extra exons).
- Per-boundary extension lengths must be recorded in bases for every matched isoform pair (e.g., canonical PAX6 isoform: 816 bp StringTie2 vs 776 bp VIsoQLR, i.e., 40 bp total outer extension).
- Matched isoform counts and exon-chain assignments should be reproducible with the isoform_comparison.R script from the VIsoQLR_benchmark repository.

## Limitations

- StringTie2 boundary extension appears to be a caller artifact — it is not present in the BAM files shared with VIsoQLR — and was also observed in the SIRVs analysis, so it should not be interpreted as biological sequence.
- VIsoQLR's automatic analysis can miss isoforms whose reads fall below the default 2% CEC read threshold; lowering the threshold detects more isoforms and requires manual parameter tuning per gene.
- Small-abundance isoforms (e.g., Iso5 at 4% and Iso8 at 1.7%) are hard to distinguish from noise in electropherograms, complicating cross-tool validation.
- VIsoQLR is designed for close inspection and fine-tuning of one gene at a time, not high-throughput transcriptome-wide comparison.

## Evidence

- [discussion] VIsoQLR detects with 99% accuracy of exon coordinates 72% of the known isoforms using SIRVs, outperforming the other tools.: "VIsoQLR detects with 99% accuracy of exon coordinates 72% of the known isoforms using SIRVs, outperforming the other tools."
- [discussion] StringTie2 boundary extension is an artifact: "This extension (also seen in the SIRVs analysis) seems to be an artifact as it is not present in BAM files used by StringTie2 and VIsoQLR"
- [discussion] StringTie2 missed isoforms with missed exons: "Its missed isoforms (Iso4 and Iso9), appearing in the electropherogram and detected by VIsoQLR, have missed exons 5 and 6, respectively."
- [readme] Benchmark repo contains isoform comparison scripts: [evidence span withheld — anchor status: external_doc]
- [discussion] Missing isoforms attributed to default threshold: "most missing isoforms were below 2% (default value). At the same time, lowering this threshold has the cost of detecting more isoforms"
