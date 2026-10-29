---
name: tissue-mapping-rate-statistical-comparison
description: Use when RNA-seq reads from multiple tissue groups have been aligned
  (e.g., STAR Log.final.out files) and you need to quantify whether mapping-quality
  metrics such as % uniquely mapped or % 'too short' reads differ systematically by
  tissue.
when_to_use_negative:
- The goal is allele-specific expression testing from VCF files — that is handled
  by the VADT pipeline, not alignment-metric comparison.
- Alignments were produced with a different mapper or settings (e.g., not STAR two-pass
  with chimeric alignment enabled), so 'too short' and chimeric flags are not comparable
  to the published thresholds.
- Samples do not group into tissue classes, leaving no basis for a between-tissue
  ANOVA or per-tissue summaries.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: STAR 2.5.2b
  role: Two-pass aligner against Gallus_gallus_5.0 (Ensembl release 86 GTF index)
    whose alignment logs supply the uniquely mapped, 'too short', and chimeric read
    percentages being compared across tissues
- name: FastQC
  role: Pre-alignment sequence quality review of all 68 fastq files before mapping
    metrics are computed
- name: Picard 2.8.1
  role: Marks duplicate reads on the aligned BAMs prior to gene-coverage QC
- name: RSeQC 2.6.4
  role: geneBody_coverage.py run on duplicate-marked BAMs to confirm no systematic
    gene-coverage bias by tissue that could confound the mapping comparison
- name: SciPy
  role: Provides the one-way ANOVA on per-tissue 'too short' percentages and the linear
    regression of percent chimeric versus percent unmappable reads for muscle samples
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:846812a4cf4010b530bd200309b4f7b79461062b4c3aac51eb713b099d25b335
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
  - SciPy
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# tissue-mapping-rate-statistical-comparison

## Summary

Extracts per-sample STAR alignment statistics (uniquely mapped, unmapped 'too short', and chimeric read percentages) from a two-pass RNA-seq pipeline and tests whether mapping quality differs across tissue groups (liver, abdominal fat, breast muscle) using a one-way ANOVA and a chimeric-vs-unmapped regression.

## When to use

Use when RNA-seq reads from multiple tissue groups have been aligned (e.g., STAR Log.final.out files) and you need to quantify whether mapping-quality metrics such as % uniquely mapped or % 'too short' reads differ systematically by tissue.

## When NOT to use

- The goal is allele-specific expression testing from VCF files — that is handled by the VADT pipeline, not alignment-metric comparison.
- Alignments were produced with a different mapper or settings (e.g., not STAR two-pass with chimeric alignment enabled), so 'too short' and chimeric flags are not comparable to the published thresholds.
- Samples do not group into tissue classes, leaving no basis for a between-tissue ANOVA or per-tissue summaries.

## Inputs

- 68 paired-end RNA-seq FASTQ files from broiler liver, abdominal fat, and breast muscle (SRA SRP058295 and SRP255211)
- Gallus_gallus_5.0 genome build with STAR index built from the Ensembl release 86 GTF
- Per-sample STAR alignment logs (Log.final.out) containing uniquely mapped, unmapped 'too short', and chimeric read percentages

## Outputs

- alignment_quality_metrics.csv containing per-sample and per-tissue mean STAR mapping metrics, the one-way ANOVA p-value for 'too short' percentages, and the muscle chimeric-vs-unmapped regression R-squared

## How to apply

Download the 68 broiler fastq samples (SRA SRP058295 for abdominal fat, SRP255211 for breast muscle/liver) and verify read quality with FastQC before proceeding. Align reads with STAR 2.5.2b in two-pass mode against the Gallus_gallus_5.0 genome build using an index built from the Ensembl release 86 GTF, enabling STAR's chimeric alignment setting; mark duplicates with Picard 2.8.1 and run RSeQC 2.6.4 geneBody_coverage.py on the BAMs to confirm no systematic gene-coverage bias by tissue. Parse each sample's STAR alignment log to extract the percentages of uniquely mapped reads, unmapped reads flagged 'too short', and chimeric reads, then group samples by tissue (breast muscle n=23, abdominal fat n=22, liver n=23). Compute per-tissue averages, run a one-way ANOVA (SciPy) on the 'too short' percentages across tissues, and fit a linear regression of percent chimeric reads versus percent unmappable reads for the muscle samples only. Write the summary metrics table, including the ANOVA p-value and the regression R-squared, to alignment_quality_metrics.csv. The rationale is that a large excess of 'too short' flags in one tissue signals a genome-build/annotation limitation rather than a sequencing failure, which must be quantified before downstream ASE analysis.

## Related tools

- **STAR 2.5.2b** (Two-pass aligner against Gallus_gallus_5.0 (Ensembl release 86 GTF index) whose alignment logs supply the uniquely mapped, 'too short', and chimeric read percentages being compared across tissues)
- **FastQC** (Pre-alignment sequence quality review of all 68 fastq files before mapping metrics are computed)
- **Picard 2.8.1** (Marks duplicate reads on the aligned BAMs prior to gene-coverage QC)
- **RSeQC 2.6.4** (geneBody_coverage.py run on duplicate-marked BAMs to confirm no systematic gene-coverage bias by tissue that could confound the mapping comparison)
- **SciPy** (Provides the one-way ANOVA on per-tissue 'too short' percentages and the linear regression of percent chimeric versus percent unmappable reads for muscle samples)

## Examples

```
from scipy import stats; f_stat, p_value = stats.f_oneway(liver_too_short, fat_too_short, muscle_too_short)  # expect p ≈ 9.76e-24
```

## Evaluation signals

- Per-tissue unique mapping rates reproduce the published values (~90.98% liver, ~90.78% abdominal fat, ~80.11% breast muscle).
- Average 'too short' read percentages reproduce ~5.33% (liver), ~4.50% (abdominal fat), and ~15.39% (breast muscle) with ANOVA p ≈ 9.76e-24.
- The muscle chimeric-versus-unmappable linear regression reproduces R² ≈ 0.5202.
- Sample grouping matches the published design: breast muscle n=23, abdominal fat n=22, liver n=23 (68 total).
- alignment_quality_metrics.csv includes both the ANOVA p-value and the regression R-squared as required outputs.

## Limitations

- Poor muscle mapping is attributed to the current genome build and annotation of muscle-related genes (Gallus_gallus_5.0 / Ensembl release 86); the article reports the analysis was also repeated with the Gallus_gallus_6.0 GTF, so results are build-dependent.
- The 'too short' metric conflates biological differences with annotation/assembly gaps and is descriptive rather than causal.
- Tissue groups are unbalanced (n=22 vs n=23), and the comparison depends on STAR-specific unmapped and chimeric categories that other aligners define differently.

## Evidence

- [methods] breast muscle samples had a significantly higher percentage of reads considered “too short” according to STAR (avg. 15.39% compared to avg. 4.50% for abdominal fat and avg 5.33% for liver: "breast muscle samples had a significantly higher percentage of reads considered “too short” according to STAR (avg. 15.39% compared to avg. 4.50% for abdominal fat and avg 5.33% for liver"
- [methods] aligning sequencing reads using STAR (version 2.5.2b) with the two-pass setting29,30 to the Gallus_gallus_5.0 genome build: "aligning sequencing reads using STAR (version 2.5.2b) with the two-pass setting29,30 to the Gallus_gallus_5.0 genome build"
- [methods] SRA accession numbers: abdominal fat samples SRP058295 and breast muscle/liver samples SRP255211: "SRA accession numbers: abdominal fat samples SRP058295 and breast muscle/liver samples SRP255211"
- [discussion] The biggest problem we encountered with our entire analysis pipeline was mapping issues with muscle tissue samples: "The biggest problem we encountered with our entire analysis pipeline was mapping issues with muscle tissue samples"
