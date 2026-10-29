---
name: correlation-analysis-of-baf-and-expression-signals
description: Use when you have CaSpER outputs from bulk RNA-seq (BAF shift signal
  from BAFExtract plus multiscale smoothed expression per chromosome arm) and need
  to quantify whether LOH/bajo-allelic-imbalance arms co-vary with expression loss,
  e.g. chr1p/chr6q/chr22q in meningioma.
when_to_use_negative:
- 3′-only sequencing data (e.g. DropSeq-style) where the BAF shift signal is degraded
  because variants are measured only near gene 3′ ends, making BAF-expression correlations
  unreliable
- You need focal or gene-level CNV calls (extractSegmentSummary/gene.matrix) rather
  than arm-level association between allelic imbalance and expression
- Single-cell samples where a few high-read cells may dominate the pooled BAF shift
  signal, biasing the correlation
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: CaSpER
  role: Produces the multiscale smoothed expression signal and BAF-integrated CNV
    calls (CreateCasperObject, runCaSpER) that are correlated at chromosome-arm level
  repo: github.com/akdess/CaSpER
- name: BAFExtract
  role: Generates the B-allele frequency shift signal from mapped RNA-seq reads without
    a heterozygous variant call set; its output is the BAF side of the correlation
  repo: github.com/akdess/BAFExtract
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-13779-x
    title: CaSpER identifies and visualizes CNV events by integrative analysis of
      single-cell or bulk RNA-sequencing data
  source_content_hash: sha256:91152480650b7cde0477e9a0a43014d01276aeb105c70832fbceaed37f60482b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BAFExtract
  - CaSpER
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# correlation-analysis-of-baf-and-expression-signals

## Summary

Quantifies the relationship between B-allele frequency (BAF) shift signal (allelic imbalance/LOH) and multiscale-smoothed gene expression signal per chromosome arm, as demonstrated with CaSpER on the yale_meningioma bulk RNA-seq dataset (chr1p, chr6q, chr22q Pearson correlations). It tests whether arms with allelic imbalance show concordant expression changes.

## When to use

Use when you have CaSpER outputs from bulk RNA-seq (BAF shift signal from BAFExtract plus multiscale smoothed expression per chromosome arm) and need to quantify whether LOH/bajo-allelic-imbalance arms co-vary with expression loss, e.g. chr1p/chr6q/chr22q in meningioma.

## When NOT to use

- 3′-only sequencing data (e.g. DropSeq-style) where the BAF shift signal is degraded because variants are measured only near gene 3′ ends, making BAF-expression correlations unreliable
- You need focal or gene-level CNV calls (extractSegmentSummary/gene.matrix) rather than arm-level association between allelic imbalance and expression
- Single-cell samples where a few high-read cells may dominate the pooled BAF shift signal, biasing the correlation

## Inputs

- Normalized bulk RNA-seq gene expression matrix (yale_meningioma, hg19 gene coordinates)
- BAFExtract output files (BAF/allelic shift signal) parsed with readBAFExtractOutput(sequencing.type='bulk')
- hg19 cytoband definitions and centromere coordinates from the CaSpER package
- Gene annotation from generateAnnotation(id_type='ensembl_gene_id', ishg19=T)

## Outputs

- Per-chromosome-arm Pearson correlation coefficients and P-values between BAF shift and smoothed expression signal (chr1p, chr6q, chr22q)
- Metrics table written to meningioma_cnv_benchmark_results.csv
- Arm-level CNV context from extractLargeScaleEvents(final.objects, thr=0.75) used for interpretation

## How to apply

Run the CaSpER pipeline on the normalized yale_meningioma expression matrix with BAF signal from readBAFExtractOutput(path='./meningioma_baf\', sequencing.type='bulk'), building the casper object via CreateCasperObject(sequencing.type='bulk', cnv.scale=3, loh.scale=3, method='iterative') and calling runCaSpER(object, removeCentromere=T, cytoband=cytoband, method='iterative') so pairwise scale comparisons of expression and BAF are performed on hg19 coordinates with centromeres removed. For each target chromosome arm, pair the arm-level BAF shift signal with the arm-level smoothed expression signal and compute Pearson correlations. Expected meningioma results: chr1p r=-0.43 (P=0.0008), chr6q r=-0.54 (P=0.0003), chr22q r=-0.14 (P=0.43), showing significant negative BAF-expression correlations on deletion-prone arms but not chr22q. Rationale: CaSpER integrates multiscale-smoothed expression and allelic shift signals, so a negative correlation indicates arms where LOH coincides with reduced transcript levels; write the metrics to meningioma_cnv_benchmark_results.csv.

## Related tools

- **CaSpER** (Produces the multiscale smoothed expression signal and BAF-integrated CNV calls (CreateCasperObject, runCaSpER) that are correlated at chromosome-arm level) — github.com/akdess/CaSpER
- **BAFExtract** (Generates the B-allele frequency shift signal from mapped RNA-seq reads without a heterozygous variant call set; its output is the BAF side of the correlation) — github.com/akdess/BAFExtract

## Examples

```
loh <- readBAFExtractOutput(path="./meningioma_baf\", sequencing.type="bulk")
```

## Evaluation signals

- chr1p Pearson r ≈ -0.43 with P=0.0008 and chr6q r ≈ -0.54 with P=0.0003 (significant negative BAF-expression correlation)
- chr22q correlation is expected to be non-significant (r ≈ -0.14, P=0.43), confirming specificity to deletion-prone arms
- BAF input successfully parsed via readBAFExtractOutput with sequencing.type='bulk' and control sample ids supplied to CreateCasperObject
- runCaSpER executed with removeCentromere=T and hg19 cytoband so arm coordinates exclude centromeres before correlating
- All correlation metrics exported to meningioma_cnv_benchmark_results.csv

## Limitations

- 3′-only sequencing degrades the BAF shift signal because it is measured only for variants near gene 3′ ends; no BAF shift generation model accounting for 3′ bias exists yet
- RNA editing, PCR duplication artefacts, and sequencing errors may impact the BAF shift signal in complex ways and bias correlations
- Non-significant correlations (e.g. chr22q, P=0.43) can occur on arms lacking strong allelic imbalance, so null results require arm-level CNV context for interpretation

## Evidence

- [abstract] CaSpER integrates the multiscale smoothing of expression signal and allelic shift signals for CNV calling.: "CaSpER integrates the multiscale smoothing of expression signal and allelic shift signals for CNV calling."
- [methods] we perform HMM on all scales of expression signal and integrate HMM segment states with Allele Frequency Shift Information: "we perform HMM on all scales of expression signal and integrate HMM segment states with Allele Frequency Shift Information"
- [readme] BAFExtract generates the B-allele frequency shifts from mapped reads alone without the need for a heterozygous variant call set.: [evidence span withheld — anchor status: external_doc]
- [discussion] the BAF shift signal may be adversely impacted because BAF shift will be measured for only the variants that are located close to the 3′ ends of the genes.: "the BAF shift signal may be adversely impacted because BAF shift will be measured for only the variants that are located close to the 3′ ends of the genes."
