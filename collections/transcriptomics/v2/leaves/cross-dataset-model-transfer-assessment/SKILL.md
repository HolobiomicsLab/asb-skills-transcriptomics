---
name: cross-dataset-model-transfer-assessment
description: Use when a pre-trained model (e.g. aptardi's HBR model.hdf5) must be
  tested for generalizability across RNA-Seq datasets differing in tissue, organism,
  library preparation, or sequencing depth.
when_to_use_negative:
- The goal is building a custom model from gold-standard polyA sites rather than testing
  the pre-built HBR model — use aptardi Mode 2 instead of Mode 1 transfer evaluation.
- The data are non-mammalian taxa, since the manually engineered DNA sequence features
  may not apply outside of mammals and will require further research.
- The evaluation question is de novo polyA site discovery rather than measuring whether
  training-set performance is retained on held-out datasets.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: aptardi
  role: Runs in Mode 1 with the pre-built HBR model (model.hdf5) and scale.pk to generate
    aptardi-modified transcriptomes on each external dataset
  repo: github.com/luskry/aptardi
- name: HISAT2
  role: Aligns external dataset reads to hg38 with the appropriate --rna-strandness
    setting (e.g. RF for first-strand libraries)
- name: SAMtools
  role: Sorts HISAT2 alignments into sorted.bam for each dataset before aptardi input
- name: StringTie
  role: Reconstructs a sample-specific transcriptome (stringtie.gtf) from each sorted.bam
    using the Ensembl guide annotation
- name: RSEM
  role: Quantitates each aptardi transcriptome so the 2/3-samples count filter can
    remove low-quality aptardi transcripts before benchmarking
- name: FastQC
  role: Assesses read quality of downloaded SRA FASTQ files prior to alignment
- name: Cutadapt
  role: Trims adapters from reads based on FastQC results
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-21894-x
    title: Aptardi predicts polyadenylation sites in sample-specific transcriptomes
      using high-throughput RNA sequencing and DNA sequence
  source_content_hash: sha256:ba9e191589e7ab316d31555cd557697437eab4c11cb5ffe8de76d581eb1c73e6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cutadapt
  - FastQC
  - HISAT2
  - RSEM
  - SAMtools
  - StringTie
  - aptardi
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cross-dataset-model-transfer-assessment

## Summary

Benchmarks whether a machine-learning model trained on one RNA-Seq dataset (aptardi's Human Brain Reference, HBR) retains accuracy when applied to independent RNA-Seq datasets, by comparing average precision (AP) and F-measure of predicted polyA sites against gold-standard 3'-end sites in each external dataset.

## When to use

Use when a pre-trained model (e.g. aptardi's HBR model.hdf5) must be tested for generalizability across RNA-Seq datasets differing in tissue, organism, library preparation, or sequencing depth.

## When NOT to use

- The goal is building a custom model from gold-standard polyA sites rather than testing the pre-built HBR model — use aptardi Mode 2 instead of Mode 1 transfer evaluation.
- The data are non-mammalian taxa, since the manually engineered DNA sequence features may not apply outside of mammals and will require further research.
- The evaluation question is de novo polyA site discovery rather than measuring whether training-set performance is retained on held-out datasets.

## Inputs

- FASTQ files for external RNA-Seq samples (2nd HBR, UHR, BNLx, SHR) from SRA
- hg38 reference genome FASTA (hg38.fa)
- Ensembl guide annotation (GTF)
- Pre-built aptardi model (model.hdf5) and scale file (scale.pk) from src/aptardi/ml_scale
- Gold-standard 3'-end polyA site sets for each external dataset

## Outputs

- Sample-specific aptardi-modified transcriptome (name_aptardi.gtf) per dataset
- RSEM quantitation of filtered, high-quality aptardi transcriptomes
- Per-dataset average precision (AP) and F-measure values
- Combined benchmark table (aptardi_ap_f_benchmark.csv)

## How to apply

Process each external dataset (2nd HBR, UHR, BNLx, SHR) with an identical pipeline: download FASTQ from SRA, assess reads with FastQC and trim adapters with Cutadapt, align to hg38 with HISAT2 using the correct --rna-strandness setting per library preparation (e.g. RF for first-strand Illumina libraries), and sort with SAMtools. Reconstruct a sample-specific transcriptome per dataset with StringTie using the Ensembl guide annotation, then run aptardi in Mode 1 with the pre-built HBR model (model.hdf5) and scale file (scale.pk) from src/aptardi/ml_scale to produce a name_aptardi.gtf for each dataset. Quantitate each aptardi transcriptome with RSEM and apply the conservative filter removing transcripts without at least one count in 2/3 of samples to retain high-quality aptardi transcripts. Compare predicted polyA sites against each dataset's gold-standard 3'-end sites, computing average precision and F-measure, and assemble per-dataset values into a single CSV for direct comparison against the training-set baseline. The rationale is that similarity of AP between the training set and external sets demonstrates the model is not prone to overfitting and transfers across library preparations, organisms, and sequencing depths.

## Related tools

- **aptardi** (Runs in Mode 1 with the pre-built HBR model (model.hdf5) and scale.pk to generate aptardi-modified transcriptomes on each external dataset) — github.com/luskry/aptardi
- **HISAT2** (Aligns external dataset reads to hg38 with the appropriate --rna-strandness setting (e.g. RF for first-strand libraries))
- **SAMtools** (Sorts HISAT2 alignments into sorted.bam for each dataset before aptardi input)
- **StringTie** (Reconstructs a sample-specific transcriptome (stringtie.gtf) from each sorted.bam using the Ensembl guide annotation)
- **RSEM** (Quantitates each aptardi transcriptome so the 2/3-samples count filter can remove low-quality aptardi transcripts before benchmarking)
- **FastQC** (Assesses read quality of downloaded SRA FASTQ files prior to alignment)
- **Cutadapt** (Trims adapters from reads based on FastQC results)

## Examples

```
aptardi --o <outdir> --f hg38.fa --r stringtie.gtf --b sorted.bam --n model.hdf5 --t scale.pk --a rf --g name_aptardi.gtf
```

## Evaluation signals

- Reproduced AP for each dataset matches reported values within reasonable tolerance: HBR 0.65, 2nd HBR 0.63, UHR 0.63, BNLx 0.55, SHR 0.55
- Reproduced F-measure matches reported values: HBR 0.56, 2nd HBR 0.51, UHR 0.53, BNLx 0.47, SHR 0.48
- Average precision on the four external datasets falls between 0.55 and 0.63, close to the 0.65 training-set baseline, indicating consistent cross-sample generalizability
- A name_aptardi.gtf exists for every processed dataset and the filtered transcriptome retains only transcripts with at least 1 count in 2/3 of samples
- AP estimates on training and testing sets are similar, demonstrating the model is not prone to overfitting

## Limitations

- aptardi performed modestly worse on the BNLx and SHR datasets; this is hypothesized to stem from true polyA sites derived from Sprague Dawley rat rather than the specific strain, but unstranded RNA-Seq and/or different species cannot be ruled out
- aptardi makes predictions on 100 base bins (annotating the 3'-most base in the bin), a resolution limitation; do not alter --w with the pre-built model since it was built using 100 base bins
- Upstream exons are not analyzed, eliminating the possibility of identifying coding APA; a single 3'-end may also be incorporated into multiple transcripts per gene, so filtering is highly recommended

## Evidence

- [discussion] the aptardi prediction model provided for users (built from the HBR data set) performs equally well on RNA-Seq data sets: "the aptardi prediction model provided for users (built from the HBR data set) performs equally well on RNA-Seq data sets"
- [discussion] performs equally well on RNA-Seq data sets derived from different library preparations, organisms, and with different RNA-sequencing depths: "performs equally well on RNA-Seq data sets derived from different library preparations, organisms, and with different RNA-sequencing depths"
- [discussion] the similarity of the AP estimates between training and testing sets demonstrate that aptardi is not prone to overfitting: "the similarity of the AP estimates between training and testing sets demonstrate that aptardi is not prone to overfitting"
- [readme] we found a relatively conservative filter of removing transcripts without at least one count in 2/3 of samples does a good job of identifying high quality aptardi transcripts: "we found a relatively conservative filter of removing transcripts without at least one count in 2/3 of samples does a good job of identifying high quality aptardi transcripts"
- [abstract] The average precision of the aptardi model is twice that of a standard transcriptome assembler.: "The average precision of the aptardi model is twice that of a standard transcriptome assembler."
- [readme] we DO NOT recommend altering this argument using the pre-built model since this model was built using 100 base bins: "we DO NOT recommend altering this argument using the pre-built model since this model was built using 100 base bins"
