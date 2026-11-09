---
name: duplicate-read-marking
description: Use when SAM/BAM files from RNA-seq alignment (e.g. STAR two-pass output)
  feed into GATK HaplotypeCaller variant calling and duplicates must be flagged before
  Base Recalibration and calling.
when_to_use_negative:
- Input is raw FASTQ data that has not yet been aligned with STAR — duplicate marking
  requires an aligned SAM/BAM with mapping coordinates
- Downstream analysis does not use GATK variant calling or read-depth-sensitive statistics
  and duplicates can be ignored
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3170
tools:
- name: Picard 2.8.1
  role: Marks duplicate reads in STAR-aligned SAM/BAM files and converts SAM to BAM
    before GATK processing
- name: STAR 2.5.2b
  role: Produces the aligned SAM output (two-pass mode, soft-clipping permitted) that
    is the input to duplicate marking
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:6d1bceddc6b00544475a0fd0e0896a314ffd7f6709a547477e169dca80fc65da
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FastQC
  - Picard
  - Picard 2.8.1
  - STAR 2.5.2b
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# duplicate-read-marking

## Summary

Marks PCR/optical duplicate reads in aligned RNA-seq SAM/BAM files with Picard so downstream GATK variant calling is not biased by amplified fragments. In this broiler ASE study, duplicate marking with Picard 2.8.1 was a required intermediate step between STAR two-pass alignment and the GATK 3.7 pipeline.

## When to use

Use when SAM/BAM files from RNA-seq alignment (e.g. STAR two-pass output) feed into GATK HaplotypeCaller variant calling and duplicates must be flagged before Base Recalibration and calling.

## When NOT to use

- Input is raw FASTQ data that has not yet been aligned with STAR — duplicate marking requires an aligned SAM/BAM with mapping coordinates
- Downstream analysis does not use GATK variant calling or read-depth-sensitive statistics and duplicates can be ignored

## Inputs

- Aligned SAM/BAM files from STAR 2.5.2b two-pass alignment of paired-end broiler RNA-seq reads (SRA SRP058295, SRP255211) to the Gallus_gallus reference genome

## Outputs

- Duplicate-marked BAM files (SAM converted to BAM) ready for the GATK 3.7 'Split'N'Trim' and Base Recalibration steps

## How to apply

After aligning reads with STAR 2.5.2b in two-pass mode (allowing soft-clipping rather than trimming), convert the SAM output to BAM and run Picard 2.8.1 duplicate marking (MarkDuplicates) on each sample's BAM. Marked duplicate reads are flagged (not removed) so downstream tools in the GATK 3.7 pipeline — 'Split'N'Trim', Base Recalibration, and HaplotypeCaller in g.vcf mode — can account for or ignore them when calling variants from the 68 broiler samples. Duplicate marking must occur before GATK processing because recalibration and calling assume duplicates are labeled; skipping it inflates read depth and biases allele ratios, which would corrupt the downstream ASE binomial tests in VADT. Validate by confirming the BAM is readable, has duplicate flag tags applied, and that subsequent GATK steps complete without duplicate-related warnings.

## Related tools

- **Picard 2.8.1** (Marks duplicate reads in STAR-aligned SAM/BAM files and converts SAM to BAM before GATK processing)
- **STAR 2.5.2b** (Produces the aligned SAM output (two-pass mode, soft-clipping permitted) that is the input to duplicate marking)

## Examples

```
java -jar picard-2.8.1.jar MarkDuplicates I=aligned.sam O=marked.bam M=metrics.txt REMOVE_DUPLICATES=false
```

## Evaluation signals

- Output BAM is a valid, parseable file and SAM-to-BAM conversion succeeded
- Duplicate read flags are present in the BAM and marked reads are not removed, preserving total read counts for GATK depth filters (Depth < 100 filter applied later)
- Duplicate marking runs on all 68 sample BAMs before Base Recalibration and HaplotypeCaller complete without errors
- Downstream STAR alignment statistics remain consistent with the paper's reported average unique mapping rate of 87.24%

## Limitations

- Duplicate marking alone does not remove duplicates — reads are flagged; removal depends on downstream tool behavior in the GATK pipeline
- Breast muscle samples had a significantly higher percentage of reads flagged as 'too short' by STAR (avg. 15.39% vs 4.50% for abdominal fat and 5.33% for liver), which can reduce usable aligned reads entering duplicate marking and variant calling

## Evidence

- [methods] Picard marking step after STAR alignment: [evidence span withheld — anchor status: not_found]
- [methods] Pipeline ordering: STAR then Picard then GATK: [evidence span withheld — anchor status: not_found]
- [methods] GATK steps following duplicate marking: "which includes “Split’N’Trim,” “Base Recalibration” and variant calling using HaplotypeCaller (with the g.vcf setting)"
- [methods] Muscle tissue mapping issues: "breast muscle samples had a significantly higher percentage of reads considered “too short” according to STAR (avg. 15.39% compared to avg. 4.50% for abdominal fat and avg 5.33% for liver"
