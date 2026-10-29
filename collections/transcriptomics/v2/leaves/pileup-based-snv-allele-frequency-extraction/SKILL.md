---
name: pileup-based-snv-allele-frequency-extraction
description: Use when you have mapped RNA-seq reads (BAM) and need a genome-wide BAF/allelic-shift
  profile at germline SNP sites to support CNV or LOH inference, without a matched
  DNA variant call set.
when_to_use_negative:
- Data are 3'-only sequencing (e.g. DropSeq) with sparse variants near gene 3' ends,
  degrading the BAF shift signal
- You need heterozygous genotype calls for downstream variant-level analyses rather
  than genome-wide BAF shift profiles for CNV correction
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: BAFExtract
  role: Generates the B-allele frequency shift profile from mapped reads alone via
    pileup generation and per-SNV calling steps
  repo: https://github.com/akdess/BAFExtract
- name: samtools
  role: Streams SAM records from the input BAM into BAFExtract's pileup generator
    via stdin
- name: CaSpER
  role: Consumes the extracted BAF output (readBAFExtractOutput) as the allelic-shift
    signal for multiscale CNV calling
  repo: https://github.com/akdess/CaSpER
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-13779-x
    title: CaSpER identifies and visualizes CNV events by integrative analysis of
      single-cell or bulk RNA-sequencing data
  source_content_hash: sha256:83489889524978e1dbee55de7641df5671a76e169f9d2111b23f9ca933b8ba6d
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
yes_rate: 0.6923076923076923
n_claims: 13
---

# pileup-based-snv-allele-frequency-extraction

## Summary

Generates a genome-wide B-allele frequency (BAF) shift signal directly from mapped RNA-seq reads using BAFExtract's two-step pileup pipeline, without requiring a pre-called heterozygous variant set. The resulting per-SNP BAF profile feeds CaSpER's allelic-shift (LOH) signal used to correct CNV calls.

## When to use

Use when you have mapped RNA-seq reads (BAM) and need a genome-wide BAF/allelic-shift profile at germline SNP sites to support CNV or LOH inference, without a matched DNA variant call set.

## When NOT to use

- Data are 3'-only sequencing (e.g. DropSeq) with sparse variants near gene 3' ends, degrading the BAF shift signal
- You need heterozygous genotype calls for downstream variant-level analyses rather than genome-wide BAF shift profiles for CNV correction

## Inputs

- Mapped RNA-seq reads in BAM format (single-cell BAMs may be merged first)
- Genome SNP/chromosome list file (e.g. hg38.list or hg19.list from fetchChromSizes)
- Genome FASTA pileup directory (preprocessed FASTA, downloadable for hg19/hg38)

## Outputs

- Per-SNP BAF shift signal file (e.g. test.baf) produced by BAFExtract
- Smoothed multiscale BAF shift signal in R (via readBAFExtractOutput + median filter)

## How to apply

Stream mapped reads from the RNA-seq BAM with 'samtools view' and pipe into BAFExtract's compressed pileup generator (-generate_compressed_pileup_per_SAM) over a genome SNP list (e.g. hg38.list), using a minimum mapping-quality threshold of 50; note that MAPQ semantics vary by aligner, so lower this if no reads survive. Then run the per-SNV calling step (-get_SNVs_per_pileup) against the preprocessed genome FASTA pileup directory (built via -preprocess_FASTA or downloaded for hg19/hg38) with parameters: minimum pileup depth 20, minimum supporting reads per allele 4, and BAF deviation threshold 0.1, writing the output BAF file. Load the resulting BAF file in R with readBAFExtractOutput (sequencing.type='bulk') and apply the recursive per-chromosome median filter (e.g. lohCallMedianFilterByChr) across multiscale windows (loh.scale=3) to obtain the smoothed BAF shift signal at multiple scales for downstream CaSpER CNV/LOH integration.

## Related tools

- **BAFExtract** (Generates the B-allele frequency shift profile from mapped reads alone via pileup generation and per-SNV calling steps) — https://github.com/akdess/BAFExtract
- **samtools** (Streams SAM records from the input BAM into BAFExtract's pileup generator via stdin)
- **CaSpER** (Consumes the extracted BAF output (readBAFExtractOutput) as the allelic-shift signal for multiscale CNV calling) — https://github.com/akdess/CaSpER

## Examples

```
mkdir test; samtools view SRR1295366.sorted.bam | ./bin/BAFExtract -generate_compressed_pileup_per_SAM stdin hg38.list test 50 0; ./bin/BAFExtract -get_SNVs_per_pileup hg38.list test ./hg38/ 20 4 0.1 test.baf
```

## Evaluation signals

- Output BAF file (e.g. test.baf) exists and is parseable by readBAFExtractOutput with sequencing.type='bulk'
- Per-SNP entries reflect the specified filters: minimum pileup depth 20, minimum supporting reads per allele 4, BAF deviation threshold 0.1
- Reads survived the MAPQ 50 threshold — verify non-empty pileup given aligner-specific MAPQ implementations
- Smoothed signal exists at all multiscale windows (loh.scale=3) after per-chromosome median filtering
- BAF entries cover SNPs genome-wide per the supplied genome list (hg38.list/hg19.list)

## Limitations

- MAPQ 50 filtering can leave no surviving reads depending on the aligner's MAPQ implementation (per README note from Tobias Tekath)
- For 3'-only sequencing (e.g. DropSeq), the BAF shift signal is adversely impacted since shifts are measured only at variants close to gene 3' ends
- No BAF shift generation model accounting for 3' bias; pooling is believed to mitigate this but was not validated
- RNA editing, PCR duplication artefacts, and sequencing errors may impact the BAF shift signal in complex ways and bias CNV detection

## Evidence

- [readme] BAFExtract generates the B-allele frequency shifts from mapped reads alone without the need for a heterozygous variant call set.: [evidence span withheld — anchor status: external_doc]
- [readme] samtools view <bam_file> | ./BAFExtract -generate_compressed_pileup_per_SAM stdin <genome_list> <sample_dir> 50 0; ./BAFExtract -get_SNVs_per_pileup ... 20 4 0.1 <output_baf_file>: "samtools view <bam_file> | ./BAFExtract -generate_compressed_pileup_per_SAM stdin <genome_list> <sample_dir> 50 0; ./BAFExtract -get_SNVs_per_pileup"
- [methods] Step 2. Extract BAF values from RNA-Seq bam files: "Step 2. Extract BAF values from RNA-Seq bam files"
- [abstract] CaSpER integrates the multiscale smoothing of expression signal and allelic shift signals for CNV calling.: "CaSpER integrates the multiscale smoothing of expression signal and allelic shift signals for CNV calling."
- [discussion] the BAF shift signal may be adversely impacted because BAF shift will be measured for only the variants that are located close to the 3′ ends of the genes.: "the BAF shift signal may be adversely impacted because BAF shift will be measured for only the variants that are located close to the 3′ ends of the genes."
