---
name: chimeric-read-correlation-analysis
description: Use when STAR alignment logs from RNA-seq samples grouped by tissue show
  anomalous unmapped ('too short') rates and you need to test whether chimeric reads
  co-occur with unmappable reads, e.g. to diagnose genome-build or annotation-driven
  mapping failures in one tissue.
when_to_use_negative:
- The analysis goal is allele-specific expression detection from VCF files — that
  is handled by the VADT binomial-test pipeline, not STAR log statistics.
- Reads are aligned to a different genome build or annotation than Gallus_gallus_5.0
  / Ensembl release 86, so the expected per-tissue validation values would not apply.
- You need per-sample gene-body coverage assessment (RSeQC geneBody_coverage.py) rather
  than whole-sample mapping-rate summaries.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: STAR 2.5.2b
  role: Performs two-pass alignment to Gallus_gallus_5.0 with the chimeric alignment
    setting enabled and emits the per-sample alignment statistics parsed for this
    analysis
- name: FastQC
  role: Confirms FASTQ read quality before alignment so that STAR statistics are not
    confounded by poor raw data
- name: SciPy
  role: Provides the one-way ANOVA on 'too short' percentages and the linear regression
    (R-squared) for muscle chimeric versus unmapped reads
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:a7dd9f0c7063202398bb405ed87e60da73e3f925b6a14a4653c08e3adf80dc46
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FastQC
  - Picard
  - RSeQC
  - STAR 2.5.2b
  - SciPy
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# chimeric-read-correlation-analysis

## Summary

Extracts per-sample STAR alignment statistics (uniquely mapped, unmapped 'too short', and chimeric read percentages) from two-pass STAR RNA-seq runs, compares them across broiler tissue groups via one-way ANOVA, and fits a linear regression of percent chimeric versus percent unmappable reads in breast muscle to diagnose mapping problems. Quantifies tissue-specific mapping bias in the Gallus_gallus_5.0-based pipeline.

## When to use

Use when STAR alignment logs from RNA-seq samples grouped by tissue show anomalous unmapped ('too short') rates and you need to test whether chimeric reads co-occur with unmappable reads, e.g. to diagnose genome-build or annotation-driven mapping failures in one tissue.

## When NOT to use

- The analysis goal is allele-specific expression detection from VCF files — that is handled by the VADT binomial-test pipeline, not STAR log statistics.
- Reads are aligned to a different genome build or annotation than Gallus_gallus_5.0 / Ensembl release 86, so the expected per-tissue validation values would not apply.
- You need per-sample gene-body coverage assessment (RSeQC geneBody_coverage.py) rather than whole-sample mapping-rate summaries.

## Inputs

- 68 FASTQ RNA-seq samples from SRA (SRP058295 abdominal fat; SRP255211 breast muscle/liver)
- STAR 2.5.2b alignment logs (Log.final.out-style) from two-pass runs against Gallus_gallus_5.0 with the Ensembl release 86 GTF index and chimeric alignment enabled
- Tissue group labels per sample (breast muscle n=23, abdominal fat n=22, liver n=23)

## Outputs

- Per-tissue averages of uniquely mapped, unmapped ('too short'), and chimeric read percentages
- One-way ANOVA p-value on 'too short' percentages across tissues (expected ≈ 9.76e-24)
- Linear regression R-squared of percent chimeric versus percent unmappable reads for muscle samples (expected ≈ 0.5202)
- alignment_quality_metrics.csv summary metrics table

## How to apply

Align all FASTQ samples with STAR 2.5.2b in two-pass mode against the Gallus_gallus_5.0 genome build using an index built from the Ensembl release 86 GTF, with STAR's chimeric alignment setting enabled. From each sample's STAR alignment log, extract the percentages of uniquely mapped, unmapped ('too short'), and chimeric reads, and group samples by tissue (breast muscle n=23, abdominal fat n=22, liver n=23). Compute per-tissue averages and run a one-way ANOVA (SciPy) on the 'too short' percentages across the three tissues. For the muscle group only, fit a linear regression of percent chimeric reads against percent unmappable reads to test whether chimeric signal tracks reads STAR cannot map. Write the per-tissue summary metrics table, the ANOVA p-value, and the regression R-squared to alignment_quality_metrics.csv. The rationale is that a tissue with poor mapping and elevated too-short reads (muscle, avg. 15.39%) may signal a genome-build/annotation deficiency that also manifests as chimeric reads.

## Related tools

- **STAR 2.5.2b** (Performs two-pass alignment to Gallus_gallus_5.0 with the chimeric alignment setting enabled and emits the per-sample alignment statistics parsed for this analysis)
- **FastQC** (Confirms FASTQ read quality before alignment so that STAR statistics are not confounded by poor raw data)
- **SciPy** (Provides the one-way ANOVA on 'too short' percentages and the linear regression (R-squared) for muscle chimeric versus unmapped reads)

## Examples

```
from scipy import stats; slope, intercept, r_value, p_value, stderr = stats.linregress(pct_chimeric_muscle, pct_too_short_muscle)
```

## Evaluation signals

- Per-tissue unique mapping rates reproduce Table 1 values: ~90.98% liver, ~90.78% abdominal fat, ~80.11% breast muscle
- Average 'too short' reads reproduce ~5.33% (liver), ~4.50% (abdominal fat), and ~15.39% (breast muscle)
- One-way ANOVA on 'too short' percentages yields p ≈ 9.76e-24
- Muscle chimeric-vs-unmapped linear regression reproduces R2 ≈ 0.5202
- Group sizes match 68 samples total: breast muscle n=23, abdominal fat n=22, liver n=23

## Limitations

- Mapping problems in breast muscle are likely attributable to the current genome build and annotation of muscle-related genes, so results are build-dependent; the paper also repeated the prior analysis with the Gallus_gallus_6.0 GTF.
- Chimeric alignment statistics are only interpretable when STAR's chimeric alignment setting is explicitly enabled during alignment.
- The correlation (R2 ≈ 0.5202) is descriptive for the muscle group and does not establish the biological mechanism behind poor mapping.

## Evidence

- [methods] Breast muscle poor alignment with elevated too-short reads: "breast muscle samples had a significantly higher percentage of reads considered “too short” according to STAR (avg. 15.39% compared to avg. 4.50% for abdominal fat and avg 5.33% for liver"
- [methods] Two-pass STAR alignment setup producing the logs analyzed: "aligning sequencing reads using STAR (version 2.5.2b) with the two-pass setting29,30"
- [discussion] Likely cause of the muscle mapping problem: "The biggest problem we encountered with our entire analysis pipeline was mapping issues with muscle tissue samples"
