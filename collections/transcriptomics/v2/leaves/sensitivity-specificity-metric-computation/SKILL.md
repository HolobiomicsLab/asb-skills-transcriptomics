---
name: sensitivity-specificity-metric-computation
description: Use when RNA-seq CNV calls (chromosome-arm, segment, or gene-level from
  CaSpER) must be validated against an independent DNA-based gold standard such as
  genotyping-array calls, reporting TPR/FPR and PPV per event category.
when_to_use_negative:
- No independent DNA-based gold standard (e.g., genotyping-array or SNP-array calls)
  is available for the samples — the paper notes ground truth CNV calls were unknown
  for the MM135 single-cell validation, where only concordance with HONEYBADGER could
  be shown instead
- Calls come from a non-RNA-seq caller or are already in the same modality as the
  gold standard, making TPR/FPR benchmarking against genotyping arrays redundant
- The goal is CNV detection itself rather than accuracy assessment of existing calls
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: CaSpER
  role: Produces the RNA-seq-derived CNV calls being evaluated and provides calcROC
    for computing TPR/FPR/PPV against the gold standard
  repo: https://github.com/akdess/CaSpER
- name: BAFExtract
  role: Generates the B-allele frequency shift signals from mapped RNA-seq BAM reads
    used by CaSpER to correct CNV calls prior to accuracy evaluation
  repo: https://github.com/akdess/BAFExtract
- name: TCGABiolinks
  role: Downloads TCGA-GBM bulk RNA-seq BAM/expression data and matching genotyping-array
    CNV gold-standard calls from GDC
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-13779-x
    title: CaSpER identifies and visualizes CNV events by integrative analysis of
      single-cell or bulk RNA-sequencing data
  source_content_hash: sha256:3b7135d7d0be420e9edad2b37040e7a2490ca4f7cc16b4f9e112932b6794eea6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/akdess/CaSpER
  tools:
  - BAFExtract
  - CaSpER
  - TCGABiolinks
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# sensitivity-specificity-metric-computation

## Summary

Compute TPR/FPR (and PPV) accuracy metrics for RNA-seq-derived CNV calls by benchmarking CaSpER's large-scale, segment-level, and gene-level calls against genotyping-array gold standards, as done for TCGA-GBM (n=160) in the CaSpER paper.

## When to use

Use when RNA-seq CNV calls (chromosome-arm, segment, or gene-level from CaSpER) must be validated against an independent DNA-based gold standard such as genotyping-array calls, reporting TPR/FPR and PPV per event category.

## When NOT to use

- No independent DNA-based gold standard (e.g., genotyping-array or SNP-array calls) is available for the samples — the paper notes ground truth CNV calls were unknown for the MM135 single-cell validation, where only concordance with HONEYBADGER could be shown instead
- Calls come from a non-RNA-seq caller or are already in the same modality as the gold standard, making TPR/FPR benchmarking against genotyping arrays redundant
- The goal is CNV detection itself rather than accuracy assessment of existing calls

## Inputs

- CaSpER final CNV call sets (large-scale chromosome-arm matrix from extractLargeScaleEvents, segment summary, gene-level matrix from gene.matrix)
- Genotyping-array CNV calls as gold standard for the same samples (e.g., TCGA-GBM n=160)
- Sample-level call lists stratified by event type (amplification vs deletion; large-scale, segment, gene-level)

## Outputs

- TPR and FPR values per event category (large-scale amp/del, gene-level amp/del)
- PPV and TPR values for segment-level amplifications and deletions
- Summary metrics table, e.g., tcga_gbm_cnv_accuracy_metrics.csv

## How to apply

Run CaSpER on bulk RNA-seq (e.g., TCGA-GBM n=160) with BAF shift signals from BAFExtract, building the object with CreateCasperObject (cnv.scale=3, loh.scale=3, method='iterative') and runCaSpER(removeCentromere=T). Harmonize calls across N x M pairwise scale comparisons using extractLargeScaleEvents(thr=0.75), extractSegmentSummary(gamma=6), and gene.matrix to obtain final large-scale, segment, and gene-level call sets. Match RNA-seq-derived calls to the genotyping-array gold standard per sample and event type, then use CaSpER's calcROC function to compute TPR and FPR (large-scale and gene-level amplifications/deletions) and PPV/TPR for segment-level events. Judge correctness by comparing computed metrics to the paper-reported values: large-scale amplification 79.5% TPR / 2% FPR, large-scale deletion 88% TPR / 3.5% FPR, gene-level deletion 77% TPR / 3.6% FPR, gene-level amplification 61.6% TPR / 3.4% FPR, and segment-level PPV/TPR of 68.5%/66% (amplification) and 70%/76% (deletion). Write all TPR/FPR/PPV values to a metrics table (e.g., tcga_gbm_cnv_accuracy_metrics.csv).

## Related tools

- **CaSpER** (Produces the RNA-seq-derived CNV calls being evaluated and provides calcROC for computing TPR/FPR/PPV against the gold standard) — https://github.com/akdess/CaSpER
- **BAFExtract** (Generates the B-allele frequency shift signals from mapped RNA-seq BAM reads used by CaSpER to correct CNV calls prior to accuracy evaluation) — https://github.com/akdess/BAFExtract
- **TCGABiolinks** (Downloads TCGA-GBM bulk RNA-seq BAM/expression data and matching genotyping-array CNV gold-standard calls from GDC)

## Examples

```
finalChrMat <- extractLargeScaleEvents(final.objects, thr=0.75); roc <- calcROC(rnaCalls, goldStandardCalls); write.csv(roc, "tcga_gbm_cnv_accuracy_metrics.csv")
```

## Evaluation signals

- Large-scale amplification TPR/FPR should approximate the reported 79.5% / 2%
- Large-scale deletion TPR/FPR should approximate the reported 88% / 3.5%
- Gene-level deletion TPR/FPR should approximate 77% / 3.6% and gene-level amplification 61.6% / 3.4%
- Segment-level PPV/TPR should approximate 68.5%/66% for amplifications and 70%/76% for deletions
- Metrics table (tcga_gbm_cnv_accuracy_metrics.csv) is populated with TPR/FPR/PPV for all event categories and consistent with per-category expectations

## Limitations

- CaSpER accuracy is category-dependent: gene-level amplification detection is weakest (61.6% TPR), so expect lower sensitivity for focal gains than for deletions or large-scale events
- Ground-truth CNV calls may be unknown for some datasets (e.g., MM135 single-cell), limiting evaluation to concordance with another caller rather than true accuracy
- For 3'-only sequencing (e.g., DropSeq), the BAF shift signal may be adversely impacted, degrading call quality and hence benchmarked accuracy

## Evidence

- [results] CaSpER achieves 79.5% TPR / 2% FPR for large-scale amplifications, 88% TPR / 3.5% FPR for large-scale deletions: [evidence span withheld — anchor status: not_found]
- [methods] calcROC evaluation workflow: [evidence span withheld — anchor status: not_found]
- [methods] large-scale event extraction threshold: "finalChrMat <- extractLargeScaleEvents (final.objects, thr=0.75)"
- [abstract] B-allele frequency shift signal without variant call set: "CaSpER integrates the multiscale smoothing of expression signal and allelic shift signals for CNV calling."
- [discussion] unknown ground truth for MM135: "Even though the exact ground truth CNV calls are unknown for this study, we show the concordance of CaSpER CNV calls with HONEYBADGER"
- [discussion] 3' bias impacts BAF signal: [evidence span withheld — anchor status: not_found]
