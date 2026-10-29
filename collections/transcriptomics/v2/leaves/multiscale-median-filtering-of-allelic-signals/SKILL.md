---
name: multiscale-median-filtering-of-allelic-signals
description: Use when you have mapped RNA-seq BAM files and need an allelic shift
  (BAF/LOH) signal profile to correct or integrate with expression-based CNV calls,
  without a pre-existing heterozygous variant call set.
when_to_use_negative:
- '3′-only sequencing data (e.g. DropSeq-style 3′ assays): the BAF shift signal may
  be adversely impacted because BAF shift will be measured for only variants close
  to the 3′ ends of genes'
- You already have a high-quality heterozygous variant call set or SNV VCFs and need
  direct genotyping rather than BAF-shift estimation from reads alone
- Your aligner's MAPQ scale makes a minimum mapping quality of 50 filter out all reads
  — recalibrate the MAPQ cutoff before pileup generation
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0282
tools:
- name: BAFExtract
  role: Generates the B-allele frequency shifts from mapped reads alone (compressed
    pileup generation plus per-SNV BAF calling with depth 20, support 4, deviation
    0.1 filters)
  repo: https://github.com/akdess/BAFExtract
- name: samtools
  role: Streams mapped reads from the RNA-seq BAM into BAFExtract's pileup generator
- name: CaSpER
  role: Consumes the BAFExtract output via readBAFExtractOutput and applies recursive
    per-chromosome median filtering across scales (loh.scale=3) for multiscale CNV
    calling
  repo: https://github.com/akdess/CaSpER
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-13779-x
    title: CaSpER identifies and visualizes CNV events by integrative analysis of
      single-cell or bulk RNA-sequencing data
  source_content_hash: sha256:eea15d8dd9b3cd0fa7b0f35a1cb1948f3c283d42f481c3b7b5742a49d1737e38
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/akdess/BAFExtract
  tools:
  - BAFExtract
  - CaSpER
  - bamtools
  - samtools
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0282
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# multiscale-median-filtering-of-allelic-signals

## Summary

Generate a genome-wide B-allele frequency (BAF) shift signal directly from mapped RNA-seq reads with BAFExtract, then smooth it with recursive per-chromosome median filtering across multiple scales (loh.scale=3) so allelic shift (LOH) information can be integrated with expression-based CNV calling in CaSpER.

## When to use

Use when you have mapped RNA-seq BAM files and need an allelic shift (BAF/LOH) signal profile to correct or integrate with expression-based CNV calls, without a pre-existing heterozygous variant call set.

## When NOT to use

- 3′-only sequencing data (e.g. DropSeq-style 3′ assays): the BAF shift signal may be adversely impacted because BAF shift will be measured for only variants close to the 3′ ends of genes
- You already have a high-quality heterozygous variant call set or SNV VCFs and need direct genotyping rather than BAF-shift estimation from reads alone
- Your aligner's MAPQ scale makes a minimum mapping quality of 50 filter out all reads — recalibrate the MAPQ cutoff before pileup generation

## Inputs

- Mapped RNA-seq BAM file (bulk or merged single-cell)
- Genome SNP/chromosome size list (e.g. hg38.list generated with fetchChromSizes)
- Genome FASTA pileup directory produced by BAFExtract -preprocess_FASTA (prebuilt hg19/hg38 zips available)
- sequencing.type ('bulk' or single-cell) for readBAFExtractOutput
- loh.scale parameter (e.g. 3) for the multiscale median filter

## Outputs

- BAF shift signal file (e.g. test.baf) with per-SNP BAF entries after depth/support/deviation filtering
- Smoothed BAF shift signal at multiple scales (loh R object parseable by CaSpER for HMM integration)

## How to apply

First generate the compressed pileup from the RNA-seq BAM by piping 'samtools view' output into BAFExtract (-generate_compressed_pileup_per_SAM) over the genome SNP list using a minimum mapping quality threshold of 50 (adjust if your aligner's MAPQ scale makes 50 overly restrictive). Then run BAFExtract -get_SNVs_per_pileup against the preprocessed genome FASTA pileup directory with filter parameters minimum pileup depth 20, minimum supporting reads per allele 4, and BAF deviation threshold 0.1, writing the BAF shift signal to an output file (e.g. test.baf). Load the output in R with readBAFExtractOutput(sequencing.type='bulk') and apply the recursive per-chromosome median filter across multiscale windows (loh.scale=3, e.g. via lohCallMedianFilterByChr) to obtain the smoothed BAF shift signal at multiple resolutions. The multiscale smoothing is required because CaSpER performs HMM on all scales of the expression signal and integrates HMM segment states with allele frequency shift information across N×M pairwise scale comparisons. For single-cell studies, merge the single-cell BAM files first, noting that cells with substantially large numbers of reads may dominate the pooled BAF shift signal.

## Related tools

- **BAFExtract** (Generates the B-allele frequency shifts from mapped reads alone (compressed pileup generation plus per-SNV BAF calling with depth 20, support 4, deviation 0.1 filters)) — https://github.com/akdess/BAFExtract
- **samtools** (Streams mapped reads from the RNA-seq BAM into BAFExtract's pileup generator)
- **CaSpER** (Consumes the BAFExtract output via readBAFExtractOutput and applies recursive per-chromosome median filtering across scales (loh.scale=3) for multiscale CNV calling) — https://github.com/akdess/CaSpER

## Examples

```
mkdir test; samtools view SRR1295366.sorted.bam | ./bin/BAFExtract -generate_compressed_pileup_per_SAM stdin hg38.list test 50 0; ./bin/BAFExtract -get_SNVs_per_pileup hg38.list test ./hg38/ 20 4 0.1 test.snp
```

## Evaluation signals

- Output BAF file (e.g. test.baf) exists and is parseable by readBAFExtractOutput with sequencing.type='bulk'
- BAF entries reflect the specified filters: minimum pileup depth 20, minimum supporting reads per allele 4, BAF deviation threshold 0.1
- Smoothed signal exists at each of the multiscale windows (loh.scale=3) and is computed per chromosome via the recursive median filter
- Mapping-quality cutoff of 50 does not silently remove all reads — verify non-empty pileup given your aligner's MAPQ conventions
- Smoothed BAF shift signal is usable in downstream pairwise comparison of scales with expression HMM segment states

## Limitations

- 3′-only sequencing (such as DropSeq) degrades the BAF shift signal because BAF shift is measured only for variants near the 3′ ends of genes; CaSpER lacks a BAF shift generation model accounting for 3′ bias
- In single-cell data, cells with substantially large numbers of reads may dominate the pooled BAF shift signal and create biases
- RNA editing, PCR duplication artefacts, and sequencing errors may impact the BAF shift signal and bias CNV detection in complex ways
- The MAPQ=50 default can mean no reads survive filtering depending on the aligner's MAPQ implementation

## Evidence

- [readme] BAFExtract generates the B-allele frequency shifts from mapped reads alone without the need for a heterozygous variant call set.: [evidence span withheld — anchor status: external_doc]
- [readme] samtools view | BAFExtract pileup and SNV calling command with MAPQ 50 and filters 20 4 0.1: "samtools view <bam_file> | ./BAFExtract -generate_compressed_pileup_per_SAM stdin <genome_list> <sample_dir> 50 0; ./BAFExtract -get_SNVs_per_pileup  <genome_list> <sample_dir>"
- [abstract] CaSpER integrates the multiscale smoothing of expression signal and allelic shift signals for CNV calling.: "CaSpER integrates the multiscale smoothing of expression signal and allelic shift signals for CNV calling."
- [methods] we perform HMM on all scales of expression signal and integrate HMM segment states with Allele Frequency Shift Information: "we perform HMM on all scales of expression signal and integrate HMM segment states with Allele Frequency Shift Information"
- [discussion] 3′-only sequencing degrades BAF shift signal near gene 3′ ends: "the BAF shift signal may be adversely impacted because BAF shift will be measured for only the variants that are located close to the 3′ ends of the genes."
