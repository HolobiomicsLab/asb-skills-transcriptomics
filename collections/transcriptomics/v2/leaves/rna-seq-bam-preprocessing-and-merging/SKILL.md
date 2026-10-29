---
name: rna-seq-bam-preprocessing-and-merging
description: Use when you have mapped RNA-seq reads (bulk or merged single-cell BAM)
  and need a BAF shift profile without a heterozygous variant call set, e.g. to supply
  CaSpER's allelic-shift signal.
when_to_use_negative:
- Input is raw FASTQ reads rather than an aligned, sorted BAM — reads must already
  be mapped before samtools view streaming.
- You already have a high-quality heterozygous SNV call set and want DNA-based BAF
  estimation instead of BAFExtract's read-derived approach.
- The sample is 3′-only sequencing (e.g. DropSeq) where the paper notes the BAF shift
  signal may be adversely impacted because only variants near gene 3′ ends are measured.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: BAFExtract
  role: 'Generates the B-allele frequency shifts from mapped reads alone: -generate_compressed_pileup_per_SAM
    builds the pileup, -get_SNVs_per_pileup produces the per-SNP BAF shift profile.'
  repo: https://github.com/akdess/BAFExtract
- name: samtools
  role: Streams mapped reads from the input BAM (samtools view) into BAFExtract's
    pileup generator; also used to merge single-cell BAM files.
- name: CaSpER
  role: Consumes the BAFExtract output via readBAFExtractOutput and applies the multiscale
    (loh.scale=3) per-chromosome median filter to the BAF shift signal for LOH-informed
    CNV calling.
  repo: https://github.com/akdess/CaSpER
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-13779-x
    title: CaSpER identifies and visualizes CNV events by integrative analysis of
      single-cell or bulk RNA-sequencing data
  source_content_hash: sha256:d6258ae35c53f367d8d38fe548643c2de472dfc4a0f7984f768907d8a0706d14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/akdess/BAFExtract
  tools:
  - BAFExtract
  - CaSpER
  - bamtools
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# rna-seq-bam-preprocessing-and-merging

## Summary

Generate a genome-wide B-allele frequency (BAF) shift signal from mapped RNA-seq BAM reads using BAFExtract (two-step compressed pileup then per-SNV calling), for downstream multiscale smoothing and LOH-informed CNV correction in CaSpER.

## When to use

Use when you have mapped RNA-seq reads (bulk or merged single-cell BAM) and need a BAF shift profile without a heterozygous variant call set, e.g. to supply CaSpER's allelic-shift signal.

## When NOT to use

- Input is raw FASTQ reads rather than an aligned, sorted BAM — reads must already be mapped before samtools view streaming.
- You already have a high-quality heterozygous SNV call set and want DNA-based BAF estimation instead of BAFExtract's read-derived approach.
- The sample is 3′-only sequencing (e.g. DropSeq) where the paper notes the BAF shift signal may be adversely impacted because only variants near gene 3′ ends are measured.

## Inputs

- Mapped RNA-seq BAM file (bulk) or multiple single-cell BAM files to merge
- Genome SNP/chromosome size list file (e.g. hg38.list, hg19.list via fetchChromSizes)
- Genome FASTA pileup directory (preprocessed via BAFExtract -preprocess_FASTA or downloaded for hg19/hg38)

## Outputs

- BAF shift signal file per sample (e.g. test.baf / test.snp) with per-SNP allele-frequency shift entries
- Multiscale smoothed BAF shift signal (per-chromosome median-filtered, loh.scale=3) loadable by CaSpER

## How to apply

For single-cell studies, first merge individual cell BAM files into one BAM (Step 1). Then stream mapped reads with 'samtools view' and pipe into BAFExtract -generate_compressed_pileup_per_SAM over the genome SNP list (e.g. hg38.list), using a mapping quality cutoff of 50 (adjust this since aligners differ in MAPQ semantics and a cutoff of 50 may remove all reads). Next run BAFExtract -get_SNVs_per_pileup against the preprocessed genome FASTA pileup directory (hg19/hg38, or created via -preprocess_FASTA for other genomes) with filter parameters: minimum pileup depth 20, minimum supporting reads per allele 4, and BAF deviation threshold 0.1, writing the BAF shift signal to the output file. Finally, in R read the output with readBAFExtractOutput(sequencing.type='bulk') and apply the recursive per-chromosome median filter across multiscale windows (loh.scale=3, e.g. lohCallMedianFilterByChr) to obtain the smoothed BAF shift signal used by CaSpER's CNV calling.

## Related tools

- **BAFExtract** (Generates the B-allele frequency shifts from mapped reads alone: -generate_compressed_pileup_per_SAM builds the pileup, -get_SNVs_per_pileup produces the per-SNP BAF shift profile.) — https://github.com/akdess/BAFExtract
- **samtools** (Streams mapped reads from the input BAM (samtools view) into BAFExtract's pileup generator; also used to merge single-cell BAM files.)
- **CaSpER** (Consumes the BAFExtract output via readBAFExtractOutput and applies the multiscale (loh.scale=3) per-chromosome median filter to the BAF shift signal for LOH-informed CNV calling.) — https://github.com/akdess/CaSpER

## Examples

```
mkdir test; samtools view SRR1295366.sorted.bam | ./bin/BAFExtract -generate_compressed_pileup_per_SAM stdin hg38.list test 50 0; ./bin/BAFExtract -get_SNVs_per_pileup hg38.list test ./hg38/ 20 4 0.1 test.baf
```

## Evaluation signals

- The output BAF file (e.g. test.baf) exists and is parseable by readBAFExtractOutput(sequencing.type='bulk').
- Entries carry per-SNP BAF values computed under the specified filters (depth >= 20, >= 4 supporting reads per allele, BAF deviation threshold 0.1).
- The pileup step used MAPQ 50; verify reads survive — README warns a MAPQ 50 cutoff can filter out all reads depending on the aligner's MAPQ implementation.
- After readBAFExtractOutput, the smoothed signal spans multiple chromosomes and scales consistent with loh.scale=3 recursive median filtering.
- For other genomes, the FASTA pileup directory was built with BAFExtract -preprocess_FASTA and matches the genome_list coordinates.

## Limitations

- Setting the minimum mapping quality threshold to 50 could mean no reads survive filtering, depending on the aligner's MAPQ implementation.
- With 3′-only sequencing (e.g. DropSeq), the BAF shift signal may be adversely impacted because BAF shift is measured only for variants close to the 3′ ends of genes; the authors believe read pooling soothes this bias but it is not validated.
- In single-cell data, cells with substantially large numbers of reads may dominate the pooled BAF shift signal and create biases; RNA editing, PCR duplication artefacts, and sequencing errors may also bias the BAF shift signal and hence CNV detection.
- CaSpER lacks a BAF shift generation model accounting for 3′ bias, proposed as a future improvement.

## Evidence

- [readme] BAFExtract generates the B-allele frequency shifts from mapped reads alone without the need for a heterozygous variant call set.: [evidence span withheld — anchor status: external_doc]
- [readme] samtools view <bam_file> | ./BAFExtract -generate_compressed_pileup_per_SAM stdin <genome_list> <sample_dir> 50 0; ./BAFExtract -get_SNVs_per_pileup <genome_list> <sample_dir> <genome_fasta_pileup_dir> 20 4 0.1 <output_baf_file>: "samtools view <bam_file> | ./BAFExtract -generate_compressed_pileup_per_SAM stdin <genome_list> <sample_dir> 50 0; ./BAFExtract -get_SNVs_per_pileup  <genome_list> <sample_dir>"
- [methods] Step 1. Merge single cell RNA-Seq bam files (required only for single-cell studies); Step 2. Extract BAF values from RNA-Seq bam files: "Step 1. Merge single cell RNA-Seq bam files (required only for single-cell studies)"
- [methods] loh <- readBAFExtractOutput ( path="./meningioma_baf\", sequencing.type="bulk"): "loh <- readBAFExtractOutput ( path="./meningioma_baf\", sequencing.type="bulk")"
- [discussion] the BAF shift signal may be adversely impacted because BAF shift will be measured for only the variants that are located close to the 3′ ends of the genes.: "the BAF shift signal may be adversely impacted because BAF shift will be measured for only the variants that are located close to the 3′ ends of the genes."
