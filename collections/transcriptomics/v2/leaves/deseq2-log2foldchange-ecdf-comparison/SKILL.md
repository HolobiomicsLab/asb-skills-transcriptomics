---
name: deseq2-log2foldchange-ecdf-comparison
description: Use when you have a filtered DESeq2 results data frame (gene_id, log2FoldChange,
  padj) and defined gene sets (e.g. seed-match targets vs. non-targets, or miRNA predicted
  targets vs. background) and need to test whether the target set shows a cumulative
  positive or negative log2FC shift.
when_to_use_negative:
- Input is raw counts or an unfiltered DESeq2 results table that has not been passed
  through filter_res — the ECDF comparison expects a filtered data frame with gene_id,
  log2FoldChange, and padj
- The analysis goal is detecting seed matches in transcript sequences itself rather
  than testing cumulative expression shifts — use SeedMatchR()/full_search() for sequence
  search instead
- Only a single differentially expressed gene is of interest (no gene-set versus background
  distribution) — a per-gene differential test, not an ECDF shift test, is appropriate
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SeedMatchR
  role: Provides deseq_fc_ecdf() to compare log2(Fold Change) of two gene sets via
    ECDF plus statistical test; also filter_res() to preprocess DESeq2 results and
    SeedMatchR() to annotate seed-match counts used to define gene sets
  repo: https://github.com/tacazares/SeedMatchR
- name: DESeq2
  role: Produces the differential expression results data frame (log2FoldChange, padj,
    baseMean) that deseq_fc_ecdf consumes directly
- name: twosamples
  role: Provides the Wasserstein, DTS, and Wilcoxon statistics used to recompute the
    two-sample ECDF comparisons under alternative stats_test settings
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae011
    title: 'SeedMatchR: identify off-target effects mediated by siRNA seed regions
      in RNA-seq experiments'
  source_content_hash: sha256:3b4dc627b10c032388f01873504e53c1a929859dd3b4f4ac0b5956f9ce2a4fa8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tacazares/SeedMatchR
  tools:
  - AnnotationHub
  - DESeq2
  - R
  - SeedMatchR
  - twosamples
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# deseq2-log2foldchange-ecdf-comparison

## Summary

Compare the empirical cumulative distribution of DESeq2 log2FoldChange values between a seed-target gene set and a background gene set, testing for cumulative differential expression shifts with KS, Wasserstein, DTS, or Wilcoxon statistics via SeedMatchR's deseq_fc_ecdf.

## When to use

Use when you have a filtered DESeq2 results data frame (gene_id, log2FoldChange, padj) and defined gene sets (e.g. seed-match targets vs. non-targets, or miRNA predicted targets vs. background) and need to test whether the target set shows a cumulative positive or negative log2FC shift.

## When NOT to use

- Input is raw counts or an unfiltered DESeq2 results table that has not been passed through filter_res — the ECDF comparison expects a filtered data frame with gene_id, log2FoldChange, and padj
- The analysis goal is detecting seed matches in transcript sequences itself rather than testing cumulative expression shifts — use SeedMatchR()/full_search() for sequence search instead
- Only a single differentially expressed gene is of interest (no gene-set versus background distribution) — a per-gene differential test, not an ECDF shift test, is appropriate

## Inputs

- Filtered DESeq2 results data frame containing columns gene_id, log2FoldChange, padj (optionally baseMean, symbol, seed-match count columns like mer7m8/mer8)
- Gene list of lists: target gene IDs (seed-match targets or miRDB-predicted miRNA targets, score >= 90) and background gene IDs
- Optional: stats_test choice among 'KS', 'Wass', 'DTS', 'Wilcoxen' and alternative (e.g. 'greater')

## Outputs

- Test statistic and p-value per comparison (e.g. Dstat and p-value for KS), exportable to CSV such as mirna_ecdf_ks_results.csv or ecdf_stat_test_results.csv
- ECDF plot of log2FoldChange distributions for target versus background gene sets (e.g. mirna_target_ecdf.png, Fig. 1F style)
- Optional per-test results table combining KS, Wass, DTS, and Wilcoxen statistics

## How to apply

First filter the DESeq2 results with SeedMatchR::filter_res (tuning fdr_cutoff and fc_cutoff, e.g. fdr_cutoff=1, fc_cutoff=0 for the Schlegel_2022 Ttr D1 30 mg/kg example, which reduces 32883 genes to a smaller set) and remove NA entries. Define gene sets from the annotated results — e.g. mer7m8 targets as gene_id with mer7m8 >= 1, background as mer7m8 == 0 — or, for the miRNA-perturbation use case, miRDB-predicted targets (score >= 90) of the top-10 expressed rat liver miRNAs (GSE172269) versus non-targets. Call SeedMatchR::deseq_fc_ecdf(res, list('Background'=..., 'targets'=...)) with stats_test set to 'KS', 'Wass', 'DTS', or 'Wilcoxen' and alternative='greater' for a one-sided positive shift; the KS variant bases inference on the D statistic (Dstat), the measured difference between the two ECDFs. Export the test statistic and p-value (e.g. to mirna_ecdf_ks_results.csv or ecdf_stat_test_results.csv) and render the ECDF plot for visual confirmation that the curves are separated.

## Related tools

- **SeedMatchR** (Provides deseq_fc_ecdf() to compare log2(Fold Change) of two gene sets via ECDF plus statistical test; also filter_res() to preprocess DESeq2 results and SeedMatchR() to annotate seed-match counts used to define gene sets) — https://github.com/tacazares/SeedMatchR
- **DESeq2** (Produces the differential expression results data frame (log2FoldChange, padj, baseMean) that deseq_fc_ecdf consumes directly)
- **twosamples** (Provides the Wasserstein, DTS, and Wilcoxon statistics used to recompute the two-sample ECDF comparisons under alternative stats_test settings)

## Examples

```
ecdf.results = deseq_fc_ecdf(res, list("Background" = background.list, "mer7m8" = mer7m8.list))
```

## Evaluation signals

- KS test with alternative='greater' yields a significant positive shift of the target log2FC distribution relative to background (small p-value, non-trivial Dstat)
- ECDF plot renders with Background and target curves clearly separated (Fig. 1F style)
- Results table contains all four statistics (KS, Wass, DTS, Wilcoxen) each with a p-value when replicating deseq_fc_ecdf across stats_test settings
- Gene sets partition correctly: e.g. mer7m8 set defined as mer7m8 >= 1 and mer8 == 0, mer8 set as mer8 >= 1, background as mer7m8 == 0 and mer8 == 0
- Filtered input retains the required columns (gene_id, log2FoldChange, padj) and NA entries removed before testing

## Limitations

- Results interpretation depends on filtered inputs; filter_res output dimensions vary with fdr_cutoff/fc_cutoff settings (README example shows 13582 x 8 before filtering), so cutoff choices affect the distribution being tested
- miRNA-target perturbation detection (e.g. via miRDB score >= 90 predictions and top-10 expressed liver miRNAs from GSE172269) is an extended use case and its validity rests on the prediction quality and background definition
- The one-sided 'greater' alternative assumes the expected direction of the shift; running with the wrong alternative would mask real effects

## Evidence

- [readme] Cumulative changes in the profile of genes expression can be visualized and tested with the emperical distribution function (ecdf) coupled with a statistical test such as the Kolmogorov-Smirnov test.: "Cumulative changes in the profile of genes expression can be visualized and tested with the emperical distribution function (ecdf) coupled with a statistical test such as the Kolmogorov-Smirnov test."
- [methods] deseq_fc_ecdf designed to work directly with a DESeq2 results data frame: "The function `deseq_fc_ecdf` is designed to work directly with a DESeq2 results data frame."
- [methods] filter_res used to filter results by log2FoldChange, padj, baseMean, and remove NA entries: "The function `filter_res()` can be used to filter a results file by log2FoldChange, padj, baseMean, and remove NA entries."
- [other] KS test basis is the D statistic, the measured difference between the ECDFs of the seed-match gene set and the background gene set: [evidence span withheld — anchor status: not_found]
- [other] positive increase in expression of predicted miRNA targets compared to background: [evidence span withheld — anchor status: not_found]
