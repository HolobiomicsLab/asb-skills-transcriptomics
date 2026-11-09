---
name: paired-end-fragment-extraction
description: Use when STAR-aligned paired-end scRNA-seq BAMs (e.g., Smart-seq2 data
  like gray mouse lemur testes cells) must be converted to strand-specific interval
  fragments before groHMM TAR discovery, because groHMM consumes read coverage intervals
  rather than raw BAM alignments.
when_to_use_negative:
- Input is already a fragment/interval file or a single-cell TAR expression matrix
  rather than aligned paired-end BAMs
- Data are unaligned FASTQs or 10x-style aligned outputs — start instead from the
  from_fastqs or from_cellranger Snakemake workflows
- Sequencing is single-end, where -bedpe paired-fragment extraction does not apply
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: bedtools
  role: Extract paired-end read fragments from STAR-aligned BAMs via bamtobed -bedpe
- name: STAR
  role: Produces the paired-end BAM alignments that this skill consumes
- name: groHMM
  role: Consumes the extracted fragments to call strand-specific TARs per cell
  repo: https://bioconductor.org/packages/release/bioc/html/groHMM.html
- name: rtracklayer
  role: Loads annotation/bed files for aTAR vs uTAR labeling after TAR calling
  repo: https://bioconductor.org/packages/release/bioc/html/rtracklayer.html
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:fb116003596a9f8843c6509cdd9ac12ef04269cc8c37afd5c9e0355e3b569dce
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - STAR
  - Seurat
  - Snakemake
  - bedtools
  - featureCounts
  - groHMM
  - rtracklayer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# paired-end-fragment-extraction

## Summary

Extracts strand-aware read fragments from STAR-aligned paired-end single-cell RNA-seq BAM files (e.g., Smart-seq2) using bedtools bamtobed with the -bedpe option, converting aligned pairs into interval inputs suitable for groHMM transcriptionally active region (TAR) calling.

## When to use

Use when STAR-aligned paired-end scRNA-seq BAMs (e.g., Smart-seq2 data like gray mouse lemur testes cells) must be converted to strand-specific interval fragments before groHMM TAR discovery, because groHMM consumes read coverage intervals rather than raw BAM alignments.

## When NOT to use

- Input is already a fragment/interval file or a single-cell TAR expression matrix rather than aligned paired-end BAMs
- Data are unaligned FASTQs or 10x-style aligned outputs — start instead from the from_fastqs or from_cellranger Snakemake workflows
- Sequencing is single-end, where -bedpe paired-fragment extraction does not apply

## Inputs

- STAR-aligned paired-end single-cell RNA-seq BAM files (one per cell, e.g., Smart-seq2)
- Reference genome annotation for downstream aTAR/uTAR labeling (refFlat/gtf)

## Outputs

- Per-cell paired-end read fragment interval files (bedpe-derived BED fragments) ready for groHMM
- Per-cell groHMM TAR calls (BED), later labeled aTAR/uTAR and exported as refFlat and gtf annotations

## How to apply

For each single-cell sequencing file aligned with STAR in paired-end mode, run bedtools bamtobed with the -bedpe option set to extract the mate-linked read fragments as interval records. Feed these strand-specific fragments into groHMM, which uses a hidden Markov model to divide the genome into transcribed and untranscribed states based on read coverage in each bin, generating TARs per cell. TARs are subsequently labeled as aTAR or uTAR by overlap with existing gene annotations via rtracklayer, and features are counted per cell with featureCounts using strandedness 1 (-s 1) and the paired-end option (-p). The rationale is that paired-end fragment extraction preserves the strand and span of the original fragment, which is required for correct strand-specific TAR boundary detection in full-length Smart-seq2 data.

## Related tools

- **bedtools** (Extract paired-end read fragments from STAR-aligned BAMs via bamtobed -bedpe)
- **STAR** (Produces the paired-end BAM alignments that this skill consumes)
- **groHMM** (Consumes the extracted fragments to call strand-specific TARs per cell) — https://bioconductor.org/packages/release/bioc/html/groHMM.html
- **rtracklayer** (Loads annotation/bed files for aTAR vs uTAR labeling after TAR calling) — https://bioconductor.org/packages/release/bioc/html/rtracklayer.html

## Examples

```
bedtools bamtobed -bedpe -i cell_aligned.bam > cell_fragments.bed
```

## Evaluation signals

- Each output fragment record corresponds to a read pair in the source BAM, with strand and span consistent with the original paired-end fragment
- Downstream groHMM run completes and emits chr*_HMM.bed files; absence of these files with 'no lines available in input' in calcHMMrefFlat indicates empty/failed fragment extraction
- Strand specificity holds: featureCounts on the resulting TARs is run with -s 1 and -p and yields non-degenerate per-cell count vectors
- On the Smart-seq2 gray mouse lemur testes benchmark, the resulting uTAR matrix supports differential calls matching Supplementary Fig. 2b (CCER1 in spermatocytes, TBXAS1 in spermatogonium, ncRNA LOC105867359 in spermatids)

## Limitations

- groHMM R script failures producing empty chr*_HMM.bed files ('no lines available in input') are a known failure mode, typically caused by groHMM or rtracklayer not being installed/loadable
- Skill assumes paired-end, STAR-aligned input; the reference workflows otherwise start from raw fastq (from_fastqs) or cellranger count output (from_cellranger)

## Evidence

- [methods] Extract read fragments from the paired-end data using bedtools bamtobed with the '-bedpe' option set.: [evidence span withheld — anchor status: not_found]
- [intro] groHMM HMM binning rationale: "groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin"
- [readme] Workflow entry points: "Current workflows support starting with either raw fastq files (*from_fastqs*) or if you have already aligned with cellranger count (*from_cellranger*)."
- [readme] groHMM failure mode: "This error indicates that the groHMM R script did not finish running to generate groHMM bed files."
