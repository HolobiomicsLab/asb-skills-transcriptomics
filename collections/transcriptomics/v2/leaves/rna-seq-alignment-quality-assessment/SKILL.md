---
name: rna-seq-alignment-quality-assessment
description: Use when RNA-seq FASTQ files from multiple tissue groups must be aligned
  and you need per-sample STAR alignment statistics (unique mapping %, unmapped 'too
  short' %, chimeric %) to compare alignment quality across groups before ASE or expression
  analysis.
when_to_use_negative:
- The input is already-aligned BAM data and the goal is variant calling (GATK HaplotypeCaller)
  or ASE detection with VADT rather than alignment quality comparison
- Only a single tissue group is present, so cross-tissue ANOVA of alignment metrics
  has no comparison basis
- The genome/annotation pair differs (e.g., Gallus_gallus_6.0 GTF), since the expected
  metric values here are specific to Gallus_gallus_5.0 with Ensembl release 86 annotation
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: FastQC
  role: Initial per-FASTQ sequence quality review before alignment
- name: STAR 2.5.2b
  role: Two-pass alignment to Gallus_gallus_5.0 with chimeric alignment enabled; source
    of uniquely mapped, 'too short', and chimeric read percentages
- name: Picard 2.8.1
  role: Marks duplicate reads on STAR output BAMs prior to coverage QC
- name: RSeQC 2.6.4
  role: geneBody_coverage.py on Picard-processed BAMs to confirm no systematic gene-coverage
    bias by tissue
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:ce918533e0c1ff8362c04d20192cf82146bb485d8606c87cfe0686d879fd6f14
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FastQC
  - Picard
  - Picard 2.8.1
  - RSeQC
  - RSeQC 2.6.4
  - STAR 2.5.2b
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# rna-seq-alignment-quality-assessment

## Summary

Assess RNA-seq read alignment quality per sample using FastQC, two-pass STAR 2.5.2b alignment to Gallus_gallus_5.0, Picard duplicate marking, and RSeQC geneBody_coverage, then aggregate STAR log metrics (uniquely mapped, 'too short', chimeric reads) by tissue and test differences with one-way ANOVA. It exists to detect systematic mapping failures — here, significantly poorer alignment in breast muscle vs. abdominal fat and liver broiler tissues — before downstream variant/ASE analysis.

## When to use

Use when RNA-seq FASTQ files from multiple tissue groups must be aligned and you need per-sample STAR alignment statistics (unique mapping %, unmapped 'too short' %, chimeric %) to compare alignment quality across groups before ASE or expression analysis.

## When NOT to use

- The input is already-aligned BAM data and the goal is variant calling (GATK HaplotypeCaller) or ASE detection with VADT rather than alignment quality comparison
- Only a single tissue group is present, so cross-tissue ANOVA of alignment metrics has no comparison basis
- The genome/annotation pair differs (e.g., Gallus_gallus_6.0 GTF), since the expected metric values here are specific to Gallus_gallus_5.0 with Ensembl release 86 annotation

## Inputs

- 68 RNA-seq FASTQ samples from SRA (SRP058295 for abdominal fat, SRP255211 for breast muscle/liver)
- Gallus_gallus_5.0 genome build with STAR index built from Ensembl GTF release 86
- Per-sample STAR alignment logs (Log.final.out)

## Outputs

- Per-sample alignment metrics: uniquely mapped %, unmapped 'too short' %, chimeric read %
- Per-tissue averages of unique mapping rates and 'too short' reads
- alignment_quality_metrics.csv including ANOVA p-value and muscle chimeric-vs-unmapped regression R-squared
- geneBody_coverage plots from RSeQC confirming no systematic coverage bias by tissue

## How to apply

First review all FASTQ files with FastQC to confirm basic sequence quality. Align reads with STAR 2.5.2b in two-pass mode against the Gallus_gallus_5.0 genome build, using an index built from the Ensembl release 86 GTF, and enable STAR's chimeric alignment setting; then mark duplicates with Picard 2.8.1 and run RSeQC 2.6.4 geneBody_coverage.py on the resulting BAMs to rule out systematic gene-coverage bias by tissue. Extract per-sample percentages of uniquely mapped, unmapped ('too short'), and chimeric reads from the STAR alignment logs and group by tissue (breast muscle n=23, abdominal fat n=22, liver n=23). Compute per-tissue averages, run a one-way ANOVA (SciPy) on the 'too short' percentages across tissues, and fit a linear regression of percent chimeric reads versus percent unmappable reads for the muscle samples. Flag any tissue whose mapping metrics deviate substantially from the others — the rationale is that such deviations (e.g., likely due to genome build and annotation of muscle-related genes) propagate errors into downstream ASE detection.

## Related tools

- **FastQC** (Initial per-FASTQ sequence quality review before alignment)
- **STAR 2.5.2b** (Two-pass alignment to Gallus_gallus_5.0 with chimeric alignment enabled; source of uniquely mapped, 'too short', and chimeric read percentages)
- **Picard 2.8.1** (Marks duplicate reads on STAR output BAMs prior to coverage QC)
- **RSeQC 2.6.4** (geneBody_coverage.py on Picard-processed BAMs to confirm no systematic gene-coverage bias by tissue)

## Examples

```
STAR --runMode alignReads --twopassMode Basic --genomeDir Gallus_gallus_5.0_index --readFilesIn R1.fastq.gz R2.fastq.gz --readFilesCommand zcat --outSAMtype BAM SortedByCoordinate
```

## Evaluation signals

- Per-tissue unique mapping rates reproduce Table 1 values: ~90.98% liver, ~90.78% abdominal fat, ~80.11% breast muscle
- Per-tissue average 'too short' read percentages reproduce ~5.33% (liver), ~4.50% (abdominal fat), ~15.39% (breast muscle)
- One-way ANOVA on 'too short' percentages across tissues yields p ≈ 9.76e-24
- Linear regression of percent chimeric vs. percent unmappable reads for muscle samples reproduces R2 ≈ 0.5202
- geneBody_coverage.py output shows no systematic gene-coverage bias by tissue

## Limitations

- Breast muscle samples had significantly higher 'too short' percentages (avg. 15.39%), and the authors attribute mapping issues to the current genome build and annotation of muscle-related genes, so absolute metrics are build-dependent
- Repeating the analysis with the Gallus_gallus_6.0 GTF was needed to test robustness of prior results, indicating annotation choice materially affects these metrics
- Alignment quality assessment alone does not resolve the biological mechanism behind poor muscle mapping; it flags the problem for downstream interpretation

## Evidence

- [methods] Two-pass STAR alignment to Gallus_gallus_5.0: "aligning sequencing reads using STAR (version 2.5.2b) with the two-pass setting29,30"
- [methods] Breast muscle alignment failure signal: "breast muscle samples had a significantly higher percentage of reads considered “too short” according to STAR (avg. 15.39% compared to avg. 4.50%"
- [methods] FastQC pre-alignment QC: "All fastq files were reviewed for sequence quality using FastQC24 and all samples passed this quality control step"
- [methods] RSeQC coverage check: "gene coverage was assessed using RSeQC’s (version 2.6.4) geneBody_coverage.py37"
- [results] Expected validation metrics for the summary table: [evidence span withheld — anchor status: not_found]
