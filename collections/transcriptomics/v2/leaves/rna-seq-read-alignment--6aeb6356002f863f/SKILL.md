---
name: rna-seq-read-alignment
description: Use when you have raw FASTQ reads (paired-end, e.g., first-strand Illumina
  libraries) and need a sorted, genome-aligned BAM as input to downstream transcriptome
  reconstruction (StringTie) or aptardi polyA-site prediction.
when_to_use_negative:
- The input is already a genome-aligned sorted BAM from a previous run — re-aligning
  adds nothing downstream
- The goal is quantifying expression against a fixed transcriptome only (no transcript
  reconstruction or 3'-end refinement planned) — a direct quantification workflow
  such as Salmon/RSEM on an existing GTF is more appropriate
- The reads are unstranded and/or from a non-mammalian species outside the scope tested
  (the paper notes aptardi performed modestly worse on the unstranded, cross-species
  BNLx/SHR data and that engineered DNA features may not apply outside mammals)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: HISAT2
  role: Splice-aware alignment of trimmed RNA-Seq reads to hg38 or mm10 with --rna-strandness
    RF and --dta flags so alignments support transcript assembly
- name: SAMtools
  role: Sorts HISAT2 alignments into a coordinate-sorted BAM (sorted.bam) required
    by StringTie and aptardi; should be installed outside the aptardi conda environment
  repo: https://github.com/luskry/aptardi
- name: FastQC
  role: Assesses raw read quality to determine which adapter/trimming steps are needed
    before alignment
- name: Cutadapt
  role: Trims adapters and low-quality bases flagged by FastQC prior to alignment
provenance:
  source_task_ids:
  - task_007
  - task_003
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:0cd3aaf39b1cd233fe7378e71a3b9b00272512bab0a60ecfc095cb195f39e77a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luskry/aptardi
  tools:
  - Arriba
  - BEDtools
  - Cutadapt
  - FastQC
  - HISAT2
  - SAMtools
  - STAR
  - StringTie
  - TSNAD v2.0
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# rna-seq-read-alignment

## Summary

Splice-aware alignment of RNA-Seq reads to a reference genome with HISAT2 in transcriptome-assembly mode (e.g., --rna-strandness RF --dta), followed by SAMtools sorting, producing the sorted BAM that StringTie reconstruction and aptardi polyA prediction consume.

## When to use

Use when you have raw FASTQ reads (paired-end, e.g., first-strand Illumina libraries) and need a sorted, genome-aligned BAM as input to downstream transcriptome reconstruction (StringTie) or aptardi polyA-site prediction.

## When NOT to use

- The input is already a genome-aligned sorted BAM from a previous run — re-aligning adds nothing downstream
- The goal is quantifying expression against a fixed transcriptome only (no transcript reconstruction or 3'-end refinement planned) — a direct quantification workflow such as Salmon/RSEM on an existing GTF is more appropriate
- The reads are unstranded and/or from a non-mammalian species outside the scope tested (the paper notes aptardi performed modestly worse on the unstranded, cross-species BNLx/SHR data and that engineered DNA features may not apply outside mammals)

## Inputs

- FASTQ reads (single- or paired-end, e.g., Illumina first-strand libraries)
- HISAT2 genome index (e.g., for hg38 or mm10)
- Quality/adapter assessment from FastQC guiding Cutadapt trimming

## Outputs

- Coordinate-sorted BAM of genome-aligned RNA-Seq reads (sorted.bam) suitable for StringTie and aptardi Mode 1

## How to apply

First assess read quality with FastQC and trim adapters/low-quality sequence with Cutadapt based on what the quality reports show. Align the trimmed reads to the reference genome (mm10 for the mouse brain-vs-liver analysis, hg38 for the human HBR/UHR/BNLx/SHR data) using HISAT2 with RNA-Seq options: -q --reorder -t --rna-strandness RF (matching the first-strand library protocol) and --dta so alignments carry the metadata StringTie needs for transcript assembly. Sort the resulting SAM into a coordinate-sorted BAM with SAMtools (v.1.9), since aptardi and StringTie both require sorted BAM input. Verify the install context first: aptardi's bioconda environment warns that the bioconda samtools and bedtools packages may prevent aptardi from working, so keep SAMtools/BEDtools outside the aptardi conda environment.

## Related tools

- **HISAT2** (Splice-aware alignment of trimmed RNA-Seq reads to hg38 or mm10 with --rna-strandness RF and --dta flags so alignments support transcript assembly)
- **SAMtools** (Sorts HISAT2 alignments into a coordinate-sorted BAM (sorted.bam) required by StringTie and aptardi; should be installed outside the aptardi conda environment) — https://github.com/luskry/aptardi
- **FastQC** (Assesses raw read quality to determine which adapter/trimming steps are needed before alignment)
- **Cutadapt** (Trims adapters and low-quality bases flagged by FastQC prior to alignment)

## Examples

```
hisat2 -q -p 5 --reorder -t --rna-strandness RF --dta -x <hisat2_index> -1 sample_R1.fq.gz -2 sample_R2.fq.gz | samtools sort -o sorted.bam
```

## Evaluation signals

- FastQC reports show no adapter contamination or overrepresented low-quality sequences after Cutadapt trimming
- HISAT2 summary reports a reasonable overall alignment rate, consistent with reference (hg38 or mm10) and strandedness settings
- The output BAM is coordinate-sorted and readable by samtools view/quickcheck
- Downstream sanity: the sorted BAM produces a stringtie.gtf with the expected flag (--rf -G guide) and, in the reproduced studies, supports recovery of validation values such as 26 brain-specific / 69 liver-specific unannotated polyA sites or CCND1/DICER1/TIMP2 proximal APA calls
- aptardi accepts the BAM without orientation errors, confirming the --rna-strandness/--a fr setting matches the library protocol

## Limitations

- The strandness flag must match the library protocol (RF for the first-strand Illumina libraries used here); the article notes it could not rule out library- strandedness issues on the unstranded BNLx/SHR data sets
- Bioconda samtools/bedtools packages inside the aptardi conda environment can break aptardi, per the README requirement note
- Alignment quality depends on upstream quality assessment; the paper only recommends 'performing any processing steps based on the results' rather than prescribing fixed trim thresholds

## Evidence

- [methods] Align reads to genome with HISAT2 and sort bam with SAMtools: [evidence span withheld — anchor status: not_found]
- [methods] StringTie command line from the paper's workflow: "stringtie sorted.bam --rf -o stringtie.gtf -G <guide_file>"
- [methods] FastQC/Cutadapt recommendation: [evidence span withheld — anchor status: not_found]
- [readme] Readme warning about bioconda samtools/bedtools in the aptardi environment: "The bioconda samtools and bedtools packages may prevent aptardi from working, please make sure these packages are not installed in the same environment as aptardi"
- [readme] Readme orientation default for paired-end alignments: "--a/-a <fr or rf> Upstream/downstream mate orientations for paired-end alignment against the forward reference strand, fr = firststrand"
