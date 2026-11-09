---
name: stringtie2-long-read-transcript-assembly
description: Use when benchmarking long-read isoform assemblers on GMAP-aligned PacBio/ONT
  BAM data against a known reference isoform set (e.g. 68 SIRV isoforms), where StringTie2
  must serve as the transcriptome-based comparator.
when_to_use_negative:
- Your goal is interactive per-gene inspection and fine-tuning of splice sites — the
  article positions VIsoQLR for that, whereas StringTie2 is a non-interactive assembler
  comparator.
- Data are short reads without long-read (PacBio/ONT) alignments — the '-L' long-read
  mode and this benchmark setting target long-read sequencing input.
- You need isoform detection counts with very high sensitivity to low-abundance transcripts
  below ~2% — StringTie2 underperformed here and the article notes most missing isoforms
  were below the 2% default read threshold.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: StringTie2
  role: Transcriptome-based isoform assembler run in long-read mode ('-L', '-f 0')
    on the SIRV BAM as the benchmark comparator
- name: GMAP
  role: Aligner producing the GFF3/BAM alignments of PacBio reads used as StringTie2
    input
- name: VIsoQLR
  role: Interactive analyzer providing the reference GMAP alignments and the 49/68
    (72%) detection benchmark against which StringTie2's 12/68 is compared
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: FLAIR
  role: Second comparator assembler (default parameters with SIRV GTF; 37/68, cosine
    similarity 0.68) benchmarked alongside StringTie2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:e8a95b2a4515d063d81082e2c7f5594bfbe539d278d28b5a2acd8cd5210cac55
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - FLAIR
  - GMAP
  - StringTie2
  - VIsoQLR
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# stringtie2-long-read-transcript-assembly

## Summary

Reproduces the paper's benchmark of StringTie2 as a transcriptome-based isoform assembler on GMAP-aligned PacBio long-read data: running StringTie2 in long-read mode ('-L') with '-f 0' on the SIRV benchmark, where it detected 12 (18%) of the 68 known SIRV isoforms versus VIsoQLR's 49 (72%) and FLAIR's 37 (54%).

## When to use

Use when benchmarking long-read isoform assemblers on GMAP-aligned PacBio/ONT BAM data against a known reference isoform set (e.g. 68 SIRV isoforms), where StringTie2 must serve as the transcriptome-based comparator.

## When NOT to use

- Your goal is interactive per-gene inspection and fine-tuning of splice sites — the article positions VIsoQLR for that, whereas StringTie2 is a non-interactive assembler comparator.
- Data are short reads without long-read (PacBio/ONT) alignments — the '-L' long-read mode and this benchmark setting target long-read sequencing input.
- You need isoform detection counts with very high sensitivity to low-abundance transcripts below ~2% — StringTie2 underperformed here and the article notes most missing isoforms were below the 2% default read threshold.

## Inputs

- GMAP-aligned long-read alignments in BAM format with a BAI index (SIRV PacBio Iso-Seq data)
- SIRV reference GTF annotation with exon coordinates for the 68 known isoforms
- Reference SIRV isoform abundance profile for cosine-similarity quantification

## Outputs

- StringTie2-assembled isoform set (GTF) from long-read BAM input
- Isoform detection counts versus the 68 known SIRV isoforms (expected 12/68, 18%)
- Cosine similarity between predicted and reference isoform abundances (expected 0.42)

## How to apply

Take the GMAP-aligned SIRV read alignments (BAM plus BAI index) produced by VIsoQLR's mapping panel, and run StringTie2 in long-read mode using '-L' together with '-f 0' so low-confidence transcripts are not filtered. Compare the assembled isoform set against the 68 known SIRV reference isoforms counting exact matches (expected: 12/68, i.e. 18%). Compute the cosine similarity between StringTie2's predicted isoform abundance profile and the reference SIRV abundances (expected: 0.42, versus 0.68 for FLAIR). In the article's benchmark, StringTie2 also extended the boundaries of some isoforms and failed to call isoforms lacking certain exons (e.g. Iso4 and Iso9 in the PAX6 minigene comparison, with missed exons 5 and 6), so boundary artifacts should be checked against the BAM evidence. The benchmark scripts in TBLabFJD/VIsoQLR_benchmark (`isoform_analysis.sh` for assembly with default parameters, `isoform_comparison.R` for the comparison) document this procedure.

## Related tools

- **StringTie2** (Transcriptome-based isoform assembler run in long-read mode ('-L', '-f 0') on the SIRV BAM as the benchmark comparator)
- **GMAP** (Aligner producing the GFF3/BAM alignments of PacBio reads used as StringTie2 input)
- **VIsoQLR** (Interactive analyzer providing the reference GMAP alignments and the 49/68 (72%) detection benchmark against which StringTie2's 12/68 is compared) — https://github.com/TBLabFJD/VIsoQLR
- **FLAIR** (Second comparator assembler (default parameters with SIRV GTF; 37/68, cosine similarity 0.68) benchmarked alongside StringTie2)

## Examples

```
stringtie sirv_gmap_aligned.bam -L -f 0 -o sirv_stringtie2.gtf
```

## Evaluation signals

- Detection count against the 68 known SIRV isoforms equals 12 (18%) when StringTie2 is run with '-L' and '-f 0' on the same GMAP-aligned BAM.
- Cosine similarity between StringTie2's predicted isoform abundances and reference SIRV abundances equals 0.42 (FLAIR 0.68, VIsoQLR 49/68 detection with 99% exon-coordinate accuracy).
- Isoform boundaries in the StringTie2 GTF are cross-checked against the BAM reads; boundary extensions observed in the paper appear only in the GTF, not in the BAM, and should be flagged as artifacts.
- Isoforms with missed exons (e.g. Iso4 lacking exon 5, Iso9 lacking exon 6 in the PAX6 minigene) are absent from the StringTie2 output, confirming the documented failure mode.
- Re-running the pipeline via TBLabFJD/VIsoQLR_benchmark scripts (isoform_analysis.sh then isoform_comparison.R) reproduces the same comparison counts.

## Limitations

- StringTie2 extends the boundaries of some isoforms in its GTF output, an artifact not present in the BAM files used by StringTie2 and VIsoQLR.
- StringTie2 does not call isoforms without certain exons — it missed Iso4 and Iso9 (which have missed exons 5 and 6) and did not detect the low-abundance Iso5 (4%) and Iso8 (1.7%) in the PAX6 minigene comparison.
- No software version numbers are reported for StringTie2 in the article, so exact reproducibility depends on the version in the benchmark scripts.

## Evidence

- [discussion] VIsoQLR detects with 99% accuracy of exon coordinates 72% of the known isoforms using SIRVs, outperforming the other tools.: "VIsoQLR detects with 99% accuracy of exon coordinates 72% of the known isoforms using SIRVs, outperforming the other tools."
- [discussion] it extends the boundaries of some isoforms, and it does not call isoforms without certain exons: "it extends the boundaries of some isoforms, and it does not call isoforms without certain exons"
- [discussion] This extension (also seen in the SIRVs analysis) seems to be an artifact as it is not present in BAM files used by StringTie2 and VIsoQLR: "This extension (also seen in the SIRVs analysis) seems to be an artifact as it is not present in BAM files used by StringTie2 and VIsoQLR"
- [readme] This repostory contains the scripts used for data preprocessing, isoform calling and isoform comparison of VIsoQLR, FLAIR and StringTie2.: [evidence span withheld — anchor status: external_doc]
