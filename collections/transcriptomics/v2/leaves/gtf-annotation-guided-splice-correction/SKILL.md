---
name: gtf-annotation-guided-splice-correction
description: Use when assembling or correcting isoforms from long-read RNA-seq alignments
  (e.g. GMAP-aligned PacBio BAM) where a trusted GTF annotation with exon coordinates
  is available and detection accuracy against known isoforms must be benchmarked.
when_to_use_negative:
- No reference GTF with exon-coordinate evidence exists for the locus or organism
  being analyzed — de novo isoform discovery methods without annotation guidance are
  required instead.
- Input is short-read RNA-seq data — the benchmark and the '-L' long-read mode of
  StringTie2 are defined for long-read (PacBio/nanopore) alignments.
- The goal is interactive per-gene fine-tuning of splice sites and read thresholds
  (e.g. the 2% CEC cutoff) — VIsoQLR's interactive editor, not annotation-guided correction
  by FLAIR/StringTie2, is the appropriate tool.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: FLAIR
  role: Isoform detector run with default parameters and a supplied GTF so its exon
    coordinates guide isoform correction of the aligned reads
- name: StringTie2
  role: Comparator assembler run in long-read mode ('-L') with '-f 0' to prevent filtering
    of low-confidence transcripts
- name: GMAP
  role: Aligns raw long-read FASTQ reads to a GMAP reference index, producing the
    BAM/BAI alignments the isoform tools consume
- name: VIsoQLR
  role: Reference isoform detector and interactive editor whose GMAP-based benchmark
    defines the expected detection counts (49/68, 72%) for validating this skill
  repo: https://github.com/TBLabFJD/VIsoQLR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:4b1e528d75e6b384a1b204046a5e45e359b6f7fae1eca173694591a4b1b34e10
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - FLAIR
  - GMAP
  - StringTie2
  - VIsoQLR
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# gtf-annotation-guided-splice-correction

## Summary

Guides long-read isoform assembly and splice-site correction with a known GTF annotation (e.g. the SIRV GTF with 68 reference isoforms) so that exon coordinates from the annotation, rather than de novo inference alone, define detected transcripts. The VIsoQLR paper benchmarks this approach by running FLAIR (default parameters plus GTF) and StringTie2 ('-L -f 0') on the same GMAP-aligned PacBio BAM.

## When to use

Use when assembling or correcting isoforms from long-read RNA-seq alignments (e.g. GMAP-aligned PacBio BAM) where a trusted GTF annotation with exon coordinates is available and detection accuracy against known isoforms must be benchmarked.

## When NOT to use

- No reference GTF with exon-coordinate evidence exists for the locus or organism being analyzed — de novo isoform discovery methods without annotation guidance are required instead.
- Input is short-read RNA-seq data — the benchmark and the '-L' long-read mode of StringTie2 are defined for long-read (PacBio/nanopore) alignments.
- The goal is interactive per-gene fine-tuning of splice sites and read thresholds (e.g. the 2% CEC cutoff) — VIsoQLR's interactive editor, not annotation-guided correction by FLAIR/StringTie2, is the appropriate tool.

## Inputs

- GMAP-aligned long-read read alignments (BAM plus BAI index)
- GTF annotation file with exon coordinates (e.g. the SIRV GTF covering 68 known isoforms)
- Raw long-read FASTQ reads and reference sequence FASTA (for GMAP index building)
- Set of known reference isoforms for exact-match comparison

## Outputs

- Assembled isoform sets (per tool) as transcript annotations comparable to the reference GTF
- Exact-match detection counts against the 68 known SIRV isoforms (e.g. FLAIR 37/68, StringTie2 12/68)
- Cosine similarity between each tool's predicted isoform abundance profile and reference SIRV abundances (e.g. FLAIR 0.68, StringTie2 0.42)

## How to apply

Start from the GMAP-aligned long-read alignments (BAM plus BAI index) produced by the mapping step. Run FLAIR with default parameters, supplying the SIRV GTF annotation so its exon coordinates guide isoform correction. Run StringTie2 on the same BAM in long-read mode ('-L') with '-f 0' to prevent filtering of low-confidence transcripts. Compare each tool's assembled isoform set against the 68 known SIRV reference isoforms using exact matches, then compute the cosine similarity between each tool's predicted isoform abundance profile and the reference SIRV abundances. Expected benchmark results on the PacBio SIRV data: FLAIR detects 37/68 (54%) with cosine similarity 0.68, StringTie2 detects 12/68 (18%) with cosine similarity 0.42, versus VIsoQLR's 49/68 (72%) with 99% exon-coordinate accuracy.

## Related tools

- **FLAIR** (Isoform detector run with default parameters and a supplied GTF so its exon coordinates guide isoform correction of the aligned reads)
- **StringTie2** (Comparator assembler run in long-read mode ('-L') with '-f 0' to prevent filtering of low-confidence transcripts)
- **GMAP** (Aligns raw long-read FASTQ reads to a GMAP reference index, producing the BAM/BAI alignments the isoform tools consume)
- **VIsoQLR** (Reference isoform detector and interactive editor whose GMAP-based benchmark defines the expected detection counts (49/68, 72%) for validating this skill) — https://github.com/TBLabFJD/VIsoQLR

## Examples

```
bash isoform_analysis.sh   # from TBLabFJD/VIsoQLR_benchmark: data acquisition, GMAP index generation, FLAIR and StringTie2 analysis with default parameters
```

## Evaluation signals

- FLAIR detection count on the SIRV benchmark equals 37/68 (54%) of known isoforms detected as exact matches.
- StringTie2 detection count equals 12/68 (18%) when run with '-L -f 0' on the same GMAP-aligned BAM.
- Cosine similarity between FLAIR's predicted isoform abundance profile and reference SIRV abundances equals 0.68; StringTie2's equals 0.42.
- VIsoQLR comparison point: 72% of known SIRV isoforms detected with 99% exon-coordinate accuracy — results far outside this range suggest alignment or annotation input errors.
- Isoform comparison uses exact matches of assembled transcript exon structures against the 68-isoform SIRV reference set, not fuzzy overlap.

## Limitations

- StringTie2 extends the boundaries of some isoforms — an artifact not present in the BAM files used by StringTie2 and VIsoQLR — and does not call isoforms lacking certain exons.
- VIsoQLR's automatic analysis misses isoforms whose reads fall below the default 2% CEC read threshold, and recovering them requires manual configuration of parameters; lowering the threshold detects more isoforms at a cost.
- No software version numbers are reported for VIsoQLR, StringTie2, or FLAIR, so exact reproducibility of the 37/68 and 12/68 counts may depend on tool versions.
- The benchmark scripts in TBLabFJD/VIsoQLR_benchmark analyze SIRV transcripts on a public PacBio RNA-seq dataset; counts are tied to that SIRV setting and may not transfer directly to other annotations or read depths.

## Evidence

- [intro] Annotation with exon coordinates guides isoform detection: [evidence span withheld — anchor status: not_found]
- [discussion] Benchmark context for VIsoQLR, StringTie2 and FLAIR isoform quality: "we have performed a benchmark to determine the quality of the isoforms detected by VIsoQLR, StringTie2 and FLAIR software in a typical LRS experiment."
- [discussion] Reference detection accuracy on SIRV isoforms: "VIsoQLR detects with 99% accuracy of exon coordinates 72% of the known isoforms using SIRVs, outperforming the other tools."
- [discussion] StringTie2 boundary-extension artifact and missed exon-skip isoforms: "it extends the boundaries of some isoforms, and it does not call isoforms without certain exons"
- [readme] Benchmark repository scripts cover FLAIR and StringTie2 runs with default parameters: [evidence span withheld — anchor status: external_doc]
- [methods] Mapped alignments delivered as BAM plus BAI index feed the isoform callers: "two download bottoms will appear: the mapped reads (BAM) and its index (BAI)."
