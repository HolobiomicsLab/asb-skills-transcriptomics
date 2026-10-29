---
name: spike-in-read-alignment-to-ercc-reference
description: 'Use when validating droplet-based scRNA-seq sensitivity with ERCC synthetic
  spike-ins: FASTQ reads from spike-in-only GEMs must be aligned to the ERCC reference
  (not a genome) to count UMIs per ERCC transcript.'
when_to_use_negative:
- Input FASTQs are from cellular samples and must be quantified against hg19/mm10
  gene references with the full Cell Ranger pipeline — ERCC-only alignment would discard
  the transcriptome signal.
- The goal is absolute ERCC molecule quantification or spike-in normalization of expression
  levels, not assessment of the droplet platform's conversion efficiency and UMI-to-molecule
  correlation.
- Read2/Read1 structure differs from the 3' counting chemistry described here (e.g.
  earlier libraries with 5 nt UMI and 5 bp Read2) without adjusting barcode/UMI extraction
  accordingly.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
tools:
- name: STAR
  role: Aligns Read1 (cDNA insert) reads to the ERCC reference index built from cms_095047.txt
  repo: https://github.com/10XGenomics/single-cell-3prime-paper
- name: Cell Ranger Single-Cell Software Suite
  role: Reference pipeline for sample demultiplexing, barcode processing, and single-cell
    3' gene counting; supplies the barcode/UMI validity and correction rules replicated
    for ERCC counting
  repo: https://github.com/10XGenomics/cellranger
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:f88f693c3ca8d88dc296ad263149ac6ddc68fd9906e8dcbe09b262f68690f313
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/single-cell-3prime-paper
  tools:
  - Cell Ranger Single-Cell Software Suite
  - STAR
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# spike-in-read-alignment-to-ercc-reference

## Summary

Aligns scRNA-seq Read1 (cDNA insert) reads from ERCC spike-in GEM runs to the ERCC synthetic RNA reference so that UMI counts per ERCC transcript can be recovered and conversion efficiency quantified. This is how the Zheng et al. droplet 3' counting system's sensitivity (6.7–8.1% conversion, r=0.96) was validated.

## When to use

Use when validating droplet-based scRNA-seq sensitivity with ERCC synthetic spike-ins: FASTQ reads from spike-in-only GEMs must be aligned to the ERCC reference (not a genome) to count UMIs per ERCC transcript.

## When NOT to use

- Input FASTQs are from cellular samples and must be quantified against hg19/mm10 gene references with the full Cell Ranger pipeline — ERCC-only alignment would discard the transcriptome signal.
- The goal is absolute ERCC molecule quantification or spike-in normalization of expression levels, not assessment of the droplet platform's conversion efficiency and UMI-to-molecule correlation.
- Read2/Read1 structure differs from the 3' counting chemistry described here (e.g. earlier libraries with 5 nt UMI and 5 bp Read2) without adjusting barcode/UMI extraction accordingly.

## Inputs

- ERCC spike-in GEM sequencing FASTQ files (Mix1 and Mix2 at 1:50 dilution) from SRA SRP073767 or support.10xgenomics.com/single-cell/datasets
- ERCC reference transcript sequences in FASTA from tools.thermofisher.com/content/sfs/manuals/cms_095047.txt
- Expected ERCC molecule counts per transcript as loaded into the GEMs

## Outputs

- STAR alignments of Read1 cDNA-insert reads against the ERCC reference (BAM/SAM)
- Per-ERCC-transcript valid cell-barcode/UMI count table
- Recomputed sensitivity metrics: inferred conversion efficiency per experiment, Pearson r between mean UMI counts and expected ERCC molecule counts, and CV2 across ERCC experiments (e.g. written to ercc_sensitivity_metrics.csv)

## How to apply

Download the ERCC Mix1 and Mix2 (1:50 dilution) GEM FASTQs (SRA SRP073767, mirrored at support.10xgenomics.com/single-cell/datasets) and the ERCC reference sequences from tools.thermofisher.com/content/sfs/manuals/cms_095047.txt, then build a STAR index on the ERCC reference alone. Align Read1 — the read carrying the cDNA insert in this 3' counting chemistry — with STAR. Count valid cell-barcode/UMI combinations per ERCC transcript using the paper's UMI validity rules: quality score >10, non-homopolymer, and 1-Hamming-distance correction to the more-abundant UMI for the same cell barcode and gene. Compute inferred conversion efficiency as the fraction of expected ERCC molecules represented by counted UMIs, plus the Pearson correlation of mean UMI counts versus expected molecule counts and the CV2 across ERCC experiments.

## Related tools

- **STAR** (Aligns Read1 (cDNA insert) reads to the ERCC reference index built from cms_095047.txt) — https://github.com/10XGenomics/single-cell-3prime-paper
- **Cell Ranger Single-Cell Software Suite** (Reference pipeline for sample demultiplexing, barcode processing, and single-cell 3' gene counting; supplies the barcode/UMI validity and correction rules replicated for ERCC counting) — https://github.com/10XGenomics/cellranger

## Examples

```
STAR --genomeDir ercc_star_index --readFilesIn ERCC_mix1_R1.fastq.gz --readFilesCommand zcat --outSAMtype BAM SortedByCoordinate --outFileNamePrefix ercc_mix1_
```

## Evaluation signals

- Inferred conversion efficiency per ERCC experiment falls within the reported 6.7–8.1% range for both Spike-in Mix1 and Mix2 at 1:50 dilution.
- Pearson correlation between mean sequenced UMI counts and expected ERCC molecule counts is approximately 0.96.
- Coefficient of variation squared (CV2) across all ERCC experiments is approximately 7%.
- All counted UMIs pass the validity filters: sequencing quality score >10, non-homopolymer, and 1-Hamming-distance corrected to the more-abundant UMI per cell barcode and gene.
- Output metric file (e.g. ercc_sensitivity_metrics.csv) contains one conversion-efficiency value, one Pearson r, and one CV2 per ERCC experiment with no missing entries.

## Limitations

- Some earlier libraries in this dataset were made with a 5 nt UMI and 5 bp Read2, so UMI extraction parameters must be adjusted per library to avoid miscounting.
- The cDNA conversion efficiency estimate assumes expected ERCC molecule counts loaded into GEMs are accurate; errors in spike-in dilution propagate directly into the 6.7–8.1% estimate.
- Only confidently mapped, non-PCR-duplicate reads with valid barcodes and UMIs should contribute to counts; relaxing MAPQ or duplicate filters will inflate UMI counts and the apparent conversion efficiency.

## Evidence

- [methods] ERCC reference alignment: "For ERCC samples, ERCC reference (https://tools.thermofisher.com/content/sfs/manuals/cms_095047.txt) was used."
- [methods] STAR alignment of Read1: "Read1, which contains the cDNA insert, was aligned to an appropriate reference genome using STAR35."
- [methods] UMI validity and correction rules: "UMIs with sequencing quality score >10 were considered valid if they were not homopolymers."
- [methods] UMI 1-Hamming correction: "A UMI that is 1-Hamming-distance away from another UMI (with more reads) for the same cell barcode and gene is corrected to the UMI with more reads."
- [discussion] Data availability: "Data are also available at http://support.10xgenomics.com/single-cell/datasets."
- [readme] Dataset download location: [evidence span withheld — anchor status: not_found]
