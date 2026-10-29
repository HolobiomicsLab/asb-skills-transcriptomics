---
name: baf-shift-signal-extraction
description: Use when RNA-seq BAM files (bulk or single-cell) must be converted into
  per-sample BAF shift profiles to supply the LOH/allelic-shift input for CaSpER CNV
  calling.
when_to_use_negative:
- You have only a normalized expression matrix with no BAM files, so no read-level
  base observations exist from which to compute BAF shifts
- Your goal is germline SNV genotyping rather than somatic CNV/LOH detection from
  RNA-seq signal profiles
- Your data are 3'-only (e.g., DropSeq-style) with very sparse near-3' variants and
  you cannot pool enough reads to support a usable BAF shift signal
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: BAFExtract
  role: Generates the B-allele frequency shift profile from mapped reads via a two-step
    compressed pileup + SNV-per-pileup extraction; the primary executable for this
    skill
  repo: https://github.com/akdess/BAFExtract
- name: samtools
  role: Pipes the BAM file into the SAM stream consumed by BAFExtract -generate_compressed_pileup_per_SAM
    (stdin)
- name: CaSpER
  role: 'Downstream consumer: loads the BAFExtract output with readBAFExtractOutput
    and integrates it as the allelic-shift signal in CreateCasperObject/runCaSpER'
  repo: https://github.com/akdess/CaSpER
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_007
  - task_008
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-13779-x
    title: CaSpER identifies and visualizes CNV events by integrative analysis of
      single-cell or bulk RNA-sequencing data
  source_content_hash: sha256:9e721b39d6ccf2a721ff98df43394c694e9f7f5ac3442bce934087927e1257b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/akdess/BAFExtract
  tools:
  - BAFExtract
  - CaSpER
  - TCGABiolinks
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# baf-shift-signal-extraction

## Summary

Extracts B-allele frequency (BAF) shift signals from mapped RNA-seq reads using BAFExtract, the allelic-shift/LOH signal source that CaSpER integrates with multiscale-smoothed expression to call CNV events, without requiring a heterozygous variant call set.

## When to use

Use when RNA-seq BAM files (bulk or single-cell) must be converted into per-sample BAF shift profiles to supply the LOH/allelic-shift input for CaSpER CNV calling.

## When NOT to use

- You have only a normalized expression matrix with no BAM files, so no read-level base observations exist from which to compute BAF shifts
- Your goal is germline SNV genotyping rather than somatic CNV/LOH detection from RNA-seq signal profiles
- Your data are 3'-only (e.g., DropSeq-style) with very sparse near-3' variants and you cannot pool enough reads to support a usable BAF shift signal

## Inputs

- Aligned RNA-seq BAM files (bulk samples or merged single-cell BAMs)
- Reference genome FASTA (hg19 or hg38) preprocessed into genome_fasta_pileup_dir via BAFExtract -preprocess_FASTA
- genome_list chromosome-sizes file (e.g., generated with fetchChromSizes hg19 > hg19.list)
- samtools for piping BAM to SAM stream

## Outputs

- Per-sample BAF shift profile file (output_baf_file, e.g., test.snp)
- LOH signal object in R loaded via readBAFExtractOutput, consumed by CreateCasperObject as the allelic-shift input for runCaSpER

## How to apply

Run BAFExtract in two steps per sample: first generate a compressed pileup from the SAM stream (samtools view piped in, minimum mapping quality 50), then run -get_SNVs_per_pileup with the genome_list, preprocessed genome_fasta_pileup_dir (hg19/hg38 from BAFExtract's Dropbox links, or built with -preprocess_FASTA), and parameters 20 4 0.1 to emit the final output BAF file. For single-cell studies, merge the scRNA-seq BAM files first and pool BAF shift per patient via a one-to-many loh.name.mapping, since CaSpER consumes patient-pooled BAF signal. The resulting BAF files are read into R with readBAFExtractOutput(path=..., sequencing.type='bulk') and passed as the loh signal to CreateCasperObject, where they are integrated with HMM segment states from expression at cnv.scale=3/loh.scale=3. Adjust the MAPQ threshold if your aligner's MAPQ values differ, because setting 50 can leave no reads surviving the filter.

## Related tools

- **BAFExtract** (Generates the B-allele frequency shift profile from mapped reads via a two-step compressed pileup + SNV-per-pileup extraction; the primary executable for this skill) — https://github.com/akdess/BAFExtract
- **samtools** (Pipes the BAM file into the SAM stream consumed by BAFExtract -generate_compressed_pileup_per_SAM (stdin))
- **CaSpER** (Downstream consumer: loads the BAFExtract output with readBAFExtractOutput and integrates it as the allelic-shift signal in CreateCasperObject/runCaSpER) — https://github.com/akdess/CaSpER

## Examples

```
samtools view SRR1295366.sorted.bam | ./bin/BAFExtract -generate_compressed_pileup_per_SAM stdin hg38.list test 50 0; ./bin/BAFExtract -get_SNVs_per_pileup hg38.list test ./hg38/ 20 4 0.1 test.snp
```

## Evaluation signals

- A non-empty output BAF file is produced for every sample; if zero reads survive, revisit the MAPQ threshold (50 is aligner-dependent)
- readBAFExtractOutput successfully loads the files and CreateCasperObject accepts them as the loh signal without mapping errors against hg19/hg38 annotation
- Downstream runCaSpER reproduces expected arm-level BAF-expression correlations, e.g., chr1p r2 = -0.43 (P=0.0008) on the yale_meningioma dataset
- CNV accuracy benchmarked with calcROC against a genotyping-array gold standard reaches reported ranges (e.g., 95% TPR / 0.3% FPR large-scale deletions in meningioma; 88% TPR / 3.5% FPR large-scale deletions in TCGA-GBM)
- For single-cell data, pooled per-patient BAF signal yields expected subclone patterns (e.g., MGH31 5q amp vs 14q del) rather than cell-level noise

## Limitations

- 3'-only sequencing (e.g., DropSeq) degrades the BAF shift signal because shifts are measured only for variants near the 3' ends of genes; the read-pooling mitigation is asserted but not validated
- The default minimum mapping quality of 50 is aligner-dependent and can filter out all reads for some aligners
- In single-cell pooled BAF, cells with substantially large read counts may dominate the BAF shift signal and introduce bias
- RNA editing, PCR duplication artefacts, and sequencing errors may impact the BAF shift signal in complex ways and bias CNV detection

## Evidence

- [readme] BAFExtract generates the B-allele frequency shifts from mapped reads alone without the need for a heterozygous variant call set.: [evidence span withheld — anchor status: external_doc]
- [readme] README usage command for BAF extraction: "samtools view <bam_file> | ./BAFExtract -generate_compressed_pileup_per_SAM stdin <genome_list> <sample_dir> 50 0; ./BAFExtract -get_SNVs_per_pileup"
- [readme] MAPQ threshold caveat: [evidence span withheld — anchor status: external_doc]
- [methods] Pipeline step in the article methods: "Step 2. Extract BAF values from RNA-Seq bam files"
- [discussion] 3'-bias limitation: "the BAF shift signal may be adversely impacted because BAF shift will be measured for only the variants that are located close to the 3′ ends of the genes."
- [discussion] Single-cell pooling bias: "If some cells have a substantially large number of reads, they may dominate the BAF shift signal and may potentially create biases."
