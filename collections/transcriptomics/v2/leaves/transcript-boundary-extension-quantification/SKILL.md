---
name: transcript-boundary-extension-quantification
description: Use when comparing isoform calls from StringTie2 and VIsoQLR on the same
  BAM alignments (e.g., SIRV spike-ins or a PAX6 minigene ONT assay) and transcript
  start/end coordinates disagree between tools, or when StringTie2 fails to call isoforms
  that lack certain exons.
when_to_use_negative:
- The two tools were run on different BAM alignment files, so boundary differences
  cannot be attributed to the caller rather than to the input alignments.
- The goal is generic isoform detection benchmarking (e.g., VIsoQLR vs FLAIR accuracy
  on SIRVs with 99% exon-coordinate accuracy) rather than characterization of boundary
  and missed-exon artifacts specific to StringTie2.
- Input is raw FASTQ or unaligned reads rather than per-tool isoform coordinate tables
  plus shared BAMs.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: VIsoQLR
  role: Produces the reference consensus exon-coordinate isoforms against which StringTie2
    boundary extensions are measured
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: StringTie2
  role: Isoform caller whose outer-boundary extensions and missed-exon behaviors are
    quantified relative to VIsoQLR calls
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:48b8047f3be824d487f4ec42e21e6a43655af520ead96fe3893e1e1b4ec731ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - StringTie2
  - VIsoQLR
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# transcript-boundary-extension-quantification

## Summary

Quantifies systematic outer-exon boundary extension in StringTie2-called isoforms relative to VIsoQLR consensus isoforms from long-read sequencing data, and classifies the extension as an alignment artifact by cross-checking the shared BAM alignments. It also records StringTie2 missed-exon behaviors (isoforms not called vs. called with extra exons).

## When to use

Use when comparing isoform calls from StringTie2 and VIsoQLR on the same BAM alignments (e.g., SIRV spike-ins or a PAX6 minigene ONT assay) and transcript start/end coordinates disagree between tools, or when StringTie2 fails to call isoforms that lack certain exons.

## When NOT to use

- The two tools were run on different BAM alignment files, so boundary differences cannot be attributed to the caller rather than to the input alignments.
- The goal is generic isoform detection benchmarking (e.g., VIsoQLR vs FLAIR accuracy on SIRVs with 99% exon-coordinate accuracy) rather than characterization of boundary and missed-exon artifacts specific to StringTie2.
- Input is raw FASTQ or unaligned reads rather than per-tool isoform coordinate tables plus shared BAMs.

## Inputs

- VIsoQLR isoform coordinates table (consensus exon coordinates per isoform, e.g., from task_001 expected_outputs[0])
- StringTie2 isoform coordinates table (e.g., from task_002 expected_outputs[0])
- BAM (plus BAI) alignments of the long reads shared by both tools
- Optionally, known SIRV or minigene (e.g., PAX6) reference isoforms for matching

## Outputs

- stringtie2_behavior_table.csv with matched isoform IDs, boundary extension lengths in bp per boundary, BAM artifact status, and missed/extra-exon annotations

## How to apply

Load the VIsoQLR isoform coordinate table and the StringTie2 isoform coordinate table produced for the same gene/locus, match isoforms between the two callers, and compute the per-boundary extension in bases of each StringTie2 isoform against the VIsoQLR consensus exon coordinates (in the PAX6 minigene case, the canonical isoform is 816 bp by StringTie2 vs 776 bp by VIsoQLR). Separately, screen the StringTie2 call set for missed central exons: isoforms present in VIsoQLR but absent from StringTie2 calls (e.g., Iso4 and Iso9, which carry missed exons 5 and 6) and isoforms StringTie2 calls with extra exons. Flag each boundary extension as an artifact by verifying it is absent from the BAM files shared by both tools, since the extension 'is not present in BAM files used by StringTie2 and VIsoQLR'. Assemble matched isoform IDs, extension lengths in bp, artifact status, and missed/extra-exon annotations into a single table (stringtie2_behavior_table.csv).

## Related tools

- **VIsoQLR** (Produces the reference consensus exon-coordinate isoforms against which StringTie2 boundary extensions are measured) — https://github.com/TBLabFJD/VIsoQLR
- **StringTie2** (Isoform caller whose outer-boundary extensions and missed-exon behaviors are quantified relative to VIsoQLR calls)

## Examples

```
Rscript isoform_comparison.R
```

## Evaluation signals

- Every boundary-extension entry in the output table is annotated with its BAM-file artifact status.
- All isoforms exhibiting missing-exon behavior appear in the table with the corresponding StringTie2 behavior (not called vs. called with extra exons).
- Extension lengths are reported in bases per boundary and are consistent with coordinate differences between matched isoform pairs (e.g., 816 bp vs 776 bp for the canonical PAX6 isoform).
- Flagged extensions are confirmed absent from the shared BAM alignments, and the same extension pattern should replicate in the SIRVs analysis.

## Limitations

- The boundary extension appears to be an artifact of StringTie2 rather than a biological signal, as it is not present in the BAM files used by both tools; it should not be interpreted as real transcript sequence.
- VIsoQLR misses low-abundance isoforms below the default 2% read threshold, so isoform matching between tools can be incomplete without manual lowering of the read-threshold parameter.
- No software version numbers are reported for VIsoQLR or StringTie2, so exact reproducibility of the boundary-extension magnitudes is not guaranteed.

## Evidence

- [discussion] StringTie2 boundary extension and missed-exon behavior: "it extends the boundaries of some isoforms, and it does not call isoforms without certain exons"
- [discussion] Boundary extension is an artifact absent from BAMs: "This extension (also seen in the SIRVs analysis) seems to be an artifact as it is not present in BAM files used by StringTie2 and VIsoQLR"
- [discussion] Specific missed isoforms with missed exons: "Its missed isoforms (Iso4 and Iso9), appearing in the electropherogram and detected by VIsoQLR, have missed exons 5 and 6, respectively."
- [discussion] VIsoQLR accuracy context for using its calls as reference: "VIsoQLR detects with 99% accuracy of exon coordinates 72% of the known isoforms using SIRVs, outperforming the other tools."
- [readme] Benchmark scripts for tool comparison: [evidence span withheld — anchor status: external_doc]
