---
name: star-log-metric-extraction
description: Use when you have STAR alignment log files (Log.final.out) from multiple
  RNA-seq samples and need per-sample or per-group comparison of uniquely mapped,
  'too short' unmapped, and chimeric read percentages to diagnose tissue- or condition-specific
  mapping problems.
when_to_use_negative:
- Input is VCF/genotype data for ASE analysis — use VADT (binomial test with Benjamini–Hochberg
  FDR) rather than STAR log parsing
- Reads were aligned with a non-STAR aligner whose log schema does not contain the
  'too short' unmapped category, so the extracted metrics are not comparable
- Goal is gene-body coverage bias assessment from BAM files — that requires RSeQC
  geneBody_coverage.py on the alignments, not STAR log metrics alone
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STAR 2.5.2b
  role: Two-pass aligner producing the per-sample Log.final.out files from which uniquely
    mapped, 'too short' unmapped, and chimeric read percentages are extracted
- name: FastQC
  role: Pre-alignment FASTQ quality review ensuring all 68 samples passed quality
    control before STAR alignment and log extraction
- name: RSeQC 2.6.4
  role: geneBody_coverage.py run on Picard-deduplicated BAMs to confirm the tissue
    mapping differences are not a systematic gene-coverage bias
- name: SciPy
  role: One-way ANOVA on 'too short' percentages and linear regression of chimeric
    versus unmapped read percentages
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:46fb7d028bad9a21c8e3fdabc2a59811cfe6ad68646c9706f6721ff611022a83
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FastQC
  - Picard
  - RSeQC
  - RSeQC 2.6.4
  - STAR
  - STAR 2.5.2b
  - SciPy
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# star-log-metric-extraction

## Summary

Extracts per-sample alignment quality metrics (uniquely mapped %, unmapped 'too short' %, chimeric %) from STAR two-pass Log.final.out files and compares them across tissue groups, as done for the 68-sample broiler RNA-seq dataset to expose muscle-specific mapping failures.

## When to use

Use when you have STAR alignment log files (Log.final.out) from multiple RNA-seq samples and need per-sample or per-group comparison of uniquely mapped, 'too short' unmapped, and chimeric read percentages to diagnose tissue- or condition-specific mapping problems.

## When NOT to use

- Input is VCF/genotype data for ASE analysis — use VADT (binomial test with Benjamini–Hochberg FDR) rather than STAR log parsing
- Reads were aligned with a non-STAR aligner whose log schema does not contain the 'too short' unmapped category, so the extracted metrics are not comparable
- Goal is gene-body coverage bias assessment from BAM files — that requires RSeQC geneBody_coverage.py on the alignments, not STAR log metrics alone

## Inputs

- STAR Log.final.out (or Log.out) alignment log files, one per sample, from two-pass STAR runs
- FASTQ files for the 68 RNA-seq samples (SRA SRP058295 abdominal fat; SRP255211 breast muscle/liver)
- STAR genome index built from Gallus_gallus_5.0 with the Ensembl release 86 GTF
- Tissue/sample-group mapping (breast muscle n=23, abdominal fat n=22, liver n=23)

## Outputs

- alignment_quality_metrics.csv containing per-tissue average uniquely mapped %, average 'too short' unmapped %, chimeric read %, the one-way ANOVA p-value on 'too short' percentages, and the muscle chimeric-vs-unmapped regression R-squared
- Per-sample table of uniquely mapped, unmapped ('too short'), and chimeric read percentages grouped by tissue

## How to apply

Align each FASTQ (here 68 samples: SRP058295 abdominal fat, SRP255211 breast muscle/liver) with STAR 2.5.2b in two-pass mode against the Gallus_gallus_5.0 build using an index built from the Ensembl release 86 GTF, enabling STAR's chimeric alignment setting so chimeric percentages appear in the logs. Parse each sample's STAR alignment log to extract the percent of uniquely mapped reads, the percent of unmapped reads flagged 'too short', and the percent of chimeric reads, then group samples by tissue (breast muscle n=23, abdominal fat n=22, liver n=23). Compute per-tissue averages for each metric, then run a one-way ANOVA (SciPy) on the 'too short' percentages across tissues and fit a linear regression of percent chimeric versus percent unmappable reads for the muscle samples only. Write the per-sample and per-tissue summary, including the ANOVA p-value and regression R-squared, to alignment_quality_metrics.csv. Optionally corroborate with RSeQC 2.6.4 geneBody_coverage.py on the Picard 2.8.1-deduplicated BAMs to confirm the mapping deficit is not a gene-body coverage artifact.

## Related tools

- **STAR 2.5.2b** (Two-pass aligner producing the per-sample Log.final.out files from which uniquely mapped, 'too short' unmapped, and chimeric read percentages are extracted)
- **FastQC** (Pre-alignment FASTQ quality review ensuring all 68 samples passed quality control before STAR alignment and log extraction)
- **RSeQC 2.6.4** (geneBody_coverage.py run on Picard-deduplicated BAMs to confirm the tissue mapping differences are not a systematic gene-coverage bias)
- **SciPy** (One-way ANOVA on 'too short' percentages and linear regression of chimeric versus unmapped read percentages)

## Examples

```
from scipy import stats; f, p = stats.f_oneway(muscle_too_short, fat_too_short, liver_too_short)
```

## Evaluation signals

- Per-tissue unique mapping rates reproduce Table 1 values: ~90.98% liver, ~90.78% abdominal fat, ~80.11% breast muscle
- Average 'too short' percentages reproduce ~5.33% (liver), ~4.50% (abdominal fat), and ~15.39% (breast muscle), with one-way ANOVA p ≈ 9.76e-24
- Linear regression of percent chimeric versus percent unmappable reads for muscle samples reproduces R² ≈ 0.5202
- RSeQC geneBody_coverage.py output shows no systematic gene-coverage bias by tissue, so the metric differences reflect alignment, not coverage bias
- Chimeric percentages are present for every sample, confirming STAR's chimeric alignment setting was enabled

## Limitations

- Muscle mapping problems are attributed to the current genome build (Gallus_gallus_5.0) and annotation of muscle-related genes, so absolute values are build- and annotation-dependent
- Metrics are specific to STAR 2.5.2b with the Ensembl release 86 GTF index; different STAR versions or GTF releases will change the extracted percentages
- Chimeric read percentages are only available if STAR's chimeric alignment setting is explicitly enabled at alignment time

## Evidence

- [methods] muscle too-short elevation: "breast muscle samples had a significantly higher percentage of reads considered “too short” according to STAR (avg. 15.39% compared to avg. 4.50% for abdominal fat and avg 5.33% for liver"
- [methods] two-pass STAR alignment: "aligning sequencing reads using STAR (version 2.5.2b) with the two-pass setting29,30"
- [methods] sample accessions: "SRA accession numbers: abdominal fat samples SRP058295 and breast muscle/liver samples SRP255211"
- [discussion] muscle mapping caveat: "The biggest problem we encountered with our entire analysis pipeline was mapping issues with muscle tissue samples"
- [methods] fastq QC gate: "All fastq files were reviewed for sequence quality using FastQC24 and all samples passed this quality control step"
