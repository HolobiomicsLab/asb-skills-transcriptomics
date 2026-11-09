---
name: rna-seq-read-alignment-multi-aligner
description: Use when you need to demonstrate that a mapping-derived signal (e.g.
  a shift, bias, or clustering pattern) is algorithm-independent by repeating the
  same alignment on identical input FASTQ with multiple aligners and comparing downstream
  distributions.
when_to_use_negative:
- The question is about alignment quality or sensitivity of a single aligner rather
  than testing whether a pattern persists across aligners — a single-alignment benchmark
  suffices.
- The data are paired-end or spliced RNA-Seq where intron-aware alignment is required;
  the article's STAR run explicitly specifies no introns for its single-end P. patens
  UMI data.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3308
tools:
- name: STAR
  role: First aligner; aligns the UMI-clipped reads to the P. patens genome specifying
    no introns, producing a sorted SAM file
  repo: https://github.com/alexdobin/STAR
- name: GSNAP
  role: Second aligner; produces an independent sorted SAM alignment for cross-aligner
    comparison
- name: HISAT2
  role: Third aligner; produces an independent sorted SAM alignment for cross-aligner
    comparison
- name: umi_counterizer.pl
  role: Perl script run on each sorted SAM file with the article's max_edit_distance
    setting to collapse duplicate UMI at the same position
  repo: https://github.com/ncgr/UMI-analysis
- name: umi_analyzer_uthash
  role: C program (using uthash preprocessor macros) that emits umi_positions.txt
    listing each UMI and its genomic mapping positions
  repo: https://github.com/ncgr/UMI-analysis
- name: umi_clusterizer.pl
  role: Perl script producing position_umi_counts.bed counting unique UMIs per genomic
    position from umi_positions.txt
  repo: https://github.com/ncgr/UMI-analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-018-31064-7
    title: Unique Molecular Identifiers reveal a novel sequencing artefact with implications
      for RNA-Seq based gene expression analysis
  source_content_hash: sha256:4526c4fcf8776e96b8357b52b8f202df29e62b7496f0ce88d5f8f7a570ae4583
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexdobin/STAR
  tools:
  - C
  - GSNAP
  - HISAT2
  - STAR
  - Text::Levenshtein::XS
  - Try::Tiny
  - umi_analyzer_uthash
  - umi_clusterizer.pl
  - umi_counterizer.pl
  - uthash
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# rna-seq-read-alignment-multi-aligner

## Summary

Align the same quality-filtered, UMI-clipped FASTQ reads to a genomic reference with three independent aligners (STAR with no introns, GSNAP, and HISAT2) to test whether an observed pattern — here the UMI-read mapping shift artefact in P. patens single-cell RNA-Seq — is an artefact of one alignment algorithm or persists across aligners.

## When to use

Use when you need to demonstrate that a mapping-derived signal (e.g. a shift, bias, or clustering pattern) is algorithm-independent by repeating the same alignment on identical input FASTQ with multiple aligners and comparing downstream distributions.

## When NOT to use

- The question is about alignment quality or sensitivity of a single aligner rather than testing whether a pattern persists across aligners — a single-alignment benchmark suffices.
- The data are paired-end or spliced RNA-Seq where intron-aware alignment is required; the article's STAR run explicitly specifies no introns for its single-end P. patens UMI data.

## Inputs

- Quality-filtered, UMI-clipped FASTQ reads (e.g. run_170420) from single-cell RNA-Seq of Physcomitrella patens
- P. patens genomic reference sequence (FASTA plus aligner index for STAR, GSNAP, and HISAT2)
- Separation threshold for mapping-shift distances (article default: 3 bp, configurable)

## Outputs

- One sorted SAM file per aligner (STAR, GSNAP, HISAT2)
- all_sam_umis.txt and collapsed_sam_umis.txt per alignment from umi_counterizer.pl
- umi_positions.txt per alignment from umi_analyzer_uthash
- position_umi_counts.bed per alignment from umi_clusterizer.pl
- Three overlaid mapping-shift density curves (one per aligner), reproducing Fig. 5

## How to apply

Take the quality-filtered, UMI-clipped reads (e.g. run_170420 FASTQ after fastq_qual_filter and fastq_umi_clipper) and run three separate alignments to the P. patens genomic reference: STAR specifying no introns, GSNAP, and HISAT2, each producing a sorted SAM file. Feed each sorted SAM into the identical downstream pipeline (umi_counterizer.pl with the article's max_edit_distance setting, then umi_analyzer_uthash, then umi_clusterizer.pl) so that any divergence in the final distributions can be attributed to the aligner alone. For each alignment, compute mapping-shift distances between reads sharing a UMI, treating reads separated by three or fewer base pairs as derived from the same starting mRNA (threshold configurable). Plot the shift-distance distributions as three density curves, one per aligner, and overlay them. The rationale is that if all three aligners yield the same basic pattern with a pronounced mode and reads mapping to either side, the artefact is aligner-independent rather than a STAR-specific behaviour.

## Related tools

- **STAR** (First aligner; aligns the UMI-clipped reads to the P. patens genome specifying no introns, producing a sorted SAM file) — https://github.com/alexdobin/STAR
- **GSNAP** (Second aligner; produces an independent sorted SAM alignment for cross-aligner comparison)
- **HISAT2** (Third aligner; produces an independent sorted SAM alignment for cross-aligner comparison)
- **umi_counterizer.pl** (Perl script run on each sorted SAM file with the article's max_edit_distance setting to collapse duplicate UMI at the same position) — https://github.com/ncgr/UMI-analysis
- **umi_analyzer_uthash** (C program (using uthash preprocessor macros) that emits umi_positions.txt listing each UMI and its genomic mapping positions) — https://github.com/ncgr/UMI-analysis
- **umi_clusterizer.pl** (Perl script producing position_umi_counts.bed counting unique UMIs per genomic position from umi_positions.txt) — https://github.com/ncgr/UMI-analysis

## Examples

```
STAR --genomeDir pp_patens_index --readFilesIn run_170420.clipped.fq --outFileNamePrefix star_run1/ --outSAMtype SAM; samtools sort -o star_run1/aligned.sorted.sam star_run1/Aligned.out.sam; perl umi_counterizer.pl star_run1/aligned.sorted.sam 1 > star_run1/umi_counts.txt
```

## Evaluation signals

- The three density plots of UMI-read mapping-shift distances overlap across STAR, GSNAP and HISAT2, each showing a pronounced mode with reads mapping to either side, reproducing Fig. 5.
- Each aligner run yields a valid sorted SAM file that umi_counterizer.pl can consume, producing all_sam_umis.txt and collapsed_sam_umis.txt logs.
- Each umi_positions.txt from umi_analyzer_uthash parses correctly (space-delimited UMI plus one or more chromosome:position strings), confirming identical downstream parsing across alignments.
- The basic shift pattern (mode position plus symmetric flanking reads) is qualitatively the same in all three alignments, demonstrating aligner independence.

## Limitations

- No version numbers are given for any of the aligners or scripts, so exact reproducibility of alignment details is not guaranteed.
- The 3 bp separation threshold for calling reads 'same starting mRNA' is an arbitrary, configurable parameter; the true distribution of mapping shift sizes is unknown and shifts larger than 3 bp could occur.
- Downstream scripts assume single-end sequencing, a fixed UMI length, and strict input formats ('There is no flexibility here, and not much error checking' in umi_analyzer_uthash).

## Evidence

- [methods] We use [STAR](https://github.com/alexdobin/STAR) specifying no introns.: "We use [STAR](https://github.com/alexdobin/STAR) specifying no introns."
- [results] Shift pattern aligner-independent: [evidence span withheld — anchor status: not_found]
- [discussion] 3 bp same-mRNA threshold: "we considered reads to be derived from the same starting mRNA if they share a UMI and the mapping locations are separated by three or fewer base pairs"
- [readme] STAR alignment output requirement: "Do this however you want. We use [STAR](https://github.com/alexdobin/STAR) specifying no introns. To get to the next step you need a sorted SAM file."
- [readme] umi_counterizer collapse rule: [evidence span withheld — anchor status: external_doc]
