---
name: baf-shift-signal-generation-from-rnaseq-bam
description: Use when you have mapped bulk or merged single-cell RNA-seq BAM files
  and need a genome-wide BAF shift signal to correct/confirm CaSpER CNV calls, without
  a pre-existing heterozygous SNV call set.
when_to_use_negative:
- Input is 3'-only sequencing (e.g. DropSeq) — the BAF shift signal is degraded because
  BAF is measured only for variants near the 3' ends of genes
- Input is already an expression matrix or a feature count table with no BAM — allele
  frequencies cannot be computed from reads alone
- You already have high-quality DNA-seq data and a heterozygous variant call set;
  a DNA-based CNV caller is the more direct route
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: BAFExtract
  role: Generates the compressed pileup from streamed SAM records and calls per-SNV
    BAF shifts with depth/support/deviation filters
  repo: https://github.com/akdess/BAFExtract
- name: samtools
  role: Streams mapped reads out of the input BAM into BAFExtract's pileup generator
- name: CaSpER
  role: 'R-side consumption: readBAFExtractOutput(sequencing.type=''bulk'') parses
    the BAF file and the per-chromosome median filter smooths it at loh.scale=3'
  repo: https://github.com/akdess/CaSpER
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-13779-x
    title: CaSpER identifies and visualizes CNV events by integrative analysis of
      single-cell or bulk RNA-sequencing data
  source_content_hash: sha256:aa5018514f684f5f64dbb863f43f7a6658de83e77dfea822235041c9a2dbfeba
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# baf-shift-signal-generation-from-rnaseq-bam

## Summary

Generates a genome-wide B-allele frequency (BAF) shift profile directly from mapped RNA-seq reads using BAFExtract (no heterozygous variant call set required), then smooths it with CaSpER's recursive per-chromosome median filter across multiscale windows (loh.scale=3) for LOH/CNV correction.

## When to use

Use when you have mapped bulk or merged single-cell RNA-seq BAM files and need a genome-wide BAF shift signal to correct/confirm CaSpER CNV calls, without a pre-existing heterozygous SNV call set.

## When NOT to use

- Input is 3'-only sequencing (e.g. DropSeq) — the BAF shift signal is degraded because BAF is measured only for variants near the 3' ends of genes
- Input is already an expression matrix or a feature count table with no BAM — allele frequencies cannot be computed from reads alone
- You already have high-quality DNA-seq data and a heterozygous variant call set; a DNA-based CNV caller is the more direct route

## Inputs

- Mapped RNA-seq BAM file (bulk, or merged single-cell RNA-seq BAMs)
- Genome list / chrom-size SNP list file (e.g. hg38.list or hg19.list from fetchChromSizes)
- Genome FASTA pileup directory (preprocessed with BAFExtract -preprocess_FASTA; prebuilt hg38/hg19 downloads available)

## Outputs

- Per-SNP BAF shift output file (e.g. test.baf / test.snp)
- Smoothed multiscale BAF shift (LOH) signal object from readBAFExtractOutput + per-chromosome median filter (loh.scale=3)

## How to apply

Stream mapped reads with 'samtools view <bam_file>' piping into BAFExtract -generate_compressed_pileup_per_SAM (stdin) over the genome SNP list (e.g. hg38.list from fetchChromSizes) using a minimum mapping quality cutoff of 50. Then run BAFExtract -get_SNVs_per_pileup against the preprocessed genome FASTA pileup directory (built via BAFExtract -preprocess_FASTA) with filter parameters: minimum pileup depth 20, minimum supporting reads per allele 4, and BAF deviation threshold 0.1, writing the BAF shift file (e.g. test.baf). Load the output in R with readBAFExtractOutput(path, sequencing.type='bulk') and apply the recursive per-chromosome median filter across multiscale windows (loh.scale=3) to obtain the smoothed BAF shift signal used downstream for allelic-shift integration with expression HMM states.

## Related tools

- **BAFExtract** (Generates the compressed pileup from streamed SAM records and calls per-SNV BAF shifts with depth/support/deviation filters) — https://github.com/akdess/BAFExtract
- **samtools** (Streams mapped reads out of the input BAM into BAFExtract's pileup generator)
- **CaSpER** (R-side consumption: readBAFExtractOutput(sequencing.type='bulk') parses the BAF file and the per-chromosome median filter smooths it at loh.scale=3) — https://github.com/akdess/CaSpER

## Examples

```
mkdir test; samtools view SRR1295366.sorted.bam | ./bin/BAFExtract -generate_compressed_pileup_per_SAM stdin hg38.list test 50 0; ./bin/BAFExtract -get_SNVs_per_pileup hg38.list test ./hg38/ 20 4 0.1 test.snp
```

## Evaluation signals

- The output BAF file (e.g. test.baf) exists and is parseable by readBAFExtractOutput(sequencing.type='bulk')
- Per-SNP entries respect the specified filters: minimum pileup depth 20, minimum 4 supporting reads per allele, BAF deviation threshold 0.1
- Reads entering the pileup passed the minimum mapping quality threshold 50 set in the -generate_compressed_pileup_per_SAM call
- The smoothed signal spans multiple scales consistent with loh.scale=3 (recursive per-chromosome median filtering)
- BAM chromosome naming is consistent with the genome list (hg38.list/hg19.list) and the preprocessed FASTA pileup directory

## Limitations

- MAPQ cutoff of 50 can differ wildly across aligners and may drop all reads for some aligner implementations (per the README note from Tobias Tekath)
- In 3'-only sequencing assays (e.g. DropSeq), the BAF shift signal is adversely impacted because shifts are measured only for variants close to the 3' ends of genes
- In single-cell studies, cells with substantially large numbers of reads may dominate the pooled BAF shift signal and create biases
- RNA editing, PCR duplication artefacts, and sequencing errors may bias the BAF shift signal; CaSpER lacks a 3'-bias-aware BAF generation model

## Evidence

- [readme] BAFExtract generates the B-allele frequency shifts from mapped reads alone without the need for a heterozygous variant call set.: [evidence span withheld — anchor status: external_doc]
- [readme] README usage command: "samtools view <bam_file> | ./BAFExtract -generate_compressed_pileup_per_SAM stdin <genome_list> <sample_dir> 50 0"
- [readme] README MAPQ caveat: [evidence span withheld — anchor status: external_doc]
- [methods] Article methods BAF loading: "loh <- readBAFExtractOutput ( path="./meningioma_baf\", sequencing.type="bulk")"
- [abstract] Article abstract on integration: "CaSpER integrates the multiscale smoothing of expression signal and allelic shift signals for CNV calling."
